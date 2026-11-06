# System Architecture & Technical Specification

**Personal Expense Tracker — Android Mobile App**

This document describes the architectural principles, component relationships, data flow pipelines, database schemas, security design, and technical specifications for the Personal Expense Tracker Android application (Database v4).

---

## 🏛️ Architecture Overview

The application strictly follows **Google's Modern Android Development (MAD)** guidelines and principles of **Clean Architecture**. It is structured into three primary layers: **Presentation**, **Domain**, and **Data**.

```
+-----------------------------------------------------------------------------------+
|                                PRESENTATION LAYER                                 |
|  [MainActivity] -> [AppNavHost] (Navigation Compose)                              |
|  Composables: DashboardScreen, TransactionsScreen, AccountsScreen,               |
|               CategoriesScreen, BudgetsScreen, AnalyticsScreen,                  |
|               AdvanceScreen, SettingsScreen, DuplicateCleanupScreen              |
|  Theme Engine: AppThemePresets (Indigo, Emerald, Ocean, Rose, Midnight)           |
|  State Management: ViewModel + StateFlow (Unidirectional Data Flow)               |
+-----------------------------------------------------------------------------------+
                                          |
                                          v
+-----------------------------------------------------------------------------------+
|                                   DOMAIN LAYER                                    |
|  Utils: CurrencyFormatter (Multi-Currency), DateUtils (Sanitizer)                 |
|  SMS Engine: SmsParser, SmsReaderUtil, SmsReceiver (Local RegEx)                  |
|  Bank Import: BankCsvParser, Import Templates Engine                              |
|  OCR Scanning: ReceiptParser Engine                                               |
|  Duplicates Engine: DuplicateDetector (Exact & Date/Amount Matching)             |
|  Accounting Engine: Balance Sheet, P&L, Cash Flow, Trial Balance, Tax Summary     |
|  Integrity Engine: Transaction Integrity & Parity Audit                           |
+-----------------------------------------------------------------------------------+
                                          |
                                          v
+-----------------------------------------------------------------------------------+
|                                    DATA LAYER                                     |
|  Single Source of Truth: ExpenseRepository / ExpenseRepositoryImpl                |
|  Serialization & Backups: CSV, JSON & Raw SQL Dump Serialization / Deserialization|
|  Local Database: AppDatabase (Room 2.7.0 SQLite ORM, Database v4)                 |
|  Entities: CategoryEntity, TransactionEntity, BudgetEntity,                       |
|            RecurringTransactionEntity, AccountEntity, GoalEntity,                 |
|            RuleEntity, DebtEntity, SharedExpenseEntity,                           |
|            NotificationEntity, ImportTemplateEntity                               |
|  DAOs: CategoryDao, TransactionDao, BudgetDao, RecurringTransactionDao,           |
|        AccountDao, GoalDao, RuleDao, DebtDao, SharedExpenseDao,                   |
|        NotificationDao, ImportTemplateDao                                         |
+-----------------------------------------------------------------------------------+
```

---

## 🔒 Security, Hardcoding & Vulnerability Audit

1. **Zero Hardcoded Credentials or API Keys**:
   - The application codebase has been thoroughly audited: zero hardcoded passwords, tokens, API keys, or cloud endpoints exist in source code or resources.
2. **Parameterized SQL Queries**:
   - All Room DAOs use parameterized compile-time validated SQL queries (`@Query`, `@Insert`, `@Update`, `@Delete`), completely immune to SQL Injection attacks.
3. **SQLite Integer Overflow Protection**:
   - All aggregate queries (`IncomeSum`, `ExpenseSum`, `SavedSum`, `TotalNetWorth`) use SQLite's `TOTAL(...)` 64-bit floating point function. SQLite's `TOTAL()` function never raises an integer overflow exception under large amounts.
4. **Offline Processing & Local Privacy**:
   - Bank SMS alerts, OCR receipt parsing, and bank statement text/CSV files are processed 100% on-device in local memory thread contexts. No financial data, SMS content, or telemetry is ever transmitted over the network.
