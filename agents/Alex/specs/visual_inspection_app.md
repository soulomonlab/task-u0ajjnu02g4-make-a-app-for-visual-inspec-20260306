# Feature: Visual Inspection App for Aircraft Manufacturing

**Goal:** Provide inspectors with a fast, reliable app to capture, annotate, and log defects during aircraft manufacturing to reduce rework and improve traceability.

**North Star Impact:** Reduce defect-to-resolution time by 40% and increase inspection coverage to 90% of critical assemblies.

**Users:**
- Primary: Line inspectors (tablet/mobile) capturing defects on the shop floor.
- Secondary: Quality engineers reviewing reports and analytics.

**RICE Score:** Reach=250 inspectors/Q × Impact=2 × Confidence=70% / Effort=8w = 43.75 (~44)

**Kano Category:** Must-have (core quality workflow)

**Acceptance Criteria:**
- [ ] Inspector can open app on tablet/phone and start a new inspection.
- [ ] Inspector can capture high-resolution photos, add bounding-box or freehand annotations, select defect type from taxonomy, and save an inspection record.
- [ ] App allows offline capture and queues uploads when connectivity is restored.
- [ ] Images are stored in scalable object storage; metadata (inspection record, location, part ID, inspector, timestamp, annotations) stored in relational DB.
- [ ] Backend API supports create/read/list inspection records; supports pagination and filtering by part, line, status, date.
- [ ] Integration points defined: 1) MES for part/work-order lookup (read), 2) QMS for defect lifecycle status updates (write). If integration unavailable, allow CSV export/import.
- [ ] Admin UI (web) for QA engineers to review images, add comments, change defect status, and generate basic reports (daily defect count by part/line).
- [ ] Performance: backend handles 10k images/day; API 95th percentile < 300ms under expected load.
- [ ] Security: role-based access (inspector, QA, admin); images access-controlled; data encrypted at rest and in transit.

**Out of Scope:**
- Automated defect detection (ML). This will be a separate epic.
- Deep analytics dashboards beyond basic reports.

**Success Metrics:**
- Adoption: % of inspections done via app vs paper (target 60% in Q1 roll-out)
- Speed: median time to log defect < 90s
- Quality: reduction in repeat defects for addressed issues by 25% in 6 months

**Implementation Notes / Key Decisions (PO):**
- Form factor: Progressive Web App (PWA) first for rapid deployment across tablets/phones; revisit native if camera access limits performance.
- Storage: Images → S3 (or equivalent); Metadata → Postgres.
- API: REST v1 for MVP. Use signed URLs for direct image upload to object store.
- Offline: local SQLite or IndexedDB cache with background sync.
- Integrations: MES read is high priority for reducing manual input; plan for a connector module.

**GitHub Issue:** Created after PRD saved.
