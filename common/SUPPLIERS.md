# Supplier codes

ERP supplier code = column `coorcode` (H) in **XRBUYOR1**. Use it to filter one supplier's purchase history.

| Supplier | ERP code (coorcode) | Products | Name suffix in ERP | Container prefix | XRBUYOR1 lines* | Status |
|---|---|---|---|---|---|---|
| **CNS** | **ค.022** | Towels, bath mats, bathrobes, bath rugs | `CN` | C (C24, C27…) | 1,385 (im25-151 → im26-593) | ✅ matches the C27 file (e.g. 21500342 in im25-151) |
| CNS (earlier code?) | จ.006 | Towels | `CN` | – | 330 (im25-001 → im25-241) | ⚠️ **to confirm**: CN towels in 2025 only, so possibly CNS's earlier code |
| **HU** | **อ.042** | Duvet inserts, pillows, toppers | `HU` | – | 534 | ✅ products end in `HU` |
| **NT** (Nantong) | **น.008** | White fabric KSN… | – | N (N14…) | 220 | ⚠️ **to confirm** (KSN fabric) |
| **ZS** (Hebei Xiucheng) | **ฮ.005** | White fabric KSJ… | – | – | 75 | ⚠️ **to confirm** (KSJ fabric) |
| **FS** | **อ.085** | Pillows, toppers | `FX` | – | 451 | ⚠️ **to confirm**: products end in `FX` |

\* Counted in the XRBUYOR1 tab of *CNS 2026 ชน Price AKJ* on 20/11/26 (rows 2–2996).

## How to use
- Last year's price for one supplier: add `(XRBUYOR1!$H:$H="<code>")` to the FILTER in the W formula if the same Product Code could come from more than one supplier.
- In the XRORDDT3 file, column `Supplier` shows the short name (e.g. `CNS2.`), not the ERP code.

When Em confirms a ⚠️ row, change its status to ✅.
