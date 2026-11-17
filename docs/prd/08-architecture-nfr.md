# 08 — Technical Architecture & Non-Functional Requirements

## 1. Pendekatan

**Rekomendasi desain:** modular monolith, satu codebase backend dan satu PostgreSQL, domain boundaries eksplisit, dependency internal lewat service/event terkontrol. Tidak perlu microservice, message broker besar atau ERPNext penuh untuk MVP; keputusan framework tetap **[OPEN]** sebelum implementasi.

```text
Web/PWA (Owner/Admin/Cashier)
          │ HTTPS
       API/BFF
          │
   Auth + RBAC + Store Scope
          │
 ┌────────┼──────────────┬─────────────┐
 │        │              │             │
Catalog  Procurement   Inventory      POS
 │        │              │             │
 └────────┴──────────────┴──────┬──────┘
                               │
                       Finance / Reports
                               │
                      Closing / Audit
                               │
                     PostgreSQL + Backup
```

Domain service bertanggung jawab atas write invariants; laporan membaca view/data terposting. Store scope melekat ke semua query data bisnis. Jangan membuat circular dependency: Procurement memanggil Inventory posting, POS memanggil Inventory posting, Finance menerima peristiwa terposting atau query read model.

## 2. Stack kandidat (belum final)

- **Frontend:** Next.js + TypeScript, Tailwind CSS, responsive tablet/mobile/desktop, PWA online-first.
- **Backend:** NestJS + TypeScript dengan modul per domain.
- **Database:** PostgreSQL; ORM Prisma atau Drizzle (evaluasi migration/transaction support).
- **Authentication:** server-managed sessions/httpOnly cookies atau JWT dengan refresh/rotation, RBAC policy middleware.
- **Deploy:** Docker di satu VPS atau managed platform dengan TLS, external backup, env secrets dan observability dasar.
- **Print:** browser print untuk MVP, dukungan ESC/POS printer thermal pada P1.
- **Alternatif ERPNext:** memungkinkan menggunakan Frappe ERPNext jika akuntansi penuh lebih mendesak, tetapi UI/operasional sederhana dan database ramping mendorong opsi custom. Jangan berasumsi kode saat ini sudah menentukan stack.

## 3. Kontrak antar modul

- Catalog mengelola identitas dan metadata produk; tidak menulis saldo stok.
- Procurement mengelola list, receipt, payment pembelian; meminta Inventory mencatat IN setelah verify.
- Inventory menjadi pemilik ledger dan invariant saldo nonnegatif; tidak menagih pelanggan.
- POS mengelola sale/payment dan memicu Inventory OUT dalam transaksi DB yang aman.
- Finance menyusun laporan dari transaksi posted, expense dan cash movement; tidak memodifikasi sale.
- Audit menyimpan histori sensitif dan workflow closing; tidak mengubah ledger tanpa adjustment resmi.
- Configuration menyimpan fitur aktif, threshold variance, tax/discount policy bila nanti diperlukan.

## 4. Nonfunctional requirements

| ID | Aspek | Target minimum MVP |
| --- | --- | --- |
| NFR-001 | Konsistensi | DB transaction/locking mencegah double posting, oversell dan inkonsistensi sale/payment/stock |
| NFR-002 | Idempotensi | Semua endpoint post/confirm/void/payment mendukung key atau unique source |
| NFR-003 | Security | HTTPS, password hash, RBAC server-side, store scoping, sanitasi input, validasi schema |
| NFR-004 | Audit | Semua aksi sensitif beraktor, timestamp dan alasan; posted ledger append-only |
| NFR-005 | Responsivitas | UI dapat dipakai pada layar ponsel dan tablet; tombol POS cukup besar |
| NFR-006 | Performa | Target p95 katalog/search dan response POS normal < 500ms pada pilot LAN/internet sehat, diukur dan disesuaikan |
| NFR-007 | Availability | Target awal 99% per bulan untuk operasional online (target awal, bukan SLA komersial) |
| NFR-008 | Backup | Backup harian terenkripsi, retensi 14 hari minimal; uji restore sebelum produksi |
| NFR-009 | Recovery | RPO ≤ 24 jam, RTO ≤ 8 jam sebagai baseline kecil; evaluasi bila bisnis makin bergantung |
| NFR-010 | Observability | Log struktur, request/correlation ID, error tracking, health checks |
| NFR-011 | Accessibility | Form berlabel, kontras memadai, keyboard flow dasar dan pesan error jelas |
| NFR-012 | Privacy | PII minimal; customer anonim default; tidak menyimpan bukti sensitif sembarangan |
| NFR-013 | Localization | IDR, bahasa Indonesia, kalender Asia/Jakarta; UTC di database |
| NFR-014 | Deployment | Environment dev/staging/prod; migration terkontrol; secret tidak masuk repository |
| NFR-015 | Testing | Unit domain invariants, integration DB, permission, e2e checkout & closing |

## 5. Secure operations

- Rate-limit login dan endpoint sensitif; anti-CSRF jika cookie session; CORS dikonfigurasi ketat.
- QRIS static verification harus memakai sumber merchant yang tepercaya; screenshot pelanggan hanya bukti pendukung, bukan final.
- Tidak menyimpan kredensial merchant, password mentah, atau seluruh payload pembayaran sensitif dalam log.
- Backup diuji dengan restore, bukan hanya keberhasilan job backup.
- Audit logs dapat menggunakan append-only privilege dan retention policy; penanganan data pribadi mengikuti hukum yang berlaku setelah legal review.
- Feature flags untuk opsi kompleks (negative stock, discount, split payment, multi-store); default paling aman.
- Hak Owner luas tetapi tindakan terposting tetap immutable dan terdeteksi.

## 6. Evolusi arsitektur

Phase 1: satu store, satu DB, API REST, online-first.  
Phase 2: barcode/thermal, QRIS gateway dan bank reconciliation, stock count sesi, multi-store membership.  
Phase 3: ekstraksi modul/service **hanya** bila bottleneck terukur; POS offline perlu outbox, operation IDs, conflict resolution, inventori reservation, dan sync replay tests sebelum dijanjikan.

## 7. Pilihan yang menunggu keputusan

Hosting/cloud provider, bentuk session auth, ORM, skema nomor invoice, stok dengan satuan pecahan, biaya pembelian tambahan dan model QRIS (statis/manual vs dynamic gateway) ditandai [OPEN] dan harus diputuskan sebelum build fitur terkait.
