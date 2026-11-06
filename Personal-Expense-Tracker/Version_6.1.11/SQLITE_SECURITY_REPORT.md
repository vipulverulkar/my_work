# SQLite Security Architecture & Protection Specification

**Application**: Personal Expense Tracker (`com.expensetracker.mobile`)  
**Database**: Jetpack Room 2.7.0 (SQLite ORM, Database v4)  
**Security Status**: 🟢 **SECURE & ISOLATED**

---

## 🏛️ Executive Summary

The embedded **Room SQLite database** (`expense_tracker.db`) in the Personal Expense Tracker Android application is protected across **7 distinct technical security layers**, spanning process sandboxing, hardware biometric gatekeeping, compile-time query parameterization, arithmetic overflow safeguards, scoped storage isolation, and complete air-gapped network isolation.

```
+-------------------------------------------------------------------------------+
|                       SQLITE DATABASE SECURITY ARCHITECTURE                   |
|                                                                               |
|  [1] Linux Process Sandbox : Mode 0600 UID Private Storage Isolation         |
|  [2] Biometric Gatekeeper  : Android TEE / StrongBox Hardware Enclave         |
|  [3] Query Parameterization: 100% Parameterized Room Prepared Statements      |
|  [4] Overflow Safeguards   : SQLite TOTAL(...) 64-bit Aggregate Calculations  |
|  [5] Scoped Storage SAF    : User-Granted Document URIs for Exports           |
|  [6] Air-Gapped Isolation  : Zero INTERNET Permission & Zero Network Traffic    |
|  [7] AES-256 Encryption    : Optional SQLCipher Page-Level Encryption        |
+-------------------------------------------------------------------------------+
```

---

## 🛡️ Detailed Technical Security Layers

### 1. Android Process & Storage Sandboxing
- **Private Internal Directory**: The SQLite database file resides exclusively at `/data/data/com.expensetracker.mobile/databases/`.
- **Linux UID Discretionary Access Control (DAC)**: Android assigns a unique Linux User ID (UID) to the app. File permissions (`mode 0600` / `rwx------`) restrict database read and write access **exclusively to the app's own process UID**. No other third-party application on the device can access, read, or modify the database file.

### 2. Biometric Hardware Gatekeeper (`androidx.biometric`)
- **Hardware Enclave (TEE / StrongBox)**: When App Lock is enabled in Settings, [`BiometricPromptManager.kt`](app/src/main/java/com/expensetracker/mobile/domain/security/BiometricPromptManager.kt) requires hardware-backed Fingerprint or Face Unlock verification before the UI initializes any Room SQLite DAO queries or renders financial data.

### 3. SQL Injection Immunization (Parameterized Room DAOs)
- **Compile-Time Bound Parameters**: All database operations in Room Data Access Objects (DAOs) use parameterized query bindings (`@Query("SELECT * FROM transactions WHERE date >= :startDate")`).
- **Zero Raw String Concatenation**: User input strings are bound as literal parameters by SQLite's prepared statements (`sqlite3_bind_*`), completely immunizing the database against SQL Injection attacks.

### 4. Integer Overflow Protection (`TOTAL(...)`)
- **64-bit Floating-Point Aggregates**: Aggregate sum queries for Income, Expenses, Saved amounts, and Net Worth use SQLite's native `TOTAL(...)` function instead of `SUM(...)`.
- **Overflow Resilience**: SQLite's `TOTAL()` uses 64-bit IEEE 754 floating-point arithmetic and never throws an integer overflow exception or returns `NULL` under large financial balance calculations.

### 5. Scoped Storage Access Framework (SAF) Isolation
- **No Direct Filesystem Access**: Data exports (CSV spreadsheets or SQL database dumps) and restores use Android's native Scoped Storage Access Framework (`CreateDocument` and `OpenDocument`).
- **User-Granted Document URIs**: Database backups are written exclusively to specific document URIs explicitly chosen by the user, keeping public storage isolated.

### 6. Complete Air-Gapped Network Isolation
- **Zero Network Permissions**: The Application does not declare `INTERNET` permission in `AndroidManifest.xml`.
- **Zero External Telemetry**: The database operates in a completely air-gapped environment. Transaction records, account balances, and receipt scans cannot be transmitted outside the device.

### 7. Optional SQLCipher AES-256 Database Encryption
- For enterprise-grade **Encryption-at-Rest**, the database can be transparently encrypted on disk using **SQLCipher for Android** (`net.zetetic:android-database-sqlcipher` / `androidx.sqlite:sqlite-framework`), which encrypts every SQLite database page with **256-bit AES-CBC encryption**.
