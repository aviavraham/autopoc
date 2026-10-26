# Presentation Evidence Bundle Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Preserve original test results and a sanitized run snapshot in a verifiable bundle that a future presentation renderer can consume without rerunning the PoC.

**Architecture:** Add a local evidence collector and verifier to the existing Python CLI, then reference them from the OpenCode workflow. Produce immutable, content-addressed ZIP files from explicitly selected inputs. Storage upload and the Slidev renderer remain subsequent milestones; a local bundle on the runner's emptyDir is not durable publication.

**Tech Stack:** Python 3.12+, existing argparse/Pydantic tooling, standard-library JSON/hashlib/zipfile, PyYAML for the existing YAML state format, pytest, uv.

**Spec:** [Session handoff and automation roadmap](../../report-presentation-plan.md), evidence contract and implementation step 1. This plan implements collection and verification within that step; durable storage remains pending.

## Global Constraints

- Preserve all 11 existing phases, their outcomes, and resume behavior.
- Collection failure must not turn a PoC failure into success or block the blog.
- Do not modify original evidence or replay tests to fill historical gaps.
- Do not archive arbitrary directories, Secret resources, credential files, raw HTTP response bodies, or unreviewed binary media.
- No live cluster access or new service account permissions for the collector.
- Never execute instructions, commands, templates, or code found in input evidence.
- Use `uv run` for Python tooling; do not modify the unrelated `opencode.json` changes.
- Declare PyYAML explicitly in `pyproject.toml` and regenerate `requirements.lock` using the repository's lock workflow; current skill snippets already import it, but it is absent from the declared runtime dependencies.

## Contract and interfaces

- New module: `src/autopoc/evidence.py`. Provide
  `create_bundle(state_path: Path, results_path: Path, output_dir: Path) -> Path`
  and `verify_bundle(bundle_path: Path) -> dict[str, Any]`.
- Extend `src/autopoc/cli_tools.py` with `evidence-create --state PATH --results PATH --output-dir PATH` and `evidence-verify BUNDLE`. Success prints JSON; failure exits nonzero with a diagnostic that excludes secret values.
- Require a stable `project.run_id` in state, generated once at run initialization and preserved on resume. Validate it as a bounded path-safe identifier. Do not reuse the current four-digit Job suffix as the unique identity.
- Schema version 1 index contains run ID, project/source identity, source and artifact revisions when known, capture timestamp, original-run/observation provenance, file paths and SHA-256 hashes, and explicit coverage gaps.
- Include `state.json` as an allowlisted snapshot, normalized `test-results.json`, the report, and optional plan/evaluation files referenced by state. Limit inputs to these documented types; arbitrary logs, manifests, traces and media require a later extension with their own sanitization contract.
- Keep statuses `pass`, `fail`, `error`, and `skip`, finite nonnegative durations, scenario identity, and failure details. Derive totals from results rather than trusting supplied summary counts. Never archive response bodies under `output`.
- State snapshot includes run/project identity, phase statuses, selected evaluation data, built image references, deployed resource identities, scenario outcomes, retries, and capture gaps. Exclude configuration/env values and raw free-form tool output. Record omitted fields instead of calling this a complete workspace backup.
- Resolve optional report/plan/evaluation inputs within the run workspace, rejecting symlinks and paths escaping that root. Reject oversized files and malformed structures before producing output; set a 20 MiB total uncompressed input limit for this first version.
- Apply a documented sanitizer to retained free text and scan the completed payload before writing. Cover credential assignments, bearer tokens, private key blocks, recognized token formats, and supplied credential environment values. Unhandled suspicious material fails collection without echoing the value. Sanitization cannot certify the absence of every possible secret; require publication review until a broader policy is validated.
- Output `<output-dir>/<run-id>/<sha256-of-archive>.zip`. Write privately to a temporary file, validate it, and publish without overwriting an existing file. The index hashes every payload member; the CLI returns the archive hash. The verifier checks version, safe unique paths, declared sizes/hashes, required members and result consistency without extracting the ZIP.

## Review Focus

