# Architecture — Personal Expense Tracker

A single-container **Flask + PostgreSQL** web application for tracking income,
expenses, budgets, recurring schedules and reports. This document describes how
the code is organised, how a request flows through it, the data model, and the
operational design. For end-user instructions see [`USER_GUIDE.md`](USER_GUIDE.md).

- **Version:** 2.0.0 (`app/config.py:APP_VERSION`)
- **Backend:** Python 3.12, Flask 3.x, PostgreSQL (only supported engine)
- **Frontend:** Server-rendered Jinja2 templates, vanilla JS (Chart.js from CDN), one CSS file
- **Packaging:** One Docker image runs both the app (gunicorn) and PostgreSQL

---

## 1. High-level overview

```
                                   ┌───────────────────────────────┐
 Browser (desktop / phone, PWA)    │  Flask app  (gunicorn, 2 wrk) │
 ┌───────────────────────┐         │  ┌─────────────────────────┐  │
 │  Jinja-rendered HTML  │◄────────┼──┤  Blueprint layers        │  │
 │  style.css / app.js   │  HTTPS  │  │  auth · dashboard · …    │  │
 │  Chart.js (CDN)       │─────────┼─►│  controllers (routes)    │  │
 └───────────────────────┘  forms  │  │  models (SQL via db.py)  │  │
                                   │  └───────────┬─────────────┘  │
                                   │              │ psycopg3        │
                                   └──────────────┼────────────────┘
                                                  ▼
                                        ┌──────────────────────┐
                                        │  PostgreSQL server   │
                                        │  (same container)    │
                                        └──────────────────────┘
             /data volume:  sessions · backups · secret key
```

Everything is one process tree inside one container. `entrypoint.sh` starts
PostgreSQL, prepares the role/database, then `exec`s gunicorn as an unprivileged
user.

---

## 2. Application factory & request lifecycle

Entry points:

- `run.py` — `app = create_app()`; `python run.py` runs the dev server,
  `gunicorn -c gunicorn.conf.py run:app` runs production.
- `app/__init__.py:create_app()` — builds the Flask app.

`create_app()` performs, in order:

1. `app.config.from_object(Config)` (env-driven; `config.py` has already loaded
   the optional `app/.env` into the environment at import time), optional test
   overrides.
2. Warn if `SECRET_KEY` is the insecure dev default.
3. `_init_sentry(app)` — optional Sentry (guarded import).
4. `_init_sessions(app)` — server-side sessions (Flask-Session + cachelib;
   falls back to cookie sessions).
5. `csrf.init_app(app)` and `limiter.init_app(app)`.
6. `app.teardown_appcontext(close_db)`.
7. `_register_template_helpers` (filters + context processor).
8. `_register_hooks` (security headers, `/healthz`, `/manifest.webmanifest`,
   optional `/metrics`, error handlers).
9. `_register_blueprints`.
10. `init_db(app.config)` — create tables/indexes and migrate.
11. Startup maintenance: audit retention purge, optional auto-backup.
12. Warn if default credentials are still in use.

### Request pipeline

```
request ─► auth.require_login (before_app_request; the login guard)
         ─► view function (controller) ─► models ─► db.py ─► PostgreSQL
         ─► after_request: security headers + Prometheus counter
         ─► teardown_appcontext: close_db()
```

The **single global guard** is `app/modules/auth/controllers.py:
require_login`, registered as `@bp.before_app_request`. It:

- Lets `PUBLIC_ENDPOINTS` through (`auth.login`, `auth.login_totp`, `healthz`,
  `manifest`, `static`).
- Returns **401 JSON** for `/api*` and `auth.ping` when unauthenticated.
- Otherwise redirects to `/login`.
- For an authenticated user, resolves `g.is_admin` and the effective idle
  timeout, stamps `g.font_zoom`, and enforces the idle timeout (clears the
  session and redirects when expired). Every request refreshes `last_active`.

Admin-only routes are protected per view with the `@admin_required` decorator
(thin wrapper over `g.is_admin`).

---

## 3. Layered MVC per module

Each feature under `app/modules/` is a self-contained blueprint with three
layers:

