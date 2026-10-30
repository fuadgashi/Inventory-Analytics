# The model

[← README](../README.md)

**Star schema.** Facts and dimensions are separate tables. Every relationship is
many-to-one and single-direction, from the many side to the one side; none filters both
ways.

## Facts — grain and window

Grain is the first thing to know about a fact table; every measure depends on it.

| Fact | One row is | Window |
|---|---|---|
| **`FactStock`** | one **product lot in one location of one warehouse, at one stock status**. Carries the days to expiry, and the warehouse's cost and list price for the product | current stock only: a snapshot taken at each refresh, positive balances, warehouses that hold stock |
| **`FactInvoiceLine`** | one **document line**: a document, a product, a quantity, prices. Sales and purchase lines alike; `DimDocumentType` says which | 1 January of the prior year onwards |

`FactStock` has no date: it is the stock now, whatever the date slicers show.
`FactInvoiceLine` is the sales pace behind cover and the forecast, and the purchase prices
behind the derived cost.

## Dimensions

| Dimension | Describes |
|---|---|
| `DimProduct` | products, with their brand, division, pack sizes and weights; `IsGoods` tells goods from service and allowance lines |
| `DimDivision` | the commercial book a product is sold under, and the distribution-cost rate the stock valuation reads (`DistributionCostRate`) |
| `DimOrgUnit` | organisational units: the warehouses stock sits in, and the units documents are issued from — see [business.md](business.md#warehouses-and-organisational-units) |
| `DimDocumentType` | document types, with the flags `IsSalesOut` and `IsPurchase` |
| `DimDate` | the calendar: one row per day of the prior and the current year, marked as the date table; `Working Day` holds the working-day rule |

## Disconnected tables

| Table | Role |
|---|---|
| `ExpiryRiskBand` | the six expiry-risk bands, the Band slicer on Expiry; hosts `Visual Filter` — see [stock.md](stock.md#expiry-and-stock-risk) |
| `ExpiryInterval` | days-to-expiry intervals, read by `Stock Value by Expiry Interval` |

Neither relates to a fact: measures read the selection and filter `FactStock` themselves.

## Infrastructure

| Table | Role |
|---|---|
| `Measure` | the measure host: every business measure lives here, except `Visual Filter` |
| `Labels` | every user-facing string of the report, one row per owner, label type and culture |
| `LabelMeasures` | one text measure per label, each a `LabelText` lookup against `Labels`; also `User Culture` — see [report.md](report.md#labels) |

## Relationships

| From (many) | To (one) |
|---|---|
| `FactStock[ProductId]` | `DimProduct[ProductId]` |
| `FactStock[WarehouseId]` | `DimOrgUnit[OrgUnitId]` |
| `FactInvoiceLine[ProductId]` | `DimProduct[ProductId]` |
| `FactInvoiceLine[OrgUnitId]` | `DimOrgUnit[OrgUnitId]` |
| `FactInvoiceLine[DocumentTypeId]` | `DimDocumentType[DocumentTypeId]` |
| `FactInvoiceLine[Date]` | `DimDate[Date]` |
| `DimProduct[DivisionId]` | `DimDivision[DivisionId]` |

- **A single-direction relationship filters from the one side to the many side**
  [[relationships]](sources.md#relationships), so a product or division pick reaches both
  facts, and a date pick reaches sales only.
- **A warehouse pick reaches both facts too**: stock in that warehouse, and the sales issued
  from it. Measures that price stock from sales remove the warehouse filter first
  (`REMOVEFILTERS( DimOrgUnit )`), so a warehouse that sells nothing is still valued — see
  [stock.md](stock.md#valuation-ias-2).
- **Key columns share a data type** across every relationship. `ProductId` is text on all
  three tables, so its slicers offer a search box; the other keys are whole numbers.

## Query groups

Every query sits in one of four groups, declared in `model.tmdl`:

| Group | Queries |
|---|---|
| `Fact` | `FactStock`, `FactInvoiceLine` |
| `Dimension` | `DimProduct`, `DimDivision`, `DimOrgUnit`, `DimDocumentType` |
| `Model` | `DimDate`, `Measure`, `Labels`, `LabelMeasures`, `ExpiryRiskBand`, `ExpiryInterval` |
| `Parameters` | `pServer`, `pDatabase` |

## Captions

`cultures/en-US.tmdl` gives fields short captions for the report — `Stock Qty`, `Avg Week` —
where the name is long. Captions translate object metadata, not data values
[[translations]](sources.md#translations); the report text that is data lives in `Labels`.

## Data access options

`model.tmdl` sets two options of the M data engine, `legacyRedirects` and
`returnErrorValuesAsNull` [[data-access-options]](sources.md#data-access-options).

- **`returnErrorValuesAsNull`** makes an error in one cell of a query's result load as a
  null instead of failing the refresh, so a refresh can succeed with missing values. It does
  not touch DAX.
- **`legacyRedirects`** allows redirects from HTTPS to HTTP. The only data source is SQL
  Server, so there is nothing to redirect.
