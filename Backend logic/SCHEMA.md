# PokéDecks — Schema Definitions

All files live under `%TEMP%\PokeLog\` unless noted as bundled.

---

## Bundled Data Files

These files are compiled into the application binary and are not user-editable.

### `variants.json`
Global lookup of variant abbreviations to full names. Source of truth for legend rendering and PDF export.

```json
{
  "Std": "Standard",
  "H":   "Holo",
  "RH":  "Reverse Holo",
  "CH":  "Cosmos Holo",
  "RHC": "Reverse Holo Cosmos",
  "MB":  "Master Ball",
  "PC":  "Pokemon Center",
  "EB":  "EB Games",
  "GS":  "GameStop"
}
```

### `set-variants.json`
Maps set IDs to their available variant abbreviations. Sets not present here fall back to the full global variant list with only Std, H, and RH visible by default.

```json
{
  "sv151en": ["Std", "H", "RH", "CH", "RHC", "MB", "PC", "EB", "GS"],
  "sv1en":   ["Std", "H", "RH"]
}
```

### `languages.json`
Language codes and display names iterated during index build.

```json
[
  { "code": "en", "name": "English" },
  { "code": "fr", "name": "French" }
]
```

---

## Cached Files (`%TEMP%\PokeLog\`)

### Sets Index — `sets/index.json`
Written on every app launch after fetching from TCGdex. Drives the Add Set modal.

```json
[
  {
    "id":          "sv03.5en",
    "seriesKey":   "sv",
    "series":      "Scarlet & Violet",
    "setPrintId":  "MEW",
    "name":        "151",
    "language":    "English",
    "langCode":    "en",
    "releaseDate": "2023-09-22",
    "sortKey":     "2023-09-22_2023-03-31_004",
    "hasCards":    true
  }
]
```

| Field | Description |
|---|---|
| `id` | App-internal set ID: TCGdex set ID + language code |
| `seriesKey` | TCGdex internal series ID — language-agnostic, used as the deduplication key and folder name. Displayed in upper case. |
| `series` | Series display name (English) |
| `setPrintId` | Set's own printed card abbreviation (`abbreviation.official` from TCGdex), or `null` if not supplied |
| `releaseDate` | Set's own release date in `YYYY-MM-DD` |
| `sortKey` | Composite sort key: `{setDate}_{seriesDate}_{orderInSeries}` |
| `hasCards` | `false` if the TCGdex cards array was empty — set cannot be added |

### Per-Set Cache — `sets/{langCode}/{seriesKey}/{id}.json`
Written on first selection of a set. Card list only — no variant data.

```json
{
  "id":          "sv03.5en",
  "series":      "Scarlet & Violet",
  "seriesKey":   "sv",
  "setPrintId":  "MEW",
  "name":        "151",
  "language":    "English",
  "releaseDate": "2023-09-22",
  "cards": [
    { "id": 1,   "name": "Bulbasaur" },
    { "id": 2,   "name": "Ivysaur" },
    { "id": 165, "name": "Mewtwo ex (SIR)" }
  ]
}
```

Card `id` is the card's printed collector number as an integer. Secret rares and special illustration rares continue the integer sequence beyond the set's advertised total.

---

## User State — `user/state.json`

```json
{
  "sets": [
    { "id": "sv03.5en", "visible": true,  "order": 0 },
    { "id": "sv1en",    "visible": false, "order": 1 }
  ],
  "visibleColumns": {
    "sv03.5en": {
      "Std": true,
      "H":   true,
      "RH":  true,
      "MB":  false
    }
  },
  "collection": {
    "sv03.5en": {
      "1": { "Std": 1 },
      "6": { "Std": 1, "RH": 2 }
    }
  },
  "duplicates": {
    "sv03.5en": {
      "3": { "Std": 2, "RH": 1 }
    }
  },
  "apiKeys": {
    "pokemonTcg": ""
  }
}
```

### Collection card states

Each value in the `collection` map is an integer representing the state of that card/variant cell:

| Value | State | Description |
| ----- | ----- | ----------- |
| `0` | Empty | Not tracked (default; key may be absent) |
| `1` | Owned | Card is in the collection |
| `2` | Wishlisted | Card is wanted |
| `3` | On order | Card has been ordered |
| `4` | N/A | Not applicable for this card |

**Backwards compatibility:** Files written by versions prior to 2.2.0 used boolean values (`true` / `false`). On load and import, `true` is automatically promoted to `1` (Owned) and `false` to `0` (Empty).

---

## Export Format — JSON Backup

```json
{
  "setId": "sv03.5en",
  "collection": {
    "1": { "Std": 1 },
    "6": { "Std": 1, "RH": 2 }
  },
  "duplicates": {
    "3": { "Std": 2, "RH": 1 }
  },
  "visibleColumns": {
    "sv03.5en": {
      "Std": true,
      "H":   true,
      "RH":  true,
      "MB":  false
    }
  }
}
```

`collection` values use the same numeric state encoding as `user/state.json` (see table above). Files exported by versions prior to 2.2.0 used boolean values and are automatically normalised on import.

Import is matched by `setId`. Importing a backup overwrites the data for that set regardless of which set is currently active. If the set is not yet in the user's list it is added automatically.

---

## Error Log — `log/api.log`

Plain text, one line per entry, appended on every API or filesystem error.

```
[2024-11-03 14:22:01] fetchSetsIndex:network: TypeError: Failed to fetch
[2024-11-03 14:22:31] loadSetData(sv03.5en): HTTP 404 Not Found for set "sv03.5"
```
