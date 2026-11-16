# Allocation Tracker

Distributor allocation tracking for Cash Cards Unlimited, a trading-card business. The tracker reads distributor invoices, sales orders and emails, works out how much product each distributor allocated to the business week by week, and shows the results on a dashboard in your browser. It can also export an Excel workbook and build a "meeting pack" (Excel, dashboard HTML and PDF) for sharing.

Everything runs on your own computer. The app is a small local web server at `http://127.0.0.1:8765`, the data is a SQLite database, and no AI is used: documents are read with fixed rules for each distributor.

> **This file is generated.** `Allocation Tracker.py` is built from the `tracker/` package by `packaging/make_single_file.py`. That package is not in this repository. The module sources are stored as strings inside the file and loaded at startup. To change behaviour, edit the package where it is kept and regenerate this file. Edits made directly to this file are lost the next time it is generated.

## Features

- **Invoice PDF import.** Recognises each distributor from fixed text on the page and reads its layout. Rules exist for Panini America, The Topps Company, Magazine Exchange, ACD Distribution, GTS Distribution, Universal Distribution (including Alliance Game Distributors), PHD Games / Peachstate Hobby, Southern Hobby, Great American Distribution, Mad Al Distributors, License 2 Play Toys, L2 Global, Sweet Deal, Crossing Inc and L5 Sports Cards. Other sellers go through a general reader that only accepts a line when quantity x price = amount.
- **Checks before loading.** Every document is checked (for example, that the line totals add up to the subtotal). A document that fails goes to **Review** with the reasons. It is never partly loaded.
- **Duplicate detection.** Identical files, identical text, the same distributor and number, or the same distributor, date, total and lines are caught. Nothing is ever hard-deleted.
- **Gmail import.** Imports from a Google Takeout `.mbox` export, or fetches new mail straight from Gmail over read-only IMAP using an app password. Only labels you map to a distributor, or mail from known distributor sender domains, are read.
- **Product matching.** Matches by manufacturer code family (Pokemon, Magic, Bandai, Panini), then UPC, then the distributor's own SKU, then a keyword dictionary for company, game and category.
- **Allocation rules.** Weeks run Monday to Sunday. Lines are classed as Allocated or Extras/Restock. Open sales orders count as On order. Orders with no invoice after 120 days become "unfilled". Shipment emails are matched to invoices within 21 days.
- **Dashboard.** Includes Needs attention, Who is up / who is down, Allocation over time, Distributors over time, Asked for vs. allocated vs. shipped, Insights, By game, Set launches, Products, Timeline notes, Shipping and Paperwork coverage. The dashboard can be saved as a self-contained, offline `.html` file with a light/dark theme and sortable tables with CSV download.
- **Excel export.** Includes the sheets About, Weekly by game, Weekly by set, Weekly by product, Line items, Invoices and Coverage.
- **Meeting pack.** A dated folder containing the Excel file, the dashboard HTML, a PDF of the dashboard and a copy of the database.
- **Rebuild.** Re-reads everything from a Gmail export into a separate database and compares it with your current data. You then switch to the rebuilt data or discard it. Your review decisions and edits are carried over.
- **Backups.** Automatic daily database backups.

## Requirements

- **Python 3.12 or newer.** The startup check says 3.10, but the file uses f-string syntax that only Python 3.12+ can parse.
- **Python packages:** `pdfplumber==0.11.4` and `openpyxl==3.1.5`. The `setup` command installs them.
- **For the meeting-pack PDF only:** Microsoft Edge or Google Chrome. On a Mac, Chromium or Brave also work. If none is found, the pack is built without the PDF.
- **For fetching Gmail directly only:** a Gmail account with 2-Step Verification and an app password.

## How to run

There is no prebuilt app in this repository. Run the tracker from source.

**Windows**

```
py "Allocation Tracker.py" setup     # first time only: installs pdfplumber and openpyxl
py "Allocation Tracker.py"           # starts the app and opens it in your browser
```

**macOS**

