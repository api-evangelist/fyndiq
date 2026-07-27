---
name: Manage the Fyndiq product catalog
description: Create, list, update, and remove product articles on the Fyndiq marketplace using the Merchant API.
api: openapi/fyndiq-merchant-api-openapi.yml
operations: [createArticle, listArticles, retrieveArticle, retrieveArticleBySku, updateArticle, updateArticlePrice, updateArticleQuantity, deleteArticle, listCategories]
---

# Manage the Fyndiq product catalog

Operate a merchant's product catalog on Fyndiq (Sweden's bargain marketplace) through the Merchant API.

## Authentication
Every request uses HTTP Basic Authentication: send an `Authorization: Basic <base64(merchantID:token)>` header. Use `Content-Type: application/json`. TLS 1.2+ is required. Base URLs: live `https://merchants-api.fyndiq.se/api/v1/`, sandbox `https://merchants-api.sandbox.fyndiq.se/api/v1/` (request sandbox credentials from Fyndiq).

## Steps
1. **Discover categories.** Call `listCategories` (`GET /categories/{market}/{language}`) for the target market (e.g. `SE`) to get valid category values before creating articles.
2. **Create an article.** Call `createArticle` (`POST /articles`) with localized `title`/`description` arrays, `sku`, `quantity`, per-market `price`, `images`, and `categories`. A `201` returns the new article `id`. A `409` means the SKU already exists — retrieve it instead with `retrieveArticleBySku` (`GET /articles/sku/{sku}`).
3. **List / inspect.** Use `listArticles` (`GET /articles`, paginated up to 1000) to enumerate the catalog, or `retrieveArticle` (`GET /articles/{article_id}`) for one item.
4. **Update.** Use the narrow endpoints when possible: `updateArticlePrice` (`PUT /articles/{article_id}/price`) and `updateArticleQuantity` (`PUT /articles/{article_id}/quantity`); or `updateArticle` (`PUT /articles/{article_id}`) for other fields. Success is `204`.
5. **Remove.** Call `deleteArticle` (`DELETE /articles/{article_id}`) — this soft-deletes (sets status to deleted); a `204` confirms.

## Rules
- Errors return a `{ "description": ..., "errors": {...} }` envelope (not RFC 9457). Handle `400` (invalid payload — read `errors` for offending fields), `401` (bad credentials), `403` (permission denied), `404`/`409`.
- No idempotency key is supported; guard creates by checking SKU existence first.
- For high volume use the bulk endpoints (`bulkCreateArticles` ≤100, `bulkUpdateArticles` ≤200) instead of looping single calls.
