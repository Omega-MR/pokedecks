# PokéDecks — Data Flow

All persistent data lives under `%TEMP%\PokeLog\`. Nothing is written outside this folder. No network requests are made except to TCGdex (and optionally pokemontcg.io in a future release).

---

## App Launch

```
App starts
  └─ Load /user/state.json
  │    → Restore collection, duplicates, column settings, set list, API keys
  │    → Set first visible set as active
  │
  └─ Load /sets/index.json (if exists) → populate set catalogue immediately
  └─ Fetch fresh index from TCGdex in background (always, not just on first launch)
       → On success: overwrite /sets/index.json
       → On failure: retain cached index, show sync error, retry after 30s
```

---

## User Adds a Set

```
User opens Add Set modal
  └─ Modal reads from in-memory setsIndex (populated from /sets/index.json)
  └─ Drill-down: Series → Language → Set
  └─ User clicks a set tile
       → Set entry written to managedSets[] in state
       → State saved to /user/state.json
       → Set becomes active, user navigated to Collection tab
```

---

## User Opens a Set

```
User selects a set (tab bar or Sets page)
  └─ Check /sets/{lang}/{series}/{id}.json exists
       → Exists: read file, render card table
       → Missing:
            └─ Fetch from TCGdex: GET /v2/{lang}/sets/{tcgdexId}
            └─ Shape response to per-set cache schema
            └─ Write to /sets/{lang}/{series}/{id}.json
            └─ Render card table
```

---

## Collection / Duplicate Update

```
User clicks a collection cell
  └─ Left-click cycles: 0 (empty) → 1 (owned) → 2 (wishlisted) → 3 (on order) → 1 → …
  └─ Right-click toggles N/A: non-4 → 4 (N/A); 4 → 0 (empty)
  └─ State updated in memory immediately (React state)
  └─ Write to /user/state.json
       (triggers on any change to collection, duplicates, visibleColumns, managedSets, apiKeys)

User increments / decrements a duplicate counter
  └─ Counter updated in memory immediately
  └─ Write to /user/state.json
```

---

## Variant Resolution (render time)

```
Active set selected
  └─ Look up setId in set-variants.json
       → Found: use that set's variant array
       → Not found: use full global variant list, default-visible = [Std, H, RH]
  └─ For each variant abbreviation: look up full name in variants.json
  └─ Intersect with visibleColumns[setId] from user state → active columns
  └─ Render card table with active columns only
```

---

## Export

```
JSON export
  └─ Read collection[setId], duplicates[setId], visibleColumns[setId] from state
  └─ Write to user-chosen .json file
  └─ Default filename: PokeDecks_{series}_{name}_{lang}_Backup.json

PDF export (Collection)
  └─ Temporarily set activeTab = "collection"
  └─ Trigger browser print dialog on the rendered card table
  └─ Print stylesheet hides UI chrome, renders cover page + full card table
  └─ Cover page summary: per-variant owned / not-owned counts (visible columns only; N/A excluded)
  └─ Restore previous activeTab on afterprint event

PDF export (Wishlist)
  └─ Temporarily set activeTab = "wishlist"
  └─ Same print flow as Collection
  └─ Rows with no visible variant at state 2 (wishlisted) are print:hidden
  └─ Cells not at state 2 are print:invisible (column structure preserved, only wishlist icons visible)
  └─ Cover page summary: per-variant wishlisted counts (only variants with count > 0)
  └─ Restore previous activeTab on afterprint event

PDF export (Duplicates)
  └─ Temporarily set activeTab = "duplicates"
  └─ Same print flow as Collection
  └─ Rows with no visible variant duplicate count > 0 are print:hidden
  └─ Cover page summary: per-variant duplicate totals
  └─ Restore previous activeTab on afterprint event
```

---

## Import

```
User selects a .json backup file
  └─ Parse file, read setId
  └─ Normalise collection values: boolean true → 1 (owned), false → 0 (empty)
       (handles backups created before v2.2.0)
  └─ Overwrite collection[setId], duplicates[setId], visibleColumns[setId] in state
  └─ If setId not in managedSets: add it automatically
  └─ Save state to /user/state.json
```

---

## Cache Clear

```
User clicks "Clear card cache" in Settings
  └─ Delete all language subdirectories under /sets/ (recursive)
  └─ /sets/index.json is NOT deleted
  └─ /user/state.json is NOT deleted (collection data preserved)
  └─ Card data re-fetched from TCGdex on next set selection
```

---

## Full Reset

Deleting `/user/state.json` resets the app to a clean state. The set catalogue (`/sets/index.json`) and all per-set card caches can be deleted independently — they will be re-fetched from TCGdex on next use.
