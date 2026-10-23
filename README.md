# Data Engineering Showcase — Junjie Lei

An interactive map of the organizations I've built ETL / data-engineering pipelines for, across the United States.

**Live:** https://junjieleicoe.github.io/data-engineering-showcase/

![layers](https://img.shields.io/badge/layers-6-22d3ee) ![pipelines](https://img.shields.io/badge/ETL%20pipelines-140%2B-fbbf24)

## What it is
A single-page Leaflet map (no build step) showing client engagements grouped into layers:

| Marker | Layer | Meaning |
|--------|-------|---------|
| ● circles (blue/green/violet/pink) | Municipal · County · Education · Public Agency | Public-sector sectors served (anonymized) |
| ★ gold badges | Enterprise & National Brands | Flagship clients (hover for name) |
| ◆ cyan diamonds | ETL / Data Engineering Pipelines | Pipelines I built (hover for name) |

Each badge carries an industry emoji: 🏛️ government, 🎓 education, 🏥 healthcare, 🏦 finance, ✈️ airline, 🏗️ construction, 🛡️ security, and so on.

**Timeline.** On load the map replays 2021 → today, popping each client in the month its first pipeline was built. Drag the chart to scrub, or link straight to a moment with `#2023` / `#2024-06` (`#now` skips the intro).

Tap a legend row to filter a layer, or an Industries chip to filter by industry.

## Files
- `index.html` — the page (presentation, map logic, timeline). Loads Leaflet, D3 and Inter from CDN.
- `data.js` — the dataset (`window.SHOWCASE`): header text, layers, and the `knownFolders` registry. **This is the file that gets updated when new clients are added.**
- `.nojekyll` — tells GitHub Pages to serve files as-is.

## Updating it
New clients are added automatically by the **`showcase-map`** Claude Code skill, which scans my local `r-workspace`, geolocates any new client pipelines, appends them to `data.js`, and pushes here. To do it manually, add `{at:[lat, lng], name:"Company Name", since:"YYYY-MM", kind:"health"}` to the `pipeline` category in `data.js` and commit.

## Deploy (one-time)
Served from the `main` branch root via GitHub Pages (Settings → Pages → Source: `main` / `/`).

## Embed
```html
<iframe src="https://junjieleicoe.github.io/data-engineering-showcase/"
        style="width:100%;height:600px;border:0;border-radius:12px"></iframe>
```

---
*Built with [Claude Code](https://claude.com/claude-code).*
