# Research runner control

Private control repository for trusted research jobs on a Windows self-hosted runner.

## Current status

The manual Windows hardware/runtime diagnostic and a manually dispatched Xe-131 native Mathematica/Python full-cross-section workflow are configured. The calculation workflow is pinned to a reviewed LZ research commit and uploads an artifact to this control repository; it has not yet been executed or scientifically validated. No automatic publishing to research repositories is configured.

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

## Native Xe-131 Mathematica full-cross-section run

A manually dispatched Windows self-hosted workflow is provided at
[xe131-native-full-dsigma.yml](.github/workflows/xe131-native-full-dsigma.yml).
It checks out the reviewed LZ research revision
60443717f66e72f84fdb04232e7a74398a95916d, runs the native Wolfram Language
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
independent no-factor-four control at 2e-12.

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
