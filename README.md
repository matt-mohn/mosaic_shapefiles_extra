# Mosaic — extra state shapefiles

Ready-to-use precinct shapefiles for [Mosaic](https://matt-mohn.github.io/mosaic_python/), with population, race/age demographics, and recent election results pre-joined.

Each `<State>_DRA_Mosaic.zip` contains a single TIGER/Line VTD shapefile with 14 attributes:

| Column | Type | Meaning |
|---|---|---|
| `GEOID` | string | 11-char VTD GEOID20 (state FIPS + county FIPS + VTD) |
| `CTY` | string | 3-char county FIPS |
| `POP_TOTAL` | int | 2020 total population |
| `VAP_TOTAL` | int | 2020 voting-age population |
| `VAP_WHITE` | int | 2020 VAP, White alone, not Hispanic |
| `VAP_BLACK` | int | 2020 VAP, Black any-part (incl. Hispanic) |
| `VAP_LATINO` | int | 2020 VAP, all Hispanic regardless of race |
| `VAP_ASIAN` | int | 2020 VAP, Asian any-part (incl. Hispanic) |
| `TRUMP_16` / `CLINTON_16` | int | 2016 presidential R / D votes |
| `TRUMP_20` / `BIDEN_20` | int | 2020 presidential R / D votes |
| `TRUMP_24` / `HARRIS_24` | int | 2024 presidential R / D votes |

Full schema notes and definitional choices: see [`DATA_LICENSE.md`](DATA_LICENSE.md).

## Per-state notes

- **Estimated 2024 results** (Arkansas, Connecticut, Maine, Michigan, New Jersey, Oklahoma, Oregon, Pennsylvania) — these files swap `TRUMP_24` / `HARRIS_24` for `DT_EST_24` / `KH_EST_24` / `EST_24_FLG`. The estimates come from applying each county's reported 2020 → 2024 swing uniformly across its precincts, so county and state sums match true 2024 totals exactly. Per-precinct values are modeled — treat with caution.
- **Island-precinct deletions** (California, New York, Rhode Island) — offshore precincts that are disconnected from the rest of the state's polygon graph have been removed, because Mosaic's ReCom requires a connected precinct adjacency graph (this is also why Hawaii is not included). The affected populations are small.
- **Florida** — the shapefile is accurate for use in Mosaic, but it will not round-trip cleanly back into Dave's Redistricting App due to precinct changes in a few counties. A fix is in progress.

## License & use restrictions

These shapefiles are joins of two upstream sources:

1. **TIGER/Line 2020 VTD shapefiles** — U.S. Census Bureau (public domain).
2. **Precinct-level demographic and election data** — [Dave's Redistricting App](https://davesredistricting.org) (DRA).

Because they incorporate DRA-packaged data, the joined shapefiles inherit DRA's license:

- **Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)** — https://creativecommons.org/licenses/by-sa/4.0/
- **You may not sell this data under any circumstances** (DRA's explicit no-sale clause).
- Any redistribution must be under a compatible ShareAlike license and must attribute DRA and the upstream election-data providers (VEST, RDH, Census Bureau).

Mosaic itself is MIT-licensed; this data is not. Use accordingly.

See [`DATA_LICENSE.md`](DATA_LICENSE.md) for the full attribution table, upstream provider list, and schema choices baked into the join.
