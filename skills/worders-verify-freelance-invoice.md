---
name: Verify a freelancer's invoice against their purchase orders
description: Cross-check a submitted freelance invoice against the freelancer's purchase orders and jobs in Worders before approving payment.
api: openapi/worders-api-openapi-original.yml
operations:
  - GET /V1/freelancers
  - GET /V1/freelancers/{freelancer_id}/purchase_orders
  - GET /V1/purchase_orders/{po_number}
  - GET /V1/invoices
  - GET /V1/invoices/{id}
generated: '2026-07-21'
method: generated
---

# Verify a freelance invoice

Authenticate every call with a service API key: `Authorization: Bearer wrd_live_...`
(issued from the admin UI). All endpoints are under `https://api.worders.net`.
Note: this spec declares no operationIds — operations are referenced by METHOD + path.

1. **Find the freelancer** — `GET /V1/freelancers?name=...` and pick the matching
   `id` from the results.
2. **Pull their purchase orders** — `GET /V1/freelancers/{freelancer_id}/purchase_orders`.
   Each PurchaseOrderSummary carries `po_number`, `status`, `total_amount`, `currency`,
   and `jobs_count`.
3. **Open the PO named on the invoice** — `GET /V1/purchase_orders/{po_number}` returns
   the PurchaseOrderDetail with every Job (`job_no`, `status`, `price`, `currency`,
   `language_pair`, `order_reference`, `delivered_on`).
4. **Locate the invoice** — `GET /V1/invoices` (filter with `page`/`per_page` and query
   params), then `GET /V1/invoices/{id}` for the full record: `invoice_number`,
   `po_number`, `totals`, `status`, `payment_due_date`, and `items[]` lines.
5. **Reconcile** — match invoice lines to delivered jobs on the PO: amounts, currency,
   and `order_reference` must agree, and jobs should be in a delivered status before
   the invoice is approved.

Error handling: a `401` means the bearer key or session is invalid; a `404` on
`/V1/invoices/{id}` or `/V1/purchase_orders/{po_number}` means an unknown id — errors
arrive as `{"error": {"code", "message", "details"}}`. Lists paginate with
`page`/`per_page` and echo a `meta` block (`total`/`page`/`per_page`); walk pages until
`page * per_page >= total`.
