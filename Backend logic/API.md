# PokéDecks — API Integration

Card data is sourced exclusively from [TCGdex](https://tcgdex.dev). No API key is required. All calls are unauthenticated GET requests.

---

## Endpoints Used

### Series list
```
GET https://api.tcgdex.net/v2/{lang}/series
```
Returns all series for the given language code. Used as the first pass during index build to discover series IDs and groupings.

### Series detail
```
GET https://api.tcgdex.net/v2/{lang}/series/{seriesId}
```
Returns series metadata including an ordered list of set stubs. Used to determine set order within each series and the series-level release date fallback.

### Set detail
```
GET https://api.tcgdex.net/v2/{lang}/sets/{setId}
```
Returns full set metadata and the card list. Called twice in the application lifecycle:

1. **During index build** — fetched for every set in every language to obtain the set's own `releaseDate`, `abbreviation.official`, and whether the `cards` array is non-empty (`hasCards`). Card details are discarded at this stage.
2. **On first set selection** — fetched when a user opens a set that has no local cache. The full `cards` array is kept and written to the per-set cache file.

---

## Index Build Strategy

The index is built in two phases on every app launch (and on manual refresh):

1. Fetch all series for each supported language to collect set IDs and their series context.
2. Fetch every individual set to get accurate per-set metadata.

Requests are dispatched using a worker-pool pattern with a concurrency limit of 5 simultaneous requests to avoid overwhelming the API.

Sets where the `cards` array is empty are flagged `hasCards: false` in the index and cannot be added by the user.

---

## Supported Languages

Language support is driven by a bundled `languages.json` file. The application iterates all entries and fetches sets for each. If TCGdex has no data for a given language/set combination the request returns a non-200 status and is silently skipped.

Current entries: English, French, German, Spanish, Italian, Portuguese, Japanese, Korean, Chinese (Traditional), Indonesian, Thai, Polish, Russian, Dutch.

---

## Error Handling

Network and filesystem errors are handled separately:

- **Network failure** — sets `syncError: "network"`. App displays "Card database unreachable — check your connection" and retries automatically after 30 seconds.
- **Write failure** — sets `syncError: "local"`. App displays "Couldn't save data — check your device" and retries automatically after 30 seconds.

All errors are appended to `%TEMP%\PokeLog\log\api.log` with a timestamp for local diagnostics.

---

## Planned: PokéTCG API (pokemontcg.io)

A secondary data source is planned as a stretch goal. The user supplies their own API key via the Settings modal. It will be used to supplement TCGdex data only — filling gaps where TCGdex coverage is incomplete. The key is stored in `%TEMP%\PokeLog\user\state.json` under `apiKeys.pokemonTcg`. The input field is present in the current build but disabled pending implementation.
