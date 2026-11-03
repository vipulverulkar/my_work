# BharatLedger User Manual

**Version:** 0.1.0  
**Applies to:** Android, iOS, Windows, and Linux builds  
**Audience:** Business owners, accountants, and data-entry operators

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [System Requirements](#2-system-requirements)
3. [Install and Run](#3-install-and-run)
4. [First-Time Setup](#4-first-time-setup)
5. [Navigation and Keyboard Shortcuts](#5-navigation-and-keyboard-shortcuts)
6. [Dashboard](#6-dashboard)
7. [Ledgers](#7-ledgers)
8. [Inventory Items](#8-inventory-items)
9. [Vouchers and Invoices](#9-vouchers-and-invoices)
10. [Reports and GST Returns](#10-reports-and-gst-returns)
11. [Settings](#11-settings)
12. [Backups, Restore, and Data Location](#12-backups-restore-and-data-location)
13. [Troubleshooting](#13-troubleshooting)
14. [Frequently Asked Questions](#14-frequently-asked-questions)
15. [Appendix A: Keyboard Shortcuts](#appendix-a-keyboard-shortcuts)
16. [Appendix B: CSV Import Formats](#appendix-b-csv-import-formats)
17. [Appendix C: Build From Source](#appendix-c-build-from-source)
18. [Appendix D: Compliance Notes and Limitations](#appendix-d-compliance-notes-and-limitations)

---

## 1. Introduction

BharatLedger is a local-first, GST-ready accounting application for Indian
MSMEs and accounting professionals. It runs without an external server and
stores each firm's data in its own SQLite database on the device.

Key capabilities:

- Double-entry accounting with debit/credit validation
- Indian financial year (1 April to 31 March) and FY-wise voucher numbering
- GST invoicing with CGST, SGST, UTGST, IGST, and Cess
- GSTIN validation, HSN/SAC lookup, place of supply, and reverse charge
- Sales, purchases, receipts, payments, contra, journal, credit/debit notes
- Bill of Supply, Export Invoice, Delivery Challan, orders, and quotations
- GSTR-1, GSTR-3B, and GSTR-4 exports
- Aging, Profit & Loss, Balance Sheet, and Trial Balance
- Inventory with HSN/SAC, stock movements, weighted-average valuation
- TDS calculation and TDS payment posting
- UPI QR codes, PDF invoices, and WhatsApp payment reminders
- Local and encrypted backups, multi-firm management, and audit trail
- English, Hindi, Hinglish, Marathi, Tamil, and Gujarati interface

Screenshots are available in the [screenshots folder](screenshots/).

---

## 2. System Requirements

### Android
- Android phone or tablet
- Android version supported by the installed Flutter SDK
- Roughly 100 MB free storage
- Internet is not required for normal accounting use

### iOS
- iPhone or iPad
- iOS version supported by the installed Flutter SDK
- A build signed with an Apple developer certificate

### Windows
- Windows 10 or Windows 11, 64-bit
- 200 MB free disk space
- Microsoft Visual C++ runtime (normally installed with Windows)
- For running from source: Visual Studio 2022 with the **Desktop development
  with C++** workload and the Windows 10/11 SDK
- For MSIX packaging: Windows 10/11 SDK (`MakeAppx.exe`) and optionally
  `signtool.exe` with a code-signing certificate

### Linux
- Ubuntu, Debian, or Fedora, 64-bit
- GTK 3 runtime libraries
- `libsqlite3` runtime
- For running from source: `clang`, `cmake`, `ninja-build`, `pkg-config`,
  `libgtk-3-dev`, `libblkid-dev`, `liblzma-dev`, and `libsqlite3-dev`
- For AppImage packaging: `curl` (appimagetool is downloaded automatically)
- For Debian packaging: `dpkg-deb` (package `dpkg-dev`)

---

## 3. Install and Run

### 3.1 Android

#### Option A: Install an APK directly

1. Obtain the release APK from the build output or a release download:
   `dist/android/bharat_ledger_android_<version>_release.apk`
2. Copy the APK to the phone or tablet.
3. Open the APK using the device file manager.
4. If prompted, allow **Install unknown apps** for the file manager or browser.
5. Tap **Install**, then **Open**.

Using `adb` from a computer:

```bash
adb install -r dist/android/bharat_ledger_android_<version>_release.apk
adb shell monkey -p com.bharatledger.bharat_ledger 1
```

#### Option B: Play Store / internal testing

Upload the `.aab` file from `dist/android/` to the Google Play Console.
An AAB cannot be installed directly on a phone.

#### Run on a connected Android device from source

```bash
flutter pub get
flutter run -d android
```

### 3.2 Windows

1. Obtain or build the Windows ZIP:
   `dist/windows/bharat_ledger_windows_x64_<version>_release.zip`
2. Extract the entire ZIP to a folder. Do not run the EXE from inside the ZIP.
3. Open the extracted folder and run `bharat_ledger.exe`.
4. If Windows SmartScreen appears, choose **More info** and then
   **Run anyway** only when you trust the build.

The folder also contains required DLLs and the `data` folder. Keep all of
them together when moving the application.

#### MSIX package

If you have the signed MSIX package:

```powershell
Add-AppxPackage .\bharat_ledger_windows_x64_<version>.msix
```

Then launch **BharatLedger** from the Start menu. An unsigned MSIX requires a
trusted code-signing certificate before Windows will install it.

Run from source:

```powershell
flutter pub get
flutter run -d windows
```

### 3.3 Linux

1. Obtain or build the Linux archive:
   `dist/linux/bharat_ledger_linux_x64_<version>_release.tar.gz`
2. Extract it:

```bash
mkdir -p ~/BharatLedger
tar -xzf dist/linux/bharat_ledger_linux_x64_<version>_release.tar.gz \
  -C ~/BharatLedger
```

3. Run the application:

```bash
cd ~/BharatLedger
./bharat_ledger
```

If the executable bit is missing:

```bash
chmod +x bharat_ledger
```

#### AppImage

The AppImage requires the GTK 3 and SQLite runtime libraries listed in
[System Requirements](#2-system-requirements).

```bash
chmod +x bharat_ledger_<version>_x86_64.AppImage
./bharat_ledger_<version>_x86_64.AppImage
```

#### Debian package

```bash
sudo apt install ./bharatledger_<version>_amd64.deb
```

The package installs the application under `/opt/bharatledger` and adds a
`bharatledger` launcher and desktop entry.

Install runtime libraries on Debian/Ubuntu if needed:

```bash
sudo apt update
sudo apt install -y libgtk-3-0 libblkid1 liblzma5 libsqlite3-0
```

Run from source:

```bash
flutter pub get
flutter run -d linux
```

### 3.4 iOS (optional)

iOS builds require macOS and Xcode.

```bash
flutter pub get
flutter run -d ios
```

If CocoaPods dependencies are used, Flutter will prompt you to run
`pod install` inside `ios/` during the first build.

For distribution, open `ios/Runner.xcworkspace` in Xcode, configure your Apple
developer team and bundle identifier, then create an archive.

### 3.5 Build & single-click setup scripts

The repository includes build entry points and single-click installer/launchers per platform.

#### Single-click setup & launchers:
- **Linux**: `./tool/install_and_launch_linux.sh` (installs system dependencies, configures Flutter, compiles release, creates menu shortcut, and launches)
- **Windows**: Double-click `tool/install_and_launch_windows.bat` or run `powershell -ExecutionPolicy Bypass -File tool/install_and_launch_windows.ps1` (verifies Visual Studio C++ build tools, compiles release, creates Desktop shortcut, and launches)

#### Build scripts:

| Platform | Command | Output |
| --- | --- | --- |
| Android | `./tool/build_android.sh` | `dist/android/*.apk` and `*.aab` |
| Linux | `./tool/build_linux.sh` | `dist/linux/*.tar.gz` |
| Windows | `powershell -ExecutionPolicy Bypass -File tool/build_windows.ps1` | `dist/windows/*.zip` |
| Linux AppImage | `./tool/package_linux.sh --appimage` | `dist/linux/*.AppImage` |
| Linux DEB | `./tool/package_linux.sh --deb` | `dist/linux/*.deb` |
| Windows MSIX | `powershell -ExecutionPolicy Bypass -File tool/package_windows_msix.ps1` | `dist/windows/*.msix` |

Options:

```bash
./tool/build_android.sh --debug       # debug APK + AAB
./tool/build_android.sh --apk-only    # APK only
./tool/build_android.sh --aab-only    # Play Store AAB only
./tool/build_android.sh --fast        # skip analyzer and tests
./tool/build_linux.sh --debug
./tool/build_linux.sh --fast
```

```powershell
.\tool\build_windows.ps1 -DebugBuild
.\tool\build_windows.ps1 -SkipTests
.\tool\package_windows_msix.ps1 -SkipBuild
.\tool\package_windows_msix.ps1 -CertificatePath .\cert.pfx -CertificatePassword "secret"
```

Set `VERSION=<label>` before a Bash build to control the artifact filename.

> Android release builds for the Play Store require a signing keystore and
> `android/key.properties`. Debug builds work without signing.

---

## 4. First-Time Setup

On first launch, open **Settings** and complete the firm profile.

1. Open **Settings** from the sidebar, the top bar, or the mobile menu.
2. Enter the **Business name**. This is the name printed on invoices.
3. Enter the **Legal / trade name** if different.
4. Enter the 15-character **GSTIN**, if registered.
5. Select the **State**. The state determines CGST/SGST versus IGST.
6. Enter the business **Address**, **Phone**, and **Email**.
7. Enter the **UPI VPA** to enable UPI QR codes on invoices.
8. Set the **Invoice prefix** (default `INV`).
9. If the business is under the composition scheme, enable
   **Composition scheme dealer**. Sales vouchers are then created as
   **Bill of Supply** by default.
10. Tap **Save settings**.

The financial year is calculated automatically from the voucher date. Voucher
numbers use the format `PREFIX/FY/0001`.

On a fresh install, BharatLedger **automatically seeds a demo company** on
first launch so every device (Android, iOS, Windows, and Linux) starts with
sample parties, items, and vouchers. The seed is recorded in the database and
only runs once per firm; it is skipped when the active firm already has
vouchers or has already been seeded.

To start with a blank company instead, build or run with:

```bash
flutter run -d linux --dart-define=SEED_DEMO=false
```

You can also load the demo company manually from
**Settings → Data Tools → Demo Data**. Demo data can only be loaded when the
current firm has no vouchers.

---

## 5. Navigation and Keyboard Shortcuts

### Desktop and large screens
- Persistent left sidebar: Dashboard, Vouchers, Ledgers, Items, Reports,
  Settings.
- The top bar shows the current section, firm name, financial year, language
  selector, refresh button, and Settings button.

### Mobile and small screens
- Bottom navigation: Dashboard, Vouchers, Ledgers, Items, Reports.
- Floating **Quick Invoice** button opens a new voucher.
- Settings are available from the top bar.

### Keyboard shortcuts (Windows/Linux)

| Shortcut | Action |
| --- | --- |
| `Alt + A` | New voucher from any main screen |
| `Ctrl + S` or `Alt + S` | Save the active voucher |
| `Esc` | Cancel or go back |
| `Enter` | Move to the next form field |

### Top-bar actions
- **Language** — switch interface language.
- **Refresh** — reload data from the local database.
- **Settings** — open firm and application settings.

---

## 6. Dashboard

The Dashboard gives a quick view of the business:

- **Today's sales** and **today's purchases** with counts
- **Cash in hand** and **bank balance**
- **Receivables** and **payables**
- **GST liability** for the current month, including output tax and input credit
- **Sales trend** for the last six months
- **Recent vouchers**

Quick action buttons create Sales, Purchase, Receipt, and Payment vouchers, or
open Reports.

---

## 7. Ledgers

Ledgers are the accounts used in vouchers: customers, suppliers, cash, bank,
sales, purchases, tax accounts, and expense accounts.

### Add a ledger

1. Open **Ledgers**.
2. Tap **Add ledger**.
3. Enter the ledger **Name**.
4. Select an **Account group**, for example:
   - Sundry Debtors — customers
   - Sundry Creditors — suppliers
   - Bank Accounts, Cash-in-Hand
   - Direct/Indirect Income and Expenses
   - Duties & Taxes
5. Enter the **GSTIN**, if applicable. State and PAN details are verified
   locally.
6. Select the **State**.
7. Enter **Phone** and **Billing address**. These are used on invoices and
   printouts.
8. Enter the **Opening balance** and choose **Dr** or **Cr**.
9. Tap **Save**.

Party ledgers are those in **Sundry Debtors** or **Sundry Creditors**. They
appear in invoice party dropdowns.

### Edit or delete
Use the row menu to edit or delete a ledger. A ledger used in vouchers cannot
be deleted until those vouchers are removed.

### Send a payment reminder
For party ledgers, choose **Send reminder** to open WhatsApp with a pre-filled
message containing the ledger name and outstanding balance.

---

## 8. Inventory Items

Items represent goods and services sold or purchased.

### Add an item

1. Open **Items**.
2. Tap **Add item**.
3. Enter the item **Name**.
4. Enter the **HSN/SAC code**. Use the HSN lookup button to search by keyword
   or code.
5. Optionally enter an **SKU**.
6. Select the **Unit of measure** (PCS, KGS, BOX, NOS, LTR, MTR, etc.).
7. Select the **GST rate** (0%, 0.25%, 3%, 5%, 12%, 18%, 28%).
8. Enter the **Cess rate**, if applicable.
9. Enter **Purchase price** and **Sales price**.
10. Enter **Opening stock**.
11. Enable **This is a service / SAC item** for services. Services do not
    change stock.
12. Tap **Save**.

### Stock register and status
Use **Stock register** on an item row to see movement history (IN, OUT,
ADJUST), rates, dates, and voucher references. Items are flagged as
**Out of stock** or **Low stock** in the item list.

---

## 9. Vouchers and Invoices

Open **Vouchers** to list, search, filter, create, view, print, or delete
vouchers. The voucher type selector includes:

| Voucher type | Purpose |
| --- | --- |
| Sales | Tax invoice for outward supplies |
| Purchase | Purchase bill / inward supply |
| Receipt | Money received |
| Payment | Money paid |
| Contra | Cash-to-bank or bank-to-cash transfer |
| Journal | Multi-line adjustment entries |
| Credit Note | Sales return or sales reduction |
| Debit Note | Purchase return or purchase reduction |
| Bill of Supply | Exempt/non-GST or composition supplies |
| Export Invoice | Zero-rated export supply |
| Delivery Challan | Stock movement without accounting entries |
| Purchase Order, Sales Order, Quotation | Non-posting business documents |

### 9.1 Create a Sales or Purchase invoice

1. Open **Vouchers → New voucher** (or press `Alt + A`).
2. Select **Sales** or **Purchase**.
3. Choose the **Date**.
4. Select the **Party**.
5. For sales, confirm the **Place of supply**.
6. Add **Narration** if required.
7. For purchases, optionally enable **Reverse charge (RCM)**.
8. For sales, select the **Supply category**:
   - Taxable
   - Exempt
   - Nil Rated
   - Zero Rated
   - Export
   - SEZ Supply
9. Enter **Payment terms** and **Due date** if required.
10. Add one or more item lines:
    - Select the item
    - Enter quantity
    - Adjust the rate if needed
11. Review the GST breakup: taxable value, CGST, SGST or UTGST, IGST, Cess,
    round-off, and grand total.
12. Tap **Save voucher**.

The application validates that total debits equal total credits before saving.
GST is split automatically:

- Same state as the firm → CGST + SGST
- Union Territory without legislature → CGST + UTGST
- Different state → IGST

### 9.2 Reverse charge (RCM) purchases

For purchases where the buyer must pay GST directly to the government:

1. Select **Purchase**.
2. Enable **Reverse charge (RCM)**.
3. Add the item or service lines.
4. Save.

The purchase value is credited to the supplier, input tax is debited, and the
tax is credited to **RCM Payable** instead of being paid to the supplier. The
purchase is included in GSTR-3B section 3.1(d).

### 9.3 Bill of Supply, Export Invoice, and Delivery Challan

- **Bill of Supply** — used for exempt/non-GST supplies and composition
  dealers. No GST is charged.
- **Export Invoice** — zero-rated export document. GST is not charged.
- **Delivery Challan** — records stock movement only. It does not create
  accounting entries or appear in GST returns.

### 9.4 Credit Notes and Debit Notes

- **Credit Note** reverses a sale: debits the sales return/account and output
  taxes, credits the customer.
- **Debit Note** reverses a purchase: debits the supplier, credits the
  purchase/account and input taxes.

### 9.5 Receipts, Payments, Contra, and TDS

For **Receipt** and **Payment**, select the two ledgers and enter the amount.
For **Contra**, select the cash/bank accounts.

On a **Payment**, the **TDS** button opens the TDS calculator. Select the
section, enter the gross amount, choose whether the deductee is a company, and
indicate whether PAN is available. Apply the calculation to post:

- Debit: expense/party ledger (gross)
- Credit: bank/cash ledger (net)
- Credit: TDS Payable (TDS amount)

Supported sections include 194C, 194J, 194J technical, 194I land/building,
194I plant/machinery, 194H, 194A, 194Q, 194D, 194IA, 194IB, 194M, 194N,
and 194O. Thresholds and the higher 20% rate for missing PAN are applied.

### 9.6 Journal vouchers

A journal voucher has two or more lines. For each line select a ledger, choose
**Debit** or **Credit**, and enter the amount. The screen shows total debit,
total credit, and whether the voucher is balanced. A journal cannot be saved
unless debits equal credits.

### 9.7 Print, PDF, UPI QR, and WhatsApp

Open a saved voucher to:

- View the GST invoice preview
- Generate a PDF and print it
- Show a UPI QR code for the invoice amount
- Send a WhatsApp payment reminder for sales vouchers

The PDF includes firm details, buyer details, GSTIN, place of supply,
reverse-charge status, HSN/SAC, tax breakup, amount in words, declaration,
payment terms, due date, and a UPI QR code when a UPI VPA is configured.
Bill of Supply, Export Invoice, and Delivery Challan documents use their own
titles.

### 9.8 Search, filter, view, and delete

- Search by voucher number, party, or narration.
- Filter by voucher type using the chips.
- Open a voucher row to view entries, item lines, tax details, and actions.
- Deleting a voucher is a soft delete: balances and stock are reversed, the
  voucher disappears from lists and reports, and an audit record is retained.

---

## 10. Reports and GST Returns

Open **Reports** and choose a date range:

- This Month
- This Quarter
- This Financial Year
- Custom Range

### 10.1 GSTR-1
Outward supply return data:

- B2B invoices by GSTIN
- B2C Small and B2C Large
- Credit and debit notes (registered and unregistered)
- Exports
- Nil, exempt, and zero-rated supplies
- HSN summary
- Document series summary

Export as JSON or CSV. The JSON structure follows the GSTR-1 sections; always
validate against the GST portal schema before filing.

### 10.2 GSTR-3B
Summary return data:

- Outward taxable supplies
- Zero-rated, exempt, and nil supplies
- Inward supplies liable to reverse charge
- Eligible ITC
- Net tax payable after ITC

Export as JSON or CSV.

### 10.3 GSTR-4
Annual composition return summary. Intended for composition dealers. Export
as JSON.

### 10.4 Aging Report
Receivables and payables are matched bill-wise using FIFO settlement and
grouped into:

- 0–30 days
- 31–60 days
- 61–90 days
- Over 90 days

Due dates are used when entered on the voucher; otherwise the voucher date is
used.

### 10.5 Profit & Loss
Shows operating revenue, cost of goods sold, gross profit, other income,
expenses, opening stock, closing stock, and net profit. Closing stock uses the
weighted-average purchase rate where purchase history exists.

### 10.6 Balance Sheet
Shows assets, liabilities, capital, current-year profit, and derived closing
stock. The status indicator shows whether total assets equal total liabilities.

### 10.7 Trial Balance
Lists ledgers with debit and credit balances and totals. Export as CSV.

### 10.8 Exporting reports
Use **Export JSON** or **Export CSV** in the report tabs. Files are saved in
the application documents folder under `BharatLedger/exports`. The application
shows the saved path and allows copying it.

---

## 11. Settings

### 11.1 Firm profile
Edit business name, legal name, GSTIN, state, address, phone, email, UPI VPA,
invoice prefix, and composition-scheme status.

### 11.2 Language
Choose English, Hindi, Hinglish, Marathi, Tamil, or Gujarati. The selection is
saved and restored on the next launch.

### 11.3 Appearance
Choose System, Light, or Dark mode. The selection is saved.

### 11.4 Multi-firm management
Use the firm selector to switch between businesses. Each firm has a separate
SQLite database, so ledgers, items, vouchers, and settings are isolated. Use
**Add new firm** to create a new company.

### 11.5 Data tools
- **Local backup** — creates a `.db` copy using SQLite `VACUUM INTO`.
- **Encrypted export** — creates a password-protected `.blbak` vault using
  PBKDF2-HMAC-SHA256 and AES-256-GCM.
- **Restore backup** — imports a `.blbak` vault as a separate firm.
- **Demo data** — loads a sample company into an empty firm.
- **Database location** — shows the exact path of the active firm database.

Encrypted backup passphrases must be at least 8 characters. Store the
passphrase separately; it cannot be recovered.

### 11.6 CSV import
Import ledgers or items in bulk from CSV files. See
[Appendix B](#appendix-b-csv-import-formats).

### 11.7 About
Shows version, developer, license, and third-party license information.

---

## 12. Backups, Restore, and Data Location

BharatLedger is local-first. There is no cloud account requirement.

Recommended practice:

1. Create a local backup after major data entry.
2. Create an encrypted backup before moving devices.
3. Keep backups on a separate drive or personal cloud storage.
4. Test a restore before relying on a backup.

Restoring an encrypted vault adds a new firm. Switch to it from
**Settings → Multi-firm**.

The active database path is shown in **Settings → Data Tools**. On Linux it is
typically under `~/.local/share/<application-id>/BharatLedger/`. On Windows it
is under the user's application data folder. Always use the path shown in the
app instead of guessing.

---

## 13. Troubleshooting

### The app does not start
- Confirm the platform requirements are installed.
- On Linux, install GTK 3 and SQLite runtime libraries.
- On Windows, keep the EXE, DLLs, and `data` folder together.
- Run from source with `flutter run -d <platform>` to see detailed errors.

### Android says "App not installed"
- The APK may be corrupted or signed with a different key. Rebuild or obtain a
  fresh APK.
- Uninstall an older build with the same package name if the signing key
  changed.
- Enable installation from unknown sources for the file manager.

### Windows SmartScreen warning
The build is not code-signed. Use **More info → Run anyway** only for builds
you trust, or sign the executable for distribution.

### Linux says a shared library is missing
Install the GTK 3 and SQLite runtime packages listed in
[System Requirements](#2-system-requirements).

### GSTIN is rejected
A valid GSTIN must be 15 characters, start with a valid state code, contain a
valid PAN pattern, and pass the checksum. Check for a typo or a missing
character.

### HSN/SAC is rejected
HSN/SAC codes must be numeric and 4, 6, or 8 digits long.

### A voucher will not save
The most common causes are:

- Total debits do not equal total credits.
- No party or item line was added.
- System ledgers are missing. Restart the app so they are re-created.
- A Cess or RCM ledger is missing.
- The same debit and credit ledger was selected in a simple voucher.

### Balance Sheet shows a difference
The report includes current-year profit and derived closing stock. Differences
usually mean opening balances or stock were entered outside the app. Review
opening balances and inventory opening stock.

### Reports show no data
- Check the report date range.
- Confirm vouchers were saved, not left as drafts.
- Deleted vouchers are excluded from reports.

---

## 14. Frequently Asked Questions

**Does BharatLedger need internet?**
No, for normal accounting. Internet is used only for optional actions such as
WhatsApp reminders or external integrations.

**Where is my data stored?**
In a local SQLite database. The exact path is shown in Settings → Data Tools.

**Can I use multiple businesses?**
Yes. Each firm has its own isolated database.

**Can I import existing data?**
Yes, ledgers and inventory items can be imported from CSV. Vouchers must be
entered in the app.

**Does it support composition dealers?**
Yes. Enable the composition scheme in the firm profile. New sales vouchers
default to Bill of Supply and GST is not charged.

**Does it support reverse charge?**
Yes, on purchase vouchers. The tax is posted to RCM Payable and reported in
GSTR-3B.

**Does it file GST returns automatically?**
No. It prepares JSON/CSV data for GSTR-1, GSTR-3B, and GSTR-4. Upload the
exported data through the GST portal or your filing utility after validation.

**Are e-invoice and e-Way Bill supported?**
The app includes a configurable IRP/NIC gateway client. Live use requires
valid IRP/NIC credentials or a NIC-compliant signing middleware.

**Can I recover a forgotten backup passphrase?**
No. The encrypted vault uses the passphrase to derive the encryption key.

**Does deleting a voucher remove it permanently?**
No. Deletion is a soft delete with balance and stock reversal. An audit record
is kept.

---

## Appendix A: Keyboard Shortcuts

| Shortcut | Action |
| --- | --- |
| `Alt + A` | New voucher |
| `Ctrl + S` | Save voucher |
| `Alt + S` | Save voucher |
| `Esc` | Cancel / go back |
| `Enter` | Next field |
| `Tab` | Next control |

---

## Appendix B: CSV Import Formats

### Ledger CSV

Required column: `name`

```csv
name,group_name,gstin,state_code,phone,opening_balance
Sharma Traders,Sundry Debtors,27AAPFU0939F1ZV,27,9876543210,0
Shree Distributors,Sundry Creditors,27ABCDE1234F1Z0,27,9898989898,0
```

Notes:
- `group_name` should match an existing account group. If omitted or not
  found, the import uses Sundry Debtors.
- `opening_balance` is a number. Use a negative value for credit balances if
  required by your workflow.
- Column headers are case-insensitive and spaces become underscores.

### Inventory CSV

Required column: `name`

```csv
name,hsn_sac_code,sku,unit_of_measure,gst_rate,purchase_price,sales_price,opening_stock
Basmati Rice 5kg,1006,RICE5,PCS,5,380,450,20
AC Repair Service,9987,ACSERV,HRS,18,0,800,0
```

Notes:
- `unit_of_measure` defaults to `PCS` when omitted.
- `gst_rate` defaults to 18 when omitted.
- Imported items are treated as goods. Create services manually or edit after
  import.

---

## Appendix C: Build From Source

### Prerequisites
- Flutter SDK (stable channel)
- Dart 3.0 or later
- Platform toolchain from [System Requirements](#2-system-requirements)

### Run

```bash
flutter pub get
flutter test
flutter analyze
flutter run -d linux      # or windows / android / ios
```

### Build with the platform scripts

```bash
./tool/build_android.sh          # APK + AAB
./tool/build_linux.sh            # tar.gz
./tool/package_linux.sh          # AppImage + DEB
powershell -ExecutionPolicy Bypass -File tool/build_windows.ps1
powershell -ExecutionPolicy Bypass -File tool/package_windows_msix.ps1
```

Artifacts are written to `dist/android`, `dist/linux`, and `dist/windows`.

The Linux build script checks for `clang`, `cmake`, `ninja`, `pkg-config`,
GTK 3, `libblkid`, `liblzma`, and SQLite development packages and prints
distribution-specific install commands when something is missing.

The Windows build script checks for Visual Studio 2022 with the C++ desktop
workload and the Windows 10/11 SDK, and prints installation instructions when
they are missing. The MSIX script additionally requires `MakeAppx.exe`.

Demo data is seeded automatically on first launch. To build a blank production
app, pass the Dart define through Flutter:

```bash
flutter build linux --release --dart-define=SEED_DEMO=false
flutter build windows --release --dart-define=SEED_DEMO=false
flutter build apk --release --dart-define=SEED_DEMO=false
```

### Seed a demo database from the command line

```bash
flutter test tool/seed_demo.dart
```

To target a specific database:

```bash
BHARAT_DB_PATH=/path/to/bharat_ledger.db flutter test tool/seed_demo.dart
```

---

## Appendix D: Compliance Notes and Limitations

- BharatLedger enforces double-entry balance at the application and database
  layers.
- GST rates, cess, GSTIN checksum, financial year, and voucher numbering follow
  standard Indian conventions.
- GSTR exports are prepared for filing workflows but should be validated
  against the current GST portal schema before submission.
- Live e-invoicing and e-Way Bill generation require IRP/NIC credentials or a
  compliant signing gateway.
- The application is a record-keeping and return-preparation tool. It is not a
  substitute for professional tax or audit advice.
- Voucher editing is available through the data/repository layer; the entry
  screen currently focuses on creating new vouchers. Corrections can be made by
  deleting and re-entering a voucher, with audit records retained.
- Accountants should review opening balances, stock valuation, ITC
  eligibility, and reverse-charge treatment before filing returns.
