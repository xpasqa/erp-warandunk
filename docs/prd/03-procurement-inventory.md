# 03 — Procurement & Inventory Requirements

## 1. Domain overview

Procurement menyimpan **rencana belanja**, **penerimaan barang**, dan **pertanggungjawaban dana**. Inventory menyimpan **master produk**, **pergerakan stok**, dan **saldo stok**. Ketiganya terhubung tetapi tidak digabung menjadi satu tabel.

## 2. Shopping list

**PROC-001** Owner membuat shopping list `DRAFT`, menambah/menghapus baris produk dengan jumlah rencana dan catatan. Produk boleh dipilih dari katalog; item baru boleh berupa free-text hingga tahap receiving.  
**PROC-002** Satu list dapat diunduh/di-print sebagai daftar belanja yang mudah dibagikan (print/PDF browser pada MVP; CSV opsional).  
**PROC-003** Status: `DRAFT`, `ON_PROCESS`, `PARTIALLY_RECEIVED`, `RECEIVED`, `CANCELLED`. Rider tidak perlu login/tracking.  
**PROC-004** Daftar menyimpan pembuat, tanggal, lokasi, catatan, serta histori perubahan.  
**PROC-005** Belanja parsial atau lebih dari satu kali penerimaan diizinkan. Kelebihan qty memerlukan konfirmasi verifikator.

## 3. Purchase receiving dan verifikasi

**PROC-006** Admin/Owner mencatat barang nyata yang diterima: SKU/produk, qty aktual, unit, harga satuan beli, diskon/biaya bila ada, supplier opsional, nota/bukti opsional, dan tanggal penerimaan.  
**PROC-007** Item baru dapat dibuat menjadi produk dengan kategori, harga jual, SKU, satuan, minimum stock, optional metadata.  
**PROC-008** Penerimaan `DRAFT` tidak memengaruhi stok. Saat `VERIFIED/POSTED`, buat `stock_movements` positif atomik.  
**PROC-009** Tolak posting ganda dengan unique `source_type + source_line_id + movement_kind` atau idempotency key.  
**PROC-010** Catat perbedaan rencana vs aktual, alasan kekurangan/kelebihan, harga beli aktual, dan verifikator.  
**PROC-011** Setelah diposting tidak boleh edit qty/harga historis langsung. Gunakan reversal/koreksi dengan alasan serta approval.

## 4. Settlement belanja

**PROC-012** Payment/settlement pembelian adalah transaksi tersendiri dari stock posting; receipt dapat `UNPAID`, `PARTIALLY_PAID`, atau `PAID` setelah validasi pembayaran.  
**PROC-013** Pembayaran dapat berasal dari kas operasional, transfer owner, atau sumber dana lain yang dikonfigurasi. Hanya pembayaran yang memengaruhi kas warung yang dicatat sebagai `cash_movements` keluar.  
**PROC-014** Total pembayaran tidak boleh melebihi kewajiban receipt tanpa kebijakan biaya tambahan/uang muka terpisah.  
**PROC-015** Simpan bukti pembayaran, actor, payee, sumber dana, waktu, jumlah dan nomor referensi opsional.  
**PROC-016** List `RECEIVED` tidak otomatis berarti `PAID`, begitu pula sebaliknya.

## 5. Master katalog dan stok

**CAT-001** Produk: `sku` unik per store; nama; kategori; satuan (`pcs` default); barcode opsional; harga jual IDR; minimum stock; aktif/nonaktif; `metadata` JSONB untuk atribut opsional.  
**CAT-002** Kategori dibuat/diubah oleh Owner/Admin; produk yang sudah pernah dipakai pada transaksi tidak boleh hard-delete.  
**CAT-003** Harga jual dapat diedit oleh yang berwenang, tetapi histori `sale_items` tidak berubah.  
**CAT-004** Satuan pecahan hanya jika dikonfigurasi; tipe NUMERIC untuk qty dan unit cost.  
**INV-001** Saldo on-hand diturunkan dari penjumlahan ledger signed `stock_movements` per `store_id, product_id`. Cache boleh ada sebagai proyeksi yang terjaga atomik.  
**INV-002** Movement masuk akibat receipt terposting; keluar akibat sale paid; koreksi saat adjustment; reversal saat pembatalan resmi.  
**INV-003** Movement mempunyai jenis, signed quantity, nilai modal saat posting, actor, waktu, dan referensi dokumen sumber.  
**INV-004** Default melarang saldo negatif. Per transaksi gunakan row-level locking atau mekanisme atomik lain untuk mencegah overselling paralel.  
**INV-005** Stok awal dimasukkan sebagai opening adjustment terotorisasi, bukan angka awal yang disembunyikan.  
**INV-006** Untuk biaya barang dijual gunakan moving weighted average (WAC) per produk dan store; nilai HPP sale item disimpan sebagai snapshot saat posting.  
**INV-007** Dashboard stok menampilkan on-hand, stok minimum, stok rendah, stok habis, dan transaksi mutasi.

## 6. Aturan WAC

Ketika barang masuk: `new_avg_cost = (old_qty * old_avg_cost + received_qty * unit_landed_cost) / (old_qty + received_qty)`; jika old_qty = 0, average cost = unit landed cost. Saat jual, `COGS = sold_qty * avg_cost_at_sale`, tidak mengubah average cost. Retur/penyesuaian mengikuti nilai cost yang direferensikan atau kebijakan koreksi yang tercatat. **Pembelian barang bukan langsung seluruhnya biaya HPP.**

**[OPEN]** Biaya transport pembelian dan potongan supplier: apakah dialokasikan ke landed cost per unit atau dicatat sebagai expense? MVP default: biaya terpisah kecuali dimasukkan manual ke unit landed cost.

## 7. Kasus tepi

- Satu list diterima 2 kali; stock IN dibuat per receipt, list menjadi `PARTIALLY_RECEIVED` lalu `RECEIVED`.
- Barang datang rusak: qty accepted berbeda dari qty delivered; hanya qty accepted yang diposting. Qty rusak dicatat sebagai catatan/variance.
- Produk yang dinonaktifkan tetap tampil di transaksi historis tetapi tidak bisa dipilih untuk penjualan baru.
- Edit harga jual tidak mengubah keranjang yang sudah dibayar; keranjang belum dibayar wajib revalidasi harga sebelum checkout.
- Barang habis tidak dapat dijual, bahkan jika kasir membuka dua tab sekaligus.
- Pembelian dicatat dari dana pribadi Owner tidak otomatis mengurangi kas laci kasir.

## 8. Acceptance procurement/inventory

- **INV-T01:** Receipt draft 24 botol tidak mengubah stok; setelah verified+posted, stok bertambah tepat 24.
- **INV-T02:** Ulangi request post yang sama: tetap satu set movement.
- **INV-T03:** Dua penjualan paralel untuk satu unit tersisa: tepat satu berhasil.
- **INV-T04:** Perubahan modal/harga jual master tidak merevisi cost/price snapshot histori.
- **INV-T05:** Belanja sebagian dapat dituntaskan kemudian, tanpa menambah qty dua kali.
- **INV-T06:** Payment pembelian dan receiving dapat berbeda hari tanpa kehilangan status kewajiban.
