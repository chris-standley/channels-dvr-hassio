# Channels DVR (Intel / AMD) — Home Assistant Add-on

Run [Channels DVR Server](https://getchannels.com/dvr-server/) as a Home
Assistant add-on, wrapping the **official** `fancybits/channels-dvr` Docker
image. This variant uses **VA-API** for Intel Quick Sync / AMD hardware
transcoding. For NVIDIA GPUs, use the **Channels DVR (NVIDIA)** add-on instead.

Record live TV from HDHomeRun tuners and TV Everywhere, with a full DVR,
program guide, and native apps on every screen — managed by Home Assistant
Supervisor.

- **Web UI / API:** port `8089`
- **Networking:** host network (required for tuner discovery + DLNA)
- **Config/database:** the add-on's own config directory (backed up)
- **Recordings:** the Home Assistant `/media` share
- **Hardware transcoding:** `/dev/dri` (VA-API — Intel Quick Sync / AMD) enabled by default

See [DOCS.md](DOCS.md) for full installation, storage, networking, and hardware
transcoding details.

> **Unofficial:** this is a community add-on and is **not affiliated with,
> endorsed by, or supported by Fancy Bits / Channels**. It wraps the official,
> unmodified `fancybits/channels-dvr` image. "Channels" and related marks belong
> to their respective owners. See [../DISCLAIMER.md](../DISCLAIMER.md).
