# Glossary

[← README](../README.md)

The model's own terms. Each rule is stated only in its own document; this page says what a
word means and where to read more.

| Term | Means | See |
|---|---|---|
| **All stock** | every warehouse that holds stock; `Stock Quantity` and its family | [stock.md](stock.md#two-families-on-purpose) |
| **Stock in expiry scope** | stock in the warehouses that track shelf life, `FactStock[InExpiryScope]` | [stock.md](stock.md#two-families-on-purpose) |
| **Fresh lot** | a lot not yet past its expiry date, or without one | [stock.md](stock.md#expiry-and-stock-risk) |
| **Band** | the expiry-risk class of a product's fresh stock — At Risk, Watch, Healthy, No Sales, No Expiry Date — or the expired lots, Expired | [stock.md](stock.md#expiry-and-stock-risk) |
| **Interval** | a span of days to expiry: Expired, 0–30 … 180+ days | [stock.md](stock.md#expiry-and-stock-risk) |
| **Days of cover** | working days the stock lasts at the average quantity sold per working day | [stock.md](stock.md#expiry-and-stock-risk) |
| **Cover to shelf life** | days of cover over remaining shelf life, on one basis; at or above 1 the stock expires before it sells | [stock.md](stock.md#expiry-and-stock-risk) |
| **Weeks of cover** | weeks the stock lasts at the average weekly sales: by quantity or by value | [ordering.md](ordering.md#cover-in-weeks) |
| **List value** | stock at the warehouse's list price, with VAT, before discount — a management view | [stock.md](stock.md#valuation-ias-2) |
| **Cost value** | stock at the source's weighted-average cost — the book value | [stock.md](stock.md#valuation-ias-2) |
| **Derived cost** | a product's cost from its purchases and sales, used where the source holds none and to check the cost it holds | [stock.md](stock.md#valuation-ias-2) |
| **Carrying amount** | the lower of cost and net realisable value (IAS 2) | [stock.md](stock.md#valuation-ias-2) |
| **Net price** | the price after discounts, with VAT | [measures.md](measures.md#prices-and-vat) |
| **Regular price** | the document price before discounts, with VAT | [measures.md](measures.md#prices-and-vat) |
| **Goods** | products that are not service or allowance lines, `DimProduct[IsGoods]` | [measures.md](measures.md#goods-only-for-cost) |
| **Working day** | Monday to Saturday | [measures.md](measures.md#working-days) |
| **Actual Year** | this year to date, cut at today | [measures.md](measures.md#time-windows-are-functions) |
| **YTD / YTG** | year to date / year to go: up to today / from today to 31 December | [measures.md](measures.md#time-windows-are-functions) |
| **Twelve Months** | from twelve months before today, within the date slicers | [measures.md](measures.md#time-windows-are-functions) |
| **Order requirement** | rest-of-year forecast less stock on hand | [ordering.md](ordering.md) |
| **Readiness** | whether stock may be sold: For Sale or Not for Sale, by its status | [report.md](report.md#slicers) |
| **Organisational unit** | a unit documents are issued from; a warehouse is one kind | [business.md](business.md#warehouses-and-organisational-units) |
