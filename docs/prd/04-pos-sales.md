# 04 — Point of Sale & Sales Requirements

## 1. Tujuan

POS cepat di HP/tablet/desktop untuk penjualan retail walk-in. Customer login **tidak diperlukan**. Semua penjualan terkait satu `store_id`, cashier actor, sesi kasir, payment, dan movement persediaan.

## 2. Alur checkout

1. Cashier login dan membuka `pos_session` dengan opening float bila wajib.
2. Cari produk berdasarkan nama, kategori, SKU/barcode (scanner P1); tampilkan stok, harga jual dan status aktif.
3. Tambah item ke keranjang, ubah qty, hapus item; backend menghitung ulang harga dan total secara otoritatif.
4. Pilih pembayaran **CASH** atau **QRIS**. Diskon manual hanya sesuai permission; pajak/service fee belum aktif default.
5. Cash: masukkan nominal diterima, hitung kembalian; nominal harus cukup. QRIS: tunjukkan kode statis/alat merchant, cocokkan transaksi melalui aplikasi/merchant sebelum konfirmasi `PAID`.
6. POST transaksi secara atomik: simpan sale + items (harga, cost snapshot), payment confirmed, stock OUT, dan event audit. Payment gagal/pending tidak membuat stock OUT permanen.
7. Tawarkan struk setelah transaksi berhasil (print browser/thermal di fase selanjutnya), bukan syarat penjualan.
8. Transaksi masuk ringkasan shift dan finance reporting.

## 3. Kebutuhan fungsional

**POS-001** Cari/filter produk aktif dan pilih produk dengan informasi harga jelas.  
**POS-002** Keranjang mendukung qty, penghapusan, subtotal dan total; semua angka dihitung ulang server.  
**POS-003** Harga satuan dan total disimpan pada `sale_items` saat sale confirmed.  
**POS-004** Cash menyimpan `cash_received`, `change_amount` dan net cash retained; kembalian bukan revenue.  
**POS-005** QRIS menyimpan `payment_method=QRIS`, referensi merchant opsional dan `verified_by/verified_at`; pengesahan harus terpisah dari klaim pelanggan.  
**POS-006** Pada MVP satu metode pembayaran per sale. Split payment masuk P1 dan skema dapat berkembang lewat beberapa baris `payments`.  
**POS-007** Struk mencantumkan nomor transaksi, tanggal lokal, item, qty, harga, total, metode bayar, nama toko; customer opsional.  
**POS-008** Cashier dapat melihat transaksi di shift miliknya dan cetak ulang struk dengan watermark/reprint indicator opsional.  
**POS-009** Pembatalan transaksi setelah paid memerlukan permission dan alasan, menghasilkan reversal stok dan pembayaran (jika pengembalian dana benar-benar terjadi) secara terlacak; tidak boleh hard delete.  
**POS-010** Default diskon nonaktif untuk Cashier; Admin/Owner dapat aktifkan permission dan batasnya.  
**POS-011** Checkout wajib punya idempotency key dari client; retry timeout tidak membuat sale ganda.  
**POS-012** Blokir checkout jika tidak ada shift aktif bila kebijakan sesi diaktifkan (default: aktif).  
**POS-013** Saldo stok harus cukup saat checkout (bukan hanya ketika tambah keranjang).
**POS-014** Pembayaran pending/failed tidak dianggap pendapatan, tidak mengurangi stok, dan tidak tercatat sebagai kas fisik.

## 4. Status

Sale: `DRAFT` (keranjang sementara) → `AWAITING_PAYMENT` → `PAID` atau `CANCELLED`; setelah PAID dapat menjadi `VOIDED` melalui dokumen reversal beralasan. **P1**: `PARTIALLY_REFUNDED` dan `REFUNDED`.

Payment: `PENDING` → `CONFIRMED` / `FAILED` / `CANCELLED`; untuk QRIS confirmed dapat tetap `UNSETTLED` pada level bank settlement. Jangan menyamakan payment confirmed dengan uang sudah masuk rekening.

## 5. Kasus tepi

- Timeout setelah berhasil: retry dengan key sama mengembalikan sale identik.
- Harga berubah saat keranjang terbuka: checkout menampilkan perubahan harga dan meminta persetujuan.
- Produk dinonaktifkan di tengah checkout: item ditolak dengan pesan jelas.
- QRIS pending: transaksi tidak menjadi paid sampai konfirmasi; sediakan retry/cancel.
- Cash drawer: hanya cash sales dan cash movements yang masuk perhitungan kas fisik; QRIS tidak masuk.
- Void: transaksi historis tetap ada, laporan bersifat net terhadap reversal.
- Transaksi POS tidak boleh mengubah receipt atau harga modal.

## 6. Acceptance

- **POS-T01:** 2 Aqua @Rp5.000 + 1 Pocari @Rp9.000 → total Rp19.000.
- **POS-T02:** Pembayaran cash Rp20.000 → kembalian Rp1.000, kas bertambah Rp19.000.
- **POS-T03:** QRIS tanpa verifikasi tidak mengurangi stok; setelah verified, stok berkurang sekali.
- **POS-T04:** Refresh/retry request confirm identik tidak menggandakan sale/movement.
- **POS-T05:** Void paid dengan approval mengembalikan stok dan mencatat cash out hanya bila refund cash benar-benar diberikan.
