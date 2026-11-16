# Allocation Tracker

Distributor allocation tracking for Cash Cards Unlimited, a trading-card business. It is for the people who order from distributors and need to see how much product each one allocated. The tracker reads distributor invoices, sales orders and emails, works out each distributor's allocation week by week, and shows the results on a dashboard in your browser. It can also export an Excel workbook and build a "meeting pack" (Excel, dashboard HTML and PDF) for sharing.

Everything runs on your own computer. The app is a small local web server at `http://127.0.0.1:8765`, the data is a SQLite database, and no AI is used: documents are read with fixed rules for each distributor.

## Quick start

1. Install Python 3.12 or newer.
2. Install the two packages (first time only):
   - Windows: `py "Allocation Tracker.py" setup`
   - macOS / Linux: `python3 "Allocation Tracker.py" setup`
3. Start the app: `py "Allocation Tracker.py"` (or `python3 ...`). It opens `http://127.0.0.1:8765/` in your browser.
4. Add data on the **Upload** page (invoice PDFs) or the **Emails** page (Gmail export or direct fetch), then open **Dashboard**.

Keep the terminal window open while you use the app. To stop it, close that window or choose **More > Stop the tracker**.

> **This file is generated.** `Allocation Tracker.py` is built from the `tracker/` package by `packaging/make_single_file.py`. Neither is in this repository. The module sources are stored as strings inside the file and loaded at startup, so edits made directly to this file are lost the next time it is generated. To change behaviour, edit the package where it is kept and regenerate this file.

## Features

- **Invoice PDF import.** Recognises each distributor from fixed text on the first page and reads its layout. Rules exist for Panini America, The Topps Company, Magazine Exchange, ACD Distribution, GTS Distribution, Universal Distribution (including Alliance Game Distributors), PHD Games / Peachstate Hobby, Southern Hobby, Great American Distribution, Mad Al Distributors, License 2 Play Toys, L2 Global, Sweet Deal, Crossing Inc and L5 Sports Cards. Other sellers go through a general reader that only accepts a line when quantity x price = amount.
- **Checks before loading.** Every document is checked (for example, that the line totals add up to the subtotal). A document that fails goes to **Review** with the reasons. It is never partly loaded.
- **Duplicate detection.** Identical files and identical text are skipped. The same distributor and number, or the same distributor, date, total and lines, are caught and sent to Review when the content differs. Nothing is ever hard-deleted.
- **Gmail import.** Imports from a Google Takeout `.mbox` export, or fetches mail straight from Gmail over read-only IMAP using an app password. Only mail under labels you map to a distributor, or from a sender domain already known to belong to a distributor, is read.
- **Product matching.** Each line is matched by manufacturer code family (Pokemon, Magic, Bandai, Panini), then UPC, then the distributor's own SKU. A line that matches nothing becomes a new product, and a keyword dictionary fills in its company, game and category.
- **Allocation rules.** Weeks run Monday to Sunday. Lines are classed as Allocated or Extras/Restock. Open sales orders count as On order. Orders with no invoice after 120 days become "unfilled". Shipment emails are matched to invoices within 21 days.
- **Dashboard.** Sections: Needs attention, Who is up, who is down, Allocation over time, Distributors over time, Asked for vs. allocated vs. shipped, Insights, By game, What we did, Set launches, Products, Timeline notes, Shipping and Paperwork coverage. The dashboard can be saved as a self-contained, offline `.html` file with a light/dark theme and tables with CSV download.
- **Excel export.** Sheets: About, Weekly by game, Weekly by set, Weekly by product, Line items, Invoices, Coverage, Timeline notes, Missing invoices and Review.
- **Meeting pack.** A dated folder containing the Excel file, the dashboard HTML, a PDF of the dashboard and a copy of the database.
- **Rebuild.** Re-reads everything from a Gmail export (and the PDFs already loaded) into a separate database and compares it with your current data. You then switch to the rebuilt data or discard it. Review decisions, receipt dates, manual entries, product edits and re-classifications are carried over.
- **Backups.** An automatic database backup at most once a day, when the app starts.

