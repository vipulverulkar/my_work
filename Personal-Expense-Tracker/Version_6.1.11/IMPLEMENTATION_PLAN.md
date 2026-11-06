# Mobile App Feature Parity & Release Implementation Plan

Comprehensive technical implementation plan for building, upgrading, and releasing the Personal Expense Tracker Android App.

---

## 🏛️ Phase 1: Feature Parity & Architecture Alignment

### 1. Database v4 Schema & Data Layer
- **New Entities**: `RuleEntity` (`transaction_rules`), `DebtEntity` (`debts`), `SharedExpenseEntity` (`shared_expenses`), `NotificationEntity` (`notifications`), `ImportTemplateEntity` (`import_templates`).
- **New DAOs**: `RuleDao`, `DebtDao`, `SharedExpenseDao`, `NotificationDao`, `ImportTemplateDao`.
- **AppDatabase v4**: Registered all 11 entities and DAOs with version migration support.

### 2. Domain & Repository Modules
- **Automated Transaction Rules**: RegEx pattern matching against note or merchant fields to auto-assign categories and tags.
- **Debt Payoff Engine**: Principal tracking, Avalanche vs. Snowball payoff strategy simulators, repayment transactions linked to paying accounts (`is_savings = 1`).
- **Group Shared Expenses**: Participant split shares and single/bulk settlements.
- **Financial Goals**: Sinking fund targets, progress tracking, and deposit contributions.
- **Accounting Statements**: Balance Sheet ($Assets = Liabilities + Equity$), Profit & Loss, Statement of Cash Flows, Double-Entry Trial Balance, and Tax Summary.
- **In-App Notification Alerts**: Notification center tracking budget warning thresholds and bill due dates.
- **Trash Bin Recovery**: Soft-delete listing (`deleted_at`), restore action, and permanent purge action.

---

## 🚀 Phase 2: System Features & UX Upgrades

1. **Biometric Security & App Lock**: Integrated `androidx.biometric:biometric` for Fingerprint / Face unlock prompt with `FragmentActivity` lifecycle integration.
2. **Glance Home Screen App Widget**: Home screen widget built with Jetpack Glance (`androidx.glance:glance-appwidget`) displaying live Net Worth overview.
3. **Native PDF Report Export**: Native A4 PDF document generator using Android's `PdfDocument` engine for Balance Sheet, P&L, and Tax Summary reports.
4. **Custom Financial Year Configuration**: Configurable Financial Year starting month (`setFyStartMonth`) with `getFinancialYearBounds()` calculating dynamic fiscal year ranges.
5. **Interactive Trend Charts in Analytics**: Canvas-drawn dual-color bar/trend chart visualizing 6-month Income vs. Expense trajectories.
6. **Launcher App Shortcuts**: Shortcuts for *"Add Expense"*.
7. **Voice Quick Add**: Speech recognition (`RecognizerIntent.ACTION_RECOGNIZE_SPEECH`) parsing spoken sentences into amount, category, and note fields.

---

## 📦 Phase 3: Play Store Release Configuration

1. **Release Signing**: Configured `signingConfigs.release` in `build.gradle.kts` to read keystore properties.
2. **Privacy Policy**: Generated `PRIVACY_POLICY.md` detailing 100% on-device processing, SMS disclosures, and Data Safety declaration.
3. **App Bundle (`.aab`) Compilation**: Compiled signed Android App Bundle (`app-release.aab`) via `./gradlew app:bundleRelease`.

---

## 🌟 Phase 4: v6.1.10 Usability & Data Onboarding Polish

1. **Material 3 Calendar DatePicker (`DatePickerField`)**:
   - Replaced manual text typing for dates across all dialogs and sheets (`AddEditTransactionSheet`, `AddRecurringDialog`, `RecordPaymentDialog`, `AddGoalDialog`, `ContributeGoalDialog`, `SettleShareDialog`).
   - Standardized ISO `yyyy-MM-dd` storage with user-friendly calendar picker modal.
2. **Full Account & Net Worth Management**:
   - Implemented Account CRUD (`addAccount`, `updateAccount`, `deleteAccount`) in `AccountDao` and `ExpenseRepository`.
   - Built interactive `AddAccountDialog` with auto-prefilling for edit mode.
   - Added Net Worth card and edit/delete actions with confirmation safety dialogs on `AccountsScreen`.
3. **First-Launch Setup & Dummy Data Notification**:
   - On first installation database creation, an in-app system notification is recorded alerting users of pre-seeded showcase data.
   - Prominently displays a First-Time Setup card on the Dashboard offering 1-tap **"Delete Dummy Data"** or **"Keep / Dismiss"**.
   - Added smart sample detection ensuring existing users with real data updating the app are never prompted.
4. **Transactions 3-Dots Menu Optimization**:
   - Removed duplicate "Auto-Pick SMS" option from the Transactions screen menu for a cleaner user experience.

---

## 🎯 Phase 5: v6.1.11 Milestone & Release Initialization

1. **Version Centralization**:
   - Advanced centralized semantic version to `6.1.11` (Android `versionCode 6111`) across backend, desktop, mobile app, and documentation.
2. **Quality Verification**:
   - Verified 100% test pass rate across both Android test harness and backend test suite.

