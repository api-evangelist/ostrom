---
name: Link a partner user and read their contracts
description: As a PARTNER client, link an Ostrom customer, then read their orders, contracts, and smart-meter consumption by external user id.
api: openapi/ostrom-openapi-original.json
operations: [createOAuth2Token, linkUser, getContractsByExternalUserId, getContractEnergyConsumptionByExternalUserId, getOrdersByExternalUserId]
---

# Link a partner user and read their data

For PARTNER clients that build multi-user apps. A user must consent before a partner
can access their data.

## Auth
1. `createOAuth2Token` (`POST /oauth2/token`) with your PARTNER `client_id`/`client_secret`
   via HTTP Basic + `grant_type=client_credentials`. Use the returned Bearer token.

## Steps
1. Link the customer — `linkUser` (`POST /users/link`, role PARTNER). This associates an
   Ostrom user with your partner client under a partner-scoped `externalUserId`.
2. List the user's contracts — `getContractsByExternalUserId`
   (`GET /users/{externalUserId}/contracts`).
3. Pull smart-meter consumption — `getContractEnergyConsumptionByExternalUserId`
   (`GET /users/{externalUserId}/contracts/{contractId}/energy-consumption`) with required
   `startDate`, `endDate` (ISO 8601 UTC) and `resolution` (`HOUR`/`DAY`/`MONTH`).
4. Optionally read orders — `getOrdersByExternalUserId` (`GET /users/{externalUserId}/orders`).
5. To revoke, `disconnectUser` (`DELETE /users/{externalUserId}/disconnect`).

## Rules
- PARTNER role required; the user must have consented to share data with your client.
- Use the sandbox test accounts (sandbox/ostrom-sandbox.yml) to exercise linking end to end.
- Rate limit 50 req/min; errors are `{ type, detail }` JSON.
