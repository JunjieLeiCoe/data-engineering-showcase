# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A static, single-page Leaflet map of the organizations Junjie has built ETL / data-engineering pipelines for. No build step, no dependencies to install. Served by GitHub Pages from `main` / root:
https://junjieleicoe.github.io/data-engineering-showcase/

**This repo is PUBLIC on purpose** (GitHub Pages needs it). Never flip it to private — that takes the site down. Skills that assert "repo must be private" (`done-r`, `push-to-github`) do not apply to that check here.

## Hard rules

- Everything committed is world-readable. Only employer **name + city/state + a record count** ever goes in `data.js`. Never SSNs, employee names, DOB, pay, or anything copied out of a client file.
- Deploy = commit + push to `main`. Pages rebuilds in ~1 minute.
- Set a git identity before committing. The WSL root user has none configured, so commits fall back to `root@<work hostname>`, which leaks the machine name into public history and doesn't count toward the GitHub profile.

## Commands

```bash
# validate data.js parses + recount markers (run after every edit)
node -e 'global.window={}; eval(require("fs").readFileSync("data.js","utf8"));
const s=window.SHOWCASE; console.log(s.categories.map(c=>c.key+":"+c.pts.length).join(" "), "| known:", s.knownFolders.length)'

# local preview screenshot (headless Windows Chrome; delete _preview.png afterwards, never commit it)
"/mnt/c/Program Files/Google/Chrome/Application/chrome.exe" --headless=new --disable-gpu --hide-scrollbars \
  --window-size=1400,800 --virtual-time-budget=9000 \
  --screenshot="C:\\Users\\JLei\\my_projects\\data-engineering-showcase\\_preview.png" \
  "file:///C:/Users/JLei/my_projects/data-engineering-showcase/index.html"
```

## Files

| File | Role |
|---|---|
| `index.html` | Page, styles and all map logic (Leaflet 1.9.4 + Inter from CDN) |
| `data.js` | The dataset, `window.SHOWCASE`. The only file a normal update touches |
| `README.md` | Public description. Its pipelines badge must match `meta.statValue` |
| `.nojekyll` | Makes Pages serve files as-is |

## data.js shape

- `generatedAt` — date of the last update.
- `meta` — header text. `title` and `statValue` both carry the headline number ("140+"); keep them and the README badge in sync.
- `categories` — six layers. Public-sector layers (`municipal`, `county`, `education`, `agency`) are anonymous `[lat, lng]`. Named layers (`enterprise`, `pipeline`) are `[lat, lng, "Name"]` with an optional 4th element = verified CLNraw record count, shown in the tooltip as "Name · N records".
- `knownFolders` — every r-workspace folder already triaged (added *or* deliberately skipped). The update skill diffs the live folder list against this, so every processed folder must be appended, including the skipped util folders.

## How updates happen

The `showcase-map` skill does the whole loop: diff r-workspace folders against `knownFolders`, pull employer name + location + record count for each new client, append to the `pipeline` layer, validate, commit, push. Conventions it follows:

- New clients go in `pipeline`. `enterprise` (gold stars + the Featured Clients chips) is only for nationally known brands, and only when Junjie asks.
- Location = employer HQ if known, else the dominant employee city in the CLNraw (aggregate counts only).
- Exact coordinate collisions get nudged ~0.03° so both markers stay visible.
- One marker per client, even when a folder produces several CLNraws (sum the record counts). A folder holding genuinely separate entities in different states gets one marker each.
- The headline number is bumped by hand when enough pipelines are added; it is a rounded-down figure, not a computed one.

## index.html notes

- `data.js` is loaded with a cache-buster that changes every 10 minutes, so data updates show without a hard refresh. `index.html` itself can still be cached — after changing it, hard-refresh to check.
- Marker style is chosen by category key: `enterprise` → star, `pipeline` → diamond, everything else → pulsing dot. Legend rows toggle layers.
- The default view fits the lower 48 only (Alaska and outliers are excluded from the bounds on purpose).
- **Basemap:** CARTO `dark_all` raster tiles with the key in `CARTO_KEY`. Since September 2026 CARTO returns an "API KEY REQUIRED" placeholder tile (HTTP 200, so no tile error fires) when the key is missing or invalid. The key is necessarily visible to anyone loading the page; the free tier is 5M requests/month for non-commercial use. Esri's keyless Dark Gray Canvas was tried as a replacement and rejected: lighter gray, and noticeably slower.

## Current data state (2026-09-30)

194 markers: municipal 19, county 8, education 26, agency 6, enterprise 7, pipeline 128.

Locations from the last batch that were not confirmed against a hardcoded address and are worth a second look:

- Community Health System Services — no HQ found; placed at the dominant employee city (Elmira, NY).
- Devoted Guardians — placed at Albuquerque, NM from the data; the brand's public HQ is in Arizona.
- Gateway Fiber — Wright City, MO, unverified; data only shows employees around Warrenton / Wentzville.
- License Office Services — Cassville, MO per one directory; another lists Jefferson City.
- City of Sanger — appears twice: the older anonymous municipal dot and the new named pipeline diamond (nudged north).
