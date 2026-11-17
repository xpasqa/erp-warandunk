# 11 — User Stories, Acceptance Criteria & Test Matrix

**Tujuan:** setiap requirement mempunyai hasil observabel, role yang tepat, dan bukti untuk QA. Format Given/When/Then dapat dikonversi menjadi automated tests.

## 1. User stories P0

| Story ID | Sebagai | Saya ingin | Agar |
| --- | --- | --- | --- |
| US-001 | Owner | Membuat dan mengekspor shopping list | Pembelian dapat dilakukan tanpa akun rider |
| US-002 | Cashier | Menginput realisasi belanja dan mengunggah struk wajib | Setiap penerimaan memiliki bukti sebelum diverifikasi |
| US-003 | Owner/Admin | Review bukti struk, harga dan qty lalu approve/reject | Stok hanya bertambah setelah persetujuan |
| US-004 | Owner/Admin | Mencatat settlement pembelian terpisah | Stok dan pembayaran tidak dicampur |
| US-005 | Cashier | Menjual cepat via cash atau QRIS | Pembayaran dan stok tercatat |
| US-006 | Cashier | Mencetak struk opsional | Customer mendapat bukti |
| US-007 | Admin | Menutup dan memverifikasi kas harian | Selisih dapat ditelusuri |
| US-008 | Owner | Melihat stock ledger dan dashboard | Kontrol stok dan pendapatan |
| US-009 | Owner | Mengontrol role dan aktivitas sensitif | Akses sesuai tanggung jawab |
| US-010 | Admin | Mencatat biaya operasional | Arus kas dan laba indikatif lebih akurat |

## 2. Acceptance scenarios P0

| Test ID | Given | When | Then |
| --- | --- | --- | --- |
| AC-001 | Owner login | Buat shopping list berisi 2 SKU dan 1 free-text | List tersimpan DRAFT dengan requested qty |
| AC-002 | List DRAFT | Export | Daftar readable/printable mencantumkan item, qty, note |
| AC-003 | List ON_PROCESS | Rider membeli barang di luar sistem | Tidak perlu user rider; status tetap merepresentasikan progres |
| AC-004 | Receipt DRAFT qty 24 | Lihat saldo | Stok belum bertambah |
| AC-005 | Cashier telah submit receipt disertai struk valid | Owner/Admin approve | Tepat satu stock movement IN per line; receipt APPROVED_POSTED |
| AC-006 | Receipt sudah approved | Retry approve | Response idempotent, tidak ada tambahan stok |
| AC-007 | Sebagian item diterima | Posting receipt parsial | List PARTIALLY_RECEIVED, remaining qty terlihat |
| AC-008 | Item bebas belum ada produk | Admin onboard saat receiving | Produk baru dengan kategori/SKU/price tersedia |
| AC-009 | Receipt posted tetapi belum dibayar | Admin lihat settlement | UNPAID; stok sudah ada |
| AC-010 | Purchase payment dari kas | Admin konfirmasi pembayaran | Satu cash OUT, outstanding berkurang |
| AC-011 | Purchase payment dari dana owner pribadi | Admin konfirmasi | Tidak mengurangi laci kas |
| AC-012 | Product aktif, on-hand 2 | Cashier menambah 2 ke keranjang | Total harga benar |
| AC-013 | Item harga Rp5.000 x2 dan Rp9.000 x1 | Checkout | Server total Rp19.000 |
| AC-014 | Total Rp19.000 | Bayar cash Rp20.000 | Sale PAID, change Rp1.000, stock OUT sesuai item |
| AC-015 | Sale QRIS belum merchant verified | Cashier mencoba finalisasi tanpa validasi | Sale bukan PAID, stock tidak OUT |
| AC-016 | Merchant payment confirmed | Authorized verifier posts checkout | Sale PAID sekali; QRIS tidak masuk cash drawer |
| AC-017 | Saldo satu unit | Dua checkout simultan qty satu | Hanya satu boleh sukses; saldo tidak negatif |
| AC-018 | Checkout timeout sesudah sukses | Retry key/payload sama | Satu sale, satu payment, satu stock OUT |
| AC-019 | Cashier terautentikasi | Akses harga modal/profit | Ditolak 403 |
| AC-020 | Cashier aktif | Panggil endpoint purchase-receipts | Ditolak 403 |
| AC-021 | Harga jual master berubah | Buka sale paid lama | Harga line historis tidak berubah |
| AC-022 | WAC cost Rp3.000, terjual 2 unit | Laporan HPP | HPP Rp6.000; stok sisa masih dinilai sebagai persediaan |
| AC-023 | Sale cash Rp19.000, QRIS Rp9.000 | Hitung expected drawer | Bertambah hanya Rp19.000 |
| AC-024 | Opening 100k, cash sales 250k, cash out 50k, counted 295k | Submit closing | Expected 300k, variance −5k |
| AC-025 | Closing variance bukan nol | Submit tanpa reason | Validasi menolak |
| AC-026 | Closing submitted oleh Cashier | Cashier mencoba approve sendiri | Ditolak |
| AC-027 | Closing approved | Edit expected cash secara PATCH | Ditolak; memerlukan koreksi terkontrol |
| AC-028 | Sale PAID | Admin void tanpa alasan/permission | Ditolak |
| AC-029 | Sale PAID, approved void dan refund dilakukan | Proses reversal | Stock IN dan cash refund tercatat; laporan net benar |
| AC-030 | Product on-hand 10, fisik 8 | Adjustment -2 disetujui | Movement ADJUSTMENT -2 dengan reason dan approver |
| AC-031 | QRIS confirmed Senin, settled Selasa | Filter berdasarkan tanggal transaksi vs settlement | Revenue Senin, settlement Selasa |
| AC-032 | User Store A | Minta data Store B | 403/404 tanpa kebocoran data |
| AC-033 | Owner melihat audit trail | Filter void dan stock adjust | Actor, reason, referensi, timestamp tersedia |
| AC-034 | Produk nonaktif tetapi pernah dijual | Filter history | Transaksi lama tetap dapat ditampilkan |
| AC-035 | Ada dua kasir dan dua session | Admin melihat daily report | Agregasi akurat tanpa menggandakan pembayaran |

