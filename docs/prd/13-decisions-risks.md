# 13 — Architecture Decision Log, Assumptions & Risks

**Aturan:** keputusan berstatus `PROPOSED` masih dapat berubah; `ACCEPTED` membutuhkan konfirmasi Owner/Product Lead. Jangan menganggap rekomendasi teknis di sini sudah final.

## 1. Decision log

| ID | Status | Decision/proposal | Rasional |
| --- | --- | --- | --- |
| ADR-001 | PROPOSED | Custom modular monolith, bukan ERPNext untuk MVP | UI warung lebih sederhana, kendali domain dan database kecil |
| ADR-002 | PROPOSED | PostgreSQL sebagai system of record | Relasional, transaksi ACID, constraint, JSONB |
| ADR-003 | PROPOSED | Next.js/TypeScript frontend dan NestJS backend | Stack typed, modular dan ekosistem luas |
| ADR-004 | PROPOSED | Stock movement append-only + optional balance projection | Traceability dan concurrency |
| ADR-005 | PROPOSED | WAC per produk/store sebagai costing awal | Lebih sederhana dari batch FIFO dan cukup untuk produk retail |
| ADR-006 | PROPOSED | Cash/QRIS POS, QRIS statis manual merchant verification | Mengurangi integrasi eksternal awal, menjaga verifikasi |
| ADR-007 | PROPOSED | Satu store MVP dengan store_id pada data | Jalur ekspansi tanpa kompleksitas UI cabang |
| ADR-008 | PROPOSED | Customer anonim default | Customer account belum memberi nilai MVP |
| ADR-009 | PROPOSED | Browser-print receipt dan online-first PWA | Operasional ringan dan biaya integrasi rendah |
| ADR-010 | PROPOSED | Operational finance, bukan general ledger penuh | Laporan dasar cash/inventory/COGS tanpa ERP accounting besar |
| ADR-011 | PROPOSED | Approved transactions reversible only, not mutable | Audit dapat dipercaya |
| ADR-012 | PROPOSED | Owner/Admin membuat shopping list, Cashier input dan wajib upload struk; satu approval Owner/Admin memicu stock IN | Maker-checker tanpa double entry |
| ADR-013 | PROPOSED | Bukti struk dalam private object storage | Keamanan dan audit bukti |

## 2. Open questions yang perlu dijawab sebelum implementasi terkait

| ID | Prioritas | Pertanyaan | Default sementara |
| --- | --- | --- | --- |
| OQ-001 | HIGH | Build custom atau menjadikan ERPNext basis? | Custom modular monolith |
| OQ-002 | HIGH | QRIS statis manual atau dynamic gateway sejak awal? | Manual merchant verification |
| OQ-003 | HIGH | Apakah seluruh Admin boleh membuat shopping list? | Boleh, Owner dapat mematikan permission |
| OQ-004 | HIGH | Berapa batas selisih kas yang butuh Owner escalation? | Semua variance butuh alasan; threshold belum ada |
| OQ-005 | HIGH | Bagaimana cash drawer dan pembayaran dari dana pribadi Owner dipisah? | Catat funding source eksplisit |
| OQ-006 | HIGH | Apakah diskon dan full void Admin memerlukan approval Owner? | Admin terbatas, void wajib alasan |
| OQ-007 | HIGH | Apakah checkout QRIS perlu bukti referensi merchant wajib? | Verifikasi merchant wajib; reference optional jika statis |
| OQ-008 | MED | Bagaimana kebijakan expense vs landed cost? | Biaya lain sebagai expense hingga disepakati |
| OQ-009 | MED | Apakah satuan pecahan (kg/liter) dibutuhkan? | Qty NUMERIC siap, UI default pcs |
| OQ-010 | MED | Kasir blind count atau melihat expected cash saat submit? | Expected disimpan server, UI policy belum final |
| OQ-011 | MED | Multi-session/kasir per outlet serentak? | Ya, masing-masing shift sendiri |
| OQ-012 | MED | Printer receipt 58mm/80mm dan dukungan bluetooth? | Browser print dulu |
| OQ-013 | MED | Hosting, backup retention dan RPO/RTO? | Target awal NFR, perlu validasi biaya |
| OQ-014 | MED | Refund/retur parsial diperlukan saat rilis? | Full void dengan izin P0, partial refund P1 |
| OQ-015 | MED | Apakah harga sudah mencakup pajak dan biaya lain? | Tidak ada perhitungan pajak khusus pada MVP |
| OQ-016 | MED | Nama outlet, format SKU, invoice numbering, branding struk? | Configurable default |
| OQ-017 | LOW | Program loyalty/customer database? | P2 |
| OQ-018 | HIGH | Pembatasan self-approval saat Admin sendirian? | Cashier submit dan Owner/Admin approve; tidak ada self-approval |
| OQ-019 | HIGH | Apakah Owner wajib approve final setelah Admin verifikasi? | MVP cukup satu approval Owner atau Admin; mandatory Owner adalah konfigurasi lanjutan |
| OQ-020 | MED | Retensi struk dan kondisi penjual tidak memberi struk? | MVP struk JPEG/PNG/PDF wajib, batas 10 MB |

