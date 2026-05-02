ARG FRIGATE_VERSION=0.17.1

FROM ghcr.io/blakeblackshear/frigate:${FRIGATE_VERSION}

ARG HAILO_VERSION=4.23.0
ARG TARGETARCH

RUN set -eux; \
    case "${TARGETARCH}" in \
        amd64) arch="x86_64" ;; \
        arm64) arch="aarch64" ;; \
        *) echo "Unsupported arch: ${TARGETARCH}" >&2 && exit 1 ;; \
    esac; \
    pip uninstall hailort -y 2>/dev/null || true; \
    wget -qO- "https://github.com/frigate-nvr/hailort/releases/download/v${HAILO_VERSION}/hailort-debian12-${TARGETARCH}.tar.gz" \
        | tar -C / -xzf -; \
    pip install --no-cache-dir \
        "https://github.com/frigate-nvr/hailort/releases/download/v${HAILO_VERSION}/hailort-${HAILO_VERSION}-cp311-cp311-linux_${arch}.whl"
