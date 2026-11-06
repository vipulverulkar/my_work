# Privacy Policy for Personal Expense Tracker

**Last Updated**: October 2025  
**Developer**: Vipul Verulkar  
**Contact Email**: `vipulverulkar@gmail.com`  
**Application**: Personal Expense Tracker (`com.expensetracker.mobile`)

---

## 🔒 1. Overview & Core Philosophy

**Personal Expense Tracker** is designed with a **100% Offline-First and Privacy-First** architecture. The Application operates completely locally on your Android device. We believe your personal financial data belongs solely to you.

- **Zero Data Collection**: We do not collect, transmit, upload, or store any of your personal or financial data on external servers or cloud services.
- **Zero Third-Party Sharing**: No data, telemetry, analytics, or transaction details are ever shared with third parties, advertisers, or analytics networks.
- **100% Local Storage**: All transaction logs, account balances, categories, savings goals, debts, and settings are stored locally on your device in an embedded Room SQLite database.

---

## 📱 2. Permissions Disclosures & Data Usage

### A. SMS Permissions (`READ_SMS` & `RECEIVE_SMS`)
- **Purpose**: The Application includes an optional **Bank SMS Auto-Pick Engine**.
- **How It Works**: When granted, the Application reads incoming or inbox SMS notifications strictly from recognized banking institutions (such as HDFC, SBI, ICICI, Axis, Paytm, UPI) to parse transaction amounts, dates, and merchant names.
- **Local Thread Execution**: SMS content is processed **100% locally in local memory thread contexts**. The SMS text is never saved off-device or transmitted over any network.
- **User Control**: SMS access is strictly optional and can be revoked at any time via Android system settings.

### B. Camera Permission (`CAMERA`)
- **Purpose**: Used solely for the optional **In-App Receipt OCR Scanning** feature.
- **How It Works**: Allows taking photo snapshots of physical purchase receipts to extract total amounts and merchant names using on-device ML Kit Text Recognition.
- **Local Processing**: Camera frames and OCR recognition execute completely on-device. No images are uploaded to any external server.

### C. Biometric Permission (`USE_BIOMETRIC`)
- **Purpose**: Used for the optional **App Lock** security feature.
- **How It Works**: Authenticates your identity via Android's native Fingerprint / Face Unlock prompt before displaying financial records.
- **Hardware Isolation**: Biometric verification is handled by Android's hardware security module (TEE/StrongBox); the Application never accesses or stores raw biometric data.

---

## 💾 3. Data Storage & Local Backups

- All financial records reside on your device's internal storage (`/data/data/com.expensetracker.mobile/databases/`).
- **Data Export**: Any backup exports (CSV spreadsheet or SQL dump) created by the user are stored strictly in local locations selected by the user via Android's Scoped Storage Access Framework (SAF).
- **Data Deletion**: Users can clear all application data at any time via **Settings ➔ Delete All Application Data**, which permanently wipes all database records locally. Uninstalling the app also removes all local database files.

---

## 🛡️ 4. Google Play Data Safety Declaration

| Data Safety Question | Declaration |
| :--- | :--- |
| **Does this app collect user data?** | **No** (Zero data collected) |
| **Does this app share user data with third parties?** | **No** (Zero data shared) |
| **Are data transfers encrypted in transit?** | Not Applicable (No network data transfers) |
| **Can users request data deletion?** | **Yes** (Instant 1-tap local data purge in Settings) |

---

## 📬 5. Contact Us

If you have any questions or feedback regarding this Privacy Policy, please contact:

- **Developer**: Vipul Verulkar
- **Email**: `vipulverulkar@gmail.com`
- **Location**: India 🇮🇳
