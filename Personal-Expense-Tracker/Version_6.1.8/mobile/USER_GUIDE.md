# 📱 User Guide — Personal Expense Tracker Mobile App (v6.1.8)

Welcome to the **Personal Expense Tracker** Android mobile application! This guide provides a detailed walkthrough of all features, themes, multi-currency settings, date preferences, transaction editing, recurring schedules, financial planning tools, and backup/restore workflows.

---

## 🔒 Privacy, Security & Offline First

- **Zero Registration or Account Required**: Open the app and start tracking immediately.
- **SQLCipher AES-256 Database Encryption**: All database records on disk are transparently encrypted at rest with hardware-backed Android Keystore keys.
- **Biometric App Lock**: Secure your financial data with Fingerprint / Face Unlock before displaying records.
- **100% Offline Storage**: All transactions, categories, budgets, debts, goals, and recurring schedules are stored strictly on your device in an embedded Room SQLite database. Zero cloud syncing or external telemetry.

---

## 📱 App Walkthrough & Screenshots

````carousel
![Dashboard Screen](/home/vipul/.cache/Google/AndroidStudio2026.2.1/projects/mobile_app.3a6f1706/.artifacts/4e94aeeb-1f64-4382-9bd5-b49548eed5d5/dashboard.png)
<!-- slide -->
![Transactions Screen](/home/vipul/.cache/Google/AndroidStudio2026.2.1/projects/mobile_app.3a6f1706/.artifacts/4e94aeeb-1f64-4382-9bd5-b49548eed5d5/transactions.png)
<!-- slide -->
![Categories Screen](/home/vipul/.cache/Google/AndroidStudio2026.2.1/projects/mobile_app.3a6f1706/.artifacts/4e94aeeb-1f64-4382-9bd5-b49548eed5d5/categories.png)
<!-- slide -->
![Statistics & Analytics Screen](/home/vipul/.cache/Google/AndroidStudio2026.2.1/projects/mobile_app.3a6f1706/.artifacts/4e94aeeb-1f64-4382-9bd5-b49548eed5d5/stats.png)
<!-- slide -->
![Advance Tools Screen](/home/vipul/.cache/Google/AndroidStudio2026.2.1/projects/mobile_app.3a6f1706/.artifacts/4e94aeeb-1f64-4382-9bd5-b49548eed5d5/advance.png)
<!-- slide -->
![Settings & Themes Screen](/home/vipul/.cache/Google/AndroidStudio2026.2.1/projects/mobile_app.3a6f1706/.artifacts/4e94aeeb-1f64-4382-9bd5-b49548eed5d5/settings.png)
````

---

## 🧭 Main Navigation & Tools Overview

The bottom navigation bar provides 1-tap access to 5 primary screens:

| Tab | Icon | Description |
| :--- | :---: | :--- |
| **Dashboard** | 📊 | Real-time balance hero card, income vs expense totals, daily spending pace, top bar Notification Bell, and `⚙️` Settings access. |
| **Transactions** | 🧾 | Full transaction ledger with search, segmented filters, in-app editing (`✏️`), Voice Quick Add (`🎙️`), and Top Bar menu (`⋮`). |
| **Categories** | 🏷️ | Manage Income & Expense categories, color badges, and savings goal indicators. |
| **Stats** | 📈 | Daily average spend, 6-Month Cash Flow Trend chart, financial health pulse, top expense, and category breakdowns. |
| **Advance** | ⚙️ | Bank & SMS Import tools, Financial Planning tools (Debts, Rules, Shared Splits, Goals, Reports, Trash), CSV/SQL Backup/Restore. |

---

## 💵 1. Recording, Editing, Voice Quick Add & Scheduling

### Manual Entry
1. Tap the **`+` (Floating Action Button)** on the Dashboard or Transactions screen.
2. Select **`Expense`** or **`Income`**.
3. Enter the **Amount** in your selected currency (*e.g., ₹, $, €, £, ¥, A$*).
4. Select the **Category** from the dropdown menu.
5. *(Optional)* Choose a **Repeat / Schedule** frequency (**One-time**, **Daily**, **Weekly**, **Monthly**, **Yearly**).
6. Enter an optional **Note / Description** (*e.g., Swiggy Lunch, House Rent*).
7. Tap **`Save Transaction`**.

