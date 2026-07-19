# Changelog

## 1.0.2

- Pin the base image by digest for reproducible builds, with automated
  base-image updates via CI (a new version is published when the upstream
  image changes).
- Fix the Web UI link so Home Assistant Supervisor accepts the configuration.

## 1.0.0

- Initial release. Channels DVR Server with Intel Quick Sync / AMD VA-API
  hardware transcoding, wrapping the official `fancybits/channels-dvr` image.
