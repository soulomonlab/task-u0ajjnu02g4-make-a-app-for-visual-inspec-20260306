# Visual Inspection — Signed URL Batch API

Purpose
- Provide frontend with short-lived signed upload URLs for direct-to-storage uploads (S3/compatible).

Branch
- feature/visual-inspection-signed-urls-#159

API Summary
- POST /api/v1/uploads/signed-urls/batch
  - Auth: Bearer JWT (Authorization header)
  - Max batch size: 20 files
  - Signed URL expiry: 10 minutes (600 seconds)
  - Allowed upload method: PUT (recommended) or POST (if client requires)

Request Body (application/json)
{
  "project_id": "string",              // optional if scoped by token
  "files": [
    {
      "filename": "photo1.jpg",
      "content_type": "image/jpeg",
      "size": 123456,
      "checksum": "sha256:...",        // optional but recommended (for later verification)
      "metadata": {"camera":"rear","orientation":"landscape"}
    }
  ]
}

Response (200 OK)
{
  "batch_id": "string",
  "expires_in": 600,                    // seconds until the signed URLs expire
  "files": [
    {
      "file_id": "uuid",               // internal id to reference the uploaded blob
      "filename": "photo1.jpg",
      "upload_url": "https://storage.s3...",
      "method": "PUT",
      "headers": {
        "Content-Type": "image/jpeg"
      }
    }
  ]
}

Notes / Decisions
- Max batch size = 20 to match frontend batching decision (Kevin). This is enforced server-side (413 if exceeded).
- Signed URLs expire in 10 minutes: short enough to reduce risk, long enough for retries and mobile uploads.
- We return per-file file_id so the frontend can call a separate "complete" endpoint to confirm each upload and attach annotation metadata.
- Rate limiting: default throttles per API key/user: 30 batch requests/minute (P50 traffic assumption). Per-file upload limits are enforced at storage layer; recommend client-side 3 concurrent uploads.
- Retry/backoff: frontend should use exponential backoff; server allows idempotent retries because file_id is returned and the complete endpoint is idempotent.

Errors
- 400 Bad Request: malformed body, missing fields
- 401 Unauthorized: invalid/missing token
- 413 Payload Too Large: files.length > 20
- 429 Too Many Requests: rate limit exceeded
- 500 Internal Error

Post-upload verification
- POST /api/v1/uploads/complete
  - Body: {"batch_id":"...","uploads":[{"file_id":"...","checksum":"..."}]} 
  - Server validates checksum (if provided) and marks blob as ready; triggers any post-processing (thumb gen, virus scan, indexing).

Security
- Require Authorization: Bearer <JWT> with permission scope uploads:create or project-specific write access.
- CORS: storage buckets must allow browser PUT from our frontend origins.
- Content-Type enforcement: signed URL will require a specific Content-Type header; server validates on complete.

Observability
- Add OpenTelemetry span for batch request + individual post-upload verification.
- Log batch_id, user_id, file_count, sizes (aggregated) for monitoring.

Examples
- See output/docs/openapi_visual_inspection.yaml for machine-readable spec.
