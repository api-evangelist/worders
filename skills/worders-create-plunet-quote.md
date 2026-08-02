---
name: Create and track a Plunet quote
description: Create a quote in Plunet through the Worders API and read it back with its per-language price lines.
api: openapi/worders-api-openapi-original.yml
operations:
  - GET /V1/customers
  - POST /V1/quotes
  - GET /V1/quotes
  - GET /V1/quotes/{id}
generated: '2026-07-21'
method: generated
---

# Create and track a quote

Authenticate with `Authorization: Bearer wrd_live_...` (service API key from the
admin UI); `POST /V1/quotes` accepts bearer keys only. Base URL
`https://api.worders.net`. This spec declares no operationIds — operations are
referenced by METHOD + path.

1. **Resolve the customer** — `GET /V1/customers?name=...`.
2. **Create the quote** — `POST /V1/quotes` with a JSON body whose top-level key is
   `quote` (a QuoteCreateRequest). Omitting the `quote` key returns `400 missing
   quote key`; an unknown customer returns `404`.
3. **Confirm** — a `201` returns QuoteCreateResponse (`id`, `plunet_id`,
   `order_reference`, `status`, `external_id`, `items`, `urls`).
4. **Read it back** — `GET /V1/quotes/{id}` returns the Quote with `valid_until`,
   `project_name`, client currency pricing, and `items[]`; each QuoteItem carries a
   `price_line[]` of PriceLineEntry rows (service, unit, amount, unit_price, total)
   per language pair.
5. **Monitor** — `GET /V1/quotes` lists quotes (paginated with `page`/`per_page` +
   `meta`); a quote id passed to `GET /V1/orders/{id}` deliberately returns `404`.

No idempotency key is documented — verify via `GET /V1/quotes` before retrying a
create. Errors use the `{"error": {"code", "message", "details"}}` envelope.
