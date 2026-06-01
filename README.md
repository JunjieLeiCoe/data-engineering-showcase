# Data Engineering Showcase — Junjie Lei

An interactive map of the organizations I've built ETL / data-engineering pipelines for, across the United States.

**Live:** https://junjieleicoe.github.io/data-engineering-showcase/

![layers](https://img.shields.io/badge/layers-6-22d3ee) ![pipelines](https://img.shields.io/badge/ETL%20pipelines-120%2B-fbbf24)

## What it is
A single-page Leaflet map (no build step) showing client engagements grouped into layers:

| Marker | Layer | Meaning |
|--------|-------|---------|
| ● blue/green/violet/pink dots | Municipal · County · Education · Public Agency | Public-sector sectors served (anonymized) |
| ★ gold stars | Enterprise & National Brands | Flagship clients (hover for name) |
| ◆ cyan diamonds | ETL / Data Engineering Pipelines | Pipelines I built (hover for name) |

Tap any legend row to filter a layer.

## Files
- `index.html` — the page (presentation + map logic). Loads Leaflet + Inter from CDN.
- `data.js` — the dataset (`window.SHOWCASE`): header text, layers, and the `knownFolders` registry. **This is the file that gets updated when new clients are added.**
- `.nojekyll` — tells GitHub Pages to serve files as-is.

## Updating it
New clients are added automatically by the **`showcase-map`** Claude Code skill, which scans my local `r-workspace`, geolocates any new client pipelines, appends them to `data.js`, and pushes here. To do it manually, just add a `[lat, lng, "Company Name"]` entry to the `pipeline` category in `data.js` and commit.

## Deploy (one-time)
Served from the `main` branch root via GitHub Pages (Settings → Pages → Source: `main` / `/`).

## Embed
```html
<iframe src="https://junjieleicoe.github.io/data-engineering-showcase/"
        style="width:100%;height:600px;border:0;border-radius:12px"></iframe>
```

---
*Built with [Claude Code](https://claude.com/claude-code).*
