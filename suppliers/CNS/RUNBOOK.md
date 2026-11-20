# CNS container price check: runbook for tab "Claud input"

**Purpose:** when Em drops a CNS XRORDDT3 file, load the lines shipping in one container into tab **Claud input** and check every line against the standard price list.
**Em provides only:** the file, plus the container and ETD, e.g. `C27 ETD 12/11`.
**Claude does everything below without asking**, then reports the summary (step 8).

---

## 0. Files and sheets

| What | Where |
|---|---|
| Input file | `XRORDDT3 dd mm yy Cxx.xls/.xlsx` (attached by Em) |
| Target sheet | **CNS 2026 ชน Price AKJ**: `14OelXk5j7NDckN0ITjvc3ga3pyipedmOZzkNYaWJsNc` |
| Target tab | **Claud input** (sheetId `554134813`) |
| History tab | **CNS_Old** (sheetId `42373617`): all 2026 containers C1…; header on row 7, data from row 8 |
| Last-year prices | Tab **XRBUYOR1** (I = prodcode, K = THB price, X = year) |
| Standard price list | **AKJ_Standard_Price_List_Towel_Bathrobe**: `1BLhG19g-ILbZfCpdbCchXt_OdgSBkYQvE-cE-jueEW0` |
| ↳ Std codes | Tab `Price_List`, column A (Std Code). AC/X must use a code that exists here |
| ↳ USD/lb rates | Tab `Rate_Card` |
| ↳ Running-stock table | Tab `Sheet1` (rows "running stock towels,white"): Product Codes + per-piece price |

**Data rule:** never edit source data (XRBUYOR1, CNS_Old, the price list, the xls). If something looks wrong, tell Em.

---

## 1. Read the Excel file

1. Copy the attachment to a new folder. If the file is `.xls`, convert it to xlsx first with `soffice --headless --convert-to xlsx`. Open it with openpyxl (`data_only=True`).
2. Use the **container sheet** (usually the last sheet, named like `C27`), not the raw `XRORDDT3 dd mm yy` sheet.
3. Find the header row. It is the row whose column B = `Date` (normally row 6). The columns are:
   `B Date · C Purchase order · D Supplier · E Product Code · F Product Full Name · G Remark · H Order · I Price/Unit · J Discount · K GB Qty · L Non Received Qty · M Amount · N Section`, then one or more `etd dd/mm` columns, each followed by an `amount` column (sometimes a `balance` column too).
4. **Pick the ETD column** whose header matches the ETD Em gave (e.g. `etd 12/11`; ignore case and spaces). Only use that column.
5. **Filter:** keep only rows where that ETD cell is a number other than 0 **and** Product Code (E) is not blank.
   - Skip the total row (blank code), and any `discount` or `total` rows at the bottom. **Read those bottom rows anyway:** any discount (e.g. `discount 21500369 … -2720`) goes into the note of the matching line.
6. Check the result: the row count, Σ ETD qty and Σ ETD qty × Price/Unit must equal the file's total row for that ETD. Report all three numbers.

---

## 2. Where to write in Claud input

1. Read `'Claud input'!A:A` to find the last used data row.
2. If there is **anything below** that row (Em sometimes keeps test cells there), **insert** the needed number of blank rows right after the last data row instead of writing over it (`insertDimension`, inheritFromBefore).
3. The tab often has a **basic filter** (e.g. showing only the latest container). copyPaste fails while rows are hidden, so:
   1. Save the filter spec (`get_spreadsheet` fields `sheets.basicFilter`).
   2. Run `clearBasicFilter`.
   3. Copy the last existing data row (all columns A:AE) onto the new rows with `PASTE_NORMAL`. This brings the formats and formulas.
   4. Write the values (step 3).
   5. Put the filter back with `setBasicFilter` and the same criteria. Add the new container to the hidden values only if Em asks.
4. **Never delete or move columns** without re-checking formulas. Deleting columns once broke U with #REF!.

---

## 3. Column layout (A → AE) and how each one is filled

