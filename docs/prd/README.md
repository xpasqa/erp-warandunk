# WARANDUNK — Product Requirements Documentation (PRD)

**Versi:** 0.1.0 (baseline kebutuhan, sebelum implementasi)  
**Tanggal:** 2026-11-17  
**Status:** Draft for validation  
**Pemilik produk:** Owner operasional warung  
**Ruang lingkup awal:** Warung penjualan barang/minuman/makanan kemasan di GOR badminton. **Tidak mencakup booking, rental lapangan, atau manajemen olahraga.**

## Ringkasan

WARANDUNK adalah Mini ERP ringan untuk mencatat rantai transaksi **daftar belanja → penerimaan dan verifikasi → stok tersedia → penjualan POS → pencatatan kas/QRIS dan HPP → penutupan shift → audit harian dan laporan**. Sistem didesain sebagai **modular monolith** agar domain dapat digunakan ulang pada warung, kantin, retail, hingga F&B sederhana tanpa menggandakan logika ledger.

Dokumen ini merupakan spesifikasi produk, **bukan bukti implementasi**. Keputusan yang masih perlu divalidasi ditandai **[OPEN]** dalam dokumen terkait. Rekomendasi teknologi dan contoh endpoint bukan keputusan final.

## Indeks dokumen

| File | Muatan |
| --- | --- |
| [01-product-brief.md](01-product-brief.md) | Visi, masalah, ruang lingkup, KPI dan asumsi |
| [02-roles-permissions.md](02-roles-permissions.md) | Persona, RBAC, matriks akses dan approval |
| [03-procurement-inventory.md](03-procurement-inventory.md) | Shopping list, penerimaan, verifikasi, settlement pembelian dan stok |
| [04-pos-sales.md](04-pos-sales.md) | POS, pembayaran, struk, void, refund dan anomali |
| [05-finance-reporting.md](05-finance-reporting.md) | Kas, QRIS, HPP, laba rugi operasional dan laporan |
| [06-daily-audit.md](06-daily-audit.md) | Tutup shift, rekonsiliasi, stok opname, temuan dan review |
| [07-data-model.md](07-data-model.md) | Entitas, relasi, tipe data, constraint, indeks, audit trail |
| [08-architecture-nfr.md](08-architecture-nfr.md) | Modul, stack rekomendasi, keamanan, performa, deployment |
| [09-api-contracts.md](09-api-contracts.md) | Konvensi REST, endpoint dan contoh payload |
| [10-ux-navigation.md](10-ux-navigation.md) | Peta layar dan requirement pengalaman pengguna |
| [11-acceptance-criteria.md](11-acceptance-criteria.md) | User stories, skenario uji dan Definition of Done |
| [12-roadmap-backlog.md](12-roadmap-backlog.md) | Fase, dependensi, prioritas dan backlog |
| [13-decisions-risks.md](13-decisions-risks.md) | ADR awal, risiko, asumsi, pertanyaan terbuka |
| [14-glossary.md](14-glossary.md) | Istilah operasional dan teknis |
| [15-status-transitions.md](15-status-transitions.md) | State machine, event dan invariant lintas modul |

## Prinsip yang tidak boleh dilanggar

1. **Shopping list tidak sama dengan stok tersedia.** Kuantitas hanya bertambah setelah penerimaan diverifikasi/posting.
2. **Stok bersumber dari stock movement ledger**, bukan angka di master produk yang diedit langsung.
3. **Harga beli dan harga jual adalah snapshot transaksi.** Pembaruan master tidak mengubah sejarah.
4. **Penjualan dan pengurangan stok diposting atomik**, hanya setelah pembayaran dinyatakan sah.
5. **Pembayaran QRIS dan settlement ke bank adalah dua peristiwa berbeda.** Screenshot pembeli bukan bukti final.
6. **Kas fisik, penjualan, pengeluaran pembelian dan biaya operasional berbeda konsep.** Pembelian stok tidak otomatis seluruhnya menjadi HPP saat itu.
7. **Audit mencatat siapa, kapan, perubahan dan alasan.** Tidak boleh menghapus histori transaksi terposting tanpa mekanisme pembalik.
8. **Role-based permission di server**, bukan mengandalkan tombol tersembunyi di frontend.
9. **MVP satu warung, desain siap banyak lokasi** menggunakan `store_id` pada peristiwa bisnis.
10. **Sederhana untuk operator; ketat untuk akurasi data.** Tidak menambahkan modul rental lapangan.

## Hasil yang ingin dicapai pada MVP

- Owner membuat/mengekspor daftar belanja; Admin menerima, memverifikasi dan merekonsiliasi biaya pembelian.
- Kasir menambahkan item ke keranjang, menagih cash atau QRIS, dan mencetak struk secara opsional.
- Stok, pembayaran, pengeluaran serta harga pokok tersimpan dengan hubungan yang dapat diaudit.
- Admin menutup shift, mengisi uang fisik, memeriksa selisih, dan menyiapkan laporan harian.
- Owner memperoleh dashboard penjualan, margin, arus kas operasional, stok rendah, selisih, dan aktivitas pengguna.

## Pengelolaan perubahan

- Dokumentasikan perubahan perilaku bisnis pada file domain terkait dan cantumkan pembaruan di `13-decisions-risks.md` jika memengaruhi desain.
- Tandai requirement dengan identitas (contoh: `INV-001`, `POS-004`) agar dapat ditautkan ke issue, API test, dan QA.
- Perubahan status atau nilai transaksi terposting harus dijelaskan beserta dampaknya pada stok, kas dan audit.
- Konfirmasi asumsi bisnis terbuka sebelum menjadikannya kode atau kebijakan akuntansi final.
