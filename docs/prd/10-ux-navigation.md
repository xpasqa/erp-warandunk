# 10 — UX, Screens & Navigation

## 1. Desain pengalaman

Antarmuka POS harus **lebih cepat dari menggunakan spreadsheet**: sedikit tindakan, teks jelas, harga besar, tombol pembayaran menonjol, dan toleran jaringan lambat tanpa double sale. Owner/Admin membutuhkan detail audit, sedangkan Cashier hanya fungsi yang diperlukan untuk menjual.

Navigasi role-aware, bukan akses keamanan utama. Bahasa antarmuka default Indonesia, format IDR dan tanggal Asia/Jakarta. Tidak menampilkan fitur rental lapangan.

## 2. Sitemap MVP

```text
Login
├─ Dashboard (Owner/Admin)
│  ├─ Sales Summary / Cash vs QRIS
│  ├─ Low Stock / Purchase Pending
│  └─ Closing Variance
├─ Catalog
│  ├─ Categories
│  └─ Products (SKU, unit, prices, min stock, metadata)
├─ Procurement
│  ├─ Shopping Lists → Create/Edit → Export/Print
│  ├─ Cashier Purchase Entry → Qty/Cost → Upload Struk → Submit
│  ├─ Owner/Admin Receipt Approval → Review Struk → Approve & Post / Reject
│  └─ Purchase Payments / Settlement
├─ Inventory
│  ├─ Stock On-hand / Low Stock
│  └─ Stock Movements / Adjustment
├─ POS
│  ├─ Open Session
│  ├─ Product Grid/Search → Cart → Checkout
│  ├─ Payment Cash/QRIS → Receipt
│  └─ History / Void (permission gated)
├─ Finance
│  ├─ Expenses & Cash Movement
│  └─ Sales/Profit/Stock/Payment Reports
├─ Closing & Audit
│  ├─ Submit Shift Close
│  ├─ Approve Shift Close
│  ├─ Daily Closing / Variance
│  └─ Audit Trail
└─ Settings (Owner)
   ├─ Users / Roles
   └─ Store / Policy / Receipt
```

## 3. Screen specifications

### POS Screen

- Area produk: search by name/SKU/barcode, filter category, stok/price visible, status unavailable disabled.
- Area keranjang: selected items, qty +/−, total fixed/sticky, clear cart dengan konfirmasi.
- Dialog checkout: total, metode CASH/QRIS. CASH menunjukkan input nominal, quick denominations opsional, kembalian. QRIS menunjukkan instruction dan tombol konfirmasi merchant (restricted).
- Response loading atau retry harus tetap membawa idempotency key sama; disable double submit; success menampilkan nomor transaksi & print.
- Keyboard shortcuts pada desktop P1; pada tablet field jumlah lebih besar.

### Shopping List Screen

- Draft editor dengan produk katalog atau free-text item, target qty, note, CTA Save/Start/Export.
- Status dan progress received per baris; item tidak otomatis menjadi stok saat status `ON_PROCESS`.

### Receiving Screen

- Cashier membuka list ON_PROCESS, mengisi barang/qty/harga beli aktual, usulan harga jual dan **wajib mengunggah struk foto/PDF** sebelum Submit.
- Owner/Admin melihat preview struk bersebelahan dengan data item, qty, harga dan total, lalu klik Approve & Post atau Reject (alasan wajib).
- Jika rejected, Cashier melihat alasan, memperbaiki form/struk dan submit ulang. Jika approved stok masuk; sebelumnya stok tidak berubah.
- Settlement dana tetap pada tab/step terpisah dari posting stok.

### Inventory Screen

- Per kategori/produk: on-hand, minimum stock, last inbound/outbound, unit dan penanda stok rendah.
- Movement timeline: sumber, qty signed, actor, waktu; adjustment butuh alasan.

### Closing Screen

- Cashier menginput counted cash tanpa menampilkan/mengunci expected cash sesuai kebijakan blind count [OPEN]; setidaknya expected disimpan server.
- Admin melihat expected vs counted, variance, rincian cash sale/cash movement dan QRIS secara terpisah.
- Wajib alasan untuk variance nonzero; review/approve atau reject.

### Reporting Screen

- Filter tanggal, store, metode pembayaran, cashier; kartu KPI dan tabel drill-down.
- Tampilkan label "indikatif" pada laba operasional MVP, bukan laporan keuangan audited.

## 4. Empty/error states

| Kondisi | UI behavior |
| --- | --- |
| Belum ada produk | Prompt Owner/Admin menambah kategori & produk |
| Belum ada stok | POS tidak mengizinkan checkout; tautkan proses receiving hanya untuk Admin |
| QRIS belum diverifikasi | Tampilkan Pending, jangan cetak struk paid |
| Stok habis saat submit | Tampilkan qty tersedia dan pilihan kurangi qty |
| Network timeout | Pesan "cek status transaksi" lalu retry idempotent, bukan checkout baru |
| Permission denied | Tombol tidak ditampilkan; deep link API tetap 403 |
| Closing variance | Tampilkan selisih dan kolom alasan wajib |

## 5. Accessibility dan responsiveness

- Responsif minimal 360px mobile hingga 1280px desktop; grid produk dan keranjang adaptif.
- Fokus tab/keyboard untuk input, label accessible, error field spesifik, ukuran tap target memadai.
- Tidak hanya mengandalkan warna untuk status; selalu ada teks.
- Print stylesheet memprioritaskan lebar 58/80mm pada fase thermal, dan A4/browser print untuk MVP.