| Layer | File | Responsibility |
|---|---|---|
| **Controller** | `controllers.py` | Parse/validate the request, call the model, render a template or redirect, flash messages, write audit entries. No SQL. |
| **Model** | `models.py` | All SQL and domain rules. Returns plain dicts/lists; money leaves as **rupees (floats)** via `from_cents`. |
| **View** | `templates/...` | Jinja2 templates extending `base.html`. |

Views never contain SQL; models never touch `request`. The `?` placeholder in
model SQL is translated to psycopg's `%s` by `app/db.py:to_pg`, so model code
stays engine-agnostic and readable.

---

## 4. Project structure

```
run.py                    dev server + gunicorn `run:app`
gunicorn.conf.py          2 workers, port from $PORT, preload_app
entrypoint.sh             starts Postgres, seeds role/db, generates SECRET_KEY, execs app
Dockerfile                multi-stage build (wheels + runtime with PostgreSQL)
expenses.sql              reference schema (same as init_db)
seed_demo.py              demo data generator (~130 transactions, 6 months)
seed_bulk.py              bulk seeder (~10,000 transactions over 10 years)
app/
  __init__.py             create_app, hooks, context processor, blueprint registry
  config.py               env-driven configuration (single source of defaults; loads app/.env)
  .env.example            documented template of the optional config file (app/.env is git-ignored)
  db.py                   Postgres adapter, schema bootstrap + migrations, tsvector search
  helpers.py              INR formatting, cents conversion, month/date & pagination utils
  jobs.py                 automatic SQL-dump backups + rotation
  templates/
    base.html             app shell (nav, toasts, theme, zoom, idle countdown)
    _macros.html          shared macros (zoom_attr)
    error.html            404/500 page
  static/
    style.css             theme + :root design tokens (type/radius/space scales)
    app.js                theme & fullscreen toggles, INR input formatting, validation, bulk select, idle timer
    receipts/             uploaded receipt attachments
    icon.svg              PWA icon
  modules/
    auth/                 login (+ TOTP second factor), roles, profile, user admin, audit log
    dashboard/            home: totals, 6-month trend, top spending, budget alerts
    transactions/         list/filter/sort/paginate, CRUD, splits, receipts, bulk ops, trash, CSV import/export
    categories/           category CRUD, income/expense, savings-bucket flag
    budgets/              monthly limits + status
    recurring/            scheduled transactions (daily/weekly/monthly/yearly) + engine
    reports/              daily/monthly/yearly/top/categories/insights/budgets/forecast + PDF/Excel
    data/                 SQL/JSON backup & restore, bulk CSV import/restore
    bank_import/          bank CSV upload → column map → preview → import (+ saved templates)
    import_templates/     CSV encoding detection, column-mapping guess/apply helpers
    settings/             admin editor for whitelisted app/.env tunables
    api/                  read-only JSON endpoints (/api and /api/v1)
tests/
  test_app.py             pytest suite (Postgres required)
```

---

## 5. Module reference

### auth (`app/modules/auth/`)
Authentication, account settings, user administration and the audit log.

- **Routes:** `GET/POST /login`, `GET/POST /login/totp`, `GET/POST /logout`,
  `GET /session/ping`, `GET/POST /profile`, `POST /profile/font-size`,
  `POST /profile/totp/{enable,confirm,disable}`, `GET/POST /users`,
  `GET /users/<id>`, `POST /users/{password,role,delete}/<id>`, `GET /audit`,
  `POST /audit/purge`.
- **Model:** password hashing (werkzeug), roles, TOTP (pyotp, guarded),
  per-user idle timeout, per-user font zoom, managed-expense stats, audit writes.
- **Templates:** `login.html`, `totp.html`, `profile.html`, `users.html`,
  `user_profile.html`, `audit.html`.

### dashboard (`/`)
- **Route:** `GET /`.
- Runs the recurring engine (`run_due`) on every load so the dashboard is
  current, then aggregates today/month/all-time totals, a 6-month trend, top
  categories, recent activity and budget alerts.

