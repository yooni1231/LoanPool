# 📷Camera Loan Management System

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

---

## Overview

This system helps teams track the location, status, and loan history of # cameras. It supports multiple simultaneous users via Google Sheets sync, works offline with browser localStorage, and requires no server or build tools — just two files.

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
| **Waiting List** | Queue of companies waiting for a specific model, with ETA tracking |
| **Loan Calendar** | Monthly calendar view showing all active and past loan periods |
| **Model Summary** | Per-model breakdown of total / available / on loan / unavailable |
| **Loan History** | Per-camera history drawer showing every checkout and return event |
| **PDF Attachments** | Upload and view PDF files (e.g. shipping docs) attached to each camera |
| **Name Badges** | Color-coded badges for quick visual identification |
| **Accessory Checklist** | 18 selectable accessory items per camera (cables, mounts, boards, etc.) |
| **Google Sheets Sync** | Auto-syncs to Google Sheets every 60 seconds; changes persist across devices |
| **Offline Mode** | Falls back to browser localStorage when no Sheet is connected |
| **One-click Return** | ✔️ button marks a camera as returned, resets location, and logs the event |

---

## File Structure

```
#-loan-manager/
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
2. Name it anything (e.g. `# Camera DB`)
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

   > This creates two sheets inside your spreadsheet: `cameras` and `waiting`, with the correct column headers and text formatting on date columns.

---

### Step 3 — Deploy as a Web App

1. In the Apps Script editor, click **Deploy → New deployment**

2. Click the gear icon ⚙ next to "Select type" and choose **Web app**

3. Fill in the deployment settings:

   | Setting | Value |
   |---|---|
   | Description | `# Camera API` (or anything) |
   | Execute as | **Me** |
   | Who has access | **Anyone** |

4. Click **Deploy**

5. If prompted, click **Authorize access** and follow the Google sign-in flow

6. After deployment, you will see a **Web app URL** that looks like:
   ```
   https://script.google.com/macros/s/AKfycb.../exec
   ```

7. **Copy this URL** — you will need it in the next step

> ⚠️ **Important:** Every time you edit `Code.gs` after the initial deployment, you must create a **new deployment** (not update the existing one) for the changes to take effect. Click **Deploy → New deployment** each time.

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

The Apps Script backend manages two sheets automatically.

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

### `waiting` sheet (8 columns)

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

---

## Using the App

### Adding a Camera

1. Click **＋ 카메라 추가** in the inventory section
2. Fill in the required fields: **S/N**, **M/N**, **Current Location**
3. Optionally set a name and badge color for quick identification
4. Check off included accessories from the list
5. Set the status and dates if the camera is already on loan
6. Attach any PDF documents (shipping labels, agreements, etc.)
7. Click **저장**

### Checking a Camera Out

- Open the edit modal (✏️) for the camera
- Set **Status** to `📤 대여 중`
- Fill in **Out date** and **Current Location** (the borrower's company name)
- Click **저장** — a loan history entry is created automatically

### Marking a Camera as Returned

- Click the **✔️** button on any active loan row
- The system automatically sets status to `반납 완료`, sets today as the return date, and resets the location to `# Oslo`
- A return event is logged in the camera's loan history

### Waiting List

- Click **＋ 추가** to add a company to the waiting queue
- Each waiting entry shows whether a matching model is currently available (green badge)
- The **ETA** (estimated ship date) can be edited inline directly in the table by hovering the date and clicking **✏ 수정** — no modal needed
- Click ✏️ to edit all fields of a waiting entry

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
| `psu_z2` | PSU (External AC-DC) # 2/2+ |
| `psu_z3` | PSU (External AC-DC) # 3 |
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
| `data_cable_one` | Data cable - # One+ |
| `check_note` | ⚠ Need to check Note or Files |

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

**Fix:** Run `initSheets()` from the Apps Script editor. This sets columns H, I, J (`lastUpdate`, `out`, `inn`) and column F of the waiting sheet (`eta`) to `@STRING@` (plain text) format, which prevents automatic date conversion.

The `formatDate()` helper in `Code.gs` also handles reading back any values that were already converted, normalizing them back to `YYYY-MM-DD` strings.

---

### S/N and Color fields are swapped

**Cause:** The sheet's column order does not match the expected header order.

**Fix:** Run `initSheets()` once from the Apps Script editor. It overwrites row 1 with the correct headers without touching your data rows. The backend reads columns by header name (not by position), so as long as the headers are correct, data is always mapped correctly.

---

### "Connection failed" when testing the URL

Check the following:

- The URL starts with `https://script.google.com/macros/s/`
- The deployment was set to **Execute as: Me** and **Access: Anyone**
- You created a **New deployment** after your last code change (not just saved the script)
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
- PDF files are stored as base64 `dataUrl` strings inside the `pdfs` JSON column — large PDFs will increase sheet cell size significantly
- The backend uses `getDataRange().getValues()` which reads the entire sheet on every request; for very large datasets (500+ cameras), consider adding pagination
- All API calls go through `fetch()` directly to the Apps Script web app URL; no authentication token is required because the deployment is set to "Anyone"
- Auto-refresh interval is 60 seconds (configurable in the `setInterval` call near the bottom of the script section)
