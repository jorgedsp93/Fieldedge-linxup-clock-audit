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
