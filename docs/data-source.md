# Data source, refresh and publishing

[← README](../README.md)

One source: **SQL Server**, reached through two Power Query parameters, `pServer` and
`pDatabase`, never a literal connection string. Both hold placeholders in the repository;
set them to the real server and database in Desktop.

## Source views

The queries read eight views in schema `inv`. Each row below lists the columns the queries
select. Building these views over a source system is the only source-specific work: every
table, column and calculation of the model is named after them.

| View | One row is | Columns read |
|---|---|---|
| `inv.StockLot` | a product lot in one location of one warehouse, at one stock status | `WarehouseId`, `LocationId`, `ProductId`, `LotNumber`, `StockStatusId`, `QuantityOnHand`, `ExpiryDate`, `UnitCost`, `ListPrice` |
| `inv.StockStatus` | a stock status | `StockStatusId`, `StockStatusName`, `IsSellable` |
| `inv.OrgUnit` | an organisational unit | `OrgUnitId`, `OrgUnitName`, `SortOrder`, `HoldsStock`, `TracksShelfLife` |
| `inv.Product` | a product | `ProductId`, `ProductName`, `SortOrder`, `DivisionId`, `BrandId`, `UnitsPerCase`, `UnitsPerTradeCase`, `CasesPerPallet`, `NetWeightKg`, `GrossWeightKg`, `IsGoods`, `CountsTowardSales` |
| `inv.Brand` | a brand owner | `BrandId`, `BrandName` |
| `inv.Division` | a product division | `DivisionId`, `DivisionName` |
| `inv.DocumentType` | a document type | `DocumentTypeId`, `DocumentTypeName`, `IsSalesOut`, `IsPurchase` |
| `inv.DocumentLine` | a sales or purchase document line, with its header's fields | `DocumentTypeId`, `OrgUnitId`, `ProductId`, `DocumentDate`, `Quantity`, `UnitPrice`, `NetUnitPrice`, `VatRate`, `UnitCost`, `SupplierUnitPrice` |

**Classifications live in the source, not in the queries.** Which document types are sales or
purchases, which warehouses hold stock and which track shelf life, which products are goods
and which count toward sales: each is a column of a view, so no query carries a list of codes.
A new document type or warehouse is classified where it is created; until it is, it counts
in no total.

**Units.** Costs are without VAT. `UnitPrice` (before discounts) and `NetUnitPrice` (after
them) include VAT; `VatRate` is a percentage. Weights are in kilograms.

**What the measures expect of the views.** The queries pass these columns through unchanged,
so the views must hold them as the measures read them:

- **`UnitCost` in `inv.StockLot` is blank where no cost is held, never 0.** A blank cost falls back
  to the derived cost; a 0 would value the stock at nothing — see
  [stock.md](stock.md#valuation-ias-2).
- **`UnitCost` in `inv.StockLot` is one value per warehouse and product**, repeated on each of its
  lots: the source's average cost. The cost check counts one exception per warehouse and
  product on that basis.
- **Every product has a division.** The carrying amount deducts the division's
  distribution-cost rate; a product with none has nothing deducted.

## Which model table reads which view

| Model table | Views |
|---|---|
| `FactStock` | `inv.StockLot`, `inv.OrgUnit` (holds stock, tracks shelf life), `inv.StockStatus` (name, sellable) |
| `FactInvoiceLine` | `inv.DocumentLine`, `inv.Product` (counts toward sales) |
| `DimProduct` | `inv.Product`, `inv.Brand` |
| `DimDivision` | `inv.Division`, plus the distribution-cost rate, set in the query |
| `DimOrgUnit` | `inv.OrgUnit` |
| `DimDocumentType` | `inv.DocumentType` |

`DimDate`, `Labels`, `ExpiryRiskBand` and `ExpiryInterval` are built inside the model, from
no source.

## Queries

Every source table is a **native SQL query** inside `Sql.Database(..., [Query = "..."])`
with an explicit one-hour `CommandTimeout`. The SQL does the work: joins, computed columns,
filters, the date window. The M after it sets types, marks keys and adds the few columns that are rules of the model.

- **A native query does not fold.** The mashup engine can't fold steps after a native SQL
  query [[pq-folding]](sources.md#pq-folding), so every filter lives inside the SQL text.
- **`FactInvoiceLine` joins `inv.Product` with INNER on purpose**, to leave out at the source
  the products that do not count toward sales. An inner join discards unmatched rows [[sql-from]](sources.md#sql-from):
  a line whose product is missing from the catalogue would drop out with its value.
- **The fact window** is `DocumentDate >= DATEFROMPARTS(YEAR(GETDATE()) - 1, 1, 1)`: the prior
  and the current year, rolling on 1 January. `DimDate` restates the same window; change the
  two together.
- **Days to expiry are counted at refresh.** `FactStock[DaysToExpiry]` is
  `DATEDIFF(DAY, today, ExpiryDate)` in the source query, so between refreshes it lags behind
  the calendar by the days since the last refresh.

## Refresh and publishing

The model is refreshed in Desktop, tested, and published from `Inventory.pbip`: publishing
creates a semantic model with the name of the file, and republishing replaces the model of
that name [[desktop-publish]](sources.md#desktop-publish). The order is always: commit → refresh →
test → publish.

**Credentials** live in each developer's Desktop and in the service, never in the repository.