### transactions
- **Routes:** `GET /transactions`, `GET /transactions/export`,
  `POST /transactions/import`, `POST /transactions/bulk`, `POST /add`,
  `GET/POST /edit/<id>`, `POST /duplicate/<id>`, `POST /delete/<id>`,
  `GET /trash`, `POST /trash/<id>/{restore,purge}`,
  `GET /transactions/split/<group>`, `POST /transactions/split/<group>/delete`.
- **Model:** validation, split lines
  (a shared `split_group` id), soft delete/restore/purge, CSV import/export,
  filtered list/sums, full-text note search (tsvector, ILIKE fallback).
- **Templates:** `list.html`, `edit.html`, `trash.html`, `split.html`.

### categories
- **Routes:** `GET/POST /categories`, `GET/POST /categories/edit/<id>`,
  `POST /categories/delete/<id>`, `POST /categories/savings/<id>`.
- **Model:** CRUD, income/expense typing, savings-bucket flag, default seeding,
  usage counts.

### budgets
- **Routes:** `GET/POST /budgets`, `POST /budgets/delete/<category_id>`.
- **Model:** per-category monthly limit upsert, month status (spent, %, over/near).

### recurring
- **Routes:** `GET/POST /recurring`, `POST /recurring/<id>/{pause,resume,delete}`,
  `POST /recurring/run`.
- **Model:** `FREQUENCIES`, `_advance` (monthly/yearly keep the day-of-month
  anchor), `validate`, `due_count`, and `run_due` — the idempotent generation
  engine (one transaction per due date, capped by `RECURRING_MAX_CATCHUP`).

### reports
- **Routes:** `GET /reports` (views: `daily`, `monthly`, `yearly`, `yearly-trends`, `monthly-trends`,
  `top`, `categories`, `insights`, `budgets`, `forecast`), `GET /reports/export-xlsx`,
  `GET /reports/export-pdf`.
- **Model:** daily/monthly/yearly rollups, YoY comparison, 12-month trends,
  multi-year series (`yearly_series`), top
  transactions, per-category stats & year matrix, weekday/member/savings
  insights, budget variance and a 3-month cash-flow forecast.
- **View:** a single `reports.html` switching on `view`; Chart.js bar/line/pie
  charts, theme-aware.

### data
- **Routes:** `GET /data`, `GET /data/backup`, `GET /data/backup.json`,
  `POST /data/restore`, `POST /data/restore-csv`, `POST /data/import`.
- **Model:** `export_sql_backup`/`restore_sql_backup` (primary, data-only SQL
  dump; only DELETE/INSERT against known tables accepted, literal-only values),
  legacy JSON export/restore, CSV import + CSV replace, sequence reset.

### bank_import (`/bank-import`)
- **Routes:** `GET /`, `POST /map`, `POST /preview`, `POST /import`,
  `POST /template/save`, `POST /template/delete/<id>`.
- **Flow:** upload → stage in session → auto-guess columns → preview (validated,
  nothing written) → import. Duplicate-protected per upload.
- **Model helpers:** `import_templates` (encoding detection, header aliasing,
  debit/credit pairing, date/amount parsing, saved mapping templates).

### settings (`/settings`)
- **Routes:** `GET/POST /settings` (admin only), `POST /settings/database`
  (admin only), `POST /settings/restart` (admin only).
- **Scope:** view/edit a whitelisted, non-secret subset of the `app/.env`
  tunables (display, sessions, account policy, pagination, import guards,
  rate limit, maintenance). Each spec points at the `Config` attribute that
  holds its default, keeping `config.py` the single source of truth. Model
  helpers read/validate/upsert `KEY=VALUE` lines, preserving comments and
  unrelated keys; writes are atomic (mode `0600`). Other secrets and infra
  keys are not exposed. Restart to apply.
- **Database:** switch between **Local** (the default — the app's own /
  built-in database; removes `DATABASE_URL` from the file) and **Remote**
  (host/port/db/user/password, composed into a `DATABASE_URL`). The mode is
  inferred: a file-saved `DATABASE_URL`, or an environment URL whose host is
  not local, reads as `remote`; otherwise `local`. The password is never
  rendered back (masked as `****`, blank keeps the stored one), the connection
  is tested before anything is saved, and audit entries exclude credentials. A
  `DATABASE_URL` set in the real environment shadows the file and is flagged
  in the UI.