| Col | Header | Fill with |
|---|---|---|
| A | ตู้ | Container from Em, e.g. `C27` |
| B | ETD | ETD date as a **serial number** (date − 1899-12-30), format dd/mm/yy |
| C | Supplier | `CNS` |
| D | Date | PO date (file B) as a serial number |
| E | Purchase order | file C |
| F | Product Code | file E **as text** |
| G | Product Full Name | file F |
| H | Remark | file G (blank if None) |
| I | Order | file H |
| J | Price/Unit | file I (USD/pc) |
| K | Discount | file J |
| L | GB Qty. | file K |
| M | Non Received Qty. | file L |
| N | Amount | file M |
| O | Section | file N |
| P | etd xx | ETD qty from the chosen ETD column |
| Q | amount | formula `=P{r}*J{r}` (comes with the copied row) |
| R | GB Status | `Pending` |
| S | เลขใบขน | blank |
| T | Invoice | container, e.g. `C27` |
| U | Price Check | **value** worked out in step 5. Not a formula |
| V | Price Check (เทียบกับปีก่อน) | formula, see below |
| W | Price Check (เทียบกับปีก่อน) | formula: last year's price in USD, see below |
| X | Std Code | Code from `Price_List!A` (step 4) |
| Y | Repeat/New | `Repeat` or `New` (step 4.6) |
| Z | Price ( before * lbs / 12) | USD/lb rate used for the chosen Repeat/New |
| AA | แปลงราคาซื้อเป็น std | formula `=IF(OR(J{r}="",AB{r}=""),"",J{r}/AB{r}*12)`: the PO price as USD/lb |
| AB | lbs/dz (ที่ใช้คำนวณ Std) | **Actual** lb/dz from the product name (not the code's lb). Blank for bathrobes and non-towels |
| AC | เคยซื้อมาก่อนไหมปีนี้ | formula, see below |
| AD | ตู้ไหนล่าสุด | formula, see below |
| AE | Cluade note | How the standard was worked out, plus tags (step 4.7) |

**V formula**
```
=IFERROR(IF(W{r}=J{r},"ราคาเท่ากับปีที่แล้ว",IF(J{r}<W{r},"ราคาลดลง "&TEXT((W{r}-J{r})/W{r},"0.00%"),"ราคาเพิ่ม "&TEXT((J{r}-W{r})/W{r},"0.00%"))),"")
```
**W formula** (2025 price in THB ÷ 35 → USD)
```
=iferror(IF(F{r}="","",IFERROR(MODE.SNGL(FILTER(XRBUYOR1!$K:$K,(XRBUYOR1!$I:$I=F{r})*(XRBUYOR1!$X:$X=2025)))/35,AVERAGE(FILTER(XRBUYOR1!$K:$K,(XRBUYOR1!$I:$I=F{r})*(XRBUYOR1!$X:$X=2025)))/35)),"")
```
**AD formula** (the latest earlier container in CNS_Old with the same code, by container number)
```
=IF(F{r}="","",IFERROR(ARRAYFORMULA(LOOKUP(2,1/((CNS_Old!$F$8:$F&""=F{r}&"")*(IFERROR(--REGEXEXTRACT(CNS_Old!$A$8:$A,"\d+"),99)<--REGEXEXTRACT(A{r},"\d+"))),CNS_Old!$A$8:$A)),"-"))
```
**AC formula** (compares with the price in that latest container)
```
=IF(F{r}="","",IF(AD{r}="-","ไม่เคยซื้อ",LET(p,IFERROR(--ARRAYFORMULA(LOOKUP(2,1/((CNS_Old!$F$8:$F&""=F{r}&"")*(IFERROR(--REGEXEXTRACT(CNS_Old!$A$8:$A,"\d+"),99)<--REGEXEXTRACT(A{r},"\d+"))),CNS_Old!$J$8:$J)),""),IF(p="","เคยซื้อ ราคาเปลี่ยน",IF(ABS(J{r}-p)<0.005,"เคยซื้อ ราคาเดิมกับ PO ล่าสุด","เคยซื้อ ราคาเปลี่ยน (PO ล่าสุด "&TEXT(p,"0.00")&")")))))
```
V, W, AA, AC and AD all come with the copied row. Only check they point at the new row.

---

## 4. Working out the standard price for each line

Read the product name, e.g. `handtowel doubleloop 15*30 4lbs white - - jac.thecharm - cn`:
type = 1st word · loop = 2nd word · size = `W*H` · lb = number before `lbs` (it may have a space, e.g. `5 lbs`, or a decimal, e.g. `16.8lbs`) · colour = 5th word · `jac`/`jacquard` anywhere = jacquard · `em.`/`emb` = embroidery.

### 4.1 Rates (USD/lb, Repeat / New), from Rate_Card
| Code | Repeat | New |
|---|---|---|
| RS-WH running stock white | 2.96 | – |
| DLP-WH double loop plain white | – | 3.05 |
| DLJ-WH double loop jacquard white | 3.05 | 3.05 |
| SLP-WH single loop plain white | 3.25 | 3.38 |
| SLJ-WH single loop jacquard white | 3.35 | 3.48 |
| DLP-LT / MD / DK | 4.10/4.27 · 4.17/4.34 · 4.22/4.39 | |
| SLP-LT / MD / DK | 4.15/4.32 · 4.25/4.42 · 4.30/4.47 | |
| STR-WB stripe | 4.80 | 4.99 |
| Embroidery add-on (USD/pc) | face 0.07 · hand 0.13 · bath 0.20 | **always use the Repeat price, for both Repeat and New** |
| Bathrobe (USD/pc) | waffle 61×50 9.03/9.50 · waffle emb 9.22/9.70 · velour 14.5/14.8 · velour emb 14.8/15.1 · shawl +0.50/+0.70 | |

### 4.2 Em's rules, applied in this order
1. **Running item.** If the Product Code is in the supplier's running-stock table (Price list `Sheet1`), use the table's per-piece price as the standard (Repeat only, no New). Code `TW-<base>-RS-WH`, rate 2.96.
   | Item | Product Codes (normal hem / adm hem) | USD/pc | Base code |
   |---|---|---|---|
   | face 12*12 1.5lb | 21100009 / 21100087 | 0.37 | FT-12x12-1.5LB |
   | hand 16*30 3.5lb | 21200015 / 21200201 | 0.86 | HT-16x30-3.5LB |
   | hand 16*30 4lb | 21200050 / 21200199 | 0.98 | HT-16x30-4LB |
   | hand 18*32 5lb | 21200065 / 21200200 | 1.23 | HT-18x32-5LB |
   | bath 30*60 16lb | 21500033 / 21500380 | 3.94 | BT-30x60-16LB |
   | bath 30*60 18lb | 21500044 / 21500339 | 4.44 | BT-30x60-18LB |
   | bath 30*60 20lb | 21500057 / 21500381 | 4.93 | PT-30x60-20LB |
   | bath 40*60 24.5lb | 21500071 / 21500392 | 6.04 | PT-40x60-24.5LB |
   | bathmat 18*28 8lb | 21300014 / 21300097 | 1.97 | BM-18x28-8LB |
   (bathmat 20*30 10lb = 2.46 in the table, but it has no Product Code there; see open decisions)
2. **Jacquard bath mat** (box frames, welcome, dryhere, rhombus, etc.): DLJ-WH 3.05 / 3.05. Code `TW-BM-…-RS-WH`, because Price_List only has RS-WH for bath mats.
3. **Colour.** Orange, darkgrey, darkbrown, black, brown, navy and royalblue count as **dark** (DLP-DK). Blue was treated as dark in C26 (not yet confirmed).
4. **Colour jacquard** = colour rate + 0.15 USD/lb (dark jacquard 4.37 / 4.54). Code: plain `…-DLP-DK`.
5. **White double loop plain**: Repeat = RS-WH 2.96, New = DLP-WH 3.05.
6. **White double loop jacquard**: DLJ-WH 3.05 / 3.05. **Single loop**: SLP-WH or SLJ-WH.
7. **Embroidery**: towel price + Repeat embroidery add-on for both Repeat and New.
8. **Bathrobe waffle 61×50 kimono** (a/rope or s/rope, no emb): BR-WAF-61x50, 9.03 / 9.50.
9. **Anything else** (pillow, topper, bath rug, tablecloth, slipper, belt, chest pocket…): no standard. Leave X–Z and AB blank. Note: `ไม่อยู่ใน price list (ไม่ใช่ผ้าขนหนู/เสื้อคลุม)`.

### 4.3 Per-piece standard
`Std = ROUND(rate × actual lb/dz ÷ 12, 2)` (+ embroidery). Work out both Std Repeat and Std New; they are used only for U and are not stored in their own columns.
*(The supplier rounds down, so many lines are 1 cent below Std. Switching to round-down is still waiting for Em's decision.)*

### 4.4 Std Code (X): must exist in Price_List column A
Format `TW-<FT|HT|BT|PT|BM>-<WxH>-<lb>LB-<rate code>`. If the real size or weight isn't in the list, use the **nearest weight in the same category** (Em OK'd this):
- FT → `12x12-1.5LB`
- HT → `16x30-3.5LB`, `16x30-4LB`, `18x32-5LB` (anything 5lb or more → 18x32-5LB; 4.5lb → 16x30-4LB)
- BT/PT → `27x54-14LB`, `30x60-16LB`, `30x60-18LB`, `PT-30x60-20LB`, `PT-40x60-24.5LB`, `PT-40x80-28LB`
- BM → `18x28-8LB`, `20x30-10LB` (only `-RS-WH` exists)

The rate code must be one that Price_List has for that size: RS-WH, DLP-WH, DLJ-WH, SLP-WH, SLJ-WH, DLP/SLP-LT/MD/DK, STR-WB.

### 4.5 lbs (AB)
Always the **actual** lb from the product name, so `Z × AB ÷ 12` gives back the standard.

### 4.6 Repeat/New (Y) and rate (Z)
- Choose **New** if the PO price is strictly closer to Std New than to Std Repeat. Otherwise choose **Repeat**.
- Z = the rate for that choice.
- For white double loop plain, the code follows the choice: Repeat → `-RS-WH` (2.96), New → `-DLP-WH` (3.05).

### 4.7 Cluade note (AE): write it in Thai, joined with ` · `
- Method text, e.g. `คำนวณจาก USD/lb × lb/dz ÷ 12 · double loop plain white (Repeat = RS-WH 2.96, New = DLP-WH 3.05)`
- Tags:
  - `[Running item] Product Code … อยู่ในตาราง running stock ของ supplier (… = x.xx USD/pc) → ใช้ราคาตารางเป็น Std, ไม่มีราคา New`
  - `[code ใกล้เคียง] ขนาดจริง <WxH-lbLB> ไม่มีใน price list → ใช้ <code size>`
  - `[code RS-WH] BM ใน price list มีแค่ RS-WH → rate ใช้ DLJ-WH 3.05`
  - `[code DLP-DK] ไม่มี code jacquard สี → rate = DLP-DK + 0.15`
- File notes: partial shipments (`ส่งบางส่วน: PO x ส่ง Cxx y`), discounts or remakes from the file's bottom rows, and the RFID note for 21200219 (`ดูแท็บ Note: ติด RFID + ค่าติด (+0.10$/pc)`).

---

## 5. Price Check (U): write it as a value

The inputs are J (PO price), Std Repeat (R), Std New (N), and V and W after the formulas have calculated. Read them back after writing.

1. If R exists and (|J−R| < 0.005, or N exists and |J−N| < 0.005), **or** V = `ราคาเท่ากับปีที่แล้ว` → `ราคาถูกต้อง`
2. Else if R does not exist and W is blank → `PO ไหม่`
3. Else if R does not exist → copy V's text (e.g. `ราคาลดลง 29.42%`)
4. Else if J > R → `ราคาสูงกว่า Std Repeat +0.xx`, and add ` (ไม่เกิน New)` if N exists and J ≤ N
5. Else → `ราคาต่ำกว่า Std -0.xx`

Conditional colours on U2:U1000 are already set (green correct · orange ไม่เกิน New · red สูงกว่า/เพิ่ม · yellow ต่ำกว่า/ลดลง · blue PO ไหม่). They are text-based, so they copy to other tabs.

---

## 6. Write order (Sheets connector)

1. `update_values` A:P, R:T, X:Z, AB, AE (values; dates as serial numbers; Product Code as text).
2. Read back V and W, work out U, then `update_values` on U.
3. Re-read the new rows to check (step 7).
4. Restore the basic filter.

`update_values` parses formulas. `update_formulas` fails, so don't use it.

---

## 7. Check before reporting
- Row count, total qty and total USD match the file.
- Every X code exists in Price_List column A.
- No #REF!, #N/A or #VALUE! in Q, V, W, AA, AC or AD.
- Spot-check 3 lines by hand: Z × AB ÷ 12 should equal the standard (or the running-table price for running items).

---

## 8. Report to Em (short, Thai/English as Em writes)
1. One line: container, ETD, lines / pcs / USD, which rows (e.g. rows 77–130).
2. A table of lines to look at:
   - price **below Std** by more than 0.02, or **above Std**;
   - running items below the table price (e.g. 21200201 at 0.69 vs 0.86);
   - **price changed vs the latest container** (AC = เคยซื้อ ราคาเปลี่ยน);
   - **new items** (ไม่เคยซื้อ);
   - big changes vs last year (V more than ±10%);
   - partial shipments, discounts, remakes.
3. Count of lines matching the standard, lines 1–cent off (rounding), and non-towel lines.
4. Data errors found in source tabs (just report them; Em fixes them).
5. Open decisions (below), only if they affect this container.

---

## 9. Open decisions (as of 19/11/26)
- **Rounding:** switch the standard to ROUNDDOWN to match the supplier? (not answered)
- **21300108** bath mat 20×30 10lb jac. twoboxframes adm hem, always 2.46 = the running-table price for 20×30. Treat it as a running item? (not answered)
- **Blue** = dark? (treated as dark in C26, not confirmed)
- **CNS_Old C26 rows:** ETD/PO dates look wrong (ETD 11/01/26 instead of 01/11/26). Reported; Em fixes.
