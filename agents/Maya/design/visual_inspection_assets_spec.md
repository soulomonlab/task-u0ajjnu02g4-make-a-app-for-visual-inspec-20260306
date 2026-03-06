# Visual Inspection — Asset & Export Spec

Summary
- Purpose: Provide pixel-ready assets and tokens for Kevin to build the visual-inspection prototype.
- Preferred flow: Invite maya@team to Figma (fastest for iterations). Fallback: export assets into repo branch feature/visual-inspection-ui/assets using naming below.

User flow (high-level)
1. User opens tablet landscape (primary breakpoint). Canvas area 60% L / Side panel 40% R.
2. User annotates image with minimal toolset (point, rectangle, freehand, color pick, undo/redo).
3. User exports flattened PNG @2x and annotation JSON; icons and tokens provided separately.

Component & export list
- Artboards: Visual Inspection / Canvas (master), Visual Inspection / Side Panel
- Exports required:
  - Flattened screen PNG @2x (for prototype): canvas/<screen>@2x.png
  - Annotation data JSON: annotations/<screen>-annotations.json
  - SVG icons: icons/icon-<name>.svg (individual files, optimized)
  - Design tokens: tokens.json (colors, spacing, typography, radii)

Naming conventions (strict)
- tokens.json
- icons/icon-<semantic-name>.svg (e.g., icon-zoom-in.svg, icon-annotate-rect.svg)
- canvas/visual-inspection-main@2x.png
- canvas/visual-inspection-sidepanel@2x.png
- annotations/visual-inspection-main-annotations.json

Export settings
- PNG: 2x scale, flattened, sRGB, export at artboard size (ensure width <= 4096px per canvas cap). Prefer 2048 px width for tablet landscape if artboard is 1024pt wide.
- SVG: "Outline strokes" (or ensure consistent stroke widths), minified.
- tokens.json: include hex colors, font-family names, font-sizes (px), spacing scale (8pt baseline).

Figma pages & layers to export
- Page: "Visual Inspection"
  - Frame: "Canvas - Main" → export canvas/visual-inspection-main@2x.png
  - Frame: "Side Panel" → export canvas/visual-inspection-sidepanel@2x.png
  - Frame: "Assets" → export icons/ and tokens.json

Decisions / rationale
- Prefer Figma invite (A): enables quick iteration, small tweaks, and exporting missing states—reduces back-and-forth.
- Fallback export (B) accepted but must follow naming conventions exactly to avoid integration friction.
- tokens.json + SVGs separate to let frontend generate theme and reuse icons easily.

What I need from Kevin
- Option A: Invite maya@team to the Figma file (Editor or Commenter+Export rights). I'll export final assets and place them under output/design/assets/ then hand off for prototype.
- Option B: If you must export, place files in branch feature/visual-inspection-ui/assets following naming conventions above and tell me the repo path.

Acceptance criteria
- I can access Figma OR repo branch contains all files listed above with correct names.
- PNGs are 2x and width <= 4096px.
- tokens.json present and parsable.

Files I will create on receipt
- output/design/assets/{png,svg,tokens.json,annotations.json}
- output/design/visual_inspection_prototype.fig-export-log.md (export notes)

Questions
- Confirm preferred artboard width (defaulting to 1024pt → 2048px @2x). If different, tell me target px width.

