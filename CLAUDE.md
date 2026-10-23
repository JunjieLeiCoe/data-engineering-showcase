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
  "file:///C:/Users/JLei/my_projects/data-engineering-showcase/index.html#now"
```

The page autoplays a timeline on load, so screenshot with a hash: `#now` = final state, `#2023` or `#2023-06` = the map as of that month. Headless Chrome on Windows won't go narrower than ~500px; to check the phone layout, screenshot a throwaway page that iframes `index.html#now` at 390x844.

## Files

| File | Role |
|---|---|
| `index.html` | Page, styles, map logic and the timeline (Leaflet 1.9.4, D3 7.9.0, Inter from CDN) |
| `data.js` | The dataset, `window.SHOWCASE`. The only file a normal update touches |
| `README.md` | Public description. Its pipelines badge must match `meta.statValue` |
| `.nojekyll` | Makes Pages serve files as-is |

## data.js shape

- `generatedAt` — date of the last update.
- `meta` — header text. `title` and `statValue` both carry the headline number ("140+"); keep them and the README badge in sync.
- `kinds` — industry key → `{emoji, label}`. The emoji is drawn inside the marker badge; the Industries chips (top-right) count and filter by kind.
- `categories` — six layers, each with a default `kind`. Every point is an object: `{at:[lat,lng], name, records, since:"YYYY-MM", kind, emoji, est}`.
  - `at` and `since` are required. `since` is the month of the first pipeline and drives the timeline.
  - `name` only on named layers (`enterprise`, `pipeline`); public-sector points are anonymous. `records` = verified CLNraw row count, shown as "Name · N records".
  - `kind` falls back to the layer's kind; `emoji` overrides the kind's emoji for one client (🍕 for pizza places, ✈️ for United, etc.).
  - `est:1` marks a placeholder `since`: no matching r-workspace folder was found. 49 of the anonymous public-sector dots are spread over Jun 2021 – Jun 2024, and 3 named clients (Advanced Steel Erectors, MZM Foods, Total Productive Staffing) sit at 2025-01. Replace them if real dates turn up.
  - How `since` was derived (Oct 2026): the earliest `.R` file mtime in the client's folder(s) outside `_ref_prior/`, capped at the month the client first appeared in `data.js` (scripts re-edited later would otherwise look newer than they are).
- `knownFolders` — every r-workspace folder already triaged (added *or* deliberately skipped). The update skill diffs the live folder list against this, so every processed folder must be appended, including the skipped util folders.

## How updates happen

The `showcase-map` skill does the whole loop: diff r-workspace folders against `knownFolders`, pull employer name + location + record count for each new client, append to the `pipeline` layer, validate, commit, push. Conventions it follows:

- New clients go in `pipeline`. `enterprise` (gold stars + the Featured Clients chips) is only for nationally known brands, and only when Junjie asks.
- Location = employer HQ if known, else the dominant employee city in the CLNraw (aggregate counts only).
- Exact coordinate collisions get nudged ~0.03° so both markers stay visible.
- `since` for a new client = the month its first script was written (usually the current month). Pick a `kind` from `kinds`; add a per-client `emoji` only when it reads better than the kind's.
- One marker per client, even when a folder produces several CLNraws (sum the record counts). A folder holding genuinely separate entities in different states gets one marker each, but a single client relationship that pays through per-state entities (Consumer Direct Care Network: 16 company codes, 14 states) stays one marker at HQ.
- The headline number is bumped by hand when enough pipelines are added; it is a rounded-down figure, not a computed one.

## index.html notes

- `data.js` is loaded with a cache-buster that changes every 10 minutes, so data updates show without a hard refresh. `index.html` itself can still be cached — after changing it, hard-refresh to check.
- Markers are emoji badges. Category key picks the frame: `enterprise` → larger gold circle with a ★, `pipeline` → cyan diamond, everything else → a circle in the layer color. Badge size follows zoom (`--s` on the map container). Legend rows toggle layers, Industries chips filter by kind; both combine with the timeline.
- Timeline (bottom dock): on load it plays from the month before the first `since` to `generatedAt` (~12 s), popping markers in as their month arrives. The D3 chart is stacked bars of new clients per quarter, colored by layer; the current quarter's bar grows and glows as months tick, future quarters show as faint ghosts. Drag it to scrub, hover a bar for its count. Legend counts, the Industries chips and the "recent clients" ticker all follow the playhead. `prefers-reduced-motion` skips the autoplay.
- The default view fits the lower 48 only (Alaska and outliers are excluded from the bounds on purpose).
- **Basemap:** CARTO `dark_all` raster tiles with the key in `CARTO_KEY`. Since September 2026 CARTO returns an "API KEY REQUIRED" placeholder tile (HTTP 200, so no tile error fires) when the key is missing or invalid. The key is necessarily visible to anyone loading the page; the free tier is 5M requests/month for non-commercial use. Esri's keyless Dark Gray Canvas was tried as a replacement and rejected: lighter gray, and noticeably slower.

## Current data state (2026-10-23)

197 markers: municipal 19, county 8, education 26, agency 6, enterprise 7, pipeline 131. By first-pipeline year: 2021 10, 2022 16, 2023 25, 2024 30, 2025 48, 2026 68 (2021–2024 includes the placeholder public-sector dates).

Locations worth a second look:

- Community Health System Services — no HQ found; placed at the dominant employee city (Elmira, NY).
- Devoted Guardians — placed at Albuquerque, NM from the data; the brand's public HQ is in Arizona.
- Gateway Fiber — Wright City, MO, unverified; data only shows employees around Warrenton / Wentzville.
- License Office Services — Cassville, MO per one directory; another lists Jefferson City.
- City of Sanger — appears twice: the older anonymous municipal dot and the new named pipeline diamond (nudged north).
- HealthQuest Therapeutics — Houston, TX from the data (704 of 1,798 rows), nudged off Taqueria La Reyna's exact point.
- LD Solutions — Denver, CO (FirstLight Home Care of Denver), nudged off Colorado Security Agency's exact point.
- Aveena's Loving Companion Care — moved from Chantilly, VA to Jamaica, Queens, NY (HubSpot company record + 39K of 48K rows are NY).
