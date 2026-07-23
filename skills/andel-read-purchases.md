---
name: Read member purchases from the Andel Data Exchange
description: Authenticate to the Andel Data Exchange API and list or look up member purchase events for the plans you are authorized for.
api: openapi/andel-data-exchange-openapi.yaml
operations: [listPurchases, getPurchase]
---

# Read member purchases

Use this skill to pull member purchase events (an RFC-9457-style purchase-data interchange, not a regulated-claims feed) from Andel for the plans your credentials are authorized for.

## Authenticate

1. Obtain an OAuth 2.0 client-credentials access token from Descope:
   `POST https://api.descope.com/oauth2/v1/apps/token` with your client id/secret and scope `purchases:read`.
2. The returned bearer token carries a `plans` claim — you will only see purchases for those plans.
3. Send it as `Authorization: Bearer <token>` on every request.

## List purchases (`listPurchases`)

- `GET /purchases` — ordered by `purchased_at` descending.
- Filter with query params: `member_id`, `plan_id`, `ndc` (NDC-11), `prescriber_spi`, `event_type`, `status`.
- Page forward with cursor params `since` (cursor or ISO 8601 UTC timestamp) and `limit`; bound with `until`.
- Loop: keep the last item's cursor and pass it as `since` until a short/empty page is returned.

## Get one purchase (`getPurchase`)

- `GET /purchases/{purchase_id}` for a single record.

## Errors & tracing

- Errors are RFC 9457 `application/json` Problem objects (`type`, `title`, `status`, `detail`, `andel_request_id`).
- On `401` refresh the token; on `403` the plan is outside your `plans` claim; on `429` back off and retry; on `5xx` retry with backoff and cite `andel_request_id` to support.

## Test first

Point at the Postman mock sandbox `https://7403d846-765d-4d63-9e5c-b7f0ab21a354.mock.pstmn.io/exchange/v1` (auth not enforced) before hitting production `https://api.andel.org/exchange/v1`.
