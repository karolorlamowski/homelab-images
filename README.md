# homelab-images

Custom Docker images for homelab use.

## Frigate

Based on the official [Frigate NVR](https://github.com/blakeblackshear/frigate) image. The official image ships with HailoRT 4.21.0, which is incompatible with Talos 12.6 — this build swaps in HailoRT 4.23.0 (must match the hailo driver on the nodes).

### Image

```
ghcr.io/karolorlamowski/frigate:latest                       # newest Frigate release
ghcr.io/karolorlamowski/frigate:<major.minor>                # newest patch of that minor, e.g. 0.18
ghcr.io/karolorlamowski/frigate:<frigate-version>            # e.g. 0.18.2
ghcr.io/karolorlamowski/frigate:<frigate-version>-hailo<hailort-version>   # e.g. 0.18.2-hailo4.23.0
```

### Build

Every stable Frigate release from `MIN_FRIGATE_VERSION` (set in the workflow) upward is built against the HailoRT version pinned in `frigate/Dockerfile`. The workflow runs daily, on every push and on manual dispatch, and only builds versions whose `<version>-hailo<hailort>` tag is missing from the registry — so new upstream releases are picked up automatically, and bumping `HAILO_VERSION` rebuilds all of them.

A new upstream release is built only once it is `MIN_RELEASE_AGE_DAYS` (3) days old, so a release that gets pulled or hotfixed within days never reaches GHCR. Renovate in homelab-infra relies on this: it cannot age-gate GHCR tags itself, so it takes our tags as soon as they appear.

Manual dispatch accepts `versions` (comma-separated list) and `force_build`.