## 3. Risk register

| Risk ID | Risiko | Dampak | Mitigasi |
| --- | --- | --- | --- |
| R-001 | Kasir menandai QRIS lunas padahal belum masuk merchant | Kehilangan uang | Verifikasi pada merchant app, referensi dan audit; bank settlement terpisah |
| R-002 | Concurrent checkout oversells last stock | Stok negatif dan laporan salah | DB locking dan integration concurrency test |
| R-003 | Timeout menghasilkan transaksi ganda | Double charge dan double stock OUT | Idempotency key, unique posting source |
| R-004 | Biaya pembelian dianggap HPP total hari itu | Laba salah | Snapshot WAC/COGS saat sale, inventory valuation |
| R-005 | Role hanya disembunyikan via UI | Kebocoran harga modal/pengaturan | Server-side RBAC dan store scoping |
| R-006 | Audit dapat dihapus | Tidak bisa investigasi | Append-only log, controlled reversal |
| R-007 | QRIS settlement bank terlambat | Cash flow keliru | Report receivable/pending dan tanggal settlement |
| R-008 | Perangkat kasir atau internet gagal | Operasional terganggu | Online-first + SOP manual yang direkonsiliasi; offline P2 |
| R-009 | Kegagalan backup | Kehilangan data | Encrypted backups + restore drill |
| R-010 | Custom fields jadi field bisnis kritis di JSONB | Query/report sulit | Kolom relasional untuk atribut utama |
| R-011 | Approval orang yang sama menutupi selisih | Risiko fraud | Separation of duties, owner exception and logging |
| R-012 | Scope creep ke booking lapangan/full accounting | Rilis terlambat | Tegaskan out-of-scope dan gate perubahan |
| R-013 | Bukti struk hilang/berubah setelah submit | Audit pembelian lemah | Private file storage, checksum, attachment history |
| R-014 | Qty/harga Cashier berbeda dari struk | Stok/modal salah | Review struk berdampingan dan reject dengan alasan |

## 4. Kebijakan penanganan open decisions

Setiap open question dicatat bersama pilihan, dampak tabel/API dan kapan harus diputuskan. Sebelum implementasi OQ terkait, status ADR harus diperbarui menjadi ACCEPTED/REJECTED dengan tanggal dan penanggung jawab. Perubahan kebijakan posting/payments wajib memperbarui data model, tests, dan dokumentasi SOP.

## 5. Catatan regulasi

Persyaratan hukum perlindungan data, penyimpanan nota, pajak lokal, dan penggunaan integrasi QRIS melalui penyelenggara resmi perlu diverifikasi terhadap kondisi usaha saat penerapan. Dokumen ini tidak menggantikan nasihat akuntansi/hukum profesional.
