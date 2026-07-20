# Site Journal — project guide for Claude

Personal Ontario Parks campsite journal built for Katsuma Onishi. He books campsites for family and friends; the app lets him walk a campground, rate sites 1-10, star wishlist sites, and attach notes/photos so he knows what to book next time. Mobile-first PWA, tested on iPhone, hosted via GitHub Pages, used offline in parks.

## Repo layout
- `index.html` — the entire app: HTML + CSS + vanilla JS + an embedded minified copy of the park data in `<script type="application/json" id="parks-data-embedded">`. No build system, no frameworks.
- `parks-data.json` — pretty-printed park data. The app boots from the embedded copy instantly, then refreshes from this file in the background.
- `service-worker.js` — offline cache. Cache name is `scout-vNN`.
- `manifest.json`, icons — PWA install.
- `pipeline/` — SQLite data pipeline (Katsuma is learning SQL; keep it approachable):
  - `schema.sql` — tables parks, campgrounds, sites (one row per site), trails.
  - `build_db.py` — all park data lives inline here (PARKS / CAMPGROUNDS / TRAILS / F facilities dicts). Running it rebuilds `scout.db`.
  - `export_to_app.py` — exports scout.db → parks-data.json (alphabetical by park name). `--all` includes unverified sites (currently everything). It also enforces the capitalization house style on campground descriptions.

## THE DEPLOY LOOP (run after ANY data change)
```
cd pipeline
python3 build_db.py
python3 export_to_app.py --all
cp parks-data.json ../parks-data.json
```
Then in `index.html`: replace the embedded JSON block with the new data, minified
(`json.dumps(data, ensure_ascii=False, separators=(',',':'))`), regex on
`<script type="application/json" id="parks-data-embedded">.*?</script>`.

After ANY change to index.html or service-worker.js: **bump the cache version**
(`scout-vNN` → `scout-vNN+1`) in service-worker.js, and validate the JS:
extract the plain `<script>` blocks (NOT the application/json one) to a temp file
and run `node --check` on it.

## Hard rules / house style
- **NO em dashes anywhere** (—). Use commas. This applies to code, data, and UI text.
- Comma/middot lists capitalize each segment's first letter ("All electric, Near the beach"). `export_to_app.py` enforces this for campground subs automatically.
- Region strings are `"<Broad> · <Town>"`. Broad buckets: `Northern Park`, `Near North Park`, `Algonquin` (plain, no "Park"), `Central Park`, `Southeast Park`, `Southwest Park`. Examples: `Near North Park · Britt`, `Algonquin · Highway 60`.
- **Never rename storage keys**: localStorage `ontario-scout-v2` (ratings/notes/wishlist), `site-journal-theme`, `site-journal-theme-vars`, `site-journal-unlocks`; IndexedDB `scout-photos`. Renaming loses user data.
- Fractions, never percentages, in all UI ("3/652 sites", not "0%").
- No `:hover` styles (they stick on touch). Press feedback via `:active` scale only.
- Text on accent-coloured backgrounds uses `var(--paper)`, never `#fff` (dark themes have light accents).
- **Spacing system** (use these, never ad-hoc pixel values): gaps come in exactly three sizes via tokens `--gap-s` (6px, micro), `--gap-m` (10px, standard: list rhythm, element gaps), `--gap-l` (12px, large clusters). Card interiors use `--pad-card` (14px 16px); sheet rows, inputs, and list rows use `--pad-row` (12px 14px). Stacked cards/rows separate with `margin-bottom:var(--gap-m)`; sections separate with 20-22px. Pills are 8px 13px, small chips 7px 12px, primary buttons 10px 14px. All park-page boxes (About toggle, campground rows, trail cards, stat tiles, wishlist items, top-site rows) share one height token `--row-h` (64px) with contents vertically centred; top-site rows stay borderless with hairline separators. Radii come in exactly three tokens plus two shapes: `--r` (16px, cards and sheets), `--r-sm` (12px, rows, inputs, buttons, dots), `--r-xs` (8px, tiny badges and photo thumbs), `99px` for pills, `50%` for circles. Never introduce a raw radius value.
- **Touch targets**: every interactive control is at least 44px tall (Apple HIG). Interactive pills (`.fchip`, `.cg-rate`) are min-height 44px / padding 10px 16px / 13px text; display-only pills (`.tr-rate`) may be smaller. Primary action buttons are min-height 48px. Small tap areas get padding+negative-margin expansion rather than layout changes (see `.backbtn`).

