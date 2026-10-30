# DAX quality

[← README](../README.md)

How the model's DAX is kept to its standards: nothing unused, no duplicate, one folder per
subject, every measure described and captioned, and every change proven not to move a number.

**Everything here is measured from the files.** `scripts/dax_audit.py` reads the model and the
report: every measure and function, and every place a visual, page or filter binds a measure.
It sees this report only: it cannot see Analyze in Excel or another report built on the
model.

| Command | Does |
|---|---|
| `python scripts/dax_audit.py` | prints every count below |
| `python scripts/dax_audit.py --strict` | the same, and exits 1 if a count that must be zero is not; runs on every pull request |
| `python scripts/dax_audit.py --register` | regenerates the [measure register](#measure-register) in this document |
| `python scripts/dax_audit.py --baseline baseline.dax` | writes the regression query of [How a DAX change is tested](#how-a-dax-change-is-tested) |
| `powershell -File scripts/dax_baseline.ps1 -QueryFile baseline.dax -OutCsv before.csv` | runs it against the model open in Desktop |
| `python scripts/dax_audit.py --compare before.csv after.csv` | compares two runs, measure by measure |
| `python scripts/dax_audit.py --json out.json` | writes every name behind every number |
| `python scripts/rag_colors.py` | writes the colour measure of every index that asks for one — see [Colour measures](#colour-measures) |

## What counts as used

A measure is **used** when a visual, page or filter of the report binds it, when a used
measure references it, or when a function a used measure calls reads it (`User Culture`, read
by `LabelText`). A measure no visual binds is still **kept when the docs name it** in
backticks, or name its folder: the cost check and the IAS 2 write-down are described in
[stock.md](stock.md#valuation-ias-2) as part of the design. Everything a kept measure calls is
kept with it.

## Counts that must be zero

`--strict` fails the pull request when any of these is not zero.

| Count | What it catches |
|---|---|
| unresolved report references | a visual bound to a measure that no longer exists |
| unused measures, unused label measures, unused functions | DAX nothing reaches: removed, not kept "in case" |
| exact duplicates | two measures with the same expression, ignoring spaces and comments |
| text typed into visuals | a field renamed inside a visual instead of captioned in the model — see [report.md](report.md#labels) |
| fields sharing one caption in a visual | two columns of one table or matrix under the same heading |
| colour measures out of date | a colour measure that no longer matches its index's annotation |
| measures without a description, format or folder | the standards in [maintenance.md](maintenance.md#engineering-standards); a colour measure is text and carries the text format hint instead of a format |
| variables without the `_` prefix | a `VAR` not in English snake_case with the `_` prefix — see [maintenance.md](maintenance.md#naming) |
| colours not from `RagColor` | a theme colour name typed into a measure instead of returned by `RagColor` |
| `IFERROR` | an error hidden instead of prevented: the model divides with `DIVIDE` and tests with `ISBLANK` |
| `FILTER` over a whole table | a filter argument built on a whole table: the model's filter arguments name columns |
| sales rebuilt inline | quantity × net price summed outside `Sales Net Price` — see [measures.md](measures.md#sales-is-one-measure) |
| visible measures without a caption | a field that would show its model name in a visual |

The audit also prints, without failing, the **shapes** that three or more measures share: the
expression with each measure reference replaced by a placeholder.

| Shape | Measures | Why they stay named measures |
|---|---|---|
| a measure repeated under a short name, in `Lookup\Stock` | 10 | the tooltip page lists them one by one, each under its own caption |
| `TwelMonth( [x] )` | 6 | the twelve-month window is already one function; each measure is a figure a visual or the valuation reads by name |
| `YearToDay( [x] )` | 4 | the same: the year-to-date window lives in the function, the measure names the figure |

## Colour measures

An index asks for a colour with one annotation on the measure:

- `annotation RagColor = UpIsGood`: a rise is good.
- `annotation RagColor = UpIsBad`: a rise is bad.

`python scripts/rag_colors.py` then writes its colour measure, hidden, in
`Formatting\RAG Color`: `RagColor( [index], centre )`, or with the sign turned for `UpIsBad`.
Edit the annotation, never the colour measure; `--check` runs on every pull request and fails
when a colour measure is out of step. A colour with its own rule (the expiry-risk colours) is
written by hand and left alone by the script.

## How a DAX change is tested

Run before the change and after it, on the same refresh.

| Test | How | Passes when |
|---|---|---|
| **Numbers unchanged** | `--baseline`, then `dax_baseline.ps1` on the model open in Desktop, before and after; `--compare` | every kept numeric measure, every year: no value appears or vanishes, and a number differs by less than one part in 10¹² |
| **No broken binding** | `python scripts/model_check.py` and `python scripts/dax_audit.py` | 0 findings; `unresolved report references: 0` |
| **The audit's counts** | `python scripts/dax_audit.py --strict` | every count that must be zero is zero |
| **Visuals render** | open the pages listed under *Used on* in the register | no visual shows an error or a blank |
| **Documents** | `python scripts/docs_check.py` | 0 findings |

**The baseline needs data.** `dax_baseline.ps1` reads the model open in Power BI Desktop, so
it runs only once the model is connected to a source and refreshed — see
[open-items.md](open-items.md). The baseline query itself is generated from the files and
needs nothing: `--baseline` writes one query per folder, every measure by `DimDate[Year]`, and
the PowerShell script runs them as one query and saves one row per year. The comparison allows
a relative difference of 10⁻¹², so two runs may differ only in their last digits; a real change
moves a figure by far more.

## Measure register

Every analytical measure in its place: its folder, whether it is visible, and the visuals
that use it. Regenerated by `python scripts/dax_audit.py --register`; edit the model, not
this table.

<!-- register:start -->
86 analytical measures.

### Calculation Roots

| Measure | Folder | Visible | Used on |
|---|---|---|---|
| Cost Price | Prices | yes | inside 2 measures (Cost Price Twelve Months, …) |
| Net Price ex VAT | Prices | yes | inside 1 measure (Net Price ex VAT Twelve Months) |
| Purchase Net Price ex VAT | Prices | no | inside 1 measure (Unit Cost Derived) |
| Regular Price | Prices | yes | inside 1 measure (Regular Price Twelve Months) |
| Sales Net Price | Sales | yes | inside 4 measures (Index Sales Net Price Actual/Prior YTD, …) |
| Sales Quantity | Sales | yes | inside 3 measures (Sales Quantity Actual Year, …) |
| Sales Selling Price | Sales | yes | inside 1 measure (Sales Selling Price Twelve Months) |
| Product Days Since Last Sale | Stock | yes | inside 1 measure (Stock Days Since Last Sale) |
| Stock Carrying Amount | Stock | yes | inside 1 measure (Stock Write-Down) |
| Stock Cost Value | Stock | yes | inside 3 measures (Stock Cost, …) |
| Stock List Value | Stock | yes | Expiry matrix, Stock card, Stock combo chart +1 |
| Stock Packs | Stock | yes | Order table |
| Stock Pallets | Stock | yes | Order table |
| Stock Quantity | Stock | yes | Order table |
| Stock Quantity Expiry Scope | Stock | yes | Expiry matrix, Expiry table |
| Stock Regular Price Value | Stock | yes | named in the docs |
| Stock Without Sales Twelve Months | Stock | yes | inside 1 measure (Stock Without Sales) |
| Stock Write-Down | Stock | yes | named in the docs |

### Calendar

| Measure | Folder | Visible | Used on |
|---|---|---|---|
| Working Days Passed | Working Days | yes | Order card |
| Working Days per Month | Working Days | no | inside 1 measure (Sales Net Price Monthly Average) |
| Working Days per Week | Working Days | no | inside 3 measures (Sales Net Price Weekly Average, …) |
| Working Days Remaining | Working Days | yes | Order card |
| Working Days Share | Working Days | no | inside 2 measures (Cover to Shelf Life Ratio Base, …) |

### Formatting

| Measure | Folder | Visible | Used on |
|---|---|---|---|
| Expiry Interval Color | RAG Color | no | Expiry column chart |
| Expiry Risk Color | RAG Color | no | Expiry matrix |
| Expiry Risk Severity | RAG Color | no | named in the docs |
| Index Sales Quantity Actual/Prior YTD Color | RAG Color | no | Order table |
| Stock Days of Cover Color | RAG Color | no | TT-Stock-Stat matrix |
| Stock Expired Color | RAG Color | no | TT-Stock-Stat matrix |
| Supplier Expiry Risk Color | RAG Color | no | Expiry bar chart |

### Lookup

| Measure | Folder | Visible | Used on |
|---|---|---|---|
| Stock At Risk % | Stock | yes | TT-Stock-Stat matrix |
| Stock Cost | Stock | yes | TT-Stock-Stat matrix |
| Stock Days of Cover | Stock | yes | TT-Stock-Stat matrix |
| Stock Days Since Last Sale | Stock | yes | TT-Stock-Stat matrix, Stock matrix |
| Stock Days To Next Expiry | Stock | yes | TT-Stock-Stat matrix |
| Stock Expired | Stock | yes | TT-Stock-Stat matrix |
| Stock Expiring 30 Days | Stock | yes | TT-Stock-Stat matrix |
| Stock Expiring 90 Days | Stock | yes | TT-Stock-Stat matrix |
| Stock Pallets Count | Stock | yes | TT-Stock-Stat matrix |
| Stock Products Count | Stock | yes | TT-Stock-Stat matrix |
| Stock Share | Stock | yes | TT-Stock-Stat matrix |
| Stock Units | Stock | yes | TT-Stock-Stat matrix |
| Stock Value | Stock | yes | TT-Stock-Stat matrix |
| Stock Weeks of Cover | Stock | yes | TT-Stock-Stat matrix |
| Stock Without Sales | Stock | yes | TT-Stock-Stat matrix |

### Prices

| Measure | Folder | Visible | Used on |
|---|---|---|---|
| Cost Price Twelve Months | Product Prices | yes | inside 1 measure (Unit Cost Derived) |
| Net Price ex VAT Twelve Months | Product Prices | yes | inside 2 measures (Stock Carrying Amount, …) |
| Regular Price Twelve Months | Product Prices | yes | inside 1 measure (Stock Regular Price Value) |

### Sales

| Measure | Folder | Visible | Used on |
|---|---|---|---|
| Index Sales Net Price Actual/Prior YTD | Net Price | yes | Stock card |
| Sales Net Price Actual Year | Net Price | yes | Stock card |
| Sales Net Price Monthly Average | Net Price | yes | Stock combo chart, Stock matrix |
| Sales Net Price Prior Year | Net Price | yes | inside 1 measure (Sales Net Price Prior Year YTD) |
| Sales Net Price Prior Year YTD | Net Price | yes | Stock card |
| Sales Net Price Twelve Months | Net Price | yes | inside 3 measures (Sales Net Price Monthly Average, …) |
| Sales Net Price Weekly Average | Net Price | yes | Stock matrix |
| Index Sales Quantity Actual/Prior YTD | Quantity | yes | Order table |
| Sales Quantity Actual Year | Quantity | yes | Order table |
| Sales Quantity Actual Year Daily Average | Quantity | yes | Order line chart, Order table |
| Sales Quantity Actual Year YTG | Quantity | yes | Order line chart, Order table |
| Sales Quantity Daily Average | Quantity | yes | inside 4 measures (Cover to Shelf Life Ratio Base, …) |
| Sales Quantity Prior Year | Quantity | yes | inside 2 measures (Sales Quantity Prior Year YTD, …) |
| Sales Quantity Prior Year YTD | Quantity | yes | Order table |
| Sales Quantity Prior Year YTG | Quantity | yes | Order table |
| Sales Quantity Twelve Months | Quantity | yes | Order line chart |
| Sales Selling Price Twelve Months | Selling Price | yes | inside 1 measure (Stock Weeks of Cover Value) |
| Sales Net Price Twelve Months Share | Share | yes | Stock combo chart |

### Stock

| Measure | Folder | Visible | Used on |
|---|---|---|---|
| Cover to Shelf Life Ratio | Balance | yes | Expiry matrix |
| Cover to Shelf Life Ratio Base | Balance | no | inside 1 measure (Cover to Shelf Life Ratio) |
| Expiry Date | Balance | yes | Expiry table |
| Order Requirement | Balance | yes | Order table |
| Sales Quantity Daily Average Expiry Scope | Balance | yes | Expiry matrix |
| Shelf Life Days | Balance | yes | Expiry matrix, Expiry table |
| Stock Days of Cover Quantity | Balance | yes | Expiry matrix |
| Stock Value Share | Balance | yes | Stock combo chart |
| Stock Weeks of Cover Quantity | Balance | yes | Order table |
| Stock Weeks of Cover Value | Balance | yes | Stock card, Stock combo chart, Stock matrix |
| Cost Exceptions Count | Cost Check | yes | named in the docs |
| Stock Cost Exception Value | Cost Check | yes | named in the docs |
| Unit Cost Derived | Cost Check | yes | inside 5 measures (Cost Exceptions Count, …) |
| Unit Cost ERP | Cost Check | yes | inside 1 measure (Unit Cost Ratio ERP/Derived) |
| Unit Cost Ratio ERP/Derived | Cost Check | yes | named in the docs |
| Expiry Risk Band | Expiry | yes | inside 4 measures (Expiry Risk Color, …) |
| Expiry Risk Status | Expiry | yes | Expiry matrix |
| Stock Expired Value | Expiry | yes | inside 2 measures (Stock At Risk %, …) |
| Stock Value by Expiry Interval | Expiry | yes | Expiry card, Expiry bar chart, Expiry column chart |
| Visual Filter | Expiry | yes | Expiry matrix |
<!-- register:end -->
