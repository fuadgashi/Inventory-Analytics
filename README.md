# Inventory Analytics

Power BI inventory analytics: a semantic model and report (PBIP, DAX, TMDL) for the stock of a
distributor of fast-moving consumer goods — what is in stock and what it is worth under IAS 2,
whether it will sell before it expires, and how much to order for the rest of the year.

**One model, one report.** `Inventory.SemanticModel` holds the tables, relationships and
calculations; `Inventory.Report` holds three pages and a tooltip on it — see
[report.md](docs/report.md). Open `Inventory.pbip` in Power BI Desktop.

**Anonymised.** Every table, column and calculation is named in English, after a generic
source schema of SQL Server views ([data-source.md](docs/data-source.md)). Nothing in the
repository names a business, a person, a place or a source system behind the data, and the
connection parameters are placeholders. The only name is the author's, in the
[licence](#licence). The report is in English.

**No figures in the documents.** Totals, balances and row counts move with every refresh.
These documents hold structure, rules and decisions; the measurement behind a change lives
in its pull request.

## Documents

Read [maintenance.md](docs/maintenance.md) before you edit anything.

**The business and the model**

| Document | Covers |
|---|---|
| [business.md](docs/business.md) | the three questions the model answers, warehouses and organisational units, shelf life |
| [model.md](docs/model.md) | the star schema: facts with grain and window, dimensions, infrastructure tables, relationships, query groups |
| [data-source.md](docs/data-source.md) | the source views and their columns, the parameters, the fact window, refresh and publishing |
| [measures.md](docs/measures.md) | the core measure rules: one sales measure, VAT, per-unit figures, working days, the time functions |
| [dax-quality.md](docs/dax-quality.md) | how the DAX is kept to its standards: the audit, the colour measures, how a DAX change is tested, the measure register — measured by `scripts/dax_audit.py` |

**Subject areas**

| Document | Covers |
|---|---|
| [stock.md](docs/stock.md) | the two stock families, IAS 2 valuation, the cost check, expiry risk |
| [ordering.md](docs/ordering.md) | the rest-of-year forecast, the order requirement, cover in weeks |

**The report**

| Document | Covers |
|---|---|
| [report.md](docs/report.md) | pages, page navigation, slicers, the tooltip, labels, number formats, sort order |
| [mobile.md](docs/mobile.md) | the phone layout standard and its checker |

**Working on the project**

| Document | Covers |
|---|---|
| [maintenance.md](docs/maintenance.md) | engineering standards, naming, editing with Desktop open, the checks |
| [decisions.md](docs/decisions.md) | design decisions a future maintainer would otherwise re-litigate |
| [open-items.md](docs/open-items.md) | what the model needs before it runs on real data |
| [glossary.md](docs/glossary.md) | the model's own terms |
| [sources.md](docs/sources.md) | the external references behind every statement on how Power BI, DAX, SQL Server or a standard behaves |

**Sources.** A statement about how something outside the project behaves cites an entry
in [sources.md](docs/sources.md); a statement with no source is not written.

**Checks.** `scripts/docs_check.py` checks every link, anchor and citation in these
documents and in `CLAUDE.md`, and that each model object, page and script they name in
backticks exists. `scripts/model_check.py` checks that every reference in the model and the
report resolves, that relationships join keys of one type, and that nothing publishable
carries an e-mail address (but the licence's contact), an IP address or a currency symbol. `scripts/dax_audit.py` measures
the DAX against its standards and `scripts/rag_colors.py` keeps the colour measures in step
with their indexes — see [dax-quality.md](docs/dax-quality.md). `scripts/mobile_check.py`
checks the phone layout against [mobile.md](docs/mobile.md). All five run on every pull request
and every push to `main` (`.github/workflows/checks.yml`).

## Licence

© 2026 Fuad Gashi. All rights reserved. Published to show the work and the method; no licence
to use, copy, modify or distribute any part of it is granted. To use any part of it, email
fuaddi.g@gmail.com — permission is usually given. See [LICENSE](LICENSE).
