---
name: Register and verify Ostrom webhooks
description: Register a partner webhook subscription, trigger a test event, and verify the X-Ostrom-Signature HMAC-SHA1 payload signature.
api: openapi/ostrom-openapi-original.json
operations: [createOAuth2Token, createWebhook, testWebhook, getWebhooks, deleteWebhook]
---

# Register and verify webhooks

For PARTNER clients receiving Ostrom events over HTTP.

## Auth
1. `createOAuth2Token` (`POST /oauth2/token`) with PARTNER credentials; use the Bearer token.

## Steps
1. Generate a secure secret (pseudorandom, >= 128-bit), e.g.
   `crypto.randomBytes(32).toString("hex")`. Store it server-side.
2. Register the subscription — `createWebhook` (`POST /webhooks`) with body
   `{ "url": "<your https endpoint>", "secret": "<secret>" }`.
3. Trigger a test delivery — `testWebhook` (`POST /webhooks/{id}/test`).
4. In your receiver, verify each `POST`:
   - Read `X-Ostrom-Signature`.
   - Compute `sha1=` + HMAC-SHA1 hex digest of the raw JSON body keyed by your secret.
   - Compare with a timing-safe equality check (never `==`); reject on mismatch.
5. Manage subscriptions with `getWebhooks` (`GET /webhooks`), `getWebhook`
   (`GET /webhooks/{id}`), and `deleteWebhook` (`DELETE /webhooks/{id}`).

## Rules
- Only PARTNER clients can manage webhooks.
- Signature algorithm is HMAC-SHA1; the payload is the UTF-8 JSON request body.
- Rate limit 50 req/min; errors are `{ type, detail }` JSON.
