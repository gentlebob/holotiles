# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file (`index.html`) YouTube video conference viewer — a Zoom-like layout for multiple simultaneous YouTube embeds. No build step, no dependencies beyond the YouTube IFrame API loaded from CDN.

## Running

Open `index.html` directly in a browser, or serve it with any static file server:

```
npx serve .
python3 -m http.server
```

The backend API (`/api/videos`) is currently mocked inside `fetchVideos()` in the `<script>` block. Swap that function body for a real `fetch()` call when a backend is available.

## Architecture

Everything lives in `index.html` in three sections:

**CSS** — two layout modes driven by `@media (min-aspect-ratio: 1/1)`:
- Landscape: main tile on the left, secondary strip on the right
- Portrait: main tile on top, secondary strip on the bottom

**DOM structure** — all `.player-container` divs are siblings inside `#conference` and are **never reordered**. Layout is applied purely via `position: absolute` + `left/top/width/height` so iframes never reload when swapping videos.

**JavaScript state:**
- `videos[]` — source-of-truth list from the API (`{id, title}`)
- `players[]` — `YT.Player` instances, indexed to match `videos[]`, never reordered
- `containers[]` — corresponding DOM elements, same indexing
- `positionOrder[]` — maps display slot → player index; slot 0 is always the main tile

**Swap logic** (`onPlayerClick`): only mutates `positionOrder`, then calls `applyLayout()` to reposition containers via CSS. No `loadVideoById` calls, no DOM moves.

**Layout** (`applyLayout` / `calculateRects`): called on init and via `ResizeObserver`. Computes pixel rects for all slots and writes them to each container's inline style.

**Audio**: all players start muted (browser autoplay policy). The 🔇/🔊 button toggles `mainMuted`; `updateAudio()` unmutes only `players[positionOrder[0]]`.
