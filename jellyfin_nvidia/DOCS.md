# Jellyfin (NVIDIA)

This app runs [Jellyfin](https://jellyfin.org/) using the official
`ghcr.io/jellyfin/jellyfin` image, configured for **NVIDIA NVENC** hardware
transcoding.

Everything about Jellyfin is identical to the **Jellyfin (Intel / AMD)** app —
the only difference is the transcoding backend and the host requirements below.

## ⚠️ Read this first: NVIDIA host requirements

NVIDIA GPU passthrough to a Home Assistant app is **not** officially supported
by Home Assistant and is **not** plug-and-play. For NVENC to work you need
**all** of the following:

1. **A host with an NVIDIA GPU and the NVIDIA driver installed.** The kernel
   driver lives on the host, not in the container. Verify with `nvidia-smi`.
2. **A Home Assistant *Supervised* install** (or a self-managed Supervisor) on a
   general-purpose Linux OS. **Home Assistant OS (HAOS) does not include NVIDIA
   drivers**, so this app cannot do NVENC on a stock HAOS device.
3. **The `/dev/nvidia*` device nodes present on the host** (created by the
   driver). This app maps them in (see below). If your host uses the
   `nvidia-container-toolkit`, the `NVIDIA_VISIBLE_DEVICES` env var set in the
   Dockerfile also applies.

If you don't meet these requirements, install the **Jellyfin (Intel / AMD)** app
instead, or run without hardware transcoding.

## Device mapping

`config.yaml` maps these NVIDIA device nodes into the container:

```yaml
devices:
  - /dev/nvidia0
  - /dev/nvidiactl
  - /dev/nvidia-uvm
  - /dev/nvidia-uvm-tools
  - /dev/nvidia-modeset
```

- If a listed device **does not exist** on your host, the app will fail to
  start. Remove nodes you don't have, or add extras (e.g. `/dev/nvidia1`).
- Confirm which nodes exist with `ls -l /dev/nvidia*` on the host.

## Installation & access

1. Add this repository (**Settings → Apps → App Store → ⋮ → Repositories**):

   ```
   https://github.com/chris-standley/home-assistant-addons
   ```

2. Install **Jellyfin (NVIDIA)** and start it.
3. **On your LAN:** `http://<your-ha-ip>:8096`. **Remotely:** use **Open Web UI**
   (ingress) — authenticated through Home Assistant, no exposed port.

> Do not run this app and the **Jellyfin (Intel / AMD)** app at the same time —
> they would both try to bind port 8096.

## Storage

Same layout as the Intel/AMD app:

| Purpose                      | Container path | Home Assistant location                   |
| ---------------------------- | -------------- | ----------------------------------------- |
| Config / database / metadata | `/config`      | The app's own config dir (`addon_config`) |
| Media library                | `/media`       | The `/media` share                        |

> This app uses a **separate** config directory from the Intel/AMD app (its slug
> differs). To migrate, copy the old `/config` contents into this app's config
> dir so you keep users, settings, and metadata.

## Enabling NVENC in Jellyfin

1. **Dashboard → Playback → Transcoding**.
2. Set **Hardware acceleration** to **Nvidia NVENC**.
3. Enable the codecs your GPU supports (H.264/HEVC, and AV1 on newer cards).

## Verifying it works

- **Dashboard → Playback** shows the active hardware acceleration.
- On the host, run `nvidia-smi` while a stream is transcoding — you should see
  the Jellyfin/ffmpeg process using the GPU encoder.

## Troubleshooting

- **Won't start / device error:** a mapped `/dev/nvidia*` node doesn't exist —
  run `ls -l /dev/nvidia*` and adjust the `devices:` list.
- **Falls back to software transcoding:** the container can't reach the driver.
  Confirm `nvidia-smi` works on the host and that the driver version is
  compatible with the image's ffmpeg; consider the `nvidia-container-toolkit`
  runtime.
- **HAOS user:** NVENC is not available on Home Assistant OS — use the Intel/AMD
  app or a Supervised install.

## Support

- Jellyfin docs: https://jellyfin.org/docs/
- Jellyfin community: https://forum.jellyfin.org/
- Official image: https://github.com/jellyfin/jellyfin/pkgs/container/jellyfin
