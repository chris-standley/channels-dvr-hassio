# Chris's Home Assistant Add-ons

A Home Assistant add-on repository for running [Channels DVR
Server](https://getchannels.com/dvr-server/) from the **official**
`fancybits/channels-dvr` Docker image.

> ## ⚠️ Unofficial — not affiliated with Fancy Bits / Channels
>
> **This is an unofficial, community project. It is NOT affiliated with,
> endorsed by, or supported by Fancy Bits, LLC or the Channels team.** It does
> not modify or redistribute the Channels software — it only **wraps the
> official, unmodified `fancybits/channels-dvr` Docker image** so Home Assistant
> can manage it. "Channels" and related marks belong to their respective owners.
> For Channels support, use <https://community.getchannels.com>. See
> [DISCLAIMER.md](DISCLAIMER.md) for the full statement.

## Add-ons

Pick the one that matches your GPU — install **only one** (they both bind host
port 8089 and would conflict):

### [Channels DVR (Intel / AMD)](./channels_dvr)

Hardware transcoding via **VA-API** — Intel Quick Sync or AMD. This is the right
choice for most people (any x86 host with an Intel or AMD GPU). Also runs fine
with the GPU device removed for software-only transcoding.

### [Channels DVR (NVIDIA)](./channels_dvr_nvidia)

Hardware transcoding via **NVIDIA NVENC** (`fancybits/channels-dvr:nvidia`).
Requires a host with NVIDIA drivers and GPU device access — generally a **Home
Assistant Supervised** install, **not** Home Assistant OS. See that add-on's
docs before installing.

> **Which is which?** *Quick Sync* is Intel-only; AMD uses its VCN encoder.
> On Linux both are reached through the same `/dev/dri` render node, so the
> Intel/AMD add-on covers both. NVIDIA (NVENC) needs the separate add-on.

## Installing this repository

> The repository must be **public** on GitHub — Home Assistant clones it
> anonymously. (Private repos aren't supported via the Store UI; use the local
> add-on method below instead.)

1. In Home Assistant: **Settings → Add-ons → Add-on Store**.
2. Open the ⋮ menu (top-right) → **Repositories**.
3. Add this repository's URL:

   ```
   https://github.com/chris-standley/channels-dvr-hassio
   ```

4. The add-ons above will appear in the store, ready to install.

[![Open your Home Assistant instance and show the add add-on repository dialog with a specific repository URL pre-filled.](https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg)](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Fchris-standley%2Fchannels-dvr-hassio)

### Local install (no public GitHub repo needed)

Copy the add-on folder (`channels_dvr/` or `channels_dvr_nvidia/`) into the
`/addons` directory on your Home Assistant host (via the Samba or SSH add-on).
It will appear under **Local add-ons** in the store.

## How updates work

There are **two independent** update tracks — this is the key thing to
understand:

1. **The Channels DVR server updates itself.** The `fancybits/channels-dvr`
   image does **not** contain the server binary; it downloads the latest server
   from Fancy Bits on first start and then keeps itself up to date in place
   (into the config directory). So the DVR application stays current on its own —
   you don't rebuild the add-on for normal Channels updates. You can also trigger
   updates from the Channels web UI.

2. **The Docker base image** (`fancybits/channels-dvr:latest`) provides the
   runtime around the server — ffmpeg, OS libraries, GPU userspace. It changes
   infrequently. To pull a newer base image:
   - Bump `version` in the add-on's `config.yaml`, commit, and push. Home
     Assistant will show an **Update** button, and updating rebuilds from the
     newer `latest`.
   - Or use **Rebuild** on the add-on's Info page (⋮ menu) to force a fresh
     build without a version change.

   Because the `Dockerfile` builds `FROM ...:latest`, two rebuilds at different
   times can produce different base images even at the same add-on `version`. If
   you want fully reproducible builds, pin a specific tag or digest in the
   `Dockerfile` (e.g. `FROM fancybits/channels-dvr@sha256:...`) and bump it
   deliberately.

> Upstream has a known, occasional self-update hiccup
> (`remove /channels-dvr/latest: directory not empty`). That's a Channels/image
> issue, not this add-on; restarting the add-on generally clears it.

## Recommended two-drive storage layout

This repo is designed for a Home Assistant host with **two drives**:

| Drive | Holds | How it's used |
| ----- | ----- | ------------- |
| **Drive 1 — data disk** | HA config, HA database, add-on configs, **Channels' own database** | The default HAOS data disk (`/mnt/data`). Small, fast, included in backups. |
| **Drive 2 — media disk** | **Channels recordings** (and any other bulk media you store here) | HAOS `/media` folder, moved to the second disk. Bulk storage. |

How the add-on splits across those drives:

- **Channels config + database →** the add-on's `addon_config` directory, which
  lives on the **data disk (drive 1)**. It's small and gets backed up. ✔
- **Channels recordings →** the Home Assistant **`/media`** share, mounted into
  the container at `/shares/DVR`. Point `/media` at **drive 2** and recordings
  land there. ✔

### Setting it up in Home Assistant OS

1. Attach the second drive to the host.
2. Go to **Settings → System → Storage**. HAOS (10.2+ / Core 2023.6+) lets you
   move the **media** folder onto the second disk. Select the new drive for
   media. (This moves only `/media` — your config/DB/add-ons stay on the data
   disk, which is exactly what you want.)
3. Install this add-on. Its recordings automatically use `/media` → now on
   drive 2. Channels' database stays on drive 1 via `addon_config`.

### Sharing drive 2 with other media apps

Any other add-on that mounts Home Assistant's `/media` shares the **same
drive 2**, so recordings sit alongside your other media:

- Channels writes recordings under `/media` (it sees them as `/shares/DVR`, so
  on disk they appear as `/media/TV`, `/media/Movies`, etc.).
- Other media apps can read those folders (e.g. `/media/TV`, `/media/Movies`)
  alongside any of your own media stored elsewhere under `/media`.

> **Want Channels recordings in a dedicated subfolder** (e.g. `/media/DVR`)
> instead of at the media root? In the Channels web UI, go to **Manage Storage**
> and add/point the DVR storage path at a subfolder. The add-on mounts the whole
> `/media` share, so any subfolder under it is available to Channels.

## Repository layout

```
.
├── repository.yaml          # Repository metadata (name, url, maintainer)
├── README.md                # This file
├── DISCLAIMER.md            # Full non-affiliation / trademark statement
├── channels_dvr/            # Intel / AMD (VA-API) add-on
│   ├── config.yaml          # Add-on configuration / manifest
│   ├── Dockerfile           # Wraps fancybits/channels-dvr:latest
│   ├── README.md            # Store summary
│   └── DOCS.md              # Full documentation (Documentation tab)
└── channels_dvr_nvidia/     # NVIDIA (NVENC) add-on
    ├── config.yaml          # Wraps fancybits/channels-dvr:nvidia
    ├── Dockerfile
    ├── README.md
    └── DOCS.md
```

### Optional assets

Home Assistant shows default artwork if none is provided. To brand an add-on,
drop these PNGs into its folder:

- `icon.png` — 128×128 (square) add-on icon
- `logo.png` — ~250×100 store logo

## Disclaimer

Unofficial community project; not affiliated with or endorsed by Fancy Bits /
Channels, or Home Assistant / Open Home Foundation. See
[DISCLAIMER.md](DISCLAIMER.md).
