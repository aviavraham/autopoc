# Vale Prose Linting - Detailed Implementation Plan

## Progress

| Phase | Status | Notes |
|-------|--------|-------|
| Phase 1: Core Infrastructure | Done | All 4 tasks complete |
| Phase 2: Agent Integration | Done | All 3 agents integrated |
| Phase 3: Testing & Validation | Done | 21 tests, all pass |

## Architecture

The Vale linting feature is implemented as a **shared utility function**
(`vale_lint_and_revise`) in `src/autopoc/tools/vale_lint.py`, not as a
separate graph node. Each markdown-producing agent calls it inline after
writing its artifact.

### Flow

```
Agent writes markdown file
  -> vale_lint_and_revise(file_path, llm, max_revisions)
    -> Run `vale --output=JSON <file>`
    -> If findings exist:
      -> Feed findings + original content to LLM with revision prompt
      -> Write revised content to file
      -> Re-run Vale
      -> Repeat up to max_vale_revisions times
    -> Return (revised_content, findings_summary)
```

### Key Design Decisions

1. **Utility function, not graph node**: Simpler, no new edges/routing
2. **Optional Vale**: Silently skip with warning if `vale` binary missing
3. **Conservative revision**: LLM treats Vale findings as suggestions
4. **No pipeline failure**: Vale issues never block the pipeline

## Phase 1: Core Infrastructure

### 1.1 Config field (`config.py`) -- Done
- Added `max_vale_revisions: int = Field(default=3, ...)`

### 1.2 State field (`state.py`) -- Done
- Added `vale_findings: list[dict]` to PoCState

### 1.3 Vale utility (`src/autopoc/tools/vale_lint.py`) -- Done
- `run_vale(file_path) -> list[dict]`: Run vale --output=JSON
- `vale_lint_and_revise(file_path, llm, max_revisions) -> tuple[str, list]`:
  Full lint-revise loop
- Handles missing `vale` binary gracefully

### 1.4 Revision prompt (`src/autopoc/prompts/vale_revision.md`) -- Done
- System prompt for conservative, selective revision
- Emphasizes technical accuracy over style compliance

## Phase 2: Agent Integration

### 2.1 poc_report agent -- Done
- Calls `vale_lint_and_revise` after writing poc-report.md, before commit

### 2.2 poc_plan agent -- Done
- Calls `vale_lint_and_revise` after writing poc-plan.md, before commit
- Integrated in both phase 1 (one-shot) and phase 2 (ReAct fallback) paths

### 2.3 blog_post agent -- Done
- Calls `vale_lint_and_revise` after finalization, before commit

## Phase 3: Testing & Validation

### 3.1 Unit tests -- Done (21 tests)
- TestValeAvailable: vale binary detection (2 tests)
- TestRunVale: subprocess handling, JSON parsing, error cases (8 tests)
- TestFormatFindings: findings formatting (2 tests)
- TestValeLintAndRevise: full loop logic (9 tests)
  - No-findings early return
  - Revision fixes issues
  - Max revisions respected
  - Truncation guard
  - LLM failure handling
  - Missing file handling
  - Code fence stripping
  - Explicit max_revisions parameter
  - Multi-part LLM content

### 3.2 Verification -- Done
- `ruff check src/ tests/` -- All checks passed
- `pyright src/` -- 0 errors, 0 warnings
- `pytest tests/ --ignore=tests/e2e` -- 629 passed

---

# OpenCode Harness Rewrite - Implementation Plan

## Progress

| Phase | Status | Notes |
|-------|--------|-------|
| Phase 1: Design | Done | Architecture doc + 3 skills + 14 reference files |
| Phase 2: Foundation | Pending | CLI wrappers, opencode.json, skill testing |
| Phase 3: Container Image | Pending | Dockerfile update, Makefile |
| Phase 4: K8s Manifests & Scripts | Pending | Job, CronJob, run-autopoc.sh |
| Phase 5: Cleanup | Pending | Remove LangGraph, agents, graph, old tests |
| Phase 6: Testing & Validation | Pending | New test suite, E2E validation |
| Phase 7: GitHub Repository Safety | Done | Default branch protection and dedicated AutoPoC write branch |

## Architecture

Replace LangGraph multi-agent pipeline with OpenCode skill-driven architecture.
Full design: `.opencode/plans/opencode-harness-rewrite.md`

### Key Changes
- LangGraph -> OpenCode skills (run-poc, run-sheet, blog-create)
- ReAct agents -> OpenCode following skill instructions
- PoCState TypedDict -> Progressive YAML state file
- LangChain @tools -> bash commands + standalone Python scripts
- shiv binary container -> OpenCode binary container

### Skill Files Created

```
.opencode/skills/
  run-poc/
    SKILL.md                     # 11-phase pipeline instructions
    references/
      intake.md                  # Phase 1 analysis rules
      poc-plan.md                # Phase 4 plan generation
      containerize.md            # Phase 5 Dockerfile creation
      deploy.md                  # Phase 7 K8s manifest generation
      poc-execute.md             # Phase 9 test script generation
      poc-report.md              # Phase 10 report generation
      state-schema.md            # YAML state file schema
      retry-strategy.md          # Retry loop logic
      error-triage.md            # Apply failure classification
      ubi-dockerfile-rules.md    # UBI/OpenShift compatibility rules
  run-sheet/
    SKILL.md                     # Sheet-driven batch pipeline
    references/
      prefilter.md               # Heuristic scoring rules
  blog-create/
    SKILL.md                     # Blog generation pipeline (adapted from ai-asset-registry)
    assets/
      blog-template.html         # HTML preview template
    references/
      scoring.md                 # Review scoring and iteration rules
      reviewer-architect.md      # Structure reviewer rubric
      reviewer-content.md        # Content reviewer rubric
      reviewer-formatting.md     # Formatting reviewer rubric
      reviewer-image.md          # Image reviewer rubric
      html-preview-guide.md      # HTML conversion guide
```

## Phase 1: Design -- Done

### 1.1 Architecture Document -- Done
- Created `.opencode/plans/opencode-harness-rewrite.md`
- Covers: execution model, state file design, skill design, container image,
  K8s manifests, standalone scripts, test strategy, migration path, dependencies,
  risk assessment

