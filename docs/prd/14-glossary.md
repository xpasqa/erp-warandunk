# 14 — Glossary & Ubiquitous Language

| Istilah | Definisi operasional |
| --- | --- |
| WARANDUNK | Aplikasi Mini ERP warung GOR; bukan aplikasi rental lapangan |
| Store/outlet | Lokasi penjualan dan pemilik saldo stok/kas operasional |
| Owner | Role tertinggi, akses penuh namun tetap diaudit |
| Admin | Verifikator pembelian, katalog, cash/daily closing menurut izin |
| Cashier | Pelaku transaksi POS dan submit tutup shift |
| Customer walk-in | Pembeli tanpa akun; customer record opsional |
| Rider | Orang yang membeli barang dari daftar; **bukan** role MVP |
| Shopping List | Rencana barang dan target qty yang hendak dibeli; tidak mengubah stok |
| On Process | Belanja sedang dijalankan di luar aplikasi |
| Purchase Receipt | Bukti barang fisik diterima, diperiksa, dan akan diposting ke stok |
| Received qty | Qty fisik diterima dari pemasok |
| Accepted qty | Qty yang diverifikasi layak masuk stok |
| Purchase Settlement | Pencatatan pembayaran/pertanggungjawaban pembelian, terpisah dari penerimaan |
| Product Catalog | Data master produk, SKU, kategori, unit, harga aktif, metadata |
| SKU | Kode unik produk dalam store |
| On-hand | Qty tersedia berdasarkan seluruh movement terposting |
| Stock Movement | Catatan immutable perubahan qty signed (+IN, −OUT) dengan sumber dokumen |
| Stock Posting | Commit final yang membuat ledger movement dan saldo berubah |
| Opening Stock | Saldo awal yang dibuat sebagai adjustment resmi |
| WAC | Weighted Average Cost, rata-rata modal bergerak per SKU/store |
| HPP / COGS | Harga pokok barang yang terjual, disnapshot ketika sale posted |
| POS | Point of Sale: pemilihan barang, pembayaran, struk |
| Sale PAID | Transaksi penjualan sudah dibayar valid, stok keluar dan revenue diakui dalam model operasional |
| Payment Confirmed | Pembayaran sah menurut prosedur cashier/merchant yang berlaku |
| QRIS | Pembayaran QR; di MVP melalui verifikasi merchant manual |
| QRIS Settlement | Pencairan/pencocokan hasil QRIS ke rekening bank; bisa berbeda tanggal dari sale |
| Cash Drawer | Uang tunai fisik di laci; QRIS tidak termasuk |
| Cash Movement | Penambahan/pengeluaran kas non-sales dan refund yang ditelusuri ke sumber |
| POS Session | Sesi shift kasir dengan opening, transaksi, closing |
| Session Closing | Pengajuan dan persetujuan akhir kas suatu sesi |
| Daily Closing | Agregasi/review satu hari bisnis per store |
| Stock Count | Penghitungan stok fisik dan perbandingan dengan ledger |
| Cash Variance | Counted cash dikurangi expected cash |
| Gross Sales | Penjualan kotor sebelum diskon dan retur |
| Net Sales | Penjualan setelah diskon dan retur |
| Gross Profit | Net Sales dikurangi HPP |
| Operating Expense | Beban operasional nonpersediaan |
| Operational Profit | Indikasi gross profit dikurangi expense, bukan laporan akuntansi formal |
| Audit Log | Histori tindakan sensitif dan alasan/perubahan terkait |
| Idempotency Key | Token mencegah pengulangan request membuat transaksi ganda |
| Reversal/Void | Transaksi koreksi/pembalik yang tidak menghapus transaksi asli |
| Modular Monolith | Satu deploy aplikasi dengan batas modul bisnis dan kontrak eksplisit |
| JSONB Metadata | Properti produk opsional yang tidak dipakai sebagai pusat perhitungan |
| P0 / P1 / P2 | Harus ada untuk MVP / peningkatan berikutnya / pengembangan jangka lanjut |
| [OPEN] | Kebijakan atau requirement belum diputuskan final |
