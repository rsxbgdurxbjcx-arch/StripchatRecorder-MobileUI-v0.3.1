# StripchatRecorder-MobileUI-v0.3.1

[简体中文](README.md) | [English](README.en.md)

A self-hosted Stripchat live stream recorder with a web-based management UI. Supports automatic recording, post-processing pipelines, and multi-channel notifications.

> This project is forked from [ChanTrail/StripchatRecorder](https://github.com/ChanTrail/StripchatRecorder), with the following additions:
> - **Android mobile UI adaptation**: bottom navigation bar (with icons), 2-column streamer cards, compact recordings table, fixed page size
> - **Login system**: initial account `sr-mobileui` / password `admin`, changeable in settings
> - **Telegram auto-delete after upload**: automatically deletes local video files after successful Telegram upload

[![License: GPL-3.0](https://img.shields.io/badge/License-GPL--3.0-blue.svg)](https://www.gnu.org/licenses/old-licenses/gpl-3.0.html)

---

## Features

- Monitor multiple streamers and auto-record when they go live
- Web UI for managing streamers, recordings, and post-processing
- **Android mobile UI adaptation**: bottom navigation bar (with icons), 2-column streamer cards, compact recordings table, toast notifications not blocking bottom nav
- **Login system**: token-based authentication to protect the Web UI from unauthorized access
- **Streamer Finder**: discover streamers via [camgirlfinder.net](https://camgirlfinder.net)
- **HLS Relay**: proxy a streamer's live stream to any player without recording
- Supports split network proxies: configure Stripchat API proxy and CDN chunk proxy separately
- **Mouflon HLS decryption**: manage `pkey → pdkey` key pairs
- Configurable post-processing pipeline with pluggable modules:
  - **contact_sheet** — generates a tiled preview image with timestamps
  - **filter_short** — deletes recordings below a minimum duration
  - **notify_discord** — sends recording info and cover image to a Discord Webhook
  - **notify_telegram** — sends recording info, cover image, and video via MTProto (**supports auto-delete local file after upload**)
- Disk space monitoring on the recordings page
- Real-time UI updates via Server-Sent Events with multi-client sync
- Dark/light mode following system theme

---

## Quick Start (Docker)

### Deployment

```bash
git clone https://github.com/rsxbgdurxbjcx-arch/StripchatRecorder-MobileUI-v0.3.1.git
cd StripchatRecorder-MobileUI-v0.3.1
docker compose up -d
```

After startup, open `http://localhost:4040` in your browser and log in with the initial account `sr-mobileui` / password `admin`.

> **Deployment speed**: This project pulls the pre-built Docker image `chantrail/stripchat-recorder:latest` from Docker Hub for the runtime environment, and injects the custom backend binary (with mobile UI + login system) and the modified `notify_telegram` module binary via volume mount. The entire deployment process typically completes within **2 minutes**.

The Docker image runs in Server mode (port 4040), with configuration written to the mounted `config/settings.json`.

`docker-compose.yml` configuration (included in the repository):

```yaml
services:
  stripchat-recorder:
    image: chantrail/stripchat-recorder:latest
    container_name: stripchat-recorder
    restart: unless-stopped
    environment:
      - TZ=Asia/Shanghai
      # - LANGUAGE=en-US  # Set interface language: zh-CN (default) or en-US
    ports:
      - "4040:3030"
    volumes:
      - ./data/logs:/app/stripchat-recorder/logs
      - ./data/recordings:/app/stripchat-recorder/recordings
      - ./data/modules:/app/stripchat-recorder/modules
      - ./data/config:/app/stripchat-recorder/config
      # Override the in-image backend binary with custom build (mobile UI + login system)
      - ./data/bin/stripchat-recorder:/app/stripchat-recorder/stripchat-recorder
```

### Main Settings

The following options can be configured in the Web UI "Settings" page:

| Setting | Description |
|---------|-------------|
| Output directory | Recording file save path |
| Max concurrent recordings | Maximum number of simultaneous recordings, `0` = unlimited |
| Poll interval | Interval to check if streamers are online (seconds), range 10–300 |
| Merge format | Format for auto-merging segments after recording: `mp4` (default), `mkv`, `ts` |
| Auto-record on live | Whether new streamers have auto-record enabled by default |
| Max temp dir usage | Maximum temporary file usage for post-processing modules (GB), `0` = unlimited, default 50 GB |

### Network Proxies and Mirror

In the "Network" section of the settings page, you can configure the API proxy, CDN proxy, and Stripchat mirror separately.

### Mouflon HLS Decryption Keys

Stripchat encrypts HLS segment filenames (Mouflon system). If you encounter issues downloading segments during recording, you need to fill in the corresponding `pkey → pdkey` key pairs in the "Mouflon Decryption Keys" section of the settings page.

### HLS Relay

In Server mode, you can directly open the following URL in a player to play the live stream:

```
http://localhost:4040/stream/{modelname}
```

---

## New Features

### 1. Android Mobile UI Adaptation

- **Bottom navigation bar**: On mobile (< 768px), the desktop sidebar is hidden and a bottom navigation bar with functional icons is displayed
- **2-column streamer cards**: On mobile, the streamer list displays 2 cards per row, with compact card height and fully visible thumbnails
- **Compact recordings table**: On mobile, the recordings table uses smaller font size, reduced spacing, and adapts to narrow screens
- **Toast notifications not blocking**: Toast notifications are moved up to avoid blocking the bottom navigation bar
- **Fixed page size**: Uses `position: fixed` to fix the viewport, preventing layout jumps caused by Android browser address bar expansion/contraction
- **Input zoom prevention**: Mobile input fields use 16px font size to prevent Android Chrome focus zoom
- **Post-processing input alignment**: All module input fields are aligned with uniform height

### 2. Login System

- **Initial account**: `sr-mobileui` / password: `admin`
- **Authentication mechanism**: Token-based authentication, all API requests carry `Authorization: Bearer {token}` after login
- **SSE authentication**: SSE connections pass tokens via query parameters (EventSource does not support custom headers)
- **Change password**: In the "Settings" page, you can change the account and password. After changing, you are automatically logged out and need to re-login
- **Persistence**: Account and password are stored in `config/auth.json` (password hashed with SHA-256)
- **Security**: After changing the password, all logged-in tokens are invalidated, forcing re-login

### 3. Telegram Auto-Delete After Upload

In the Web UI "Post-processing" page, when editing the `notify_telegram` node parameters, you can see the "**Delete local file after upload**" switch:

- **Off (default)**: Local video files are retained after successful upload
- **On**: After the video file is successfully uploaded to Telegram, the local video file and its metadata are immediately deleted

---

## Post-processing Modules

Modules are standalone executables that implement a simple protocol. The repository includes the pre-compiled binary for the modified `notify_telegram` module in `data/modules/notify_telegram_v030`. The other three modules are provided by the Docker image.

---

## Tech Stack

- **Frontend:** Vue 3, TypeScript, Vite, Tailwind CSS, Reka UI
- **Backend:** Rust, Axum
- **Post-processing modules:** Rust (standalone binaries)
- **Container:** Debian, ffmpeg

---

## Acknowledgments

This project is based on [ChanTrail/StripchatRecorder](https://github.com/ChanTrail/StripchatRecorder). Thanks to the original author for their contribution.

---

## License

This project is released under the [GNU General Public License v3.0](https://www.gnu.org/licenses/old-licenses/gpl-3.0.html).

---

## Disclaimer

This project is for technical research and learning purposes only. Users bear all risks related to deployment, operation, and compliance.
