---
name: Upload and process an invoice
description: Upload an invoice document to Ottimate, poll extraction, review coded line items and approvers, then mark it exported.
api: openapi/qubiqle-openapi-original.json
operations:
- post-oauth-token
- post-invoices-upload
- get-invoices-uploads
- get-invoices-root
- get-invoices-id
- patch-invoices-id
- get-invoices-id-approvers
- post-invoices-mark-exported
---

# Upload and process an invoice

Use this skill to push an invoice document through Ottimate's capture → code → export lifecycle. Develop against Sandbox first.

## Steps
1. `post-oauth-token` — obtain a bearer token (see the vendor-sync skill for the exact call). Send `X-Api-Key` + `Authorization: Bearer` on all calls below.
2. `post-invoices-upload` — POST `/invoices/upload` with a file upload or a URL to download. Max file size 25MB (larger returns `413`). Send an `Idempotency-Key` so a retry does not create a duplicate invoice.
3. `get-invoices-uploads` — GET `/invoices/uploads` to confirm the upload was accepted and extraction (InstantCapture) started.
4. `get-invoices-root` — GET `/invoices` (filter by `status` / `payment_status`, page/limit) to find the created invoice, then `get-invoices-id` — GET `/invoices/{id}` for header, line items, GL splits, images, and history.
5. `patch-invoices-id` — PATCH `/invoices/{id}` to correct header fields, line items, and `dimensions` (GL coding) before export.
6. `get-invoices-id-approvers` — GET `/invoices/{id}/approvers` to see the approval routing.
7. Once approved and coded, `post-invoices-mark-exported` — POST `/invoices/mark-exported` with the invoice ids to flag them exported to your ERP. Use `post-invoices-mark-unexported` to reverse.

## Rules
- File uploads are non-JSON: an `Idempotency-Key` retry always returns `422` — use a NEW key per upload attempt.
- Long/bulk operations return a batch id — poll `get-batch-id-progress` / `get-batch-id-results`; requests time out at 29s.
- Only 2xx idempotent responses are cached (24h). Errors: `{code, message, request_id, timestamp}`.