5. **Scoped Storage Access Framework (SAF)**:
   - Storage exports and imports use Android's native Storage Access Framework (`CreateDocument` and `GetContent`), which operate on scoped, user-granted document URIs rather than direct filesystem paths.

---

## 🎯 Key Architectural Principles

1. **100% Offline-First & Local-First**: All application data is stored locally on the device using an embedded **Room (SQLite)** database. No network connection is required.
2. **Unidirectional Data Flow (UDF)**:
   - State flows **downward** from ViewModels to Composable screens via immutable `StateFlow<UiState>`.
   - User events flow **upward** from Composables to ViewModels as function calls.
3. **Reactive Streams**: Database changes emitted by Room DAOs via Kotlin `Flow` automatically update repository streams, triggering UI re-composition in real time.
4. **Single Source of Truth (SSOT)**: The `ExpenseRepository` acts as the single gateway for all data access across the app.
5. **Integer Cents / Paise Precision**: Money values are stored as 64-bit integers (`Long` in paise/cents) avoiding floating-point rounding discrepancies.
6. **Dynamic Theming & Multi-Currency Engine**: `AppThemePreset`, `CurrencyFormatter.activeCurrencySymbol`, and `DateUtils.activePattern` state updates trigger instant app-wide UI recomposition.

---

## 📂 Project Structure

```
mobile_app/app/src/main/java/com/expensetracker/mobile/
│
├── ExpenseTrackerApp.kt       # Application class (Database & Repository Singleton)
├── MainActivity.kt            # Single Activity entry point with Edge-to-Edge support
│
├── data/                      # Data Layer
│   ├── local/
│   │   ├── AppDatabase.kt     # Room database (Database v4) with auto-seeding
│   │   ├── dao/               # Room Data Access Objects (TOTAL aggregate queries)
│   │   │   ├── CategoryDao.kt
│   │   │   ├── TransactionDao.kt
│   │   │   ├── BudgetDao.kt
│   │   │   ├── RecurringTransactionDao.kt
│   │   │   ├── AccountDao.kt
│   │   │   ├── GoalDao.kt
│   │   │   ├── RuleDao.kt
│   │   │   ├── DebtDao.kt
│   │   │   ├── SharedExpenseDao.kt
│   │   │   ├── NotificationDao.kt
│   │   │   └── ImportTemplateDao.kt
│   │   ├── entity/            # Room Entity Data Classes
│   │   │   ├── CategoryEntity.kt
│   │   │   ├── TransactionEntity.kt
│   │   │   ├── BudgetEntity.kt
│   │   │   ├── RecurringTransactionEntity.kt
│   │   │   ├── AccountEntity.kt
│   │   │   ├── GoalEntity.kt
│   │   │   ├── RuleEntity.kt
│   │   │   ├── DebtEntity.kt
│   │   │   ├── SharedExpenseEntity.kt
│   │   │   ├── NotificationEntity.kt
│   │   │   └── ImportTemplateEntity.kt
│   │   └── model/             # Composite Room join models (e.g. TransactionWithDetails)
│   └── repository/            # Clean Repository Pattern
│       ├── ExpenseRepository.kt     # Repository interface
│       └── ExpenseRepositoryImpl.kt # Repository implementation (CSV, JSON & SQL export/import)
│
├── domain/                    # Domain Layer
│   ├── sms/                   # Bank SMS Auto-Pick Engine
│   │   ├── ParsedSmsTransaction.kt
│   │   ├── SmsParser.kt       # Bank SMS RegEx Parser (HDFC, SBI, ICICI, Axis, Paytm, UPI)
│   │   ├── SmsReaderUtil.kt   # Inbox ContentResolver reader
│   │   └── SmsReceiver.kt     # BroadcastReceiver for real-time incoming SMS
│   └── util/
│       ├── BankCsvParser.kt   # Multi-format Fixed-Width & Delimited CSV parser
│       ├── CurrencyFormatter.kt # Dynamic Multi-Currency formatting (₹, $, €, £, ¥, A$)
│       ├── DateUtils.kt       # Configurable date formats with 2-digit year sanitizer
│       └── ReceiptParser.kt   # OCR receipt parser helper
│
└── ui/                        # Presentation Layer
    ├── components/            # Reusable UI Composable Widgets
    │   ├── CategoryBadge.kt
    │   ├── EmptyState.kt
    │   ├── MonthSelector.kt
    │   ├── StatCard.kt
    │   └── TransactionItem.kt
    ├── navigation/            # Navigation Compose definitions
    │   ├── AppNavHost.kt      # Main NavHost with single-line aligned bottom navigation
    │   └── Screen.kt          # Navigation destinations & bottom bar items
    ├── theme/                 # Material Design 3 Styling & Theme Engine
    │   ├── Color.kt           # Color palette & AppThemePresets (Indigo, Emerald, Ocean, Rose, Midnight Gold)
    │   ├── Theme.kt           # Material 3 Dynamic Theme setup
    │   └── Type.kt            # Typography definitions
    └── screens/               # Feature Screens & ViewModels
        ├── dashboard/         # DashboardScreen, DashboardViewModel
        ├── transactions/      # TransactionsScreen, TransactionsViewModel, AddEditTransactionSheet, BankImportDialog
        ├── accounts/          # AccountsScreen, AccountsViewModel, AddAccountDialog, TransferDialog
        ├── categories/        # CategoriesScreen, CategoriesViewModel, AddCategoryDialog
        ├── budgets/           # BudgetsScreen, BudgetsViewModel, AddBudgetDialog
        ├── analytics/         # AnalyticsScreen, AnalyticsViewModel
        ├── advance/           # AdvanceScreen, AdvanceViewModel, AddRecurringDialog, RestoreBackupDialog
        └── settings/          # SettingsScreen, SettingsViewModel, Theme Presets, Date Format, CSV/SQL Backup, Data Clear
```

