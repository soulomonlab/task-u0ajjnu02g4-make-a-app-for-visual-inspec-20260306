# Visual Inspection App — Design Spec (Tablet-first PWA + Admin Web UI)

## Overview
Deliverables in this file:
- Clickable tablet-first wireframes (prototypes linked separately)
- Component list and interaction specs
- Design tokens, color/typography, spacing
- Assets inventory and export guidance
- Key design decisions and trade-offs (camera capability fallback)

Reference: PRD → output/specs/visual_inspection_app.md
Target: Tablet-first PWA (landscape preferred) for field inspectors; Admin web UI for desktop review and reporting.
Constraints: Images stored in S3 (signed URLs), metadata in Postgres, offline support (IndexedDB/SQLite), REST API, performance target 10k images/day & API 95p <300ms.

---

## 1) User flows (high-level)

A. Field Inspector — Single Inspection
1. Launch PWA (offline-first cache) -> authenticate (SSO / token)
2. Select site/asset from list or QR scan
3. Capture images (camera UI) -> review thumbnails
4. Annotate image(s) (rectangle/arrow/text) -> add metadata (type, severity, notes)
5. Save locally (IndexedDB) if offline; show sync queue
6. Automatic/upload sync when online (signed URL retrieved, file PUT to S3)
7. Confirmation toast + item appears in Admin queue

B. Admin Reviewer
1. Login -> Dashboard (filters, counts, recent submissions)
2. Open submission -> view images, annotations, metadata
3. Add review status (accept, reject, request follow-up), bulk actions, export CSV

---

## 2) Wireframes (ASCII)

Tablet — Main Inspection Screen (landscape)

+--------------------------------------------------------------------+
| TopBar: Back | Project/Asset selector (drop) | Sync status | Avatar |
+--------------------------------------------------------------------+
| Left pane (30%): Asset list / Filters | Right pane (70%): Camera / Preview |
| [Asset list]                            | [Camera view live preview]       |
| - Site A                                 | ------------------------------  |
| - Site B                                 | [Capture Button] [Gallery]      |
|                                          | [Last 4 thumbnails horizontally]|
+--------------------------------------------------------------------+
| Bottom action bar: Capture | Annotate | Metadata | Save Offline | Sync |
+--------------------------------------------------------------------+

Capture Modal
+-----------------------------------+
| Live camera feed                   |
| [Capture] [Flash] [Switch cam]     |
| Thumbnail strip | Retake | Use photo |
+-----------------------------------+

Annotation Screen
+------------------------------------------------+
| Image (full screen)                            |
| Annotation tools (left vertical): Arrow, Rect,  |
| Text, Color, Undo                              |
| Right panel: Metadata form, Tags, Severity     |
| Save | Cancel                                   |
+------------------------------------------------+

Admin — Submission List (Desktop)
+---------------------------------------------------------------+
| TopBar: Search | Filters (date, asset, severity) | Export | User|
+---------------------------------------------------------------+
| Table: Submission ID | Asset | Date | Thumbs | Status | Actions  |
| Row expand -> detail panel with full images + annotations       |
+---------------------------------------------------------------+

---

## 3) Component list (priority)

Core components (deliver to frontend):
- AppShell (TopBar, left navigation, responsive grid)
- AssetList (search + filter + QR scan entry)
- CameraCapture (live view, capture, manual upload fallback)
- ThumbnailStrip (select/reorder)
- ImageAnnotator (shapes, arrow, text, color picker, undo/redo)
- MetadataForm (fields per PRD: type, severity, inspector, notes)
- OfflineQueue (card list with retry/sync status)
- SyncIndicator (global + per-item status)
- AdminTable (sortable, paginated), DetailDrawer
- Toasts & Modals (confirmations, errors)

Each component must be keyboard accessible and have ARIA labels.

---

## 4) Interaction specs (key flows)

Capture flow:
- Tap Capture -> show flash/capture animation -> push image to preview + open review modal
- In review modal, options: Retake (discard), Use photo (adds to thumbnails)
- From thumbnails user can long-press to reorder or swipe to delete

Annotate flow:
- Enter annotator -> default tool: rectangle
- Draw with touch; gesture: two-finger pinch to zoom; single-finger drag to move canvas
- Undo/redo (persisted per image)
- Save persists annotated PNG (flattened) for upload; also store vector overlay in metadata for backend replay

Offline behavior:
- Save stores image blob + metadata in IndexedDB (or SQLite if available)
- UI shows OfflineQueue with item status: Pending, Uploading, Failed, Synced
- On network restoration, client requests signed URL for each item and uploads directly to S3; backend receives metadata separate from file upload and verifies checksum

Conflict resolution:
- If server rejects metadata (validation), item moves to Failed with error and inspector can edit and resubmit

Performance notes for frontend:
- Batch signed URL requests (up to 20) to reduce round trips
- Uploads should be retried with exponential backoff; cap concurrency (3 simultaneous uploads)

---

## 5) Camera capability & native fallback (decision + trade-offs)

Observed constraint: Some tablets/browsers expose limited camera control (no high-res, slow capture, no background upload, or poor autofocus). Options:

1) Pure PWA approach (default)
- Pros: Single codebase, no app store friction, faster dev
- Cons: Potential degraded camera UX on some Android tablets/iOS Safari limitations (background uploads limited, inconsistent camera APIs)

2) Thin native wrapper (Capacitor / Cordova)
- Pros: Access to native camera APIs, background uploads, better control over file paths
- Cons: Extra build/deploy complexity, users must install or accept prompt; small native maintenance cost

3) Native micro-app (Kotlin/Swift) for high-volume sites
- Pros: Best performance & reliability for camera-heavy workflows
- Cons: Highest cost and time; fragmentation across platforms

Recommendation: Start with PWA first (meets PRD). Build camera abstraction layer in code that allows swapping implementation; if field testing shows poor capture quality on target devices, deliver a P0 decision to implement Capacitor wrapper for installing as a native shell. Documented trade-off: accelerated timeline vs camera reliability.

---

## 6) Design tokens (key)

Colors:
- Primary: #0057D9
- Primary-600 (hover): #0046b3
- Success: #16A34A
- Warning: #F59E0B
- Danger: #DC2626
- Surface / Card: #FFFFFF
- Muted text: #6B7280

Typography:
- Heading: Inter 600
- Body: Inter 400
- Sizes: H1 28px, H2 22px, Body 16px, Small 13px

Spacing: base = 8px (2,4,8,16,24,32)

Buttons:
- Primary: filled primary + white text
- Secondary: outline primary

Icons: outline system icons (camera, sync, cloud, annotate, trash, check)

Export guidance: SVG for icons; PNG @2x and @3x for raster assets.

---

## 7) Assets deliverables
- Icon set (camera, sync, offline, annotate) — SVG + PNG @2x/@3x
- UI kit: buttons, inputs, modals as Figma components
- Sample annotated images (4) for dev/testing
- Color/typography tokens JSON for theming

---

## 8) Acceptance criteria (done when)
- Clickable tablet-first wireframes (Figma prototype link included in PR) — provides navigation between flows
- Component spec + props documented (story-like examples)
- Assets & design tokens exported into design package
- Camera fallback decision and implementation plan documented (this file)

---

## 9) Next steps for frontend
- Implement UI in React (PWA) using the tokens; prioritize CameraCapture, ImageAnnotator, OfflineQueue
- Build Storybook stories for core components

---

## Decision log
- PWA-first, tablet-first (aligned with PRD)
- Start with PWA; implement camera abstraction to enable Capacitor native wrapper later if field tests fail
- Batch signed URL requests and limit concurrent uploads to meet performance targets



