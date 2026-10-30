# The business

[← README](../README.md)

A distributor buys fast-moving consumer goods, keeps them in its warehouses and sells them
to shops and other customers. Much of the stock has a shelf life. The model answers three
questions about that stock:

1. **What is in stock, and what is it worth?** At cost, which is the book value, and at
   selling prices, which is how the commercial team reads it — see [stock.md](stock.md).
2. **Will it sell before it expires?** Days of cover against remaining shelf life, product
   by product — see [stock.md](stock.md#expiry-and-stock-risk).
3. **How much should be ordered?** The quantity expected to sell for the rest of the year,
   less what is on hand — see [ordering.md](ordering.md).

All three are answered from two sources: the current stock, lot by lot, and the sales and
purchase lines since 1 January of the prior year. There is no purchase-order or supplier
data; the order requirement comes from sales pace, not from supply.

## Warehouses and organisational units

Stock sits in warehouses; documents are issued from organisational units. **Not every unit
is a warehouse**, and not every warehouse holds goods for resale:

- **Warehouses that hold stock** are flagged in the source (`HoldsStock`); only their stock
  is loaded.
- **Warehouses that track shelf life** are flagged too (`TracksShelfLife`): the depots
  holding perishable goods for resale. The expiry measures read only these; warehouses
  holding non-perishable or non-resale stock are left out of expiry and kept in total stock.
- Other units issue documents but hold no stock. A stock-by-warehouse view is meaningful
  only for the warehouses that hold stock.

## Products, brands and divisions

- A **product** is one catalogue article, counted in its base unit — pieces or kilograms —
  and packed in cases and pallets.
- Its **brand** is the brand owner it is booked under, usually, not always, the company it is
  bought from.
- Its **division** is the commercial book it is sold under. Each division carries the
  distribution-cost rate the stock valuation reads.
- **Service and allowance lines** sit in the catalogue too. They carry no real cost or
  selling price and are kept out of cost and margin (`IsGoods`).
