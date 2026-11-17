# 09 — API Contract (Proposed REST v1)

**Catatan:** Contoh rancangan API, belum diimplementasikan. Semua endpoint memerlukan auth dan permission kecuali login/health. Resource selalu ditetapkan dalam store scope yang diotorisasi server. Format response final dapat disusun menjadi OpenAPI saat implementasi.

## 1. Konvensi

Base path `/api/v1`. JSON body; UTC ISO-8601 timestamps; IDR integer Rupiah pada `amount`, `price`; quantities decimal serialized sebagai string jika membutuhkan presisi; pagination `limit/cursor`. Semua write signifikan menerima `Idempotency-Key`.

Success contoh:

```json
{"data":{"id":"sale_01","status":"PAID","total":19000},"meta":{"request_id":"req_01"}}
```

Error contoh:

```json
{"error":{"code":"INSUFFICIENT_STOCK","message":"Stok Aqua 600 ml tidak cukup","details":{"available_qty":"1","requested_qty":"2"}},"meta":{"request_id":"req_02"}}
```

HTTP: `400` invalid input; `401` unauthenticated; `403` forbidden; `404` not found/out-of-scope; `409` conflict/status/stock/duplicate; `422` business validation; `429` rate limited; `500` unexpected. Jangan membocorkan keberadaan resource store lain.

## 2. Auth/config/catalog

| Method | Endpoint | Permission |
| --- | --- | --- |
| POST | `/auth/login` | Public |
| POST | `/auth/logout` | Authenticated |
| GET | `/me` | Authenticated |
| GET/POST | `/stores/{storeId}/categories` | products.read/write |
| GET/POST | `/stores/{storeId}/products` | products.read/write |
| PATCH | `/stores/{storeId}/products/{id}` | products.write |
| GET | `/stores/{storeId}/settings` | config.read |
| PATCH | `/stores/{storeId}/settings` | config.manage |

## 3. Shopping list & receiving

| Method | Endpoint | Perilaku |
| --- | --- | --- |
| POST | `/stores/{storeId}/shopping-lists` | Buat draft |
| GET | `/stores/{storeId}/shopping-lists` | List/filter status |
| PATCH | `/stores/{storeId}/shopping-lists/{id}` | Edit draft |
| POST | `/stores/{storeId}/shopping-lists/{id}/start` | Set ON_PROCESS |
| GET | `/stores/{storeId}/shopping-lists/{id}/export` | Print/CSV list |
| POST | `/stores/{storeId}/purchase-receipts` | Catat receiving draft |
| POST | `/stores/{storeId}/purchase-receipts/{id}/post` | Verify dan stock IN secara atomik |
| POST | `/stores/{storeId}/purchase-receipts/{id}/payments` | Settlement dana, tidak mengulang stok IN |

Contoh receiving:

```json
{
  "shopping_list_id":"list_01",
  "supplier_name":"Toko Grosir",
  "items":[
    {"product_id":"prod_aqua","shopping_list_item_id":"line_01","accepted_qty":"24","purchased_unit_cost":3000,"landed_unit_cost":3000}
  ],
  "note":"Nota fisik diperiksa"
}
```

Response draft memuat `status=DRAFT`. Endpoint `/post` memvalidasi dokumen dan melakukan `posted_at`, `verified_by`, `stock_movements` IN.

## 4. Inventory

| Method | Endpoint | Perilaku |
| --- | --- | --- |
| GET | `/stores/{storeId}/inventory/balances` | Stok saat ini dan minimum |
| GET | `/stores/{storeId}/inventory/movements` | Ledger/filter product, date |
| POST | `/stores/{storeId}/inventory/adjustments` | Adjustment beralasan dan approval |
| GET | `/stores/{storeId}/inventory/low-stock` | Under min stock |

## 5. POS

| Method | Endpoint | Perilaku |
| --- | --- | --- |
| POST | `/stores/{storeId}/pos-sessions` | Open shift |
| GET | `/stores/{storeId}/sales` | List sales by permission |
| POST | `/stores/{storeId}/sales` | Create pending checkout |
| POST | `/stores/{storeId}/sales/{id}/confirm-payment` | Atomik paid + stock OUT |
| POST | `/stores/{storeId}/sales/{id}/void` | Reversal dan audit |
| GET | `/stores/{storeId}/sales/{id}/receipt` | Printable receipt |
| POST | `/stores/{storeId}/pos-sessions/{id}/submit-close` | Counted cash & variance |
| POST | `/stores/{storeId}/pos-sessions/{id}/approve-close` | Admin/Owner verifies |

Checkout request:

```json
{
  "session_id":"sess_01",
  "items":[{"product_id":"prod_aqua","qty":"2"},{"product_id":"prod_pocari","qty":"1"}],
  "client_total":19000
}
```

Server menghitung `total` sendiri dan mengembalikan harga versi server; `client_total` hanya pemeriksaan konflik.

Payment confirmation:

```json
{
  "method":"CASH",
  "cash_received":20000,
  "confirmation_source":"CASHIER_COUNT"
}
```

Untuk QRIS: `method=QRIS`, `confirmation_source=MERCHANT_APP`, `merchant_reference` bila ada. Server tidak boleh mengesahkan pembayaran QRIS secara otomatis hanya karena frontend menyatakan berhasil; permission dan prosedur verifikasi diperlukan.

## 6. Finance, closing, audit

| Method | Endpoint | Perilaku |
| --- | --- | --- |
| POST | `/stores/{storeId}/expenses` | Create expense |
| POST | `/stores/{storeId}/cash-movements` | Cash IN/OUT |
| GET | `/stores/{storeId}/reports/sales` | Sales by period |
| GET | `/stores/{storeId}/reports/inventory` | Stock valuation/movements |
| GET | `/stores/{storeId}/reports/finance` | Operational profit/cash |
| POST | `/stores/{storeId}/daily-closings` | Submit daily close |
| POST | `/stores/{storeId}/daily-closings/{id}/approve` | Verify |
| GET | `/stores/{storeId}/audit-logs` | Owner/Admin scoped |

## 7. Cross-cutting guarantees

1. `Idempotency-Key` disimpan dengan scoped actor/store/operation dan body hash; pengulangan key+body identik mengembalikan hasil sama; key sama payload berbeda = 409.
2. Pagination dan export menghormati store/permission; sort and filter di-allowlist.
3. Semua money/qty divalidasi positif dan server menghitung financial totals.
4. Posted state transitions tidak dapat dilakukan dengan generic PATCH; gunakan action endpoint.
5. Correlation/request ID dicatat pada business event dan audit log.
6. API error tidak memaparkan stacktrace/secret.
7. Refund partial, bank settlement API, dan scanner integration akan disusun sebagai endpoint tambahan pada fase berikutnya.