## Requirements

- **Python 3.12 or newer.** The startup check says 3.10, but the file uses f-string syntax that only Python 3.12+ can parse.
- **Python packages:** `pdfplumber==0.11.4` and `openpyxl==3.1.5`. The `setup` command installs them with pip. These are the only third-party packages used.
- **tkinter (optional):** used only by the **Browse** button on the Emails page. It ships with the python.org installers; some Linux and Homebrew Pythons need it installed separately. You can upload the `.mbox` or type its path instead.
- **For the meeting-pack PDF only:** Microsoft Edge or Google Chrome in its standard install location. On a Mac, Chromium or Brave also work. If none is found, the pack is built without the PDF.
- **For fetching Gmail directly only:** a Gmail account with 2-Step Verification, an app password, and IMAP enabled in Gmail settings.

## How to run

There is no prebuilt app in this repository. Run the tracker from source.

```
py "Allocation Tracker.py" setup         # Windows, first time only: installs pdfplumber and openpyxl
py "Allocation Tracker.py"               # Windows: starts the app and opens it in your browser

python3 "Allocation Tracker.py" setup    # macOS / Linux
python3 "Allocation Tracker.py"
```

The app opens at `http://127.0.0.1:8765/`. If something is already listening on that port (normally an earlier copy of the tracker), the command prints "The tracker is already running" and opens that page instead of starting a second copy.

### Pages

| Page | Purpose |
|---|---|
| Dashboard | Charts and totals |
| Review | Documents and products that need a decision or detail |
| Upload | Add invoice PDFs by hand (drag and drop), choosing which Gmail label they came from |
| Emails | Import a Takeout `.mbox`, map Gmail labels to distributors, or fetch mail from Gmail |
| Products | Every product and its details |
| Invoices | Every invoice, and which ones have arrived |
| More > Actions & notes | Timeline notes |
| More > Meeting pack | Build the meeting pack, or export Excel only |
| More > Manual entry | Enter a document by hand |
| More > Batches | Import history, and undo an upload |
| More > Health | Data checks |
| More > Report a problem | Write a note to `Tester notes.txt`. Nothing is sent anywhere. |
| More > Stop the tracker | Stop the app. Your data is kept. |

### Command line

Run `py "Allocation Tracker.py" <command>` on Windows, or `python3 "Allocation Tracker.py" <command>` on macOS or Linux. With no command, the tracker runs `serve`. Every command except `setup` needs the packages installed first.

| Command | What it does |
|---|---|
| `setup` | Install `pdfplumber` and `openpyxl` with pip |
| `serve [--port N] [--no-browser]` | Start the browser app (default port 8765) |
| `import <folders-or-PDFs...> --label pending\|processed\|history [--note TEXT] [--newest-first]` | Import invoice PDFs. Folders are searched recursively. `--newest-first` loads the most recently modified files first. |
| `recompute` | Re-run product matching, classification and allocation rules |
| `mbox-labels <file.mbox>` | List the Gmail labels in a Takeout export, with counts |
| `map-label <label> <distributor> [--pending]` | Assign a Gmail label to a distributor. Use `""` as the distributor to skip a label. `--pending` marks it as holding invoices not yet received. |
| `import-mbox <file.mbox>` | Import emails under mapped labels |
| `rebuild [mbox] [--pdfs DIR...] [--no-archive]` | Build `tracker.rebuild.db` from the export, the archived PDFs (unless `--no-archive`) and any extra PDF folders, then print a before/after comparison |
| `rebuild-switch` / `rebuild-discard` | Switch to the rebuilt data (the old data is saved to `backups/` first), or throw it away |
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
- **Gmail Takeout export.** The `.mbox` file from takeout.google.com > Mail. On the **Emails** page, upload it, pick it from the list of exports found in Downloads, Desktop or Documents, or give its path; then choose which distributor each label belongs to. Uploaded files are copied to `imports/` in the data folder. Importing the same export twice is safe, because emails already seen (by Message-ID) are skipped. PDF attachments go through the invoice reader.
- **Gmail direct fetch.** On the **Emails** page, enter the Gmail address and a 16-letter app password, then press **Fetch new emails** (or **Fetch all emails** for the whole mailbox history).
  - The mailbox is opened read-only, so nothing is marked read, moved or deleted.
  - The first fetch looks back 365 days. Later fetches start 7 days before the newest stored email, and repeats are skipped.
  - If "remember on this computer" is ticked, the app password is stored encrypted for your user account only: with DPAPI in `gmail.json` on Windows, or as the Keychain item "Allocation Tracker Gmail" on a Mac. Remembering is not available on Linux; untick it there. Your normal Google password is never used.

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

