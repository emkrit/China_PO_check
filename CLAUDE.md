# Instructions for Claude

This repository holds Em's runbooks for checking China import PO prices (AK&J Textile).

## When Em sends a file plus a line like `C27 ETD 12/11`
1. Work out the supplier from the file (column `Supplier`, e.g. `CNS2.`) or from the container prefix (C = CNS).
2. Read `common/CONVENTIONS.md`, then `suppliers/<SUPPLIER>/RUNBOOK.md`.
3. Follow the runbook exactly. Don't ask questions the runbook already answers.
4. Report the results in the format the runbook gives.

## When Em changes or adds a rule
- Update the runbook (and "Open decisions") in the same session, then commit with a clear message, e.g. `CNS: blue counts as dark`.

## Never
- Edit source data: ERP exports, XRBUYOR1, the history tabs, the standard price list. Report errors to Em instead.
- Delete or move columns in a Claude tab without re-checking every formula that refers to them.
