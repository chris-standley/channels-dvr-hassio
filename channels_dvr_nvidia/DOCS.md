# Channels DVR (NVIDIA)

This add-on runs [Channels DVR Server](https://getchannels.com/dvr-server/) using
the official `fancybits/channels-dvr:nvidia` image, which supports **NVIDIA
NVENC** hardware transcoding.

Everything about DVR functionality is identical to the **Channels DVR
(Intel / AMD)** add-on — the only difference is the transcoding backend and the
host requirements below.

## ⚠️ Read this first: NVIDIA host requirements

NVIDIA GPU passthrough to a Home Assistant add-on is **not** officially
supported by Home Assistant and is **not** plug-and-play. For NVENC to work you
need **all** of the following:

1. **A host with an NVIDIA GPU and the NVIDIA driver installed.** The kernel
   driver lives on the host, not in the container. Verify with `nvidia-smi`.
2. **A Home Assistant *Supervised* install** (or a self-managed Supervisor) on a
   general-purpose Linux OS. **Home Assistant OS (HAOS) does not include NVIDIA
   drivers**, so this add-on cannot do NVENC on a stock HAOS device (Green,
   Yellow, ODROID, generic-x86-64 HAOS image, etc.).
3. **The `/dev/nvidia*` device nodes present on the host.** These are created by
   the driver. This add-on maps them in (see below). If your setup uses the
   `nvidia-container-toolkit`, its `NVIDIA_VISIBLE_DEVICES` env var (set in the
   Dockerfile) also applies.

If you don't meet these requirements, install the **Channels DVR (Intel / AMD)**
add-on instead, or run without hardware transcoding.

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

- If a listed device **does not exist** on your host, the add-on will fail to
  start. Remove nodes you don't have, or add extras (e.g. `/dev/nvidia1` for a
  second GPU).
- Confirm which nodes exist with: `ls -l /dev/nvidia*` on the host.

## Installation

1. Add this repository to Home Assistant (**Settings → Add-ons → Add-on Store →
   ⋮ → Repositories**):

   ```
   https://github.com/chris-standley/channels-dvr-hassio
   ```

2. Install **Channels DVR (NVIDIA)**. The first build pulls the upstream NVIDIA
   image and can take several minutes.
3. Start the add-on and open the web UI at `http://<your-ha-ip>:8089`.

## First-time setup

Open the web UI on port **8089** and follow the Channels onboarding — add your
HDHomeRun tuners or TV Everywhere logins. Then, under **Settings** in the
Channels UI, confirm that hardware transcoding is set to **NVIDIA**.

## Storage

| Purpose            | Container path  | Home Assistant location                      |
| ------------------ | --------------- | -------------------------------------------- |
| Config + database  | `/channels-dvr` | The add-on's own config dir (`addon_config`) |
| Recordings         | `/shares/DVR`   | The `/media` share                           |

> **Note:** this add-on uses a **separate** config directory from the Intel/AMD
> add-on (its slug differs). If you're migrating from the Intel/AMD add-on, copy
> the contents of the old add-on's config dir into this one, or point both at
> the same shared folder, so you keep your DVR database and settings.

## Networking

Runs with **host networking** for HDHomeRun discovery (SSDP/Bonjour) and DLNA,
binding to the host's port **8089**. Do not run this add-on and the Intel/AMD
add-on at the same time — they would both try to bind port 8089.

## Verifying NVENC works

- Check the add-on **Log** tab after starting a transcode; Channels logs the
  transcoder it selects.
- On the host, run `nvidia-smi` while a stream is transcoding — you should see
  the `channels-dvr` process using the GPU's encoder.

## Updating

Same two-track model as the Intel/AMD add-on: the **Channels server updates
itself** at runtime (no rebuild needed for normal Channels updates), while the
**NVIDIA base image** (`fancybits/channels-dvr:nvidia`) is refreshed by bumping
`version` in `config.yaml` and updating, or using **Rebuild** (⋮ menu). If you
also update the host NVIDIA driver, rebuild so the image's CUDA/NVENC libraries
stay compatible.

## Troubleshooting

- **Add-on won't start / device error:** a mapped `/dev/nvidia*` node doesn't
  exist. Run `ls -l /dev/nvidia*` on the host and adjust the `devices:` list.
- **Starts but falls back to software transcoding:** the container can't reach
  the driver. Confirm `nvidia-smi` works on the host and that the host driver
  version matches what the CUDA/NVENC libraries in the image expect; consider
  using the `nvidia-container-toolkit` runtime on the host.
- **HAOS user:** NVENC is not available on Home Assistant OS. Use the Intel/AMD
  add-on or a Supervised install.

## Support

- Channels DVR docs & forum: https://getchannels.com/docs/ and
  https://community.getchannels.com/
- Official Docker image: https://hub.docker.com/r/fancybits/channels-dvr
