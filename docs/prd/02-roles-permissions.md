# 02 — Actors, Roles & Authorization

## 1. Model otorisasi

Gunakan **RBAC server-side** berbasis permission `resource.action` dan `store_id`. Menu UI mengikuti izin dari server, tetapi endpoint tetap memeriksa hak akses. Akun Owner/Admin/Cashier dapat memiliki role configurable; tidak perlu menanamkan logika role string ke dalam controller. Customer belum wajib dibuat.

Perubahan permission, override, void, dan koreksi stok harus masuk `audit_logs`. Untuk bisnis kecil, role default disediakan; Owner dapat membuat varian role pada fase pengembangan.

## 2. Role bawaan

- **OWNER:** akses semua lokasi, pengaturan, user, audit, laporan, stok, POS, pembelian, approval; aktivitas override tetap diaudit.
- **ADMIN:** membuat daftar belanja, memeriksa struk yang diunggah Cashier, approve/reject receipt sehingga stok masuk, mengelola katalog, pembayaran pembelian, POS dan closing; **tidak** dapat mengambil alih seluruh hak Owner.
- **CASHIER:** menginput pembelian aktual dan mengunggah struk (hanya biaya pada input miliknya terlihat, bukan harga modal historis), submit/resubmit untuk review; melakukan POS dan submit penutupan. Tidak dapat membuat shopping list, approve/reject receipt, mengubah harga aktif, atau mem-post stok.
- **CUSTOMER (future):** riwayat pembelian pribadi/loyalty jika akun dibuat; walk-in tidak memerlukan akun.
- **RIDER:** tidak membuat user; status belanja cukup mewakili pengerjaan.

## 3. Matriks permission awal

Legenda: **F** = penuh, **Y** = boleh, **A** = butuh approval, **R** = baca terbatas, **—** = tidak.

| Permission / Aksi | Owner | Admin | Cashier |
| --- | :---: | :---: | :---: |
| `users.manage` | F | — | — |
| `permissions.manage` | F | — | — |
| `config.manage` | F | R | — |
| `products.read` | F | Y | R |
| `products.write` | F | Y | — |
| `purchase_list.create/export` | F | Y | — |
| `purchase_receipt.create/submit/upload_receipt` | F | Y | Y |
| `purchase_receipt.review/approve/reject` | F | Y | — |
| `purchase_payment.settle` | F | Y | — |
| `inventory.view` | F | Y | R** |
| `inventory.adjust` | F | A | — |
| `pos.sell` | F | Y | Y |
| `pos.discount` | F | Y* | A |
| `pos.void/refund` | F | Y* | A |
| `pos.open/submit_close` | F | Y | Y |
| `closing.approve` | F | Y*** | — |
| `finance.expense.create` | F | Y | — |
| `reports.operational` | F | Y | R** |
| `reports.profit/cogs` | F | Y* | — |
| `audit_logs.read` | F | R | — |

\* Dapat dipersempit konfigurasi Owner.  
\** Hanya item/summary relevan untuk POS aktif miliknya.  
\*** Admin tidak boleh menyetujui closing miliknya sendiri jika kebijakan two-person control aktif; pada usaha satu operator Owner bisa memberi exception tertulis.

## 4. Kebijakan akses detail

1. Setiap endpoint melakukan autentikasi, cek permission, ownership, dan batas `store_id`.
2. Pelaku transaksi tidak boleh diam-diam mengganti `created_by`, `approved_by`, `store_id`, total, atau timestamp dari payload klien.
3. Perubahan role/logout melakukan invalidasi session sesuai kebijakan.
4. Harga modal, biaya, margin, serta laporan laba bukan data yang boleh diakses Cashier.
5. Tombol "void", "adjust", atau "override" membawa alasan wajib dan approver ketika ditetapkan.
6. Owner memiliki seluruh hak bisnis, tetapi tidak dapat menghapus jejak audit terposting.
7. Login salah berulang diberi throttling, error tidak membocorkan akun terdaftar.
8. Gunakan role permission dinamis; administrasi permission custom ditunda bila perlu, namun skemanya disiapkan sejak awal.

## 5. Acceptance

- **AUTH-001:** Cashier yang memanggil API pembelian langsung mendapat `403`, meskipun memakai request manual.
- **AUTH-002:** Admin tidak dapat mengubah role dirinya menjadi Owner.
- **AUTH-003:** User yang tidak terhubung ke suatu store tidak dapat melihat transaksinya.
- **AUTH-004:** Aksi void/penyesuaian menyimpan actor, alasan, approver, dan waktu.
- **AUTH-005:** Menonaktifkan akun membuat login baru ditolak dan session dibatalkan sesuai kebijakan.

## 6. Maker-checker untuk pembelian

- Owner **dan** Admin boleh membuat shopping list; Cashier hanya melihat list aktif yang ditugaskan/dalam store.
- Cashier menginput realisasi, harga beli dari struk, qty dan usulan harga jual; akses biaya terbatas pada receipt yang dibuatnya.
- File struk wajib diunggah sebelum submit. Status submitted belum menambah stok.
- Owner/ Admin mengecek kecocokan struk dan item. Satu approval cukup untuk MVP dan otomatis membuat stock IN; uploader/inputter tidak boleh approve sendiri.
- Saat reject, approver memberi alasan; Cashier memperbaiki dan resubmit. Semua keputusan tercatat di audit.
- Keputusan apakah Owner harus selalu melakukan approval final walaupun Admin sudah memverifikasi tetap [OPEN] untuk fase kebijakan berikutnya.
