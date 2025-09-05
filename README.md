# FieldEdge ↔ Linxup Clock Audit (Google Apps Script)

Reconciles **FieldEdge** timesheets with **Linxup** trips in Google Sheets, flags **home clock‑ins/outs** and **overtime**, and writes a plain‑English reason in the **Note** column for every highlight.

## What it does

- Parses a FieldEdge timesheet CSV (“**Timesheet Raw**” sheet).
- Summarizes per employee per day: **first valid clock‑in** and **last valid clock‑out** (ignores rows ≤ 3 minutes by default).
- Pulls **Linxup trips/usage/stops/visits** in batched windows and loosely matches them to employees using a **pinned name map**.
- Builds a “**Clock‑in Audit**” sheet with:
  - FE clock in/out, Linxup trip start/end, start/end geofence labels, and Δ minutes.
  - **Conditional formatting**:
    - Overtime → **light yellow** (Note includes “Overtime 1.5” or “Overtime 2” when detectable).
    - **Home at Clock In** → soft red.
    - **Home at Clock Out** → red with white text.
  - **Note column** clearly explains every highlight (“Home at Clock In”, “Home at Clock Out”, “Overtime 1.5/2”, etc.).

## Why

This helps ops quickly see risky behavior such as clocking in from **Home** at the same time driving starts, or clocking out when the trip ends at **Home**, and provides a paper trail backed by GPS events.

## Requirements

- Google account with access to Google Sheets & Apps Script.
- Linxup API token.
- A FieldEdge CSV export with columns: `Employee`, `Status Date`, `Start Time`, `End Time`, `Duration`, `Pay Item`.

## Setup

1. **Create a Google Sheet** with two sheets:
   - **Timesheet Raw** (you will paste the CSV at A1)
   - **Clock‑in Audit** (script will populate this)
2. **Open Extensions → Apps Script** and paste the code from this repo into `Code.gs`.
3. **Script Properties** (Apps Script → Project Settings → Script properties):
   - `TIMEZONE` – e.g. `America/Toronto`.
   - `LINXUP_TOKEN` – your Linxup API token (secret).
   - *(Optional)* `AUDIT_SHEET_ID` – the spreadsheet ID if running the script from another container.
   - *(Optional)* `AUDIT_TRIPS_PAD_HOURS` – default 12.
   - *(Optional)* `MATCH_WINDOW_MIN` – default 120.
   - *(Optional)* `LABEL_WINDOW_MIN` – default 15.
   - *(Optional)* `LOOKER_URL` – dashboard URL (or leave the placeholder).
   - **`PIN_MATCHES_JSON`** – JSON mapping from the short timesheet name to the Linxup driver name you want to match (example below).
4. **Employee mapping example** for `PIN_MATCHES_JSON` (do **not** commit real names):
   ```json
   {
     "A. Worker": "Alice Worker",
     "B. Driver": "Bob Driver"
   }

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
(Optional) Add the on‑edit trigger via the menu (script does this automatically):

Clock‑in Audit → Reset tracker / Build Clock‑in Audit menu appears when the sheet opens.

Usage

Paste the FieldEdge CSV at A1 of the Timesheet Raw sheet.

From the menu Clock‑in Audit → Build Clock‑in Audit (CSV + Linxup).

Review the Clock‑in Audit sheet:

Yellow rows = Overtime (Note shows Overtime 1.5/Overtime 2 when detectable).

Soft red = Home at Clock In (times within ≤ 5 minutes, or FE clock‑in ≤ Linxup start).

Strong red + white text = Home at Clock Out (FE clock‑out within ≤ 5 minutes of Linxup end at Home).

Note column explains why a row is highlighted.

Configuration notes

Minimum FE duration to consider a row valid: 3 minutes (strict “> 3”).

Matching window for FE vs. Linxup candidates: 120 minutes by default.

Geofence label resolution tries Usage then Stops, then Visits, then raw trip text.

Privacy & Security

No secrets in code: All tokens/IDs are read from Script Properties.

No PII in repo: Do not commit real employee names; set them via PIN_MATCHES_JSON.

Add a .gitignore (see repo) to avoid accidentally publishing IDs from .clasp.json if you use clasp.

Troubleshooting

“LINXUP_TOKEN not set” – Set LINXUP_TOKEN in Script Properties.

“applyAuditFormattingRules_ is not defined” – This repo uses highlightOvertimeRows_() only; remove any older calls to applyAuditFormattingRules_.

If geofence labels look sparse, increase LABEL_WINDOW_MIN (default 15).
