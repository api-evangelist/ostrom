---
name: Fetch day-ahead electricity spot prices
description: Authenticate with the Ostrom API and retrieve day-ahead EEX spot prices for a German zip code and time window.
api: openapi/ostrom-openapi-original.json
operations: [createOAuth2Token, getSpotPrices]
---

# Fetch day-ahead spot prices

Retrieve day-ahead electricity spot prices from Ostrom for a given zip code and time window.

## Auth
1. Get an access token — `createOAuth2Token` (`POST /oauth2/token`). Use HTTP Basic
   auth with base64(`client_id:client_secret`) and body `grant_type=client_credentials`.
   Against `https://auth.sandbox.ostrom-api.io/oauth2/token` (sandbox) or the
   production auth host. The response returns `access_token` (Bearer, `expires_in` ~3599s).

## Steps
1. Call `getSpotPrices` (`GET /spot-prices`) with `Authorization: Bearer <access_token>`.
   Required query params: `startDate`, `endDate` (ISO 8601, **UTC**), and `resolution`
   (`HOUR` | `DAY` | `MONTH`). Optional `zip` (German postal code).
2. Read the `data[]` array; each item is a price interval where `date` is the *valid-from*
   timestamp (e.g. `2023-10-22T01:00:00.000Z` covers 01:00–02:00).

## Rules
- Tomorrow's price loads daily between 3–5pm CET/CEST; query after that for next-day data.
- If `zip` is omitted, `netKwhTaxAndLevies`, gross fees, and grid-fee fields return `0`.
- Rate limit: 50 req/min. On `429 too_many_requests`, back off using `X-RateLimit-Reset`.
- Errors use a flat `{ type, detail }` JSON envelope (see errors/ostrom-problem-types.yml).
