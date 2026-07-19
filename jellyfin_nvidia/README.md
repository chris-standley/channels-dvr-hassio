# Jellyfin (NVIDIA) — Home Assistant App

Run [Jellyfin](https://jellyfin.org/) as a Home Assistant app, wrapping the
**official** `ghcr.io/jellyfin/jellyfin` image, configured for **NVIDIA NVENC**
hardware transcoding.

For Intel or AMD GPUs, use the **Jellyfin (Intel / AMD)** app instead.

- **Web UI / API:** port `8096` (direct LAN), plus **ingress** for authenticated
  remote access through Home Assistant
- **Config/database:** the app's own config directory (backed up)
- **Media library:** the Home Assistant `/media` share
- **Hardware transcoding:** NVIDIA NVENC via `/dev/nvidia*` devices

> ⚠️ **NVIDIA in Home Assistant is not plug-and-play.** It requires a host with
> the NVIDIA driver installed and GPU device access — generally a **Home
> Assistant Supervised** install on a Linux host. **Home Assistant OS (HAOS)
> does not ship NVIDIA drivers**, so NVENC will not work there. Read
> [DOCS.md](DOCS.md) before installing.

See [DOCS.md](DOCS.md) for full requirements, installation, and troubleshooting.

> **Unofficial:** this is a community app and is **not affiliated with,
> endorsed by, or supported by the Jellyfin project**. It wraps the official,
> unmodified `jellyfin/jellyfin` image. "Jellyfin" and related marks belong to
> their respective owners. See [../DISCLAIMER.md](../DISCLAIMER.md).
