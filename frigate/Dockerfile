ARG FRIGATE_VERSION=0.18.2

FROM ghcr.io/blakeblackshear/frigate:${FRIGATE_VERSION}

# Must match the hailo kernel driver shipped by the Talos extension on the nodes.
ARG HAILO_VERSION=4.23.0
ARG TARGETARCH
ARG PIP_BREAK_SYSTEM_PACKAGES=1

# Debian release and Python ABI are detected from the upstream image, so the
# right HailoRT artifacts are picked if Frigate moves to a newer base.
RUN set -eux; \
    case "${TARGETARCH}" in \
        amd64) arch="x86_64" ;; \
        arm64) arch="aarch64" ;; \
        *) echo "Unsupported arch: ${TARGETARCH}" >&2 && exit 1 ;; \
    esac; \
    . /etc/os-release; \
    debian="debian${VERSION_ID}"; \
    py="$(python3 -c 'import sys; print(f"cp{sys.version_info.major}{sys.version_info.minor}")')"; \
    base_url="https://github.com/frigate-nvr/hailort/releases/download/v${HAILO_VERSION}"; \
    pip uninstall hailort -y 2>/dev/null || true; \
    rm -f /usr/local/lib/libhailort.so*; \
    wget -qO- "${base_url}/hailort-${debian}-${TARGETARCH}.tar.gz" \
        | tar -C / --strip-components=1 -xzf -; \
    ldconfig; \
    pip install --no-cache-dir "${base_url}/hailort-${HAILO_VERSION}-${py}-${py}-linux_${arch}.whl"

# Fail the build if the swap did not take effect.
RUN set -eux; \
    test -e "/usr/local/lib/libhailort.so.${HAILO_VERSION}"; \
    test "$(ls /usr/local/lib/libhailort.so.* | wc -l)" -eq 1; \
    test "$(pip show hailort | sed -n 's/^Version: //p')" = "${HAILO_VERSION}"; \
    python3 -c "import hailo_platform"

LABEL io.hailo.hailort.version="${HAILO_VERSION}"
