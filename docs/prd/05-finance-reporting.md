# 05 — Finance, Cash Flow & Reporting

## 1. Posisi domain Finance

MVP adalah **operational finance**, bukan general ledger akuntansi double-entry formal. Finance merupakan pembacaan konsisten atas `sales/payments`, `purchase_receipts/purchase_payments`, `stock_movements`, `expenses`, `cash_movements`, dan `pos_sessions`. Hindari menyalin semua angka ke tabel finance independen yang rawan drift.

Tiga ukuran harus dibedakan: **revenue** (penjualan sah), **profit** (revenue dikurangi HPP dan biaya), **cash flow** (uang masuk/keluar). Mencatat pembelian stok sebesar Rp300.000 hari ini tidak otomatis mengurangi laba hari ini Rp300.000; stok yang belum terjual adalah aset.

## 2. Transaksi keuangan

**FIN-001** Payment sale yang `CONFIRMED` berkontribusi ke penjualan dan kategori metode pembayaran.  
**FIN-002** `CASH` sale menambah cash expected sebesar net yang tersimpan setelah kembalian; QRIS tidak menambah laci kas.  
**FIN-003** Cash movement punya `type=IN/OUT`, jumlah positif, kategori, alasan, aktor, waktu, store, session opsional dan referensi sumber.  
**FIN-004** Pengeluaran harian (`expenses`) dapat dicatat Admin/Owner (mis. tisu, transport, listrik) dengan kategori, sumber pembayaran, bukti opsional. Hindari menghitung satu expense dua kali jika berasal dari purchase payment.  
**FIN-005** Purchase payment dapat berasal dari `CASH_DRAWER`, `STORE_BANK`, `OWNER_PERSONAL` atau kategori sumber dana lain; dampak arus kas disesuaikan asalnya.  
**FIN-006** QRIS payment confirmed dicatat pada saldo **QRIS receivable/unsettled** untuk laporan operasional sampai dicocokkan dengan settlement bank; potongan MDR dapat dilaporkan terpisah.  
**FIN-007** Laporan dapat disaring tanggal, store, metode bayar, kategori, cashier; gunakan UTC pada penyimpanan dan Asia/Jakarta untuk kalender operasional.  
**FIN-008** Simpan snapshot cost-of-goods-sold pada baris sale saat posting, bukan hitung ulang dari rata-rata modal hari ini.
**FIN-009** Semua angka uang disimpan sebagai bilangan bulat Rupiah (`BIGINT`) pada MVP; biaya per unit/rata-rata dapat `NUMERIC` presisi tinggi. Terapkan pembulatan konsisten saat posting.
**FIN-010** Data exported CSV dan ringkasan printable; ekspor XLSX lanjutan P1.

## 3. Metrik dan rumus

| Metrik | Definisi |
| --- | --- |
| Gross sales | Jumlah harga kotor item sebelum discount/refund |
| Discount | Jumlah discount sah |
| Net sales | Gross sales − discount − refund/void net (sebelum pajak jika kelak aktif) |
| Cash sales | Net paid via CASH yang confirmed |
| QRIS sales | Net paid via QRIS yang confirmed |
| Total COGS/HPP | Σ qty terjual bersih × unit cost snapshot; disesuaikan reversal |
| Gross profit | Net sales − COGS |
| Operating expense | Pengeluaran nonpersediaan yang termasuk beban periode |
| Indicative operating profit | Gross profit − operating expense (bukan laba pajak/akuntansi formal) |
| Stock on-hand | Σ qty signed stock ledger |
| Stock valuation | Σ qty tersisa × weighted average unit cost |
| Expected drawer cash | Opening float + cash sales + cash IN − cash OUT − cash refunds |
| Cash variance | Counted physical cash − expected drawer cash |
| QRIS pending settlement | QRIS confirmed − bank settled (dan adjustment/refund terkait) |
| Purchase spending | Jumlah purchase payment benar-benar dibayarkan dalam periode |
| Purchase outstanding | Receipt payable − confirmed purchase payments |

**Penting:** Penjualan adalah event saat sale paid/confirmed, sedangkan bank settlement QRIS boleh terjadi pada hari yang berbeda. Laporkan tanggal transaksi dan tanggal settlement terpisah.

## 4. Dashboard dan laporan

**RPT-001 Daily Sales:** total paid, sale count, AOV, discount, void, cash vs QRIS, cashier/shift.  
**RPT-002 Stock:** on-hand, minimum, stockout, valuation, masuk/keluar, penyesuaian dan item terlaris.  
**RPT-003 Procurement:** shopping list by status, receipt value, received variance, unpaid/part-paid/paid.  
**RPT-004 Finance:** total uang masuk/keluar menurut source, gross profit indikatif, expense category, cash closing difference, QRIS not-yet-settled.  
**RPT-005 Audit:** closing pending/approved, variance, manual adjustments, post-sale void dan aktivitas sensitif.

## 5. Aturan periode dan koreksi

- Periode transaksi dilihat pada `occurred_at`, periode audit dapat dilihat berdasarkan `closed_at`.
- Posted sale atau receipt tidak diedit diam-diam; reversal menyebabkan net report bertambah/berkurang pada tanggal reversal dengan opsi tampilan berdasarkan dokumen asal.
- Refund cash mengurangi kas saat dibayar kembali, bukan hanya saat sale menjadi VOIDED.
- Report harus bisa direkonsiliasi ke daftar transaksi sumber (*drill-down*).
- Untuk tax, fee, diskon bertingkat, piutang supplier dan akuntansi formal: tetapkan kebijakan sebelum mengaktifkan fitur terkait.

## 6. Acceptance

- **FIN-T01:** Pembelian 24 unit @Rp3.000 tidak langsung menjadi HPP Rp72.000; setelah jual 2 unit, HPP Rp6.000 (tanpa stok lama dan biaya tambahan).
- **FIN-T02:** Cash sale Rp19.000 + QRIS Rp9.000 → drawer expected bertambah Rp19.000, sales total Rp28.000.
- **FIN-T03:** QRIS confirmed hari Senin, settle bank Selasa: revenue Senin; bank settlement Selasa.
- **FIN-T04:** Cash out dari pembayaran pembelian yang sama hanya muncul sekali.
- **FIN-T05:** Laporan profit tidak terlihat oleh Cashier.
