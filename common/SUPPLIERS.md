# Chinese suppliers: ERP codes

Source: the ERP supplier register **SUPPDET2** (`15laO9SvsZZiEtdkP1ZKbzYT5n_fdm6UiuUQ1JnXkric`, printed 04/05/26), with the list of Chinese supplier codes confirmed by Em on 20/11/26.
The ERP code is column `coorcode` (H) in **XRBUYOR1**. The short name (ชื่อเล่น) is what XRORDDT3 shows in column `Supplier`.

| Short name | ERP code | Company (SUPPDET2) | Products | Status |
|---|---|---|---|---|
| **CNS** | **ค.022** (`CNS2.`) | CANASIN INTERNATIONAL TEXIILE (JIANGSU) CO., LTD. | Towels, bath mats, bathrobes, bath rugs (`CN`) | ✅ current code, used from im25-151 |
| CNS | จ.006 (`CNS`) | JIANGSU CANASIN TEXTILE TECHNOLOGY CO., LTD. | Towels (`CN`) | ✅ earlier CNS code (2025 POs im25-001 → im25-241) |
| **HU** | **อ.042** | ANHUI XIAZHEN DOWN & FEATHER CO., LTD. | Duvet inserts, pillows, toppers (`HU`) | ✅ |
| **NT** | **น.008** | NANTONG AOMIAN TEXTILE CO., LTD | White fabric (KSN…) | ✅ |
| **ZS** | **ฮ.005** | HEBEI XIUCHENG HOME TEXTILE CO., LTD. | White fabric (KSJ…) | ✅ |
| **FX / FS** | **อ.085** (`FX2`) | ANHUI HONREN TEXTILE CO., LTD. | Pillows, toppers (`FX`) | ✅ current code |
| FX / FS | อ.015 (`FX`) | ANHUI ZHONGQIONHONYU HOME TEXTILE TECHNOLOGY CO.,LTD. | Pillows, toppers | ✅ earlier code. SUPPDET2 marks it **ไม่ใช้** (no longer used) |
| **AMEY** | **A-001** | JIANGSU AMEY HOSPITALITY CO.,LTD | – | ✅ |

## Rules
- **One supplier can have two codes** (old and new company or account): CNS = ค.022 + จ.006, FX = อ.085 + อ.015. For history and last-year price, **include both codes**, e.g. `(XRBUYOR1!$H:$H="ค.022")+(XRBUYOR1!$H:$H="จ.006")`.
- In the XRORDDT3 file, column `Supplier` shows the short name (e.g. `CNS2.` = ค.022). Use it to pick the runbook.
- Not in this list: SUPPDET2 also has **จ.020** "Jiangsu Canasin Textile Technology Co.,Ltd." (a third Canasin record). Em's list doesn't include it, so ask before using it.
