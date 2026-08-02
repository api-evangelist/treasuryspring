---
name: Subscribe to a Fixed Term Fund
description: Discover available indications for an entity, validate a subscription, submit it, and track it until it becomes a live holding.
api: openapi/treasuryspring-openapi-original.json
operations: [get.entities, get.indications, get.indication, post.validate, post.subscribe, get.subscription, get.holding]
base_url: https://api.treasuryspring.com/api/v1
sandbox_url: https://api.sandbox.treasuryspring.com/api/v1
---

# Subscribe to a Fixed Term Fund (FTF)

A subscription is the request to invest in an FTF; a holding is the live record
created once an accepted subscription has been processed.

## Steps

1. **Pick the entity** — `get.entities` (`GET /entity`) to get the `entity_code`
   that will invest.
2. **Browse opportunities** — `get.indications` (`GET /indication/{code}`) lists the
   FTFs the entity can currently subscribe to (summary: `uid`, currency, product,
   indicativeYield, sector, term).
3. **Inspect one indication** — `get.indication`
   (`GET /indication/{code}/{uid}`) for full terms (subscription dates, minimum/
   maximum, obligor exposure, extension fields).
4. **Validate first** — `post.validate` (`POST /validate`) to pre-check a prospective
   subscription (amount, indication, entity) before committing. Fix any 422 issues.
5. **Submit** — `post.subscribe` (`POST /subscribe`) to submit the subscription request.
   Capture the returned `subscription.uid`.
6. **Track to holding** — poll `get.subscription`
   (`GET /subscription/{entity_code}/{uid}`) until `holdingUid` is populated.
7. **Confirm the live position** — once `holdingUid` exists, call `get.holding`
   (`GET /holding/{entity_code}/{holdingUid}`) and treat the holding as the source of truth.

## Rules

- Always `post.validate` before `post.subscribe`; the API documents **no idempotency
  key**, so do not blindly retry a `POST /subscribe` — re-validate and re-check
  subscription state first (see `conventions/treasuryspring-conventions.yml`).
- Authenticate per `skills/treasuryspring-authenticate-and-list-holdings.md`.
- Respect each indication's subscription window and min/max amount.
