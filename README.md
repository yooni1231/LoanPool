# 📷Camera Loan Management System

**Version 1.2.0** · Last updated 2026-06-17 · [Version History](#version-history)

A web-based camera inventory and loan tracking system for managing 3D cameras. Built with a single HTML file for the frontend and Google Apps Script as the backend, with Google Sheets as the database.

---

## Table of Contents

1. [Overview](#overview)
2. [Features](#features)
3. [File Structure](#file-structure)
4. [Quick Start (Offline / Local Mode)](#quick-start-offline--local-mode)
5. [Google Sheets Integration](#google-sheets-integration)
   - [Step 1 — Create a Google Sheet](#step-1--create-a-google-sheet)
   - [Step 2 — Set Up Apps Script](#step-2--set-up-apps-script)
   - [Step 3 — Deploy as a Web App](#step-3--deploy-as-a-web-app)
   - [Step 4 — Connect the Frontend](#step-4--connect-the-frontend)
6. [Sheet Structure](#sheet-structure)
7. [Using the App](#using-the-app)
8. [Data Fields Reference](#data-fields-reference)
9. [Troubleshooting](#troubleshooting)
10. [Version History](#version-history)

---

## Overview

This system helps teams track the location, status, and loan history of 3D cameras — **and the cable inventory that goes out with them.** It supports multiple simultaneous users via Google Sheets sync, works offline with browser localStorage, and requires no server or build tools — just two files.

```
index.html   ← Frontend (open in any browser)
Code.gs      ← Backend (paste into Google Apps Script)
```

---

## Features

| Feature | Description |
|---|---|
| **Inventory Table** | View all cameras with name badge, S/N, M/N, accessories, location, and dates |
| **Status Tracking** | Available / On Loan / Overdue / Returned / Unavailable |
| **Waiting List** | Queue of companies waiting for a specific model, with ETA tracking and a notes field |
| **Loan Calendar** | Monthly calendar view showing all active and past loan periods |
| **Model Summary** | Per-model breakdown of total / available / on loan / unavailable |
| **🧵 Cable Inventory** | Track power & ethernet extension cables by stock count. Availability is auto-computed as `total − (cables checked on cameras that are out/overdue + open standalone loans)` |
| **Standalone Cable Loans** | Loan cables out on their own (without a camera): pick the cable type, quantity, borrower, and dates; one-click ✔️ return |
| **Editable Cable Stock** | Change the total stock count per cable type directly in the table |
| **Loan History** | Per-camera history drawer showing every checkout and return event |
| **PDF Attachments** | Upload and view PDF files (e.g. shipping docs) attached to each camera |
| **Name Badges** | Color-coded badges for quick visual identification |
| **Accessory Checklist** | 18 selectable accessory items per camera (cables, mounts, boards, etc.); tracked cables also show live `available / total` stock |
| **Google Sheets Sync** | Auto-syncs cameras, waiting list, and cable loans to Google Sheets every 60 seconds; changes persist across devices |
| **Offline Mode** | Falls back to browser localStorage when no Sheet is connected |
| **One-click Return** | ✔️ button marks a camera (or a standalone cable loan) as returned and logs the event |

---

## File Structure

```
camera-loan-manager/
├── index.html     # Complete single-file frontend (HTML + CSS + JS)
└── Code.gs        # Google Apps Script backend (paste into Apps Script editor)
```

No npm, no build step, no server. Open `index.html` directly in a browser.

---

## Quick Start (Offline / Local Mode)

If you just want to try the app without Google Sheets:

1. Download `index.html`
2. Open it in any modern browser (Chrome, Firefox, Safari, Edge)
3. The app runs immediately using browser `localStorage` for storage

> ⚠️ **Note:** Data stored in localStorage is browser-specific and will be lost if you clear browser data. For persistent, shared storage across devices, connect Google Sheets (see below).

---

## Google Sheets Integration

This is the recommended setup for teams. All data is saved to a Google Sheet and synced automatically every 60 seconds.

### Step 1 — Create a Google Sheet

1. Go to [sheets.google.com](https://sheets.google.com) and create a new blank spreadsheet
2. Name it anything (e.g. `Camera DB`)
3. Leave it open — you'll need its Apps Script editor in the next step

---

### Step 2 — Set Up Apps Script

1. In your Google Sheet, click **Extensions → Apps Script** in the top menu bar

   > This opens the Apps Script editor in a new tab

2. In the editor, **delete all existing code** in the `Code.gs` file

3. **Copy the entire contents of `Code.gs`** and paste it into the editor

4. Press **Ctrl + S** (or Cmd + S on Mac) to save

5. In the editor, run `initSheets()` once to create the sheet headers:
   - Click the function dropdown (next to the ▶ Run button) and select `initSheets`
   - Click **▶ Run**
   - If prompted, click **Review Permissions → Allow**

   > This creates three sheets inside your spreadsheet: `cameras`, `waiting`, and `cables`, with the correct column headers and text formatting on date columns.

---

### Step 3 — Deploy as a Web App

1. In the Apps Script editor, click **Deploy → New deployment**

2. Click the gear icon ⚙ next to "Select type" and choose **Web app**

3. Fill in the deployment settings:

   | Setting | Value |
   |---|---|
   | Description | `Camera API` (or anything) |
   | Execute as | **Me** |
   | Who has access | **Anyone** |

4. Click **Deploy**

5. If prompted, click **Authorize access** and follow the Google sign-in flow

6. After deployment, you will see a **Web app URL** that looks like:
   ```
   https://script.google.com/macros/s/AKfycb.../exec
   ```

7. **Copy this URL** — you will need it in the next step

> ⚠️ **Important:** After editing `Code.gs`, the changes only go live when you publish a new version.
> - **First time:** use **Deploy → New deployment** (generates the URL above).
> - **After that, to keep the same URL:** use **Deploy → Manage deployments → ✏️ (edit) → Version: New version → Deploy**. This updates the existing deployment in place, so you don't have to re-paste the URL into the frontend.
> - **Deploy → New deployment** also works but generates a **new URL** each time, which you must then re-enter in the frontend's ⚙ settings.

---

### Step 4 — Connect the Frontend

1. Open `index.html` in your browser

2. Click the **⚙ Google Sheets 설정** button in the top-right corner

3. Paste the Web app URL from Step 3 into the URL input field

4. Click **연결 테스트** (Connection Test) to verify the connection
   - You should see: `✅ 연결 성공! 카메라 N개 확인됨`

5. Click **저장 & 데이터 불러오기** (Save & Load Data)

The app will immediately load all existing data from the Sheet and begin auto-syncing every 60 seconds. The sync status indicator in the top bar will turn green.

---

## Sheet Structure

The Apps Script backend manages three sheets automatically: `cameras`, `waiting`, and `cables`.

### `cameras` sheet (13 columns)

| Column | Field | Description |
|---|---|---|
| A | `id` | Unique numeric ID (auto-assigned) |
| B | `name` | Display name shown as badge (e.g. "Alpha") |
| C | `color` | Badge hex color (e.g. `#0052CC`) |
| D | `sn` | Serial Number |
| E | `mn` | Model Number (e.g. `ZVD2-M130`) |
| F | `items` | JSON array of included accessory IDs |
| G | `location` | Current location / company holding the camera |
| H | `lastUpdate` | Date of last record update (`YYYY-MM-DD`) |
| I | `out` | Checkout date (`YYYY-MM-DD`) |
| J | `inn` | Return date (`YYYY-MM-DD`) |
| K | `status` | `available` / `out` / `overdue` / `returned` / `unavailable` |
| L | `note` | Free-text notes |
| M | `pdfs` | JSON array of attached PDF objects |

> Columns H, I, J are formatted as **plain text** by `initSheets()` to prevent Google Sheets from auto-converting dates into serial numbers.

### `waiting` sheet (9 columns)

| Column | Field | Description |
|---|---|---|
| A | `id` | Unique numeric ID |
| B | `mn` | Model Number being requested |
| C | `company` | Company or contact name |
| D | `start` | Request date or "ASAP" |
| E | `returnPlan` | Intended loan duration (e.g. "3 weeks") |
| F | `eta` | Estimated ship date (`YYYY-MM-DD`) |
| G | `contact` | Email or phone |
| H | `prio` | Priority: `high` / `med` / `low` |
| I | `note` | Free-text notes (extra items needed, special requests, etc.) |

> Column F (`eta`) is formatted as **plain text** to preserve `YYYY-MM-DD` strings.

### `cables` sheet (8 columns)

Holds **standalone cable loans** (cables loaned out without a camera). Cables bundled with a camera are not stored here — they live in the camera's `items` array and are counted automatically.

| Column | Field | Description |
|---|---|---|
| A | `id` | Unique numeric ID |
| B | `sku` | Cable type ID (e.g. `pwr_ext_5m`, `eth_cable_25m`) |
| C | `qty` | Number of cables loaned out in this record |
| D | `borrower` | Company / person holding the cable(s) |
| E | `out` | Loan start date (`YYYY-MM-DD`) |
| F | `due` | Expected return date (`YYYY-MM-DD`) |
| G | `returned` | Return date (`YYYY-MM-DD`); empty while still on loan |
| H | `note` | Free-text notes |

> Columns E, F, G are formatted as **plain text** by `initSheets()`. A record counts as "on loan" (and reduces available stock) whenever `returned` is empty.

> **Total stock counts** per cable type are **not** stored in the sheet — they are kept in the browser's localStorage (see [Troubleshooting](#cable-totals-differ-between-devices)).

---

## Using the App

### Adding a Camera

1. Click **＋ 카메라 추가** in the **카메라 / 케이블 재고 현황** section
2. Fill in the required fields: **S/N**, **M/N**, **Current Location**
3. Optionally set a name and badge color for quick identification
4. Check off included accessories from the list — tracked cables show a live `가용 N/총 M` (available/total) badge so you can see remaining stock while loaning
5. Set the status and dates if the camera is already on loan
6. Attach any PDF documents (shipping labels, agreements, etc.)
7. Click **저장**

> When a camera's status is `대여 중` (out) or `연체` (overdue), every tracked cable checked on it is automatically deducted from that cable's available stock.

### Checking a Camera Out

- Open the edit modal (✏️) for the camera
- Set **Status** to `📤 대여 중`
- Fill in **Out date** and **Current Location** (the borrower's company name)
- Click **저장** — a loan history entry is created automatically

### Marking a Camera as Returned

- Click the **✔️** button on any active loan row
- The system automatically sets status to `반납 완료`, sets today as the return date, and resets the location to `# #`
- A return event is logged in the camera's loan history

### Cable Inventory

The **🧵 케이블 재고 현황** card shows, per cable type: total stock, how many are out with cameras, how many are out via standalone loans, total on loan, and current availability (or an `초과` warning if loans exceed stock).

- **Edit total stock:** click the number in the **총 재고** column, type the new value, and click away (or press Enter). Availability and the camera-modal badges update immediately.
- **Loan a cable on its own:** click **＋ 케이블 단독 대여**, choose the cable type (the dropdown shows `가용/총`), set quantity, borrower, and dates. The app blocks quantities above what's available.
- **Manage standalone loans:** the **📤 케이블 단독 대여 현황** table lists every open loan with ✔️ return, ✏️ edit, and 🗑️ delete buttons.

### Waiting List

- Click **＋ 추가** to add a company to the waiting queue
- Each waiting entry shows whether a matching model is currently available (green badge)
- The **ETA** (estimated ship date) can be edited inline directly in the table by hovering the date and clicking **✏ 수정** — no modal needed
- Click ✏️ to edit all fields of a waiting entry, including the **note** field

### Viewing Loan History

- Click **📋** on any camera row to open the history drawer
- Shows all checkout and return events with dates, locations, and notes

### Calendar View

- Displays all active and past loans as color-coded bars across the calendar
- Use the filter buttons to view a specific camera only
- Navigate months with **이전 / 다음** buttons

---

## Data Fields Reference

### Camera Status Values

| Status | Meaning |
|---|---|
| `available` | Ready to be loaned out |
| `out` | Currently on loan |
| `overdue` | On loan past the expected return date |
| `returned` | Loan completed, back in stock |
| `unavailable` | Not available for loan (repair, reserved, etc.) |

### Accessory IDs (used in `items` JSON array)

| ID | Label |
|---|---|
| `camera_body` | Camera Body |
| `psu_z2` | PSU (External AC-DC) Zivid 2/2+ |
| `psu_z3` | PSU (External AC-DC) Zivid 3 |
| `pwr_ext_5m` | Power Extension 5m |
| `pwr_ext_10m` | Power Extension 10m |
| `pwr_ext_20m` | Power Extension 20m |
| `pwr_ext_ra_3m` | Power Extension RA 3m (angled) |
| `eth_cable_5m` | Data cable Ethernet 5m |
| `eth_cable_10m` | Data cable Ethernet 10m |
| `eth_cable_25m` | Data cable Ethernet 25m |
| `eth_ext_ra_3m` | Data Extension Ethernet RA 3m (angled) |
| `cal_board_big` | Calibration board (Big) |
| `cal_board_small` | Calibration board (Small) |
| `tripod_mount` | Tripod mount |
| `robot_mount` | Robot mount |
| `stationary_mount` | Stationary mount |
| `data_cable_one` | Data cable - Zivid One+ |
| `check_note` | ⚠ Need to check Note or Files |

### Cable Stock SKUs (tracked in the Cable Inventory card)

These reuse the same IDs as the accessory checklist above, so checking a cable on a camera and tracking its stock stay in sync. Defaults are set in the `CABLE_STOCK` constant in `index.html` and can be overridden per browser via the editable **총 재고** column.

| SKU | Label | Default Total |
|---|---|---|
| `pwr_ext_5m` | Power 5m | 4 |
| `pwr_ext_10m` | Power 10m | 1 |
| `pwr_ext_20m` | Power 20m | 1 |
| `eth_cable_5m` | Ethernet 5m | 4 |
| `eth_cable_10m` | Ethernet 10m | 1 |
| `eth_cable_25m` | Ethernet 25m | 1 |

### Cable Loan Fields (used in the `cables` sheet)

| Field | Description |
|---|---|
| `sku` | One of the Cable Stock SKUs above |
| `qty` | Quantity loaned out in this record (≥ 1) |
| `borrower` | Company / person holding the cable(s) |
| `out` / `due` / `returned` | Loan start / expected return / actual return dates (`YYYY-MM-DD`) |
| `note` | Free-text notes |

### Waiting List Priority Values

| Value | Label |
|---|---|
| `high` | 🔴 High (immediate) |
| `med` | 🟠 Medium |
| `low` | 🟢 Low (date confirmed) |

---

## Troubleshooting

### Dates disappearing after saving

**Cause:** Google Sheets automatically converts `YYYY-MM-DD` strings into Date objects when the column format is not set to plain text.

**Fix:** Run `initSheets()` from the Apps Script editor. This sets the date columns to plain-text (`@`) format — cameras H/I/J (`lastUpdate`, `out`, `inn`), waiting F (`eta`), and cables E/F/G (`out`, `due`, `returned`) — which prevents automatic date conversion.

The `formatDate()` helper in `Code.gs` also handles reading back any values that were already converted, normalizing them back to `YYYY-MM-DD` strings.

---

### S/N and Color fields are swapped

**Cause:** The sheet's column order does not match the expected header order.

**Fix:** Run `initSheets()` once from the Apps Script editor. It overwrites row 1 with the correct headers without touching your data rows. The backend reads columns by header name (not by position), so as long as the headers are correct, data is always mapped correctly.

---

### Cable loans not syncing to the Sheet

**Cause:** The `cables` sheet or the cable handlers (`getCables` / `saveCable` / `deleteCable`) are missing from `Code.gs`, or the deployment wasn't updated after pasting the new code.

**Fix:**
- Make sure you pasted the latest `Code.gs` (the one that defines `SHEET_CABLES` and the cable CRUD functions).
- Run `initSheets()` once to create the `cables` sheet.
- Publish a new version (see [Step 3](#step-3--deploy-as-a-web-app)).
- Until the backend supports cables, cable loans are still saved to **localStorage** and work in that browser — they just won't sync across devices.

---

### Cable totals differ between devices

**Cause:** Total stock counts (the editable **총 재고** column) are stored in the browser's **localStorage**, not in the Sheet. Cable *loans* sync across devices, but the *totals* are per-browser unless the backend is extended to store them (e.g. via `PropertiesService`).

**Fix:** Set the same totals on each device, or extend `Code.gs` with a `saveCableTotal` handler and return `cableTotals` from `getAll` (the frontend already reads/writes these when the backend supports them).

---

### Cable availability looks wrong

Availability is computed as:

```
available = total − (cables checked on cameras with status out/overdue) − (open standalone cable loans)
```

Check that returned cameras/loans are actually marked returned. A camera only deducts cable stock while its status is `out` or `overdue`; a standalone loan only counts while its `returned` date is empty. If loans exceed stock, the availability cell shows a red `초과` (over) warning instead of a negative number.

---

### "Connection failed" when testing the URL

Check the following:

- The URL starts with `https://script.google.com/macros/s/`
- The deployment was set to **Execute as: Me** and **Access: Anyone**
- You published a new version after your last code change (not just saved the script)
- Your Google account has authorized the script to run (check for a permission prompt)

---

### Changes not reflecting for other users

The app auto-syncs every 60 seconds. To force an immediate refresh, click the **↻ 새로고침** button in the live bar above the inventory table.

---

### App works but data resets on page refresh

This means Google Sheets is not connected. The yellow banner at the top of the page will indicate this. Follow the [Google Sheets Integration](#google-sheets-integration) steps to connect a Sheet. Until connected, data is stored only in the browser's `localStorage`.

---

## Technical Notes

- The frontend is a **single self-contained HTML file** with no external dependencies or build step
- Data is stored as plain strings in Google Sheets; JSON fields (`items`, `pdfs`) are serialized/deserialized automatically
- **Cable availability is computed, not stored:** it's derived live from camera `items` (for cameras that are out/overdue) plus open rows in the `cables` sheet, so bundling a cable with a camera automatically reduces its available count with no double entry
- **Cable stock totals** live in localStorage (`zivid_cableTotals`); the frontend already calls `saveCableTotal` and reads `cableTotals` from `getAll`, so a small backend addition makes them shared too
- A standalone cable loan is reused as a fungible quantity (`qty`), unlike cameras which are tracked as individual serialized units
- PDF files are stored as base64 `dataUrl` strings inside the `pdfs` JSON column — large PDFs will increase sheet cell size significantly
- The backend uses `getDataRange().getValues()` which reads the entire sheet on every request; for very large datasets (500+ cameras), consider adding pagination
- All API calls go through `fetch()` directly to the Apps Script web app URL; no authentication token is required because the deployment is set to "Anyone"
- Auto-refresh interval is 60 seconds (configurable in the `setInterval` call near the bottom of the script section)

---

## Version History

Versioning follows [Semantic Versioning](https://semver.org/) — `MAJOR.MINOR.PATCH`:

- **MAJOR** — breaking changes to the sheet structure or data format (a re-`initSheets()` or data migration is required)
- **MINOR** — new features, backward-compatible (existing data keeps working)
- **PATCH** — bug fixes and small tweaks, no schema change

> The repository has no GitHub Releases; these numbers are assigned from the commit history as a reference scheme. Dates reflect the relevant commits.
>
> ⚠️ **Don't confuse this with the Apps Script deployment "version."** When you redeploy `Code.gs` (Deploy → Manage deployments → New version), Google increments its own deployment version number. That is unrelated to the app versions below.

| Version | Date | Summary |
|---|---|---|
| **1.2.0** | 2026-06-17 | Cable inventory & standalone cable loans |
| **1.1.0** | 2026-03-31 | PDF → Google Drive saving, date-format hardening, waiting-list notes |
| **1.0.0** | 2026-03-17 | First documented release |

---

### 1.2.0 — 2026-06-17

**Cable inventory management.**

- Added the **🧵 케이블 재고 현황** card: per-type total stock, on-loan counts (camera-bundled vs. standalone), total out, and live availability with an `초과` (over) warning
- Added **standalone cable loans** — loan cables without a camera (type, quantity, borrower, out/due dates) with ✔️ return, ✏️ edit, 🗑️ delete
- **Editable total stock** per cable type, inline in the table (stored in `localStorage` as `zivid_cableTotals`)
- Cable availability is **auto-computed** from cameras that are out/overdue (via their `items`) plus open standalone loans — bundling a cable with a camera deducts stock with no double entry
- Camera modal checklist now shows a live `가용 N/총 M` badge next to tracked cables
- Renamed the inventory section to **카메라 / 케이블 재고 현황**
- Cleaned up accessory labels (`Zivid 2/2+`, `Zivid 3`, `Zivid One+`)

**Backend (`Code.gs`):**

- New `cables` sheet (8 columns: `id, sku, qty, borrower, out, due, returned, note`), created by `initSheets()` with plain-text date columns E/F/G
- New routes: `getCables` (GET), `saveCable` / `deleteCable` (POST); `getAll` now returns `cables`
- Frontend also reads/writes `cableTotals` (graceful no-op until a backend handler is added)

> **Upgrade note:** re-run `initSheets()` once to create the `cables` sheet, then publish a new deployment version.

---

### 1.1.0 — 2026-03-31

- PDF attachments now upload to **Google Drive** (`uploadPdfToDrive`) for permanent storage, with local in-memory caching as a fallback
- Hardened date handling so `YYYY-MM-DD` values survive Google Sheets' auto-conversion (`@` plain-text columns + `formatDate()` normalization)
- Added a **note** field to the waiting list (waiting sheet grew from 8 to 9 columns)
- Refactored camera normalization / data-loading logic

---

### 1.0.0 — 2026-03-17

First documented release.

- Camera inventory table with name badges, S/N, M/N, accessory checklist, location, and dates
- Status tracking (available / out / overdue / returned / unavailable) and one-click return
- Waiting list with availability badges and inline ETA editing
- Monthly loan calendar and per-model summary
- Per-camera loan history
- Google Sheets sync (60-second auto-refresh) with localStorage offline fallback