- **Restart:** the **↻ Restart app** button triggers `_restart_process()` on a
  short delay so the response flushes first — a gunicorn worker signals its
  master (`SIGHUP` → graceful reload of all workers), while a dev-server
  process re-execs itself (`os.execv`). Disabled under `TESTING` so tests
  never kill the runner.

### api (`/api`, `/api/v1`)
Read-only JSON: `GET /summary` (`?month=`/`?year=`), `GET /transactions`
(`?limit=`, `?type=`). Mounted at both the legacy `/api` and versioned
`/api/v1` prefixes; the same handlers serve both.

---

## 6. Data model

Money columns are **integer cents** (no float rounding).
Dates are stored as `TEXT` in `YYYY-MM-DD`. Schema lives in `app/db.py:SCHEMA`
and mirrors `expenses.sql`.

```
categories(id, name UNIQUE, type ∈ {income,expense}, is_savings ∈ {0,1})
import_templates(id, name UNIQUE, mapping JSON-text, created_at)
users(id, username UNIQUE, password_hash, role ∈ {admin,viewer},
      totp_secret, totp_enabled, session_timeout_minutes, font_zoom, created_at)
transactions(id, amount*, type, category_id→categories, date, note,
             split_group, user_id→users, created_at, deleted_at,
             receipt_path, search_tsv TSVECTOR GENERATED)
budgets(id, category_id UNIQUE→categories, monthly_limit*)
recurring_transactions(id, user_id→users, amount*, type, category_id, note,
             frequency ∈ {daily,weekly,monthly,yearly},
             next_run_date, start_date, end_date, anchor_day, active, last_run_at,
             created_at)
audit_logs(id, user_id→users, action, target_table, target_id, timestamp, details)
```

`*` = integer cents. Indexes: `idx_tx_date`, `idx_tx_cat`, `idx_tx_type`,
`idx_tx_user`, `idx_tx_tsv` (GIN on `search_tsv`).

**Migrations** (`db.py:_migrate`, idempotent, run on every boot): money REAL→cents,
legacy table renames (`recurrences`→`recurring_transactions`,
`audit_log`→`audit_logs`), additive columns (`users.font_zoom`,
`categories.is_savings`, `transactions.{user_id,deleted_at,
split_group,receipt_path}`, `recurring_transactions.{start_date,end_date,
anchor_day}`), tsvector column + GIN index, anchor backfill, and dropping retired
tables/columns (`members`, `transactions.member_id`, and the former
`currency`/`orig_amount` columns).

### Key domain rules

- **Amounts.** Entered in rupees and stored as integer cents (`helpers.to_cents`,
  `ROUND_HALF_UP`).
- **Savings buckets.** A category flagged `is_savings` is excluded from all
  spending totals, the savings rate and budget checks; its amounts are shown as
  *Saved*. Savings buckets cannot carry budgets.
- **Splits.** One logical expense is stored as several rows sharing a
  `split_group`; the remainder stays on the main category. A split line cannot
  change the group's type.
- **Soft delete.** Deleting a transaction sets `deleted_at` (Trash); restore
  clears it, purge removes the row and its receipt file.

---

## 7. Key data flows

**Add a transaction** — `POST /add` → `models.validate` (amount → cents) →
optional `parse_splits` → optional receipt save → one `INSERT` (or several for a
split, sharing a group) → flash → redirect.

**Recurring generation** — dashboard load (or `POST /recurring/run`) calls
`run_due(db, today)`: selects active schedules due today, inserts one
transaction per occurrence advancing the cursor and preserving the day-of-month
anchor, stops at the end date or the catch-up cap (capping pauses the schedule).

**Reports** — `GET /reports?view=…` validates params (invalid → flash+redirect,
defaults stripped from the URL), dispatches to the matching `_view` helper,
which calls report model functions and renders the theme-aware charts. Exports
reuse the same models.

**Bank import** — file is size-checked, encoding-detected, parsed; mapping is
auto-guessed then user-adjustable; preview validates rows without writing;
import writes rows with per-row duplicate protection.

