# Open items

[← README](../README.md)

What the model needs before it runs on real data. Each item says what is needed and links to
where it is explained. An item that is closed is removed from here in the same pull request
that closes it.

**The source views.** The queries read eight views in schema `inv` that a source system must
provide. *Needed:* the views, built over the source, with the columns listed in
[data-source.md](data-source.md#source-views).

**The connection.** `pServer` and `pDatabase` hold placeholders. *Needed:* the real server and
database, set in Desktop, never committed.

**The distribution-cost rate.** `DimDivision` carries a sample rate, the same for every division;
`Stock Carrying Amount` deducts `DistributionCostRate` from the net selling price. *Needed:*
each division's own rate, in the `DimDivision` query or added to `inv.Division`. See
[stock.md](stock.md#valuation-ias-2).

**A provision for slow-moving stock.** Stock with no sale in twelve months stays at cost and is
shown separately (`Stock Without Sales Twelve Months`). *Needed:* whether to provide for it,
and by how much.
