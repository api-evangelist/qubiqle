---
name: Sync vendors and dimensions to Ottimate
description: Keep an ERP's chart of accounts in sync with Ottimate by listing, creating, and bulk-upserting accounting vendors and dimensions.
api: openapi/qubiqle-openapi-original.json
operations:
- post-oauth-token
- get-vendors-root
- post-vendors-root
- post-vendors-bulk
- get-dimensions-root
- post-dimensions-root
- post-dimensions-bulk
---

# Sync vendors and dimensions to Ottimate

Use this skill to mirror an ERP's vendors and accounting dimensions into Ottimate so invoices can be auto-coded and matched. Always develop against Sandbox first (`https://sandbox-api.ottimate.com/v1`).

## Auth (once per token lifetime)
1. `post-oauth-token` — POST `/oauth/token` with headers `X-Api-Key: <key>` and body `grant_type=client_credentials`, `client_id`, `client_secret`, `scope=accounts.can_access_dashboard`. Cache the returned `access_token` until `expires_in` elapses or you get a `401`.
2. On every subsequent call send BOTH `X-Api-Key: <key>` and `Authorization: Bearer <access_token>`.

## Dimensions
3. `get-dimensions-root` — GET `/dimensions` (page/limit) to see what already exists.
4. For a few new dimensions, `post-dimensions-root` — POST `/dimensions`. For a full ERP sync, `post-dimensions-bulk` — POST `/dimensions/bulk` (create-or-update). Dimensions must exist before POs or invoices reference them.

## Vendors
5. `get-vendors-root` — GET `/vendors` (filter by `ottimate_company_id`) to reconcile.
6. `post-vendors-root` for one vendor, or `post-vendors-bulk` — POST `/vendors/bulk` for a batch.

## Rules
- Send an `Idempotency-Key` (UUIDv4/ULID) on every POST — retention is 24h; a same-key retry with an identical JSON body replays the cached 2xx (`X-Idempotent-Replayed: true`), a different body returns `422`.
- Bulk endpoints may run async and return a batch id — poll `get-batch-id-progress` / `get-batch-id-results`.
- Respect limits: 1,000 req/s steady, 2,000 burst, 10,000/day per key.
- Errors come back as `{code, message, request_id, timestamp}` — log `request_id` for support.
