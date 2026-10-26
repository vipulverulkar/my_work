# Personal Expense Tracker ₹

Track income, expenses, budgets, accounts, goals, debts and reports — a
Flask + PostgreSQL / SQLite web and desktop app with dark mode, interactive
charts, automated rules, bill calendar, bank file ingestion, and multi-user roles.

![Dashboard](screenshots/dashboard.png)

## Features

**Money & Wealth Management**
- **Dashboard** — total balance, monthly summary (Monthly Total Expense, Month saved, Savings rate), Yearly Expense (configured Financial Year, defaulting to Indian FY: 1 Apr – 31 Mar, or Calendar Year), **Safe-to-Spend / Day** dynamic spending pacer, quick transaction entry, recent transactions ledger with **`⚡ Spike`** anomaly indicators, active budget warnings, and live notification badges
- **Accounts & Net Worth** — track multiple bank accounts, savings, credit cards, cash wallets, investments, and loans with live balance calculation, total liquid balance, total net worth, and atomic inter-account transfers
- **Financial Goals & Sinking Funds** — goal tracking with visual progress gauges, deposit contributions, target completion dates, and automatic status updates (active / achieved / archived)
- **Debt Payoff Planner** — track debts, APR interest rates, minimum payments, and calculate payoff timelines and interest savings comparing **Avalanche** (highest interest first) vs. **Snowball** (lowest balance first) strategies
- **Shared Group Expenses** — split any transaction across friends or housemates, monitor outstanding net balances per person, and settle splits individually or in bulk
- **Bill Calendar & Cashflow Forecast** — interactive monthly calendar view visualizing scheduled bills, debt due dates, subscription renewals, and projected end-of-month cashflow
- **Automation Rules Engine** — automated transaction categorizer & tagger applying pattern/regex rules to merchant and notes upon manual entry or bank statement ingestion
- **Savings buckets** — flag any expense category as 🏦 savings (a `Savings` bucket is seeded); parked money is excluded from spending totals and the savings rate, and shown as Saved instead across dashboard, reports, API and CSV totals
- **Transactions** — filter (type / category / account / tags / user / month / full-text note search), sort, paginate, edit, duplicate; receipt attachments (image/PDF, max 5 MB); custom tags (e.g. `trip:japan`, `work`); guided bulk actions (delete to Trash, or re-categorize); inter-category transfer (linked debit/credit pair); **`⚡ Spike`** anomaly indicators for transactions $>2.5\times$ category average; **`🛡 Integrity Check`** for transaction consistency validation and non-destructive anomaly repair
- **Recurring** — daily / weekly / monthly / yearly schedules with a start date and optional end date (rent, salary, subscriptions); due items are generated automatically on dashboard load
- **Split transactions** — one expense spread over up to 3 extra categories, linked as a group so reports stay accurate
- **Budgets & Rollover** — monthly limits per category with progress bars, over-budget warnings, inline & dedicated budget editing (`/budgets/edit/<id>`), and optional month-to-month budget rollover for unused balances
- **Proactive Alerts & Notifications** — in-app notification center with badge indicator for budget threshold warnings (85% and 100%), upcoming debt bills, and system notices
- **Reports** — dedicated **Overview** view (Last 6 months trend chart & table, Top spending donut & category shares with **Month-over-Month (MoM) Shift Badges**, **50 / 30 / 20 Financial Health Breakdown**, Active monthly budgets); daily and monthly charts, budget-vs-actual variance, 3-month cash-flow forecast, tags breakdown, printable statement (`/reports/print`), PDF + Excel (`.xlsx`) export
- **More reports** — yearly overview with year-over-year growth and a 12-month savings-rate trend; **yearly trends** and **monthly trends** tabs; top transactions with average-ticket and share; category × month deep-dive matrix; insights (50/30/20 compliance health, weekday spending pattern, spend per member, savings-bucket mix vs spending, recurring commitments)
- **1-Page Monthly A4 Statement** — clean printable statement (`GET /reports/print`) optimized with `@media print` for physical paper filing or Save as PDF, complete with executive KPI strip, 50/30/20 ratio, category breakdown, budget variance, and account balance snapshot
- **Trash** — soft deletion of transactions with one-click restore or permanent purge
- **Nav extras** — dark/light theme toggle (persisted, respects OS preference) and fullscreen toggle (`⛶`); installable PWA manifest (`/manifest.webmanifest`, standalone display)