---

## 🛠️ Layer Detailed Specs

### 1. Data Layer (`data/`)

#### Room SQLite Entities (Database v4)
- **`CategoryEntity`** (`categories`): Represents Income and Expense categories (`id`, `name`, `type` ["income"/"expense"], `is_savings` [0/1], `color`, `icon`).
- **`AccountEntity`** (`accounts`): Multi-account management (`id`, `name`, `type` ["Checking", "Savings", "Credit Card", "Cash", "Investment", "Loan"], `balance` [in paise], `account_number`, `institution`, `active`, `created_at`).
- **`TransactionEntity`** (`transactions`): Core General Ledger transactions (`id`, `amount` [in paise], `type`, `category_id`, `account_id`, `date`, `note`, `tags`, `is_tax_deductible`, `split_group`, `created_at`, `deleted_at`, `receipt_path`).
- **`BudgetEntity`** (`budgets`): Monthly expense limits per category (`id`, `category_id`, `monthly_limit`, `rollover`).
- **`RecurringTransactionEntity`** (`recurring_transactions`): Scheduled recurring transactions (`id`, `amount`, `type`, `category_id`, `frequency` ["daily", "weekly", "monthly", "yearly"], `note`, `start_date`, `next_due_date`, `end_date`, `active`).
- **`GoalEntity`** (`goals`): Sinking fund targets (`id`, `name`, `target_amount`, `current_amount`, `target_date`, `category_id`, `notes`, `status` ["active", "achieved", "archived"]).
- **`RuleEntity`** (`transaction_rules`): Automated merchant & note transaction rules engine (`id`, `pattern`, `match_field` ["note", "merchant"], `category_id`, `tags`, `active`, `created_at`).
- **`DebtEntity`** (`debts`): Debt payoff & acceleration planner (`id`, `name`, `principal`, `interest_rate`, `min_payment`, `due_day`, `notes`, `recurring_id`, `created_at`).
- **`SharedExpenseEntity`** (`shared_expenses`): Group expense splitting (`id`, `transaction_id`, `person_name`, `share_amount`, `is_settled`, `settled_at`).
- **`NotificationEntity`** (`notifications`): Proactive in-app alert notifications (`id`, `type`, `title`, `message`, `link`, `is_read`, `created_at`).
- **`ImportTemplateEntity`** (`import_templates`): Saved bank statement CSV column mapping templates (`id`, `name`, `mapping`, `created_at`).

---

