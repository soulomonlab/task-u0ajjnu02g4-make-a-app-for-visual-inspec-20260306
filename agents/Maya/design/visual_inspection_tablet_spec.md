# Visual Inspection — Tablet (Landscape) Design Spec

Summary
- Deliverable: Tablet-first camera + annotation flows (landscape primary) for Visual Inspection feature.
- File: output/design/visual_inspection_tablet_spec.md
- Purpose: Unblock Kevin for Storybook components and pixel polish.

Key decisions (high-level)
- Tablet-first, landscape primary: target users will hold tablets in landscape during inspections → maximizes horizontal workspace for toolbar + thumbnail strip.
- Minimal annotation toolset: pen, highlighter, undo, color picker, opacity slider, text label. Keep tools lightweight for speed.
- Live camera preview occupies left 60% of width; annotation canvas overlays preview when in annotate mode; right 40% houses thumbnail strip + metadata panel.
- Persistent capture CTA (bottom-right) to match ergonomic reach.

Screens (priority order)
1. Camera Capture (landscape)
2. Annotation Mode (full-screen overlay from Camera)
3. Thumbnail & Offline Queue Screen (right pane + modal)
4. Upload Status / Retry UI

User flow
- Open app → Camera Capture (preview, top toolbar, persistent capture) → Capture → Show annotation overlay with captured image + tools → Save/Done → Thumbnail added to strip + queued upload → Offline queue modal shows pending items.

Components & specs
- AppShell
  - Layout: 12-column grid (tablet), gutters 16px, safe margins 24px
- CameraCapture
  - Elements: live preview area (video), fallback input[type=file capture], capture button (primary), flash toggle, front/back camera toggle.
  - Interactions: short haptics on capture (platform), immediate thumbnail creation on capture.
- ImageAnnotator
  - Tools: Pen (1-6px), Highlighter (10-20px with blending), Text label (drag-to-place), Undo/Redo stack (depth 20), Zoom & pan gestures (pinch + drag)
  - Layers: image base layer → annotation vector layer(s) → temporary selection layer
  - Export: flattened PNG (export at device pixel ratio x2) + separate JSON of annotation vectors (for possible re-editing)
- ThumbnailStrip
  - Horizontal scroll, drag-to-reorder, long-press to open context menu (delete, annotate, retry)
- OfflineQueue UI
  - Badge with pending count, modal lists items with status, retry all, cancel

Accessibility
- Buttons minimum 44x44pt target
- Color contrast AA for critical text and controls
- Provide keyboard accessibility for annotation toolbar controls (for desktop/tablet with keyboard)

Assets to export (Figma -> deliverable)
- Screens (tablet landscape): CameraCapture, Annotation, ThumbnailStrip, OfflineQueue, UploadStatus as PNG @2x
- Icons: capture.svg, flip_camera.svg, flash.svg, pen.svg, highlighter.svg, text.svg, undo.svg, redo.svg, upload.svg (SVG, optimized)
- Component tokens: color palette, typography scale, spacing scale (JSON or tokens file)
- Annotation tool presets: brush sizes (px), default colors

Export settings & naming
- PNGs: 2x export, suffix @2x (e.g., camera_capture@2x.png)
- SVGs: Clean, no raster effects; separate icon set
- Tokens: tokens.json

Developer notes
- Provide both flattened PNG and annotation vector JSON for each captured image so frontend can show thumbnail quickly and allow re-editing.
- Annotation export should be lightweight (vector commands + stroke color/width) to store in IndexedDB.
- Canvas resolution: use device pixel ratio × capture size; cap at 4096px on longest side to avoid memory issues.

Open items / requests (blocking for implementation)
- I need Figma file link or invite (maya@team) to export assets. If you can't invite, Kevin — please export the PNGs/SVGs per the list and attach them to the PR branch: feature/visual-inspection-ui/assets.
- Confirm capture fallback policy: should we allow upload of original full-resolution camera image or only flattened annotated export? (I recommend storing both: original full-res + annotated export JSON)

What I’ll deliver after Figma access
- Exported assets placed under output/design/assets/ (PNGs + SVGs + tokens.json)
- A small clickable prototype (Figma -> prototype share link) showing capture → annotate → save flow

References
- Device target: tablet landscape (iPad Air/Pro, Android 10-inch). Grid and spacing chosen for 10" - 12.9".

Contact
- If Marcus provides any API constraints for upload (signed-url batch shape, max_batch_size), I’ll update the UploadStatus UI to reflect retry limits and concurrency indicators.

---
Created by: Maya (Designer)
