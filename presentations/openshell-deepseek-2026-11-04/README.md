# OpenShell with Red Hat EnMaaS DeepSeek

Run `ac05c639abd848998db29ce2352d9a0f` uses
`enmaas/rits/deepseek-ai/DeepSeek-V4.1-Flash-1017` with the saved
`deploy/providers/enmaas-deepseek.json` profile. Source is NVIDIA/OpenShell
`f4a6137513b777c9b362be54baf388f7d45398aa`.

## Original validation

The final original execution `a23ddad8e73849acb7f3bf68b8e09c18` passed all five
planned checks: gateway health, real Linux/nonroot process execution, workspace
persistence across stop/start, allowed example.com access and denied
api.github.com access with a matching supervisor policy event. The first
harness attempt had a workload-template naming error. Both original executions
and all ten recorded attempts remain preserved. No tests were replayed for media.

This is a single-replica functional PoC. Filesystem isolation, independent CNI
enforcement, production OIDC, HA/load/failover and wider Kubernetes portability
were not validated. Network denial establishes supervisor/proxy mediation.

All three UBI images were built from source on their first attempt with unique
`deepseek-ac05c639` tags. Gateway took 38m47s, supervisor 19m18s and sandbox
13m27s. The run used operator-supplied, previously corrected reference
Dockerfiles, prepared fork/protection and cluster grants/placement, plus a
reporting recovery. It is not an unattended run or a controlled model benchmark.

## Source and report

Source changes are on the dedicated
[DeepSeek branch](https://github.com/aicatalyst-team/openshell/tree/autopoc-deepseek-20261104-ac05c639).
The immutable [PoC report](https://github.com/aicatalyst-team/openshell/blob/9e697ea1e0468e277d8d36d0c0fdc0ea41de4c91/runs/ac05c639abd848998db29ce2352d9a0f/report/1a1053badda53ac136bb42795a6c1c781dfd001c9001d82934199ce04212ee1c/poc-report.md)
contains scoped results and original build/deployment provenance.
The immutable [blog draft](https://github.com/aicatalyst-team/openshell/blob/e9f08c609b589ff444a6b9d7ab8d1e4ac62201b7/runs/ac05c639abd848998db29ce2352d9a0f/blog/1d13de78567e157000434afd45f9555ca80f13891f8b39591f4facf1d7dea57f/blog-post.md)
and local HTML preview are also available. Best draft v8 scored 7.75 and was
accepted under the non-interactive PoC threshold of 7.0; it did not meet the
general 8.0 target. The worker exceeded the nine-iteration review limit before
selecting its best draft. The draft retains an unfilled hero-image placeholder.

Sealed evidence contains the earlier pre-blog report revision c559a007.
The final report revision above adds blog bookkeeping; original test evidence
and all five outcomes remain unchanged.

The gateway and supervisor remain in `poc-openshell-deepseek`; the separate
workspace PVC is retained in `autopoc-test`. The earlier Qwen run is untouched.

## Reviewed rendering input

- Evidence: `671776db2d8baaf94790692a8bb98e8d0d4eb99d19c229fb88709344fab7ef80`
- Independently approved narrative: `388609a3dd17771af217553dcb08f2b5dbd144bba86f3eb9b5874bcfb6ce3569`
- Review: `223ee6d68d4f8749fa45aa290b476f0265c67523657d289fd3a34fe00fc248f6`
- Sealed v2 input: `296f5503c725f6527e2bae76d48093c52d47483f840e4c7fb69219396ea1f2f1`
- Renderer: `quay.io/aicatalyst/autopoc-executive-renderer@sha256:adb000ed14305bc3cbd2b9d7d6bc967f98bb26265e52d15ec40f5c08e570beca`

Independent narrative and blog reviews used the same DeepSeek EnMaaS endpoint.
The initial OpenCode reviewer waited on an external-directory permission prompt;
a report-only replacement parsed a run-scoped `/workspace/**` allowance and
completed independent API reviews. A new child Task was not verified.

## Publication

Google Drive/Docs publication remains blocked by the configured service
account's account-not-found error. No cloud media or Docs link is claimed.
Offline restricted OpenShift rendering is verified. Job
`autopoc-openshell-deepseek-redhat-20261104-4` used restricted-v2, UID
1000880000, no API token/cloud credentials, a read-only root and an egress-deny
policy. Limits were 2 CPU, 3Gi memory and 4Gi scratch. It completed in 213.735s,
with 2,013,990,912-byte peak memory and 30,562,722-byte final scratch footprint.

The [six-page PDF](output/ocp-redhat/executive-summary.pdf),
[two-minute silent video](output/ocp-redhat/executive-summary.mp4),
[editable source ZIP](output/ocp-redhat/executable-source.zip),
[browser ZIP](output/ocp-redhat/static-site.zip),
[executive summary](output/ocp-redhat/executive-summary.md),
[manifest](output/ocp-redhat/presentation-manifest.json),
[report](output/poc-report.md) and [blog preview](output/blog/blog-preview.html)
are available locally. Serve the extracted browser ZIP over HTTP.

All artifact hashes/sizes, archive integrity, exact recovery seal and packaged
branding bytes/notices were verified. PDF fonts are embedded Red Hat
Display/Text/Mono; video is H264, 1280x720, 30fps, exactly 120s, without audio.
Every PDF page and all four animated acts, including the opening transition,
were visually reviewed. The source ZIP includes the selected reviewed seal.

Earlier renders exposed label/caption overlap, camera compression and opening
camera clipping. They are historical attempts. The selected input uses compact
vertical tiers and combines health/execution in one of four acts, covering all
five scenarios. Actual frozen camera projection sampled at 500 instants found
no estimated label collisions or clipping; manual visual QA was also required.
No feature code or PoC test result was changed.

The regular selected seal is retained locally in `inputs/render-inputs.zip`
and on the run PVC at
`/workspace/render-inputs/operator-executive/render-inputs.zip`. Use the exact
renderer digest above for recovery. Media status is `generated_unpublished`;
this acceptance did not validate production Google retrieval/dispatch/upload.
Generated inputs, outputs and visual QA files are ignored by Git; all required
branding assets/fonts/notices are already versioned in `renderer/assets/redhat/`.
