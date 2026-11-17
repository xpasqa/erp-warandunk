# 12 — Implementation Roadmap & Prioritized Backlog

**Strategi:** kerjakan berdasarkan dependensi domain agar tidak membangun POS di atas stok yang salah. Durasi bukan komitmen; estimasi ditetapkan setelah desain final, kapasitas tim dan integrasi diputuskan.

## 1. Dependency graph

```text
Foundation (DB, Auth, Store, RBAC)
      ↓
Catalog / Categories / Product
      ↓
Shopping List → Cashier Entry + Struk Upload → Owner/Admin Approval → Stock Ledger / WAC
      ↓                     ↓
      └───────────────→ POS + Session + Payments
                              ↓
                      Cash Flow / Expenses
                              ↓
                      Daily Close / Audit
                              ↓
                    Reports / Pilot / Release
```

## 2. Tahapan

### M0 — Discovery & design freeze

- Finalisasi open decisions prioritas tinggi dan skema role.
- Wireframe POS/receiving/closing; review status transitions.
- ERD detail & migrasi versi 1, OpenAPI draft, acceptance scenarios.
- Exit: domain ownership, proses uang/stok dan WAC disetujui.

### M1 — Technical foundation (P0)

- Repo application structure, dev/staging/prod configs, Docker, migrations, seed.
- Auth/login/logout, session, store scoping, RBAC, audit middleware.
- Basic test harness/CI, health endpoint, structured logs.
- Exit: unauthorized operations blocked, seed repeatable, migrations tested.

### M2 — Procurement & inventory (P0)

- Category/product CRUD, SKU unique, prices, min stock, metadata.
- Shopping list draft/active/export oleh Owner/Admin.
- Purchase receipt realisasi oleh Cashier, wajib upload struk, submit/reject/resubmit.
- Review bukti dan approval Owner/Admin; hanya approve memicu stock IN, sekaligus onboarding produk.
- Stock movement ledger, on-hand projection, weighted average cost, stock opening.
- Purchase payment settlement by source.
- Exit: AC-001 to AC-011, AC-017, AC-021, AC-022 relevant tests pass.

### M3 — POS operational (P0)

- Cashier shift, catalog search, cart, server total, cash checkout/change.
- QRIS manual merchant verification, payment confirmation, idempotency.
- Stock OUT atomic, receipt printing, transaction history, controlled full void.
- Exit: AC-012 to AC-020, AC-028/29 tests pass; tablet smoke test.

### M4 — Finance & audit (P0)

- Expense/cash movement, finance read models.
- Session submit-close, admin approve, variance reason, daily closing.
- Cash/QRIS/stock report and QRIS settlement status tracking.
- Audit trail filtering, export basic, period reconciliation.
- Exit: AC-023 to AC-027, AC-031 to AC-035 and operational close test pass.

### M5 — Pilot & stabilization

- Enter opening inventory and starting cash; reconcile against real records.
- Validate user permissions with Owner/Admin/Cashier.
- Verify backup restore, idempotency and concurrent checkout.
- Fix high-severity findings, prepare operational SOP and release.
- Exit: MVP exit criteria fulfilled and Owner signs off.

## 3. Prioritized backlog

| ID | Feature | Priority | Dependency |
| --- | --- | --- | --- |
| B-001 | Auth, RBAC, store scope | P0 | None |
| B-002 | Category/product master | P0 | B-001 |
| B-003 | Shopping list & export | P0 | B-002 |
| B-004 | Cashier receiving input + mandatory struk + Owner/Admin approve → stock IN | P0 | B-002/B-003 |
| B-005 | Stock movement & on-hand/WAC | P0 | B-002 |
| B-006 | Purchase settlement | P0 | B-004 |
| B-007 | POS cart & paid cash | P0 | B-005/B-001 |
| B-008 | QRIS merchant-confirm workflow | P0 | B-007 |
| B-009 | POS session & cash movement | P0 | B-007 |
| B-010 | Void/reversal & audit trail | P0 | B-007/B-005 |
| B-011 | Daily closing and cash variance | P0 | B-009 |
| B-012 | Reports sales, stock, finance | P0 | B-005/B-007/B-011 |
| B-013 | Expenses & cash source reconciliation | P0 | B-009 |
| B-014 | Print browser receipt | P0 | B-007 |
| B-015 | Barcode/scanner | P1 | B-002/B-007 |
| B-016 | Dedicated thermal print | P1 | B-014 |
| B-017 | Stock count sessions & partial refund | P1 | B-005/B-010 |
| B-018 | QRIS gateway + settlement batch | P1 | B-008 |
| B-019 | Multiple stores / membership UI | P1 | B-001 |
| B-020 | Customers & loyalty | P2 | B-007 |
| B-021 | Bundling/recipes/expiry | P2 | B-002/B-005 |
| B-022 | Offline transaction queue/sync | P2 | B-007 + strong sync design |

## 4. Tugas nonkode

- Tentukan apakah kasir wajib membuka sesi setiap shift.
- Tentukan siapa memegang uang saat pembelian (kas toko/owner pribadi).
- Tentukan apakah QRIS statis manual dapat diterima untuk pilot.
- Siapkan daftar SKU, harga modal awal dan saldo stock opening.
- Pilih proses verifikasi QRIS serta SOP jika merchant app tidak tersedia.
- Tentukan personel yang memverifikasi closing jika Owner/Admin bekerja sendiri.
- Siapkan printer/browser/device, koneksi internet dan rencana backup.

## 5. Release gates

Setiap milestone memiliki: demo alur, reproducible tests, migrasi, rollback plan, RBAC review, audit log, update dokumentasi dan sign-off user operasional. Feature yang belum valid tidak boleh diam-diam dinyalakan di production.
