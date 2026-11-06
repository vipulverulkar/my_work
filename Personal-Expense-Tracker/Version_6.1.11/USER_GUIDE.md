# 📱 User Guide — Personal Expense Tracker Mobile App (v6.1.11)

Welcome to the **Personal Expense Tracker** Android mobile application! This guide provides a detailed walkthrough of all features, themes, multi-currency settings, date preferences, transaction editing, recurring schedules, financial planning tools, and backup/restore workflows.

---

## 🔒 Privacy, Security & Offline First

- **Zero Registration or Account Required**: Open the app and start tracking immediately.
- **SQLCipher AES-256 Database Encryption**: All database records on disk are transparently encrypted at rest with hardware-backed Android Keystore keys.
- **Biometric App Lock**: Secure your financial data with Fingerprint / Face Unlock before displaying records.
- **100% Offline Storage**: All transactions, categories, budgets, debts, goals, and recurring schedules are stored strictly on your device in an embedded Room SQLite database. Zero cloud syncing or external telemetry.

---

## 📱 App Walkthrough & Screenshots

| Dashboard | Transactions | Categories |
| :---: | :---: | :---: |
| ![Dashboard Screen](screenshot/dashboard.png) | ![Transactions Screen](screenshot/transactions.png) | ![Categories Screen](screenshot/categories.png) |

| Statistics & Analytics | Advance Tools | Settings & Themes |
| :---: | :---: | :---: |
| ![Statistics Screen](screenshot/stats.png) | ![Advance Tools Screen](screenshot/advance.png) | ![Settings Screen](screenshot/settings.png) |

| Accounts & Net Worth | Notifications & Alerts |
| :---: | :---: |
| ![Accounts & Net Worth Screen](screenshot/accounts.png) | ![Notifications & Alerts Screen](screenshot/notifications.png) |

---

## 🧭 Main Navigation & Tools Overview

The bottom navigation bar provides 1-tap access to 5 primary screens:

| Tab | Icon | Description |
| :--- | :---: | :--- |
| **Dashboard** | 📊 | Real-time balance hero card, first-launch setup reminder, income vs expense totals, daily spending pace, top bar Notification Bell, and `⚙️` Settings access. |
| **Transactions** | 🧾 | Full transaction ledger with search, segmented filters, in-app editing (`✏️`), Voice Quick Add (`🎙️`), and Top Bar menu (`⋮`). |
| **Categories** | 🏷️ | Manage Income & Expense categories, color badges, and savings goal indicators. |
| **Stats** | 📈 | Daily average spend, 6-Month Cash Flow Trend chart, financial health pulse, top expense, and category breakdowns. |
| **Advance** | ⚙️ | Bank Statement Importer, Financial Planning tools (Debts, Rules, Shared Splits, Goals, Reports, Trash, Duplicate Cleanup), CSV/SQL Backup/Restore. |

---

## 🚀 1. First-Launch Setup & Sample Data Reminder

When installing the app for the first time:
- **Pre-Loaded Showcase Data**: 4 sample demo transactions (*Salary, House Rent, Groceries, Dinner*) are added so you can immediately see charts, statistics, and category distributions in action.
- **Top Bar Notification**: The notification bell alerts you with a *"Sample Demo Data Loaded"* notice.
- **Dashboard Banner**: An eye-catching reminder card at the top of the Dashboard prompts you:
  - **`Delete Dummy Data`**: 1-tap action with a confirmation dialog that deletes all 4 demo transactions while leaving your accounts and categories completely intact.
  - **`Keep / Dismiss`**: Dismisses the reminder if you prefer to keep testing with sample figures.
- **Safety Guarantee**: Existing users updating the app will **never** have their real data deleted or be shown this prompt.

---

## 🏦 2. Accounts & Net Worth Management

Tap the **✨ Total Net Balance Hero Card** or **Accounts Quick Action** on the Dashboard:
- **Total Net Worth Overview**: Calculates your aggregate liquid wealth and net cash across all active bank, wallet, and investment accounts.
- **Add Account**: Tap the **`+ Add Account`** button to create a new account:
  - Specify the Account Name (*e.g., HDFC Salary, SBI Savings, Cash Wallet, Zerodha Demat*).
  - Select Account Type (*Checking, Savings, Credit Card, Cash, Digital Wallet, Investment*).
  - Enter the Opening Balance (in ₹, $, €, etc.).
  - Specify the Financial Institution (*e.g., HDFC Bank, Physical Cash*).
- **Edit Account (`✏️`)**: Tap the edit pencil icon on any account card to modify the name, account type, bank institution, or adjust the balance.
- **Delete Account (`🗑️`)**: Delete unused or closed accounts with confirmation protection. Linked transactions remain safely archived.

---

## 💵 3. Recording, Editing, Voice Quick Add & Scheduling

