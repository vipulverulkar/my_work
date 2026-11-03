# Personal Expense Tracker — Android Mobile App (v6.1.8)

A lightweight, user-friendly, and modern Android mobile application built using Google's **Modern Android Development (MAD)** architecture.

---

## 📱 App Screenshots

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

## 🌟 Key Features

- **100% Offline & Privacy-Focused**: Embedded **Jetpack Room (SQLite)** database with transparent **SQLCipher AES-256 Encryption-at-Rest** and hardware-backed Android Keystore keys.
- **Biometric Security & App Lock**: Secure your financial data with Fingerprint / Face Unlock before displaying records.
- **Glance Home Screen App Widget**: Home screen widget (`androidx.glance:glance-appwidget`) displaying live Net Worth overview and 1-tap app launch shortcut.
- **Modern Jetpack Compose & Material Design 3 UI**: 5 custom **Color Theme Presets** (*Indigo Royal, Emerald Mint, Ocean Sapphire, Sunset Rose, Midnight Gold*).
- **Real-Time Financial Dashboard**: Total Net Balance hero card, income vs expense totals, daily spending pace, top bar Notification Bell with unread badge counter, and auto-recurring catchup engine.
- **Voice Quick Add (`🎙️`)**: Speech recognition (`RecognizerIntent.ACTION_RECOGNIZE_SPEECH`) parsing spoken sentences into amount, category, and note fields.
- **Financial Planning & Accounting Tools**:
  - 💳 **Debt Payoff Planner**: Avalanche vs. Snowball payoff strategy simulators, principal reduction, and EMI repayments.
  - ⚡ **Automated Transaction Rules**: RegEx pattern matcher to auto-assign categories and tags.
  - 👥 **Group Shared Expenses**: Track split shares across friends and participant settlements.
  - 🎯 **Savings Goals & Sinking Funds**: Progress percentage bars, target dates, and deposit contributions.
  - 📊 **Accounting Financial Statements**: Balance Sheet ($Assets = Liabilities + Equity$), Profit & Loss, Statement of Cash Flows, Double-Entry Trial Balance, Tax Summary, and **Native A4 PDF Export**.
  - 🗑️ **Trash Bin Recovery**: Soft-delete listing (`deleted_at`), restore action, and permanent purge action.
- **SMS Auto-Pick & Bank Statement Importer**: Auto-scans bank/UPI SMS alerts locally (*HDFC, SBI, ICICI, Axis, Kotak, Paytm, GPay, PhonePe*) and parses CSV/text bank statements.
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

## 👨‍💻 Developer Information

- **App Version**: 6.1.8 (Code: 618)
- **Developer Name**: Vipul Verulkar
- **Location**: India 🇮🇳
- **Contact Email**: `vipulverulkar@gmail.com`
