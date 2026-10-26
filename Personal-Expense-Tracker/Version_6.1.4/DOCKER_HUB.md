# Pull from Docker Hub

Prebuilt image: `vipulverulkar/personal-expense-tracker-v6.1:latest` (or version tag `:6.1`)

One container runs both the app and PostgreSQL — no compose setup needed.

![Dashboard](screenshots/dashboard.png)

## Artifacts & downloads

- **GitHub Releases:** [Releases Page](https://github.com/vipulverulkar/Exp_tracker_V6.1_with_docker_postgres_and_sqllite/releases) — download prebuilt standalone executables and docker archives.
- **Docker image:** [`vipulverulkar/personal-expense-tracker-v6.1`](https://hub.docker.com/r/vipulverulkar/personal-expense-tracker-v6.1) on Docker Hub.
- **Desktop builds (Linux / Windows):** [CI/CD workflow artifacts](https://github.com/vipulverulkar/Exp_tracker_V6.1_with_docker_postgres_and_sqllite/actions/workflows/ci_cd.yml) — download `Personal-Expense-Tracker_v6.1.4_linux` or `Personal-Expense-Tracker_v6.1.4_windows`.
- **Source code:** [GitHub repository](https://github.com/vipulverulkar/Exp_tracker_V6.1_with_docker_postgres_and_sqllite).

## Pull

```bash
docker pull vipulverulkar/personal-expense-tracker-v6.1:latest
# or pinned version:
# docker pull vipulverulkar/personal-expense-tracker-v6.1:6.1
```

## Create `.env` file

No `.env` is strictly required — the image runs with defaults and
auto-generates `SECRET_KEY` on first launch. Docker's `--env-file` flag does
need the file to exist, so either create one, use an empty `touch .env`, or
omit `--env-file .env` entirely. For production, create one:

**Linux / macOS (bash):**
```bash
# minimal .env — recommended for production:
cat > .env <<'EOF'
SECRET_KEY=change-me-to-a-long-random-string
POSTGRES_PASSWORD=change-me-to-a-strong-db-password
EOF

# optional: start from the full template if you cloned the repo:
# cp app/.env.example .env
```

**Windows (PowerShell):**
```powershell
@"
SECRET_KEY=change-me-to-a-long-random-string
POSTGRES_PASSWORD=change-me-to-a-strong-db-password
"@ | Out-File -Encoding utf8 .env
```

Easiest on Windows: open Notepad, paste those 2 lines, save as `.env` in the folder where you run `docker run`.

Generate a strong key with:
```bash
python3 -c "import secrets; print(secrets.token_hex(32))"
```

The `docker run --env-file .env` commands below expect `.env` in the
current directory. You can also pass vars inline with `-e KEY=value`
instead of a file. (The app also reads its own `app/.env` at startup, and
admins can edit a safe subset of those settings from the UI — mount
`-v myenv:/app/app/.env` or set `ENV_FILE=/data/.env` to persist them.)

## Environment variables

| Variable | Required? | Default | Description |
|---|---|---|---|
| `SECRET_KEY` | Recommended in prod | auto-generated to `/data/.secret_key` | Flask session signing key. Explicit env always wins. |
| `POSTGRES_USER` / `POSTGRES_DB` | No | `exptracker` | In-container Postgres role / database. |
| `POSTGRES_PASSWORD` | Recommended | `exptracker` | In-container Postgres password. Change it. |
| `DATABASE_URL` | No | in-container server | Set only for an external DB, e.g. `postgresql://user:pass@host:5432/exptracker`. |
| `PORT` | No | `5000` | App HTTP port (match `-p` mapping). |
| `CURRENCY_SYMBOL` | No | `₹` | Display symbol in the UI and CSV. |
| `FINANCIAL_YEAR_START_MONTH` | No | `4` | Financial year start month (4 = Indian FY: 1 Apr – 31 Mar default; 1 = Calendar Year). |
| `DATE_FORMAT` / `DATETIME_FORMAT` | No | `%d/%m/%Y` / `%d/%m/%Y %H:%M:%S` | Date-only and timestamp display formats (Python strftime). |
| `SESSION_TIMEOUT_MINUTES` | No | `5` | Idle auto-logout minutes. |
| `DEFAULT_FONT_ZOOM` (`MIN_FONT_ZOOM`/`MAX_FONT_ZOOM`) | No | `90` (`70`/`160`) | UI font size (%) applied to every page; each user can override it on Profile. |
| `AUTO_BACKUP` / `BACKUP_KEEP` | No | off / `10` | Startup SQL-dump backup + rotation. |
| `ENV_FILE` | No | `app/.env` | Config file to read at startup (e.g. mount `/data/.env` to persist UI settings). |
| `MIGRATE_FROM_DATABASE_URL` | No | unset | Source PostgreSQL DSN to automatically migrate data from at first startup. |
| `SENTRY_DSN` | No | unset | Enables Sentry when set. |

Full list with all tunables: see `app/.env.example` in the repo.

## Run

**Linux / macOS (bash) — needs no `.env` file:**

```bash
docker run -d -p 5000:5000 \
  -v exptracker-pg:/var/lib/postgresql/data \
  -v exptracker-data:/data \
  --name personal-expense-tracker-v6.1 vipulverulkar/personal-expense-tracker-v6.1:latest
# open http://127.0.0.1:5000
```

**Windows (CMD or PowerShell — single line, copy-paste as-is):**

```powershell
docker run -d -p 5000:5000 -v exptracker-pg:/var/lib/postgresql/data -v exptracker-data:/data --name personal-expense-tracker-v6.1 vipulverulkar/personal-expense-tracker-v6.1:latest
```

## First login

Open `http://localhost:5000` (or `http://127.0.0.1:5000`).

- Default account: **`admin` / `admin`** (seeded on first boot).
- **Change the password immediately** from **Master Data ▾ → Users**.

## Where data lives

| Data | Container path | Where to mount |
|---|---|---|
| PostgreSQL database | `/var/lib/postgresql/data` | Named volume `exptracker-pg` (don't bind-mount a Windows folder — Postgres needs POSIX permissions) |
| App data (sessions, backups, receipts, secret key) | `/data` | Named volume `exptracker-data` |
| UI settings file (optional) | `/data/.env` | Set `ENV_FILE=/data/.env` to persist changes made in **Master Data → Settings** |

## Seed demo data

To explore the app with realistic transactions, budgets, accounts, goals, and debts:

```bash
docker exec -it personal-expense-tracker-v6.1 python3 seed_demo.py
```

## Stop, start, update

```bash
# stop:
docker stop personal-expense-tracker-v6.1

# start again:
docker start personal-expense-tracker-v6.1

# update to latest (keeps your data because volumes are kept):
docker pull vipulverulkar/personal-expense-tracker-v6.1:latest
docker stop personal-expense-tracker-v6.1
docker rm personal-expense-tracker-v6.1
docker run -d -p 5000:5000 \
  -v exptracker-pg:/var/lib/postgresql/data \
  -v exptracker-data:/data \
  --name personal-expense-tracker-v6.1 vipulverulkar/personal-expense-tracker-v6.1:latest
```

## Backup & restore

The container includes a full **Backup & Restore** UI at **Data ▾ → Backup & restore** (download `.sql` dumps, full-schema dumps, all-tables CSV backup, one-click restore, and ledger integrity verification via `🛡 Integrity Check`).

To take raw database dumps from outside:

```bash
# backup:
docker exec personal-expense-tracker-v6.1 pg_dump -U exptracker exptracker > backup.sql

# restore:
cat backup.sql | docker exec -i personal-expense-tracker-v6.1 psql -U exptracker exptracker
```

## Screenshots

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

---

## License

This project is free software: you can redistribute it and/or modify it under
the terms of the **GNU General Public License, version 3 or later
(GPL-3.0-or-later)**, as published by the Free Software Foundation.
