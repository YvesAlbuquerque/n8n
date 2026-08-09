# Loomlight n8n Fork Policy

This repository is a fork of [`n8n-io/n8n`](https://github.com/n8n-io/n8n). The fork exists to support controlled internal Loomlight and Polite Goblin customizations while preserving a reviewable path back to upstream.

## Branch roles

- `master` is the upstream mirror. It must remain free of Loomlight-specific commits and should only move by fast-forward to `n8n-io/n8n:master` after comparison.
- `loomlight/main` is the persistent integration branch for reviewed Loomlight-specific changes.
- Short-lived feature branches start from the current `loomlight/main` and merge back through pull requests.

Do not implement Loomlight changes directly on `master` or `loomlight/main`.

## Upstream synchronization

1. Read the current SHAs of `YvesAlbuquerque/n8n:master` and `n8n-io/n8n:master`.
2. Compare the commits and confirm the fork branch has no fork-only divergence.
3. Fast-forward fork `master` to the verified upstream SHA without force.
4. Open a pull request from `master` into `loomlight/main`.
5. Review conflicts, migrations, Docker changes, runner compatibility, and relevant release notes before merge.
6. Build and validate the fork image before changing any infrastructure image reference.

If `master` contains fork-only commits or cannot fast-forward, stop and investigate. Do not force-update it to conceal divergence.

## Fork-specific release workflow

`.github/workflows/release-loomlight-image.yml` is intentionally documented here rather than in the upstream-owned `.github/WORKFLOWS.md`. Keeping fork-only automation documentation isolated reduces recurring conflicts when synchronizing `master` from upstream.

The workflow:

- runs validation on every pull request targeting `loomlight/main`, without path filtering, because repository actions, scripts, patches, packages, and Docker context can all affect the image;
- gives pull-request validation only `contents: read` permission;
- builds the application and the existing upstream n8n Dockerfile for `linux/amd64`;
- verifies the built container with an explicit `n8n --version` smoke test;
- publishes only on pushes to `loomlight/main` or explicit manual dispatch;
- grants `packages: write` only to the publish job;
- smoke-tests the immutable image after publication.

The workflow publishes to:

```text
ghcr.io/yvesalbuquerque/n8n
```

Published image tags include:

- an immutable tag containing the source n8n version and commit SHA;
- `loomlight-main` as a convenience pointer for evaluation only.

Runtime infrastructure must pin an immutable tag or digest. It must not deploy `loomlight-main` directly.

## Separation of responsibilities

- This repository owns n8n source customization and image production.
- `loomlight-infra` owns runtime image selection, deployment, persistence, backup, rollback, monitoring, and operational evidence.
- Publishing an image does not deploy it.
- Merging source changes does not prove runtime compatibility.

## Validation gates

Before an image becomes a runtime candidate:

- the source build completes;
- the Docker image builds for the target architecture;
- the container reports the expected n8n version;
- the image is identified by immutable tag or digest;
- server and external runner versions are checked for compatibility;
- database migration impact is reviewed;
- `loomlight-infra` defines backup, rollback, canary, health, workflow, JavaScript runner, and Python runner validation.

Do not claim the fork image is deployed, restore-validated, or production-ready based only on a successful build.
