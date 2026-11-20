# China_PO_check

Price checks for AK&J's import POs from Chinese suppliers. Each supplier gets its own runbook. **The PO lines are checked against each supplier's standard price and its purchase history before payment, per container.**

The goals are to:
- catch price-entry mistakes and sudden price increases or decreases before paying;
- show for each line whether it's a repeat item at the same price, a repeat item with a changed price, or a new item;
- leave flagged lines for an employee to fill in a reason.

## Suppliers

| Supplier | What we buy | Google Sheet | Claude tab | History | Runbook | Status |
|---|---|---|---|---|---|---|
| **CNS** | Towels, bath mats, bathrobes, pillows | CNS 2026 ชน Price AKJ | `Claud input` | `CNS_Old`, `XRBUYOR1` | [suppliers/CNS/RUNBOOK.md](suppliers/CNS/RUNBOOK.md) | ✅ in use (C24–C27) |
| **HU** | – | HU 2026 ชน Price AKJ | `Claude` | `HU` | [suppliers/HU/README.md](suppliers/HU/README.md) | 🟡 runbook to write |
| **NT** (Nantong) | Fabric | NT2026 ชน Price AKJ | `Price Check` | `XRBUYOR1` | [suppliers/NT/README.md](suppliers/NT/README.md) | 🟡 runbook to write |

## How a run works (all suppliers)

1. **Em sends** the ERP export file (XRORDDT3) plus one line naming the container and ETD, e.g. `C27 ETD 12/11`.
2. **Claude** reads `CLAUDE.md` and then the supplier's runbook, and follows it step by step:
   1. pick the rows for that ETD from the file;
   2. add them to the supplier's Claude tab;
   3. check each line against the standard price and purchase history;
   4. fill every column.
3. **Claude reports** a short summary: totals, the lines to look at, and any data errors found in source tabs.
4. **Em** reviews the flagged lines and fixes any source-data errors herself.

Shared rules are in [common/CONVENTIONS.md](common/CONVENTIONS.md).

## Layout

```
CLAUDE.md               ← instructions Claude reads first
common/CONVENTIONS.md   ← rules shared by every supplier
suppliers/CNS/RUNBOOK.md
suppliers/HU/README.md
suppliers/NT/README.md
```

## Changing a rule

When Em decides a new pricing rule (e.g. "blue = dark"), update the supplier's runbook in the same session. That way the next container uses it. Open decisions are listed at the end of each runbook.
