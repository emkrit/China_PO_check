# NT (Nantong): PO price check (runbook to write)

What we know so far:
- Fabric imports, checked per container (e.g. N14) in sheet **NT2026 ชน Price AKJ**: tab `Price Check` against the `XRBUYOR1` purchase history.
- The check runs **twice a month** on newly opened POs. Every PO is checked before payment.
- NT has a supplier quote (standard price). **Gatekeeper rule:** stop if the price is above the quote for that date.
- Related: Em is building **AKJ PO Price Master**, a separate Google Sheet that acts as the gatekeeper for every supplier's price trend.
  - ZS (Hebei Xiucheng) has no standard price, so it is compared with its price trend; anything over +5% needs approval.
- ERP notes:
  - The rate is fixed at 35 THB/USD.
  - ERP widths /110 = 112" and /120 = 122" share one SKU.
  - Codes with `#` are a slightly different colour.

To do: write `RUNBOOK.md` in the same format as `../CNS/RUNBOOK.md`.