## App behaviours to preserve
- Home = alphabetical park list; search bar matches parks / campgrounds / "Hemlock 112" / trails.
- Swipe RIGHT anywhere on a park page = back to All parks. Rating + settings sheets swipe DOWN to close (drags starting in a textarea are ignored; horizontal swipes ignored); background scroll locks while a sheet is open.
- Parks with exactly one campground open that campground by default.
- Site chips: number is the main element; rated = heatmap colour + score badge bottom-right; wishlist star top-left (amber); note OR photo = small star top-right; note AND photo = medallion (circle with a negative-space star, `MEDAL_SVG`). Trails use the same star/medallion system, top-right of the card.
- Progress bars / stats stay hidden until at least one site in that park is rated.
- Park page hierarchy: section labels (`.seclabel`, e.g. Campgrounds / Trails) are `var(--ink)`, never the accent; campground and trail names carry the accent (`var(--forest)`); the About card matches campground-card styling but its title is `var(--ink)` (only tappable campground/trail names carry the accent). The park-page top ribbon reads `< All parks`. For Algonquin split parks the page h2 reads `Algonquin · {Name}` (h2 remains the theme-unlock tap target); the About toggle reads `About {Name}`; the campground header shows its real name.
- Rating sheet: for sites the green context line reads "Campground · Park" (for the Algonquin split parks, where campground name == park name, it reads "Algonquin · <name>").

## Theme system (unlockables)
- Tap "Site Journal" title → settings sheet: "Theme" heading, swatch list, bare fraction counter (e.g. "3/29"). List has natural height up to 7 rows, then fixed 396px with inner scroll.
- Default theme is named `Default` (id `forest`, defined in `:root`). Every park has exactly one theme; **theme name = park name exactly**, theme id = park id.
- Unlock: tapping the park's name (h2) on its page → toast "✦ <Name> theme unlocked · Tap to use". Grundy Lake's first unlock appends "· Shout out Laurie" (easter egg, keep forever).
- Palettes are generated from 3 seeds `{paper, ink, primary, dark?}` via `buildVars()` (mixhex). Dark themes set `dark:true` (light ink, dark shadows). Alphabetically early parks = light themes; later = dark/moodier. Keep every theme unique.
- Rating heatmap (`STOPS`, clay→green) is tinted ~14% toward the active theme via `color-mix(in srgb, <stop> 86%, var(--forest))` in `scoreColor()`. Don't hardcode score colours elsewhere.
- **Every new park batch must add a matching theme** in `PARK_THEMES` (the x/NN counter updates automatically).

## Data status (v57)
- 58 parks, ~15,318 sites, 150 trails. Second batch (next-tier busiest, GTA outward): Earl Rowe, Rock Point, Bass Lake, McRae Point, Craigleith, Selkirk, Port Burwell, Point Farms, Inverhuron, Sauble Falls. Near-exact anchors: Sauble Falls East numbering (100-154), Bass Lake per-campground counts, published barrier-free site numbers at Earl Rowe (90/102/230/242), Bass Lake (5/6), McRae Point (51/53), Turkey Point (81/212). Craigleith and Inverhuron have no in-app trails (none confirmed in-park). All sites `verified=0` (estimates) EXCEPT ranges corrected from official Ontario Parks campground-map PDFs for Grundy Lake, Killbear, Killarney George Lake (those PDFs are text-extractable only for "Northeast Zone" maps; southern parks' maps are image-only).
- Algonquin is split into 9 top-level parks (one per car campground): ids `algonquintea, algonquincanisbay, algonquinmew, algonquintworivers, algonquinpog, algonquinkearney, algonquinraccoon, algonquinrock, algonquinachray`, names "Tea Lake", "Canisbay", etc. Corridor trails were assigned to the nearest campground by KM. A one-time JS migration (`migrateAlgonquin` / `migrateAlgPhotos`) moves any old `algonquin#` keys — keep it in place.
- The fishing chip links to `https://katsuma0.github.io/onfishingreg/#zone=NN` (zone parsed from the park's fmz field).

## Park roadmap
Katsuma's full top-30 list is COMPLETE as of v56, plus a second batch of the next 10 busiest southern parks in v57 (Lake Superior, Turkey Point, Emily, Mara, Bonnechere, Mikisew, Rondeau, Neys, Chutes, and Fairbank all added with themes). Batchawana Bay was skipped: it is a day-use-only park with no campsites. If more parks are requested later: research campground names/site ranges via web search; mark estimates `verified=0`; blurbs get the park's character + location, no em dashes; add a matching theme (light for early alphabet, dark for later); region format `"Broad Park · Town"` (Algonquin plain). Site numbering for most 2024+ additions is estimated; northern parks (Lake Superior, Neys, Chutes, Fairbank, Mikisew) may have text-readable Northeast Zone campground-map PDFs for a future verification pass.

## Skipped / declined (don't re-suggest)
- Campground map images/PDFs in the app (too heavy for now).
- Percentages anywhere. Hover styles. Export/import backup buttons.
