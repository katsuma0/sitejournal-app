# Site Journal — Project Handoff
This document is the full context for working on Site Journal. Read it before touching anything. It covers what the app is, why it exists, how its owner thinks, every feature, the design system, the writing voice, and the working rules that keep the project safe. Companion files in this repo: `CLAUDE.md` (hard technical rules), `DESIGN.md` (universal design system), `FORMAT-SPEC.md` and `SEARCH-SPEC.md` (pixel specs shared with the sibling fishing app).

---

## 1. What this is

Site Journal is a personal, phone-first web app for rating Ontario Provincial Park campsites. One person built it for himself: **Katsuma Onishi**, who books the camping trips for his friends and family and got tired of gambling on unknown sites. The app contains **every reservable Ontario Park** — 124 parks, 20,031 individual campsites (verified against reservations.ontarioparks.ca on 2026-07-21), ~211 trails — and lets him walk a campground, rate each site 0–5, jot a note, attach photos, wishlist the ones worth booking, and rate campgrounds, trails, and whole parks.

The origin story matters and is in the app itself: a camper named **Laurie** and her husband at Grundy Lake (she introduced them to Bowser, the famous snapping turtle of Gurd Lake) inspired the whole thing. There is a hidden dedication to her (`forlaurie` in the search bar). Grundy Lake is the emotional home of the app; its theme is never changed.

