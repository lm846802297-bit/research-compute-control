# Repository instructions — research runner control

These instructions are mandatory for agents and contributors. Read [README.md](README.md) before changing workflows, runner configuration, or research artifact handoff. Follow the canonical [RESEARCH_COMPUTE_POLICY.md](RESEARCH_COMPUTE_POLICY.md) in the destination research repository: ordinary GPT chat uses ChatGPT Linux; Codex/Work may use the user's Windows local environment or a GPT-managed cloud computer. This repository's Windows self-hosted Actions runner remains a separate, explicitly authorized independent-verification path.

## Trusted-runner security

- Use this private control repository's self-hosted runner only for explicitly reviewed, trusted, manually dispatched jobs.
- Never run untrusted pull-request code, arbitrary user-supplied shell commands, or unreviewed refs on the persistent host.
- Pin reusable workflow and research source checkouts to reviewed full commit SHAs. Use least-privilege credentials and do not persist checkout credentials.
- Do not enable automatic cross-repository publishing without an explicitly approved design, scoped credentials, and a review/approval gate.

## Research-result artifact handoff

- A ZIP/TAR from a local runner, GitHub Actions artifact, ChatGPT handoff, collaborator, or other delivery source is a transfer bundle. **Extract it before upload** and commit the required native files individually to the destination research repository, preserving paths and filenames.
- Do not commit a newly generated delivery ZIP/TAR as the only or substitute archive of extracted source, inputs, outputs, reports, metadata, or manifests.
- Retain an original upstream archive only when required for provenance/reproducibility or when extracted content cannot legally be redistributed. Record source/version, byte size, SHA-256, extraction command, and per-file checksums; commit all extracted files that may be redistributed.
- Include run provenance and SHA-256 manifests. After publishing, retrieve the files from GitHub and verify paths, counts, byte sizes, and checksums.
- Keep archival status separate from numerical and scientific validation status. A successful upload, checksum, or workflow run is not by itself scientific validation.

## CI is optional; research work must not wait for CI

Continuous integration (CI) is optional supporting infrastructure. It is not a prerequisite, acceptance gate, or merge condition for research work.

- Do not require a CI run to start, continue, or complete a calculation; perform numerical or scientific validation; prepare or archive results; commit or push research files; open or update a pull request; mark a pull request ready for review; or merge a pull request.
- Start primary calculations according to the active mode: ChatGPT Linux in ordinary GPT chat, the user's Windows local environment in Codex/Work local execution, or a GPT-managed cloud computer in Codex/Work cloud execution. The Windows self-hosted Actions runner in this repository is a separate execution path for explicitly authorized independent verification; it is not the same as direct Codex/Work use of the Windows computer. Do not pause a feasible primary calculation solely because GitHub Actions is queued, unavailable, disabled, blocked by account/billing limits, or failing for infrastructure reasons.
- In this research architecture, GitHub Actions (including this self-hosted runner) is used for independent verification, regression testing, CI, and explicitly approved native cross-checks; it is not the default primary calculation engine. Record exact source commits, inputs, settings, environment, logs, and outputs for every such run.
- This policy does not waive scientific or numerical validation. Perform the checks appropriate to the claim—such as analytic identities, dimensional and unit checks, limiting cases, independent implementations, convergence, uncertainty estimates, provenance, and output-integrity checks—locally or on an approved runner, and record exactly what was performed. CI neither substitutes for these checks nor adds a mandatory gate.
- If CI is run, report its exact commit SHA and actual result. If it is not run, queued, blocked, cancelled, or failed, record that status truthfully and distinguish infrastructure failure from a scientific or numerical failure. Never describe an unrun check as passed.
- Base review and merge decisions on the actual diff, reproducibility, scientific evidence, scope, and the checks that are genuinely relevant to the change—not on whether a CI badge is green. Do not configure CI status checks as required branch-protection gates under this repository policy. Do not change repository protection settings without explicit authorization.
- CI becomes mandatory for a particular task only when the author explicitly requests it for that task.