These paths are in the data folder:

| Path | Contents |
|---|---|
| `tracker.db` | The SQLite database, which is the master copy of all data |
| `archive/` | One copy of every PDF loaded, named by fingerprint |
| `imports/` | `.mbox` exports uploaded through the Emails page |
| `backups/` | Database backups. `tracker-YYYYMMDD-HHMMSS.db` is taken at startup (at most once a day) and with each meeting pack; the 30 newest are kept. `before-rebuild-*.db` is taken when you switch to rebuilt data; the 10 newest are kept. |
| `meeting_packs/YYYY-MM-DD meeting pack/` | `Distributor Allocation.xlsx`, `Distributor Allocation dashboard.html`, `Distributor Allocation summary.pdf` and `tracker-backup.db` |
| `Distributor Allocation.xlsx` | Written by `export` and by **Export Excel** on the Meeting pack page |
| `tracker.rebuild.db`, `rebuild_report.json` | A rebuilt database and its comparison, waiting to be switched in or discarded |
| `logs/tracker.log` | Problems and slow pages (rotating 1 MB log, with 5 older copies kept) |
| `gmail.json` | Gmail address, plus the encrypted app password on Windows if remembered |
| `edge-profile/` | Separate browser profile used to print the PDF |

**More > Report a problem** writes `Tester notes.txt` to the folder that contains the data folder (from source, the folder with `Allocation Tracker.py`; for the Mac app, the data folder itself).

Excel files are output only. Edits made to them are never imported back.

## Troubleshooting

- **`SyntaxError` at startup.** You are running Python older than 3.12. Install Python 3.12 or newer.
- **"Missing: pdfplumber, openpyxl".** Run the `setup` command once.
- **The browser does not open.** Go to `http://127.0.0.1:8765/` yourself.
- **"The tracker is already running" but the page is not the tracker.** Another program is using port 8765. Start with `serve --port 8800` and open `http://127.0.0.1:8800/`.
- **A document went to Review.** It failed a check, for example totals that do not add up, an unknown distributor, or a possible duplicate. The reasons are listed on the Review page.
- **A scanned PDF reads as empty.** Scanned PDFs have no text, so enter the document under **More > Manual entry**.
- **PDFs in a folder are skipped on macOS or Linux.** Folder search matches the lowercase extension `.pdf` only. Rename `.PDF` files, or pass them as file arguments.
- **The meeting pack has no PDF.** Install Edge or Chrome in its standard location.
- **Gmail fetch is rejected.** Use an app password, not your Google password. 2-Step Verification and IMAP must both be on. Fetching a long history can take a long time; importing a Takeout export is the alternative.
- **"Could not save the sign-in".** Untick "remember on this computer" (always needed on Linux).
- **Windows or macOS blocks the file from running.** Allow it under the system's security/privacy settings ("Run anyway" / "Open anyway").
- **Checking data health.** Run `check`, or open **More > Health**.

## Building

The single file is generated from the `tracker/` package by `packaging/make_single_file.py`. Neither is in this repository.

The code refers to a Windows program (`Allocation Tracker.exe`), a Mac app and a `Start Tracker.bat` launcher. Their build scripts also live with the package, not here.

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
