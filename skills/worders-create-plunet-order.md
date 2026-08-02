---
name: Create a Plunet order from a customer template
description: Create a translation order in Plunet through the Worders API, optionally starting from one of the customer's Plunet order templates.
api: openapi/worders-api-openapi-original.yml
operations:
  - GET /V1/customers
  - GET /V1/customers/{customer_id}/templates
  - POST /V1/orders
  - GET /V1/orders/{id}
generated: '2026-07-21'
method: generated
---

# Create a Plunet order

Write operations require a service API key (`Authorization: Bearer wrd_live_...`);
the session cookie is not accepted on `POST /V1/orders`. Base URL
`https://api.worders.net`. This spec declares no operationIds — operations are
referenced by METHOD + path.

1. **Resolve the customer** — `GET /V1/customers?name=...`; keep the customer `id`
   (and note the Plunet cross-reference ids on returned resources).
2. **List their templates (optional)** — `GET /V1/customers/{customer_id}/templates`
   returns the PlunetTemplate list (`id`, `name`, `description`) available for that
   customer.
3. **Create the order** — `POST /V1/orders` with a JSON body whose top-level key is
   `order` (an OrderCreateRequest). Omitting the `order` key returns `400 missing
   order key`; an unknown customer returns `404`.
4. **Confirm** — a `201` returns OrderCreateResponse: local `id`, `plunet_id`,
   `order_reference`, `status`, `external_id`, `items`, and `urls`. Follow up with
   `GET /V1/orders/{id}` to read the full Order (client, items, totals, due_date).

There is no documented idempotency key on this API — do not blindly retry a POST
that may have succeeded; instead list `GET /V1/orders` and check for the
`order_reference`/`external_id` before re-submitting. Errors use the
`{"error": {"code", "message", "details"}}` envelope.
