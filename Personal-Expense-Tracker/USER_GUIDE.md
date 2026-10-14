# User Guide — Personal Expense Tracker

A day-to-day guide to every screen. For the technical design, see
[`ARCHITECTURE.md`](ARCHITECTURE.md).

---

## 1. Signing in

1. Open the app in a browser (default `http://127.0.0.1:5000`).
2. Sign in with the seed account **`admin` / `admin`** (unless changed).
3. **Change the password immediately** — see §13 or ask an admin to reset it.

If two-factor authentication is enabled on your account, you'll be asked for a
6-digit code from your authenticator app after your password.

**Accounts are shared, but you are not signed out of your data:** sessions are
server-side and idle-timed (default **5 minutes**). A live countdown shows in
the top bar; any activity resets it. When it reaches zero you're signed out
automatically.

---

## 2. Roles

| Capability | Admin | Viewer |
|---|:--:|:--:|
| Browse dashboard, transactions, reports, budgets, categories, recurring, data | ✅ | ✅ |
| Add / edit / duplicate / delete transactions, bulk actions | ✅ | — |
| Manage categories, budgets, recurring schedules | ✅ | — |
| Bank import | ✅ | — |
| Backup / restore / CSV import | ✅ | — |
| Manage users, view the audit log | ✅ | — |
| Edit site-wide settings (`app/.env`) | ✅ | — |
| Change **own** font size and set up own 2FA | ✅ | ✅ |
| Change idle timeout | ✅ | — |

Write actions are enforced on the server, not just hidden in the UI.

---

## 3. Navigation

- **Dashboard** — home.
- **Transactions ▾** — All transactions · Recurring · Budgets · Bank import (admin)
- **Reports ▾** — Daily · Monthly · Yearly · Yearly trends · Monthly trends · Top · Categories · Insights · Budgets · Forecast (all views + exports).
- **Master Data ▾** — Categories · Users (admin) · Settings (admin) · Profile
- **Data ▾** — Backup & restore · Audit log (admin)
- **Logout**, a **🌙/☀️ theme toggle**, and a **⛶ fullscreen toggle**.

---

## 4. Dashboard

- **Total balance** — all-time income − spending (savings buckets stay part of
  your money).
- **Month income / expense / saved**, savings rate, today's figures.
- **Budgets** — count and how many are over/near their limit.
- **Add transaction** form — the quickest way to log a spend or income (§5).
- **Last 6 months** chart — income vs expense bars with a savings line.
- **Recent activity** and **top spending categories** for the current month.
- Budget alerts appear at the top when a category is near (⚠️) or over (🚨) budget.

The dashboard also **runs due recurring schedules** automatically each time it
loads, so newly due items appear without any manual step.

---

## 5. Transactions

### 5.1 Add a transaction
Use the dashboard form or the **All transactions** page:

1. **Amount** — type normally (e.g. `1,00,000.00`); it's formatted for you.
2. **Expense / Income** — toggles which categories are offered.
3. **Category** — required; must match the type.
4. **Date** — defaults to today; future dates are rejected.
5. **Note** (optional), **For** (which member the entry is managed for),
   **Receipt** (image or PDF, size-limited).
6. **Split across categories** (optional) — spread one expense over up to three
   extra categories; the remainder stays on the main category. All lines are
   linked as a group.

### 5.2 Find and manage
- **Filters:** type, category, member, month (dropdown), and full-text note
  search.
