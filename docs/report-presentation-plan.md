# Headless report presentations and UI test evidence

Status: requirements captured; implementation and Slidev feasibility validation pending.

Related work: [AIPCC-32887](https://redhat.atlassian.net/browse/AIPCC-32887)
and [AIPCC-33008](https://redhat.atlassian.net/browse/AIPCC-33008).

## Goal

Generate a presentation automatically from a completed AutoPoC run as a
headless step after `poc_report`. No interactive Slidev login, desktop session,
or installation on the operator's computer should be required.

AutoPoC's existing model can summarize the report and evidence into `slides.md`.
Slidev itself does not require an AI model: rendering an existing deck is a
separate, repeatable operation. A fixed template can also generate slides
without a model call.

## Inputs and evidence preservation

Preserve the PoC report, `poc-state.yaml`, test results, generated manifests,
sanitized execution logs, available traces, screenshots, and recordings before
the original runner pod is removed. The existing recorder skill notes that the
original state file is ephemeral; live cluster discovery alone cannot recover
the complete historical test evidence.

Associate artifacts with the project, run identifier, scenario identifier,
attempt, capture time, and pass/fail/error outcome. Maintain an artifact index
that maps slide claims and selected media back to their source evidence.
Report unsupported or inconclusive results explicitly rather than inventing
measurements or treating missing tests as passes.

## UI capture during validation

For projects with a UI, capture evidence during the original Playwright test
execution. A later replay is a separate run and must not be presented as the
original validation result. Projects without a UI skip browser capture and
still use their report, API/CLI test results, and execution logs.

- Take screenshots at meaningful checkpoints: initial page, completed action,
  visible result, and failure state when capture remains possible. Use these
  images directly in the presentation.
- Record selected UI validation scenarios to show the actual interaction and
  outcome. Save recordings for later selection of short demo clips.
- Save Playwright traces for selected UI scenarios, including failed attempts,
  to investigate actions, DOM snapshots, and network activity. A Playwright
  trace is distinct from a video and from application/distributed traces.
- Keep scenario outcomes and failure details alongside the media. Record
  capture failures separately so missing media does not hide test failures.
- Keep credentials and sensitive input out of captured scenes. Sanitize logs
  and review screenshots, recordings, and trace contents before publishing;
  traces can contain request/response data and page content.

## Proposed generation flow

1. Read the archived report and indexed run evidence.
2. Generate a concise `slides.md` covering project purpose, environment and
   architecture, deployment, validation results, relevant failures/retries,
   and the evidence-based suitability conclusion and limitations.
3. Add selected UI screenshots, diagrams, short sanitized excerpts, and source
   references. Use presenter notes for explanatory detail where appropriate.
4. Render Slidev with headless Chromium through Playwright and inspect the
   slides for overflow, missing assets, and unreadable diagrams.
5. Save the Markdown source, assets, browser-viewable deck, PDF, and artifact
   index alongside the report; link the outputs from the run.

Recommended execution approach: a separate post-report container Job with
Node.js, pinned Slidev dependencies, Playwright, Chromium, and required fonts.
It consumes archived evidence so presentation generation can be retried
without repeating the PoC tests. Presentation-generation failures should be
recorded separately from the PoC validation outcome. These are proposed
implementation choices, pending feasibility validation.

## Video feasibility and remaining decisions

Evaluate recording the Slidev deck with Playwright and using ffmpeg to combine
the result with selected UI test clips. Slidev's built-in recording starts
through the browser UI; reliable headless automation must be demonstrated.
Its documented CLI export formats do not include video.

The following remain open:

- Whether the first implementation delivers a presentation only or both a
  presentation and summary video, and whether UI clips appear in that video.
- Whether narration is required and, if so, its source and synchronization.
- The artifact storage destination, access policy, and retention limits.
- Capture scope, duration/size limits, and retry behavior in the container.

The first feasibility run should produce a viewable deck from one completed
PoC, demonstrate headless rendering, and verify that the claims and captured
UI results can be traced to the same run. Video composition remains part of
the Epic evaluation rather than a completed capability.

## References

- [Slidev syntax and presenter notes](https://sli.dev/guide/syntax)
- [Slidev exports](https://sli.dev/guide/exporting.html)
- [Slidev recording](https://sli.dev/features/recording)
- [Existing report instructions](../.opencode/skills/run-poc/references/poc-report.md)
- [Existing recorder skill](../.opencode/skills/record-demo/SKILL.md)
