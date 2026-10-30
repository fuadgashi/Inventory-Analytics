# Stock

[← README](../README.md)

## Two families, on purpose

- **All stock** — `Stock Quantity`, `Stock List Value`, `Stock Cost Value`,
  `Stock Regular Price Value`, `Stock Packs`, `Stock Pallets`. Every warehouse that holds
  stock.
- **Stock with a shelf life** — `Stock Quantity Expiry Scope`, `Shelf Life Days`,
  `Expiry Date` and every expiry measure, scoped to `FactStock[InExpiryScope]`: the
  warehouses that track shelf life ([business.md](business.md#warehouses-and-organisational-units)).

`Stock Quantity` and `Stock Quantity Expiry Scope` are two different questions, not one
measure with a missing filter. Do not "align" them.

Stock has no date: the date slicers do not move `FactStock`. The readiness slicer
(`SaleReadiness` › `StockStatus`) does: nothing selected is total stock; For Sale leaves
unsellable stock out.

## Valuation (IAS 2)

Stock is valued at cost, and no higher than it can be sold for [[ias-2]](sources.md#ias-2).

| Measure | What it is |
|---|---|
| `Stock Cost Value` | stock at **cost**: the book value |
| `Stock Carrying Amount` | the lower of cost and net realisable value, lot by lot |
| `Stock Write-Down` | cost less carrying amount |
| `Stock Without Sales Twelve Months` | cost of the stock of products with no sale in twelve months |
| `Stock List Value` | stock at the **list price, with VAT and before discount**: a management view |
| `Stock Regular Price Value` | stock at the twelve-month regular selling price: a management view |

- **Cost is the source's own**: `FactStock[UnitCost]`, the running weighted-average cost per
  warehouse and product. A weighted average is one of the cost formulas IAS 2 allows
  [[ias-2]](sources.md#ias-2). Only where the source holds no cost does a **derived cost**
  fill in (`Unit Cost Derived`): the twelve-month purchase cost, then the twelve-month cost on
  sales lines, then the last known cost.
- **A suspicious cost is listed, never overridden.** The `Stock\Cost Check` measures flag a
  cost more than twice, or less than half, the derived one — typically a bulk pack costed per
  pack against a per-unit purchase. The correction belongs in the source.
- **Net realisable value** is the product's twelve-month net selling price without VAT, less
  its division's `DistributionCostRate`. **Expired lots** are written down in full. A product
  with no sale in twelve months has no measurable selling price, so it stays at cost;
  `Stock Without Sales Twelve Months` shows it separately.
- **List price:** `Stock List Value` is quantity × `FactStock[ListPrice]`, row by row. Each
  warehouse keeps its own price, and every product in stock is valued, sold or not. Stock
  share, the expiry values and cover in weeks read stock on this basis; cover in weeks divides
  it by sales at the **regular** price, so both sides are before discount.
- **Prices from sales iterate products, not stock rows**, and remove the warehouse filter. A
  row-level pass over `FactStock` would carry each row's warehouse to the sales, and a
  warehouse that sells nothing would be valued at zero.

## Expiry and stock risk

The Expiry page asks one question: **will this stock sell before it expires?**

```
Stock Days of Cover Quantity  =  Stock Quantity Expiry Scope / Sales Quantity Daily Average
Shelf Life Days               =  days to expiry, weighted by quantity, fresh lots only
Cover to Shelf Life Ratio     =  cover / (shelf life x Working Days Share)
```

At or above **1** the stock expires before it sells. **A low ratio is the healthy case**,
which is why the bands are named for risk, not for stock level.

| # | Band | Rule | What it means |
|---:|---|---|---|
| 1 | Expired | the expired lots — a part of the stock, not of the product | already a loss: clear it, return it or write it off |
| 2 | At Risk | ratio ≥ 0.8, or the fresh stock expires today | may not sell before it expires — order less, sell down |
| 3 | Watch | 0.4 ≤ ratio < 0.8 | real exposure, not yet loss |
| 4 | Healthy | ratio < 0.4 | turns well inside its life |
| 5 | No Sales | no days of cover — the product does not sell | dead stock, a commercial problem |
| 6 | No Expiry Date | cover exists, the fresh lots carry no expiry date | a source data gap |

- **Every lot sits in exactly one band.** Expired lots are band 1; a product's fresh lots
  take the product's band (2–6), judged on the fresh stock only. A product with one expired
  piece shows that piece under Expired and the rest under its band, and the parts add up.
- **The thresholds live in one measure**, `Expiry Risk Band`. `Visual Filter` (the Band
  slicer), `Expiry Risk Severity`, `Expiry Risk Color` and `Expiry Risk Status` derive from it.
- **The band is computed once per row, in `Visual Filter`**, and nowhere else. The matrix
  rows are already the band's products, so the stock measures only ask whether Expired is
  selected. Computing the band inside each stock measure would repeat the costliest
  calculation on the page — average daily sales — for every cell.
- **Bands 5 and 6 stay apart** because they have different owners: sales, and the source data.
- **`ExpiryInterval`** groups remaining shelf life (Expired · 0–30 · 31–60 · 61–90 ·
  91–180 · 180+ days). The horizon chart shows Expired and 0–90 days, each its own column; the
  brand bar shows 0–90 days only, so expired value is not mixed into a brand's bar. 180+ is
  left out of both: it holds most of the value and would flatten every bar that needs action.

**Traps already hit:**

- **A blank ratio compares as 0** [[dax-operators]](sources.md#dax-operators), so an untested
  blank would fall into the healthiest band. Blank cases are tested first.
- **Shelf life is averaged over fresh lots only.** A plain average would read a lot weeks
  past its date as healthy beside fresh lots, and a product whose lots have all expired would
  come out blank and be banded No Expiry Date.
- **A ratio does not aggregate.** Above product grain `Cover to Shelf Life Ratio` averages the
  product ratios, and `Expiry Risk Status` is blank.
- **No filter on `Visual Filter` on the slicer that feeds it**: the measure would depend on
  itself.
- **Read the No Sales band in units or at cost**, never at a price from sales: it has none.

## Known limitations

What the model does today and a reader could mistake for something else:

- **Days to expiry are fixed at refresh** ([data-source.md](data-source.md#queries)); bands and
  intervals lag by the days since the last refresh.
- **`Expiry Date` is the latest expiry** among the lots in scope, not the earliest.
- **Stock measures built on `TwelMonth` move with the date slicers**: `Unit Cost Derived`,
  `Stock Carrying Amount` and `Stock Without Sales Twelve Months` read twelve months back from
  today within the slicers, so a past period changes a valuation of current stock.
- **Packs and pallets leave out products without pack sizes**: the division by a blank
  `UnitsPerCase` or `CasesPerPallet` is blank.
- **`Stock Value Share` removes every filter**, so the readiness, location and lot slicers do
  not change its denominator.
- **Some list-price measures read as "value"** on cards and in the tooltip (`Stock Value`,
  `Stock Expired`); they are at list price with VAT, not at cost.
