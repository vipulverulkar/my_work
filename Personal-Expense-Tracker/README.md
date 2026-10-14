# Personal Expense Tracker 💰

Track income, expenses, budgets and reports — a Flask + PostgreSQL web
app with dark mode, charts, recurring transactions and roles.

![Dashboard](screenshots/dashboard.png)

## Features

**Money management**
- **Dashboard** — balance, monthly stats, savings rate, 6-month trend, top spending, recent activity, budget alerts
- **Savings buckets** — flag any expense category as 🏦 savings (a `Savings` bucket is seeded); parked money is excluded from spending totals and the savings rate, and shown as Saved instead — across the dashboard, reports, API and CSV totals. Savings buckets can't carry budgets
- **Transactions** — filter (type / category / user / month dropdown / full-text note search), sort, paginate, edit, duplicate; receipt attachments (image/PDF, max 5 MB); guided bulk actions (delete to Trash with confirm, or re-categorize) via row checkboxes; every entry records who it is managed for
- **Recurring** — daily / weekly / monthly / yearly schedules with a start date and optional end date (rent, salary, subscriptions); due items are generated automatically on dashboard load, finished schedules show as ended
- **Split transactions** — one expense spread over up to 3 extra categories, linked as a group so reports stay accurate
- **Budgets** — monthly limits per category with progress bars and over-budget alerts
- **Reports** — daily and monthly charts, budget-vs-actual variance, 3-month cash-flow forecast, PDF + Excel (`.xlsx`) export
- **More reports** — yearly overview with year-over-year growth and a 12-month savings-rate trend; **yearly trends** and **monthly trends** tabs (year-by-year totals with growth + 36-month momentum across all years); top transactions (largest income/expense entries) with per-category average-ticket and share; per-category × month deep-dive matrix with total/average/best month; insights (weekday spending pattern, spend per member, savings-bucket mix vs spending, recurring commitments)
- **Trash** — deleted transactions are kept (soft delete) with restore / permanent purge
- **Nav extras** — dark/light theme toggle (persisted, respects OS preference) and fullscreen toggle (`⛶`); installable PWA manifest (`/manifest.webmanifest`, standalone display)

**Data**
- **Import & export** — CSV export (respects filters) and import (additive — never deletes)
- **Bank statement import** (admin) — guided upload → column mapping → preview → import flow for bank CSVs (2 MB / 2000-row caps, 8-row preview, staging in server-side session, saved column-mapping templates)
- **Backup & restore** — SQL-dump backup of the whole database; restore from `.sql`, legacy `.json`, or CSV files (validated first — invalid files change nothing); optional automatic startup backups with rotation

**Accounts & security**
- **Roles** — admin (full access) and viewer (read-only); every write route is server-enforced
- **Two-factor auth** — TOTP via any authenticator app, enforced as a second login step
- **User management** — add accounts, reset passwords, change roles (passwords: min 8 chars, letters + digits)
- **Font size** — per-user zoom of the whole UI (70–160%) from Profile, with live preview; stored on the account and applied to every page (site default 90%)
- **Audit log** — deletes, password resets, imports and role changes are recorded; retention purging
- **Site settings** — admins edit a safe, non-secret subset of the `app/.env` config (currency, display formats, session/account policy, pagination, import limits, rate limit, backups) from **Master Data → Settings**; changes apply on restart
- **Sessions** — idle auto-logout after 5 min (per-user override, `SESSION_TIMEOUT_MINUTES` for the default) with a live nav countdown that resets on activity, warns at 1 min and signs out automatically; server-side sessions (CacheLib filesystem backend), login rate limiting, CSRF protection on all forms
- **Ops** — health check (`/healthz`), Prometheus metrics (`/metrics`), JSON API (`/api/v1/*`, legacy `/api/*`), optional Sentry

## Documentation

- **[User Guide](USER_GUIDE.md)** — how to use every screen (sign-in, transactions, budgets, reports, imports, backup, profile, users, settings).
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
python3 seed_demo.py  # optional demo data (~130 transactions + budgets)
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

## Screenshots

Captured with the demo/bulk seeders — login, day-to-day pages, users, reports and the numbered backup/restore flow:

