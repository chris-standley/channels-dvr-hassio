# Channels DVR (NVIDIA) — Home Assistant App

Run [Channels DVR Server](https://getchannels.com/dvr-server/) as a Home
Assistant app, wrapping the **official** `fancybits/channels-dvr:nvidia`
image for **NVIDIA NVENC** hardware transcoding.

For Intel or AMD GPUs, use the **Channels DVR (Intel / AMD)** app instead.

- **Web UI / API:** port `8089`
- **Networking:** host network (required for tuner discovery + DLNA)
- **Config/database:** the app's own config directory (backed up)
- **Recordings:** the Home Assistant `/media` share
- **Hardware transcoding:** NVIDIA NVENC via `/dev/nvidia*` devices

> ⚠️ **NVIDIA in Home Assistant is not plug-and-play.** It requires a host with
> the NVIDIA driver installed and GPU device access. This generally means a
> **Home Assistant Supervised** install on a Linux host — **Home Assistant OS
> (HAOS) does not ship NVIDIA drivers**, so NVENC will not work there. Read
> [DOCS.md](DOCS.md) before installing.

See [DOCS.md](DOCS.md) for full requirements, installation, and troubleshooting.

> **Unofficial:** this is a community app and is **not affiliated with,
> endorsed by, or supported by Fancy Bits / Channels**. It wraps the official,
> unmodified `fancybits/channels-dvr:nvidia` image. "Channels" and related marks
> belong to their respective owners. See [../DISCLAIMER.md](../DISCLAIMER.md).
