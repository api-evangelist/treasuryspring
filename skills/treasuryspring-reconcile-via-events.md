---
name: Reconcile holdings via the event stream and webhooks
description: Consume the normalised event stream with a durable checkpoint cursor, and register a webhook for push notifications.
api: openapi/treasuryspring-openapi-original.json
operations: [get.events, put.event.checkpoint, patch.event.checkpoint, post.webhook, delete.webhook]
base_url: https://api.treasuryspring.com/api/v1
sandbox_url: https://api.sandbox.treasuryspring.com/api/v1
---

# Reconcile holdings via the event stream and webhooks

TreasurySpring emits normalised lifecycle events (SUBSCRIBED, ISSUED, EXTENDED,
FINALIZED, REDEEMED, CANCELLED, ...) for integration and reconciliation.

## Pull the stream (durable cursor)

1. **Create/get a checkpoint** — `put.event.checkpoint`
   (`PUT /event/checkpoint/{name}`) to establish a named, server-managed cursor.
2. **Read events** — `get.events` (`GET /event`), passing `start_cursor` = your
   checkpoint's position. Also supports `end_cursor`, `min_created_at`, `limit`.
   Each page returns `EventPageInfo.endCursor` and `hasNextPage`.
3. **Advance the checkpoint** — `patch.event.checkpoint`
   (`PATCH /event/checkpoint/{name}`) with the latest `endCursor` so the next run
   resumes exactly where you stopped (idempotent, at-least-once semantics).
4. Loop while `hasNextPage` is true.

## Push (webhooks)

- **Register** — `post.webhook` (`POST /webhook`) with `{ "url": "https://.../hook" }`
  to receive notifications at your callback URL.
- **Deregister** — `delete.webhook` (`DELETE /webhook`) to stop notifications.

## Rules

- Events are discriminated by `eventType`; branch on it (see
  `asyncapi/treasuryspring-events-webhooks.yml` for the full type list).
- Treat delivery as at-least-once: dedupe on the event's stable cursor/id and make
  your handler idempotent — the API itself documents no idempotency key.
- Authenticate per `skills/treasuryspring-authenticate-and-list-holdings.md`.
