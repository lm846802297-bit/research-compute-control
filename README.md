# Research runner control

Private control repository for trusted research jobs on a Windows self-hosted runner.

## Current status

The Windows self-hosted runner is configured for native Wolfram/Mathematica research jobs. The seven-isotope AFH v6.0 eight-response workflow is now pinned to LZ-3-body-for-hdm commit dcefbeb7ad705b6c03f2b4689200eb1d9252e67b. Its latest run (44222502400) is queued for the self-hosted runner; no native output or scientific validation is available yet. The workflow uploads an artifact to this control repository and does not publish to the research repository.

## Register the Windows runner

1. Open Settings > Actions > Runners > New self-hosted runner in this repository. Select Windows and x64.
2. On the target computer, follow GitHub's current download, checksum, extraction, and configuration commands. The registration token expires; do not share it or commit it.
3. Use the runner name `RTX5060-WIN11` and add custom label `research-gpu` when prompted. GitHub supplies `self-hosted`, `Windows`, and `X64` automatically.
4. For an initial interactive run, start `run.cmd`. The machine must remain awake. For a service, select the service option during configuration and use an appropriately restricted dedicated Windows account; installing the service requires an administrator shell.
5. Check the runner is Online/Idle. Under Actions, manually run `Windows runner diagnostics` on `main`. It checks CPU, RAM, `nvidia-smi`, and Python availability from the runner account. Logs contain hardware details and are visible to users with repository access.

The custom label is mandatory for the diagnostic. A GPU label does not install drivers, CUDA, Python, or make CPU-only code run on a GPU. Offline runners cause jobs to wait.

## Research integration policy

- Only manually dispatched workflows in this private repository should use its runner. Do not execute untrusted PRs, arbitrary shell commands supplied as inputs, or unreviewed repository refs on the persistent host.
- For cross-repository reusable workflows, use a reviewed full commit SHA for the workflow and separately pin the research source checkout to a reviewed full commit SHA. Record code, data and dependency versions, command, parameters, environment and hashes for each run.
- Public research repos can expose a `workflow_call` entry point. Caller and called workflows owned by the same account can use runners available to the caller. Do not assume existing workflows are automatically compatible with Windows, the GPU, or `workflow_call`.
- Private called workflows require explicit access settings. Cross-repository checkout of private source needs separately scoped read credentials; the control repository's `GITHUB_TOKEN` does not grant access to another private repository. Use least privilege and do not persist checkout credentials.
- The default `GITHUB_TOKEN` does not automatically grant write access to a different research repo. Review results first; any automated cross-repository publishing requires separate narrowly scoped credentials and explicit configuration. Treat run artifacts as belonging to this control repository and subject to retention limits.
- Never overwrite research files or treat a passing technical diagnostic as validation of a physical result.

## Seven-isotope native AFH v6.0 eight-response cross-check

The reviewed workflow is
[xe131-afh-spatial-native.yml](.github/workflows/xe131-afh-spatial-native.yml).
It checks out the exact LZ research commit
`dcefbeb7ad705b6c03f2b4689200eb1d9252e67b` and runs the AFH v6.0 native
Wolfram exporter for all seven available xenon default-density isotopes:
Xe-128, Xe-129, Xe-130, Xe-131, Xe-132, Xe-134, and Xe-136.

The calculation exports AFH `ResponseNuclear[y,index,0,0]` for indices 1--8
on the common 2,401-point grid from 0 to 0.6 GeV. The eight channels are
M, Sigma-double-prime, Sigma-prime, Phi-double-prime, Phi-tilde-prime,
Delta, M--Phi-double-prime interference, and Sigma-prime--Delta interference.
It preserves the raw isoscalar package convention and exports imaginary
residuals separately. The same pinned source is used to generate the Python
database; the workflow then compares all 16,807 isotope/q rows and all eight
responses point by point. Acceptance requires relative tolerance 5e-7,
absolute tolerance 2e-12, and negligible native imaginary parts.

The workflow also checks the raw-to-physical conversion
`R_X_raw = 4*pi*W_X/(2*J_A+1)` and applies the equal-p/n factor four only
in the physical output columns. It does not compute finite-t nucleon form
factors, the three-body `L_munu^(3) W_A^(munu)` contraction, halo folding,
or detector predictions.

Before running, confirm that the Windows runner is Online/Idle, that
`wolframscript -version` works under its runner account, and that the
`LZ_RESEARCH_READ_TOKEN` repository secret has Contents: Read-only access
to `lm846802297-bit/LZ-3-body-for-hdm`. The workflow does not write back to
the research repository.

The current native run is 44222502400, attempt 2, and remains queued.
It pins the LZ research checkout to
`dcefbeb7ad705b6c03f2b4689200eb1d9252e67b`.

**Do not conflate this native run with the separate provisional Python
artifact used by the PR #58 fixed-speed diagnostic.** That artifact is from
LZ workflow run 44222087512 and records source commit
`7cb5867fd40ce59194ba9a03252196f1fb38ccdd`, not the native run's
`dcefbeb...` source commit. The two commits are different. We checked the
calculation-critical Git blobs at both commits and confirmed they are
identical:

| Calculation input | Git blob SHA (same at both commits) |
|---|---|
| Seven-isotope/eight-response Python generator | `592334e17731278a397cc7a09bd4af7bfe00094b` |
| Native Wolfram exporter | `efffa3876fc61adfc3c80ea8e5f81296623a18ad` |
| Native comparison script | `9b858cafff6d33dcce95637d914e2b77ab17d2d6` |
| AFH v6.0 package source | `91b43b1a4596bebd6073ecd44c0c9464e5dd0c63` |
| AFH Xe-131 base implementation | `5280eb8844fcb562e783a9f74860aedd4a7f7a4e` |

All seven default-density CSV inputs (Xe-128, 129, 130, 131, 132, 134,
136) also have identical Git blob SHAs at the two commits. The native
workflow independently regenerates its Python reference database from the
pinned `dcefbeb...` checkout before comparing the native Wolfram outputs;
it does not consume the separate provisional artifact. Thus the native
comparison is self-contained and source-pinned, while the PR #58 artifact
remains a provisional fixed-speed diagnostic.

A queued run is not a calculation result and must not be reported as a
native AFH PASS.

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
