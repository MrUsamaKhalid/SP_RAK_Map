# Al Marjan Island Property Intelligence — Sykon Properties

A property-intelligence map for **Al Marjan Island, Ras Al Khaimah**: zone/project pins, price evolution, rental-yield signals, and a mortgage / ROI / affordability calculator. Built as a separate sister project to the [Dubai Property Intelligence map](https://mrusamakhalid.github.io/SP_Dubai_Property_Map/), reusing its frontend with a completely different data back end.

## Why this map is fed differently from the Dubai map

Dubai runs on official DLD open CSVs (~6.7 GB) processed by a Node pipeline. **Ras Al Khaimah publishes no equivalent open transaction dataset** — no DLD, no data.dubai, no bulk CSVs. This map is therefore fed by a small **curated JSON** maintained by hand and refreshed quarterly, seeded from published market data (ValuStrat Price Index, Reliant Surveyors, developer/broker pricing).

## Architecture

```
data/rak/almarjan.json   ←  the master dataset (edit this)
        │ fetched on page load by a ~70-line vanilla JS loader
        ▼
index.html               ←  single-file frontend (vanilla JS + Leaflet), cloned
                            from the Dubai map. Carries a byte-identical inline
                            fallback of the JSON so the page still works when
                            fetch is unavailable (e.g. opened from disk).
```

- No build tools, no npm dependencies. External resources: Google Fonts, Leaflet 1.9.4 + leaflet.heat (unpkg), MapLibre GL 5.24.0 + maplibre-gl-leaflet 0.1.4 (unpkg) drawing the OpenFreeMap vector basemap (Positron in light mode, Dark in dark mode; no API key).
- `window.rakData` exposes `citywide`, `zones`, and `getZone(slug)` in the console.

## The data contract (`data/rak/almarjan.json`)

| Key | Contents |
|---|---|
| `meta` | as-of dates, sources, refresh cadence, disclaimer |
| `citywide` | island-wide hero stats: avg AED/sqft, YoY growth, transaction growth since the Wynn announcement, foreign-buyer share, ValuStrat VPI, RAK-wide apartment/villa psf, Wynn facts |
| `zones[]` | one object per map pin (14 zones) — id, name, coordinates, developer, launch/handover, launch & current AED/sqft, growth, yield, signals, price timeline, unit prices, per-zone `source_note` |

Every displayed number traces to a field in this file: the loader rewrites the five topbar hero cards (`data-hero` attributes), the map-overlay stats, the as-of badges, and rebuilds pins/feed/ticker/calculator from `zones`.

### The 14 zones (pins)

Wynn Resort District · Oceano (Luxe) · Nikki Beach & Rosso Bay (Aldar) · Address Residences (Emaar) · Playa Del Sol (Ellington) · Costa Mare (Ellington) · La Mer by Elie Saab (ARTE) · DAMAC Shoreline · The Astera / Aston Martin (Dar Global) · Oystra / Zaha Hadid (Richmind) · Cala Del Mar (Ellington) · Pacific (Select Group, completed) · Bab Al Bahr (Al Hamra, completed) · Danah Bay (Dubai Investments, completed villas)

## Data provenance & caveats

- **Basis**: listing/asking prices, not registered transactions (RAK has no public transaction registry). Off-plan yields are projections.
- Citywide figures are **Q1 2026** (ValuStrat VPI 124.1, base Q1 2024 = 100; Reliant Surveyors listings data). Per-project prices are **mid-2026** developer/broker asking prices.
- Values marked `(est.)` are interpolated or estimated (notably launch-year psf and yearly steps for off-plan projects) — see each zone's `source_note`. Verify before quoting to clients.
- Calculator fees use the RAK schedule: 4% transfer fee (customarily split 2% buyer / 2% seller on resale; shown in full as the conservative maximum), ~AED 3,250 registration/trustee fixed fees, 0.25% + AED 290 mortgage registration, 2% + VAT agency. UAE Central Bank LTV caps are federal and unchanged.

## Quarterly refresh

1. Edit `data/rak/almarjan.json` (new VPI print, updated psf, new launches/handovers). Keep the inline `const AREAS = [...]` fallback in `index.html` in sync (copy the `zones` array across) — the page warns of nothing if they drift, the fallback is only used without a web server.
2. Test locally: `python3 -m http.server` and open `http://localhost:8000`.
3. Review on a separate branch if you want a check before going live. Nothing outside the default branch is published.
4. Push to the default branch `claude/new-session-4252nf`. **Live GitHub Pages deploys from the default branch on every push** (`.github/workflows/pages.yml`). Check the Actions run, then the live page.

## Repository layout

```
index.html                      the entire app
data/rak/almarjan.json          master curated dataset
docs/reference/                 Al Marjan land map (developer/plot reference)
.github/workflows/pages.yml     GitHub Pages deploy (on push to the default branch)
.github/workflows/live-data.yml daily FX bake into data/live/fx.json, then redeploy
```

---

*This project never touches the Dubai repo (`SP_Dubai_Property_Map`); the Dubai frontend was cloned once as a read-only visual base.*