## 🔄 Synchronized Planning Framework & Domain Modules

The Android application implements synchronized financial planning centered on the core `transactions` table:

```
                             ┌───────────────────────────────┐
                             │       ACCOUNTS MODULE         │
                             │  (Checking, Savings, Cash)    │
                             └──────────────┬────────────────┘
                                            │ 1 : N (account_id)
                                            ▼
┌─────────────────────────┐          ┌──────────────┐          ┌───────────────────────────┐
│     RULES ENGINE        │─────────►│ transactions ├─────────►│      SHARED EXPENSES      │
│ (RegEx pattern matching │  Auto    │  (Core GL)   │  1 : N   │ (Split lines attached to  │
│  note/merchant to cat)  │ Categorize└──▲───▲────▲──┘ (tx_id)  │  transactions + settles)  │
└─────────────────────────┘             │   │    │             └───────────────────────────┘
                                        │   │    │
          ┌─────────────────────────────┘   │    └─────────────────────────────┐
          │ Inserts expense on              │ Inserts expense on               │ Generates income on
          │ goal contribution               │ debt repayment                   │ settlement
          ▼                                 ▼                                  ▼
┌─────────────────────────┐   ┌─────────────────────────┐   ┌───────────────────────────┐
│      GOALS MODULE       │   │      DEBTS MODULE       │   │    SHARED EXPENSES        │
│ (Sinking Funds targets) │   │ (Loan payoff simulator) │   │ (Participant settlements) │
└─────────────────────────┘   └─────────────────────────┘   └───────────────────────────┘
```

### Module Interactions & Financial Rules:

1. **Multi-Account & Net Worth (`AccountDao` / `ExpenseRepository`)**:
   - Automatic Account Binding & Balance Sync: Transactions automatically link to the user's primary active account if unspecified, with a seamless dropdown in the UI. Adding, updating, or deleting transactions automatically updates the account's running balance in real time. Legacy unassigned transactions are automatically backfilled to the primary account.
   - Dynamic ledger balance calculation: $\text{Balance} = \text{Initial Balance} + \sum (\text{Income}) - \sum (\text{Expense})$.
   - Paired Inter-Account Transfers: Coordinated UUID `split_group` prefix `tr-*` creating an `expense` leg on origin account and `income` leg on destination account.
2. **Inter-Category Reclassifications**:
   - Reclassifies allocations using `rc-*` UUID split groups with signed netting guarantees:
     $$\text{Net Spent} = \sum (\text{CASE WHEN type = 'expense' THEN amount ELSE -amount END})$$
   - Category adjustments do not distort total net operating surplus.
3. **Automated Transaction Rules (`RuleDao`)**:
   - Compiles RegEx or substring patterns against transaction notes or merchant names to auto-categorize and tag transactions on insertion.
4. **Debt Payoff & Acceleration (`DebtDao`)**:
   - Avalanche (highest interest rate first) and Snowball (smallest principal first) payoff simulations.
   - Recording a payment decrements debt principal and generates an `expense` transaction under savings category (`is_savings = 1`), reducing liquid balance without distorting operating P&L.
5. **Group Expense Splitting (`SharedExpenseDao`)**:
   - Attaches participant split shares to existing transactions via `shared_expenses.transaction_id`.
   - Settling shares generates an `income` transaction categorized under `Settlement / Reimbursement`.
6. **Financial Goals & Sinking Funds (`GoalDao`)**:
   - Deposit contributions create `expense` transactions linked to categories with `is_savings = 1`, keeping capital allocations separate from operating expenses.
7. **Proactive Notifications (`NotificationDao`)**:
   - In-app notification center tracking budget warning thresholds, bill due dates, and system alerts.
8. **Transaction Integrity Engine (`checkTransactionIntegrity`)**:
   - Audits transfer parity (`tr-*` and `rc-*`), orphaned category references, non-positive amounts, and soft-delete states with non-destructive auto-fix.
