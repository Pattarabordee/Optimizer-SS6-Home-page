# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

Digital yearbook/showcase site for **Optimizer Super AI Engineer Season 6** (Thai AI engineering cohort). Members, clips, and gallery are loaded live from Google Sheets and Google Drive.

## Running Locally

No build step — serve `index.html` directly:

```bash
python3 -m http.server 8000
# or
npx serve .
```

## Architecture

**Single file:** all HTML, CSS, and JS live in `index.html`. The only other asset is `assets/logo.svg`.

**Data flow:**
- **Member profiles & clips** — fetched via JSONP from Google Sheets Visualization API every 30 s. Fallback hardcoded array (`optimizerMembers`) is used until the sheet responds.
- **Photo gallery** — fetched from a public Google Drive folder via `allorigins.win` proxy (CORS workaround).
- Deduplication uses a hash signature (`lastSheetSignature`, `lastDriveGallerySignature`) so the DOM is only re-rendered when data actually changes.

**Key Google IDs (in `index.html`):**

| Constant | Value |
|---|---|
| `SHEET_ID` | `1RcLKNSw49T5nktRj0YyOLTf3PyrHJXI35yVGJtTuaJ0` |
| `DRIVE_FOLDER_ID` | `1JVGRIUVJxuPbl-p6hVnVWsGBuIdnkTWl` |
| `SHEET_REFRESH_MS` | `30000` |

## Styling / Theme System

CSS custom properties define four color themes set on `<html data-theme="...">`:

- `optimizer` (default) — yellow/black
- `sky` — blue
- `emerald` — green
- `violet` — purple

Tailwind is loaded from CDN with a runtime config that maps `primary`, `secondary`, `accent` to the CSS vars. Persist selected theme via `localStorage`.

**Background pattern:** `.blueprint-bg` uses a radial gradient + CSS grid overlay.

## Key Functions

| Function | Purpose |
|---|---|
| `loadSheetData()` | JSONP request → Google Sheets |
| `sheetRowsToMembers()` | Parses Sheets response into member objects |
| `membersToClips()` | Filters members that have a video URL |
| `loadDriveGalleryData()` | Fetches Drive embed page, parses file IDs |
| `renderMemberCards()` | Writes `#member-grid` innerHTML |
| `renderClipCards()` | Writes `#clip-grid` innerHTML |
| `renderDriveGallery()` | Writes `#drive-photo-grid` innerHTML |
| `applyTheme(theme)` | Updates `data-theme` attr + localStorage |
| `escapeHtml()` | XSS prevention for all user-sourced strings |

## Page Sections (in order)

`nav` → `#home` (hero) → `#about` → `#committee` → `#members` → `#clips` → memory capsules → `#gallery` / `#recognition` → `footer`

Navigation uses hash anchors with smooth scroll; no client-side router.

## Gotchas

- All dynamic content must be passed through `escapeHtml()` before interpolation into template strings.
- `normalizeImageUrl()` converts various Google Drive share-link formats to thumbnail URLs — keep it consistent if adding new image fields.
- JSONP appends a `<script>` tag to `<head>` on every poll; old script tags accumulate but are harmless.
- The `allorigins.win` proxy is the only way the Drive gallery works from a browser without a backend — if it goes down, the gallery silently fails to load.
