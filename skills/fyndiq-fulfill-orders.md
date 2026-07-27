---
name: Fulfil Fyndiq orders
description: Retrieve marketplace orders and mark them fulfilled with tracking, or cancel them, using the Merchant API.
api: openapi/fyndiq-merchant-api-openapi.yml
operations: [listOrders, retrieveOrder, fulfillOrder, cancelOrder, createTestOrder]
---

# Fulfil Fyndiq orders

Process orders that Fyndiq buyers place against a merchant's articles.

## Authentication
HTTP Basic Authentication (`Authorization: Basic <base64(merchantID:token)>`), `Content-Type: application/json`, TLS 1.2+. Live base `https://merchants-api.fyndiq.se/api/v1/`; sandbox `https://merchants-api.sandbox.fyndiq.se/api/v1/`.

## Steps
1. **Poll for new orders.** Call `listOrders` (`GET /orders/`). By default new orders are returned; filter with the `state` query parameter (`CREATED` / `FULFILLED` / `CANCELLED`).
2. **Inspect an order.** Call `retrieveOrder` (`GET /orders/{order_id}`) to read `article_id`, quantity, price/VAT breakdown, `shipping_address`, and the `fulfillment_deadline`.
3. **Fulfil.** Once shipped, call `fulfillOrder` (`PUT /orders/{order_id}/fulfill`) with a `tracking_information` array of `{ carrier_name, tracking_number }`. A `202` confirms. A `403` means the order is already fulfilled/cancelled or has passed its fulfilment deadline.
4. **Cancel.** If you cannot ship (e.g. out of stock), call `cancelOrder` (`PUT /orders/{order_id}/cancel`). A `202` confirms; `403` means it is already in a terminal state.

## Sandbox
- In the sandbox only, call `createTestOrder` (`POST /orders/`) to generate a test order so you can exercise this whole flow end to end. This endpoint is not available in production.

## Rules
- Respect the `fulfillment_deadline` — fulfilling after it returns `403`.
- Errors use the `{ "description": ... }` envelope; treat `403` states as non-retryable for that order.
