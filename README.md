# Research runner control

Private control repository for trusted research jobs on a Windows self-hosted runner.

## Current status

Only the manual hardware diagnostic is configured. No research calculation, reusable workflow, result upload, or automatic push to research repositories is configured or validated yet.

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


## Research result artifact handoff — mandatory

Workflow artifacts in this control repository are temporary transfer products, not the permanent research archive. When a calculation artifact or result bundle is downloaded for publication to a research repository:

1. Verify the originating workflow run, source commit, artifact name, and run metadata.
2. Extract ZIP/TAR bundles before upload. Commit the required native files individually to the destination research repository, preserving the study's directory structure and filenames. This includes source code, tests, configuration, inputs, CSV/JSON/ROOT/HDF5/TXT outputs, reports, metadata, logs/checkpoints needed for audit, and manifests as applicable.
3. Do not commit a newly generated delivery ZIP/TAR as the sole or substitute copy of its extracted contents. The bundle is only a transport container.
4. Include a run manifest and SHA-256 manifest with provenance, code/input revisions, environment, command, numerical settings, output sizes/counts, and checksums.
5. After publishing, retrieve the files from the destination GitHub repository and verify paths, file counts, byte sizes, and SHA-256 checksums. Report archival completion separately from scientific validation.

An original upstream archive may be retained only when its exact container is needed for provenance/reproducibility or extracted inputs cannot legally be redistributed. Record its source/version, size, SHA-256, and extraction command, and publish all extracted files that can be redistributed. A tool-native compressed output may remain in its native format when it is itself the calculation product.

This handoff policy does not authorize automatic cross-repository publishing. Publishing still requires an explicitly reviewed workflow, narrowly scoped credentials, and a deliberate approval step; never expose tokens or run untrusted code on the persistent self-hosted runner.