## 3. Test layers

1. **Unit/domain:** state transition, nilai negatif, WAC, penjumlahan line, discount, cash variance, boundaries.
2. **Integration DB:** constraint uniqueness, row locking, atomic receipt post, atomic paid checkout, rollback on failure, concurrency.
3. **API/RBAC:** unauthorized/forbidden/out-of-store, validation, idempotency, malformed input, duplicate document.
4. **End-to-end:** onboarding produk → list → receipt → sale → close → report; browser printing smoke test.
5. **Operational rehearsal:** pilot satu hari, stok opening jelas, QRIS merchant verification, closing variance, restore backup.

## 4. Definition of Done untuk setiap fitur

- Requirement ID dan acceptance test terhubung ke issue/PR.
- API divalidasi schema, permission dan store scope; error codes terdokumentasi.
- Migrasi database aman, ada test rollback/forward dan constraint untuk invariant penting.
- Event terposting tercatat dengan actor/time/source; retry idempotent.
- Unit + integration + e2e relevan lulus di CI; review keamanan dasar.
- UI loading/error/empty state ada dan dapat dipakai mobile.
- Perhitungan kas, stok, dan laporan diuji terhadap contoh manual.
- Dokumen PRD/API diperbarui jika perilaku berubah.

## 5. Exit criteria pilot MVP

Tidak ada double sale, over-selling, mismatch sale vs ledger, missing audit untuk aksi sensitif, QRIS false-paid tanpa verifikasi, atau closing yang tidak dapat di-drill-down. Owner/Admin menyetujui contoh laporan pendapatan, stok dan HPP dari hari operasional nyata. Backup diuji bisa dipulihkan.

## 6. Tambahan skenario purchase approval

| Test | Given | When | Then |
| --- | --- | --- | --- |
| AC-036 | Cashier login | Create shopping list | 403; hanya Owner/Admin boleh |
| AC-037 | Cashier input receipt | Submit tanpa upload struk | 422; inventory tidak berubah |
| AC-038 | Cashier upload struk JPEG/PNG/PDF valid | Submit receipt | SUBMITTED, file privat, belum ada stock IN |
| AC-039 | Receipt SUBMITTED oleh Cashier | Cashier memanggil approve | 403, stok tetap |
| AC-040 | Owner/Admin membandingkan item dengan struk dan approve | Approve & Post | Movement IN sekali, approved_by dan waktu tersimpan |
| AC-041 | Struk tidak sesuai qty/harga | Admin reject dengan alasan | Cashier melihat alasan, boleh revisi/resubmit |
| AC-042 | Receipt APPROVED_POSTED | Cashier edit item/struk | Ditolak; hanya reversal resmi |
| AC-043 | File struk milik store lain | User meminta akses | 403/404, tidak bocor |
| AC-044 | File invalid MIME/ukuran | Upload | Ditolak sebelum submit |
| AC-045 | Usulan harga jual dari Cashier | Belum approved | Harga jual aktif tidak berubah |
