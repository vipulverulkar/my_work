# Architecture — Sweet Shop POS

Requirements for the system, and how the shipped app implements them.
The application is a single **Flask + SQLite** backend (`app.py` + `pos/`)
serving a **React 19** frontend (`web/`), optionally wrapped as an Electron
desktop shell or a Docker container.

## 1. Core catalog & item matrix

Support distinct item types and unit logics:

- **Loose weight items** (Kaju Katli, Gulab Jamun, Ladoo, Halwa):
  grams/kilograms entry (e.g. 0.500 kg or 250 g) with auto-calculated pricing
  based on price-per-kg.
- **Piece/count items** (Samosa, Kachori, Jalebi per piece or plate).
- **Pre-packed & barcoded goods** (sealed namkeen packets, branded gift tins):
  barcode scanner input with batch/expiry tracking.
- **Custom assorted gift boxes** (Box Builder UI):
  - pick an empty box size (500g, 1kg);
  - tap-to-add multiple loose sweets;
  - track cumulative weight capacity, deduct per-sweet inventory, and add
    standard or custom packing/box charges.

## 2. Checkout & billing workflow

- **Speed-first UI:** touch-friendly, grid-based quick-select menu categorized
  by Mithai (Dry/Milk), Namkeen, Bakery, Beverages and Festive Hampers.
- **Order types:** Counter/Takeaway, Dine-in, Bulk/Party orders, and
  Quick Delivery/Aggregator status (Zomato/Swiggy-style board).
- **Payment modes:** Cash (auto change due), Card, UPI, Split, store credit.
- **Receipts:** print-ready layouts for 3-inch, 4-inch and A4 printers; optional
  digital receipt (WhatsApp/Email).
- **Hold & resume:** park a rush bill and recall it instantly.

## 3. Inventory & production (back office)

- **Dual ledger:** separate POS retail sales from kitchen/production batch
  yields — raw materials (sugar, khoya, ghee, flour, packaging) are tracked
  apart from finished counter stock.
- **Recipe / Bill of Materials:** link finished sweets to ingredient recipes to
  compute real-time material consumption and cost-per-batch.
- **Expiry & waste:** log spoilage, tasting, staff meals and wastage with tags.
- **Low-stock & expiry alerts:** warnings for short-shelf-life items (milk
  sweets expiring within 24–48 h) and items at/below reorder level.

## 4. Role-based access control & audit

- **Roles:** Owner, Store Manager, Cashier, Kitchen.
  - Cashiers bill, hold, and process standard returns.
  - Managers apply discounts, handle drawer overrides, and adjust stock counts.
  - Owners have full access to P&L, supplier ledgers and settings.
- **Audit trail:** every bill modification, price override, void and cash-drawer
  event is recorded in `audit_logs`.

## 5. Compliance & reporting

- **Tax engine:** configurable GST slabs (5% / 12% / 18%) with HSN codes on
  invoices.
- **Reports:** end-of-day Z-report & cash reconciliation, fast- vs slow-moving
  items, wastage-vs-sales, GST and payment-mode summaries, owner P&L.

## Implementation map

| Concern | Where |
|---|---|
| Data model | `schema.sql` (SQLite); `pos/database.py` seeds + idempotent migrations |
| Pricing / gift-box math | `pos/pricing.py` (Decimal-precise) |
| Hardware | `pos/hardware.py` (scale parser, ESC/POS, drawer kick) |
| API + order/inventory logic | `app.py` |
| UI | `web/` (React 19 + Vite), `web/electron/` desktop shell |
| Run | `run.sh` / `run.bat` / Docker (`Dockerfile`, `docker-compose.yml`) |