**Data & Ingestion**
- **Bank statement import** (admin) — guided upload supporting **OFX**, **QFX**, **QIF**, and **CSV** formats with column mapping, 8-row preview, automated rule execution, and saved column-mapping templates
- **Import & export** — CSV export (respects filters) and import (additive — never deletes); full-table CSV downloads and desktop native file download bridge
- **Backup & restore** — full-schema SQL-dump backup of the whole database; **Desktop Local / Cloud Folder Auto-Backup** (`CUSTOM_BACKUP_DIR`) with 1-click backup to Google Drive / OneDrive / Dropbox / USB; restore from `.sql`, legacy `.json`, or CSV files (validated first — invalid files change nothing); database-to-database migration (automatic at startup or manual); optional automatic startup backups with rotation; **`🛡 Integrity Check`** validation on the data management page

**Accounts & Security**
- **Roles** — admin (full access) and viewer (read-only); every write route is server-enforced
- **Two-factor auth** — TOTP via any authenticator app, enforced as a second login step
- **User management** — add accounts, reset passwords, change roles (passwords: min 8 chars, letters + digits)
- **Font size** — per-user zoom of the whole UI (70–160%) from Profile, with live preview; stored on the account and applied to every page (site default 90%)
- **Audit log** — deletes, password resets, imports and role changes are recorded; retention purging
- **Site settings** — admins edit a safe, non-secret subset of the `app/.env` config (financial year start month defaulting to Indian FY: 1 Apr – 31 Mar, currency symbol, display formats, session/account policy, pagination, import limits, rate limit, backups) from **Master Data → Settings**; changes apply on restart
- **Sessions** — idle auto-logout after 5 min (per-user override, `SESSION_TIMEOUT_MINUTES` for the default) with a live nav countdown that resets on activity, warns at 1 min and signs out automatically; server-side sessions (CacheLib filesystem backend), login rate limiting, CSRF protection on all forms
- **Ops** — health check (`/healthz`), Prometheus metrics (`/metrics`), JSON API (`/api/v1/*`, legacy `/api/*`), optional Sentry

## Documentation

- **[User Guide](USER_GUIDE.md)** — how to use every screen (accounts, rules, calendar, goals, debts, shared expenses, notifications, transactions, budgets, reports, imports, backup, profile, users, settings).
- **[Architecture](ARCHITECTURE.md)** — app factory, request lifecycle, modules, data model, security and operations.

## Quickstart (local, non-Docker)