### 1.2 run-poc Skill -- Done
- SKILL.md: 11-phase pipeline with retry loops
- 10 reference files covering all phases, state schema, retry strategy, error triage

### 1.3 run-sheet Skill -- Done
- SKILL.md: 7-step sheet processing pipeline
- 1 reference file for heuristic pre-filtering

### 1.4 blog-create Skill -- Done
- SKILL.md: Adapted from ai-asset-registry/blog-create
- 6 reference files: scoring, 4 reviewers, HTML guide
- 1 asset: blog-template.html

## Phase 2: Foundation -- Pending

### 2.1 CLI Wrappers for Python Tools
Add `__main__.py` or CLI entry points to:
- `src/autopoc/tools/repo_digest.py`
- `src/autopoc/tools/gitlab_tools.py` (as gitlab_client)
- `src/autopoc/tools/github_tools.py` (as github_client)
- `src/autopoc/tools/quay_tools.py` (as quay_client)
- `src/autopoc/tools/strategy.py`
- `src/autopoc/tools/llm_proxy.py`
- `src/autopoc/sheet.py` (as sheet_reader/sheet_writer)
- `src/autopoc/tools/vale_lint.py` (as vale_runner)

### 2.2 OpenCode Configuration
- Create `opencode.json` with skill definitions and provider config

### 2.3 Headless Mode Verification
- Test OpenCode non-interactive/headless mode for pod execution
- Verify `opencode --skill run-poc --prompt "..."` syntax

## Phase 3: Container Image -- Pending

### 3.1 Dockerfile Update
- Add OpenCode binary download (from GitHub releases)
- Copy skill files into image
- Update ENTRYPOINT to opencode
- Keep existing tools (kubectl, oc, vale, podman)

### 3.2 Makefile
- Update image build/push targets
- Add OpenCode version variable

## Phase 4: K8s Manifests & Scripts -- Pending

### 4.1 Job Manifest
- Update `deploy/base/job.yaml` for OpenCode args

### 4.2 CronJob Manifest
- Update `deploy/base/cronjob.yaml` for run-sheet skill

### 4.3 Run Script
- Update `scripts/run-autopoc.sh` for new Job template

## Phase 5: Cleanup -- Pending

### 5.1 Remove LangGraph Dependencies
- Remove from pyproject.toml: langgraph, langchain-*, typer, rich
- Regenerate requirements.lock

### 5.2 Remove Obsolete Code
- Remove: src/autopoc/agents/, graph.py, state.py, cli.py, cli_batch.py
- Remove: llm.py, context.py, debug.py
- Keep: tools/, prompts/, templates/, data/

### 5.3 Remove Obsolete Tests
- Remove: test_graph_*, test_retry_loop, test_llm_fallback, test_cli_*
- Remove: all agent tests (test_intake, test_containerize, etc.)
- Keep: tool tests, API client tests, strategy tests, sheet tests

## Phase 6: Testing & Validation -- Pending

### 6.1 New Tests
- Standalone script CLI tests
- State file YAML read/write tests
- Skill file validation tests (well-formed, references exist)

### 6.2 E2E Validation
- Full pipeline run with OpenCode in pod
- Retry loop validation
- Sheet processing validation
- Blog generation validation

## Phase 7: GitHub Repository Safety -- Done ✅

### 7.1 Protect GitHub default branches -- Done ✅
- Added a reusable GitHub repository policy for forked and freshly created repositories.
- Detects the repository's actual default branch (`main`, `master`, or another configured default).
- Enables pull-request review protection and disables force pushes and deletions.

### 7.2 Isolate AutoPoC writes -- Done ✅
- Uses the `autopoc` branch for GitHub source synchronization and pipeline commits.
- Replaced GitHub `--all` and `HEAD` pushes with explicit `autopoc` pushes.
- Retains `autopoc-artifacts` for generated plans, reports, tests, and blog artifacts.

### 7.3 Verification -- Done ✅
- Added skill validation tests for the branch-protection payload and safe push commands.

## PR #57 Review Follow-up ✅

- URL-encode GitHub default branch names before using them in REST API paths.
- Require configured status checks and invalidate stale approvals after new commits.
- Add regression assertions covering the hardened branch-protection policy.

---

# AutoPoC Report Video Feature Ticket

The ticket and sample milestones below retain their historical record. For
implementation, use Phase 6 and the revised executive design as the current
scope: silent presentation video in Phase 1, Google TTS in Phase 2, and a
separate renderer Job with durable input/recovery material. Older architecture
recommendations are not execution instructions.

## Progress

| Phase | Status | Notes |
|-------|--------|-------|
| Phase 1: Ticket definition | ✅ Done | Jira Feature AIPCC-32887 created with scope and Definition of Done |
| Phase 2: Slidev feasibility Epic | ✅ Done | AIPCC-33008 filled with feasibility scope, technical constraints, and Definition of Done |
| Phase 3: Headless presentation and UI evidence requirements | ✅ Done | Documented the requested headless step and proposed capture/evidence flow; implementation choices remain under evaluation |
| Phase 4: Sample presentation and recording feasibility | Sample complete; video work transferred | Omnigent archive and Slidev/PDF complete; executive recording belongs to Phase 6, original UI clip composition is optional later work |
| Phase 5: Presentation automation handoff | Documentation ✅ Done; implementation pending | Session record, repositories/tools, reproduction and staged integration roadmap documented |
| Phase 6: Executive presentation and video scope | Core implementation merged ✅; rollout/acceptance open | Fork master merge verified 3 November; restricted OCP render and personal Drive reference passed; unattended publication, failure acceptance, model review and worker rollout remain |
| Phase 7: Upstream security remediation | ✅ Done | User merged fork PR #2; upstream PR #66 updated to 231fc31 and all six CI jobs, including security, passed |
| Phase 8: Default Red Hat presentation branding | Implementation/local acceptance ✅; OCP rollout pending | 379 Python/eight Node tests and full constrained offline UBI render passed; draft fork PR #3 ready for review; new registry publication and OCP validation remain |
| Phase 9: OpenShell PoC with fork branding updates | Intake/evaluation/fork ✅; continuation submitted | Dedicated run token successfully synchronized the pinned autopoc branch; operator verified protected main; Job r4 resumes the same run at planning |

