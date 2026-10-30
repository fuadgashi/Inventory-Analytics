# Ordering

[← README](../README.md)

The Order page asks: **how much of each product is still to order this year?** The answer
comes from sales pace and stock on hand; the model holds no purchase orders.

```
Order Requirement  =  Sales Quantity Actual Year YTG  -  Stock Quantity
```

Positive is the quantity still to order; negative means the stock already covers the rest of
the year.

## The rest-of-year forecast

`Sales Quantity Actual Year YTG` is the quantity expected to sell from today to 31 December:

```
this year's pace per working day  x  working days left  x  last year's season
```

- **This year's pace** is `Sales Quantity Actual Year Daily Average`: this year's quantity
  over the working days since the product's first sale this year (`AvgDay`), so a product
  launched in spring is not spread over the winter.
- **Working days left** is `Working Days Remaining`, from the calendar.
- **Last year's season** is last year's pace per working day after today's date, over its pace
  up to it, company-wide. Above 1 the rest of the year sells faster than the part already
  gone.
- **A product with no sales this year** takes last year's rest of the year,
  `Sales Quantity Prior Year YTG`, as it stands.

The two line charts beside it show the same pace from both sides: `Sales Quantity Twelve Months`
to date, and `Sales Quantity Actual Year YTG` from today, with the working days passed and left.

## Cover in weeks

| Measure | Stock | Over |
|---|---|---|
| `Stock Weeks of Cover Quantity` | `Stock Quantity` | this year's weekly quantity: `AvgDay( Sales Quantity Actual Year )` × `Working Days per Week` |
| `Stock Weeks of Cover Value` | `Stock List Value` | the twelve-month weekly sales at the regular price |

Both count weeks of working days since the product's first sale, not weeks with a sale: a
product sold in a few weeks only would otherwise read as many times its real pace, and its
cover far too short.

## Packs and pallets

`Stock Packs` and `Stock Pallets` express stock the way a warehouse plans: quantity over
`UnitsPerCase`, and over `UnitsPerCase` × `CasesPerPallet`. Both are blank at zero or below.

## Known limitations

- **The two cover measures use different paces**: quantity reads this year to date, value
  the rolling twelve months. On a seasonal product they disagree.
- **The forecast has no cap.** A product whose sales this year started late in a strong
  season projects that season's pace onto the rest of the year.
- **The readiness slicer reaches the requirement**: with For Sale selected, unsellable stock
  is left out of `Stock Quantity`, so `Order Requirement` rises by it.