It is a single-file vanilla app: one `index.html` with all CSS and JS inline, park data embedded as JSON inside the page, no frameworks, no build step. It ships three ways: the GitHub Pages website (katsuma0.github.io/sitejournal), an installable PWA, and a native iOS app via Capacitor (`~/Projects/sitejournal-ios` on Katsuma's Mac, run through Xcode). All ratings live on-device: localStorage for data, IndexedDB for photos. There is no server, no account, no analytics, and that is a feature — the footer promises "Ratings and photos are saved to this device only."

## 2. How Katsuma thinks about it (read this twice)

- **It is a personal tool with craft.** He iterates in tiny, specific steps and cares about details most people never notice: whether a pill is vertically centred, whether a swipe edge has a seam, whether two "5/5" badges are the same size. When he says something "looks weird," he is almost always right and the cause is real.
- **Uniformity is the aesthetic.** One score-chip design everywhere. One row rhythm. Fractions, never percentages. When day-use parks got ratings, they got a `1/1` and a full progress bar specifically so a completed collection looks uniform.
- **Whimsy is allowed, but hidden.** The app's surface is calm and plain; the personality lives underground: a hidden park (Queen Elizabeth II Wildlands) discovered through search, secret console commands, a dedication page, a theme fraction that can read 124/123. Secrets are never documented in the user-facing Versions list — themes are not mentioned there at all, by explicit decision.
- **Honesty in data.** Site numbers are researched against real Ontario Parks maps; estimates carry a flag; drive times are curated for Toronto and honestly labelled estimates for other towns; a park with no road access says "No road access."
- **He wants to be taught.** When something breaks, he appreciates the diagnosis explained plainly (the service worker saga, the flexbox compression bug). Explain causes, not just fixes.
- **Motion should feel bubbly but subtle.** Small spring overshoots, press-scale on everything touchable, haptics on meaningful taps. Never enough bounce to expose an edge behind a page.

## 3. Feature inventory (v0.179)

**Home page:**
- Eyebrow "ONTARIO PARKS", title "Site Journal", tappable version number (opens Versions sheet), one-line intro.
- Global search: matches parks, campgrounds, sites ("Hemlock 112"), and trails, word-by-word with filler words ignored ("provincial", "park", "the"). Fixed 46×22 category bubbles: Park / Camp / Site / Trail.
- Three filter chips, text-sized with adaptive labels: **Region filter** (six broad regions plus Pinned), **Sort** (Default A-to-Z, Progress, Top rated, Closest to GTA with a Departure sheet that re-centres drive times on any of ~109 Ontario towns; label degrades "From Markham" → "Markham" when space is tight), **Group by** (None, Letter in five balanced sections, Park type: Car camping / Day use / Backcountry, Region — the default, Rated-first).
- Park cards: name, metadata line (region · sites · drive time when sorting by distance), rated fraction and progress bar once started, pin badge.
- Footer: device-only disclaimer, "Created by Katsuma Onishi" linking to katsuma0.github.io, "Export a backup" / "Import a backup" (whole journal as one JSON file: ratings, notes, photos, themes, settings; export goes through the iOS share sheet where available, falls back to a download; import validates the file and arms "Tap again to restore N ratings", then replaces this device's data and reloads), and "Reset all data" (two-tap arm-and-confirm).

**Park pages:** slide in from the right; interactive swipe-back that follows the finger (rounded left edge, spring cancel, exit matches the back button). Order: Park rating card → Campgrounds (collapsible, with site-chip grids coloured by score) → Wishlist → Top sites → Stats (hidden until a site is rated; tiles, rating-spread bars in score colours, per-campground progress) → Trails → its own footer with per-park reset. About section per park: blurb, facility chips, fishing info, an FMZ pill linking to the sibling app (`https://katsuma0.github.io/on-fishing-reg/#zone=N`), official park page link.

**Rating sheets:** 0–5 dot scale (48px targets), notes, photo attach (camera + library in the iOS app), wishlist toggle for sites.

**Themes:** 123 unlockable park themes plus Default. Unlocked by tapping a park's title on its page. All are light or medium with bold accents; the *only* dark theme belongs to the hidden park — deep pine, deliberately sophisticated. Themes for the first four alphabetical parks are exact recolours of Default. The fraction at the bottom of the Themes section is out of 123; finding the hidden park makes it 124/123. Katsuma is keen on themes remaining a discovered surprise.

**Hidden layer (never in the public changelog):**
- **Queen Elizabeth II Wildlands** — invisible on the home page until searched (any part of "queen…") and tapped; then permanent. Ganaraska Trail wilderness section inside.
- Search-bar commands: `debugsearch` (diagnostics console card), `statsearch` (lifetime stats, themes at top), `dummydata` (confirm button; plants a realistic left-skewed sample season near Markham, reshuffles cleanly on each press, notes are `dummytext1…`), `dummyhundop` (everything 5/5, all themes except the egg), `-dummyhundop` (everything 0/5, all locked), `forlaurie` (the serif-italic book dedication).

**Versions:** one-line entries, varied phrasing, milestones only, no theme mentions, opened from the version number beside the title.

## 4. Design system (the short version — DESIGN.md is law)

- Palette: paper `#F1F6F1`, card white, ink `#0F1F17`, accent forest `#00753A` (+ `#18A05A`), moss `#40564B` for secondary text, mist fills, line `#D3E0D4`, amber `#EFBF25` for wishlist only, destructive `#B0574A` as underlined links.
- Font: **Hanken Grotesk** everywhere; Georgia italics only for the dedication; monospace only for console cards. Inputs always 16px (iOS zoom). Tabular numerals for fractions.
- Geometry: 18px page gutters, 720px max width, radii 16/12/8, pills 99px, 10px row gap, 44pt effective tap targets (invisible expanded hit areas where visuals are smaller).
- Motion: parkIn `.36s cubic-bezier(.32,1.08,.38,1)`; sheets `.34s cubic-bezier(.32,1.22,.38,1)`; swipe-cancel spring `(.22,1.28,.36,1)`; `:active` scale ~.98; reduced-motion honoured.
- He is especially keen on: the fixed-size category bubbles being dead-centred vertically; the universal score chip; footers living inside each view so swipes carry them; nothing dark except the egg; the version number typeset like the title's small sibling.

## 5. Writing voice (this is part of the design)

- Sentence case everywhere; uppercase only via CSS on labels.
- **No em dashes, ever.** Commas and periods.
- Middots join metadata, segments Capitalized: `Central Park · Head Lake`, `3.5 km · Moderate`.
- Fractions not percentages: `41/95`, `123 / 123`.
- Toasts: short, declarative, period-terminated, two beats max: "Cleared. Fresh start for this park." "Hidden park revealed. Welcome to the Wildlands."
- Empty states: one warm, unapologetic sentence: "No trails listed for this park yet."
- Buttons name the action plainly: "Rate this park", "Tap again to erase everything".
- First person, casual, honest, a little dry: "there are some campsites (and campgrounds) out there that are not it." Never marketing words.
- Changelog entries: terse noun phrases, one line, varied (not "Added, Added, Added").

## 6. Hard working rules (violating these has burned us before)

1. **Version lockstep:** every change bumps `scout-vNNN` in `service-worker.js` AND the `v0.NNN` in the `.ver` button, together.
2. **Never rename storage keys:** `ontario-scout-v2` (main state), `site-journal-theme`, `site-journal-theme-vars`, `site-journal-unlocks`, `site-journal-sort`, `site-journal-group`, `site-journal-origin`, IndexedDB `scout-photos`.
3. **The service worker must never register inside Capacitor** (`window.Capacitor` check). It once cache-first-served a stale index.html forever.
4. **`refreshParksFromNetwork` returns immediately under Capacitor.** A stale `www/parks-data.json` once silently replaced 124 embedded parks with an old 123-park file. Embedded data is authoritative in the app.
5. **Copy BOTH `index.html` and `parks-data.json` into `~/Projects/sitejournal-ios/www/`** on every update, then `npx cap copy ios`. The iOS app is ONE app for both halves: the fishing build (its `index.html`, `manifest.json`, icons, and `fish/`) is bundled at `www/fishing/`, and both pages switch their cross links to the local copies when they detect Capacitor.
6. **Verify a patch actually applied before bumping or committing.** We have shipped phantom commits when a script aborted mid-way. Never make a view a flex container (it broke auto-margin centring and compressed every page).
7. Data changes go through the pipeline (`pipeline/build_db.py` → `export_to_app.py --all`) and get re-embedded minified into index.html.
8. Secrets (egg, commands, themes) never appear in the Versions list.
9. Test logic in a node harness before shipping when in doubt; `debugsearch` on-device is the ground truth for device mysteries.

## 7. Current state and open threads

At **v0.179**, everything above is live. Ratings backup (export-import JSON, v0.179) is built: the backup file carries every storage key raw plus all photos, and restoring preserves the `_s5` migration marker so scores never re-migrate. Open items, in the order Katsuma has cared about them:
- `color-scheme` wiring so the iOS scroll indicator (and keyboard) matches light pages and the one dark theme.
- A subtle affordance that the title is tappable (About/Themes are otherwise invisible).
- App Store Connect / TestFlight when he is ready; Info.plist already carries the camera ("Camera for sick flicks of the campsites.") and photo-library permission strings.
- The sibling app, the Ontario fishing regulation summary (`on-fishing-reg`), shares this design system; keep the two visually interchangeable.

Work in small steps, explain causes, protect his data, keep the secrets secret, and never let two "5/5" chips disagree about their size.
