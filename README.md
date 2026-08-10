# डीलक्स सैलून — Deluxe Saloon

A single-file, mobile-first "shared listening room" web app — an illustrated saloon scene, a live visitor counter, and a YouTube-playlist-powered music player docked at the bottom.

**Live demo:** https://super-deluxe-saloon.vercel.app

---

## Features

- 🎨 Full-bleed illustrated background scene
- 🕐 Live clock (device time)
- 🟢 Real-time "X online" visitor counter, backed by Upstash Redis
- 🎵 Bottom floating player, controlling actual YouTube playlist playback via the YouTube IFrame API
  - Play / pause, next / previous track
  - Seekable progress bar with live time
  - Auto-loads the first track's title, artist, and thumbnail on page load
  - Circular album art, pill-shaped player bar
- 📱 iOS-safe inline playback (no fullscreen popup takeover)
- 📦 Fully self-contained — one HTML file, no build step, no server required

---

## Setup

Everything lives in `index.html`. Open it and find the `CONFIG` block near the top of the `<script>` tag:

```js
const PLAYLIST_ID = "PLDfWs4LxMSoI";

const UPSTASH_URL   = "https://up-humpback-190478.upstash.io";
const UPSTASH_TOKEN = "gQAAAAAAAugOAAIgcDIzYjgxMDZiMTMxYTY0YmJmOWI3YTQ0OGRiODhmZjcxMw";
```

### 1. YouTube playlist

Replace `PLAYLIST_ID` with your own playlist's ID (grab the full URL from **youtube.com** on desktop — mobile share links are often shortened/truncated and won't work).

### 2. Live visitor counter (Upstash Redis)

1. Create a free database at [upstash.com](https://upstash.com)
2. On the database page, open the **REST API** section
3. Copy the **REST URL** and the **read/write REST token** (not the read-only one — write access is required for the heartbeat to register visitors)
4. Paste them into `UPSTASH_URL` and `UPSTASH_TOKEN`

⚠️ **Note:** since this is a static, serverless page, the Upstash token is called directly from the browser and is visible in the page source. That's an acceptable tradeoff for a simple viewer counter, but don't reuse this token/database for anything sensitive.

---

## How the live counter works

Each open browser tab "heartbeats" to Redis every 12 seconds:

1. `ZADD` — adds/updates this client's timestamp in a sorted set
2. `ZREMRANGEBYSCORE` — prunes any client not seen in the last 40 seconds
3. `ZCARD` — returns the current count of active clients

This is an approximate presence count (accurate to within ~40 seconds), not an exact real-time signal — a visitor who closes the tab is only removed once their entry goes stale, not the instant they disconnect.

---

## Deployment

Static, single-file site — deploy anywhere that serves static HTML:

- **Vercel** — drag-and-drop `index.html` or connect a repo, zero config needed
- **Netlify** — same, drag-and-drop deploy
- Any static host / CDN

No build step, no environment variables needed beyond what's hardcoded in the config block above.

---

## Tech stack

- Vanilla HTML / CSS / JS — no framework, no dependencies
- [YouTube IFrame Player API](https://developers.google.com/youtube/iframe_api_reference) for playlist playback
- [Upstash Redis REST API](https://upstash.com/docs/redis/features/restapi) for the live presence counter
- Google Fonts: Yatra One (Devanagari display), Baloo 2, Poppins

---

## Known limitations

- Presence count is approximate (see above) — not instant on disconnect
- Upstash token is exposed client-side (fine for a low-stakes viewer counter, not for anything sensitive)
- Background artwork is a static image — swap the `background:url('data:image/jpeg;base64,...')` value in `.scene` to use a different one (re-encode your image as base64 and paste it in)