9. **Duplicate Detection & Clean Up Engine (`DuplicateDetector`)**:
   - Detects redundant transactions across two matching strategies: *Exact Match* (date, amount, type, normalized note) and *Date & Amount Match* (date, amount, type).
   - Groups duplicates chronologically with the oldest tagged as Original.
   - Provides safe batch soft-deletion (`deleteTransactions(ids)`), reversing associated account balances and keeping records in the Trash Bin for full recovery.

10. **First-Launch Setup & Dummy Data Notification Engine**:
    - Detects initial sample data seeding in SQLite `DatabaseCallback`.
    - Integrates a one-time setup reminder in `DashboardViewModel` with smart detection against sample transaction descriptions.
    - Prevents existing users updating the app from receiving redundant prompts.
11. **Material 3 Calendar Date Picker Component (`DatePickerField`)**:
    - Replaces manual string text fields with Material 3 `DatePickerDialog` across all modules.
    - Automatically parses and formats dates in ISO `yyyy-MM-dd` format while supporting custom display patterns.

---

## 📊 Enterprise Accounting Statements & Reports Engine

The domain layer compiles transaction records into formal accounting statements:

1. **Balance Sheet ($Assets = Liabilities + Equity$)**:
   - Asset Pillars: Current Liquid Assets (Checking, Savings, Cash) and Investment Assets.
   - Liability Pillars: Credit Cards, Personal Loans, Bank Overdrafts.
   - Equity Decomposition: $\text{Total Equity} = \text{Contributed Capital} + \text{Retained Surplus} + \text{Current P\&L}$.
2. **Profit & Loss Statement (P&L)**:
   - Operating Surplus = Operating Revenue - Operating Expenses.
   - Excludes savings bucket allocations (`is_savings = 1`) and contra-transfers (`Transfer`, `tr-*`, `rc-*`).
3. **Statement of Cash Flows**:
   - Separated into Operating, Investing (`is_savings = 1`), and Financing (debt principal repayments) cash flow activities. Reconciles liquid cash across accounts.
4. **Double-Entry Trial Balance**:
   - Audits debits and credits across the 5 Chart of Accounts pillars: `1xxx Assets`, `2xxx Liabilities`, `3xxx Equity`, `4xxx Revenue`, `5xxx Expenses` ($\sum \text{Debits} = \sum \text{Credits}$).
5. **Tax Deductions Summary**:
   - Aggregates transactions marked with `is_tax_deductible = 1` for statutory tax reporting.

---

## 💾 CSV, JSON & Raw SQL Serialization Engine

The repository provides serialization and backup tools:
- **`exportDataCsv()`**: Formats transactions into CSV spreadsheet format.
- **`importDataCsv(csvString)`**: Parses CSV text and inserts transaction records into Room SQLite.
- **`exportDataJson()`**: Exports complete relational database state into JSON format.
- **`importDataJson(jsonString)`**: Restores database state from JSON backup.
- **`exportDataSql()`**: Generates a standard SQLite dump script containing `INSERT OR REPLACE INTO ...` statements for categories, transactions, accounts, budgets, recurring schedules, rules, debts, shared expenses, and goals.
- **`importDataSql(sqlString)`**: Executes raw SQL DML statements inside an atomic database transaction (`db.beginTransaction()`).

---

## 👨‍💻 Developer Information

- **Developer Name**: Vipul Verulkar
- **App Version**: 6.1.11 (Code: 6111)
- **Location**: India 🇮🇳
- **Contact Email**: `vipulverulkar@gmail.com`

---

## 📋 Technical Specs & Dependencies

| Component | Specification |
| :--- | :--- |
| **Language** | Kotlin 2.0+ / 2.4+ |
| **Minimum SDK** | SDK 26 (Android 8.0 Oreo) |
| **Target & Compile SDK** | SDK 36 (Android 16) |
| **UI Toolkit** | Jetpack Compose BOM + Material Design 3 |
| **Architecture** | Modern Android Development (MAD) + Clean Architecture |
| **Local Database** | Android Jetpack Room `2.7.0` (SQLite ORM, Database v4) |
| **Concurrency & State** | Kotlinx Coroutines `1.9.0` & Reactive `StateFlow` |
| **Navigation** | AndroidX Navigation Compose `2.8.3` |
| **Annotation Processing**| Google KSP |
