# Visual Inspection — Frontend Implementation Notes

Branch: feature/visual-inspection-ui

Overview:
- Implement initial React component skeletons for tablet-first PWA flows: AppShell, CameraCapture, ImageAnnotator, OfflineQueue, ThumbnailStrip.
- Storybook stories TBD (will add after design QA). Create branch and PR when components reach feature completeness.

Key technical decisions (frontend):
- State: Zustand for lightweight local state; persistent offline queue stored in IndexedDB (use `idb` wrapper).
- Camera: implement camera abstraction (web fallback uses <input capture="environment"> + MediaDevices.getUserMedia when available). Keep abstraction so native Capacitor wrapper can replace implementation later.
- Offline: store image blobs + metadata in IndexedDB; batch signed-URL requests up to 20; cap concurrent uploads to 3; exponential backoff retries (base 500ms, factor 2, max 5 retries).
- Accessibility: buttons have aria-labels; image annotator will support keyboard shortcuts later.

Next steps for #ai-frontend (Kevin):
- Complete Storybook stories and visual polish for tablet landscape camera/annotation flows.
- Integrate with backend signed-URL endpoints (ask Marcus for API shape: batch size, auth headers).
- Add e2e QA scenarios and hand off to #ai-qa when stable.

Files created:
- output/code/frontend/AppShell.tsx
- output/code/frontend/CameraCapture.tsx
- output/code/frontend/ImageAnnotator.tsx
- output/code/frontend/OfflineQueue.tsx
- output/code/frontend/ThumbnailStrip.tsx
- + tests: *.test.tsx