### Manual Entry
1. Tap the **`+` (Floating Action Button)** on the Dashboard or Transactions screen.
2. Select **`Expense`** or **`Income`**.
3. Enter the **Amount** in your selected currency (*e.g., ₹, $, €, £, ¥, A$*).
4. Select the **Category** from the dropdown menu.
5. **Account / Payment Method**: Automatically selects your primary account showing its live balance. Tap to switch accounts—account balances adjust automatically upon saving!
6. **Date Selection**: Tap the **Date** field to open the native **Material 3 Calendar Date Picker**. Select any past or current date with 1 tap (no manual date text typing required).
7. *(Optional)* Choose a **Repeat / Schedule** frequency (**One-time**, **Daily**, **Weekly**, **Monthly**, **Yearly**).
8. Enter an optional **Note / Description** (*e.g., Swiggy Lunch, House Rent*).
9. Tap **`Save Transaction`**.

### Voice Quick Add (`🎙️`)
1. On the Add/Edit Transaction sheet, tap the **Microphone (`🎙️`)** icon.
2. Speak your expense naturally (*e.g., "Spent 250 rupees on groceries at D-Mart"*).
3. The speech parser auto-fills the amount, transaction type, and description!

### In-App Editing (`✏️`)
1. On the Dashboard or Transactions screen, tap any transaction card or tap the **Edit Pencil (`✏️`)** icon.
2. Modify any field—including changing the date with the Material 3 Date Picker—and tap **`Update Transaction`**.

---

## 💳 4. Financial Planning & Management Tools

Access these modules under **Advance Tools**:

- 💳 **Debt Payoff Planner**: Track outstanding loan principals, APR interest rates, due days, Avalanche vs. Snowball payoff strategy simulators, and record EMI repayments using the integrated date picker.
- ⚡ **Automated Transaction Rules**: Manage RegEx/substring pattern matching against transaction notes or merchant names to auto-assign categories and tags.
- 👥 **Group Shared Expenses**: Track split shares across friends, per-person balances, single split settlements, and bulk person settlements.
- 🎯 **Savings Goals & Sinking Funds**: Sinking fund targets, progress percentage bars, target dates selected via date picker, and deposit contributions.
- 📊 **Accounting Financial Statements**: Tabbed formal statements: **Balance Sheet** ($Assets = Liabilities + Equity$), **Profit & Loss Statement**, **Statement of Cash Flows**, **Double-Entry Trial Balance**, **Tax Deductions Summary**, and **Export A4 PDF Report**.
- 🔔 **Notifications & Alerts**: Top-bar notification bell drawer tracking budget thresholds, bill due alerts, and sample data notices. Features 1-tap dummy data cleanup, individual alert dismissal (`✕`), and a top-bar **Clear All** action.
- 🗑️ **Trash Bin Recovery**: View soft-deleted transactions (`deleted_at`), restore them, or permanently purge them.
- 🧹 **Duplicate Transactions Cleanup**: Scan your ledger for redundant or duplicate transactions with customizable matching rules:
  - **Exact Match**: Matches records having identical Date, Amount, Type, and Note.
  - **Date & Amount**: Matches records having the same Date and Amount (regardless of variations in merchant descriptions).
  - **Interactive Selection**: Review grouped duplicates with clear badges for the *Original (Oldest)* record vs *Copies*. Use quick buttons (**Keep Oldest**, **Keep Newest**, **All**, **Clear**) or toggle checkboxes per transaction.
  - **Safe Deletion & Confirmation**: Requires confirmation showing the total count and monetary amount before soft-deleting items into the Trash Bin and adjusting linked account balances. Recover anytime from the Trash Bin shortcut.

---

## 📱 5. Bank Statement & Receipt Imports

### A. Bank Statement CSV Import
1. Tap the Top Bar menu (`⋮`) on the Transactions screen (or use Advance tools) and select **`Import Bank Statement`**.
2. Pick a statement file (*from HDFC, SBI, ICICI, etc.*) or paste statement text.
3. Review the parsed list and tap **`Import Transactions`**.

### B. Receipt Scanner (OCR)
1. Tap the Top Bar menu (`⋮`) on the Transactions screen and select **`Scan Receipt (OCR)`**.
2. Capture a photo or select a receipt from your device storage to automatically extract date, merchant, and total amount.

---

## 🎨 6. Settings, Themes, Demo Data & Clear Data

In **Settings**:
- **Color Theme Presets**: Choose from 5 custom Material 3 color schemes (*Indigo Royal, Emerald Mint, Ocean Sapphire, Sunset Rose, Midnight Gold*).
- **Multi-Currency Symbol**: Select **`₹`**, **`$`**, **`€`**, **`£`**, **`¥`**, **`A$`**.
- **Financial Year Start Month**: Choose April 1 for Indian FY (Apr-Mar) or Jan 1 for Calendar Year.
- **⚡ Load Sample Demo Data**: Load sample accounts, transactions, debts, goals, rules, and notifications.
- **🗑️ Delete All Application Data**: Clear all database tables with confirmation.

---

## 👨‍💻 About & Developer Information

Viewable inside the app under **Settings $\to$ About & Developer Info**:
- **App Version**: 6.1.11 (Code: 6111)
- **Developer**: Vipul Verulkar
- **Location**: India 🇮🇳
- **Contact Email**: `vipulverulkar@gmail.com`

---

*Thank you for using Personal Expense Tracker! Track smart, save more.*