| Login | Transactions | Reports |
|---|---|---|
| ![Login](screenshots/login.png) | ![Transactions](screenshots/transactions.png) | ![Reports](screenshots/reports.png) |

| Budgets | Data & backup | Restore |
|---|---|---|
| ![Budgets](screenshots/budgets.png) | ![Data](screenshots/data.png) | ![Restore](screenshots/restore.png) |

| Users | User profile |
|---|---|
| ![Users](screenshots/users.png) | ![User profile](screenshots/user-profile.png) |

| Categories | Recurring | Trash |
|---|---|---|
| ![Categories](screenshots/categories.png) | ![Recurring](screenshots/recurring.png) | ![Trash](screenshots/trash.png) |

| Profile | Audit | Settings |
|---|---|---|
| ![Profile](screenshots/profile.png) | ![Audit](screenshots/audit.png) | ![Settings](screenshots/settings.png) |

| Yearly trends | Monthly trends |
|---|---|
| ![Yearly trends](screenshots/yearly-trends.png) | ![Monthly trends](screenshots/monthly-trends.png) |

## Docker

One container runs both the app and PostgreSQL — no compose setup needed:

```bash
docker build -t personal-expense-tracker .
docker run -d -p 5000:5000 \
  -v exptracker-pg:/var/lib/postgresql/data \
  -v exptracker-data:/data \
  --name personal-expense-tracker personal-expense-tracker
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
  --env-file app/.env --name personal-expense-tracker personal-expense-tracker
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
docker exec personal-expense-tracker pg_dump -U exptracker exptracker > personal-expense-tracker-dump.sql
```

```powershell
# same volume backup on Windows PowerShell (use ${PWD}, not $PWD):
docker run --rm -v exptracker-pg:/var/lib/postgresql/data -v exptracker-data:/data -v "${PWD}:/backup" busybox tar czf /backup/personal-expense-tracker-backup.tgz /var/lib/postgresql/data /data
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
| `DATABASE_URL` | in-container server | PostgreSQL DSN — defaults to the container's own server, e.g. `postgresql://user:password@127.0.0.1:5432/exptracker`; switch between **Local** (default) and **Remote** from **Master Data → Settings → Database** (connection is tested before save) |
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
| `BANK_IMPORT_MAX_BYTES` | `2097152` | Max bank CSV upload (bytes) |
| `BANK_IMPORT_ROW_LIMIT` | `2000` | Max bank CSV rows per import |
| `BANK_IMPORT_PREVIEW_ROWS` | `8` | Bank import preview sample size |
| `RECEIPT_MAX_BYTES` | `5242880` | Max receipt file size (bytes) |
| `RECEIPT_ALLOWED_EXTS` | `png,jpg,jpeg,gif,pdf,webp` | Allowed receipt extensions |
| `RECURRING_MAX_CATCHUP` | `366` | Recurring engine safety cap (iterations per run) |
| `PER_PAGE_DEFAULT` / `PER_PAGE_CHOICES` | `25` / `10,25,50,100` | Pagination default and allowed page sizes |
| `TRANSACTIONS_PER_PAGE_DEFAULT` | `10` | Transactions list page size |
| `SENTRY_DSN` | *(unset)* | Enables Sentry error tracking when set |
| `PORT` | `5000` | HTTP port for `python run.py` and gunicorn (`gunicorn.conf.py` honors it; the Docker healthcheck follows it too) |

### Database (PostgreSQL only)

PostgreSQL is the only supported backend (`psycopg` is in
`requirements.txt`). Inside the container the entrypoint starts a local
Postgres server and defaults `DATABASE_URL` to it — set `DATABASE_URL`
explicitly only to use an external database instead:

```bash
DATABASE_URL=postgresql://user:password@localhost:5432/exptracker
```

The schema is created (and older databases migrated) automatically on startup;
`seed_demo.py` seeds demo data into the configured database. The active
database is shown on the **Data** page and in `/healthz`.

Full-text note search uses a generated `tsvector` column with a GIN index —
with a plain case-insensitive (`ILIKE`) fallback if it is unavailable.
Category/name search is case-insensitive too.

### Backup & restore

The **Data** page is split into numbered sections so backup and restore can't
be mixed up:

