# Clock-in vs Trips Audit (FieldEdge ↔ Linxup)

Google Apps Script to reconcile FieldEdge timesheet clock-in/out with Linxup GPS trip data.  
This safe public version contains **no secrets or private data**. All configuration is loaded from Script Properties.

---

## What this script does

- Compares **FieldEdge timesheet entries** with **Linxup trip/stop/geofence data**.
- Identifies:
  - When employees actually left/arrived compared to when they clocked in/out.
  - Idling before departure and adjusts the “leave” time to the geofence exit (preferred) or stop end.
- Adds Pay Item details for the day and highlights overtime rows.
- Provides a simple menu inside Google Sheets:
  - Reset tracker (clear Timesheet Raw).
  - Open Looker Studio dashboard (optional link).
  - Build the full Clock-in Audit.

---

## Setup instructions

1. Open a new [Google Apps Script](https://script.google.com/).
2. Paste the full script file (`Code.gs`) from this repository.
3. Deploy it or bind it to the Google Sheet where you want audits to appear.
4. Run the helper `setSecretsOnce()` once, then open **Project Settings → Script Properties** and edit the keys with your real values (see below).
5. In your Sheet:
   - Ensure a sheet named **Timesheet Raw** exists. Paste your CSV there starting at cell A1.
   - The script will output results to a sheet named **Clock in Audit**.

---

## Required Script Properties

These are loaded at runtime — **do not commit real values**.

| Key                        | Example / Notes                                  |
|----------------------------|--------------------------------------------------|
| `TIMEZONE`                 | `America/Toronto`                                |
| `LINXUP_HOST`              | `https://appXX.linxup.com`                       |
| `LINXUP_BASE`              | `/ibis/rest/api/v2`                              |
| `LINXUP_TOKEN`             | *(Bearer token string)*                          |
| `AUDIT_SHEET_ID`           | Optional. Leave blank to use active sheet.       |
| `TIMESHEET_SHEET_NAME`     | `Timesheet Raw`                                  |
| `AUDIT_SHEET_NAME`         | `Clock in Audit`                                 |
| `AUDIT_TRIPS_PAD_HOURS`    | `12`                                             |
| `MATCH_WINDOW_MIN`         | `120`                                            |
| `HOME_FLAG_MIN`            | `5`                                              |
| `LABEL_WINDOW_MIN`         | `15`                                             |
| `MIN_MOVEMENT_MIN`         | `4`                                              |
| `MIN_MOVEMENT_KM`          | `2`                                              |
| `IDLE_ADJUST_MIN_MIN`      | `2`                                              |
| `IDLE_ADJUST_MAX_WINDOW_MIN` | `200`                                          |
| `MIN_CLOCK_DUR_MIN`        | `3`                                              |
| `LOOKER_URL`               | Optional dashboard URL                           |
| `PIN_MATCHES_JSON`         | JSON mapping of short names to full names        |

### Example for `PIN_MATCHES_JSON`
```json
{
  "A. Worker": "Alex Worker",
  "B. Tech": "Ben Technician"
}
Use placeholder values in public repos. Store real employee names only in your Script Properties.

Usage

Open the Google Sheet.

Paste timesheet CSV into Timesheet Raw starting at cell A1.

From the Clock-in Audit menu (added by the script):

Run Build Clock-in Audit.

Results are written to the Clock in Audit sheet:

Notes on idling, overtime, or home start/stop.

Δ In / Δ Out minutes differences.

Conditional formatting highlights overtime and “home at clock in/out”.

Safety & Privacy

No tokens or names are stored in this repo.

All sensitive values (API tokens, sheet IDs, employee names) must be set via Script Properties.

The fetch wrapper trims HTTP error bodies to avoid leaking secrets into logs.

This repo is safe to share publicly.

Development

Code is structured to be modular: config, utilities, API fetchers, and main audit builder.

Conditional formatting rules are applied programmatically.

You can adapt the PIN list to load from a hidden Config sheet instead of Script Properties if preferred.
