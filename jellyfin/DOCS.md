# Jellyfin (Intel / AMD)

[Jellyfin](https://jellyfin.org/) is a free software media system that lets you
collect, manage, and stream your movies, TV, music, and photos to apps on every
device — no subscriptions, no tracking.

This app wraps the **official** `ghcr.io/jellyfin/jellyfin` image, which bundles
`jellyfin-ffmpeg` with VA-API / QSV hardware acceleration.

## Installation

1. Add this repository to Home Assistant (**Settings → Apps → App Store →
   ⋮ → Repositories**):

   ```
   https://github.com/chris-standley/home-assistant-addons
   ```

2. Install **Jellyfin (Intel / AMD)**. The first build pulls the upstream image
   and can take a few minutes.
3. Start the app, then open it — see **Access** below.

## Access (direct + remote)

- **On your LAN:** browse to `http://<your-ha-ip>:8096`.
- **Remotely / securely:** this app enables **ingress**, so you can open Jellyfin
  from the app's page (**Open Web UI**) or pin it to the sidebar. Ingress serves
  it through Home Assistant's authenticated, TLS-terminated tunnel — so with your
  existing HA remote access (e.g. Nabu Casa Cloud) you reach Jellyfin from
  anywhere **without exposing port 8096** to the internet.

Unlike some servers, Jellyfin supports being served under a subpath, so ingress
works. (If you set a **Base URL** in Jellyfin's networking settings, leave it
blank — ingress handles the path.)

## First-time setup

Open Jellyfin and complete the setup wizard: create your admin user, then add
your media libraries pointing at folders under **`/media`** (see **Storage**).

## Storage

| Purpose                     | Container path | Home Assistant location                      |
| --------------------------- | -------------- | -------------------------------------------- |
| Config / database / metadata | `/config`      | The app's own config dir (`addon_config`)    |
| Media library               | `/media`       | The `/media` share                           |

- **Config/database** holds your users, settings, watch state, and metadata. It
  lives in the app's dedicated config directory and is included in Home
  Assistant backups. Keep it reasonably small.
- **Media** is mounted from Home Assistant's `/media` share. Point your Jellyfin
  libraries at subfolders like `/media/Movies`, `/media/Shows`, `/media/Music`.

> **Transcode cache:** by default Jellyfin writes transcode temp files under
> `/config` (on the data disk). For heavy transcoding you can point the
> transcode path at a `/media` subfolder in **Dashboard → Playback**.

### Two-drive layout

If your Home Assistant host has two drives (see the repository README), this app
fits the same split as the Channels DVR apps:

- **Drive 1 (data disk):** Jellyfin's `/config` (database/metadata) — small,
  backed up.
- **Drive 2 (media disk):** your media library via `/media`. Because Channels
  DVR also writes recordings to `/media`, both apps **share the same media
  disk** — you can add a Jellyfin library pointing at the Channels recordings
  (e.g. `/media/TV`) if you want them in Jellyfin too.

## Hardware transcoding

The app maps `/dev/dri` into the container for **VA-API** hardware transcoding
(Intel Quick Sync or AMD). To enable it:

1. In Jellyfin: **Dashboard → Playback → Transcoding**.
2. Set **Hardware acceleration** to **Video Acceleration API (VA-API)**.
3. Set the VA-API device to `/dev/dri/renderD128` (the default render node).
4. Enable the codecs your GPU supports (H.264/HEVC decode + encode).

On hosts **without** a `/dev/dri` render node (most ARM boards, including Home
Assistant Green/Yellow), remove the `devices:` block from `jellyfin/config.yaml`
or the app will fail to start; Jellyfin will still work with software
transcoding. For **NVIDIA** (NVENC), use the **Jellyfin (NVIDIA)** app.

## Updating

The base image is **pinned by digest** and updated automatically: a scheduled
GitHub Action re-pins the digest and bumps the app `version` when the upstream
image changes, so Home Assistant surfaces an **Update** on its own. Unlike some
servers, Jellyfin does **not** self-update — the image update *is* the server
update.

## Support

- Jellyfin docs: https://jellyfin.org/docs/
- Jellyfin community: https://forum.jellyfin.org/
- Official image: https://github.com/jellyfin/jellyfin/pkgs/container/jellyfin
