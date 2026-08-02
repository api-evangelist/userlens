---
name: Forward Userlens SDK events through a backend proxy
description: Route events collected by the Userlens JavaScript SDK through your
  own backend to raw.userlens.io, keeping the Write Code server-side.
api: openapi/userlens-events-openapi.yml
operations: [forwardRawEvents]
generated: '2026-07-21'
method: generated
source: https://userlens.gitbook.io/userlens-analytics/guides/api-reference
---

# Forward Userlens SDK events through a backend proxy

Use the proxy setup when you do not want the Write Code exposed client-side:
the browser SDK posts to your endpoint, and your server forwards the batch with
`forwardRawEvents` (`POST https://raw.userlens.io/raw/event`).

## Auth

Server-side only: `Authorization: Basic <base64(write_code + ":")>` (Write Code
as username, empty password). Keep the Write Code in an environment variable —
never ship it to the browser in proxy mode.

## Steps

1. Expose a backend route that accepts the SDK's event batch payload
   (`{"events": [...]}`).
2. Forward the body unchanged to `forwardRawEvents` — the batch mixes three
   shapes: auto-captured clicks (`is_raw: true`, `event` = XPath, plus a DOM
   `snapshot` tree), custom events (`is_raw: false`, `event` = name from
   `pushEvent()`), and page views (`event: "$ul_pageview"`).
3. Pass through the response status to decide client behavior: 200 success,
   400 invalid body, 401 bad Write Code, 500 server error.

## Rules

- Do not rewrite `$ul_`-prefixed properties — they carry browser/OS/page
  context Userlens expects.
- Preserve the `userId` on each event so activity lands on the right account.
- No idempotency mechanism exists; if the forward times out, a retry may
  duplicate the batch.
