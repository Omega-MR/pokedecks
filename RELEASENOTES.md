# PokéDecks — Release Notes

---

<details>
<summary><strong>v2.2.x</strong></summary>

### v2.2.1 — Legend & polish

- Moved the card state legend out of the inline controls bar and into the **Hover for Legend** modal, displayed as a left panel alongside the abbreviations with a vertical divider between them
- Version number now displayed in the sidebar beneath the app title
- "Right click to clear" note in the hover modal lightened to a more readable colour

---

### v2.2.0 — Multi-state collection tracking & Wishlist PDF

**Multi-state checkboxes**

- Collection cells now cycle through four states on left-click: **Owned** (green / checkmark) → **Wishlisted** (blue / heart) → **On order** (orange / clock) → **N/A** (barely visible) → back to Owned
- Right-click on any cell resets it directly to empty
- State legend integrated into the **Hover for Legend** hover modal alongside abbreviations (added in v2.2.1; in v2.2.0 it appeared inline in the controls bar)

**PDF export**

- Added a third PDF export option: **Wishlist** — prints only rows where at least one visible variant is marked Wishlisted
- Wishlist cover page summary shows per-variant wishlisted counts (visible columns only; only variants with at least one wishlisted card are listed)
- Collection cover page summary reworked to per-variant **Owned** / **Not owned** counts (state 1 = owned; states 0, 2, 3 = not owned; state 4 excluded from both totals; visible columns only)
- Print styling: state icons (checkmark, heart, clock) always print in black; N/A cells are fully invisible on print; empty cells render as a plain black outline

**Schema change**

- Collection values changed from boolean (`true`/`false`) to integers: `0` = empty, `1` = owned, `2` = wishlisted, `3` = on order, `4` = N/A
- Full backwards compatibility: existing state files and imported JSON backups with boolean values are automatically normalised on load (`true` → `1`, `false` → `0`)

</details>

---

<details>
<summary><strong>v2.1.x</strong></summary>

### v2.1.2 — Series index deduplication fix

**Series index**

- `seriesCode` field removed; display now derives directly from `seriesKey` (uppercased) so there is no intermediate mapping that can diverge
- `seriesKey` is now normalised to lowercase at index build time — TCGdex returns uppercase IDs (e.g. `"SV"`) for certain languages (Japanese, Korean, Indonesian, Thai, Chinese), which previously caused Scarlet & Violet and Sun & Moon to appear as two separate series entries in the Add Set modal
- JSON export default filename now uses `seriesKey` uppercased (e.g. `PokeDecks_SV_151_EN_Backup.json`) rather than `setPrintId`
- `SCHEMA.md` updated: `seriesCode` field removed, `seriesKey` description updated to note lowercase normalisation and uppercase display

---

### v2.1.1 — Series field naming & display format

**Series index**