- **Sort:** by date, amount or category; ascending/descending.
- **Pagination:** choose rows per page (default 10 for transactions).
- **Row actions:** edit, duplicate (copies with today's date), delete (→ Trash).
- **Bulk actions:** tick rows, then **delete** (with confirmation) or
  **re-categorize** the selection.
- **Split groups:** open a split to see all its lines (a group can be deleted as
  a whole).

### 5.3 Trash
Deleting moves a transaction to **Trash** (soft delete). From there you can
**restore** it or **purge** it permanently. Purging also deletes the receipt
file. A transaction can't be restored if its category was deleted meanwhile —
recreate the category or purge the row.

### 5.4 Export / import CSV
- **Export CSV** respects the current filters.
- **Import CSV** (admin) is **additive** — it never deletes. Required columns
  `date,type,category,amount`; optional `note,owner`. Unknown categories
  are created automatically; a row naming an unknown owner is skipped.

---

## 6. Recurring transactions

Schedule rent, salary, subscriptions, etc.

- **Frequency:** daily / weekly / monthly / yearly.
- **Start date** (required) and **optional end date**; monthly/yearly schedules
  keep the original day of month (Jan 31 → Feb 28 → Mar 31).
- **Pause / Resume / Delete**. Resuming an overdue schedule starts the next run
  from tomorrow (so it doesn't instantly backfill a burst). Deleting a schedule
  keeps the transactions it already generated.
- **Run now** generates anything currently due.
- A schedule past its end date shows as **ended** and stops running.

Very old start dates (over a year ago) begin their first run from today instead
of backfilling. If a schedule falls far behind, generation is capped for safety
and the schedule is paused (a warning explains this).

---

## 7. Budgets

Set a **monthly limit per expense category**.

- The page lists each budget with **spent**, **remaining**, **% used**, and a
  status: **on track**, **near** (≥ 80%), or **over**.
- Enter a limit to set/update it; clear the field to remove it.
- Over/near budgets raise alerts on the dashboard.
- **Savings buckets can't have budgets** (budgets apply to spending only).

Use the month picker to view any month.

---

## 8. Categories

- **Add** a category (name + income/expense). Duplicate names are rejected.
- **Rename** or **delete** a category. A category used by live transactions
  can't be deleted.
- **Savings bucket:** flag an **expense** category as **🏦 savings**. Money
  parked there is excluded from spending totals and the savings rate and shown
  as *Saved* instead — everywhere (dashboard, reports, API, CSV totals).
  Setting a category as savings removes any budget on it.
- Filter by type, savings/regular, used/unused, and search by name.

A `Savings` bucket is seeded by default.

---

## 9. Reports

Open **Reports** and pick a view from the tab strip. Every view supports
**⬇ PDF** and **⬇ Excel**; the CSV button exports transactions.

| View | What it shows |
|---|---|
| **📅 Daily** | One row per day for a month; income/expense/savings, trend bars, category breakdown, charts. |
| **🗓️ Monthly** | One row per month for a year; same metrics, year charts. |
| **📆 Yearly** | Year totals, **year-over-year growth** per month, and a 12-month savings-rate trend. |
| **📈 Trends** | Split into two tabs: **Yearly trends** (year-by-year totals with growth vs previous year) and **Monthly trends** (36-month income-vs-expense momentum) — across all years. |
| **🏆 Top** | Largest single income/expense entries, plus per-category entries, average ticket and share. |
| **🏷️ Categories** | A category × month matrix with yearly total, average per month and best month. |
| **💡 Insights** | Weekday spending pattern, spend **per member**, savings-vs-spending mix, and current recurring commitments. |
| **🎯 Budgets** | Budget-vs-actual variance for a month. |
| **🔮 Forecast** | A 3-month cash-flow projection (average of the last 3 complete months) plus the current balance and recurring commitments. |

Charts load from a CDN (they need internet); the tables always work offline.

---

## 10. Bank import (admin)

Import a bank/credit-card CSV with a guided flow:

1. **Upload** the file (or start from a saved template).
2. **Map columns** — the app auto-guesses date, amount (or debit/credit),
   type, description and note columns; adjust the dropdowns as needed and,
   optionally, choose a **default category**.
3. **Preview** — every row is validated and shown; nothing is written yet.
   Fix mapping issues and re-preview.
4. **Import** — rows are created, with duplicate protection (an identical
   transaction already present is skipped; repeats inside the same upload are
   kept).

Save the mapping as a **named template** to reuse next month.

---

## 11. Data — backup, restore & bulk import (admin)

The **Data** page is split into numbered sections:

1. **Backup (download, safe)** — a full **SQL dump** (`.sql`) of categories,
   budgets, transactions, users and recurring schedules. A legacy `.json`
   download is also available.
2. **Restore from a backup file (destructive)** — upload a `.sql` (or legacy
   `.json`) file to **replace all data**. Guarded by a confirmation checkbox and
   validated first: an invalid file changes nothing. Sequence counters are reset.
3. **CSV files — Add or Replace** — upload categories/budgets/transactions CSVs:
   - **Add** appends rows (never deletes).
   - **Replace** wipes and re-imports those tables atomically (a mid-restore
     failure rolls back). Users are always preserved; CSV has no recurring
     format, so recurring schedules are removed — use a `.sql` backup for a
     full-fidelity restore.

Live row counts (categories, budgets, transactions, users, recurring) are shown
so you know what a backup will contain.

**CSV formats:**
- *categories:* `name,type`
- *budgets:* `category,monthly_limit`
- *transactions:* `date,type,category,amount` (+ optional `note,owner`)

---

## 12. Trash

See §5.3. Trash is reachable from the transactions area and lists soft-deleted
rows with restore/purge actions.

---

## 13. Profile

Open **Master Data ▾ → Profile**.

- **🔠 Font size** — a slider (70–160%, step 5) that scales the **whole app**.
  The slider previews live; **Save size** keeps it for every page, **Reset**
  returns to the site default (**90%**). Your choice is saved to your account.
- **Two-factor authentication** — set up TOTP: scan/enter the shown secret into
  an authenticator app, then confirm with a code. Requires a code to disable.
- **Inactivity timeout** (admins) — a personal idle-logout override in minutes;
  blank uses the site default.
- **Expenses you manage** — totals and recent transactions attributed to you.

Changing the password: an admin resets it from **Users** (§14). If you're the
only admin, reset it directly in the database (see the README quickstart).

---

## 14. Users (admin)

**Master Data ▾ → Users**

- **Add** an account (username, password, role admin/viewer).
- **Reset password** for a user. Passwords need **8+ characters with letters and
  digits**; usernames are 3–32 characters.
- **Change role**. You can't change your own role, and the last admin is
  protected.
- **Delete** an account (not your own, not the last one). Click a user to see
  the **expense profile** — totals and recent entries managed for them.

---

## 15. Settings (admin)

**Master Data ▾ → Settings** edits a safe, **non-secret** subset of the app's
`.env` configuration — currency, display formats, session/account policy,
pagination, import limits, rate limit and backup settings.

- Values are validated on save (whole numbers within bounds, allowed choices,
  non-empty text), and each change is recorded in the audit log.
- Saving writes them to `app/.env`, preserving the file's comments and any keys
  added by hand. Secret and infrastructure keys (`SECRET_KEY`, `DATABASE_URL`,
  database credentials, paths) are **not** shown here — set those with
  environment variables.
- **Restart the app** for changes to take effect: configuration is read once at
  startup. Real environment variables always override values in `app/.env`.
  Use the **↻ Restart app** button (top-right) to apply saved changes without
  touching the terminal — it reloads the server in place (sessions are kept).

**Database** — choose **Local** (the default: the app's built-in database) or
**Remote PostgreSQL** (host, port, database, user, password). The password is
never shown back — leave it blank
to keep the stored one. **Test connection** checks reachability without
saving; **Save** tests first and refuses an unreachable database, so a bad
URL can never break the next restart. Switching back to Local removes
`DATABASE_URL` from the file.

---

## 16. Audit log (admin)

**Data ▾ → Audit log** records deletes, password resets, imports, role changes,
2FA events and restores, with actor and details. Use **Purge** to delete entries
older than a chosen number of days (the site may also auto-purge on startup).

---

## 17. UI extras

- **🌙 / ☀️ Theme** — toggles light/dark; the choice is remembered and defaults
  to your OS preference.
- **⛶ Fullscreen** — hides the browser address bar (Esc to exit).
- **Install as an app (PWA)** — use your browser's *Install app* / *Add to Home
  screen* for a standalone window.
- **Session countdown** — the ⏱ pill in the top bar warns before auto-logout.

---

## 18. Tips & troubleshooting

- **Money formatting** uses Indian grouping (e.g. `₹12,34,567.89`).
- **Amounts are stored exactly** (integer paise) — no rounding drift.
- **A category can't be deleted** while live transactions use it; re-categorize
  or delete those first (trashed rows don't block deletion).
- **Split** amounts must be less than the main amount; the remainder stays on the
  main category.
- **Recurring didn't generate?** Check the schedule is active and started on/before
  today; use **Run now** on the Recurring page.
- **Import skipped rows?** Usually a bad date/amount, an unknown owner, or a
  future date. The flash message shows the first issues.
- **Restore did nothing?** It validates first — an invalid file leaves existing
  data untouched; read the error message.
- **Charts blank?** Chart.js is loaded from a CDN; check your internet. Tables
  and exports still work.
- **Signed out too quickly?** Increase your idle timeout in Profile (admins).

---

## 19. JSON API

Read-only JSON for scripts/dashboards (requires a logged-in session):

- `GET /api/v1/summary?month=YYYY-MM` or `?year=YYYY` — period totals and
  per-category breakdown.
- `GET /api/v1/transactions?limit=50&type=expense|income` — latest transactions.

Legacy unversioned endpoints `/api/summary` and `/api/transactions` behave
identically.

---

## 20. Quick reference

| I want to… | Go to |
|---|---|
| Log a purchase or income | Dashboard **Add transaction** |
| Find/filter/edit entries | **Transactions** |
| Schedule rent/salary | **Transactions ▾ → Recurring** |
| Set a monthly cap | **Transactions ▾ → Budgets** |
| Rename/add categories or mark savings | **Master Data ▾ → Categories** |
| See charts & forecasts | **Reports** |
| Import a bank statement | **Transactions ▾ → Bank import** |
| Back up or restore | **Data ▾ → Backup & restore** |
| Change font size or enable 2FA | **Master Data ▾ → Profile** |
| Add users / reset a password | **Master Data ▾ → Users** |
| Change site-wide defaults (currency, formats, limits) | **Master Data ▾ → Settings** |
| See who did what | **Data ▾ → Audit log** |
