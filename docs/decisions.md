# Design decisions

[← README](../README.md)

The rules a future maintainer would otherwise re-litigate, each with the reason it holds.

- **Everything is named in English, after the source views.** Tables, columns, measures and
  functions read the same words as the `inv` views ([data-source.md](data-source.md)), so a
  figure traces from a visual to the SQL under one name. Mapping a source system to these
  names happens once, in the views, not in the model.
- **Classifications are columns of the source, not lists in the queries.** Sales and purchase
  document types, warehouses that hold stock or track shelf life, goods and
  products that count toward sales: each is a flag in a view. A code list inside a query goes stale silently
  when the source adds a code.
- **Two stock families, not one measure with a filter.** All stock and stock with a shelf
  life answer different questions ([stock.md](stock.md#two-families-on-purpose)).
- **The source cost is the book; it is listed, never overridden.** A cost the model thinks
  wrong goes on the cost-check list for the source to correct; the model does not replace it.
- **Band thresholds live in one measure**, `Expiry Risk Band`, and the band is computed once
  per row, in `Visual Filter` ([stock.md](stock.md#expiry-and-stock-risk)).
- **Expired stock is a band of its own**, so an expired lot is never averaged into a healthy
  shelf life, and one At Risk band replaces finer splits that held too few products to act on.
- **Averages are per working day since the first sale**, not per day with a sale. Counting
  only days with a sale reads a rarely sold product as many times its real pace, and its cover
  far too short ([measures.md](measures.md#working-days)).
- **Cover in weeks by value is list price over regular price**: both before discount, so the
  average discount does not stretch the cover.
- **Sort keys come from the source, not DAX**, in the same view as the name they sort, so the
  name and its key arrive together on every refresh ([report.md](report.md#sort-order)).
- **No section navigation.** Three pages need no menu: the tab row on each page is the whole
  navigation, so there is no menu page, drawer or bookmark to keep in step.
- **One language, kept in `Labels`.** The report is English only, yet every UI text stays a
  `Labels` row: titles never sit hard-coded on the canvas, and a language is added as data
  ([report.md](report.md#labels)).
- **The calendar restates the fact window instead of reading it.** A calendar derived from a
  fact query depends on that query, and goes short with no error when the query returns
  nothing.
