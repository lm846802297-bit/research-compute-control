# Research runner control

> **Research compute architecture:** [RESEARCH_COMPUTE_POLICY.md](RESEARCH_COMPUTE_POLICY.md) in each linked research repository is normative: ordinary GPT chat uses ChatGPT Linux for primary calculations; Codex/Work may use the user's Windows local environment or a GPT-managed cloud computer; GitHub stores canonical source/results; Actions is used for independent verification. This repository's Windows self-hosted Actions runner remains a separate, explicitly authorized cross-check path, even when it runs on the same physical PC used by Codex/Work.

Private control repository for trusted research jobs on a Windows self-hosted runner.

## Current status

The native AFH cross-check uses two distinct LZ source revisions. The native exporter checkout is pinned to `3f13b37611d043f823b39ac8ebd8ee7e7ae4ebbc`. The separate P3-B source-translated spatial artifact was generated from the checked-out merge commit `61d7391aae3028276b0846802c19f2404a8a59f4` in LZ workflow run `44950264673`, artifact `14724884350`. These revisions serve different roles and must not be substituted for one another.

The LZ native reference run `44919924790` completed successfully and uploaded artifact `14713605942`. The control-repository workflow is a separate manual cross-check. Its earlier push-triggered run `44346207069` was cancelled and produced no artifact. The PR #2 workflow is manual-only (`workflow_dispatch`); it uploads results to this private control repository and does not publish to the LZ research repository.
## Register the Windows runner

1. Open Settings > Actions > Runners > New self-hosted runner in this repository. Select Windows and x64.
2. On the target computer, follow GitHub's current download, checksum, extraction, and configuration commands. The registration token expires; do not share it or commit it.
3. Use the runner name `RTX5060-WIN11` and add custom label `research-gpu` when prompted. GitHub supplies `self-hosted`, `Windows`, and `X64` automatically.
4. For an initial interactive run, start `run.cmd`. The machine must remain awake. For a service, select the service option during configuration and use an appropriately restricted dedicated Windows account; installing the service requires an administrator shell.
5. Check the runner is Online/Idle. Under Actions, manually run `Windows runner diagnostics` on `main`. It checks CPU, RAM, `nvidia-smi`, and Python availability from the runner account. Logs contain hardware details and are visible to users with repository access.

The custom label is mandatory for the diagnostic. A GPU label does not install drivers, CUDA, Python, or make CPU-only code run on a GPU. Offline runners cause jobs to wait.

## Research integration policy

The project-wide [RESEARCH_COMPUTE_POLICY.md](RESEARCH_COMPUTE_POLICY.md) defines the primary-compute and independent-verification split. Use this Windows self-hosted runner only for explicitly scoped, trusted independent checks (including native Wolfram/Mathematica cross-checks when required); do not route the primary parameter scan here by default.

- Only manually dispatched workflows in this private repository should use its runner. Do not execute untrusted PRs, arbitrary shell commands supplied as inputs, or unreviewed repository refs on the persistent host.
- For cross-repository reusable workflows, use a reviewed full commit SHA for the workflow and separately pin the research source checkout to a reviewed full commit SHA. Record code, data and dependency versions, command, parameters, environment and hashes for each run.
- Public research repos can expose a `workflow_call` entry point. Caller and called workflows owned by the same account can use runners available to the caller. Do not assume existing workflows are automatically compatible with Windows, the GPU, or `workflow_call`.
- Private called workflows require explicit access settings. Cross-repository checkout of private source needs separately scoped read credentials; the control repository's `GITHUB_TOKEN` does not grant access to another private repository. Use least privilege and do not persist checkout credentials.
- The default `GITHUB_TOKEN` does not automatically grant write access to a different research repo. Review results first; any automated cross-repository publishing requires separate narrowly scoped credentials and explicit configuration. Treat run artifacts as belonging to this control repository and subject to retention limits.
- Never overwrite research files or treat a passing technical diagnostic as validation of a physical result.

## Seven-isotope native AFH v6.0 eight-response cross-check

The reviewed workflow is
[xe131-afh-spatial-native.yml](.github/workflows/xe131-afh-spatial-native.yml).

### Source and artifact pins

