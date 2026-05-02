# homelab-images

Custom Docker images for homelab use.

## Frigate

Based on the official [Frigate NVR](https://github.com/blakeblackshear/frigate) image. The official image ships with HailoRT 4.21.0, which is incompatible with Talos 12.6 — this build upgrades HailoRT to 4.23.0.

### Image

```
ghcr.io/karolorlamowski/frigate:latest
ghcr.io/karolorlamowski/frigate:<frigate-version>
```

### Build

A new image is built automatically on every push to `main`, but only if the current latest Frigate release is not yet present in the registry.
