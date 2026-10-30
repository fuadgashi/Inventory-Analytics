# Core measure rules

[← README](../README.md)

## Sales is one measure

Sales anywhere in this model means **`Sales Net Price`**: quantity × net price across
`FactInvoiceLine` under one mandatory filter, `DimDocumentType[IsSalesOut] = TRUE`. Without
the filter, purchases and every other document type count as sales. Products that do not count
toward sales never reach the model: the `FactInvoiceLine` query leaves them out at the source. **Never rebuild
this measure inline; call it.**

The quantity twin is `Sales Quantity`, under the same filter.

## Prices and VAT

- **Every price on `FactInvoiceLine` includes VAT**, as the source holds it.
- **`ExVatFactor` takes VAT back out**: `1 / (1 + VAT rate / 100)`, from the line's VAT rate in the source. A measure "ex VAT"
  multiplies by it.
- **`NetPrice`** is the document price after discounts; **`DocumentPrice`** is before them —
  the regular price.
- **Cost is without VAT.** `UnitCost` is without VAT on every line, as the source holds it.

| Measure | Price | VAT |
|---|---|---|
| `Sales Net Price` | net, after discounts | with |
| `Sales Selling Price` | regular, before discounts | with |
| `Net Price ex VAT` | net, per unit | without |
| `Regular Price` | regular, per unit | with |
| `Cost Price` | cost, per unit, goods only | without |
| `Purchase Net Price ex VAT` | purchase cost, per unit | without |

When someone asks for "sales", establish which basis they mean before answering.

## Goods only, for cost

Products with `IsGoods = FALSE` — service and allowance lines — carry no real cost or selling
price. `Cost Price` and every cost built on it filter them out, so unit cost × quantity gives
back the cost of sales. Sales values (`Sales Net Price`) take every product.

## Per-unit figures are quantity-weighted

Every per-unit figure is value over quantity, never an average over lines. The test: unit ×
quantity must give back the money. An average over lines counts a one-piece line as much as a
thousand-piece one.

## Working days

A working day is Monday to Saturday. The rule lives once, in the `DimDate` query
(`IsWorkingDay`); every working-day count reads `DimDate[Working Day]` and
`DimDate[Working Day Index]`.

- **`AvgDay`** divides the measure passed in by the working days of its window, ending on the
  latest date in context and never past today. On a product row the days start on the
  product's first sale in the window, company-wide, so a new product is not spread over
  months it was not sold; on any other row they start on the company's first sale, so the
  rows add up. Additive measures only.
- **Weekly and monthly averages** are `AvgDay` × `Working Days per Week` or
  `Working Days per Month`: per week or month of working days since the first sale, not per
  week with a sale.
- **Days of cover are working days.** A shelf life is in calendar days, so the expiry ratio
  puts it on the same basis: × `Working Days Share`, the calendar's share of working days.

## Time windows are functions

A time window is a call to a user-defined function, never written out in a measure.

| Function | Window |
|---|---|
| `YearToDay` | the date slicers' window within the current year, cut at today |
| `YearToGo` | the current year from today to 31 December |
| `PrevYear` | the window one year back, inside last calendar year |
| `TwelMonth` | from twelve months before today, within the date slicers |

`IndexDif` is the change against a prior value, (current − prior) / prior, with guards for
blank and negative priors. `RagColor` turns a value into a theme colour name — `good`,
`bad` or `neutral` — so colours follow the theme. An index's colour measure is generated
from an annotation — see [dax-quality.md](dax-quality.md#colour-measures).

## Formats

- **`NumberFormat`** shows at most three significant digits with K, M or B, as the measure's
  whole dynamic format string; it returns a quoted literal, so the measure stays a number
  [[dynamic-format-strings]](sources.md#dynamic-format-strings).
- **`IndexFormat`** shows an arrow and a percentage with one decimal, ▲ at or above its
  centre and ▼ below.
- **No currency symbol.** Values are plain numbers; the report shows no currency.
