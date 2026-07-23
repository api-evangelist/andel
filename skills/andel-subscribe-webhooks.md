---
name: Subscribe to Andel purchase webhooks
description: Register, list, and delete webhook subscriptions so Andel POSTs purchase.created events to your HTTPS endpoint.
api: openapi/andel-data-exchange-openapi.yaml
operations: [createSubscription, listSubscriptions, deleteSubscription]
---

# Subscribe to purchase webhooks

Use this skill to receive real-time `purchase.created` events instead of polling `GET /purchases`.

## Authenticate

Use the same OAuth 2.0 client-credentials bearer token as the read skill (Descope, scope `purchases:read`), sent as `Authorization: Bearer <token>`.

## Create a subscription (`createSubscription`)

- `POST /webhooks/subscriptions` with a JSON body:
  - `url` (required) — the HTTPS endpoint Andel will POST events to.
  - `event_types` (required) — array; currently `["purchase.created"]`.
  - `description` (optional) — human-readable label.
- The `201` response is a `SubscriptionWithSecret`: **capture the returned secret now** — it is shown once and is used to verify event signatures.

## List subscriptions (`listSubscriptions`)

- `GET /webhooks/subscriptions` returns your active subscriptions.

## Delete a subscription (`deleteSubscription`)

- `DELETE /webhooks/subscriptions/{subscription_id}` — returns `204`.

## Handling delivered events

- Each event payload is a `Purchase` object (same shape as the read endpoints).
- Verify the signature against the stored subscription secret before trusting an event.
- Errors on the management endpoints are RFC 9457 Problem objects; see `errors/andel-problem-types.yml`.
