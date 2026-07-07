---
name: fitness-overlay-youtube
description: >-
  Process Insta360 fitness videos end-to-end: match Strava activity, download FIT,
  burn HUD overlay with fitoverlay, upload to YouTube. Uses real Chrome over CDP
  (not automation browser). Use when asked to process fitness videos, post workout
  videos to YouTube, or run the fitness overlay pipeline.
---

# Fitness Overlay → YouTube

Drive the pipeline manually via browser MCP and shell — no brittle automation scripts.

## Prerequisites

- `fitoverlay` built from [fitness-overlay-v1](https://github.com/AdonisCodes/fitness-overlay-v1)
- `ffmpeg` / `ffprobe` on PATH
- Chrome DevTools MCP configured to attach to real Chrome (see `cursor/mcp.json.example`)
- `bin/chrome-cdp` launched before browser work
- Insta360 videos in iCloud watch folder: `~/Library/Mobile Documents/com~apple~CloudDocs/Fitness-Overlay-Sync`

## YouTube channel

Post to: `https://studio.youtube.com/channel/UCIBuvmVV6Z0i-dwQWnXveNQ`

## Auth: stop and wait

**Never attempt to complete login yourself.**

If the page URL or title indicates login:
- Strava: `/login`, "Log In | Strava", Apple sign-in for Strava
- YouTube/Google: `accounts.google.com`, "Sign in"

→ **Stop immediately.** Tell the user to log in in the Chrome window, then continue when they confirm.

## Chrome setup (required)

The MCP's default auto-launched Chrome uses `--enable-automation`. **Google blocks sign-in on that browser.**

1. Launch normal Chrome: `~/.local/bin/chrome-cdp` (or `bin/chrome-cdp` from this repo)
2. MCP config must use `--browser-url=http://127.0.0.1:9222` (not auto-launch)
3. Reload MCP in Cursor after config changes
4. Use `open -na` launch (script does this) so Chrome survives shell exit
5. Profile lives at `~/.chrome-cdp-profile` (copy of real Chrome profile). Do **not** re-sync while logged in.

## Pipeline

### 1. Parse video filename

Insta360: `VID_YYYYMMDD_HHMMSS_XX_NNN.mp4` → recording start time local.

Example: `VID_20260629_131331_00_042_02_12.MP4` → 2026-06-29 13:13:31.

### 2. iCloud video check

Before running `fitoverlay`, check physical size on disk:

```bash
du -h "/path/to/VID_....MP4"
```

If **0 B** (dataless iCloud placeholder):
1. `brctl download "/path/to/VID_....MP4"`
2. Poll until `du -h` shows full size (~320M for typical clip)
3. Then run fitoverlay

If already local → run fitoverlay directly.

### 3. Strava: find activity + download FIT

1. Navigate to `https://www.strava.com/athlete/training`
2. Find activity matching video date/time + duration overlap
3. Open activity → **Actions** → **Export Original**
4. FIT downloads to `~/Downloads/`

Title format for YouTube (from activity):
```
YYYY/MM/DD "Activity Name"
```
Example: `2026/06/29 "3.5km Easy Run"`

### 4. Run fitoverlay

```bash
fitoverlay \
  --fit "/path/to/activity.fit" \
  --out "/path/to/Fitness-Overlay-Sync/out" \
  --encoder h264 \
  --crf 28 \
  "/path/to/VID_....MP4"
```

Output: `out/VID_..._overlay.mp4`

Verify with `ffprobe` when done.

### 5. YouTube upload

1. Navigate to `https://studio.youtube.com/channel/UCIBuvmVV6Z0i-dwQWnXveNQ/videos/upload?d=ud`
2. Confirm correct channel is selected
3. Upload overlay mp4 via file input (`upload_file` MCP tool)
4. Set title: `YYYY/MM/DD "Activity Name"`
5. Set visibility and publish
6. Take screenshots at each step for user verification

## What never goes in this repo

- `~/.config/fitness-overlay-sync/config.json` (API keys)
- `strava_tokens.json`
- Chrome profile directories (`~/.chrome-cdp-profile`, `~/.cache/chrome-devtools-mcp/`)
- `.env` files with secrets
