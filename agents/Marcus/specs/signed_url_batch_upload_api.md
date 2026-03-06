# Signed URL Batch Upload API (Task #168)

Summary
- Purpose: Provide a single batched API that returns per-file signed upload URLs + metadata to support concurrent uploads, retry logic, and UI progress.
- File: output/specs/signed_url_batch_upload_api.md

Key decisions (reversible)
- max_batch_size: 50 (trade-off: fewer requests vs payload size)
- signed_url_expiry: 15 minutes (matches short-lived access patterns)
- upload_concurrency_limit: 6 (client/UI should cap concurrent uploads)
- Rate-limit: 5 batch requests/sec per API key or user (soft limit)
- Retryable status codes for upload requests: 429, 502, 503, 504. Retry cap: 5 attempts with exponential backoff (base 500ms, factor 2)
- Idempotency: Require Idempotency-Key header for the batch creation endpoint

API
| Method | Path | Description | Auth |
|--------|------|-------------|------|
| POST   | /api/v1/uploads/signed-urls/batch | Create a batch of signed upload URLs | Bearer Token + Idempotency-Key |

Request (application/json)
{
  "files": [
    { "filename": "image.png", "content_type": "image/png", "size": 123456 },
    { "filename": "annot.json", "content_type": "application/json", "size": 2048 }
  ],
  "max_batch_size": 50  // optional, server-enforced max 50
}

Headers
- Authorization: Bearer <access_token>
- Idempotency-Key: <uuid-v4>
- Accept: application/json

Success Response (200)
{
  "batch_id": "b123e456-...",
  "upload_concurrency_limit": 6,
  "signed_url_expiry_seconds": 900,
  "files": [
    {
      "file_id": "f123e456-...",
      "filename": "image.png",
      "upload_url": "https://s3.amazonaws.com/....?signature=...",
      "method": "PUT",
      "headers": { "Content-Type": "image/png" },
      "expires_at": "2026-03-06T15:04:05Z"
    },
    {
      "file_id": "f223e456-...",
      "filename": "annot.json",
      "upload_url": "https://s3.amazonaws.com/....?signature=...",
      "method": "PUT",
      "headers": { "Content-Type": "application/json" },
      "expires_at": "2026-03-06T15:04:05Z"
    }
  ]
}

Partial failures
- If any file cannot get a signed URL, the response will include that file with an error field and a per-file status. The client should treat file-level errors as retryable only if the code indicated is retryable.

Error codes
- 400: bad request (invalid file metadata, > max_batch_size)
- 401: unauthorized
- 403: forbidden (quota exceeded)
- 429: rate limit exceeded
- 500: server error (non-retryable for batch creation; client should retry with backoff)

Client upload guidance (UI)
- Use provided upload_concurrency_limit (6) to cap parallel uploads.
- Use per-file upload_url + method + headers for the direct upload.
- For upload HTTP responses: consider retries for 429, 502, 503, 504 with exponential backoff (max 5 attempts).
- After successful upload, call server-side finalize endpoint (POST /api/v1/uploads/complete) with { batch_id, file_ids: [...] } to mark completion and trigger backend post-processing.

Next steps / Implementation plan
1. I will implement the batch endpoint, unit tests and OpenAPI spec on branch: feat/signed-url-batch-#168
2. ETA: spec done (this file). Implementation + tests + PR: 48h (P1)

Notes for frontend
- I recommend client-side progress per file + aggregate progress. Use file_id for mapping progress and retries.
- Naming convention for returned file_id: UUIDv4.