**Backup / restore** — `GET /data/backup` streams a data-only SQL dump
(`DELETE`s + literal `INSERT`s with explicit ids). Restore validates the header
and accepts only DELETE/INSERT against known tables with literal-only values,
wipes in FK-safe order, re-inserts, resets sequences — all in one transaction
(any failure rolls back, leaving data intact). Legacy `.json` and CSV restore
are also supported.

---

## 8. Configuration

All tunables live in `app/config.py` — the **single source of defaults**. Other
modules import the module-level constants rather than re-hardcoding literals;
most are also on the `Config` class for `app.config[...]` access. Values are
read from the environment at import time (restart to apply) and documented in
[`app/.env.example`](app/.env.example) and the README table. At import the app
loads an optional `app/.env` file via `python-dotenv` (`override=False`, so real
environment variables always win; override the path with `ENV_FILE`).

Admins can edit a safe, non-secret subset of these keys from **Master Data →
Settings** (`app/modules/settings/`). The page reads and upserts `KEY=VALUE`
lines in `app/.env` (comments and unlisted keys preserved, written atomically
mode `0600`); because config is read once at startup, changes apply on restart.
Other secrets/infra keys (`SECRET_KEY`, `POSTGRES_*`, `SENTRY_DSN`,
seed credentials, paths) are deliberately excluded from the UI; `DATABASE_URL`
is managed by the dedicated Database section (masked password, tested
connection).

Groups: secrets/DB (`SECRET_KEY`, `DATABASE_URL`, `POSTGRES_*`),
display currency (`CURRENCY_SYMBOL`), sessions
(`SESSION_TIMEOUT_MINUTES`, `SESSION_TYPE`, `SESSION_FILE_DIR`,
`SESSION_COOKIE_*`, `SESSION_CACHE_THRESHOLD`), security
(`RATELIMIT_*`, `TOTP_PENDING_TTL_SECONDS`, password/username policy), UI font
size (`MIN/MAX/DEFAULT_FONT_ZOOM`, `FONT_ZOOM_STEP`), display
(`DATE_FORMAT`, `DATETIME_FORMAT`), import guards
(`CSV_IMPORT_ROW_LIMIT`, `BANK_IMPORT_*`, `RECEIPT_*`), recurring
(`RECURRING_MAX_CATCHUP`), pagination (`PER_PAGE_*`,
`TRANSACTIONS_PER_PAGE_DEFAULT`), backups
(`AUTO_BACKUP`, `BACKUP_DIR`, `BACKUP_KEEP`), audit (`AUDIT_RETENTION_DAYS`),
observability (`SENTRY_DSN`).

---

## 9. Security design

- **AuthN:** username + password (hashed with werkzeug). Optional **TOTP** second
  factor; the password-validated half-session expires after
  `TOTP_PENDING_TTL_SECONDS`.
- **AuthZ:** two roles — `admin` (full) and `viewer` (read-only). Write routes
  use `@admin_required`; the role is re-checked server-side on every request.
- **CSRF:** `flask-wtf.CSGProtect` on all POST/PUT/DELETE forms; `/healthz` and
  `/metrics` are exempt.
- **Rate limiting:** `flask-limiter`, default `RATELIMIT_DEFAULT`
  (200/min); login and 2FA endpoints 5/min. `TESTING` disables it.
- **Sessions:** server-side by default (`flask-session` + cachelib filesystem),
  `HttpOnly`, `SameSite=Lax`, `Secure` via env. Session id rotated on
  login/logout. Idle auto-logout with a live nav countdown (`/session/ping`).
- **Headers:** `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`,
  `Referrer-Policy`, `Permissions-Policy`.
- **SQL safety:** all user values are bound parameters; identifiers come from
  whitelists. The SQL-dump restore only executes literal INSERTs/DELETEs against
  a fixed table set (no expressions, no subqueries, no other statements).
- **Spreadsheet injection:** CSV/XLSX exports prefix `= + - @` values
  (`helpers.safe_sheet_value`).
- **Uploads:** receipts are extension- and size-checked and stored under
  `static/receipts` with a random prefix; path traversal is guarded on delete.

---

## 10. Frontend

