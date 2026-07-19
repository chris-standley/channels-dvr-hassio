# Channels DVR

[Channels DVR Server](https://getchannels.com/dvr-server/) records live TV from
HDHomeRun network tuners and TV Everywhere (TVE) logins, giving you a full DVR
with a program guide and native apps on Apple TV, Fire TV, Android, iOS, and the
web.

This add-on wraps the **official** `fancybits/channels-dvr` Docker image, so you
get the exact same server the Channels team ships — just managed by Home
Assistant Supervisor.

## Installation

1. In Home Assistant, go to **Settings → Add-ons → Add-on Store**.
2. Click the ⋮ menu (top-right) → **Repositories**, and add the URL of this
   repository (e.g. `https://github.com/chris-standley/channels-dvr-hassio`).
3. Find **Channels DVR** in the store and click **Install**. The first build
   pulls the upstream image and can take several minutes.
4. Start the add-on, then click **Open Web UI** (or browse to
   `http://<your-ha-ip>:8089`).

> Prefer a local install? Copy the `channels_dvr/` folder into
> `/addons/` (the `addon_configs`/local add-ons share) on your Home Assistant
> host and it will appear under **Local add-ons**.

## First-time setup

Open the web UI on port **8089** and follow the Channels onboarding:

- Add your **HDHomeRun** tuners (auto-discovered thanks to host networking) or
  configure **TV Everywhere** with your TV provider login.
- Choose your recordings location — leave it at the default so recordings land
  in the mapped `/shares/DVR` folder (see **Storage** below).

Sign-in and licensing are handled entirely inside the Channels web UI; this
add-on has no options to configure.

## Storage

| Purpose            | Container path  | Home Assistant location                     |
| ------------------ | --------------- | ------------------------------------------- |
| Config + database  | `/channels-dvr` | The add-on's own config dir (`addon_config`) |
| Recordings         | `/shares/DVR`   | The `/media` share                          |

- **Config/database** lives in the add-on's dedicated config directory and is
  included in Home Assistant backups. Keep this reasonably small.
- **Recordings** are written to Home Assistant's **media** share, so they also
  show up when you browse `Media` and survive add-on reinstalls. Recordings can
  be very large — make sure the underlying storage has room.

> **Excluding recordings from backups:** Home Assistant backs up an add-on's
> config directory, not the `media` share, so your (potentially huge)
> recordings are **not** swept into every backup by default. Good.

### Two-drive layout (recommended)

This split is ideal for a host with two drives:

- **Drive 1 (HA data disk):** HA config, database, add-on configs — including
  **Channels' own database** (via `addon_config`). Small, fast, backed up.
- **Drive 2 (media disk):** **Channels recordings** — via the `/media` share.
  Move `/media` to the second disk under **Settings → System → Storage**
  (HAOS 10.2+ / Core 2023.6+). This moves only `/media`; config/DB stay on
  drive 1. Any other add-on that mounts `/media` shares drive 2 too, so
  recordings sit alongside your other media (e.g. `/media/TV`, `/media/Movies`).
  See the repository README for the full walkthrough.

## Networking

The add-on runs with **host networking** (`host_network: true`). This is
required so Channels DVR can:

- Discover **HDHomeRun** tuners via SSDP/Bonjour on your LAN.
- Serve **DLNA** to TVs and other clients.

Because of host networking, the server binds directly to your Home Assistant
host's IP on port **8089**. That means the port cannot be remapped — if
something else already uses 8089 on the host, you'll need to free it up.

## Hardware transcoding

This add-on maps `/dev/dri` into the container to enable **VA-API** hardware
transcoding, which dramatically reduces CPU usage when remuxing or transcoding
streams. On Linux, VA-API covers both GPU vendors:

- **Intel** — Quick Sync Video (QSV)
- **AMD** — VCN encoder (via AMF)

Both are reached through the same `/dev/dri` render node, so this single add-on
handles Intel and AMD.

- On a typical **x86/amd64** Home Assistant host with an Intel or AMD GPU this
  works out of the box.
- On boards **without** a `/dev/dri` render node (most ARM devices, including
  Home Assistant **Green** and **Yellow**), the add-on will fail to start with a
  device error. In that case, edit `channels_dvr/config.yaml` and remove the
  `devices:` block:

  ```yaml
  devices:
    - /dev/dri
  ```

- For **NVIDIA** (NVENC) transcoding, install the separate **Channels DVR
  (NVIDIA)** add-on from this same repository. NVIDIA needs a different upstream
  image and driver setup — see that add-on's documentation.

## Time zone

Home Assistant Supervisor injects your configured time zone into the add-on
automatically, so scheduled recordings use your local time. Set your time zone
under **Settings → System → General** if it isn't already.

## Updating

There are two independent update tracks:

1. **The Channels DVR server updates itself.** The `fancybits/channels-dvr` image
   doesn't contain the server binary — it downloads the latest server from Fancy
   Bits on first start and keeps itself current in place (in the config
   directory). Normal Channels updates need **no** add-on rebuild; you can also
   update from the Channels web UI.
2. **The Docker base image** (ffmpeg, OS libraries, GPU userspace) changes
   rarely. To pull a newer base image, bump `version` in `config.yaml` and update
   from the add-on's **Info** page, or use **Rebuild** (⋮ menu) to force a fresh
   build. Because the image builds `FROM ...:latest`, pin a specific tag/digest
   in the `Dockerfile` if you need reproducible builds.

> Upstream occasionally logs `remove /channels-dvr/latest: directory not empty`
> during self-update — a Channels/image quirk, not this add-on. Restarting the
> add-on usually clears it.

## Troubleshooting

- **Tuners not found:** confirm the add-on is running with host networking and
  that the tuner is on the same subnet as Home Assistant. Check the **Log** tab.
- **Won't start after enabling `/dev/dri`:** your host has no render node —
  remove the `devices:` block as described above.
- **Port 8089 in use:** stop whatever else is bound to 8089 on the host; with
  host networking the port can't be remapped.
- **Where are my recordings?** In the `/media` share on your Home Assistant
  host, under the `DVR` folder.

## Support

- Channels DVR docs & forum: https://getchannels.com/docs/ and
  https://community.getchannels.com/
- Official Docker image: https://hub.docker.com/r/fancybits/channels-dvr