Run this straight from the repo against your own PostgreSQL server — no
Docker required (see [Docker](#docker) for the self-contained alternative,
which starts Postgres itself and needs no `DATABASE_URL`).

```bash
python3 -m venv .venv && source .venv/bin/activate   # optional but recommended
pip install -r requirements.txt   # runtime deps: pandas + openpyxl (Excel), cachelib (sessions), reportlab (PDF), pyotp (2FA)
                                  # test tools (pytest/pyflakes) are separate — see requirements-dev.txt
export DATABASE_URL=postgresql://user:password@localhost:5432/exptracker
python3 seed_demo.py  # optional demo data (transactions, budgets, accounts, goals, debts)
python3 run.py        # open http://127.0.0.1:5000  (PORT overrides 5000)
```

`DATABASE_URL` is **required** for this local path (Postgres-only build); the
schema is created and migrated automatically on first start. Use `python` if
your system does not have `python3`. Instead of exporting `DATABASE_URL`, you
can put it in `app/.env` (`cp app/.env.example app/.env`) — an optional config
file the app loads at startup, where real environment variables always win.

Sign in with the default credentials **admin / admin** (seeded on first boot —
change it before deploying). Configuration is read from **environment
variables**; the app also loads an optional `app/.env` file at startup (real
environment variables win). Admins can edit a safe, non-secret subset of these
settings from **Master Data → Settings**; changes apply after a restart. See
[Config](#config) and `app/.env.example` for the full list.

```bash
# change the admin password — passwords need 8+ chars, letters + digits:
python3 -c "
from werkzeug.security import generate_password_hash; import psycopg, os
db = psycopg.connect(os.environ['DATABASE_URL'])
db.execute('UPDATE users SET password_hash=%s WHERE username=%s',
           (generate_password_hash('new-passw0rd'), 'admin')); db.commit()"
```

Amounts are stored as **integer cents** in the database — no floating-point
rounding errors — and converted for display (Indian-style number formatting,
configurable symbol).

## Desktop app (standalone executable)

The app can be packaged as a standalone desktop executable that runs locally
without Docker or an external PostgreSQL server. It uses an embedded SQLite
database by default and opens in a native window (pywebview).

**Build (on the target OS — PyInstaller cannot cross-compile):**

```bash
# Linux
./build_linux.sh
# -> dist/Personal-Expense-Tracker_v6.1.4_linux

# Windows (PowerShell)
./build_windows.ps1
# -> dist\Personal-Expense-Tracker_v6.1.4_windows.exe
```

Desktop build includes all required Python modules and dependencies (Flask, CSRF, rate limiter, sessions, pandas, openpyxl, pyotp, reportlab, psycopg, dotenv).
Linux native GUI: uses system `gir1.2-webkit2-4.1` (WebKitGTK). If WebKitGTK or display is not installed on the system, the executable automatically falls back to opening the default web browser.
Windows native GUI: uses Edge WebView2 Runtime (preinstalled on Windows 10/11).

**Run:** launch `dist/Personal-Expense-Tracker_v6.1.4_linux` (Linux) or `dist\Personal-Expense-Tracker_v6.1.4_windows.exe` (Windows).

### Standalone Storage & SQL File Locations (Linux & Windows)

The standalone desktop executable automatically manages its live database, SQL backups, sessions, and configuration in standard user directories:

| Component | Linux Location | Windows Location | Description |
|---|---|---|---|
| **Live SQLite Database** | `~/.local/share/exptracker/exptracker.db` | `%LOCALAPPDATA%\exptracker\exptracker.db`<br>*(typically `C:\Users\<User>\AppData\Local\exptracker\exptracker.db`)* | The active embedded database containing all transactions, accounts, categories, and configurations. |
| **Automated Backups (`.sql`)** | `~/.local/share/exptracker/backups/` | `%LOCALAPPDATA%\exptracker\backups\` | Automated startup SQL dumps (`backup_*.sql`) generated when auto-backup is active. |
| **Settings Configuration** | `~/.local/share/exptracker/.env` | `%LOCALAPPDATA%\exptracker\.env` | Persistent environment overrides saved from the UI settings screen. |
| **Session Cache** | `~/.local/share/exptracker/flask_sessions/` | `%LOCALAPPDATA%\exptracker\flask_sessions\` | Server-side user session cache. |
| **Exported SQL Backups** | Native Save As dialog / `~/Downloads/` | Native Save As dialog / `C:\Users\<User>\Downloads\` | User-prompted exports downloaded from **Data ▾ → Backup & restore**. |
| **Reference SQL Schema** | `expenses.sql` (repository root) | `expenses.sql` (repository root) | Full reference PostgreSQL / SQLite DDL schema file. |

> **Quick Access:**
> - **Windows:** Press <kbd>Win</kbd> + <kbd>R</kbd>, enter `%LOCALAPPDATA%\exptracker`, and press **Enter** to open directly in File Explorer.
> - **Linux:** Run `cd ~/.local/share/exptracker` in your terminal or file manager.

**Remote PostgreSQL:** open **Master Data → Settings → Database**, enter/test
a `postgresql://` URL and save; restart the app to apply. Settings persist in
the data dir `.env`.

## Artifacts & downloads

- **GitHub Releases:** [Releases Page](https://github.com/vipulverulkar/Exp_tracker_V6.1_with_docker_postgres_and_sqllite/releases) — download prebuilt standalone executables and docker archives for latest releases.
- **Desktop builds (Linux / Windows):** [CI/CD workflow artifacts](https://github.com/vipulverulkar/Exp_tracker_V6.1_with_docker_postgres_and_sqllite/actions/workflows/ci_cd.yml) — download `Personal-Expense-Tracker_v6.1.4_linux` or `Personal-Expense-Tracker_v6.1.4_windows`.
- **Docker image archive:** `Personal-Expense-Tracker_v6.1.4_docker.tar.gz` from workflow artifacts or GitHub Releases.
- **Docker Hub image:** [`vipulverulkar/personal-expense-tracker-v6.1`](https://hub.docker.com/r/vipulverulkar/personal-expense-tracker-v6.1) on Docker Hub.
- **Source code:** [GitHub repository](https://github.com/vipulverulkar/Exp_tracker_V6.1_with_docker_postgres_and_sqllite).

## Screenshots

Captured with the automated headless screenshot suite (`python3 scripts/generate_screenshots.py`) covering all financial planning, tracking, wealth management, and operational views:

| Accounts & Net Worth | Rules Engine | Bill Calendar |
|---|---|---|
| ![Accounts](screenshots/accounts.png) | ![Rules](screenshots/rules.png) | ![Calendar](screenshots/calendar.png) |

| Financial Goals | Debt Payoff Planner | Shared Expenses |
|---|---|---|
| ![Goals](screenshots/goals.png) | ![Debts](screenshots/debts.png) | ![Shared](screenshots/shared.png) |

| Dashboard | Transactions | Reports |
|---|---|---|
| ![Dashboard](screenshots/dashboard.png) | ![Transactions](screenshots/transactions.png) | ![Reports](screenshots/reports.png) |

| Budgets | Notifications & Alerts | Data & Backup |
|---|---|---|
| ![Budgets](screenshots/budgets.png) | ![Notifications](screenshots/notifications.png) | ![Data](screenshots/data.png) |

| Users | User profile | Login |
|---|---|---|
| ![Users](screenshots/users.png) | ![User profile](screenshots/user-profile.png) | ![Login](screenshots/login.png) |

| Categories | Recurring | Trash |
|---|---|---|
| ![Categories](screenshots/categories.png) | ![Recurring](screenshots/recurring.png) | ![Trash](screenshots/trash.png) |

| Profile | Audit | Settings |
|---|---|---|
| ![Profile](screenshots/profile.png) | ![Audit](screenshots/audit.png) | ![Settings](screenshots/settings.png) |

| Yearly trends | Monthly trends |
|---|---|
| ![Yearly trends](screenshots/yearly-trends.png) | ![Monthly trends](screenshots/monthly-trends.png) |

| Inter-Category Transfer | Database Migration |
|---|---|
| ![Transfer](screenshots/transfer.png) | ![Migrate](screenshots/migrate.png) |

## Docker

Prebuilt image: `vipulverulkar/personal-expense-tracker-v6.1:latest` (or tag `:6.1`)
```bash
docker pull vipulverulkar/personal-expense-tracker-v6.1:latest
```

Or build locally — one container runs both the app and PostgreSQL (no compose setup needed):

```bash
# --provenance/--sbom off so the embedded container-scan VEX is honored (see
# "Container scanning" below); drop them if you don't need that.
docker build --provenance=false --sbom=false -t personal-expense-tracker-v6.1 .
docker run -d -p 5000:5000 \
  -v exptracker-pg:/var/lib/postgresql/data \
  -v exptracker-data:/data \
  --name personal-expense-tracker-v6.1 personal-expense-tracker-v6.1
# open http://127.0.0.1:5000
```

No config file is needed: the container starts Postgres, creates the database,
generates and persists a `SECRET_KEY`, and uses safe defaults for everything
else. To override settings, the config file is **optional** — add `--env-file app/.env`:

```bash
# only if you have a config file (touch app/.env, or cp app/.env.example app/.env)
docker run -d -p 5000:5000 \
  -v exptracker-pg:/var/lib/postgresql/data \
  -v exptracker-data:/data \
  --env-file app/.env --name personal-expense-tracker-v6.1 personal-expense-tracker-v6.1
```

Note: Docker's `--env-file` flag itself requires the file to exist, so only use
it when you actually have one — otherwise run the command above without it.
Alternatively the app reads its own `app/.env` at startup; mount it with
`-v "$PWD/app/.env":/app/app/.env` (or set `ENV_FILE=/data/.env`) to persist
UI-managed settings across container re-creates.

Postgres data persists in the `exptracker-pg` volume and app files
(sessions, backups) in `exptracker-data` — removing and
re-creating the container with the same volumes keeps everything. Only
deleting the volumes (`docker volume rm exptracker-pg exptracker-data`)
deletes data. The image is a multi-stage build; Postgres starts inside the
container and the app runs as a non-root user. The build is size-tuned
(~640 MB): test-only deps (`pytest`/`pyflakes`) are excluded, dependency
wheels are install-only (never stored in a layer), and the PostgreSQL JIT
libraries (`libLLVM`/`libz3`, ~155 MB) that this app never uses are dropped.

`SECRET_KEY` may be omitted: on first launch the entrypoint generates a
random one and stores it at `/data/.secret_key` (mode `0600`), so it
survives restarts via the `exptracker-data` volume and sessions stay valid.
Set `SECRET_KEY` explicitly to take full control (explicit env always wins).

```bash
# backup / restore the volumes (bash):
docker run --rm -v exptracker-pg:/var/lib/postgresql/data -v exptracker-data:/data -v "$PWD":/backup busybox tar czf /backup/personal-expense-tracker-backup.tgz /var/lib/postgresql/data /data
docker run --rm -v exptracker-pg:/var/lib/postgresql/data -v exptracker-data:/data -v "$PWD":/backup busybox tar xzf /backup/personal-expense-tracker-backup.tgz -C /
# raw database dump from the running container:
docker exec personal-expense-tracker-v6.1 pg_dump -U exptracker exptracker > personal-expense-tracker-dump.sql
```

```powershell
# same volume backup on Windows PowerShell (use ${PWD}, not $PWD):
docker run --rm -v exptracker-pg:/var/lib/postgresql/data -v exptracker-data:/data -v "${PWD}:/backup" busybox tar czf /backup/personal-expense-tracker-backup.tgz /var/lib/postgresql/data /data
```

### Container scanning

Docker Scout (used by Docker Hub) reports **3 HIGH** CVEs on the image —
`libxml2` CVE-2026-74860 and CVE-2026-86140, and `zlib` CVE-2026-85091. All
three are **base-image OS packages with no fixed version in Debian 13**, and
none are reachable by this app:

| CVE | Package | Why not affected |
|---|---|---|
| CVE-2026-74860 | libxml2 | Flaw in the libxml2 **Python SAX bindings**; not installed (no `python3-libxml2`/`lxml`, and the `libxml2` package ships no Python bindings). |
| CVE-2026-86140 | libxml2 | `xmlSnprintfElements()` DTD-validation overflow; the app parses **no untrusted DTD XML**, so no untrusted XML reaches libxml2 (pulled only by `postgresql-17`). |
| CVE-2026-85091 | zlib | Overflow is in `gz_vacate()`, introduced upstream in zlib **1.3.1.2**; the image ships `+really1.3.1` where the symbol is absent. Scanner false positive. |

These are recorded as OpenVEX `not_affected` statements in
[`vex/personal-expense-tracker.vex.json`](vex/personal-expense-tracker.vex.json),
which the image embeds at `/usr/local/share/vex/`. Docker Scout reads
`*.vex.json` from the image filesystem and filters them from its results.

> **Two requirements:** (1) the image must have **no attestations** — embed VEX
> is ignored whenever the image has any — so build/publish with
> `--provenance=false --sbom=false`; (2) the Scout CLI's default trust filter
> only accepts `*@docker.com` authors, so pass `--vex-author '.*'` when
> scanning locally.

```bash
# verify: embedded VEX removes all three (HIGH 3 -> 0)
docker scout cves vipulverulkar/personal-expense-tracker-v6.1:latest --vex-author '.*'
```

### Windows notes

- Use Docker Desktop in Linux-containers mode (the default) — the image
  is Linux-based and runs unchanged on Windows, macOS and Linux.
- Keep the **named volumes** shown above. Do not bind-mount a Windows
  folder to `/var/lib/postgresql/data` (Postgres needs POSIX file
  permissions); `/data` likewise works best as a named volume.
- Line endings: the repo ships `.gitattributes` forcing LF, and the build
  normalizes `entrypoint.sh` anyway — but if you hand-edit `app/.env` on
  Windows, save it with LF endings so no stray `\r` lands in values.

## Config

Settings come from environment variables; an optional `app/.env` file is
loaded at startup (real environment variables take precedence). A safe,
non-secret subset is editable from **Master Data → Settings** (admin) and
applies after a restart. The table below lists the common variables
(`app/.env.example` documents the rest).

| Variable | Default | Description |
|---|---|---|
| `ENV_FILE` | `app/.env` | Path of the optional config file the app loads at startup |
| `SECRET_KEY` | dev default / auto-generated in Docker | Flask sessions — set explicitly in production; when unset, the container generates a random key on first launch and persists it at `/data/.secret_key` |
| `DATABASE_URL` | in-container server | PostgreSQL or SQLite DSN — e.g. `postgresql://user:password@127.0.0.1:5432/exptracker` or `sqlite:///path/to/data.db`; switch between **Local** (default) and **Remote** from **Master Data → Settings → Database** (connection is tested before save) |
| `POSTGRES_USER` / `POSTGRES_PASSWORD` / `POSTGRES_DB` | `exptracker` | In-container Postgres role, password and database (persisted in `/var/lib/postgresql/data`) |
| `SESSION_TIMEOUT_MINUTES` | `5` | Idle minutes before auto-logout (per-user override on Profile) |
| `SESSION_TYPE` | `filesystem` | Server-side sessions; `cookie` reverts to client-side cookies |
| `CURRENCY_SYMBOL` | `₹` | Display symbol in UI and CSV |
| `DATE_FORMAT` | `%d/%m/%Y` | Date-only display format (Python strftime) |
| `DATETIME_FORMAT` | `%d/%m/%Y %H:%M:%S` | Timestamp display format (Python strftime) |
| `AUTO_BACKUP` | off | `1` = write a SQL-dump backup to `BACKUP_DIR` on every startup |
| `BACKUP_DIR` / `BACKUP_KEEP` | `backups/` / `10` | Auto-backup location and rotation count |
| `AUDIT_RETENTION_DAYS` | `0` (keep all) | Purge audit entries older than N days on startup |
| `RATELIMIT_STORAGE_URI` | `memory://` | Rate-limit store; use `redis://…` for multi-worker deployments |
| `RATELIMIT_DEFAULT` | `200 per minute` | Request rate limit (Flask-Limiter format) |
| `SESSION_CACHE_THRESHOLD` | `500` | Server-side session cache size (entries) |
| `TOTP_PENDING_TTL_SECONDS` | `600` | 2FA half-session lifetime (seconds) |
| `MIN_TIMEOUT_MINUTES` / `MAX_TIMEOUT_MINUTES` | `1` / `1440` | Per-user idle-timeout override bounds |
| `MIN_USERNAME_LEN` / `MAX_USERNAME_LEN` | `3` / `32` | Username length policy |
| `MIN_PASSWORD_LEN` | `8` | Password minimum length (letters + digits still required) |
| `MIN_FONT_ZOOM` / `MAX_FONT_ZOOM` | `70` / `160` | Per-user font-size (UI zoom) bounds in percent |
| `DEFAULT_FONT_ZOOM` | `90` | Site-wide font-size (UI zoom) used when a user has no personal override |
| `FONT_ZOOM_STEP` | `5` | Step of the Profile font-size slider (percent) |
| `DEFAULT_USERNAME` / `DEFAULT_PASSWORD` | `admin` / `admin` | First-boot seed account (only when users table is empty) |
| `CSV_IMPORT_ROW_LIMIT` | `2000` | Max rows per CSV/bank import |
| `BANK_IMPORT_MAX_BYTES` | `2097152` | Max bank file upload (bytes) |
| `BANK_IMPORT_ROW_LIMIT` | `2000` | Max bank transactions per import |
| `BANK_IMPORT_PREVIEW_ROWS` | `8` | Bank import preview sample size |
| `RECEIPT_MAX_BYTES` | `5242880` | Max receipt file size (bytes) |
| `RECEIPT_ALLOWED_EXTS` | `png,jpg,jpeg,gif,pdf,webp` | Allowed receipt extensions |
| `RECURRING_MAX_CATCHUP` | `366` | Recurring engine safety cap (iterations per run) |
| `PER_PAGE_DEFAULT` / `PER_PAGE_CHOICES` | `25` / `10,25,50,100` | Pagination default and allowed page sizes |
| `TRANSACTIONS_PER_PAGE_DEFAULT` | `10` | Transactions list page size |
| `SENTRY_DSN` | *(unset)* | Enables Sentry error tracking when set |
| `PORT` | `5000` | HTTP port for `python run.py` and gunicorn (`gunicorn.conf.py` honors it; the Docker healthcheck follows it too) |
| `MIGRATE_FROM_DATABASE_URL` | *(unset)* | Source PostgreSQL DSN to automatically migrate existing data from on startup |
| `AUTO_MIGRATE` | `off` | Set `1` to run auto-migration even if target tables already contain data |

## Tests

```bash
pip install -r requirements-dev.txt   # adds pytest + pyflakes
# Postgres or SQLite can be used; e.g. for PostgreSQL:
export DATABASE_URL=postgresql://user:password@localhost:5432/exptracker_test
python3 -m pytest -q
# lint (must report no issues):
python -m pyflakes app/ run.py seed_demo.py seed_bulk.py tests/ gunicorn.conf.py scripts/
```

## Structure

Flask MVC with an app factory (`create_app`) — each module under `app/modules/`
is a self-contained blueprint with its own **models** (`models.py` — data
access), **controllers** (`controllers.py` — routes) and **views**
(`templates/`):

```
run.py                  entry point (dev server + gunicorn `run:app`)
requirements.txt        runtime deps (installed in the Docker image)
requirements-dev.txt    adds pytest + pyflakes for local tests
requirements-desktop.txt PyInstaller & pywebview desktop dependencies
app/
  __init__.py           app factory, CSRF/rate limiting, security headers, /healthz, /metrics
  config.py             env-driven configuration (defaults + loads app/.env at startup)
  db.py                 PostgreSQL / SQLite adapter, schema bootstrap + migrations, tsvector search
  helpers.py            INR formatting, cents conversion, month/date utilities
  jobs.py               automatic backup + rotation
  templates/            shared views: base.html, error.html, _macros.html
  static/               style.css (design tokens, dark/light themes), app.js
  modules/
    accounts/           multi-account management, liquid balance, net worth, transfers
    rules/              transaction categorizer & tagger rule engine
    calendar_view/      monthly calendar of bills, debt payments & cashflow forecast
    goals/              financial targets, sinking funds, progress tracking
    debts/              debt reduction planner (Avalanche vs. Snowball payoff simulators)
    shared/             group expense splitting & participant settlements
    notifications/      in-app alerts, budget threshold warnings, bill notifications
    auth/               login (+ TOTP step), roles, profile, user admin, audit log
    dashboard/          home: summaries, monthly total expense, yearly expense, quick add, recent activity, budget alerts
    transactions/       list/filter/sort/paginate, CRUD, splits, tags, receipts, trash, integrity check
    recurring/          scheduled transactions (daily/weekly/monthly/yearly)
    categories/         category management
    budgets/            monthly limits, rollover budgets + status
    reports/            overview (6-month trend, top spending, budgets), daily/monthly charts, variance, tags breakdown, forecast, exports
    data/               bulk CSV import/restore, SQL-dump backup/restore, integrity check
    bank_import/        OFX, QFX, QIF and CSV bank statement parsing & import
    import_templates/   saved bank-import column mapping templates
    settings/           admin editor for a safe subset of app/.env tunables
    api/                JSON endpoints (/api + /api/v1, no views)
expenses.sql            PostgreSQL schema reference
seed_demo.py            demo data generator (~140 transactions, budgets, accounts, goals, debts)
seed_bulk.py            bulk seeder (~10,000 transactions over 10 years, with yearly growth)
scripts/
  generate_screenshots.py headless browser screenshot generation tool
vex/                    OpenVEX container-scan exceptions (embedded in the Docker image)
tests/                  pytest suite with conftest.py shared fixtures
screenshots/            README & documentation screenshots
```

## Developer

- Name: Vipul Verulkar
- Email: contact@vipulverulkar@gmail.com
- Location: India

## License

This project is free software: you can redistribute it and/or modify it under
the terms of the **GNU General Public License, version 3 or later
(GPL-3.0-or-later)**, as published by the Free Software Foundation.
