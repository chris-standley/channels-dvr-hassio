# Jellyfin (Intel / AMD) — Home Assistant App

Run [Jellyfin](https://jellyfin.org/) — the free software media system — as a
Home Assistant app, wrapping the **official** `ghcr.io/jellyfin/jellyfin` image.
This variant uses **VA-API** for Intel Quick Sync / AMD hardware transcoding.
For NVIDIA GPUs, use the **Jellyfin (NVIDIA)** app instead.

- **Web UI / API:** port `8096` (direct LAN), plus **ingress** for authenticated
  remote access through Home Assistant
- **Config/database:** the app's own config directory (backed up)
- **Media library:** the Home Assistant `/media` share
- **Hardware transcoding:** `/dev/dri` (VA-API — Intel Quick Sync / AMD)

See [DOCS.md](DOCS.md) for installation, ingress/remote access, storage, and
hardware transcoding details.

> **Unofficial:** this is a community app and is **not affiliated with,
> endorsed by, or supported by the Jellyfin project**. It wraps the official,
> unmodified `jellyfin/jellyfin` image. "Jellyfin" and related marks belong to
> their respective owners. See [../DISCLAIMER.md](../DISCLAIMER.md).
