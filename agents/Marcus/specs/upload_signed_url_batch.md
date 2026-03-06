# Signed-URL Batch Upload API Spec (Backend)

Purpose
- Provide a single call to obtain presigned upload URLs for a batch of files to support the visual-inspection tablet asset upload flow.

Endpoint
- POST /api/v1/uploads/signed-urls
- Auth: Bearer token (OAuth2/JWT)
- Content-Type: application/json

Constraints / Key decisions
- max_batch_size: 20 files per request (keeps request size reasonable; frontend should chunk larger uploads into multiple batches)
- max_file_size: 25 MB per file (PNG @2x for 4096px canvas fits under this threshold empirically)
- presigned_url_expiry: 300 seconds (5 minutes)
- recommended frontend concurrency: 6 parallel uploads per client (to avoid S3 throttling and UX/network spikes)
- retry policy (frontend): up to 3 retries per file with exponential backoff (200ms -> 800ms -> 1600ms)
- rate limits: 20 requests/minute per user for the signed-urls endpoint; upload PUTs are rate-limited by underlying storage
- idempotency: client may pass optional idempotency_key per batch; server returns upload_batch_id
- storage: object store (S3-compatible). Signed URLs are direct PUTs to storage.
- security: validate content-type, content-length on presigned URL generation. Signed URLs are single-use until successful PUT completes (server marks file as uploaded by polling or post-upload webhook).

Request example
POST /api/v1/uploads/signed-urls
{
  "idempotency_key": "optional-client-key-123",
  "files": [
    {"file_name": "img_001.png", "content_type": "image/png", "size": 3456789},
    {"file_name": "annotation_001.json", "content_type": "application/json", "size": 1234}
  ]
}

Response example (200)
{
  "upload_batch_id": "batch_abc123",
  "max_batch_size": 20,
  "presigned_url_expiry_seconds": 300,
  "files": [
    {
      "client_file_id": "img_001.png",
      "file_id": "file_01a2b3",
      "upload_url": "https://storage.example.com/put/...",
      "method": "PUT",
      "headers": {"Content-Type": "image/png"},
      "expires_at": "2026-03-06T12:34:56Z"
    },
    {
      "client_file_id": "annotation_001.json",
      "file_id": "file_01a2b4",
      "upload_url": "https://storage.example.com/put/...",
      "method": "PUT",
      "headers": {"Content-Type": "application/json"},
      "expires_at": "2026-03-06T12:34:56Z"
    }
  ]
}

Error responses
- 400 Bad Request: malformed JSON, > max_batch_size, any file > max_file_size
- 401 Unauthorized: missing/invalid token
- 413 Payload Too Large: single file exceeds allowed size (alternative: 416)
- 429 Too Many Requests: rate limit exceeded (response includes Retry-After header)

Post-upload lifecycle
- Frontend performs PUT to each provided upload_url with specified headers.
- Server verifies upload via storage events or HEAD checks and marks file status (uploaded/failed).
- If upload fails after retries, frontend should call POST /api/v1/uploads/complete with upload_batch_id and per-file statuses (server accepts retries within presigned_url_expiry window).

Observability
- Add OpenTelemetry spans around the signed-urls generation and up/downstream calls.
- Return x-rate-limit headers: X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset.

Notes / Trade-offs
- Chosen max_batch_size=20 balances single request overhead vs payload size for tablet asset bundles.
- presigned_url_expiry=5m is conservative to avoid long-lived URLs; if network is poor we can increase to 10m but prefer frontend retries.
- We do not support multipart resumable uploads in v1; if clients need >25MB support, add multipart endpoint in v2.

Acceptance criteria
- Frontend can request up to 20 signed URLs and successfully PUT assets; backend returns per-file file_id and tracks uploaded state.

File created by: Marcus (Backend)
