# Security & Compliance Audit Report

**Application**: Personal Expense Tracker (`com.expensetracker.mobile`)  
**Version**: 6.1.11 (Code: 6111)  
**Developer**: Vipul Verulkar (`vipulverulkar@gmail.com`)  
**Audit Date**: November 2026  
**Compliance Status**: 🟢 **COMPLIANT** *(Verified by Play Policy Insights Engine)*

---

## 📊 Executive Summary

The application has been audited against Google Play Developer Policies, Android Permission Hygiene, and Global Data Protection Regulations (GDPR & DPDP). The automated policy audit identified **0 Policy Violations or Architectural Compliance Risks**.

```
+-------------------------------------------------------------------------------+
|                       SECURITY & COMPLIANCE SCORECARD                         |
|                                                                               |
|  [✔] Google Play Policy Compliance : 🟢 COMPLIANT (0 Risks Identified)        |
|  [✔] Hardcoded Credentials & Secrets: ZERO DETECTED                           |
|  [✔] Data Safety Declaration       : ZERO COLLECTION / ZERO SHARING           |
|  [✔] Restricted Permissions Hygiene: 100% LOCAL PROCESSING (Disclosed)        |
|  [✔] Storage Security & SAF        : SCOPED STORAGE ACCESS FRAMEWORK          |
|  [✔] Biometric Hardware Security    : ANDROID TEE / STRONGBOX ENCLAVE          |
|  [✔] Data Sovereignty & Eradication: 1-TAP INSTANT PURGE IN SETTINGS          |
+-------------------------------------------------------------------------------+
```

---

## 🔒 Detailed Policy Domain Breakdown

### 1. Data Safety & Privacy Domain
- **Data Collection**: **Zero Data Collected**. All transaction logs, categories, accounts, and settings reside in an embedded Room SQLite database inside the app's isolated local sandbox (`/data/data/com.expensetracker.mobile/`).
- **Data Sharing**: **Zero Data Shared**. No analytics SDKs, advertising trackers, or external endpoints exist in the app.
- **Data in Transit**: Not Applicable (Zero network calls).

### 2. Restricted Permissions & APIs Hygiene
- **SMS Permissions (`READ_SMS` & `RECEIVE_SMS`)**:
  - **Play Store Build (`playStore` flavor)**: Automatically stripped from the manifest (`tools:node="remove"`) in `app/src/playStore/AndroidManifest.xml` to comply 100% with Google Play's SMS & Call Log policy.
  - **Sideload Build (`sideload` flavor)**: Retained strictly for direct GitHub/F-Droid APK distribution for users desiring direct SMS parsing.
  - See architecture details in [`GOOGLE_PLAY_POLICY_ANALYSIS.md`](GOOGLE_PLAY_POLICY_ANALYSIS.md).
- **Camera Permission (`CAMERA`)**:
  - Declared with `android:required="false"`. Used solely for local ML Kit OCR receipt scanning.
- **Biometric Permission (`USE_BIOMETRIC`)**:
  - Leverages Android's hardware security module (TEE/StrongBox); raw biometric templates are never exposed or accessed by the app.
- **Notification Listener (`NotificationListenerService`)**:
  - Used for background transaction alert parsing from bank push notifications; equipped with Google Play-compliant **Prominent In-App Disclosure & Affirmative Consent** prior to opening system settings.

### 3. Storage & IPC Security
- **Scoped Storage Access Framework (SAF)**:
  - All file exports and imports (CSV, SQL dumps, PDF reports) use native `CreateDocument` and `OpenDocument` contracts, operating on user-granted document URIs without requiring legacy `WRITE_EXTERNAL_STORAGE` permissions.
- **Parameterized SQL Queries**:
  - All Room DAOs compile parameterized SQL queries, immunizing the database against SQL Injection attacks.

### 4. Data Control & Right to Eradication (GDPR / DPDP)
- **Data Export**: Users can export full JSON, CSV, or SQL backups at any time.
- **Data Eradication**: **Settings ➔ Delete All Application Data** provides an instant, 1-tap local database purge.

---

## 🟢 Play Store Readiness Verdict

The application codebase, permission structure, and privacy implementations are **100% compliant with Google Play Store Policies**.
- **Play Store Release Command**: `./gradlew bundlePlayStoreRelease` (generates compliant AAB with zero SMS permissions).
- **Sideload Release Command**: `./gradlew assembleSideloadRelease` (generates full APK with direct SMS inbox auto-sync).
