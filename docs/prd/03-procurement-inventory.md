# 03 — Procurement & Inventory Requirements

## 1. Domain overview

Procurement menyimpan **rencana belanja**, **penerimaan barang**, dan **pertanggungjawaban dana**. Inventory menyimpan **master produk**, **pergerakan stok**, dan **saldo stok**. Ketiganya terhubung tetapi tidak digabung menjadi satu tabel.

## 2. Shopping list (Owner/Admin)

**PROC-001** Owner/Admin membuat dan mengedit shopping list DRAFT: produk katalog atau free-text, qty rencana, unit, catatan. Cashier **tidak** dapat membuat/mengubah shopping list.  
**PROC-002** List dapat diekspor/dicetak untuk dibawa belanja; rider tidak perlu akun.  
**PROC-003** Status list: `DRAFT`, `ON_PROCESS`, `PARTIALLY_RECEIVED`, `RECEIVED`, `CANCELLED`.  
**PROC-004** Cashier melihat daftar ON_PROCESS milik store dan mengisi realisasi pembelian berdasarkan daftar tersebut.  
**PROC-005** Sebuah list bisa dipenuhi lewat beberapa transaksi pembelian Cashier; sisa qty terlihat dan tidak boleh menggandakan penerimaan.

## 3. Purchase entry oleh Cashier dan approval berbasis struk

**PROC-006** Cashier membuat `purchase_receipt` DRAFT berdasarkan shopping list; memasukkan barang aktual, accepted qty, unit, harga beli satuan, total, catatan, usulan harga jual untuk produk baru/ubah harga. Usulan harga jual tidak mengubah produk sampai diterima approver.  
**PROC-007** Cashier **wajib mengunggah struk pembelian** (JPEG, PNG, PDF; batas awal 10 MB per file, configurable) sebelum dapat `Submit for Approval`. Foto kamera HP diperbolehkan. File disimpan privat dan metadata disimpan di database.  
**PROC-008** Status receipt: `DRAFT → SUBMITTED → APPROVED_POSTED`; jalur koreksi `SUBMITTED → REJECTED → DRAFT (revisi) → SUBMITTED`. Rejection wajib menyimpan alasan dan riwayat.  
**PROC-009** `DRAFT`, `SUBMITTED`, dan `REJECTED` tidak memengaruhi stok. Saat SUBMITTED, item/file terkunci hingga direject.  
**PROC-010** Owner atau Admin melihat preview struk dan membandingkan daftar belanja dengan barang aktual, jumlah, harga beli, total, dan usulan harga jual; dapat **Approve & Post** atau **Reject dengan alasan**.  
**PROC-011** Satu approver Owner **atau** Admin cukup dalam MVP (maker-checker). Cashier tidak dapat approve. Owner dapat meninjau approval Admin; Owner-mandatory final approval adalah opsi [OPEN] bukan aturan MVP.  
**PROC-012** **Approve & Post** menjadi satu transaksi atomik: validasi approver, struk valid masih tersimpan, item/qty/cost, lalu buat movement IN, hitung WAC, update saldo dan `approved_by/approved_at/posted_at`. Jika gagal, rollback; retry idempotent tidak menggandakan stok.  
**PROC-013** Reject mengembalikan form ke Cashier untuk revisi/resubmit. Bukti lama dan revisi harus bisa diaudit, bukan ditimpa diam-diam.  
**PROC-014** Harga jual baru/usulan dari Cashier hanya dapat diberlakukan oleh Owner/Admin saat approval; harga jual historis tidak berubah.  
**PROC-015** Setelah APPROVED_POSTED tidak boleh mengedit atau menghapus transaksi; koreksi melalui reversal/adjustment dengan izin dan alasan.  
**PROC-016** Purchase settlement/payment terpisah dari stock posting: penerimaan bisa approved tetapi biaya belum dibayar atau belum dipertanggungjawabkan.  
**PROC-017** Upload privat menggunakan MIME/magic bytes validation, size limit, otorisasi store, dan signed URL pendek; simpan `checksum`, metadata uploader, waktu, dan histori penggantian file. Jika struk tidak tersedia, receipt **tidak dapat** submit/approve pada MVP.

## 4. Settlement belanja

**PROC-018** Payment/settlement pembelian adalah transaksi tersendiri dari stock posting; receipt dapat `UNPAID`, `PARTIALLY_PAID`, atau `PAID` setelah validasi pembayaran.  
**PROC-019** Pembayaran dapat berasal dari kas operasional, transfer owner, atau sumber dana lain yang dikonfigurasi. Hanya pembayaran yang memengaruhi kas warung yang dicatat sebagai `cash_movements` keluar.  
**PROC-020** Total pembayaran tidak boleh melebihi kewajiban receipt tanpa kebijakan biaya tambahan/uang muka terpisah.  
**PROC-021** Simpan bukti pembayaran, actor, payee, sumber dana, waktu, jumlah dan nomor referensi opsional.  
**PROC-022** List `RECEIVED` tidak otomatis berarti `PAID`, begitu pula sebaliknya.

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

- **INV-T01:** Receipt Cashier DRAFT/SUBMITTED 24 botol dengan struk tidak mengubah stok; setelah Owner/Admin Approve & Post, stok bertambah tepat 24.
- **INV-T02:** Ulangi request post yang sama: tetap satu set movement.
- **INV-T03:** Dua penjualan paralel untuk satu unit tersisa: tepat satu berhasil.
- **INV-T04:** Perubahan modal/harga jual master tidak merevisi cost/price snapshot histori.
- **INV-T05:** Belanja sebagian dapat dituntaskan kemudian, tanpa menambah qty dua kali.
- **INV-T06:** Payment pembelian dan receiving dapat berbeda hari tanpa kehilangan status kewajiban.

## 9. Acceptance tambahan

- **PROC-T01:** Cashier tidak dapat membuat shopping list tetapi dapat input realisasi dari list ON_PROCESS.
- **PROC-T02:** Submit tanpa struk ditolak server; status dan saldo tidak berubah.
- **PROC-T03:** Owner/Admin dapat preview struk, membandingkan item, dan approve/reject; stock IN hanya saat approve.
- **PROC-T04:** Reject mewajibkan alasan, Cashier dapat memperbaiki input/struk dan submit ulang.
- **PROC-T05:** Cashier yang memanggil endpoint approve menerima 403.
- **PROC-T06:** Retry approval tidak menggandakan stok; file dari store lain tidak dapat dilihat atau dipakai.