- `base.html` — app shell: brand, nav (with role-aware links), theme toggle,
  fullscreen toggle, flash toasts, idle countdown, PWA manifest link, and the
  `<html>` **zoom** attribute applied from the profile font-size setting.
- `_macros.html:zoom_attr(zoom)` — the single source of the page-zoom markup
  (used by `base.html`, `login.html`, `totp.html`). `100%` = browser baseline (no
  attribute); any other value becomes an inline `zoom`.
- `style.css` — one file with `:root` design tokens: a **type scale**
  (`--fs-*`), a **radius scale** (`--radius-*`) and a **spacing scale**
  (`--space-*`). Light/dark themes via `[data-theme="dark"]`. The Profile
  font-size setting scales the whole UI, so component sizes stay consistent.
- `app.js` — theme persistence (OS preference aware), fullscreen, Indian number
  input formatting, client-side form validation, bulk row selection, idle timer.
- Charts are rendered by Chart.js (CDN) and degrade gracefully offline (tables
  always work).

---

## 11. Operations

- **Run:** `python run.py` (dev) or `gunicorn -c gunicorn.conf.py run:app`
  (2 workers, `preload_app`).
- **Docker:** one image; `entrypoint.sh` (as root) starts the in-container
  PostgreSQL, ensures the role/database, loads/generates a persisted
  `SECRET_KEY` (`/data/.secret_key`, mode 0600), defaults `DATABASE_URL`, then
  drops to the unprivileged `appuser` for gunicorn. Dependencies are installed
  at build time into a virtualenv (`/app/.venv`) owned by `appuser`, with pip
  run as that user — never as root — so nothing lands in the system Python.
  Volumes: `/var/lib/postgresql/data` (database) and `/data` (sessions,
  backups).
- **Backups:** `AUTO_BACKUP=1` writes an SQL dump on startup; `jobs._rotate`
  keeps the newest `BACKUP_KEEP`.
- **Observability:** JSON-lines logs to stdout; `GET /healthz` (public) reports
  DB status; `GET /metrics` (Prometheus, when the lib is installed); optional
  Sentry via `SENTRY_DSN`.
- **PWA:** `/manifest.webmanifest` enables "Install app" / standalone display.

---

## 12. Testing & linting

```bash
pip install -r requirements-dev.txt   # runtime deps + pytest + pyflakes
export DATABASE_URL=postgresql://user:password@localhost:5432/exptracker_test
python3 -m pytest -q          # full suite (Postgres required)
python -m pyflakes app/ run.py seed_demo.py seed_bulk.py tests/ gunicorn.conf.py
```

Tests use a real PostgreSQL database and truncate tables between tests. The
only expected pyflakes hits are the intentional side-effect imports in
`app/modules/*/__init__.py` and the `PER_PAGE_CHOICES` re-export.

---

## 13. Extending the app

To add a feature module:

1. `app/modules/<name>/__init__.py` — create a `Blueprint` (optionally a
   `url_prefix`) and import `.controllers` for route registration.
2. `models.py` — data access using `db.execute("… ? …", args)`.
3. `controllers.py` — routes; guard writes with `@admin_required`.
4. `templates/<name>/*.html` — extend `base.html`.
5. Register the blueprint in `app/__init__.py:_register_blueprints`.

Config values belong in `app/config.py` (module constant + `Config` attribute);
never hardcode a tunable literal elsewhere.

---

## 14. Design decisions

- **One container** — simplest self-hosted deployment; no compose, no external
  DB. Postgres data and app files persist on two volumes.
- **MVC per module** — features are isolated; adding/removing one is a
  directory plus a blueprint registration.
- **Integer cents** — avoids floating-point money errors end-to-end.
- **Data-only SQL backups** — portable, human-readable, and safe to validate
  (literal INSERT/DELETE only).
- **Server-side sessions** — enable the login guard, idle timeout and 2FA
  half-session without trusting the client.
- **`.env` over UI edits** — operators get a durable, text-based override store
  (`app/.env`) that admins can edit from the UI; `config.py` stays the single
  source of defaults and logic, and real env vars always win. Secrets are kept
  out of the UI.
- **CSS design tokens + page zoom** — one base drives the whole UI and the
  per-user font-size preference scales everything consistently.
