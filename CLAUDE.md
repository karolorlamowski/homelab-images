# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This repo holds custom Docker images for homelab use. Each image lives in its own subdirectory alongside a dedicated GitHub Actions workflow.

## Structure

```
frigate/
  Dockerfile                  # FROM official Frigate + HailoRT version swap
.github/workflows/
  frigate-build.yml           # Triggers on push to main; skips if tag exists in GHCR
```

## Frigate image

The official Frigate image ships HailoRT 4.21.0, which is incompatible with Talos 12.6. This build upgrades HailoRT to 4.23.0 by layering on top of the official image — no full source rebuild.

Key variables in `frigate/Dockerfile`:
- `FRIGATE_VERSION` — pinned to the last built upstream release (e.g. `0.17.1`); CI overrides this with the detected latest release at build time
- `HAILO_VERSION` — the desired HailoRT version; managed here in the Dockerfile, not in the workflow

## CI behaviour

- Trigger: push to `main`, or manual `workflow_dispatch` with optional `force_build`
- Skips build if `ghcr.io/<owner>/frigate:<version>` manifest already exists in GHCR (HTTP 200 check)
- Pushes two tags: `<version>` and `latest`
- Uses registry-based layer cache (`buildcache` tag)
- Requires **Read and write permissions** under Settings → Actions → General → Workflow permissions

## Adding a new image

1. Create a subdirectory (e.g. `home-assistant/`) with a `Dockerfile`
2. Add a corresponding workflow in `.github/workflows/` following the same pattern as `frigate-build.yml`
