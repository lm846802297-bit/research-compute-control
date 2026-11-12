# Repository instructions — research runner control

These instructions are mandatory for agents and contributors. Read [README.md](README.md) before changing workflows, runner configuration, or research artifact handoff.

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
