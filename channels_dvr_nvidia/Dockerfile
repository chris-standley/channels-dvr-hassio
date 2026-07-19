# Wrap the official Channels DVR NVIDIA image as a Home Assistant app.
#
# Tracks the fancybits/channels-dvr:nvidia tag, pinned by digest for
# reproducible builds. It bundles the NVENC/CUDA userspace libraries; the
# matching NVIDIA kernel driver and /dev/nvidia* devices must be provided by
# the host (see DOCS.md).
#
# The digest below and the app `version` are updated automatically by
# .github/workflows/update-base-image.yml when the upstream tag changes.
#
# Since Supervisor 2026.04.0 the BUILD_FROM arg is no longer injected, so we
# declare our own default here.
ARG BUILD_FROM=fancybits/channels-dvr@sha256:fd0f0fd21d70ec47f5a9237177ad9624b221b855f971d042f031daae20ccae88
FROM ${BUILD_FROM}

# The upstream image already defines ENTRYPOINT/CMD that launch the DVR server,
# so we do not override them — we only attach app metadata.

ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_DESCRIPTION="Channels DVR Server (NVIDIA) for Home Assistant"
ARG BUILD_NAME="Channels DVR (NVIDIA)"
ARG BUILD_REF
ARG BUILD_VERSION

# Hint the NVIDIA container runtime (if the host uses nvidia-container-toolkit)
# about the capabilities this container needs.
ENV NVIDIA_VISIBLE_DEVICES=all \
    NVIDIA_DRIVER_CAPABILITIES=compute,video,utility

LABEL \
  io.hass.name="${BUILD_NAME}" \
  io.hass.description="${BUILD_DESCRIPTION}" \
  io.hass.arch="${BUILD_ARCH}" \
  io.hass.type="addon" \
  io.hass.version="${BUILD_VERSION}" \
  maintainer="Chris Standley <chris@cstandley.com>" \
  org.opencontainers.image.source="https://github.com/chris-standley/home-assistant-apps"
