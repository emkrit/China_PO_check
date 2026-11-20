# Shared conventions (all suppliers)

## Data
- **Source data is read-only.** This covers the ERP exports (XRORDDT3, XRBUYOR1), history tabs and standard price lists. If something looks wrong (bad dates, prices at half value, missing years), report it to Em, who fixes it.
- **ERP exchange rate:** 35 THB/USD, fixed. Last-year prices in XRBUYOR1 are in THB, so divide by 35.
- **Last year's price** = the most common (`MODE.SNGL`) XRBUYOR1 price for that Product Code where year = 2025. If no single price is most common, use the average.
- Dates are written to Sheets as **serial numbers** (date − 1899-12-30) and formatted `dd/mm/yy`. The spreadsheet locale is th_TH.
- Product Codes are written as **text**.

## Google Sheets (connector)
- `update_values` parses formulas the way the Sheets UI does. Use it; `update_formulas` fails.
- `copyPaste` doesn't work while a basic filter is hiding rows. Save the filter, run `clearBasicFilter`, do the copy, then `setBasicFilter` with the same criteria.
- Add new rows after the last data row with `insertDimension`; never write over cells below the data. Copy the last data row (`PASTE_NORMAL`) to bring formats and formulas.
- Conditional formats on Price Check columns are text-based (no cell references), so they survive copy and paste into other tabs.

## Excel input
- `.xls` → convert with `soffice --headless --convert-to xlsx`, then read with openpyxl (`data_only=True`).
- Keep the input in its own folder and run Python with `-I`.
- Always check the row count, total qty and total amount against the file's own total row, and report all three.

## Report to Em
1. One line: container, ETD, lines / pcs / amount, and the sheet rows used.
2. A table of lines to look at only: off standard, price changed vs the latest container, new items, big changes vs last year, partial shipments, discounts or remakes.
3. Counts: how many lines are correct, 1-cent rounding, and lines with no standard.
4. Data errors found in source tabs.
5. Open decisions, only if they affect this container.

Keep it short. Em writes in English or Thai; answer the same way.
