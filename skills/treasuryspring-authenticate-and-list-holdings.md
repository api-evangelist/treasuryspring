---
name: Authenticate and list holdings
description: Exchange client credentials for a bearer token, discover the entities you can act on, and list their live holdings.
api: openapi/treasuryspring-openapi-original.json
operations: [post.token, get.entities, get.holdings, get.holding]
base_url: https://api.treasuryspring.com/api/v1
sandbox_url: https://api.sandbox.treasuryspring.com/api/v1
---

# Authenticate and list holdings

Use this to read a client's live TreasurySpring investment positions.

## Steps

1. **Get an access token** — `post.token` (`POST /oauth/token`).
   - Send `Authorization: Basic <base64(client_id:client_secret)>` and a
     `application/x-www-form-urlencoded` body with `grant_type=client_credentials`.
   - The response has `access_token`, `expires_in` (seconds), and `refresh_token`.
2. **Discover entities** — `get.entities` (`GET /entity`).
   - Returns the organisations your API user can act on behalf of (`code`, `name`).
     Most other calls require an `entity_code`.
3. **List holdings** — `get.holdings` (`GET /holding?entity_code=<code>`).
   - Paginate with `limit`/`offset`; use `PageInfo.hasNextPage` to continue.
   - Optional date filters: `min/max_subscription_date`, `min/max_maturity_date`,
     `min_modified_date` (all `YYYY-MM-DD`).
4. **Drill into one holding** — `get.holding` (`GET /holding/{entity_code}/{holding_uid}`)
   for full detail (value, yield, maturity dates, extension terms, maturity actions).

## Rules

- Send `Authorization: Bearer <access_token>` on every call after step 1.
- All amounts/values are currency-scoped; read each holding's `currency`.
- Validation failures return HTTP 422 with a `{ "detail": [...] }` body — inspect
  `detail[].loc`/`msg`. See `errors/treasuryspring-problem-types.yml`.
- Prefer the sandbox base URL while testing.
