# Winsol Territory Manager

A single-file, static HTML web app (map + partner territory manager for Belgium).
No build step, no dependencies to install — Vercel serves `index.html` as-is.

## Files in this repo

- **`index.html`** — the app itself
- **`postcode-boundaries.geojson`** — the actual map shapes, one polygon per 4-digit Belgian
  postal code (built from Statbel's official statistical sectors, dissolved by postcode). This
  replaced an earlier municipality-level shape file, so postal codes that share a town (e.g.
  Beringen's 3580/3581/3582/3583) now render as separate regions instead of one big blob.
- **`postcode-reference.xlsx`** — the default postcode → partner reference list. The app downloads
  this file every time it loads and uses it as the built-in data, unless someone has uploaded their
  own file in the "Reference" tab (which stays local to their browser).
- **`config.json`** — shared partner order/merge settings from the Admin panel (see below)
- **`README.md`** — this file

## Deploy on Vercel

1. Push this repo to GitHub (`index.html`, `postcode-boundaries.geojson`, `postcode-reference.xlsx`, `config.json`, this file).
2. Go to [vercel.com/new](https://vercel.com/new) and **Import** this GitHub repo.
3. Leave all settings as default (Framework Preset: **Other** / static) and click **Deploy**.
4. Vercel gives you a live URL like `https://your-project.vercel.app` — open/share that link (not the raw file) so it loads correctly on all devices, including iOS.

## Updating the app

Whenever you get a new version of `index.html`, just replace the file in this repo and push —
Vercel redeploys automatically and the same URL stays live with the update.

## Updating the map shapes (postcode-boundaries.geojson)

This file rarely needs to change — Belgium's postal code boundaries don't move often. If Statbel
publishes a newer statistical-sectors dataset in the future and you want to refresh it, send me
the new files (statistical sectors GeoJSON + the address-to-sector file) the same way as before,
and I'll regenerate this file using the same pipeline (sector → postcode majority match → dissolve
→ simplify).

## Updating the postcode/partner reference list

`postcode-reference.xlsx` is just a normal Excel file with these columns: `Land`, `Postcode`,
`Gemeente`, `Naam` (partner name), `Postbussen`, `Provincie`, `#2024`, `#2025`, `#2026`, `Temp`,
`Partner Type`, `Go or NoGo`, `Expansie`, `VF 2025`, `VF 2024`, `VF 2023`, `VF_2025 as B2B`,
`VF_2024 as B2B`, `VF_2023 as B2B`, plus a few optional admin columns Winsol already uses.

The `Partner Type` column powers the **Type** tab — any values that appear in it show up there
as selectable filters. Rows with the type **"Winsol concept store"** are always rendered in
yellow on the map, regardless of selection, so those locations stand out visually.

The `Go or NoGo`, `Expansie`, and `VF...` columns power the **Expansion** tab:
- `Expansie` is the zone name a postcode belongs to.
- `Go or NoGo` marks whether that specific postcode counts as part of the zone's visualization
  (a zone can mix Go and NoGo postcodes — only the Go ones are shown/colored on the map).
- Selecting one or more zones in the Expansion tab colors each zone differently on the map and
  shows a resumé (zone name, postcode count, #2025/#2024 leads, and VF 2025/2024 as B2B).
- With no zone selected, the map behaves exactly as before (colored by partner).

To update it:

1. Open `postcode-reference.xlsx` in Excel (download it from GitHub, edit, or edit directly on
   GitHub.com for small tweaks).
2. Make your changes — add/remove rows, change partner assignments, update yearly numbers, etc.
3. Save it, keeping the same filename (`postcode-reference.xlsx`) and column headers.
4. Replace the file in this GitHub repo and push.
5. Vercel redeploys automatically. Every visitor now sees the updated data on their next visit
   (their browser re-downloads this file fresh each time the app loads).

If a visitor has previously uploaded their **own** file through the app's "Reference" tab, that
personal upload takes priority in their browser until they clear it — this file is only the
shared default everyone else sees.

## Applying Admin changes for every visitor

The app's **Admin** panel (password `Admin123`) lets you reorder partners and merge duplicate
partner names (e.g. "BREBO" + "BREBO INVEST" into one). Those changes are saved to your own
browser only by default — a colleague opening the same link won't see them.

To make Admin changes apply to **everyone**:

1. In the Admin panel, save your partner order and/or merges as usual (so they're active in your browser).
2. Click **"Download config.json"** at the bottom of the Admin panel — this exports exactly what
   you just saved.
3. In this GitHub repo, replace `config.json` with the downloaded file, and push.
4. Vercel redeploys automatically. From then on, every visitor (with no local changes of their
   own) will see your saved order and merges by default.

`config.json` looks like this:

```json
{
  "partnerOrder": ["Some Partner", "Another Partner"],
  "partnerGroups": [
    { "name": "BREBO", "members": ["BREBO|", "BREBO INVEST|"] }
  ]
}
```

You can also hand-edit this file directly if you prefer, instead of using the Admin panel's
download button — just keep the same structure.
