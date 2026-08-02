---
name: Instrument a user lifecycle in Userlens
description: Identify a user, associate them with their company, and track a
  custom product event against the Userlens Events API.
api: openapi/userlens-events-openapi.yml
operations: [submitEvent]
generated: '2026-07-21'
method: generated
source: https://userlens.gitbook.io/userlens-analytics/getting-started/http-api
---

# Instrument a user lifecycle in Userlens

All three steps use the same operation — `submitEvent` (`POST https://events.userlens.io/event`) — dispatched by the `type` field.

## Auth

Send `Authorization: Basic <base64(write_code + ":")>` — the Write Code is the
username, the password is empty, and the trailing colon is required. Get the
Write Code from https://app.userlens.io/settings/integrations/userlens-sdk.
Never log it.

## Steps

1. **Identify the user** (on signup, login, or profile update). Call
   `submitEvent` with `{"type": "identify", "userId": "<id>", "source":
   "userlens-restapi", "traits": {"email": ..., "name": ..., "plan": ...}}`.
   `traits` is required for identify.
2. **Group the user into their company** (essential for B2B account analytics).
   Call `submitEvent` with `{"type": "group", "groupId": "<company-id>",
   "userId": "<id>", "source": "userlens-restapi", "traits": {"name": ...,
   "industry": ..., "employees": ...}}`.
3. **Track meaningful actions**. Call `submitEvent` with `{"type": "track",
   "userId": "<id>", "source": "userlens-restapi", "event": "Subscription
   Upgraded", "properties": {...}}`.

## Rules

- Property names prefixed `$ul_` are reserved for Userlens system/browser
  context — do not set them yourself.
- Errors are plain HTTP statuses: 400 invalid body, 401 bad/missing Write Code,
  500 server error (see errors/userlens-problem-types.yml).
- There is NO idempotency key: a retry after a timeout may duplicate an event.
  Retry only on 5xx, with backoff, and accept possible duplicates.