| Role | Exact revision or run | Artifact |
|---|---|---|
| Native AFH exporter checkout (`source_sha`) | LZ PR #63 merge commit `3f13b37611d043f823b39ac8ebd8ee7e7ae4ebbc`; native reference run `44919924790` | `afh-v6-seven-isotope-eight-response`, artifact `14713605942` |
| P3-B spatial-response input | LZ PR #67 merge checkout `61d7391aae3028276b0846802c19f2404a8a59f4`; run `44950264673`, run head `6f224347b446af5c9f67f061695ef831b6856ee0` | `provisional-afh-spatial-obdme-responses`, artifact `14724884350` |

The two commit pins are intentionally different. The native exporter and
comparison code,
`research/afh_full_response_database/run_afh_full_response_database.wls`
and `compare_native_afh_full_response.py`, exist at
`3f13b37611d043f823b39ac8ebd8ee7e7ae4ebbc`. They are absent at
`61d7391aae3028276b0846802c19f2404a8a59f4`. Therefore
`61d7391aae3028276b0846802c19f2404a8a59f4` is the P3-B artifact's
source/provenance revision, not a valid replacement for the workflow's
native-exporter `source_sha`.

The P3-B artifact was produced by a `pull_request` run. Its job log records
the checkout of merge commit `61d7391aae3028276b0846802c19f2404a8a59f4`.
Its metadata reports `native_mathematica_spatial_crosscheck=false`; it is
a source-translated comparison input, not a native AFH validation result.
The separate native reference run `44919924790` checked out
`3f13b37611d043f823b39ac8ebd8ee7e7ae4ebbc`, passed its native AFH and
spatial-multipole validation steps, and uploaded artifact `14713605942`.
The companion Xe-131 dense charge-dip validation run `44919924782`
also completed successfully.
Its artifact is 2,335,579 bytes with ZIP SHA-256
`0fed6d80870710ef044bdeb9a49de7cb01765a0553a9717ace468e60eaeebe4f`.
The P3-B artifact is 1,125,446 bytes with ZIP SHA-256
`b291a5a1bea1edfbc766084097ebef756b58ecae5fb3cf4aa9ff16a92d6af11c`.
The P3-B metadata and validation record identify it as a source-translated
provisional input, not a native AFH result. Both ZIP digests were independently
recomputed from the downloaded artifacts and match the GitHub artifact API.
The native reference run is not an execution of this control-repository workflow.

### Workflow trigger and runner

The workflow is limited to manual `workflow_dispatch`, has no dispatch
inputs, and has no `push` or `pull_request` trigger. To run this reviewed
definition before PR #2 is merged, select branch
`research/native-p3b-afh-crosscheck-pr65` in the Actions branch selector;
selecting `main` would execute the older main-branch workflow definition.
The source commit and both artifact identities are fixed in the workflow
and are changed only through a reviewed update. It runs on the
`[self-hosted, Windows, X64, research-gpu]` runner. Before dispatch,
confirm that the runner is Online/Idle, `wolframscript -version` works
under its runner account, Wolfram Engine is installed and activated, and
`LZ_RESEARCH_READ_TOKEN` has **Contents: Read-only** permission for the private source checkout and **Actions: Read-only** permission for cross-repository artifact download on
`lm846802297-bit/LZ-3-body-for-hdm`.