```
python3 "Allocation Tracker.py" setup
python3 "Allocation Tracker.py"
```

The app opens at `http://127.0.0.1:8765/`. Keep the terminal window open while you use it. To stop it, close that window or choose **More > Stop the tracker**. If the tracker is already running, starting it again just opens the existing page.

### Pages

| Page | Purpose |
|---|---|
| Dashboard | Charts and totals |
| Review | Documents and products that need a decision or detail |
| Upload | Add invoice PDFs by hand (drag and drop) |
| Emails | Import a Takeout `.mbox`, map Gmail labels to distributors, or fetch new mail from Gmail |
| Products | Every product and its details |
| Invoices | Every invoice, and which ones have arrived |
| More > Actions & notes | Timeline notes |
| More > Meeting pack | Build the meeting pack |
| More > Manual entry | Enter a document by hand |
| More > Batches | Import history |
| More > Health | Data checks |

### Command line

Run `py "Allocation Tracker.py" <command>` on Windows, or `python3 ...` on a Mac. With no command, the tracker runs `serve`.

| Command | What it does |
|---|---|
| `serve [--port N] [--no-browser]` | Start the browser app (default port 8765) |
| `import <folders-or-PDFs...> --label pending\|processed\|history [--note TEXT] [--newest-first]` | Import invoice PDFs. Folders are searched recursively. |
| `recompute` | Re-run product matching, classification and allocation rules |
| `mbox-labels <file.mbox>` | List the Gmail labels in a Takeout export |
| `map-label <label> <distributor> [--pending]` | Assign a Gmail label to a distributor. Use `""` to skip a label. `--pending` marks it as holding invoices not yet received. |
| `import-mbox <file.mbox>` | Import emails under mapped labels |
| `rebuild [mbox] [--pdfs DIR...] [--no-archive]` | Build `tracker.rebuild.db` from the export and PDFs, then print a before/after comparison |
| `rebuild-switch` / `rebuild-discard` | Switch to the rebuilt data (the old data is kept as a backup), or throw it away |
| `reread-emails` | Re-read saved emails with the current reading rules |
| `check` | Integrity and consistency checks |
| `export` | Write `Distributor Allocation.xlsx` to the data folder |
| `pack` | Build a meeting pack |
| `--help` | List all commands |

## Inputs

- **Invoice / order PDFs.** These must be text PDFs. Scanned images contain no text and cannot be read. Import labels:
  - `pending`: invoices not yet received
  - `processed`: received
  - `history`: past documents
  
  Hidden `._` files and `__MACOSX` folders are ignored.
- **Gmail Takeout export.** The `.mbox` file from takeout.google.com > Mail. Upload it on the **Emails** page, then choose which distributor each label belongs to. Importing the same export twice is safe, because emails already seen (by Message-ID) are skipped. PDF attachments go through the invoice reader.
- **Gmail direct fetch.** On the **Emails** page, enter the Gmail address and a 16-letter app password, then press **Fetch new emails**.
  - The mailbox is opened read-only, so nothing is marked read, moved or deleted.
  - The first fetch looks back 365 days. Later fetches overlap the newest stored email by 7 days, and repeats are skipped.
  - If "remember" is ticked, the app password is stored encrypted for your user account only: with DPAPI in `data/gmail.json` on Windows, or as the Keychain item "Allocation Tracker Gmail" on a Mac. Your normal Google password is never used.

## Configuration

The tracker has no config file. Settings such as Gmail label mappings, distributors and timeline notes are stored in the database and changed in the app.

| Environment variable | Purpose |
|---|---|
| `TRACKER_DATA` | Use a different data folder instead of the default (for example, for testing). Example: `TRACKER_DATA=/path/to/test-data` |

Default data folder:

| How it runs | Data folder |
|---|---|
| From source | `data/` next to `Allocation Tracker.py` |
| Packaged Windows program (`Allocation Tracker.exe`) | `data/` next to the `.exe` |
| Packaged Mac app | `~/Library/Application Support/Allocation Tracker` |

