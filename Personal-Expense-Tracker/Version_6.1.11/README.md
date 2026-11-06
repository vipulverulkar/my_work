# Personal Expense Tracker — Android Mobile App (v6.1.11)

A lightweight, user-friendly, and modern Android mobile application built using Google's **Modern Android Development (MAD)** architecture.

---

## 📱 App Screenshots

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

## 🌟 Key Features

- **100% Offline & Privacy-Focused**: Embedded **Jetpack Room (SQLite)** database with transparent **SQLCipher AES-256 Encryption-at-Rest** and hardware-backed Android Keystore keys.
- **Biometric Security & App Lock**: Secure your financial data with Fingerprint / Face Unlock before displaying records.
- **Glance Home Screen App Widget**: Home screen widget (`androidx.glance:glance-appwidget`) displaying live Net Worth overview and 1-tap app launch shortcut.
- **Modern Jetpack Compose & Material Design 3 UI**: 5 custom **Color Theme Presets** (*Indigo Royal, Emerald Mint, Ocean Sapphire, Sunset Rose, Midnight Gold*).
- **Interactive Material 3 Date Picker**: Seamless calendar date selection across all entry sheets and dialogs (*Add/Edit Transaction, Recurring Schedules, Debt Payments, Goals, Shared Expenses*).
- **Comprehensive Account & Net Worth Management**: Full CRUD support for accounts (*Checking, Savings, Credit Card, Cash, Digital Wallet, Investment*) with live balance calculation, editing, deletion protection, and a unified Net Worth summary.
- **First-Launch Setup & Dummy Data Notification**: Proactive in-app notification and Dashboard reminder card alerting users on first install to easily delete pre-loaded sample transactions with 1 tap before tracking personal expenses.
- **Real-Time Financial Dashboard**: Total Net Balance hero card, income vs expense totals, daily spending pace, top bar Notification Bell with unread badge counter, and auto-recurring catchup engine.
- **Voice Quick Add (`🎙️`)**: Speech recognition (`RecognizerIntent.ACTION_RECOGNIZE_SPEECH`) parsing spoken sentences into amount, category, and note fields.
- **Financial Planning & Accounting Tools**:
  - 💳 **Debt Payoff Planner**: Avalanche vs. Snowball payoff strategy simulators, principal reduction, and EMI repayments.
  - ⚡ **Automated Transaction Rules**: RegEx pattern matcher to auto-assign categories and tags.
  - 👥 **Group Shared Expenses**: Track split shares across friends and participant settlements.
  - 🎯 **Savings Goals & Sinking Funds**: Progress percentage bars, target dates, and deposit contributions.
  - 📊 **Accounting Financial Statements**: Balance Sheet ($Assets = Liabilities + Equity$), Profit & Loss, Statement of Cash Flows, Double-Entry Trial Balance, Tax Summary, and **Native A4 PDF Export**.
  - 🗑️ **Trash Bin Recovery**: Soft-delete listing (`deleted_at`), restore action, and permanent purge action.
  - 🧹 **Duplicate Transactions Cleanup**: Advanced duplicate detection (Exact Match & Date/Amount match), grouped transaction review, Keep Oldest / Keep Newest / manual multi-selection, user deletion confirmation, and soft-delete Trash Bin protection.
- **Bank Statement Importer & OCR**: Parses CSV/text bank statements and extracts receipt totals locally via on-device OCR.
- **Data & Demo Management**:
  - **⚡ Load Sample Demo Data**: Load realistic demo data across all modules with 1 tap.
  - **🗑️ Delete All Application Data**: Safely wipe all database tables locally.

---

## 🔒 Security & Privacy Audit Summary

- **Zero Cloud Data Exfiltration**: All data processing is executed 100% locally on-device.
- **SQL Injection Prevention**: All database queries use Room parametrized SQLite statements.
- **Integer Overflow Protection**: Aggregate calculations use SQLite's `TOTAL(...)` 64-bit floating point function.
- **Scoped Storage & SAF**: File export/import utilizes Android's native Storage Access Framework (SAF).

---

## ⚙️ Technical Specifications

- **Architecture**: Modern Android Development (MAD) with Jetpack Compose, Material 3, Clean Architecture, Repository Pattern, and Kotlin Coroutines/Flows.
- **Minimum SDK**: Android 8.0 (API level 26)
- **Target / Compile SDK**: Android 16 / 17 (API level 36)
- **Database**: Room 2.7.0 (SQLite) with transparent SQLCipher AES-256 encryption at rest.

---

## 👨‍💻 Developer Information

- **App Version**: 6.1.11 (Code: 6111)
- **Developer Name**: Vipul Verulkar
- **Location**: India 🇮🇳
- **Contact Email**: `vipulverulkar@gmail.com`