## Phase 1: Ticket Definition ✅

### 1.1 Create Jira feature ticket — ✅
- Created [AIPCC-32887](https://redhat.atlassian.net/browse/AIPCC-32887) in the AIPCC project.
- Captured report-driven video generation scope and acceptance criteria for background, setup, testing, evidence-based suitability conclusion, and report linkage.

## Phase 2: Slidev Feasibility Epic ✅

### 2.1 Fill Jira Epic AIPCC-33008 — ✅
- Added a bounded evaluation of Slidev with AutoPoC, Playwright, and the existing workflow-validation/recording capabilities.
- Recorded the documented recording/export behavior, automation questions, sample-video deliverable, and feasibility recommendation criteria.

## Phase 3: Headless Presentation and UI Evidence Requirements ✅

### 3.1 Document headless generation and UI capture requirements — ✅
- Added [the presentation plan](../../docs/report-presentation-plan.md) covering a headless post-report step without an interactive Slidev login or local operator installation.
- Recorded screenshots at meaningful UI test checkpoints, recordings of selected scenarios, and Playwright traces captured during the original validation run.
- Defined run/scenario evidence linkage, credential handling, and preservation of ephemeral run artifacts for repeatable presentation generation.
- Distinguished AutoPoC's content-generation model from Slidev's model-free rendering.
- Kept presentation/video output scope, narration, storage, and runtime limits open; the separate Job and non-blocking failure behavior are recommendations pending validation.

## Phase 4: Sample Presentation and Recording Feasibility — Historical milestone

### 4.1 Archive one completed PoC — ✅
- Archived the Omnigent run `autopoc-omnigent-8554`: pinned report/plan/evaluation/blog/manifests, original test JSON, sanitized runner and both build logs, and selected job/build metadata.
- Indexed claims and source hashes; labeled later UI/cluster observations separately.
- Recorded missing original browser/distributed traces, UI recordings and workspace state export instead of inventing those artifacts.

### 4.2 Generate and inspect the sample Slidev deck — ✅
- Created `presentations/omnigent-2026-10-20/` with 14 stakeholder slides and 4 technical appendix slides.
- Built a static browser deck and exported/visually inspected an 18-page PDF with headless Chromium.
- Documented reproducible commands, original test coverage, temporary storage, source-branch drift, SCC/report discrepancy and source-linked evidence.
- Added two TopoLoop slides: eight-resource setup topology with three acts and five-check scenario flow driven by the archived original results. Preserved pinned upstream source/licenses, reduced-motion behavior, still-frame controls and deterministic PDF frames.

### 4.3 Recording and composition feasibility — Transferred
- Executive presentation recording is required Phase 1 work in section 6.5. Composition with actual UI test clips is optional later work and is not a prerequisite for the silent executive video. No video capability is claimed by the existing sample.

## Phase 5: Presentation Automation Handoff

### 5.1 Document session and continuation plan — ✅
- Updated [the presentation handoff](../../docs/report-presentation-plan.md) with completed work, repository roles and pinned revisions, tools, sample commands, source files, and evidence boundaries.
- Verified the integration targets against the current OpenCode skill/YAML-state runtime, runner scripts, and container definitions.
- Defined an immutable evidence bundle, a separate headless renderer Job, independent presentation status, publishing, and acceptance checks.
- Added the handoff to the root README documentation index.
- Added a dedicated local HTTP section with Node/npm prerequisites, Slidev and Vite commands, localhost URLs, directory guidance, and Ctrl+C shutdown instructions.

### 5.2 Implement post-report presentation generation — Superseded by Phase 6
- The active Phase 1 deliverables are executive Markdown, editable Slidev/TopoLoop source, browser build, PDF and a silent MP4, with durable inputs, recovery and report links.
- Use the ordered work in sections 6.3–6.7. Original UI capture is optional later work; Google TTS narration is Phase 2. Runtime implementation remains pending.

### 5.3 Plan the evidence collector milestone — ✅
- Resumed from the saved handoff on 26 October 2026 and checked the current OpenCode workflow, CLI, dependency declarations, and repository state.
- Added `docs/superpowers/plans/2026-10-26-presentation-evidence.md` with the bundle contract, collector/verifier and CLI interfaces, original-result capture instructions, test coverage, and publication boundary.
- Collection implementation, storage integration, and automatic rendering remain pending. The plan is ready for review and execution-method selection.

## Phase 6: Executive Presentation and Video Scope

### 6.1 Record expanded feature requirements — ✅
- Added `docs/superpowers/specs/2026-10-26-executive-summary-design.md` for automatic executive Slidev/TopoLoop presentations and videos based on the PoC report.
- Defined repository purpose, testing strategy, actual execution and report-based OpenShift/Kubernetes suitability as required topics in both outputs.
- Included report Executive Summary links, reusable headless recording/encoding, independent failure states, and configurable publication through existing tooling.
- Recorded the user's phase boundary: silent on-screen-text video first, Google TTS voice-over in Phase 2.
- Updated the authoritative handoff and collector plan to reflect the expanded scope; feature implementation remains pending.
- An independent builder review identified one material ambiguity; clarified automated content/publication checks with no per-run human approval gate. Design approval and implementation planning remain pending.

### 6.2 Incorporate Astra assessment — ✅
- Revised the design, collector plan and handoff for all six findings: report lifecycle, durable recovery material, attempt history, extension resume, resource isolation and conflicting active scope.
- Changed the cluster execution choice to a separate renderer Job. Synchronized silent video as a Phase 1 requirement and Google TTS as Phase 2.
- Added acceptance cases for report → extension → blog persistence, original workspace deletion, interrupted uploads, completed-blog resume and forced renderer failure.
- These are documentation corrections only. Implementation remains on hold while the user reviews; a detailed whole-feature implementation plan is still required.

### 6.3 Preserve source evidence and artifact history — ✅
- ✅ Collector Task 0: implemented the common append-only Git writer and CLI; migrated evaluation/plan/tests/report/blog artifact instructions to stable staging and immutable revisions. Independent scoped review passed. Verified 71 affected tests, including real competing Git writers and source-checkout preservation.
- Implemented indexed capture, explicit final execution and incomplete-result handling, sanitized verified evidence ZIPs and historical/synthetic fixtures. Actual child-process/HTTP retry tests cover original attempts; collection never replays PoC tests.
- Evidence review found quoted-credential and index-metadata scan bypasses; both fixed with RED→GREEN regression tests and scoped re-review.

### 6.4 Seal durable render inputs and implement recovery — ✅ implementation; live acceptance open
- Implemented hash-bound review receipts, four-topic JSON and bounded graphs/scenes, pinned image identities and sealed inputs. Existing OpenCode performs production content review; manual reference review receipts are labeled synthetic.
- Added preallocated Drive IDs, upload intents/receipts, verified downloads, append-only Git checkpoints with a bounded immutable input fallback, standalone recovery and phase-independent reconciliation.
- Ten real-Git/fake-provider recovery tests pass, including missing receipts, pending input upload, pre-UID submission and immutable/symlink-safe recovery. Real personal Drive reference publication passed with verified inputs and all six output receipts.

### 6.5 Implement executive rendering and silent video — ✅ local implementation
- Implemented trusted Slidev/TopoLoop components, JSON-only narrative, explicit scene readiness/reset/still frames, source/static ZIPs, PDF, executive Markdown and silent H264 MP4 with ffprobe validation.
- Local host and constrained UBI renders passed. Full six-scene reference: 119.966667-second silent H264 at 1280x720/30 fps; 226.673-second render, peak 2,046,562,304 bytes. PDF text, topology, execution and conclusions visually inspected; two Node boundary tests pass. Broader live fixture/resource acceptance remains open. Google TTS stays Phase 2.

### 6.6 Isolate cluster execution — ✅ restricted OCP render; failure acceptance open
- Published renderer digest `sha256:f65ec5e266f743ce2ace49f978db1c99e68d8430527c9d4710ee56a4bfdab76d` under `quay.io/aicatalyst/autopoc-executive-renderer`. Added Job/RBAC/launcher configuration; renderer has no workload token or RoleBinding.
- Constrained local test: arbitrary UID, read-only root, network disabled, 2 CPUs/3 GiB/4 GiB scratch; 68.539 seconds, peak 2,465,705,984 bytes for short diagnostic. No runtime dependency downloads.
- ✅ 28 October: published image completed six-scene rendering through its normal entrypoint under restricted-v2, UID1000880000, read-only root/no workload token, 2 CPUs/3 GiB/4 GiB scratch. Runtime178.895s; memory peak1,841,668,096 bytes; scratch46,609,764 bytes. Five output hashes verified after export; six-page PDF, silent H264120s1280x720/30fps verified. Forced OOM/disk/admission acceptance and worker-image rollout remain open.

### 6.7 Publish executive links and validate end to end — implementation ✅; live validation in progress
- Implemented independent output validation/publication, durable receipts and report-link revisions after blog completion. Partial failures retain valid presentation/PDF artifacts.
- Google service account is invalid. User authorized temporary personal OAuth; all six reference artifacts uploaded and downloaded for hash verification. Checkpoint complete; report links persisted in the local reference Git remote after completed-blog state. Personal credentials stay local; cluster Secret unchanged.
- Current verification: 366 supported tests passed, 83.48% coverage; pinned Ruff clean; Pyright zero errors; two Node tests passed. Astra's five implementation findings closed with scoped re-review.
- Remaining acceptance: unattended OCP publication and failure scenarios, model-service narrative review and main-worker build/rollout. OCP offline rendering now passes. New cluster outputs remain local because the temporary personal OAuth file was removed from /tmp; original verified Drive reference remains available. No original PoC test replay; no claim of production rollout.

### 6.8 PR-readiness corrections — ✅ 28 October 2026
- Fixed the timeout test's `/proc` check/read race with a bounded wait for child termination; it still fails if the child survives. Production timeout behavior is unchanged.
- Removed trailing whitespace from the two sample font-license copies without changing their text.
- Fresh verification: 366 supported tests passed, 83.48% coverage; focused renderer suite 11 passed; Ruff clean. Unrelated `opencode.json` remains excluded.
- Renewed OCP access. The repository shell sets a different kubeconfig from the default used by `oc login`; acceptance commands explicitly select the renewed `/home/aavraham/.kube/config`.
- ✅ Restricted OCP render/export passed. The first new attempt rejected ConfigMap symlinks; the passing harness copies input to a regular scratch file. Production path validation is unchanged. Updated AGENTS.md and prepared a reviewable PR description.

### 6.9 Publish a draft PR to the user's fork — ✅ 28 October 2026
- Pushed `codex/aipcc-33008-slidev-epic` to the `aavraham` remote, `aviavraham/autopoc`, at the user's request.
- Opened [draft PR #1](https://github.com/aviavraham/autopoc/pull/1) against that fork's `master`; GitHub reports it mergeable. The description records actual validation and remaining rollout/acceptance work.
- Attached the PR to this task. Unrelated `opencode.json` remains uncommitted and excluded. No merge or default-branch push was performed.

### 6.10 Review project state after the user's merge — ✅ 3 November 2026
- Confirmed the user's fork `master` points to merge commit `6b564e5` for PR #1. The current upstream `aicatalyst-team/autopoc` master is `3f266c7` and does not contain this feature.
- Read the merged implementation, workflow/configuration references, handoff and active plans. Phase 6 rollout/acceptance remains open: production service-account repair, unattended publication, forced resource/admission failures, model-service narrative review and main-worker build/rollout.
- Fresh scoped verification: 169 Python tests passed across artifacts, capture, evidence, executive tools, Drive and skill validation; all four Node renderer tests passed. This review did not run a new render, cluster acceptance or full coverage suite.
- Existing `opencode.json`, `.coverage` and `uv.lock` changes remain excluded. Historical draft-PR and implementation-pending wording above describes earlier milestones; this entry records the verified post-merge state.

### 6.11 Check upstream integration and CI — ✅ 3 November 2026
- Refetched upstream `master`, still `3f266c7`. The feature is proposed through open, non-draft [upstream PR #66](https://github.com/aicatalyst-team/autopoc/pull/66), from `aviavraham:master` at `6b564e5` to upstream `master`.
- GitHub reports no merge conflicts (`MERGEABLE`), but merging is blocked and approval is required (`REVIEW_REQUIRED`); there are no submitted reviews.
- [PR CI run 41645246344](https://github.com/aicatalyst-team/autopoc/actions/runs/41645246344): lint, renderer tests, typecheck, Python tests and worker image build passed. Security failed: pip-audit reports 10 findings across certifi, opentelemetry-api, pyasn1 and urllib3 (the certifi advisory appears twice).
- [Upstream master CI run 41590181862](https://github.com/aicatalyst-team/autopoc/actions/runs/41590181862) has the same audit findings; its other CI jobs passed. These dependency findings therefore also exist before the executive feature is merged. No dependency changes, PR comments, pushes or merges were made during this check.

## Phase 7: Upstream Security Remediation — ✅ Done

### 7.1 Fix and verify vulnerable dependency pins — ✅ 3 November 2026
- Reproduced PR #66's security failure locally: the pinned lock produced the same 10 reported findings across four transitive dependencies.
- Regenerated `requirements.lock` with pip-compile and SHA-256 hashes under Python 3.12, selectively updating certifi to 2026.10.23, opentelemetry-api to 1.45.2, pyasn1 to 0.6.5 and urllib3 to 2.8.0. All other package versions are unchanged; no new runtime dependencies were declared.
- Verified a fresh Python 3.12 environment: hash-checked installation succeeded, pip-audit reports no known vulnerabilities, pip check passed, 371 tests passed with 85.55% coverage, pinned Ruff passed, Pyright reported zero errors/warnings, and all four renderer tests passed.
- Prepared on `codex/fix-upstream-security`. Upstream PR #66 tracks the user's fork `master`; merging this fix into that branch requires user approval. The upstream PR's existing CI result remains failed until its source is updated and CI reruns.

### 7.2 Publish the reviewable security fix — ✅ 3 November 2026
- Pushed only `codex/fix-upstream-security` and opened [fork PR #2](https://github.com/aviavraham/autopoc/pull/2), targeting the fork's `master`. GitHub reports it mergeable with no conflicts; no fork CI checks are currently reported.
- Attached fork PR #2 and upstream PR #66 to this chat. Neither default branch was pushed or merged; applying the fix to upstream PR #66 remains pending user merge approval.

### 7.3 Confirm user merge and upstream security recovery — ✅ 3 November 2026
- The user merged fork PR #2 as `231fc31`; upstream PR #66 automatically adopted that source revision. Fresh GitHub inspection confirms all six checks passed: lint, renderer tests, typecheck, security, Python tests and image build. Upstream PR #66 itself remains open.

## Phase 8: Default Red Hat Presentation Branding — Implementation/local acceptance ✅; OCP rollout pending

### 8.1 Inspect references and define the branding phase — ✅ 3 November 2026
- Read the supplied 107-page standard template, inspected representative title/content/color pages visually, recorded its SHA-256 and checked official Red Hat typography, color and logo standards.
- User confirmed Red Hat branding must be the default for newly generated presentations, PDFs and videos. Proposed a red opening scene and light content theme, local fonts/corporate assets, branded TopoLoop status colors and consistent source/browser/PDF/video outputs.
- Added [the branding design](../../docs/superpowers/specs/2026-11-03-red-hat-presentation-branding-design.md), including factual-content preservation, six-scene timing, legacy input/recovery compatibility, packaging, visual QA and rollout acceptance. An independent builder check reported no important unanswered questions.
- Created `codex/red-hat-presentation-branding` from merged fork master `231fc31`. Unrelated local files remain excluded. No renderer implementation or deployment is claimed.

### 8.2 Save all required branding resources in the repository — ✅ 3 November 2026
- User explicitly required local repository copies of every needed branding component. Saved nine official WOFF2 font files (Display/Text/Mono, Regular/Medium/Bold), unmodified official corporate/hat SVGs, upstream font license/authorship notices and a per-file source/hash/size manifest under `renderer/assets/redhat/`.
- Fonts are pinned to RedHatOfficial/RedHatFont commit `6bb1048a6402b0076ea04f42951ec66263cd1437`. Saved the supplied 107-page PDF unchanged under `docs/branding/` and documented its provenance; it is a design reference, not a runtime background.
- Verified all 13 upstream-file hashes/sizes, WOFF2 headers and font families, self-contained SVG structure and both logo previews. The reference PDF matches its original size and SHA-256. No dependency or renderer-behavior changes were made; future source/site packaging must include required local assets and licenses.
- Updated the design to require all later-added artwork, fonts, components and styling to live in the repository, without runtime CDN or operator-local-file dependencies.

### 8.3 Approve the visual design and prepare implementation — ✅ 3 November 2026
- User explicitly approved the design and requested implementation. Added the ordered implementation plan with version/packaging, shared layouts and visual acceptance tasks; execution is in progress on the existing feature branch.

### 8.4 Implement branding and compatible packaging — ✅ 3 November 2026; acceptance in progress
- Bundle official assets/fonts with provenance, implement the shared slide shell and diagram palette, and bind template revision 2 through sealing, rendering and source/recovery packaging while preserving revision-1 recovery.

#### 8.4.1 Bind template revision and local source packaging — ✅ 3 November 2026
- New seals default to revision 2; revision-1 verification/recovery remains supported, with explicit legacy sealing for the original pinned image. The new wrapper rejects fresh v1 rendering and preserves matching complete legacy output reuse.
- Forwarded sealed versions to Node and source/recovery metadata. Added allowlisted local assets/public files and renderer helpers to editable archives.
- Watched the new boundary/archive tests fail, then pass: 35 affected Python tests passed. Layout, Node validation and visual acceptance remain in progress.

#### 8.4.2 Implement the shared brand shell and offline validation — ✅ 3 November 2026
- Implemented six branded scenes using local Display/Text/Mono fonts, a red opening, light content, unmodified corporate/hat SVGs, source references and readable footer bands. Added a renderer-owned light TopoLoop palette with orange failures and green successes; vendored source is unchanged.
- Verified asset inventory/hashes before deck preparation, copied assets/notices to source and browser trees, awaited fonts/logos, and added scene overflow checks before PDF/video capture. Updated the renderer image definition to package all helpers and local assets.
- Seven Node and 25 Python input/render tests pass, including failing-first asset/version cases and a regression preventing publication of a static deck after content validation fails. Initial browser preflight and six-page PDF generation passed; full video/image/cluster acceptance follows in 8.5.

### 8.5 Verify and roll out branded artifacts — Local acceptance ✅; OCP rollout pending
- Verify representative successful/failed fixtures, every PDF page and video frames, offline assets, meaningful compatibility/overflow tests, the supported checks and a restricted OpenShift render before updating deployment configuration.

#### 8.5.1 Resolve review findings and verify local behavior — ✅ 3 November 2026
- Independent code review found one important diagram-text clipping gap, with no critical/minor findings. Added failing browser/Node regressions, then fixed graph-region bounds and deterministic checks of PDF stills, act transitions and original log events. Expanded the scenario terminal layout without dropping evidence text.
- All 379 Python tests pass with 85.62% coverage; eight Node tests pass. Actual browser checks cover six scenes, oversized claims/titles/counters/diagram labels, missing fonts/logos and zero external asset requests. Archived Omnigent, synthetic failure/incomplete and twelve-node fixtures rendered locally; all six Omnigent PDF pages were inspected.
- Full two-minute rendering in the rebuilt constrained image remains in progress. OCP acceptance/publication and deployment configuration changes are not claimed.

#### 8.5.2 Validate constrained UBI rendering and source recovery — ✅ 3 November 2026
- Built reviewed image `d8e792a29858`, local digest `sha256:cf628a4323262e6ee152e7b9825f5705c8b8cac76cf9e34d5143f12eb9bc53ac`. Normal-entrypoint render passed with read-only root, no networking, UID28128/group0, 2 CPUs, 3 GiB memory and 4 GiB scratch.
- Original reviewed Omnigent evidence produced a six-page PDF, silent120.000s H264 MP4 at1280x720/30fps, source/browser ZIPs and Markdown. Runtime213.81s; peak memory2,504,695,808 bytes; scratch45,614,563 bytes. Status is `generated_unpublished`, not deployed/published.
- Verified all output hashes/sizes, all13 original asset hashes in both archives, embedded PDF fonts and source preparation after extraction. Inspected all six final PDF pages and the late login video frame that previously clipped its duration log.
- Local ignored outputs: `presentations/omnigent-2026-10-20/output/redhat-branding-2026-11-03/`. No PoC replay. Registry publication, restricted OCP acceptance and coordinated worker/renderer rollout remain pending.

#### 8.5.3 Publish the reviewable feature branch — ✅ 3 November 2026
- Pushed `codex/red-hat-presentation-branding`, opened [draft fork PR #3](https://github.com/aviavraham/autopoc/pull/3) targeting fork master and attached it to this chat. GitHub reports no conflicts; no fork CI checks are reported.
- Updated renderer/configuration/workflow docs with local assets, v2/v1 compatibility, overflow handling and rollout requirements. No default branch was merged/pushed; deployed configuration is unchanged.

## Phase 9: OpenShell PoC — Intake/evaluation/fork ✅; continuation submitted

### 9.1 Select project and prepare the updated worker — ✅ 4 November 2026

- User selected `https://github.com/NVIDIA/OpenShell` for the new-project run using the fork's branding updates. Fork PR #3 remains open; no merge is required to test its explicit head.
- Cloned and pinned OpenShell to `f4a6137513b777c9b362be54baf388f7d45398aa`. Inspected its Helm chart, OpenShift install guide and runtime support matrix. Actual cluster qualification, Agent Sandbox prerequisites and NetworkPolicy enforcement remain unverified.
- Built a clean archived AutoPoC `560503d8c59b46cb08ede0f4f6c0b02ae898073f` as `localhost/autopoc-opencode:openshell-560503d`, image ID `e7e1fe0fa9d3ba8a2c772872a534352fc160c825d5c35d90f81c42f7f45be2a3`. Container smoke check confirmed template revision 2 and the updated branding workflow. Existing reviewed renderer remains available locally.
- Preserved the unrelated working-tree model/provider configuration, coverage file and uv.lock; the image uses the committed configuration. Saved local preflight state, pinned run prompt and build log in `/tmp/autopoc-openshell-20261104/`.
- Both saved kubeconfigs returned Unauthorized. The user supplied the cluster OAuth URL; its token login page is open for the user to authenticate and renew CLI access. No credentials were copied into planning documents.

### 9.2 Restore access, publish images and submit the Job — ✅ 4 November 2026

- Renew cluster login, inspect configured credentials safely, publish the matching worker/renderer and pin per-run digests. Local Quay login is absent; cluster credential access cannot yet be checked.
- Qualify the real cluster and run the eleven phases, preserving original validation attempts. Record mandatory failures explicitly. Reconcile branded presentation/video publication separately, including any storage failure.
- No PoC Job, OpenShell fork, deployment, test result or new published artifact is claimed from this preflight.

- Renewed CLI access with the user-provided login. Confirmed OpenShift 4.21.8, OVNKubernetes and a running Agent Sandbox controller. Runtime qualification remains part of the PoC.
- Published worker `quay.io/aicatalyst/autopoc-opencode@sha256:74edde1b4589df2fbed1d969d1516d1a0385f898d8bd25adf68ab7d0c50bab54` and renderer `quay.io/aicatalyst/autopoc-executive-renderer@sha256:adb000ed14305bc3cbd2b9d7d6bc967f98bb26265e52d15ec40f5c08e570beca`. This publishes the reviewed branding renderer; it does not yet establish restricted OCP rendering acceptance.
- Registry authentication and the pipeline GitHub identity verified. Google service-account refresh still returns account not found; cloud publication is explicitly blocked. The per-run configuration uses the user's existing Qwen provider with environment-referenced credentials, avoiding Vertex dependency. Shared CronJobs and credential Secrets remain unchanged.
- Created namespace `poc-openshell`, narrow gateway runtime RBAC and additional runner sandbox/network/exec permissions only in that namespace. No privileged SCC or cluster-admin was granted. The chart must use `rbac.clusterScoped.create=false` and gateway ServiceAccount `openshell`.
- Submitted `autopoc-openshell-20261104` in `autopoc-test` with per-run configuration, pinned images, model endpoint preflight and a 10 GiB persistent workspace. Server dry-run passed. Deployment, tests and executive outputs are not yet validated.

### 9.3 Resolve initial workspace scheduling — ✅ 4 November 2026

- Initial Job never reached its model check: IBM CSI reported the selected node had reached its maximum of 12 secondary disks. Preserved the first Job as suspended. The first replacement CPU candidate had insufficient unreserved CPU and was suspended before startup.
- Compared CPU reservations and disk attachments, then submitted `autopoc-openshell-20261104-r2` on `ocp-gb-xz56l-worker-1-hhkzb`, reusing the same empty workspace. Kubernetes reported `SuccessfulAttachVolume`; the worker image is pulling. No OpenShell validation attempt has been executed or replayed by these infrastructure retries.

### 9.4 Verify model access and correct the launcher — ✅ 4 November 2026

- In-cluster model preflight verified the configured Qwen endpoint and model. The r2 worker then exited before intake because the launcher's array-valued `--file` option consumed the following prompt as another filename. Saved its private log and retained the failed Job.
- Corrected only the per-run launch arguments to use the full prompt positionally. Server dry-run passed; `autopoc-openshell-20261104-r3` is Running, loaded the run-poc skill, created its eleven-phase checklist and began pipeline setup. Same image digests, model, namespace and persistent workspace; no PoC test replay.

### 9.5 Complete OpenShell intake and evaluation — ✅ 4 November 2026

- Verified the worker's persisted state for run `a189bdf54ac849778a8ed7b228db599f`: intake and evaluation are completed, pinned source commit is `f4a6137513b777c9b362be54baf388f7d45398aa`, and no errors are recorded. The staged repository digest and evaluation were written by the worker.
- Fork safety/protection is now in progress. Build, deployment and test phases remain pending.

### 9.6 Record mandatory fork failure and preserve recovery state — ✅ 4 November 2026; resumption blocked

- Monitor original attempts, record every phase outcome, preserve failure states and validate any generated branded artifacts from this new project. No result may be inferred from Job submission alone.

## Hosted Model Options Review — ✅ Done, 4 November 2026

### Progress

| Phase | Status | Notes |
|-------|--------|-------|
| Inspect alternatives to cluster Qwen | ✅ Done | Reviewed active configuration, committed default, launchers, Python settings and official OpenCode documentation; no inference or deployment performed |
| Review Red Hat EnMaaS onboarding | ✅ Done | Read authenticated Source page and OpenCode v1 recipe; identified gateway/authentication and exact hosted model IDs; credential/inference checks remain unperformed |
| Try DeepSeek through EnMaaS | ✅ Done | Authenticated API response/tool round trip, local OpenCode and existing worker image tool/session checks passed; opt-in configuration saved; no full PoC run or shared configuration changed |

### Review configuration and hosted provider paths — ✅

- The uncommitted `opencode.json` selects `vllm/qwen3.8-27b-abliterated` and hardcodes the cluster endpoint. The committed default is `google-vertex/claude-opus-4-6@default`. Preserved the user's existing configuration.
- Anthropic direct and Vertex are the smallest configuration changes: `scripts/run-autopoc.sh` and base Job/CronJob manifests already inject Anthropic credentials and Vertex project/location/service-account settings. A working provider credential and explicit model selection are still required; the prior OpenShell preparation recorded a Google service-account refresh failure, not successful Vertex access.
- OpenCode also documents OpenAI, OpenRouter, Bedrock, Azure and hosted OpenAI-compatible providers. These require provider-specific configuration/authentication and, where absent, credential injection into the Kubernetes workload. Provider support does not establish model availability or PoC success.
- The entrypoint only executes OpenCode. `LLM_BASE_URL`, `LLM_MODEL` and fallback fields retained in Python settings are not translated into OpenCode provider/model selection. The launcher currently has no `--model` option; OpenCode itself supports `run --model provider/model`.
- `OGX_*` controls LLM access for deployed PoC applications separately. Replacing the agent model alone does not remove any application-side Qwen dependency.
- Verification: inspected files and git diff; local OpenCode 1.14.19 help confirms `--model`; local Anthropic/Vertex model listings returned `Provider not found`, so no authenticated cloud access was verified. The worker Dockerfile pins OpenCode 1.15.10, which was not executed during this review.
- References: [OpenCode providers](https://opencode.ai/docs/providers/), [model selection](https://docs.opencode.ai/docs/models/), [CLI](https://opencode.ai/docs/cli/). Runtime configuration, credentials and cluster resources were not changed.

### Review EnMaaS connection details — ✅ 4 November 2026

- Read the user-linked [EnMaaS onboarding page](https://source.redhat.com/departments/products_and_global_engineering/workload_management/enmaas#tool-opencode) through the authenticated browser. It explicitly documents OpenCode v1's singular `provider` schema and `@ai-sdk/openai-compatible` runtime, matching the worker's major version.
- Gateway base URL: `https://api.enmaas.devshift.net/v1`. Chat Completions uses bearer authentication; Messages uses `x-api-key` plus `anthropic-version`. The documented environment variable is `ENMAAS_KEY`; AutoPoC could instead reference its existing injected `LLM_API_KEY` variable without recording any secret in JSON.
- Exact hosted model IDs documented: `rits/zai-org/glm-5-3` and `rits/deepseek-ai/DeepSeek-V4.1-Flash-1017`. A pinned OpenCode selection would be `enmaas/<exact-model-id>`. Catalog results are credential-dependent and may omit usable hosted models; absence from discovery alone is not proof of denied inference.
- The page also describes commercial model routing subject to gateway support, credentials and budget. Do not infer that every upstream model is available to this key. Provider runtime availability must be verified in the actual worker.
- The page's explicit GLM limit is 1,000,000 context tokens and 65,536 output tokens, while its example JSON declares 1,048,576 context tokens. Its generic text-only warning also conflicts with the v1 DeepSeek image example. Use text requests and conservative documented limits until independently verified; do not copy GLM metadata onto DeepSeek.
- Documentation review only: no key read or stored, no inference request, no provider configuration or active Job changed. An authenticated response and a harmless function-tool round trip remain necessary before claiming AutoPoC compatibility for the user's key/model.

### Try DeepSeek through EnMaaS — ✅ 4 November 2026

- User authorized the DeepSeek trial and supplied the key through `/home/aavraham/.config/enmaas/key` (mode 0600, outside Git). Loaded it directly into request headers/process environment without displaying or committing it.
- Tested exact model `rits/deepseek-ai/DeepSeek-V4.1-Flash-1017` at the EnMaaS Chat Completions endpoint. Plain response returned `OK`; a forced `echo_probe` function call returned correct arguments and the follow-up returned `TOOL_OK` (three requests, 882 total reported tokens).
- Isolated local OpenCode 1.14.19 completed a single permitted `pwd` tool call and returned `TOOL_OK`. The existing worker image `e7e1fe0fa9d3ba8a2c772872a534352fc160c825d5c35d90f81c42f7f45be2a3`, OpenCode 1.15.10, also completed `pwd` and generated `TOOL_OK`, independently confirmed in its exported session.
- Worker 1.15.10 JSON command output omitted final text despite a completed stored reply. Reproduced with plain/tool prompts; explicit client token limits did not change the symptom. Recorded as a command-output limitation, not an inference/tool failure; no unrelated OpenCode upgrade attempted.
- Added opt-in `deploy/providers/enmaas-deepseek.json`, referencing existing `LLM_API_KEY` injection and declaring text-only input. Updated `docs/configuration.md` with local/new-Job activation, precedence and validation limits. Verified JSON parsing, actual secret substitution and inline model precedence over the working-tree Qwen default without printing resolved credentials.
- Sanitized smoke evidence is under `/tmp/autopoc-enmaas-deepseek-20261104/`. No full eleven-phase PoC was launched, active Job restarted, shared Secret changed or default `opencode.json` modification committed.

- Worker credential received HTTP 403 for fork/repository creation and write-branch creation. It then expanded troubleshooting into write-access probes on unrelated repositories; this exceeded the requested OpenShell scope. Saved logs show failed probes (invalid branch endpoints returned 404), with no successful probe write reported. Stopped the worker and suspended Job r3; no worker Pod remains active.
- Using the existing local operator login, created the actual NVIDIA fork at `https://github.com/aicatalyst-team/openshell`, protected `main` with one approving review, enforced administration, required `Helm Lint`, disabled force pushes/deletions, and pushed pinned source f4a6137 only to `autopoc`. Verified that branch SHA and the default-branch protection. The operator credential was not copied into the cluster.
- Recorded `fork.status=failed`, sanitized authentication error and operator-stop details in the persistent run state. Intake/evaluation remain complete; plan, build, deployment and test phases are pending. Executive/blog are skipped because the mandatory fork phase stopped before report/test evidence; no branded output was generated.
- Backed up private state and staged intake/evaluation/failure documents under `/tmp/autopoc-openshell-20261104/`; source checkout/state also remain on the 10 GiB workspace PVC. Temporary recovery Pod has no API token or project credentials.
- User requested token-generation guidance. Resume requires a token restricted to `aicatalyst-team/openshell` with Contents and Administration read/write, stored in a separate per-run Secret. Do not change the shared batch token. Recovery instructions must explicitly prohibit probing unrelated repositories and must stop on authentication errors.

### 9.7 Verify the dedicated credential and submit same-run recovery — ✅ 4 November 2026

- Preserve run ID, original completed phases and failure history. Verify exact-destination access, resume from fork, and continue through the remaining phases or a new explicit mandatory failure. Google Drive service-account repair remains a separate publication prerequisite.

- User saved `autopoc-openshell-github/GITHUB_TOKEN` in `autopoc-test`. Verified repository/source reads and actual pinned-source synchronization to `autopoc` using that credential. Branch-protection API read is denied (403), so no broader token permission was requested: the operator separately verified existing main protection with local access.
- Fork setup is complete through operator actions. Prepared Job `autopoc-openshell-20261104-r4` to reuse the same run ID, source pin and PVC. Its recovery initializer configures only the exact fork remote/autopoc branch, preserves original errors and records their resolution, then advances to planning. The new token is referenced only for this Job; the shared batch Secret remains unchanged.
- Resume instructions explicitly prohibit unrelated repository probes, alternate destinations/credentials and default-branch writes, and require immediate stop on 401/403. The worker continues from phase 4; no completed PoC phase or validation test is replayed. Server dry-run passed and the Job was submitted.

### 9.8 Publish the new PoC plan and staged evaluation — ✅ 4 November 2026

- Verified actual GitHub artifact publication with the dedicated run credential: plan revision `fd8650d7837a3e70382bfe9c9efcb185ee5456edd58314d44e3d0080746867a2`, commit `af11dd785b6bf978549b9be70c83084eb9b2ebf6`; evaluation revision `f4aaea5b09642d06044a25456f91a803d9f1feee78a2a606a7c630ad9df9147d`, commit `cf562a10dde0116e5f7f524cfe204aaf65801756`. Both are append-only entries under this run on `autopoc-artifacts`.
- Configured the worker's global GitHub credential helper for the separate temporary artifact repositories. These successful publications establish write access; worker Administration permission remains unnecessary after operator-completed fork protection. Source main remains protected and untouched by AutoPoC pushes.

### 9.9 Complete and publish source container definitions — ✅ 4 November 2026

- Verified source branch commit `d441ca245ad346fb60d125477c2880420726bf71` on the exact OpenShell fork. It adds gateway/supervisor/sandbox UBI Dockerfiles, merged build-context exclusions, and copies of the plan/evaluation. Push was only to `autopoc`; the immutable plan/evaluation revisions remain on the artifact branch.
- This establishes committed container definitions, not compiled images or deployment success. Build, deployment, original validation and media verification remain outstanding.

### 9.10 Build, deploy, validate and review new-project outputs — In progress

- Confirm recovery initialization, complete planning/containerization/build/deployment/validation/reporting or record a new explicit mandatory failure. Preserve original attempts and report actual media/storage outcomes separately.

- Operator review checkpoint for final report/narrative: the five-case plan's denied-network probe uses a documentation address and accepts generic connection failure, which alone cannot prove policy caused the denial. NetworkPolicy presence also does not prove CNI enforcement. Require a reachable-target control/separate enforcement evidence before claiming isolation validation, or explicitly retain coverage gaps. Filesystem policy is not independently exercised by the five planned cases. Preserve original capture assertions/counts and never replay tests merely for slides. Review details saved privately in `/tmp/autopoc-openshell-20261104/plan-review.json`.
