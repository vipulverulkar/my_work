# BharatLedger (भारतलेजर)

> A modern, GST-ready cross-platform accounting application built for Indian MSMEs (*Vyaparis*) and accounting professionals, adhering strictly to ICAI (Institute of Chartered Accountants of India) and MCA double-entry compliance rules.

Runs natively with zero external server dependencies on **Android**, **iOS**, **Linux (Ubuntu/Debian/Fedora)**, and **Windows**.

> **Documentation:** [User Manual](doc/UserManual.md) · [Architecture](doc/Architecture.md)

---

## Visual Showcase

| Dashboard & Real-Time KPIs | Inventory & Stock Movements |
| :---: | :---: |
| ![BharatLedger Dashboard](doc/screenshots/dashboard.png) | ![Inventory & Stock Register](doc/screenshots/inventory.png) |
| **Financial Statements & Aging** | **GST Tax Invoice & UPI QR** |
| ![Financial Reports](doc/screenshots/reports.png) | ![Tax Invoice Preview](doc/screenshots/tax_invoice_preview.png) |

---

## 1. Core Architecture & Tech Stack

- **Framework**: [Flutter](https://flutter.dev) (Single unified codebase across Mobile & Desktop)
- **Programming Language**: Dart (Strict type safety, null-safe)
- **Local Database Engine**: SQLite via `sqflite` (Mobile) and `sqflite_common_ffi` (Desktop)
  - Zero-configuration local deployment
  - Strict Foreign Key constraints (`PRAGMA foreign_keys = ON`)
  - Instant ACID transactions with local-first data privacy
- **State Management**: [Riverpod](https://riverpod.dev)
- **Design System**: Material Design 3 adaptive design with regional typography and high-density desktop navigation.
  - Primary: Deep Navy/Teal (`#1A365D`)
  - Secondary: Emerald Green (`#059669` for credits / receipts)
  - Accent: Crimson (`#DC2626` for debits / payments)

---

## 2. Directory Structure (Clean Architecture)

Following strict Clean Architecture boundaries separating UI, domain rules, and data sources:

```text
bharat_ledger/
├── android/                         # Android native configurations
├── ios/                             # iOS native configurations
├── windows/                         # Windows desktop configurations
├── linux/                           # Linux desktop configurations
│
├── assets/
│   ├── icons/                       # App icons and Indian currency symbols
│   ├── locales/                     # Bhasha translation files (en.json, hi.json, mr.json)
│   └── licenses/                    # Open-source licenses (GPL-3.0)
│
└── lib/
    ├── main.dart                    # Application entry point
    │
    ├── app/                         # App widget & localization delegates
    │
    ├── core/                        # Shared utilities across the app
    │   ├── constants/               # Colors, metadata, layout thresholds
    │   ├── database/                # SQLite schema, seed chart of accounts, database helper
    │   ├── errors/                  # Domain exceptions (AccountingMismatchException, etc.)
    │   ├── localization/            # Bhasha localization (English, Hindi, Hinglish, Marathi, Tamil, Gujarati)
    │   ├── theme/                   # Material 3 Navy/Teal/Emerald design system
    │   ├── utils/                   # FY helpers, Indian number formatting (Lakhs/Crores), GST math, UPI utilities
    │   └── widgets/                 # Adaptive modals, keyboard listeners, QR dialogs, badges
    │
    └── features/                    # Feature-driven modular slices
        ├── dashboard/               # Today sales/purchases, cash-in-hand vs bank, GST liability
        ├── vouchers/                # Sales, Purchases, Receipts, Payments, Contra, Journal, Credit/Debit Notes
        ├── ledgers/                 # Sundry Debtors, Creditors, Bank, Cash, Tax ledgers
        ├── inventory/               # HSN/SAC tracked goods & services, stock valuation
        ├── reports/                 # GSTR-1, GSTR-3B, GSTR-4 JSON/CSV, Day Book, Trial Balance
        ├── settings/                # Firm profile, Multi-firm management, Local & Encrypted backups
        └── shell/                  # Adaptive navigation (Desktop sidebar vs Mobile bottom bar)
```

---

## 3. Indian Accounting & Compliance Engine

### 1. Financial Year (FY) Logic
- Automated Indian FY parsing and labeling: **April 1st to March 31st** (e.g., `FY 2026-27`).
- Dedicated FY sequence numbering for every voucher type (e.g., `INV/2026-27/0001`).

### 2. Comprehensive GST Compliance
- **Multi-Tax Calculation**: Full support for CGST, SGST, IGST, and **UTGST** (for Union Territories without legislature: Chandigarh, Dadra & Nagar Haveli and Daman & Diu, Ladakh, Lakshadweep, Andaman & Nicobar, and Other Territory).
- **HSN/SAC Code Directory & Validation**: Built-in searchable directory with common goods (HSN) and services (SAC) codes, categories, and standard GST tax rates (0%, 5%, 12%, 18%, 28%).
- **GST Invoicing & PDF Generation Engine**:
  - Automatically generates **Tax Invoices** (under Rule 46 of CGST Rules) for taxable supplies.
  - Generates downloadable and printable **PDF Invoices & Bills** with customizable firm branding, buyer details, item tables, tax breakdowns, and UPI QR code.
  - Automatically generates **Bill of Supply** (under Rule 49 of CGST Rules) for composition dealers, unregistered suppliers, and exempt supplies.
  - Generates **Credit Notes**, **Debit Notes**, **Purchase Orders**, **Sales Orders**, and **Quotations**.
- **GSTR Export Engine**:
  - **GSTR-1**: Invoice-wise outward supplies grouped into B2B (with buyer GSTIN) and B2C summaries in official JSON and CSV formats.
  - **GSTR-3B**: Complete summary return calculating outward taxable supplies, eligible Input Tax Credit (ITC), and net GST payable with JSON and CSV exports.
  - **GSTR-4**: Annual composition return payload.
- **GSTIN Structural Verification**: Offline verification of the 15-character GSTIN format, state code, checksum, and PAN extraction (identifying Companies, Proprietorships, HUF, Partnerships, and Trusts).
- **TDS / TCS Compliance**:
  - TDS calculation under Sections 194C, 194J, 194I, 194H, 194A, 194Q, 194D with individual and company rate slabs.
  - One-click TDS deduction from payment vouchers.
- **Financial Reports & Statements**:
  - **Aging Report**: Receivables and Payables aging analysis categorized into 0-30, 31-60, 61-90, and >90 day brackets.
  - **Profit & Loss Statement (P&L)**: Revenue and Expense breakdowns by account group with net profit/loss computation.
  - **Balance Sheet**: Assets vs. Liabilities & Capital comparison with balanced status indicators.
  - **Trial Balance**: Complete debit/credit trial balance across all ledger accounts.
  - **Item-wise Stock Register**: Running movement history (IN, OUT, ADJUST) with unit rates and dates.

### 3. Double-Entry Invariant Guard
- Strict validation enforcing `Total Debits == Total Credits` down to the exact paisa before any voucher can be committed to the database.

---

## 4. High-Velocity Desktop & Mobile UX

- **Interactive Visual Charts (`fl_chart`)**:
  - Dynamic monthly sales trend bar charts with touch tooltips and currency formatting on Dashboard.
- **CSV Bulk Import**:
  - Bulk import Ledgers (name, group, GSTIN, opening balance) and Inventory items (name, HSN, rate, stock) from standard CSV files.
- **Dark Mode & Regional Localization**:
  - Theme mode toggle (System, Light, Dark) persisted across restarts.
  - Multilingual support for English, Hindi, Hinglish, Marathi, Tamil, and Gujarati.
- **Tally-Style Desktop Keyboard Workflows**:
  - `Alt + A`: Quick New Voucher entry from any view.
  - `Ctrl + S` / `Alt + S`: Save & commit active voucher.
  - `Esc`: Safely cancel or back out.
  - `Enter`: Advance through form input fields.
- **Dynamic NPCI UPI QR Codes**:
  - Invoices dynamically render a compliant NPCI QR code (`upi://pay?pa=...&pn=...&am=...&tr=...&cu=INR`).
  - Customers can scan directly from screen or invoice printouts using PhonePe, Google Pay, Paytm, or BHIM.
- **WhatsApp Payment Reminders**:
  - One-click trigger to open WhatsApp Web or native WhatsApp with pre-filled balance reminder text.
- **Indian Numbering Syntax**:
  - Formatted in Lakhs and Crores (e.g., `₹1,25,000.00`).
  - Legal amount-in-words generator for invoices (e.g., *"One Lakh Twenty-Five Thousand Rupees Only"*).
- **Multi-Firm Management**:
  - Switch contexts cleanly between multiple businesses (e.g., "Primary Firm", "Branch Pune", etc.) with completely isolated SQLite databases.
- **Local & Encrypted Vault Backups**:
  - High-speed zero-downtime backups via SQLite `VACUUM INTO`.
  - Password-protected encrypted vault exports (`.blbak`).
  - Cloud backup gateway seam for Google Drive / OneDrive sync.

---

## 5. Getting Started

### Prerequisites
- [Flutter SDK](https://docs.flutter.dev/get-started/install) (v3.13+ / 3.47+)
- Dart 3.0+

Platform toolchains for building installers:
- **Android** — Android SDK + Java 17; signing keystore for Play Store release.
- **Linux** — `clang`, `cmake`, `ninja-build`, `pkg-config`, `libgtk-3-dev`, `libblkid-dev`, `liblzma-dev`, `libsqlite3-dev`.
- **Windows** — Visual Studio 2022 with the "Desktop development with C++" workload and Windows 10/11 SDK.

### Run Locally

```bash
# Clone the repository
git clone https://github.com/vipulverulkar/Accounting_App.git
cd Accounting_App

# Fetch dependencies
flutter pub get

# Run test suite
flutter test

# Run application on Desktop (Linux / Windows)
flutter run -d linux
# or
flutter run -d windows

# Run on Mobile (Android / iOS)
flutter run -d android
```

A fresh install automatically seeds a demo company on first launch (parties,
items, and sample vouchers across all voucher types). Disable it for a blank
install:

```bash
flutter run -d linux --dart-define=SEED_DEMO=false
```

### Single-Click Setup & Launchers

| Platform | Command / Action | Description |
| --- | --- | --- |
| **Linux** | `./tool/install_and_launch_linux.sh` | Installs system dependencies, configures Flutter, builds, creates menu shortcut, & launches. |
| **Windows** | Double-click `tool/install_and_launch_windows.bat` or `.\tool\install_and_launch_windows.ps1` | Verifies C++ build tools, configures Flutter, builds, creates Desktop shortcut, & launches. |

### Build per platform

Each platform has its own build script. They run `pub get`, analyze, tests, build, and copy the installable artifact into `dist/`. Use `--fast` / `-SkipTests` to skip checks, or the debug flag for a debug build.

| Platform | Command | Output |
| --- | --- | --- |
| **Android** | `./tool/build_android.sh` | `dist/android/bharat_ledger_android_<version>_release.apk` and `.aab` |
| **Linux** | `./tool/build_linux.sh` | `dist/linux/bharat_ledger_linux_x64_<version>_release.tar.gz` |
| **Windows** | `powershell -ExecutionPolicy Bypass -File tool/build_windows.ps1` | `dist/windows/bharat_ledger_windows_x64_<version>_release.zip` |

Portable desktop packages:

| Package | Command | Output |
| --- | --- | --- |
| **Linux AppImage** | `./tool/package_linux.sh --appimage` | `dist/linux/*.AppImage` |
| **Linux Debian** | `./tool/package_linux.sh --deb` | `dist/linux/*.deb` |
| **Linux both** | `./tool/package_linux.sh` | AppImage + `.deb` |
| **Windows MSIX** | `powershell -ExecutionPolicy Bypass -File tool/package_windows_msix.ps1` | `dist/windows/*.msix` |

The Linux packaging script downloads `appimagetool` automatically when it is
not installed. The Debian package requires `dpkg-deb` (package `dpkg-dev`).
The Windows MSIX script uses `MakeAppx.exe` from the Windows 10/11 SDK and can
optionally sign the package with `-CertificatePath` and `-CertificatePassword`.

Common options:

```bash
./tool/build_android.sh --debug          # debug APK + AAB
./tool/build_android.sh --apk-only       # APK only
./tool/build_android.sh --aab-only       # Play Store AAB only
./tool/build_android.sh --fast           # skip tests + analyzer
./tool/build_linux.sh --debug
./tool/build_linux.sh --fast
```

```powershell
.\tool\build_windows.ps1 -DebugBuild
.\tool\build_windows.ps1 -SkipTests
```

Set `VERSION` to override the artifact label:

```bash
VERSION=v1.2.3 ./tool/build_linux.sh
```

> Android release builds require the standard Flutter signing setup (`android/key.properties` + keystore) for Play Store distribution. Unsigned/debug builds work without it. Linux AppImage/DEB and Windows MSIX packaging can reuse an existing release bundle with `--skip-build` / `-SkipBuild`.

---

## 6. Automated CI/CD & Cross-Platform Packaging

BharatLedger features enterprise-grade GitHub Actions CI/CD workflows producing production-ready distributable packages. The same build logic is available locally through the platform scripts in `tool/`:

- **Android (`tool/build_android.sh`)** — release/debug APK and AAB into `dist/android/`.
- **Linux (`tool/build_linux.sh`)** — release/debug bundle packaged as `.tar.gz` into `dist/linux/`.
- **Windows (`tool/build_windows.ps1`)** — release/debug bundle packaged as `.zip` into `dist/windows/`.

GitHub Actions workflows:

- **Android (`.github/workflows/android.yml`)**:
  - Compiles release APK (`app-release.apk`) & Google Play App Bundle (`app-release.aab`).
  - Supports automated Keystore base64 decoding and signing.
- **Linux (`.github/workflows/linux.yml`)**:
  - Sets up native C/C++ toolchain (`clang`, `cmake`, `ninja-build`, `libgtk-3-dev`, `libsqlite3-dev`).
  - Compiles standalone x64 release bundle and packages it as `bharat_ledger_linux_x64_<tag>.tar.gz`.
- **Windows (`.github/workflows/windows.yml`)**:
  - Compiles native Win32 release binary and dependencies.
  - Automatically packages the distribution into `bharat_ledger_windows_x64_<tag>.zip`.
- **Release Matrix (`.github/workflows/release.yml`)**:
  - Triggers on version tag push (`v*.*.*`).
  - Builds all three OS targets in parallel and automatically publishes release bundles to GitHub Releases.

---

## 7. License

This project is licensed under the **GNU General Public License v3.0 (GPL-3.0)**. See [LICENSE](LICENSE) for details.
