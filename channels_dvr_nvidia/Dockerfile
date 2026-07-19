# Wrap the official Channels DVR NVIDIA image as a Home Assistant add-on.
#
# fancybits/channels-dvr:nvidia bundles the NVENC/CUDA userspace libraries.
# The matching NVIDIA kernel driver and /dev/nvidia* devices must be provided
# by the host (see DOCS.md).
#
# Since Supervisor 2026.04.0 the BUILD_FROM arg is no longer injected, so we
# declare our own default here.
ARG BUILD_FROM=fancybits/channels-dvr:nvidia
FROM ${BUILD_FROM}

# The upstream image already defines ENTRYPOINT/CMD that launch the DVR server,
# so we do not override them — we only attach add-on metadata.

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
  org.opencontainers.image.source="https://github.com/chris-standley/channels-dvr-hassio"
