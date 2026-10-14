# Pull from Docker Hub

Prebuilt image: `vipulverulkar/personal-expense-tracker:latest`

One container runs both the app and PostgreSQL — no compose setup needed.

![Dashboard](screenshots/dashboard.png)

## Pull

```bash
docker pull vipulverulkar/personal-expense-tracker:latest
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
| `DATE_FORMAT` / `DATETIME_FORMAT` | No | `%d/%m/%Y` / `%d/%m/%Y %H:%M:%S` | Date-only and timestamp display formats (Python strftime). |
| `SESSION_TIMEOUT_MINUTES` | No | `5` | Idle auto-logout minutes. |
| `DEFAULT_FONT_ZOOM` (`MIN_FONT_ZOOM`/`MAX_FONT_ZOOM`) | No | `90` (`70`/`160`) | UI font size (%) applied to every page; each user can override it on Profile. |
| `AUTO_BACKUP` / `BACKUP_KEEP` | No | off / `10` | Startup SQL-dump backup + rotation. |
| `SENTRY_DSN` | No | unset | Enables Sentry when set. |

Full list with all tunables: see `app/.env.example` in the repo.

## Run

**Linux / macOS (bash) — needs no `.env` file:**

```bash
docker run -d -p 5000:5000 \
  -v exptracker-pg:/var/lib/postgresql/data \
  -v exptracker-data:/data \
  --name personal-expense-tracker vipulverulkar/personal-expense-tracker:latest
# open http://127.0.0.1:5000
```

**Windows (CMD or PowerShell — single line, copy-paste as-is):**

```powershell
docker run -d -p 5000:5000 -v exptracker-pg:/var/lib/postgresql/data -v exptracker-data:/data --name personal-expense-tracker vipulverulkar/personal-expense-tracker:latest
```

**With a `.env` file — optional**, add `--env-file .env` (Docker requires that
file to exist, so create it first with `touch .env` or `cp app/.env.example .env`):

Linux / macOS:
```bash
docker run -d -p 5000:5000 \
  -v exptracker-pg:/var/lib/postgresql/data \
  -v exptracker-data:/data \
  --env-file .env --name personal-expense-tracker vipulverulkar/personal-expense-tracker:latest
```

Windows:
```powershell
docker run -d -p 5000:5000 -v exptracker-pg:/var/lib/postgresql/data -v exptracker-data:/data --env-file .env --name personal-expense-tracker vipulverulkar/personal-expense-tracker:latest
```

Then open http://127.0.0.1:5000

Default login is `admin / admin` on first boot (change it before exposing).

## Notes

- Postgres data persists in the `exptracker-pg` volume and app files
  (sessions, backups) in `exptracker-data` — removing and
  re-creating the container with the same volumes keeps everything. Only
  deleting the volumes (`docker volume rm exptracker-pg exptracker-data`)
  deletes data.
- `SECRET_KEY` may be omitted: on first launch the entrypoint generates a
  random one and stores it at `/data/.secret_key` (mode `0600`), so it
  survives restarts via the `exptracker-data` volume. Set `SECRET_KEY`
  explicitly to take full control (explicit env always wins).
- Keep named volumes. Do not bind-mount a Windows folder to
  `/var/lib/postgresql/data` (Postgres needs POSIX permissions).
- To use an external database instead of the in-container server:
  `-e DATABASE_URL=postgresql://user:password@host:5432/exptracker`

## Update

**Linux / macOS (bash):**

```bash
docker pull vipulverulkar/personal-expense-tracker:latest
docker rm -f personal-expense-tracker
docker run -d -p 5000:5000 \
  -v exptracker-pg:/var/lib/postgresql/data \
  -v exptracker-data:/data \
  --env-file .env --name personal-expense-tracker vipulverulkar/personal-expense-tracker:latest
```

**Windows (CMD or PowerShell):**

```powershell
docker pull vipulverulkar/personal-expense-tracker:latest
docker rm -f personal-expense-tracker
docker run -d -p 5000:5000 -v exptracker-pg:/var/lib/postgresql/data -v exptracker-data:/data --env-file .env --name personal-expense-tracker vipulverulkar/personal-expense-tracker:latest
```

## Screenshots

Captured with the demo/bulk seeders:

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


