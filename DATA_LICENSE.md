# Ready/ data — provenance & license

The shapefiles in this folder (`<State>_DRA_Mosaic.shp` and their sidecar files) are joins of two upstream sources:

1. **TIGER/Line 2020 VTD shapefiles** — U.S. Census Bureau (public domain, no attribution required).
2. **Precinct-level demographic and election data** — Dave's Redistricting App (DRA), https://davesredistricting.org, exported per-state as `precinct-data.csv`.

## License

Per the `LICENSE.txt` distributed with each DRA export:

> Copyright (c) 2026 Social Good Fund for Dave's Redistricting.
>
> This data was packaged, and may have been disaggregated and/or aggregated, by Dave's Redistricting. The data itself came from other sources as noted.
>
> Use of this data is subject to the **Creative Commons Attribution-ShareAlike 4.0 International License**. Specific datasets may be subject to additional restrictions. (https://creativecommons.org/licenses/by-sa/4.0/)
>
> **YOU AGREE NOT TO SELL ANY OF THIS DATA UNDER ANY CIRCUMSTANCES.**

Because these shapefiles incorporate DRA-packaged data, they inherit **CC BY-SA 4.0** plus DRA's explicit **no-sale** restriction. Downstream use must attribute DRA and the upstream election-data providers (below), and any redistribution must be under a compatible ShareAlike license.

## Upstream attributions (per DRA's `LICENSE.txt`)

| Year(s) | Field        | Source                                                |
|---------|--------------|-------------------------------------------------------|
| 2020    | demographics | U.S. Census Bureau (decennial PL 94-171)              |
| 2016    | elections    | Voting and Election Science Team (VEST) — CC BY 4.0   |
| 2020    | elections    | Voting and Election Science Team (VEST) — CC BY 4.0   |
| 2024    | elections    | Redistricting Data Hub (RDH)                          |

- VEST: https://election.lab.ufl.edu/precinct-data/ , https://dataverse.harvard.edu/dataverse/electionscience
- RDH: https://redistrictingdatahub.org/data/about-our-data/election-results-and-precinct-boundaries/ , https://redistrictingdatahub.org/terms-and-conditions/

## Schema choices made by this pipeline

These are definitional decisions baked into the joined output, not encoded in DRA's column names alone:

1. **`VAP_BLACK` and `VAP_ASIAN` use the "any-part" definition** — i.e. Black/Asian alone *or* in combination with other races, *including* Hispanic. This matches DRA's columns `V_20_VAP_Black` / `V_20_VAP_Asian`. The any-part definition is the convention used in Voting Rights Act Section 2 analysis (BVAP / AVAP). DRA also publishes alone-not-Hispanic variants, which this pipeline does **not** use.

2. **`VAP_WHITE` is "White alone, not Hispanic"** — DRA's `V_20_VAP_White`.

3. **`VAP_LATINO` is "All Hispanics regardless of race"** — DRA's `V_20_VAP_Hispanic`.

4. **`POP_TOTAL` is the 2020 decennial census total population** — DRA's `T_20_CENS_Total`. Not the incarcerated-adjusted variant.

5. **Split-precinct rows** — DRA exports represent precincts split across district lines with `GEOID20 = B_<parent_GEOID20>_<unique_id>`. This pipeline aggregates split rows back to their parent VTD by summing all retained numeric columns, so each output row corresponds to exactly one TIGER VTD polygon.

6. **GEOID set match is strict.** If the DRA GEOID set (after split aggregation) does not match the TIGER VTD GEOID set exactly, the state is **flagged in `join_manifest.csv` and no shapefile is written** for it. We do not silently drop or zero-fill mismatched rows.

## Output schema

Most `<State>_DRA_Mosaic.shp` files contain these **14 attributes** plus geometry:

| Column      | Type    | Meaning                                                  |
|-------------|---------|----------------------------------------------------------|
| GEOID       | string  | 11-char VTD GEOID20 (state FIPS + county FIPS + VTD)     |
| CTY         | string  | 3-char county FIPS (positions 3-5 of GEOID)              |
| POP_TOTAL   | int     | 2020 total population                                    |
| VAP_TOTAL   | int     | 2020 voting age population                               |
| VAP_WHITE   | int     | 2020 VAP, White alone not Hispanic                       |
| VAP_BLACK   | int     | 2020 VAP, Black any-part                                 |
| VAP_LATINO  | int     | 2020 VAP, Hispanic (any race)                            |
| VAP_ASIAN   | int     | 2020 VAP, Asian any-part                                 |
| TRUMP_24    | int     | 2024 presidential — Republican (Trump) votes             |
| HARRIS_24   | int     | 2024 presidential — Democratic (Harris) votes            |
| TRUMP_20    | int     | 2020 presidential — Republican (Trump) votes             |
| BIDEN_20    | int     | 2020 presidential — Democratic (Biden) votes             |
| TRUMP_16    | int     | 2016 presidential — Republican (Trump) votes             |
| CLINTON_16  | int     | 2016 presidential — Democratic (Clinton) votes           |

### Estimated-2024 schema (15 attributes)

Eight states ship with an alternate 2024 schema because precinct-level 2024 results were not available at the time of build: **Arkansas, Connecticut, Maine, Michigan, New Jersey, Oklahoma, Oregon, Pennsylvania**. In those files, `TRUMP_24` and `HARRIS_24` are replaced by:

| Column       | Type    | Meaning                                                                |
|--------------|---------|------------------------------------------------------------------------|
| DT_EST_24    | int     | 2024 R (Trump) — county-swing estimate                                 |
| KH_EST_24    | int     | 2024 D (Harris) — county-swing estimate                                |
| EST_24_FLG   | string  | Estimate flag (currently always `scaled`)                              |

Construction: each county's reported 2020→2024 D/R swing is applied uniformly to every precinct in that county. **County-level and state-level sums equal true reported 2024 totals.** Per-precinct values are modeled — treat with caution for any analysis that depends on within-county variation.

### Island-precinct deletions

Three states have precincts removed from the geometry because they are physically isolated from the rest of the state's polygon graph (offshore islands), which Mosaic's ReCom cannot handle: **California, New York, Rhode Island**. Each affects a small population. Hawaii is excluded from the repo entirely for the same reason.

CRS is preserved from the TIGER source: **NAD83 (EPSG:4269)**.

## Join manifest

`join_manifest.csv` in this folder records, per input DRA zip: file, state, status (`OK` / `FLAGGED` / `ERROR`), and detail.
