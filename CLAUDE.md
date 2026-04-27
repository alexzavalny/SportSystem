# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A personal workout-tracking PWA ("Zavalny Sport System"). No framework, no bundler, no backend — the entire application is a single `index.html` file with inline CSS and JS, served statically. Bootstrap 4.5.2 is loaded from CDN.

## Running locally

No install step needed. Serve the project root over HTTP (service workers require `localhost` or HTTPS, not `file://`):

```bash
python3 -m http.server 8080
# then open http://localhost:8080
```

There is no build process and no test suite.

## Architecture

**Everything lives in `index.html`.** The key logical pieces:

### Training cycle engine
- `dayOfYear(date)` — ordinal day of year.
- `currentProgramDay(date)` — maps ordinal into a 4-element cycle `[0, 1, 0, 2]`, producing Day 0 (rest), Day 1 (push), or Day 2 (pull). Effective rhythm: rest → push → rest → pull → repeat.
- Result is rendered via `document.write()` on page load.

### State / persistence
- Each checkbox has a unique id like `day1-exercise1-set1`.
- `saveCheckboxState(checkbox)` stores state under `YYYY-MM-DD__<id>` in `localStorage`.
- `restoreCheckboxState()` restores today's checked state; checkboxes checked *yesterday* are restored as checked **and disabled** (read-only).

### Bilingual UI
- Every visible string has two sibling `<span>` elements with classes `.language-en` and `.language-ru`.
- `switchLanguage(lang)` toggles their `display` and saves to `localStorage`.

### Service Worker (`sw.js`) — cache key: `"v4"`
- Cache-first strategy: install caches `index.html` + icons; activate deletes all caches except current version.
- **To invalidate the cache after editing `index.html`, increment the `CACHE_NAME` constant in `sw.js`.**

### PWA manifest (`manifest.json`)
- `display: standalone`, `orientation: portrait`, theme `#4A90E2`.
