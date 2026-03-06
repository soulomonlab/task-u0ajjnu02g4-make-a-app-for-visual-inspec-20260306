Title: Visual Inspection — Frontend asset + Upload API request

Situation
- Maya delivered tablet-first design spec (tablet landscape, live preview left 60% / right 40% thumbnails+meta).

Decisions made (frontend)
- Implement tablet-first landscape layout: preview 60% / side panel 40%.
- Minimal annotation toolset. Canvas capped at 4096px (long edge) to limit raster exports.
- Export requirements: flattened PNG @2x and annotation vector JSON per canvas. Also SVG icons + tokens.json.
- Assets will be placed in branch: feature/visual-inspection-ui/assets (ready for export/import).

What I need from Maya
- Either: invite maya@team to the Figma file OR provide Figma file link + confirm exportable pages/components.
- Confirm the final assets list (PNG@2x, SVG icons, tokens.json) and any naming conventions.

What I need from Marcus (blocking for UploadStatus UI)
- Signed-URL batch upload API shape to drive UploadStatus / Retry UI. Specifically provide:
  - max_batch_size (per request)
  - response per file: {file_id, upload_url, expires_at}
  - required auth headers for upload requests (if any)
  - per-second rate limits and recommended client concurrency
  - retry policy: which HTTP status codes are retryable, recommended retry cap per file
  - signed URL expiry (seconds)
  - example request/response JSON and error responses (401, 413, 429, 5xx)

Acceptance criteria for frontend changes
- UploadStatus UI shows per-file progress, concurrent upload count capped to backend recommendation, per-file retry up to backend cap, and clear error states for non-retryable errors.

Files / artifacts
- This spec: output/specs/visual_inspection_frontend_request.md

Next steps (frontend)
- I will invite maya@team to Figma once she confirms file link or I will export assets into feature/visual-inspection-ui/assets if Maya prefers export.
- Blocker: waiting on Marcus' API shape to build UploadStatus / Retry UI correctly.