The earlier control-repository push-triggered run
[44346207069](https://github.com/lm846802297-bit/research-compute-control/actions/runs/44346207069)
was cancelled and produced no artifact. The PR #2 branch removes that
automatic push trigger. Before comparison, the workflow verifies the fixed P3-B artifact status,
AFH package hash, isotope set, 16,807/9,604 row counts, and both CSV
SHA-256 values. It records the actual Windows OS, CPU, memory, GPU/driver
inventory when available, Wolfram version, Python version, pip version,
installed Python package set, and output checksums in the run archive.

The workflow uploads its result bundle to this
control repository; it does not write to the LZ research repository.

### Calculation and acceptance scope

The native exporter evaluates AFH v6.0
`ResponseNuclear[y,index,0,0]` for response indices 1--8 and the seven
available default-density isotopes: Xe-128, Xe-129, Xe-130, Xe-131,
Xe-132, Xe-134, and Xe-136. The common grid contains 2,401 points from
0 to 0.6 GeV, giving 16,807 isotope/q rows. The eight channels are
M, Sigma-double-prime, Sigma-prime, Phi-double-prime, Phi-tilde-prime,
Delta, M--Phi-double-prime interference, and Sigma-prime--Delta
interference.

The workflow preserves the raw isoscalar package convention, exports
imaginary residuals separately, and checks
`R_X_raw = 4*pi*W_X/(2*J_A+1)`; the equal-p/n factor four is applied
only once in the physical output columns. It also exports the 121-point
Xe-131 dense charge dip and rank-resolved Xe-129/Xe-131 DeltaJ/SigmaPJ
multipoles. The native spatial archive contains 9,604 amplitude rows.

Acceptance requires pointwise relative tolerance `5e-7`, absolute
tolerance `2e-12`, negligible native imaginary parts, rank-sum
reconstruction, and the Cauchy/PSD checks. The P3-B response CSV SHA-256 is
`4e0a23c18d29df3af04a2531e75b7bffeaa3251d0efe97674b2287cf3c25e7e3`;
the rank-resolved multipole CSV SHA-256 is
`e83f7499cd0af4ec8f1974238f09daf4899ca1b87205ec837ae0964a2e8532c6`.
The AFH v6.0 package source SHA-256 is
`dc0742f1c50b0a478adab62f2ddef6c82978ee034c1f177e0ed917deab5dd372`.

This workflow validates native AFH total responses and rank-resolved
spatial multipoles. It does not compute finite-`t_N` nucleon form
factors, the complete three-body `L_munu^(3) W_A^(munu)` contraction,
halo folding, or detector predictions.

### Historical PR #58 provisional artifact

The earlier provisional Python artifact used by PR #58 is a separate
historical input: LZ workflow run `44222087512`, artifact
`14391456283`. Its `source_commit.txt` records
`dcefbeb7ad705b6c03f2b4689200eb1d9252e67b`, with SHA-256
`020420f04915cf0a6cda7cc6581f55b99f672a1dc29d88c921f8fa8d5d759b11`.
This historical source pin does not replace the current native exporter
pin `3f13b37611d043f823b39ac8ebd8ee7e7ae4ebbc` or the P3-B artifact
source pin `61d7391aae3028276b0846802c19f2404a8a59f4`.

The calculation-critical Git blobs recorded for that PR #58 artifact
were identical between its PR head and its checked-out merge revision:

| Historical PR #58 calculation input | Git blob SHA |
|---|---|
| Seven-isotope/eight-response Python generator | `592334e17731278a397cc7a09bd4af7bfe00094b` |
| Native Wolfram exporter | `efffa3876fc61adfc3c80ea8e5f81296623a18ad` |
| Native comparison script | `9b858cafff6d33dcce95637d914e2b77ab17d2d6` |
| AFH v6.0 package source | `91b43b1a4596bebd6073ecd44c0c9464e5dd0c63` |
| AFH Xe-131 base implementation | `5280eb8844fcb562e783a9f74860aedd4a7f7a4e` |

The seven default-density CSV inputs (Xe-128, 129, 130, 131, 132,
134, and 136) also had identical Git blob SHAs in that historical
comparison. The PR #58 artifact remains explicitly
`PROVISIONAL_PYTHON` with `native_afh_validated=false`; it was used
only for the PR #58 provisional fixed-speed diagnostic. A queued or
cancelled job is not a native AFH PASS.

## Native Xe-131 Mathematica full-cross-section run

A manually dispatched Windows self-hosted workflow is provided at
[xe131-native-full-dsigma.yml](.github/workflows/xe131-native-full-dsigma.yml).
It checks out the reviewed LZ research revision
0aa29e0f5aa95f2cf2021dcf598067b2099544b2, runs the native Wolfram Language
Eq. (6) calculation, and compares every point with an independent Python
recomputation. It does not publish or modify the research repository.

Before running it:

1. Register the Windows runner described above and confirm it is Online/Idle.
2. In this control repository, add an Actions repository secret named
   LZ_RESEARCH_READ_TOKEN. Use a fine-grained token restricted to
   lm846802297-bit/LZ-3-body-for-hdm with **Contents: Read-only**. This is
   required because the research repository is private; the control
   repository's GITHUB_TOKEN cannot read it.
3. Run **Windows runner diagnostics** first. Confirm Python and
   wolframscript -version are available on the runner account. Wolfram
   Engine/Mathematica must already be installed and activated on that machine.
4. Under Actions, manually run **Xe-131 native Mathematica full dSigma and
   Python comparison** on main with run_mode=smoke first. After the smoke
   comparison passes, run it again with run_mode=full.

The workflow pins the research source SHA rather than checking out a moving
branch. The default smoke mode runs the green benchmark at the four explicit
101, 200, 267 and 542 keV anchors with nW,nz,nphi=(8,8,4). The full mode uses
all four benchmarks on the complete 5--800 keV grid plus those anchors, with
nW,nz,nphi=(16,16,8). Both use the pole-width prescription and threshold-u2
mapping, 2e-7 relative tolerance for I6 and dSigma/dER, and 2e-8 for the PCHIP
response translation. Both check the factor-four response mapping and the
independent no-factor-four control at 2e-12. The kinematic acceptance also
requires all 4 smoke anchors to be open and, in full mode, the independently
audited open-row counts red/green/black/blue = 157/134/75/134 (500 total),
with positive clearance from the on-shell chi2 pole.

The smoke run uploads xe131-native-mathematica-smoke; the full run uploads
xe131-native-mathematica-full to this control repository. The artifact contains the native Mathematica CSV/metadata,
pointwise Mathematica--Python comparison CSV/JSON, execution transcript,
runtime versions, run manifest, and SHA-256 manifest. The workflow does not
push to the LZ research repository. Treat the artifact as a transfer bundle:
review it, extract it, then commit the individual result files and manifests
to the research repository under its existing output structure.

The source SHA is intentionally fixed. After reviewing a new research commit,
update the pin in the workflow and this README together; do not replace it
with a moving branch or an unreviewed workflow input.

## CI is optional; compute execution does not depend on CI

CI status checks are optional and are not prerequisites for research calculations, numerical validation, artifact handoff, pull-request creation, review, or merge.

- Run calculations directly on an available local machine or an approved runner when that is the most effective route. Do not wait for a CI job to start, continue, or finish research.
- GitHub Actions in this repository may be used as an explicitly reviewed, manually dispatched compute or artifact-handoff mechanism. Such a job is an execution run, not a mandatory CI gate. Its provenance, source revision, inputs, settings, logs, and outputs must still be recorded.
- Scientific and numerical checks remain governed by the research repository's acceptance criteria and may be performed locally or on an approved runner. A CI pass does not replace them.
- If CI is queued, unavailable, disabled, blocked by account/billing limits, cancelled, or fails for infrastructure reasons, continue through an available execution route and record the CI status accurately. Do not report an unrun check as passed or treat an infrastructure failure as a scientific failure.
- Do not require CI status checks as branch-protection gates under this policy. Review and merge decisions depend on the actual diff, provenance, reproducibility, and scientific evidence. Do not change repository protection settings without explicit authorization.
- CI is required for a particular task only when the author explicitly requests it.

## Research result artifact handoff — mandatory

Workflow artifacts in this control repository are temporary transfer products, not the permanent research archive. When a calculation artifact or result bundle is downloaded for publication to a research repository:

1. Verify the originating workflow run, source commit, artifact name, and run metadata.
2. Extract ZIP/TAR bundles before upload. Commit the required native files individually to the destination research repository, preserving the study's directory structure and filenames. This includes source code, tests, configuration, inputs, CSV/JSON/ROOT/HDF5/TXT outputs, reports, metadata, logs/checkpoints needed for audit, and manifests as applicable.
3. Do not commit a newly generated delivery ZIP/TAR as the sole or substitute copy of its extracted contents. The bundle is only a transport container.
4. Include a run manifest and SHA-256 manifest with provenance, code/input revisions, environment, command, numerical settings, output sizes/counts, and checksums.
5. After publishing, retrieve the files from the destination GitHub repository and verify paths, file counts, byte sizes, and SHA-256 checksums. Report archival completion separately from scientific validation.

An original upstream archive may be retained only when its exact container is needed for provenance/reproducibility or extracted inputs cannot legally be redistributed. Record its source/version, size, SHA-256, and extraction command, and publish all extracted files that can be redistributed. A tool-native compressed output may remain in its native format when it is itself the calculation product.

This handoff policy does not authorize automatic cross-repository publishing. Publishing still requires an explicitly reviewed workflow, narrowly scoped credentials, and a deliberate approval step; never expose tokens or run untrusted code on the persistent self-hosted runner.
