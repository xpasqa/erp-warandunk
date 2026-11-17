# 06 — Daily Closing, Reconciliation & Audit

## 1. Tujuan kontrol

Daily audit memastikan: (a) kas laci sesuai bukti transaksi tunai; (b) QRIS terverifikasi pada merchant dan settlement bank dipantau terpisah; (c) stok yang keluar memiliki sumber penjualan/adjustment; (d) pembelian dan biaya mempunyai sumber dan persetujuan; (e) selisih diketahui dan diberi tindak lanjut.

**Istilah tidak boleh dicampur:** `purchase settlement` = pertanggungjawaban/pembayaran pembelian; `POS session closing` = penutupan laci kasir; `daily closing` = validasi operasional per toko/per hari; `stock count` = perhitungan barang fisik.

## 2. Alur satu shift

1. Cashier membuka session dengan `opening_cash`; identitas kasir dan waktu dicatat.
2. Selama shift, transaksi cash/QRIS terekam, beserta cash in/out resmi. Transaksi belum sah tidak dihitung.
3. Cashier mengajukan tutup shift dan mengisi `counted_cash`. Sistem menghitung `expected_cash`.
4. Sistem menampilkan selisih, paid cash sales, cash out, refund, dan QRIS confirmed sebagai **angka terpisah**.
5. Admin/Owner memeriksa, mencatat alasan selisih jika tidak nol, dan menyetujui/tolak pengajuan.
6. Setelah closing disetujui, sesi terkunci; perubahan data historis melalui reversal/adjustment tercatat dan perlu diungkap dalam laporan.
7. Admin memfinalkan daily closing setelah semua sesi masuk, pengeluaran, pembelian, dan QRIS merchant report diperiksa sesuai prosedur.

## 3. Kebutuhan

**AUD-001** Setiap sale paid dikaitkan dengan session terbuka dan cashier; satu cashier tidak boleh punya dua session aktif untuk store yang sama (default).  
**AUD-002** Opening float disimpan dan diverifikasi saat pembukaan sesuai kebijakan.  
**AUD-003** `expected_cash = opening_cash + confirmed_cash_sales + cash_in - cash_out - cash_refunds`; pembayaran QRIS tidak masuk expected cash.  
**AUD-004** Counted cash diinput manusia; `variance = counted - expected` disimpan sebagai snapshot saat submit dan dihitung ulang saat approval.  
**AUD-005** Variance ≠ 0 mewajibkan reason sebelum approval; threshold untuk eskalasi dapat dikonfigurasi.  
**AUD-006** Admin tidak boleh memverifikasi sesi yang ia submit sendiri jika mode two-person approval aktif.  
**AUD-007** Closing perlu memuat jumlah sales, cash/QRIS breakdown, void, refund, expense cash, purchase payment dari kas, saldo dan variance.  
**AUD-008** QRIS reconciliation memisahkan: verified merchant transaction vs transfer/settlement bank; fee MDR dicatat jika diketahui.  
**AUD-009** Daily closing punya `business_date` (Asia/Jakarta), store, status, verifier, totals snapshot, catatan dan audit trail.  
**AUD-010** Stock count mendukung `expected_qty` vs `physical_qty` dan adjustment setelah approval. Pada MVP, boleh berupa manual spot check dan adjustment terkontrol; sesi stock opname komprehensif P1.  
**AUD-011** Semua tindakan berisiko tinggi mencatat `actor_id`, `action`, `resource`, `resource_id`, `before/after` teredaksi, `reason`, `approval_actor` jika ada, dan `created_at`.  
**AUD-012** Hasil audit dapat difilter tanggal, store, kasir dan status; hanya role berwenang boleh melihatnya.

## 4. Audit harian contoh

Kas awal Rp100.000; cash sales confirmed Rp250.000; cash out Rp50.000; cash refunds Rp0. Expected cash Rp300.000. Kas fisik Rp295.000. **Variance −Rp5.000**, wajib alasan dan review. QRIS confirmed Rp150.000 dicatat di jalur merchant, tidak ditambahkan ke expected drawer.

## 5. Stock audit/adjustment

- Selisih produk dihitung `physical − expected`; adjustment harus mencantumkan alasan (rusak, hilang, sampling error, kedaluwarsa, salah input).
- Perubahan stok terposting hanya melalui signed movement berreferensi `ADJUSTMENT`.
- Stock count boleh dilakukan selektif pada SKU cepat terjual, bernilai tinggi, atau yang sering selisih.
- Default tidak boleh adjustment yang menyebabkan stok negatif.
- Catatan yang sudah disetujui tidak dihapus.

## 6. Acceptance

- **AUD-T01:** Cash sale + QRIS sale pada shift yang sama: QRIS tidak menambah expected cash.
- **AUD-T02:** Jika variance tidak nol dan reason kosong, approval ditolak.
- **AUD-T03:** Cashier tidak dapat memverifikasi closing sendiri.
- **AUD-T04:** Closing approved tidak bisa diedit biasa; koreksi tercatat sebagai event koreksi.
- **AUD-T05:** Stok adjustment menghasilkan movement dengan dokumen sumber dan approver.
- **AUD-T06:** QRIS belum settled tetap ditandai outstanding pada laporan tanpa menahan revenue valid.