## Outputs

All outputs are in the data folder:

| Path | Contents |
|---|---|
| `tracker.db` | The SQLite database, which is the master copy of all data |
| `archive/` | One copy of every PDF loaded, named by fingerprint |
| `backups/` | Database backups. `tracker-YYYYMMDD-HHMMSS.db` is taken daily at startup and with each meeting pack; the 30 newest are kept. `before-*.db` is taken before big changes; the 10 newest are kept. |
| `meeting_packs/YYYY-MM-DD meeting pack/` | `Distributor Allocation.xlsx`, `Distributor Allocation dashboard.html`, `Distributor Allocation summary.pdf` and `tracker-backup.db` |
| `Distributor Allocation.xlsx` | Written by `export` and by the export button in the app |
| `tracker.rebuild.db` | A rebuilt database waiting to be switched in or discarded |
| `logs/tracker.log` | Problems and slow pages (rotating log, 5 files of 1 MB) |
| `gmail.json` | Gmail fetch settings, plus the encrypted app password on Windows if remembered |
| `edge-profile/` | Separate browser profile used to print the PDF |

Excel files are output only. Edits made to them are never imported back.

## Building

The single file is generated from the `tracker/` package by `packaging/make_single_file.py`. Neither is in this repository.

The code refers to a Windows program (`Allocation Tracker.exe`), a Mac app and a `Start Tracker.bat` launcher. Their build scripts also live with the package, not here.

## Troubleshooting

- **`SyntaxError` at startup.** You are running Python older than 3.12. Install Python 3.12 or newer.
- **"Missing: pdfplumber, openpyxl".** Run the `setup` command once.
- **The browser does not open.** Go to `http://127.0.0.1:8765/` yourself. If port 8765 is taken by another program, use `serve --port 8800`.
- **A document went to Review.** It failed a check, for example totals that do not add up, an unknown distributor, or a possible duplicate. The reasons are listed on the Review page.
- **A scanned PDF reads as empty.** Scanned PDFs have no text, so enter the document under **Manual entry**.
- **The meeting pack has no PDF.** Install Edge or Chrome in its standard location.
- **Gmail fetch is rejected.** Use an app password, not your Google password. 2-Step Verification must be on. Fetching a long history can take a long time; importing a Takeout export is the alternative.
- **Windows or macOS blocks the file from running.** Allow it under the system's security/privacy settings ("Run anyway" / "Open anyway").
- **Checking data health.** Run `check`, or open **More > Health**.

## Code layout

`Allocation Tracker.py` contains:

- a short launcher: `setup`, the dependency check and `main()`
- `_SOURCES`, a dictionary of module name -> source text
- an import hook (`_Loader`) that loads those modules as if `tracker/` sat next to the file

The embedded modules:

| Module | Role |
|---|---|
| `tracker.cli` | Command-line commands |
| `tracker.server` | Local browser app (standard-library `http.server`) |
| `tracker.config` | Data folder locations |
| `tracker.db` | SQLite schema and connection |
| `tracker.extract` | PDF text extraction (pdfplumber) and fingerprints |
| `tracker.detect` | Distributor recognition from page text |
| `tracker.parsers.*` | One reader per distributor, plus `generic`, `email_rules` and `small_sellers` |
| `tracker.checks` | Pass/fail checks on parsed documents |
| `tracker.ingest` | Import pipeline and duplicate handling |
| `tracker.emails` / `tracker.gmail` | Takeout `.mbox` import and read-only IMAP fetch |
| `tracker.products` / `tracker.similar` | Product matching, classification and estimated prices |
| `tracker.allocation` / `tracker.pipeline` | Allocation rules and the recompute steps |
| `tracker.report` | Shared read-only queries |
| `tracker.dashboard` / `tracker.ui` | Dashboard HTML and shared styling |
| `tracker.export_xlsx` / `tracker.pack` | Excel export, meeting pack and backups |
| `tracker.rebuild` | Rebuild from a Gmail export and compare |
