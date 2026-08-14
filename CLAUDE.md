# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A single-file static portfolio site for Vijay Chander (Senior Creative Producer). The entire site —
markup, CSS, data, and JS — lives in one file: `index.html`. There is no build step, no package
manager, and no server-side code. The file is large (~3MB) almost entirely because of inline
base64-encoded images (headshot + Instagram Reel thumbnails).

## Running / previewing

There is no build or dev server. Open `index.html` directly in a browser, or serve it locally:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

There are no tests, linters, or CI configured in this repo.

## Structure of `index.html`

The file is organized top to bottom as:

1. `<head>` — inline `<style>` block defining CSS custom properties for a dark/light theme
   (`:root[data-theme="dark"]` / `[data-theme="light"]`), set via `localStorage('vc-theme')` and
   a `prefers-color-scheme` fallback (see the inline `<script>` in `<head>`).
2. `<body>` — static markup for the status bar, hero header, and empty section containers
   (`#work`, `#contact`) that get populated by JS at load time.
3. A large `<script>` block near the end of the file containing all data and rendering logic:
   - `HEADSHOT_SRC` — base64 data URI for the hero headshot (very long single line).
   - `CONFIG` — object holding hero copy: name, roles, tagline, stats, and "highlights"
     (`CONFIG.photo` is a placeholder value; `HEADSHOT_SRC` is what's actually rendered).
   - `WORKS` — array of portfolio items (brand, title, YouTube/Instagram `url`, `band`, `poster` type).
   - `BANDS` — the three portfolio groupings (`senior`, `production`, `content`) with titles/colors;
     `WORKS` items are filtered by `band` and rendered under matching `BANDS` sections in DOM order.
   - `LOGOS` — brand logo image sources keyed by the `logo` field used in `WORKS`.
   - `IG_THUMBS` — base64 thumbnail images for Instagram reels, keyed by the reel shortcode
     parsed out of each `WORKS` item's Instagram URL (`reel/<shortcode>/`). If a shortcode has no
     entry here, `cardHTML()` falls back to a generated branded gradient poster using `igColor()`
     and the brand's `LOGOS` entry.
   - Rendering functions: `cardHTML()` builds each work card (YouTube thumbnails are fetched live
     from `i.ytimg.com`; Instagram cards use `IG_THUMBS` or the gradient fallback). All cards are
     plain links that open the source (YouTube/Instagram) in a new tab — there is no in-page video
     player.
   - Misc behavior: theme toggle (`setTheme`), a decorative on-screen timecode clock (`tick`),
     scroll-reveal via `IntersectionObserver`, and a lightbox (`#lb`) that is wired up but not
     currently triggered by anything (dead code kept for possible future use).

## Editing content

To change what's on the site, edit the data structures inline in the trailing `<script>` block
of `index.html` — there's no separate data file:

- Hero copy/stats/highlights → `CONFIG`
- Add/remove/reorder portfolio items → `WORKS` (each item needs `band`, `brand`, `logo`, `title`, `url`, `poster`)
- Add a new portfolio grouping → `BANDS` (add `{key, color, title, note}`, then give `WORKS` items that `band` key)
- Add/replace a brand logo → `LOGOS`
- Supply a real Instagram thumbnail instead of the gradient fallback → add a base64 entry to
  `IG_THUMBS` keyed by the reel's shortcode
- Replace the headshot → replace the `HEADSHOT_SRC` base64 string (or point `CONFIG.photo` to a
  hosted URL and remove `HEADSHOT_SRC`, since `HEADSHOT_SRC||CONFIG.photo` prefers the former)

When editing, work with a code editor's search rather than reading the whole file at once — several
lines (the base64 strings) are extremely long (2MB+) and will overwhelm a naive full-file read.