- Renamed ambiguous `seriesId` field into two explicit fields: `seriesCode` (series-level shortcode) and `setPrintId` (the set's own printed card abbreviation from `abbreviation.official` in TCGdex)
- `SERIES_ABBREV` lookup table removed; series codes are now derived directly from `seriesKey` rather than a manually maintained map
- Series deduplication in Add Set modal simplified: single-pass by `seriesKey`, English display name preferred, no secondary name-based merge pass
- `SCHEMA.md` updated to reflect new field names and corrected folder path (`sets/{langCode}/{seriesCode}/` → `sets/{langCode}/{seriesKey}/`)

**Display**

- Tab bar top line now shows `SV - MEW` format (`seriesKey` uppercased + `setPrintId` if available)
- Sets page and delete confirmation now show `series - setPrintId - name` format (e.g. `Scarlet & Violet - MEW - 151`)

</details>

---

<details>
<summary><strong>v2.0.x</strong></summary>

### v2.0.2 — Settings page & export improvements

**Settings page**

- New Settings tab in sidebar (gear icon) with a dedicated full-page panel
- PokéTCG API key input (password field) — saved to `state.json` under `apiKeys.pokemonTcg`; used in the future to supplement TCGdex card data
- **Refresh sets index** button — manually refreshes the set index from TCGdex, overwriting the current `index.json` in `PokeLog/sets/`
- **Clear card cache** button — deletes all per-set `.json` files from `PokeLog/sets/` without touching collection data; shows a brief "Cleared ✓" confirmation on success

**Export modal redesign**

- Clicking **Export to PDF** now shows a sub-screen with two tiles: Collection and Duplicates — works regardless of which tab is currently active
- Chosen PDF view is rendered by temporarily switching `activeTab`, triggering `window.print()`, then restoring the previous tab via the `afterprint` event
- JSON export unchanged: still exports the full set (collection + duplicates)

**Set management UX fixes**

- First set added via the Add Set modal is automatically set as the active set — no longer requires manual tab navigation after adding
- Visibility slider on the Sets page is disabled (faded, `cursor-not-allowed`, tooltip) when only one visible set remains, preventing an all-hidden state; the set is still deletable

---

### v2.0.1 — Sets management page & modals

**Sets page**

- New **Sets** tab added to sidebar (library icon), replacing the old `+` top-bar button and Set Manager modal entirely
- Sets listed in user-defined order with drag-to-reorder via HTML5 drag-and-drop (hamburger handle); order persisted to `state.json`
- Visibility slider per set — toggles whether the set appears in the top bar tab strip; state persisted
- Delete button opens the Delete Set modal
- Clicking a set name marks it as active and navigates to the Collection tab
- Empty state shown when no sets have been added yet

**Add Set modal**

- Drill-down tile UI: **Series → Language → Set** (three levels)
- Series ordered by earliest release date within the series; Language alphabetical; Sets by release date
- Breadcrumb header shows current drill path with clickable back-navigation to earlier levels
- Already-added sets shown greyed out and non-interactive within the set tile grid
- Selecting a set appends `{id, visible: true, order}` to `managedSets` and closes the modal
- Currently sources tiles from the bundled JS `sets` array; will switch to `/sets/index.json` cache in Step 3

**Delete Set modal**

- Two-layer flow: option tiles first, then a per-option confirmation before any data is touched
- **Remove set** — removes from set list, silently attempts to delete `/sets/{id}.json` cache (no-ops if missing); collection data retained
- **Delete collection data** — wipes `collection`, `duplicates`, and `visibleColumns` for the set; set remains in list
- **Delete set entirely** — both of the above combined; confirm button is red
- Confirmation copy matches spec, with set name bolded in `SeriesId - Name (Language)` format
- **Back** button on confirmation returns to option tiles without executing

---

### v2.0.0 — Schema & data layer refactor

**Bundled data files**

- Added `public/data/variants.json` — global lookup of 9 variant abbreviations to full names
- Added `public/data/set-variants.json` — per-set list of applicable variant abbreviations; seeded with `sv151en`

**Persistence migration**

- Save path moved from `%TEMP%/poketracker_save.json` to `%TEMP%/PokeLog/user/state.json`
- On-disk format updated: `enabledSets[]` replaced by `sets[{id, visible, order}]`; `apiKeys` object added
- Auto-migration on first launch: reads old file → writes new format → deletes old file
- In-memory `enabledSets` is now a derived `useMemo` from `managedSets.filter(visible).sort(order)`

**Render-time variant resolution**

- `variants.json` and `set-variants.json` loaded at startup via `fetch()`
- `currentSetData` useMemo overlays resolved `variants` and `abbreviations` from bundled JSON onto the base set data; falls back to JS file data if JSON not yet loaded
- Variant key for Standard changed from `"base"` to `"Std"` to match `set-variants.json`; existing JSON backups must replace `"base"` with `"Std"` in `collection`, `duplicates`, and `visibleColumns` before importing

**Dynamic display names**

- `label` field removed from display logic; names composed at render time: tab bar shows `${seriesId} - ${name}` / `${language}`; PDF cover shows `${series} - ${name}` / `${language}`
- Export default filename uses composed name via `setDisplayName()` helper
- `sv151en.js` enriched with `series`, `seriesId`, `name`, `language`, `releaseDate` to support composed names ahead of Step 3 API integration

</details>

---

<details>
<summary><strong>v1.2.x</strong></summary>

### v1.2.2 — Duplicates PDF row filtering

**PDF export**

- Duplicates print now only includes rows where at least one variant has a count higher than 0 — empty rows are hidden via `print:hidden` and remain fully visible in the UI
- Filter checks all variants, including columns that are currently hidden in the column selector

---

### v1.2.1 — Full card list & set file refactor

**Set data**

- `sv151en.js` rewritten from a generated array to an explicit, fully annotated card list covering all 207 cards in official set order
- Cards grouped into labelled sections: Pokémon (001–151), Trainer Cards (152–165), Illustration Rares (166–184), Special Illustration Rares (185–199), and Hyper Rares / Gold Cards (200–207)
- Duplicate card names disambiguated with `(IR)`, `(SIR)`, and `(HR)` suffixes
- File structure documented as the canonical template for future set additions
- Removed stale `sv151.js` which was being picked up by the glob import alongside `sv151en.js`, causing the first-enable auto-activation logic to fail

</details>

---

<details>
<summary><strong>v1.1.x</strong></summary>

### v1.1.2 — Print cover page & variant naming

**PDF export — cover page**

- Print cover page added, appearing before the card list on both Collection and Duplicates exports
- Cover includes: app icon, app title, "Created by Omega-MR" byline, active set name, and tab name (Collection / Duplicates) in spaced uppercase beneath the set name
- Legend rendered on the cover page showing all abbreviation → full name mappings for the active set
- Collection cover includes a summary section: cards owned and missing on the left; per-variant owned counts on the right (full variant names, hidden columns included, only variants with at least one owned card shown)
- Duplicates cover includes a summary section: per-variant total duplicate counts distributed evenly across two columns (only variants with at least one duplicate shown)

**Variant naming**

- "Standard Collected" renamed to "Standard"; abbreviation changed from `Col` to `Std` across `sv151en.js`

</details>

---

<details>
<summary><strong>v1.0.x</strong></summary>

### v1.0.3 — UI polish & print fix

**UI**

- Sidebar label "Running Collection" renamed to "Collection"
- Duplicate controls redesigned: +/− buttons now stacked vertically to the right of the count, reducing column width significantly
- Column padding reduced: 60% less between card number and name, 40% less between name and first count column, 25% less between count columns (Duplicates tab only)
- Name column set to `whitespace-nowrap` so card names always render on a single line

**PDF export**

- Fixed print output being clipped to the visible viewport — the full card list now prints correctly by releasing the flex height chain (`h-screen`, `min-h-0`, `overflow-hidden`) on all ancestor containers via `print:` variants

---

### v1.0.2 — State persistence & startup flash fixes

**Bug fixes**

- Fixed state not persisting between sessions in the built portable executable — capability permissions were using `fs:allow-read` and `fs:allow-write` (binary file handle operations) instead of the correct `fs:allow-read-text-file` and `fs:allow-write-text-file`; save and load were silently failing
- Window background colour set to `#111827` (Tailwind `gray-900`) in `tauri.conf.json` to eliminate the white flash visible before the webview painted on startup

</details>