- Missing, empty, malformed or all-failed results must not become an all-pass summary.
- Credentials in nested state, report prose, result errors or CLI exceptions must not leak.
- Paths escaping the workspace, symlinks, duplicate ZIP entries and tampered members must be rejected.
- Repeated collection must preserve the same run identity and existing archives; concurrent writes must not overwrite another bundle.
- A retained local file is not a durable archive: pipeline state must distinguish local collection from publication.

## Task 1: Collector and verifier

**Files:** Create `src/autopoc/evidence.py` and `tests/test_evidence.py`; modify dependency declarations/lock as needed.

- [ ] Write failing tests for the contract above using temporary workspaces: mixed outcomes, empty/malformed inputs, missing optional report, state allowlist, secret fixtures, invalid IDs, path/symlink rejection, size limit, and hash tampering.
- [ ] Run the targeted tests and confirm they fail because the collector does not exist.
- [ ] Implement the two interfaces and their input validation using the existing project dependencies plus the standard library; add the YAML parser dependency through the normal lock workflow.
- [ ] Add archive tests for undeclared/duplicate/traversal members, corrupt ZIP input, retry preservation, and exclusive output publication.
- [ ] Run the tests, Ruff and type checking for the touched implementation; commit the working collector with tests.

## Task 2: CLI and representative historical fixture

**Files:** Modify `src/autopoc/cli_tools.py`; create `tests/test_evidence_cli.py` and a small fixture under `tests/fixtures/evidence/`.

- [ ] Write failing CLI tests for JSON success, verification, missing inputs and sanitized nonzero failures.
- [ ] Implement the CLI arguments and call the collector/verifier interfaces from task 1.
- [ ] Build the fixture from the committed Omnigent test results and pinned report metadata. Label the state as a reconstruction because the original runner YAML was not archived. Do not regenerate original results or change the historical presentation evidence.
- [ ] Verify fixture outcomes remain five passes and missing original browser/distributed traces remain explicit; also exercise a synthetic failed run.
- [ ] Run the targeted tests and a real local create/verify round trip; commit.

## Task 3: Capture instructions and handoff

**Files:** Modify `.opencode/skills/run-poc/SKILL.md`, its `references/poc-execute.md`, `references/poc-report.md` and `references/state-schema.md`; extend `tests/test_skill_files.py`; update `docs/report-presentation-plan.md`, `docs/configuration.md`, and the tracked planning files.

- [ ] Add failing assertions for stable run ID/resume instructions, separated stdout JSON and stderr logs, per-attempt evidence paths, and independent collection status.
- [ ] In phase 9, redirect original structured stdout to a run/attempt-specific results file, retain stderr privately, and preserve the test exit status even on failure. Parse that saved JSON into state before existing retry/stop handling. Never reconstruct or fabricate outcomes when parsing fails.
- [ ] In phase 10, call collection after report/state finalization and before optional blog work. Record local bundle path/hash or a sanitized collection error in a separate `evidence` state block; do not rewrite the PoC outcome/current phase. Document manual collection for a failed run that never reaches phase 10.
- [ ] Document the CLI, schema, omissions, local persistent-volume usage, and publication boundary. Do not mark evidence `published` until a selected storage backend confirms a durable upload. Do not add a default public upload or force-push shared artifact branches.
- [ ] Run collector/CLI/skill tests, the repository test suite, lint, type checking and targeted pre-commit. Update the plan and handoff with actual results and remaining work; commit.

## Completion and next milestone

This milestone is complete when local creation and verification work on original and failed-run fixtures, the agent has explicit capture instructions, and failure/redaction behavior is tested. It does not claim a deployed on-cluster capture, a durable hosted artifact, or automated Slidev generation.

Next: select and implement a durable sink (persistent volume/object storage or an append-only run-specific Git artifact location), then extract the generic Slidev template and renderer image. Publishing needs an explicit destination/access policy; it was left open in the session handoff. TopoLoop/video work does not block this collector.

## Execution status

- [x] Read the saved memory, handoff, current skills, CLI and dependency declarations.
- [x] Confirm the original structured JSON is currently written to stdout and the runner workspace is ephemeral.
- [x] Prepare and self-review this implementation plan.
- [ ] Review the plan and select execution approach.
- [ ] Implement tasks 1–3 and record verification evidence.