### Voice Quick Add (`🎙️`)
1. On the Add/Edit Transaction sheet, tap the **Microphone (`🎙️`)** icon.
2. Speak your expense naturally (*e.g., "Spent 250 rupees on groceries at D-Mart"*).
3. The speech parser auto-fills the amount, transaction type, and description!

### In-App Editing (`✏️`)
1. On the Dashboard or Transactions screen, tap any transaction card or tap the **Edit Pencil (`✏️`)** icon.
2. Modify any field and tap **`Update Transaction`** to save changes directly.

---

## 💳 2. Financial Planning & Management Tools

Access these modules under **Advance Tools**:

- 💳 **Debt Payoff Planner**: Track outstanding loan principals, APR interest rates, due days, Avalanche vs. Snowball payoff strategy simulators, and record EMI repayments.
- ⚡ **Automated Transaction Rules**: Manage RegEx/substring pattern matching against transaction notes or merchant names to auto-assign categories and tags.
- 👥 **Group Shared Expenses**: Track split shares across friends, per-person balances, single split settlements, and bulk person settlements.
- 🎯 **Savings Goals & Sinking Funds**: Sinking fund targets, progress percentage bars, target dates, and deposit contributions.
- 📊 **Accounting Financial Statements**: Tabbed formal statements: **Balance Sheet** ($Assets = Liabilities + Equity$), **Profit & Loss Statement**, **Statement of Cash Flows**, **Double-Entry Trial Balance**, **Tax Deductions Summary**, and **Export A4 PDF Report**.
- 🔔 **Notifications & Alerts**: Top-bar notification bell drawer tracking budget thresholds and bill due alerts.
- 🗑️ **Trash Bin Recovery**: View soft-deleted transactions (`deleted_at`), restore them, or permanently purge them.

---

## 📱 3. Bank & SMS Auto-Pick

### A. SMS Transaction Auto-Pick
1. Tap the Top Bar menu (`⋮`) on the Transactions screen (or tap **`Auto-Pick SMS`** in Advance tools).
2. The app scans your SMS inbox locally for bank/UPI alerts (*HDFC, SBI, ICICI, Axis, Kotak, Paytm, GPay, PhonePe*).
3. Review auto-detected transactions and tap **`Import All`**.

### B. Bank Statement CSV Import
1. Tap the Top Bar menu (`⋮`) on the Transactions screen and select **`Import Bank Statement`**.
2. Pick a statement file (*from HDFC, SBI, ICICI, etc.*) or paste statement text.
3. Review the parsed list and tap **`Import Transactions`**.

---

## 🎨 4. Settings, Themes, Demo Data & Clear Data

In **Settings**:
- **Color Theme Presets**: Choose from 5 custom Material 3 color schemes (*Indigo Royal, Emerald Mint, Ocean Sapphire, Sunset Rose, Midnight Gold*).
- **Multi-Currency Symbol**: Select **`₹`**, **`$`**, **`€`**, **`£`**, **`¥`**, **`A$`**.
- **Financial Year Start Month**: Choose April 1 for Indian FY (Apr-Mar) or Jan 1 for Calendar Year.
- **⚡ Load Sample Demo Data**: Load sample accounts, transactions, debts, goals, rules, and notifications.
- **🗑️ Delete All Application Data**: Clear all database tables with confirmation.

---

## 👨‍💻 About & Developer Information

Viewable inside the app under **Settings $\to$ About & Developer Info**:
- **App Version**: 6.1.8
- **Developer**: Vipul Verulkar
- **Location**: India 🇮🇳
- **Contact Email**: `vipulverulkar@gmail.com`

---

*Thank you for using Personal Expense Tracker! Track smart, save more.*
