# OpenShell: AutoPoC run and Red Hat presentation validation

Run `a189bdf54ac849778a8ed7b228db599f` tested NVIDIA/OpenShell source
`f4a6137513b777c9b362be54baf388f7d45398aa` using the AutoPoC branding feature
at `560503d8c59b46cb08ede0f4f6c0b02ae898073f`.

The source-built gateway and supervisor deployed on OpenShift 4.21.8. Final
validation recorded **3 pass, 0 fail, 1 error, 1 skip, 0 incomplete**. Health,
metrics and mTLS API checks passed. Sandbox readiness hit IBM's disk-attachment
quota; network-policy validation was skipped. Three original executions and
15 attempts remain preserved. Network/filesystem isolation, production OIDC,
load/HA and wider Kubernetes portability are unverified.

The run required operator assistance with fork protection, compiler selection,
namespace PDB permissions and gateway placement. It does not establish an
unattended successful deployment or production readiness.

## Verified local deliverables

Generated files are ignored by Git. On the machine that performed this run:

- [Six-page branded PDF](output/ocp-redhat/executive-summary.pdf)
- [Two-minute silent video](output/ocp-redhat/executive-summary.mp4)
- [Editable source ZIP](output/ocp-redhat/executable-source.zip)
- [Browser build ZIP](output/ocp-redhat/static-site.zip)
- [Executive summary](output/ocp-redhat/executive-summary.md)
- [Artifact hashes and provenance](output/ocp-redhat/presentation-manifest.json)
- [Canonical PoC report](output/poc-report.md)
- [Blog preview](output/blog/blog-preview.html)

The browser ZIP must be served over HTTP after extraction; opening its HTML
directly with `file://` is unsupported. The source ZIP includes the sealed input,
review, evidence, local branding assets, notices and dependency lock for recovery.

Published source/report records are on the
[OpenShell fork](https://github.com/aicatalyst-team/openshell/tree/autopoc).
The corrected [report revision](https://github.com/aicatalyst-team/openshell/blob/5f12aa33a71f93bd42dbb12117c77b64995cd44d/runs/a189bdf54ac849778a8ed7b228db599f/report/7e8fcd2a0e81f54ef247724da2908e3edcd19e865c75792f9b5ed23e5c79e438/poc-report.md)
and [blog revision](https://github.com/aicatalyst-team/openshell/blob/ec93cc3dd2b2ddc53dfe421aa35cee53a524d98b/runs/a189bdf54ac849778a8ed7b228db599f/blog/e78959b0eb1cda6c0c317248fc9363340347e0402339d96da771d3c7a9f36cf9/blog-post.md)
are immutable. Earlier revisions remain history.

## Restricted OpenShift render acceptance

Job `autopoc-openshell-redhat-20261104-3` completed using the renderer's normal
entrypoint and template revision 2. This was an offline acceptance harness,
independent of the production Google download/dispatch/publication path.

- Renderer: `quay.io/aicatalyst/autopoc-executive-renderer@sha256:adb000ed14305bc3cbd2b9d7d6bc967f98bb26265e52d15ec40f5c08e570beca`
- Selected sealed input: `dd362ae6985df9772882956494998794809b6acbf19c93e7ec7a3b2a2f5b64e5`
- Corrected evidence: `54a1ec8647ced009608813405c9d22819d0eb0117cafc6686d4e9555c41d2b5d`
- Separately reviewed narrative: `4f8e40cd6eaa8463955f969d14a33e901a059a3842936f70349272b22eeb130f`
- SCC `restricted-v2`, UID `1000880000`, read-only root, no workload API token
  or Google credential, and an egress-deny policy selecting only the render Pod.
- Limits: 2 CPUs, 3 GiB memory, 4 GiB scratch. Measured 222.59 seconds,
  1,894,318,080-byte peak memory and 30,753,779-byte final scratch footprint.

Artifact hashes, sizes, ZIP integrity, recovery input and every packaged branding
asset/notice were verified. All six PDF pages and late error/skip animation frames
were visually inspected. The PDF embeds Red Hat fonts. The video is H264,
1280x720, 30 fps, exactly 120 seconds, without an audio stream.

An initial render rejected a limitations overflow. A second passed automatic
checks but manual review caught overlapping diagram labels and a transcript
panel collision. Concise limitations, explicit node placement and removal of
the transcript panel resolved those issues, with separate configured-model
re-review after each input change. Automatic checks alone do not catch every
diagram-label collision; manual visual review remains necessary. No PoC tests
were replayed to generate these artifacts.

## Publication and recovery

The presentation artifacts are **generated_unpublished**. Google Drive and
Google Docs publication remain blocked by the configured service account's
`account not found` failure. No cloud presentation/video links were created.

The selected regular input ZIP is available locally at
`inputs/render-inputs.zip` and on the retained run PVC at
`/workspace/operator-executive/render-inputs.zip`. Use it with the exact renderer
digest above for recovery. Do not substitute an older seal or renderer image.
Private worker diagnostics and client TLS keys are excluded from published
artifacts and presentation packages.
