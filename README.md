# PokéDecks

A portable Windows desktop app for cataloguing your Pokémon TCG collection and duplicates. No account, no cloud, no internet required to use your saved data. Card lists are fetched automatically from [TCGdex](https://tcgdex.dev) and cached locally.

Distributed as freeware. Run the signed `.exe` — no installation required.

---

## Roadmap

| Status  | Feature                                                                                                                                                                                    |
| ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| ✅ Live | Collection & duplicate tracking                                                                                                                                                            |
| ✅ Live | TCGdex card data (automatic, all sets across all supported languages)                                                                                                                      |
| ✅ Live | JSON backup / restore                                                                                                                                                                      |
| ✅ Live | PDF export                                                                                                                                                                                 |
| Planned | **PokéTCG API integration** — supplement TCGdex card data using your own [pokemontcg.io](https://pokemontcg.io) API key, filling gaps in card listings where TCGdex coverage is incomplete |
| Planned | Variant data embedding — mapping of per-set card variants from PriceCharting, so every set shows accurate variant columns out of the box                                                   |

---

## Getting Started

1. Download `pokedecks.exe`.
2. Run it. Windows SmartScreen may warn on first run — this is expected for a self-signed executable. To suppress the warning on your machine, install the bundled `pokedecks.cer` into your Trusted Publishers store via `certmgr.msc`.
3. On first launch the app fetches the full set catalogue from TCGdex in the background. You will see **Downloading set list** in the bottom-left of the sidebar while this runs. Once complete it shows **Up to date**.
4. Click **Sets** in the sidebar, then **Add Set** to add your first card set.

All data is stored in `%TEMP%\PokeLog\` and persists across restarts.

---

## Interface Overview

The sidebar on the left drives navigation. The main area changes based on what you have selected.

### Sidebar tabs

| Tab            | What it does                                                        |
| -------------- | ------------------------------------------------------------------- |
| **Collection** | Multi-state view — mark cards as owned, wishlisted, on order, or N/A per variant |
| **Duplicates** | Counter view — track how many duplicates you hold per variant       |
| **Sets**       | Manage your added sets (add, reorder, show/hide, delete)            |
| **Settings**   | API keys and cache controls (opens as a modal)                      |

### Bottom of sidebar

| Element        | Description                                                    |
| -------------- | -------------------------------------------------------------- |
| **Sync indicator** | Shows current download status or any connection/storage errors |
| **Import**     | Restore a JSON backup file for a set                           |
| **Export**     | Export the active set to JSON or PDF                           |

---

## Managing Sets

### Adding a set

1. Go to **Sets** → click **Add Set**.
2. Drill down through three levels: **Series → Language → Set**.
3. Click a set tile to add it. Sets you have already added are greyed out. Sets with no card data in the TCGdex database are also greyed out and cannot be added.
4. The new set is added to the bottom of your list and becomes the active set.

While the set catalogue is downloading, a note appears at the top of the modal. You can still navigate the tiles using any sets already loaded.

### Reordering sets

Drag the grip handle (≡) on the left of any set row to reorder. The order here controls the order of the tabs in the top bar.

### Showing and hiding sets

Each set row has a toggle slider. Hidden sets do not appear in the top bar but retain all their collection data. You cannot hide the last visible set, and a maximum of **5 sets** can be visible at once.

Clicking a hidden set's name navigates to it temporarily — it appears in the top bar for that session only. Selecting a different set in the top bar immediately removes the temporary set from the bar.

### Deleting a set

Click the trash icon on a set row. You are presented with three options:

| Option                     | What is deleted                                                                                              |
| -------------------------- | ------------------------------------------------------------------------------------------------------------ |
| **Remove set**             | Removes from your list and clears cached card data. Collection data is kept — re-adding the set restores it. |
| **Delete collection data** | Wipes your collection, duplicates and column settings for this set. The set stays in your list.              |
| **Delete set entirely**    | Everything — the set, the cache, and all collection progress. Cannot be undone.                              |

Each option requires a confirmation step before anything is deleted.

---

## Tracking Your Collection

Select a set using the top tab bar, then use the **Collection** or **Duplicates** tab in the sidebar.

### Collection tab

Each row is a card. Each column is a variant (Standard, Holo, Reverse Holo, etc.). Each cell is a multi-state box — click to cycle through states, right-click to reset to empty.

| State | Appearance | Meaning |
| ----- | ---------- | ------- |
| Empty | Plain outline | Not tracked |
| **Owned** | Green box with checkmark | You have this card |
| **Wishlisted** | Blue box with heart | You want this card |
| **On order** | Orange box with clock | You have ordered this card |
| **N/A** | Barely visible box | Not applicable for this card |

Hover over the **Hover for Legend** label (top-left of the card table) to see both the state legend and the full abbreviation key for the active set side by side. The **Columns** button (top-right) lets you show or hide individual variant columns — hidden columns are excluded from PDF summaries.

### Duplicates tab

Same layout as Collection. Use the **+** and **−** buttons to increment or decrement how many duplicates you hold for each card/variant combination. Rows with no duplicates are hidden when printing.

### Variants and columns

Not all sets have all variants. Sets with a known variant mapping show only the variants available for that set. Sets without a mapping fall back to all global variants, with only **Standard**, **Holo** and **Reverse Holo** visible by default.

Use the **Columns** button (top-right of the card table) to show or hide individual variant columns. Your column preferences are saved per set.

Hover over the **Hover for Legend** label to see the full abbreviation key for the active set.

---

## Import and Export

### Export to JSON

Exports the active set's collection, duplicates, and column settings to a `.json` backup file. Use this to back up your progress or transfer it between devices.

### Import from JSON

Imports a previously exported `.json` file. The data is matched by `setId` — it will overwrite the data for whichever set is in the file, regardless of which set is currently active. A warning is shown before the overwrite proceeds.

If the imported set is not yet in your set list, it is added automatically.

### Export to PDF

Exports the active set to a print-ready layout via your system print dialog (save as PDF from there). You choose one of three views to print:

| View | What is printed |
| ---- | --------------- |
| **Collection** | Full card table with all states shown |
| **Wishlist** | Only rows where at least one visible variant is marked Wishlisted |
| **Duplicates** | Only rows where at least one visible variant has a duplicate count > 0 |

Each export includes a cover page with the set name, list type, variant legend, and a summary:

- **Collection** — per-variant owned and not-owned counts (only visible columns; N/A cards excluded from both totals)
- **Wishlist** — per-variant wishlisted counts (only variants that have at least one wishlisted card)
- **Duplicates** — per-variant duplicate totals

State icons (checkmark, heart, clock) always print. Background colours print only if "Print background graphics" is enabled in your browser's print dialog. The N/A state is fully invisible on print. Empty boxes render as a plain black outline.

---

## Settings

### PokéTCG API Key

Reserved for the upcoming PokéTCG API integration (see Roadmap). Paste your [pokemontcg.io](https://pokemontcg.io) key here when that feature ships. Leave blank for now — the app functions fully on TCGdex without a key.

### Refresh sets index

Manually re-fetches the full set catalogue from TCGdex and overwrites the local cache. This includes all sets across all supported languages. The catalogue is also refreshed automatically every time the app launches, so this button is mainly useful if you need a mid-session refresh.

### Clear card cache

Deletes all per-set card list files from `%TEMP%\PokeLog\sets\`. Your collection data is not affected. Card data will be re-fetched from TCGdex the next time you open each set.

---

## Sync Status Indicator

The small indicator at the bottom of the sidebar shows the current state of background downloads.

| Indicator                                 | Meaning                                                                                                                           |
| ----------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| Spinning blue circle + label              | A download is in progress                                                                                                         |
| Green checkmark — **Up to date**          | Everything is current, no activity                                                                                                |
| Red cross — **Card database unreachable** | Could not reach TCGdex. Check your internet connection. The app will retry automatically every 30 seconds.                        |
| Red cross — **Couldn't save data**        | Data was fetched but could not be written locally. Check available disk space. The app will retry automatically every 30 seconds. |

If you see a persistent error, check `%TEMP%\PokeLog\log\api.log` for the detailed error message.

---

## Local Storage

All app data lives under `%TEMP%\PokeLog\`:

| Path                             | Contents                                                            |
| -------------------------------- | ------------------------------------------------------------------- |
| `user/state.json`                | Your collection, duplicates, column settings, set list and API keys |
| `sets/index.json`                | Cached set catalogue fetched from TCGdex                            |
| `sets/{lang}/{series}/{id}.json` | Cached card list for each set you have opened                       |
| `log/api.log`                    | Error log for API and file system issues                            |

Deleting `user/state.json` resets the app to a clean state. The set catalogue and card caches can be safely deleted — they will be re-fetched on next use.

---

## Credits

Created by **Omega-MR**. Card data provided by [TCGdex](https://tcgdex.dev).
