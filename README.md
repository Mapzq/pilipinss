# Philippines PSGC Dataset — Q3 2026

## Source
Philippine Statistics Authority (PSA), *Philippine Standard Geographic Code (PSGC), 3rd Quarter 2026*.
Publication date: 30 September 2026.
Official source page: https://psa.gov.ph/classification/psgc
Official source workbook: https://psa.gov.ph/system/files/scd/PSGC-3Q-2026-Publication-Datafile.xlsx

The original PSA workbook is included unchanged as `PSGC-3Q-2026-Publication-Datafile.xlsx`.
Please acknowledge PSA as the source when redistributing or using this dataset.

## Included files
- `philippines_locations.json` — complete normalized hierarchy in one JSON object (`records` array).
- `regions.json` — 18 region records.
- `provinces.json` — 82 province records.
- `cities_municipalities.json` — 149 cities, 1,493 municipalities, 14 Manila sub-municipalities, and 2 special administrative parent nodes.
- `barangays.json` — 42,010 barangay records.
- `philippines_locations.csv` — flat version of all records for spreadsheet review/import.
- `validation_report.json` — automated count, uniqueness, and parent-link checks.
- `README.md` — this guide.
- `PSGC-3Q-2026-Publication-Datafile.xlsx` — unchanged PSA source workbook.

## Important data conventions
1. Every `psgc_code` is a **string** of ten digits. Do not convert it to a number; doing so can destroy leading zeros.
2. Each ordinary child record has `parent_psgc_code`, pointing to the immediate parent record.
3. `geographic_level` values are `Reg`, `Prov`, `City`, `Mun`, `SubMun`, `Bgy`, and `SpecialArea`.
4. The two `SpecialArea` records are PSA hierarchy nodes, not ordinary provinces or LGUs:
   - `0990100000` — City of Isabela (Not a Province), parent of City of Isabela City.
   - `1999900000` — Special Geographic Area, parent of eight BARMM SGA municipalities.
5. Manila's 14 sub-municipalities are retained. Manila barangays are linked to the applicable sub-municipality.
6. PSA's headline counts are 18 regions, 82 provinces, 149 cities, 1,493 municipalities, and 42,010 barangays. The extra 14 sub-municipalities and 2 special nodes are additional hierarchy records and are not included in those headline totals.

## JSON record shape
Each record contains:
- `psgc_code` (string)
- `name`
- `correspondence_code` (string where supplied)
- `geographic_level`
- `parent_psgc_code` (string or null)
- `old_names`
- `city_class`
- `income_classification`
- `urban_rural`
- `population_2024`
- `status`
- `administrative_note`

## Validation
See `validation_report.json` for machine-readable results. A passing report means:
- PSA headline counts match the source workbook.
- No duplicate 10-digit PSGC codes occur in the exported hierarchy records.
- Every child record has an existing parent.
- No barangay is orphaned.
- Parent levels conform to the expected hierarchy.

## FlutterFlow / Firebase integration guidance
For a free-tier, low-cost implementation, do **not** create one Firestore document per barangay unless you have a strong reason. A dataset this large is better treated as static reference data.

Recommended approach:
1. Keep the JSON files in your app's bundled assets if your FlutterFlow project supports importing/reading local JSON assets on the target platforms. Verify FlutterFlow's current asset/JSON parsing workflow for your project before implementing.
2. Load the data once and filter in memory:
   - Region dropdown: records where `geographic_level == "Reg"`.
   - Province dropdown: `parent_psgc_code == selectedRegionCode` and `geographic_level == "Prov"`.
   - City/Municipality dropdown: `parent_psgc_code == selectedProvinceCode` and level is `City` or `Mun`. For region-administered cities/municipalities, filter by selected region. Include the special parent nodes where the chosen flow needs City of Isabela or BARMM SGA.
   - Barangay dropdown: `parent_psgc_code == selectedCityMunicipalityCode` (or selected Manila sub-municipality).
3. Store the selected PSGC code in the user's event/address record, not just the display name. Store both code and label if convenient.
4. Clear all downstream selections when an upstream selection changes.
5. If FlutterFlow cannot conveniently load a bundled JSON asset on all target platforms, host a compact read-only JSON file on a static HTTPS host/CDN. This avoids Firebase Storage, Cloud Functions, and thousands of Firestore writes. Confirm the host's access rules and long-term URL stability before relying on it.

This dataset is a static snapshot of Q3 2026. Check PSA for a newer quarterly release before using it for a production system.
