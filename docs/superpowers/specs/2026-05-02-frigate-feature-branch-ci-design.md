# Frigate Feature Branch CI — Design

## Goal

Enable testing of custom Frigate images built from feature branches on the homelab k8s cluster, without affecting production `main` builds. When a non-main branch is pushed, CI builds and pushes a branch-suffixed image tag automatically.

## Workflow Changes (`frigate-build.yml`)

### Trigger

```yaml
on:
  push:
    branches: ['**']
```

Triggers on all branches. Main branch retains existing behavior; all others get the feature branch path.

### New step: Resolve tag info

Added after checkout, before the GHCR check. Sanitizes the branch name into a Docker-safe slug and sets two outputs:

- `tag_suffix` — empty string for `main`, `-<slug>` for all other branches (e.g. `-fix-hailo-libs` for branch `fix/hailo-libs`)
- `is_main` — `true`/`false`

Sanitization rules: lowercase, replace `/` and `_` with `-`, strip any character that is not `[a-z0-9-]`.

### Skip logic

| Branch | Behavior |
|--------|----------|
| `main` | Check GHCR for `<version>`. Skip if exists and `force_build != true`. |
| feature | Always build (`skip=false`). No GHCR check. |

### Image tags pushed

| Branch | Tags pushed |
|--------|-------------|
| `main` | `<version>`, `latest` |
| feature | `<version>-<branch-slug>` only |

### Build cache

Both paths share the same `buildcache` tag in GHCR. Feature branch builds benefit from cache warmed by `main` builds.

## Testing flow

1. Create a feature branch (e.g. `fix/hailo-libs`)
2. Push — CI builds and pushes `ghcr.io/karolorlamowski/frigate:0.17.1-fix-hailo-libs`
3. Update the k8s HelmRelease/deployment to use the feature tag
4. Verify Hailo detection works
5. Merge to `main` — CI builds and pushes the final `0.17.1` + `latest` tags

## Files changed

- `.github/workflows/frigate-build.yml` — add branch trigger, tag resolution step, conditional skip logic, conditional tags
