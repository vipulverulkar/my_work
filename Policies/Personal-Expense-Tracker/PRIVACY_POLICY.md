# Privacy Policy for Personal Expense Tracker

**Effective & Last Updated**: November 6, 2026  
**Application**: Personal Expense Tracker  
**Package Name (Application ID)**: `com.personalexpensetracker` (Internal Namespace: `com.expensetracker.mobile`)  
**Current Version**: 6.1.11 (versionCode: 6111)  
**Developer**: Vipul Verulkar  
**Contact Email**: `vipulverulkar@gmail.com`  
**Location**: India 🇮🇳  
**Public Repository**: [GitHub Project](https://github.com/vipulverulkar/Exp_tracker_V6.1_with_docker_postgres_and_sqllite)

---

## 🔒 1. Overview & Core Philosophy

**Personal Expense Tracker** ("the Application", "we", "us", or "our") is built on a **100% Offline-First and Privacy-First** architecture. We believe your personal financial data belongs solely to you and must remain under your exclusive control at all times.

- **Zero Network Transmission**: The Application **does not request or declare** the `android.permission.INTERNET` permission in its manifest. As a result, the Application is technically and architecturally incapable of communicating with external servers, cloud providers, advertising brokers, or analytics networks.
- **Zero Remote Data Collection**: We do not collect, transmit, upload, harvest, or monitor any personal information, financial data, device identifiers, or telemetry.
- **Zero Third-Party Sharing**: No data or transaction logs are ever shared with, sold to, or disclosed to third parties, advertisers, or trackers.
- **100% Local Encrypted Storage**: All transaction logs, accounts, balances, budgets, categories, goals, debts, and preferences are stored exclusively on your device in an isolated Room SQLite database protected by SQLCipher AES-256 encryption at rest.

---

## 📱 2. Permissions & Feature Disclosures

The Application uses only the minimal set of Android system permissions strictly necessary to deliver optional, user-initiated features. Every permission is handled entirely on-device:

### A. Camera (`android.permission.CAMERA`)
- **Purpose**: Optional **Receipt OCR Scanning**.
- **Usage**: Allows you to take a photo of a physical paper receipt to extract the transaction date, total amount, and merchant name.
- **Processing**: Camera frames and optical text recognition (ML Kit Text Recognition) are processed **100% on-device** in local memory. No photo or receipt image is ever transmitted or uploaded.
- **Hardware Declaration**: Marked with `android:required="false"` so devices without a camera can still install and run the app.
- **User Control**: Camera access is requested at runtime and is strictly optional.

### B. Biometric Authentication (`android.permission.USE_BIOMETRIC`)
- **Purpose**: Optional **App Lock & Biometric Security**.
- **Usage**: Allows securing the app with your device's fingerprint or facial recognition before displaying accounts or transaction ledgers.
- **Hardware Security**: Authentication is performed by Android's hardware security module (Trusted Execution Environment / StrongBox). The Application never accesses, receives, or stores raw biometric templates.

### C. Notification Listener Service (`NotificationListenerService`)
- **Purpose**: Optional **Bank Transaction Push Alert Detection**.
- **Usage**: Allows parsing incoming transaction push alerts from verified banking apps (e.g., UPI, HDFC, SBI, ICICI, Axis, Paytm).
- **Processing**: When explicitly enabled by the user in system settings, notification texts are evaluated **100% locally in-memory** against local RegEx patterns to extract transaction amounts and merchant details. Notification contents are never saved off-device or transmitted.
- **Prominent Disclosure**: The app displays a clear in-app disclosure and requires affirmative user consent before directing the user to Android's Notification Access settings. Users can revoke this permission at any time.

### D. SMS Permissions & Google Play Distribution Separation
- **Google Play Store Build (`playStore` flavor)**:
  - In strict compliance with Google Play's SMS and Call Log Policy, all SMS-related permissions (`READ_SMS`, `RECEIVE_SMS`, `BROADCAST_SMS`) and receivers are **completely stripped and excluded** from the Google Play release build (`tools:node="remove"` in `app/src/playStore/AndroidManifest.xml`).
  - Google Play users experience full financial management, manual transaction logging, voice quick-add, receipt OCR, bank CSV statement import, and local notification parsing without any SMS permissions.
- **Direct Sideload Build (`sideload` flavor)**:
  - Maintained strictly for users who choose to download the standalone APK from GitHub/F-Droid. When granted by the user, SMS messages from recognized banks are parsed **100% locally in-memory**.

---

## 💾 3. Data Storage, Security & Encryption-at-Rest

- **SQLCipher AES-256 Encryption**: All local SQLite database files (`/data/data/com.expensetracker.mobile/databases/`) are encrypted at rest using AES-256 via SQLCipher, secured by cryptographic keys managed by Android's hardware-backed Keystore.
- **Scoped Storage Access Framework (SAF)**: Backup exports (CSV spreadsheets, SQL dump scripts, and PDF financial statements) are generated only upon explicit user request and saved solely to the directory chosen by the user through Android's system Document Picker. The app does not request broad `MANAGE_EXTERNAL_STORAGE` or legacy `WRITE_EXTERNAL_STORAGE` permissions.
- **No Cloud Syncing**: There are no background sync tasks, cloud databases, or remote endpoints.

---

## 🗑️ 4. Data Retention, Portability & Complete Erasure

You retain complete and unfettered ownership of your financial records:

- **Data Retention**: Your records remain stored locally on your device for as long as you keep the Application installed, or until you choose to delete them.
- **Data Portability (GDPR Article 20 / DPDP)**: You can export all your transactions, accounts, and categories to open CSV spreadsheets or SQL scripts at any time via **Advance Tools ➔ Data Management** or **Settings ➔ Export Backup**.
- **1-Tap Complete Erasure (GDPR Article 17 / Right to be Forgotten)**: You can permanently wipe all accounts, transactions, categories, and settings from your device at any time by navigating to **Settings ➔ Danger Zone ➔ Delete All Application Data**. This immediately purges the encrypted SQLite database file.
- **Application Uninstall**: Uninstalling the Application through Android OS settings automatically and permanently deletes all internal database files and cryptographic key references from your device.

---

## 🛡️ 5. Google Play Data Safety Declaration

In the Google Play Console Data Safety section, our declarations are deterministic and verifiable:

| Data Safety Form Question | Declaration | Verification & Architecture |
| :--- | :--- | :--- |
| **Does the app collect any user data?** | **No** | Zero data collected. `INTERNET` permission is absent from manifest. |
| **Does the app share user data with third parties?** | **No** | Zero third-party sharing. Zero trackers or analytics SDKs. |
| **Is data encrypted in transit?** | **Not Applicable** | The app never transmits any data over any network. |
| **Is data encrypted at rest?** | **Yes** | Transparent SQLCipher AES-256 database encryption. |
| **Can users request data deletion?** | **Yes** | Instant 1-tap local database purge in Settings. |

---

## 👶 6. Children's Privacy (COPPA & GDPR-K Compliance)

The Application is a general-audience personal finance tool and does not knowingly collect or solicit personal information from children under the age of 13 (or under 16 in the EEA). Because the Application does not collect any personal data from any user, it is fully compliant with the Children's Online Privacy Protection Act (COPPA) and GDPR-K regulations.

---

## ⚖️ 7. Legal Rights & Jurisdiction (GDPR, CCPA & India DPDP Act)

Regardless of your geographic location, the Application guarantees full compliance with global privacy standards including:
- **General Data Protection Regulation (GDPR - EU)**
- **California Consumer Privacy Act (CCPA - USA)**
- **Digital Personal Data Protection Act (DPDP Act, 2023 - India)**

Because all data remains exclusively on your physical device and is never collected or accessed by the developer, your data sovereignty rights (Access, Correction, Erasure, and Portability) are fully satisfied and executable directly by you within the app's user interface.

---

## 🔄 8. Changes to This Privacy Policy

We may periodically update this Privacy Policy to reflect application updates or regulatory requirements. Any modifications will be posted in this document with an updated revision date and made available within the Application's **Settings ➔ View Privacy & Compliance Policy** dialog.

---

## 📬 9. Contact Us & Developer Verification

If you have any questions, suggestions, or concerns regarding this Privacy Policy or the security of the Application, please contact the developer:

- **Developer**: Vipul Verulkar
- **Email**: `vipulverulkar@gmail.com`
- **Application Package**: `com.personalexpensetracker` (Internal Namespace: `com.expensetracker.mobile`)  
- **Location**: Maharashtra, India 🇮🇳
- **Official Policy URL**: `https://github.com/vipulverulkar/Exp_tracker_V6.1_with_docker_postgres_and_sqllite/blob/main/mobile_app/PRIVACY_POLICY.md`
