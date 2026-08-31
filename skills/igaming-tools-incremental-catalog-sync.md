---
generated: '2026-08-30'
method: generated
name: Keep a local mirror of the iGaming catalog in sync
description: Do a first full pull of providers and slots, then keep it current with delta sync and tombstone reconcile, without burning the monthly quota.
api: openapi/igaming-tools-openapi.json
operations: [providers_list, providers_ids_list, slots_list, slots_ids_list, providers_retrieve, slots_retrieve, stats_retrieve]
source: >-
  Grounded in openapi/igaming-tools-openapi.json, captured 2026-08-30 from
  https://i-gaming.tools/docs/openapi.json. Every operationId verified verbatim in that
  spec. Sync protocol per https://i-gaming.tools/docs/sync.md, quota accounting per
  rate-limits/igaming-tools-rate-limits.yml, auth per
  authentication/igaming-tools-authentication.yml.
---

# Keep a local mirror of the iGaming catalog in sync

The iGaming Tools API is built for mirroring: it publishes a delta filter, a sync cursor header, and a tombstone reconcile endpoint. Use all three or your copy will silently drift.

## Auth
- Header: `Authorization: Token <your-hex-key>` — the literal word `Token`, **not** `Bearer`. See `authentication/igaming-tools-authentication.yml`.
- Base URL: `https://i-gaming.tools/api/v1`.
- Issue and revoke keys from the browser dashboard at `/account/`. The key is shown once, at creation.

## Steps

1. **Initial full sync** — `providers_list` (`GET /api/v1/providers/`) and `slots_list` (`GET /api/v1/slots/`). Follow the opaque `next` URL from each response envelope until it is null. Do not build cursors yourself and do not expect an `offset` parameter; there is none.
2. **Store the sync cursor.** Every 200 (and every 304) on these endpoints returns an `X-Sync-Timestamp` header. Persist it. It is the **start-of-request** timestamp, not the end, so records changed while you were paginating are not lost on the next run.
3. **Delta sync on the next run** — call the same operations with `?updated_since=<stored X-Sync-Timestamp>`. Only changed items come back. Update your stored timestamp from the new header.
4. **Reconcile deletions** — `providers_ids_list` (`GET /api/v1/providers/ids/`) and `slots_ids_list` (`GET /api/v1/slots/ids/`) return the complete set of currently-public slugs. Diff against your local set; anything of yours that is missing has been unpublished. Deltas will never tell you this — an unpublished record simply stops appearing.
5. **Fill detail on demand** — `providers_retrieve` (`GET /api/v1/providers/{slug}/`) and `slots_retrieve` (`GET /api/v1/slots/{slug}/`) for the records the delta flagged. Send `If-None-Match` with the stored ETag; a `304` means unchanged.

## Rules an agent must follow

- **Conditional requests are free against your budget.** A `304 Not Modified` does **not** decrement the monthly quota, but it *does* count toward the per-minute rate limit. Storing ETags and sending `If-None-Match` is the single biggest cost saving available on the free tier (`rate-limits/igaming-tools-rate-limits.yml`).
- **Poll the aggregate endpoints freely.** `stats_retrieve` (`GET /api/v1/stats/`) never decrements the quota. Use it as a cheap "has anything changed at all" heartbeat before spending quota on a delta run.
- **Two different exhaustion signals, two different responses.** `429` is the per-minute throttle: sleep for `Retry-After` seconds and retry, it will succeed. `402` is the monthly quota: retrying will **not** help until the reset. Read `X-Quota-Resets-At` and stop. Confusing these is the most likely failure mode against this API (`errors/igaming-tools-problem-types.yml`).
- **Watch your budget mid-run.** Every 2xx carries `X-Quota-Free-Remaining`, `X-Quota-Paid-Balance` and `X-Quota-Resets-At`. Check them while paginating a large sync rather than discovering exhaustion at page 90.
- **Page size caps at 100** on every plan, including paid. Size your pagination loop accordingly.
- **Nothing here writes.** All 29 operations are `GET`. There is no idempotency key because there is nothing to make idempotent, and there is no rollback because there is no action to take back.
- **Timestamps are ISO-8601 UTC** (`2026-07-14T12:34:56Z`). `updated_since` accepts any valid ISO-8601 datetime with a timezone.
- **`null` means "not yet available", not "not applicable"** (`https://i-gaming.tools/docs/conventions/`). Do not fold nulls into your model as a negative fact.
