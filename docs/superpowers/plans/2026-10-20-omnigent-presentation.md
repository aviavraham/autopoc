# Omnigent AutoPoC Presentation Plan

**Goal:** Deliver the approved stakeholder presentation of run
`autopoc-omnigent-8554` as editable Slidev source, a browser deck and a PDF.

**Scope approved in chat:** Twelve main slides, approximately ten minutes,
plus a technical evidence appendix. Use a red/charcoal visual style, diagrams,
source references and speaker notes. Cover outcome, purpose, scope, pipeline,
timeline, architecture, recovery, tests, deliverables, limitations, value and
next steps. This is a presentation of one run, not a new pipeline integration.

**Architecture:** A self-contained Slidev project under
`presentations/omnigent-2026-10-20/`. Evidence is pinned to GitHub commit
`f2d3c4d8183de4d67f5a2cb3135c3b653211f6e6` and original runner/build logs.
Use Slidev's native static build and Playwright PDF exporter.

**Tech stack:** Node 22, Slidev 53, default theme, Playwright/Chromium.

## Tasks

- [x] Archive reports, original test JSON, sanitized logs and selected cluster
  metadata. Index slide claims to their sources and distinguish original
  evidence from later observations. Record missing trace/media artifacts.
- [x] Author twelve stakeholder slides and a short evidence appendix with
  editable diagrams, timelines, source links and speaker notes.
- [x] Build the browser deck and export the PDF. Inspect every rendered slide,
  verify source links/local assets, check evidence and scan for credentials.
- [x] Package deliverables, document reproducible commands, update existing
  planning documents and commit only this presentation's changes.

## Verification record

- Slidev production build succeeded; PDF exported as 16 pages.
- Visually reviewed all pages and rechecked the final dark-slide contrast.
- Seven archived source hashes match; all 43 local asset references resolve.
- All 15 unique local evidence/media assets are served by the static preview.
- Scanned all 14 evidence files, decompressing logs, for recognizable credential
  patterns. Raw logs remain outside the presentation directory.
- Repository pre-commit checks passed for authored sources; archived evidence
  remains byte-identical to its indexed source snapshot.
- Independent final content review found no material issues.
- ZIP archive integrity check passed. Video work remains outside this delivery.

## Verification focus

- Five HTTP smoke checks prove reachability and expected response text only.
- The 94m31s job duration includes documentation and blog review work.
- Persistent storage, authenticated workflows and inference remain untested.
- Preserve the actual retry and security-context history without overstating
  production readiness or measured labor savings.
- Raw logs and Secret resources must never enter the deliverable or commit.
- Exported pages must remain readable with no clipped content or missing assets.

## Rulings

- Use the already selected Slidev feature branch. This task adds presentation
  content and its local rendering dependencies; it changes no application code.
- Execute directly as authorized by the user's “Approve and build” response.
  Native build/export and visual inspection provide appropriate verification
  for this reversible presentation task; no application tests are needed.
- Video feasibility remains a separate, pending task in the existing plan.
