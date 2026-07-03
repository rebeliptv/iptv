<div align="center">

# IPTV

**Self-hosted IPTV server for live TV and sports**

[![Docker Pulls](https://img.shields.io/docker/pulls/rebeliptv/iptv?style=for-the-badge&color=2496ED)](https://hub.docker.com/r/rebeliptv/iptv)
[![Docker Image Size](https://img.shields.io/docker/image-size/rebeliptv/iptv/latest?style=for-the-badge&color=1D63ED)](https://hub.docker.com/r/rebeliptv/iptv)
[![GitHub Release](https://img.shields.io/github/v/release/rebeliptv/iptv?style=for-the-badge&color=22C55E)](https://github.com/rebeliptv/iptv/releases/latest)
[![License](https://img.shields.io/badge/license-proprietary-8B5CF6?style=for-the-badge)](LICENSE)
[![Website](https://img.shields.io/badge/website-rebeliptv.net-FF6B35?style=for-the-badge)](https://www.rebeliptv.net)

Aggregates live TV channels and sports events into a single M3U playlist with full EPG guide data.<br>
Streams are fully proxied so upstream sources are never exposed to clients.<br>
Designed for **Jellyfin**, **Plex**, **Emby**, and other IPTV apps.

**🌐 New here? Follow the step-by-step [setup guide at rebeliptv.net](https://www.rebeliptv.net).**

</div>

---

## Screenshots

<details>
<summary><strong>Dashboard</strong> — system overview with live sports</summary>
<br>

![Dashboard](screenshots/dashboard.png)
</details>

<details>
<summary><strong>Channels</strong> — searchable channel list with categories</summary>
<br>

![Channels](screenshots/channels.png)
</details>

<details>
<summary><strong>Channel Detail</strong> — video player with program guide</summary>
<br>

![Channel Detail](screenshots/channel-detail.png)
</details>

<details>
<summary><strong>Guide</strong> — EPG program schedule</summary>
<br>

![Guide](screenshots/guide.png)
</details>

<details>
<summary><strong>Sports</strong> — live events sorted by start time</summary>
<br>

![Sports](screenshots/sports.png)
</details>

<details>
<summary><strong>Sport Event</strong> — live scores from ESPN</summary>
<br>

![Sport Event](screenshots/event-detail.png)
</details>

<details>
<summary><strong>Settings</strong> — API key management and server info</summary>
<br>

![Settings](screenshots/settings.png)
</details>

## Features

- **380+ TV channels** with logos and persistent channel numbers that never reshuffle
- **Live sports events** -- see [Supported Live Sports](#supported-live-sports) below
- **Custom M3U sources** -- add your own M3U URLs or upload files to merge into the playlist; edit the URL or replace the uploaded file later without losing channel numbers
- **Customize your lineup** -- enable/disable channels and drag to reorder them from the dashboard; disabled channels drop out of your playlist and guide, and both choices persist across restarts and upgrades
- **Channel categories** -- Sports, News, Local, Kids, Movies, Entertainment, Lifestyle, Documentary, and more
- **Local station detection** -- identifies call signs (KABC, WCBS, etc.)
- **Multi-source stream fallback** -- events and channels automatically try alternate providers if the primary feed fails, both at refresh time and mid-watch
- **Broadcast-network affiliate fallback** -- ABC / CBS / NBC / FOX automatically fall through to the flagship East Coast affiliate when the network's own feed isn't playable
- **Stream health monitoring** -- automatically detects offline channels and recovers them
- **ESPN-powered live sports** -- real-time scores, period-by-period linescore, venue, weather, and game situation data
- **Rich EPG data** -- 24-hour XMLTV guide with show descriptions, episode info, season/episode numbers, TV ratings, and show/movie poster artwork
- **Horizontal timeline guide** -- scrollable program grid with sticky channel column and current-time indicator
- **Theme switcher** -- light, dark, or auto (system preference detection)
- **Built-in video player** -- program progress bar, now-playing, up-next preview, and event scoreboards
- **Docker-network friendly URLs** -- one-click toggle in Settings switches copied playlist / EPG URLs between the dashboard's origin and the container's internal hostname, so Jellyfin / Plex containers on the same Docker network work without hand-editing
- **Targeted refresh controls** -- refresh just channels, just events, just the guide, or everything
- **Automatic failover** -- if a live stream hiccups mid-watch, the proxy switches to the next provider without ending the session
- **Adjustable stream buffering** -- optionally hold a few seconds of each live feed in memory before it reaches your player, so a brief provider hiccup drains the buffer instead of freezing the picture; off by default (as close to live as possible) and tunable in **Settings → Playback**
- **Cache survives restarts** -- the playlist, guide, and in-flight stream tokens are restored on boot, so clients keep playing through a container restart
- **One-click in-app updates** -- upgrade to the latest version straight from the dashboard with an **Update now** button, no command line needed (see [Updating](#updating))
- Continuous MPEG-TS stream proxy (works like a real TV tuner for Jellyfin/ffmpeg)
- Sport events sorted by start time with team logos and pregame/postgame EPG entries
- API key protection for playlist and EPG endpoints
- **Optional dashboard login** -- protect the dashboard with a username/password and add more admin accounts under Settings; leave it unset to keep the dashboard open on a trusted network
- **Guided first-run setup** -- a wizard walks first-time users through choosing a channel source, an optional playlist key, an optional dashboard login, and connecting Jellyfin / Plex
- Automatic hourly refresh with offline channel recovery every 5 minutes
- Graceful shutdown with SIGTERM handling
- Multi-architecture Docker support (amd64, arm64)

## Supported Live Sports

The following leagues currently have working live event feeds:

| League  | Sport             |
|---------|-------------------|
| **NHL** | Ice Hockey        |
| **NFL** | American Football |
| **NBA** | Basketball        |
| **MLB** | Baseball          |
| **MLS** | Soccer            |

ESPN is the source of truth for all schedules, team names, and scores — scrapers only contribute stream URLs that are cross-referenced against ESPN's canonical game data. Other leagues will be added as upstream feeds become available. To request a new league, [open a feature request](https://github.com/rebeliptv/iptv/issues/new/choose).

## Quick Start

> **Prefer a guided walkthrough?** The [setup guide at rebeliptv.net](https://www.rebeliptv.net) builds a Docker configuration tailored to your system (ports, timezone, in-app updates) and walks you through connecting Jellyfin / Plex. The manual setup below is the reference.

### Docker Compose (recommended)

Create a `docker-compose.yml`:

```yaml
services:
    iptv:
        image: rebeliptv/iptv:latest
        container_name: iptv
        hostname: iptv          # lets clients on this compose network reach us by name
        ports:
            - "8080:8080"
        # All environment variables are OPTIONAL — the defaults work out of
        # the box. Uncomment any you need (see Configuration below):
        # environment:
        #     TZ: "America/New_York"   # set to YOUR timezone, e.g. Europe/London, America/Chicago, Etc/UTC
        volumes:
            - iptv-data:/app/data
            - /var/run/docker.sock:/var/run/docker.sock   # enables one-click in-app updates
        stop_grace_period: 30s
        restart: unless-stopped

volumes:
    iptv-data:
```

```bash
docker compose up -d
```

> **Heads-up:** the `/var/run/docker.sock` mount lets the container update itself from the dashboard. It also gives the container control of the Docker daemon on the host — if you'd rather not allow that, remove the line and update with the standard `docker pull` method below instead.

### Docker Run

```bash
docker run -d \
  --name iptv \
  -p 8080:8080 \
  -v iptv-data:/app/data \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --restart unless-stopped \
  rebeliptv/iptv:latest

# Environment variables are optional (defaults work). Add -e flags as needed, e.g.:
#   -e TZ=America/New_York   (set to YOUR timezone — e.g. Europe/London, Etc/UTC)
```

## Updating

### In-app update

When a newer version is available, an **Update now** button appears on the **Dashboard** and in **Settings → Server** — click it and the server downloads the new version and restarts itself, with your data kept and no command line needed. This works out of the box with the Compose and `docker run` setups above, which mount the Docker socket so the container can update itself.

The button shows only on the `:latest` image when a newer release exists; it's hidden on version-pinned images (e.g. `:1.3.0`), which keep updating with the standard `docker pull` method below.

### Standard update

```bash
docker pull rebeliptv/iptv:latest
docker compose up -d
```

Your data lives in the `iptv-data` volume, so this just swaps in the new image — nothing is lost.

## Setting Up an API Key

Playlist and EPG endpoints can optionally be protected with an API key.

1. Open the web dashboard at `http://<server-ip>:8080`
2. Go to **Settings**
3. Click **Generate Key**
4. Use the key in your client URLs: `http://<server-ip>:8080/playlist?key=YOUR_KEY`

Without a key, playlist and EPG are accessible to anyone on your network.

## Usage with IPTV Clients

### Jellyfin

1. Go to **Dashboard > Live TV > Tuner Devices > Add**
2. Select **M3U Tuner**
3. Enter the M3U URL: `http://<server-ip>:8080/playlist?key=YOUR_KEY`
4. The EPG guide URL is auto-discovered from the playlist header
5. Save and refresh guide data

**Recommended tuner settings:**
- Allow stream sharing: **enabled**
- Auto-loop live streams: **enabled** — important: this lets playback automatically reconnect after a server restart or update. Without it, the channel simply stops when the container restarts and the viewer has to re-tune manually.
- Read input at native frame rate: **enabled**

### Plex

1. Install the **IPTV** plugin or use **xTeVe** as middleware
2. M3U URL: `http://<server-ip>:8080/playlist?key=YOUR_KEY`
3. EPG URL: `http://<server-ip>:8080/epg?key=YOUR_KEY`

### General IPTV Clients

| File             | URL                                             |
|------------------|-------------------------------------------------|
| M3U Playlist     | `http://<server-ip>:8080/playlist?key=YOUR_KEY` |
| EPG Guide (XML)  | `http://<server-ip>:8080/epg?key=YOUR_KEY`      |
| EPG Guide (Gzip) | `http://<server-ip>:8080/epg.gz?key=YOUR_KEY`   |

> **Note:** The `?key=` parameter is only required if an API key has been generated in the web dashboard under **Settings**.

## Web Dashboard

Access the dashboard at `http://<server-ip>:8080`.

On first launch a short setup wizard walks you through choosing a channel source, optionally adding a playlist key, optionally setting a dashboard login, and copying your playlist / guide URLs into your player. You can skip it and change anything later in Settings.

### Pages

- **Dashboard** -- channel/event counts, online/offline stats, live sports with start times and scores, playlist copy buttons
- **Channels** -- searchable and filterable channel list (filter by category, country, and status) with category badges and online/offline status; search matches the channel name, network, and city. Click a channel for its detail page with video player and program guide.
- **Guide** -- horizontal timeline program grid with sticky channel column, current-time indicator, and scrollable schedule
- **Sports** -- live and upcoming events grouped by date with team logos, live scores, and "Stream Not Available Yet" indicators for upcoming games. Click an event for its detail page with live scoreboard and video player.
- **Settings** -- grouped into **Server**, **Sources**, **Connect**, **Playback**, and **Access** sections: API key management, dashboard login & admin-account management, theme switcher, custom M3U source management, channel lineup management (enable/disable and drag-to-reorder), source-mode toggle (local scraping vs Rebel IPTV hosted feeds), Docker container-hostname toggle for endpoint URLs, optional channel numbers in the playlist, adjustable stream buffering, server info, version update check with one-click in-app upgrade, and targeted manual refresh (channels / guide / events / all)

### Channel Detail

Click any channel to see:
- Channel logo, categories, and status
- Embedded video player (click to watch)
- Now playing info with up-next preview
- Full program guide

### Sport Event Detail

Click any sport event to see:
- Team logos with live scores from ESPN (updates every 30 seconds)
- Period-by-period linescore
- Game status, venue, weather, and in-game situation
- Embedded video player with automatic fallback to alternate stream sources

## Configuration

| Variable        | Default      | Description                                                                                                  |
|-----------------|--------------|--------------------------------------------------------------------------------------------------------------|
| `PORT`          | `8080`       | Server port                                                                                                  |
| `HOST`          | `0.0.0.0`    | Bind address                                                                                                 |
| `CRON_SCHEDULE` | `30 * * * *` | Data refresh schedule (cron)                                                                                 |
| `SPORTS_EVENTS` | `true`       | Set to `false` to disable sports events                                                                      |
| `SPORTS_MODE`   | `false`      | Sports-only mode — disables TV channels, serves only live events                                             |
| `LEAGUES`       | `""`         | Comma-separated league codes (e.g. `NHL,NBA,MLB`) — empty means all                                          |
| `TZ`            | `Etc/UTC`    | Timezone                                                                                                     |

> **Source mode (local scraping vs hosted feeds)** is a dashboard setting, not an environment variable — see [Hosted Feeds](#hosted-feeds).

### Cron Schedule Examples

| Schedule       | Meaning                                |
|----------------|----------------------------------------|
| `30 * * * *`   | Every hour at 30 minute mark (default) |
| `*/30 * * * *` | Every 30 minutes                       |
| `0 */3 * * *`  | Every 3 hours                          |
| `0 0 * * *`    | Once daily at midnight                 |

### Blocked Sources

If some providers are blocked or geo-restricted in your region, switch to
[Hosted Feeds](#hosted-feeds) (Settings → Source mode). Your instance then
pulls ready-made channels, guide, and events from Rebel IPTV instead of
scraping the blocked providers itself, so regional blocks no longer matter.

### Hosted Feeds

By default your instance builds everything itself — it scrapes the sources,
assembles the playlist and guide, and proxies the streams. If you'd rather
your box not run the scrapers at all, switch it to hosted feeds from the
dashboard: **Settings → Source mode → "Use Rebel IPTV playlist"**.

Your instance then pulls its channels, live sports events, and guide from
Rebel IPTV's hosted service and serves them through your usual playlist,
guide, and stream endpoints — so the feeds are assembled and kept healthy
centrally instead of on your hardware. Nothing changes on the player side
(Jellyfin/Plex/Emby keep working exactly as before); only where the feeds
come from changes. In this mode there's no local scraping. Leave the
toggle off to keep your instance fully self-contained.

Your channel lineup is the same either way. Channels carry the same IDs in
both modes (they resolve against the same curated directory), so toggling
doesn't reshuffle your lineup or break your client's channel mappings and
bookmarks — only where the feeds come from changes. Switching takes effect
live: your instance rebuilds its channels, guide, and events from the
newly-selected source, and the toggle is disabled while that repopulates,
re-enabling once it finishes.

## How It Works

### Channels

On startup and once an hour, the server scrapes channel data, normalizes names, detects categories, and checks stream health. Offline channels are rechecked every 5 minutes.

Channel numbers are **stable** — every channel has a fixed number baked into the build, so your Jellyfin/Plex bookmarks survive cache purges and upgrades. Numbers are allocated in ranges:

| Range  | Use                                           |
|--------|-----------------------------------------------|
| 1–4999 | Curated broadcast networks and cable channels |
| 5000+  | Live sports events (assigned per refresh)     |
| 10000+ | Custom M3U sources (one range per source)     |

From **Settings → Channel Lineup** you can disable channels you don't watch (they drop out of your playlist, guide, and counts) and drag the rest into your own order. These are per-instance preferences saved with your data and layered on top of the built-in numbers, so they persist across restarts and upgrades.

Local broadcast stations are identified by their FCC call sign. For example, "ABC (KABC) Los Angeles" becomes **KABC Los Angeles CA (ABC)** — call sign, market, and network — and gets a logo showing the network mark with the call sign and city. Because the city and network are in the name, you can find a local station by searching for its city (e.g. **Los Angeles**) or network (e.g. **ABC**) in your player.

Channels are grouped by category in the M3U playlist using `group-title`. Multiple categories are supported (e.g., a local station gets both "Local" and "News").

### Custom M3U Sources

Add your own M3U playlists (URL or file upload) from the Settings page. Custom channels are merged into your playlist and EPG, numbered in the 10000+ range, and persist across restarts.

Each source can be edited after adding — rename it, change the URL, or upload a replacement M3U file — without losing its channel-number slot. Turn on the optional **Validate** toggle and the server health-checks each stream and marks offline ones red. While a source is being fetched or validated in the background, the entry shows a "loading" pill and updates automatically when the check completes.

### Sports Events

Sport events are scraped, then cross-referenced with ESPN's scoreboard API for accurate start times and canonical team names. Events are sorted by start time across all leagues.

Each event gets EPG entries with pregame, game, and postgame blocks:

| Sport | Pregame | Game      | Postgame |
|-------|---------|-----------|----------|
| NFL   | 30 min  | 3.5-4 hrs | 30 min   |
| NHL   | 30 min  | 3 hrs     | 30 min   |
| NBA   | 30 min  | 2.5 hrs   | 30 min   |
| MLB   | 30 min  | 3.5 hrs   | 30 min   |

Events are removed from the lineup 30 minutes after the postgame block ends.

### Stream Buffering

By default the server streams each live feed as close to real-time as possible. If your feeds occasionally freeze for a second or two when a provider hiccups, turn on **Settings → Playback → Stream buffering** and choose how many seconds to hold. The server then buffers that much of the feed in memory before delivering it to your player, so a brief upstream stall drains that cushion instead of interrupting playback.

The trade-off is start-up time: a buffered channel begins roughly that many seconds after you tune in — that wait *is* the buffer filling — and it then plays that many seconds behind live. Buffering applies in both local and hosted-feed modes and to any player (Jellyfin, Plex, Emby, or the built-in web player); some players add a little of their own start-up time on top. Leave it off to stay at the live edge.

## Persistent Data

Everything that needs to survive a restart lives in the `/app/data` volume, encrypted at rest:

- **`iptv.db.gcm`** — encrypted snapshot of all data: channels, sports events, EPG guide, custom-source definitions, your API key, and runtime metrics
- **`custom-m3u/`** — your uploaded custom M3U source files

The snapshot is AES-256-GCM encrypted, so the on-disk file is opaque — running `sqlite3 iptv.db.gcm` just reports "file is not a database". While the server is running it works against a plaintext copy held in memory that never touches persistent storage; on a clean shutdown that copy is re-encrypted back into the snapshot.

Channel numbers and the curated channel directory are baked into the image — not in persistent data — so cache purges never reshuffle numbers.

## Reverse Proxy

The server auto-detects the correct base URL from `X-Forwarded-Proto` and `X-Forwarded-Host` headers.

### Nginx

```nginx
location / {
    proxy_pass http://127.0.0.1:8080;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-Host $host;
}
```

### Traefik

```yaml
labels:
    - "traefik.enable=true"
    - "traefik.http.routers.iptv.rule=Host(`iptv.example.com`)"
    - "traefik.http.services.iptv.loadbalancer.server.port=8080"
```

## Reporting Issues

Found a bug or want a feature? [Open an issue](https://github.com/rebeliptv/iptv/issues/new/choose) using one of the templates:

- **Bug Report** -- something isn't working
- **Feature Request** -- suggest an improvement
- **Channel Request** -- missing or broken channel

## License

Proprietary -- free for personal, non-commercial use. See [LICENSE](LICENSE) for details.

**Forking this repository is a violation of the license.** This software may only be used in its published binary/container form. Forking, copying, or otherwise reproducing this repository creates an unauthorized derivative work, and all forks are subject to takedown.