1. **Backup (download, safe)** — SQL dump covering categories, budgets,
   transactions, users and recurring transactions (downloaded as
   `personal-expense-tracker-backup-….sql`). A legacy `.json`
   download is also available.
2. **Restore from backup file (destructive)** — upload a `.sql` dump (or a
   legacy `.json` snapshot) to replace all current data. Guarded by a
   confirmation checkbox, validated before anything is wiped, id sequences
   reset — and existing data is left untouched if the file is invalid.
   (Backup files keep the historical `exptracker` format id/header, so
   backups from older releases still restore.)
3. **CSV files — one picker, two modes** — upload the same three CSVs and
   choose **Add** (appends rows, never deletes) or **Replace** (wipes and
   re-imports the data tables behind a confirmation checkbox — atomic: a
   mid-restore failure rolls everything back). Users are
   always preserved (CSV has no user format); recurring schedules are
   removed. Use a `.sql` backup for a full-fidelity restore.

No upload size limit is enforced. For raw engine-level copies use `pg_dump`.

## Tests

```bash
pip install -r requirements-dev.txt   # adds pytest + pyflakes (runtime deps come too)
# Postgres must be running; point the suite at it:
export DATABASE_URL=postgresql://user:password@localhost:5432/exptracker_test
python3 -m pytest -q
# lint (the only expected hits are intentional: the route-registration
# side-effect imports in app/modules/*/__init__.py, and the
# PER_PAGE_CHOICES re-export in transactions/models.py used by its
# controller as models.PER_PAGE_CHOICES):
python -m pyflakes app/ run.py seed_demo.py seed_bulk.py tests/ gunicorn.conf.py
```

## Structure

Flask MVC with an app factory (`create_app`) — each module under `app/modules/`
is a self-contained blueprint with its own **models** (`models.py` — data
access), **controllers** (`controllers.py` — routes) and **views**
(`templates/`):

```
run.py            entry point (dev server + gunicorn `run:app`)
requirements.txt        runtime deps (installed in the Docker image)
requirements-dev.txt    adds pytest + pyflakes for local tests
app/
  __init__.py     app factory, CSRF/rate limiting, security headers, /healthz, /metrics
  config.py       env-driven configuration (defaults + loads app/.env at startup)
  db.py           PostgreSQL adapter, schema bootstrap + migrations, tsvector search
  helpers.py      INR formatting, cents conversion, month/date utilities
  jobs.py         automatic backup + rotation
  templates/      shared views: base.html, error.html
  static/         style.css (light/dark; :root design tokens — type/radius/space scales), app.js (theme + fullscreen toggles, INR formatting, validation, bulk select, idle countdown)
  modules/
    auth/         login (+ TOTP step), roles, profile, user admin, audit log
    dashboard/    home: summaries, 6-month trend, top spending, budget alerts
    transactions/ list/filter/sort/paginate, CRUD, splits, receipts, bulk ops, trash, CSV import/export
    recurring/    scheduled transactions (daily/weekly/monthly/yearly)
    categories/   category management
    budgets/      monthly limits + status
    reports/      daily/monthly charts, budget variance, forecast, yearly + YoY, top transactions, category deep-dive, insights, PDF + Excel export
    data/         bulk CSV import, CSV restore, SQL-dump (+ legacy JSON) backup/restore
    bank_import/  bank CSV upload → column map → preview → import (+ saved mapping templates, admin-only writes)
    import_templates/ CSV encoding detection + saved bank-import mapping templates
    settings/     admin editor for a safe subset of app/.env tunables
    api/          JSON endpoints (/api + /api/v1, no views)
expenses.sql      schema reference
seed_demo.py      demo data generator (~130 transactions, 6 months)
seed_bulk.py      bulk seeder (~10,000 transactions over 10 years, with yearly growth)
tests/            pytest suite
screenshots/      README screenshots
```

Schema: `categories(id, name, type)` · `transactions(id, amount*, type,
category_id, date, note, split_group, user_id,
deleted_at)` · `budgets(id, category_id, monthly_limit*)` ·
`recurring_transactions(id, amount*, frequency, next_run_date, active, …)` ·
`users(id, username, password_hash, role, totp_*, session_timeout_minutes,
font_zoom)` · `audit_logs(id, user_id, action, target_*, details)` — `*` = integer
cents.
