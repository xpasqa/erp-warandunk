# 01 — Product Brief & Scope

**Status:** Draft v0.1 | **Prioritas:** P0 untuk MVP

## 1. Latar belakang

WARANDUNK melayani transaksi jual-beli di warung yang beroperasi dalam lingkungan GOR badminton. Pengelolaan sewa lapangan berada pada sistem/proses lain dan **sengaja tidak disentuh**. Masalah operasional yang ingin diselesaikan adalah daftar belanja tidak terstruktur, stok masuk/keluar sulit ditelusuri, harga modal tidak terdokumentasi dengan baik, rekap penjualan cash/QRIS manual, dan sulitnya pembuktian selisih kas/stok saat tutup hari.

## 2. Product vision

Satu sumber kebenaran operasional untuk produk, pembelian, persediaan, penjualan, kas, dan audit dengan tampilan cepat untuk operator nonteknis. Inti domain dirancang dapat digunakan lagi untuk warung lain, retail sederhana, kantin, minimarket kecil, atau bisnis F&B kemasan.

### Sasaran

- Owner dapat mengawasi transaksi serta mengakses laporan tanpa harus berada di kasir.
- Admin dapat menerima barang, memasukkan harga aktual, menyelesaikan pertanggungjawaban pembelian, dan menutup kas harian.
- Cashier dapat melakukan checkout barang dalam langkah minimal, dengan pembatasan akses.
- Setiap perubahan kuantitas dan uang dapat ditelusuri ke dokumen sumber serta aktor.
- Sistem tidak bergantung pada CRM/customer login untuk transaksi walk-in.

## 3. Ruang lingkup

**In scope MVP (P0):** login/RBAC; kategori/master produk termasuk SKU, unit, harga jual; daftar belanja dan ekspor oleh Owner/Admin; status belanja; input penerimaan parsial/total dan struk wajib oleh Cashier; approval Owner/Admin sebelum stok masuk; harga beli per penerimaan; stock ledger; stok saat ini dan stok minimum; POS keranjang dan pembayaran cash/QRIS manual terverifikasi; struk opsional; cash session; cash in/out; biaya operasional; closing harian; laporan penjualan, kas, QRIS, HPP, margin kasar, stok, pembelian; audit log; konfigurasi dasar; penyaringan berdasarkan store.

**P1 pasca-MVP:** barcode scanner; supplier directory; retur penjualan; pembelian kredit/utang yang lebih lengkap; stock opname berbasis sesi; QRIS dinamis/gateway; dukungan printer thermal dedicated; notifikasi stok minimum; ekspor CSV/XLSX yang kaya; dukungan multi-cabang operasional.

**P2:** customer loyalty, diskon berbasis campaign, bundle, resep/bahan baku, batch/expired date, offline POS dengan sinkronisasi, integrasi akuntansi penuh, pajak sesuai konfigurasi, purchase-order/supplier approval multi-level.

**Out of scope eksplisit:** rental/booking lapangan, membership badminton, gaji/HR/payroll, full double-entry general ledger, pengiriman/kurir atau tracking rider, marketplace, manajemen produksi dapur, pricing otomatis oleh AI.

## 4. Pengguna dan situasi utama

- **Owner:** membuat daftar belanja, memverifikasi receipt Cashier, approve/reject stock posting, mengatur kebijakan, melihat laporan dan audit.
- **Admin:** membuat shopping list, memeriksa struk yang diunggah Cashier, menyetujui/menolak receipt, settlement pembelian, pengelolaan katalog dan verifier closing.
- **Cashier:** input realisasi belanja (qty, harga beli, usulan harga jual) dan unggah struk wajib, submit untuk approval; juga transaksi POS dan closing shift. Tidak bisa approve atau menambah stok langsung.
- **Customer:** pembeli tanpa akun, opsional akun di fase nanti.
- **Rider/Pembelanja:** bukan aktor aplikasi pada MVP; status `ON_PROCESS` cukup untuk mewakili barang sedang dibeli.

## 5. Alur bisnis end-to-end

`Shopping List (Owner/Admin) → Cashier Purchase Entry + Mandatory Receipt Upload → SUBMITTED → Owner/Admin Approve → Stock Movement IN → POS Sale (PAID) → Stock Movement OUT + Payment + COGS Snapshot → POS Session Closing → Daily Reconciliation → Reports`

Settlement biaya pembelian **terpisah** dari stock posting: barang dapat diterima hari ini tetapi uang baru diselesaikan besok. Pembayaran pembelian menghasilkan cash-out hanya jika benar-benar dibayar dari kas yang dikelola di sistem.

## 6. KPI dan ukuran keberhasilan

| KPI | Definisi | Target awal |
| --- | --- | --- |
| Traceability | % transaksi posted dengan actor, source, timestamp | 100% |
| Valid payment | % sale paid dengan payment yang sah dan jumlah tepat | 100% |
| Stock consistency | Ketidakcocokan sale paid vs stock movement | 0 |
| Daily closing coverage | % sesi kasir yang ditutup dengan rekonsiliasi | Target operasional [OPEN] |
| Cash variance | Selisih kas aktual vs ekspektasi | Tercatat dan diberi alasan 100% |
| Time to checkout | Dari pilih item ke struk | Baseline diukur saat pilot |
| Stock availability | Jumlah SKU di bawah minimum | Dipantau harian |

## 7. Asumsi operasional

- Satu outlet dan satu mata uang (`IDR`) saat peluncuran; desain menggunakan `store_id`.
- Semua transaksi menggunakan timezone operasional `Asia/Jakarta`; timestamps disimpan UTC.
- Semua harga jual dapat berubah, tetapi item historis menyimpan snapshot.
- Harga awal, stok awal, dan data pengguna diinput melalui setup/stock opening yang tercatat.
- Cashier tidak dapat memverifikasi pembayaran QRIS hanya dari screenshot pelanggan.
- Mode POS online-first. Offline sinkronisasi kompleks tidak masuk MVP.

## 8. Batasan keberhasilan rilis

MVP tidak boleh dinyatakan siap produksi jika masih memungkinkan: stok negatif karena race condition; penjualan ganda akibat retry; audit hilang akibat delete; QRIS belum dibayar tetapi ditandai PAID tanpa langkah verifikasi; harga historis berubah setelah edit katalog; closing cash tanpa pemisahan QRIS; atau akses cashier ke seluruh laporan/konfigurasi.

## 9. Pengelolaan kebutuhan

Tag requirement: `AUTH-`, `CAT-`, `PROC-`, `INV-`, `POS-`, `FIN-`, `AUD-`, `CFG-`, `NFR-`. Prioritas P0/P1/P2 menentukan rilis, bukan penghapusan kebutuhan. Semua asumsi yang belum diputuskan dicatat di [13-decisions-risks.md](13-decisions-risks.md).
