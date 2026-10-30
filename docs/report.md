# The report

[← README](../README.md)

**Three pages and a tooltip**, in English. The report opens on `Stock`.

| Page | Asks | Main visuals |
|---|---|---|
| `Stock` | what is in stock, and how long does it last? | stock value card; stock and sales share by brand; stock value and weeks of cover by brand; matrix by brand and product with stock value, weeks of cover, average monthly and weekly sales and days since the last sale |
| `Expiry` | will the stock sell before it expires? | lot table; expired, 30-day and 90-day cards; value at risk by brand; the cover-to-shelf-life matrix with its Band slicer; the expiry horizon by interval — see [stock.md](stock.md#expiry-and-stock-risk) |
| `Order` | how much is still to order this year? | sales to date and from today; working days passed and left; the order table with sales this and last year, stock in units, cases and pallets, cover in weeks, the rest-of-year forecast and the order requirement — see [ordering.md](ordering.md) |
| `TT-Stock-Stat` | tooltip: one product's stock at a glance | one table: value, cost, units, pallets, share, cover in weeks and days, expired and expiring stock, the share at risk, days to the next expiry, stock without sales, days since the last sale |

## Page navigation

Each page carries a **tab row** at the top right: one button per page, Stock · Expiry ·
Order, each reading its text from a label measure (`PageNav1 | Title` …). There is no menu
page and no section drawer; the three tabs are the whole navigation. Every page also has a
phone layout — see [mobile.md](mobile.md).

`TT-Stock-Stat` opens over the brand charts and the product matrix on `Stock`, and over the
brand bar and the cover matrix on `Expiry`. A tooltip page is filtered by the data point under
the cursor [[report-tooltips]](sources.md#report-tooltips): it shows the brand or product
hovered.

## Slicers

Every page has **five slicers in a row** (x = 725, 965, 1205, 1445, 1685), each under a
title button that reads a `… | SlicerTitle` label measure.

| Slot | Stock | Expiry | Order | Sync group |
|---|---|---|---|---|
| 1 | Lot | Lot | Date | `Lot` · `Date` |
| 2 | Division / Product | Division / Product | Division / Product | `Product` |
| 3 | Location | Location | Location | `Location` |
| 4 | Readiness | Readiness | Readiness | `Readiness` |
| 5 | Org / Depot | Org / Depot | Org / Depot | `Warehouse` |

- **A field sits in the same slot on every page**, and a synced slicer is the same on every
  page. Order needs the date in place of the lot: its date slicer carries a relative-date
  filter for the current period and no saved dates, so it never goes stale at New Year.
- **Division / Product** is `DivisionName` › `Brand` › `ProductName` › `ProductId`, divisions
  listed Z→A.
- **Readiness** is `SaleReadiness` › `StockStatus`: nothing selected is total stock; For Sale
  leaves unsellable stock out of the stock columns, and so out of `Order Requirement`. Sales
  columns never move with it: stock does not filter sales.
- **Org / Depot** filters both stock and sales issued from the unit — see
  [model.md](model.md#relationships).
- **The Band slicer on Expiry** (`ExpiryRiskBand[Band]`) is not synced: it exists on that page
  only. It is saved on Healthy.

## Labels

**No text is typed into a visual.** Page titles and subtitles, tab labels, visual titles and
subtitles, slicer titles, the captions under card values and every alt text are label
measures in `LabelMeasures`, each a call to `LabelText( LabelType, OwnerType, OwnerNumber )`,
which reads one row of `Labels`.

- **A missing row shows ⚠**, never a quiet fallback, so a gap is seen in review.
- **One language, English.** `User Culture` returns `en-US`, and `Labels` holds one row per
  label. To add a language: add its rows to `Labels`, and make `User Culture` return the
  viewer's culture.
- **Alt texts** are `Alt … | VisualSubtitle` label measures: the visual's kind, then its title
  and subtitle, or for the tooltip what it lists.
- **Field names** come from captions (`cultures/en-US.tmdl`): a field shows its caption in a
  visual, never a name typed into it.

## Number formats

- **`NumberFormat`** on the value measures, **`IndexFormat`** on the indexes — see
  [measures.md](measures.md#formats). A dynamic format string keeps the measure numeric
  [[dynamic-format-strings]](sources.md#dynamic-format-strings).
- **Tables and matrices show full numbers**: a format set on the field in the visual
  overrides the model's [[format-strings]](sources.md#format-strings).
- **Colours** are theme sentiment names — good, neutral, bad — from the colour measures, so
  they follow the theme. The theme is `cortnes-light.json`.

## Sort order

Two names sort by a hidden column that comes from the source:

| Column | Sorted by |
|---|---|
| `DimProduct[ProductName]` | `ProductSortOrder` |
| `DimOrgUnit[OrgUnitName]` | `OrgUnitSortOrder` |
