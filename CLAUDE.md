# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This repo holds custom Docker images for homelab use. Each image lives in its own subdirectory alongside a dedicated GitHub Actions workflow.

## Structure

```
frigate/
  Dockerfile                  # FROM official Frigate + HailoRT version swap
.github/workflows/
  frigate-build.yml           # Matrix-builds every Frigate release >= MIN_FRIGATE_VERSION missing from GHCR
```

## Frigate image

The official Frigate image ships HailoRT 4.21.0, which is incompatible with Talos 12.6. This build upgrades HailoRT to 4.23.0 by layering on top of the official image — no full source rebuild.

Key variables in `frigate/Dockerfile`:
- `FRIGATE_VERSION` — default for local builds only; CI passes each version from the build matrix
- `HAILO_VERSION` — the desired HailoRT version; managed here in the Dockerfile, not in the workflow (CI reads it from here). Must match the hailo kernel driver on the Talos nodes — don't bump it to a newer HailoRT release unless the driver is bumped too
- Debian release and Python ABI (`cp311` etc.) are detected inside the upstream image, so artifacts stay correct if Frigate changes its base
- A second `RUN` asserts the swap worked (single `libhailort.so.*`, pip version, `import hailo_platform`)

## CI behaviour

- Triggers: push to any branch, daily `schedule`, manual `workflow_dispatch` (`force_build`, `versions`)
- `resolve` job lists stable upstream releases ≥ `MIN_FRIGATE_VERSION` (workflow env) whose upstream GHCR image exists, and emits a matrix of versions whose `<version>-hailo<HAILO_VERSION>` tag is missing in our GHCR
- `build` job builds each matrix entry (amd64 + arm64); main tags: `<version>-hailo<hv>`, `<version>`, `<major.minor>` (newest patch only), `latest` (newest release only)
- Feature branches: only the newest version, tagged `<version>-<branch-slug>`, always built
- Uses GitHub Actions cache, scoped per Frigate version
- Requires **Read and write permissions** under Settings → Actions → General → Workflow permissions

## Adding a new image

1. Create a subdirectory (e.g. `home-assistant/`) with a `Dockerfile`
2. Add a corresponding workflow in `.github/workflows/` following the same pattern as `frigate-build.yml`
