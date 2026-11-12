# Data dictionary

## Source and coverage

- **Source:** US Census Bureau, International Trade API, endpoint `timeseries/intltrade/imports/hs`.
- **Measure:** general imports, monthly value in US dollars (`GEN_VAL_MO`), not adjusted for inflation.
- **Products:** 1,238 HS4 codes (4-digit Harmonized System headings).
- **Partners:** Mexico (`MX`), China (`CN`), Taiwan (`TW`), world total (`WORLD`); rest of world (`ROW`) is derived.
- **Period:** January 2017 – December 2025.
- **Raw data:** 36 JSON files (one per partner-year), 462,984 records. Not included in the repo; rebuilt by `01_extract.ipynb`.

## Periods

| Period | Years | Use |
|---|---|---|
| `baseline` | 2017–2019 | Before the trade war fully hit |
| `pandemic` | 2020–2021 | Kept in the data, excluded from comparisons |
| `transition` | 2022 | Kept in the data, excluded from comparisons |
| `recent` | 2023–2025 | Compared with the baseline |

## `clean/imports_hs4_monthly.csv.gz` (SQLite: `imports_monthly`)

One row per partner × product × month (534,816 rows). Months without trade are filled with 0.

| Column | Description | Unit |
|---|---|---|
| `partner` | `MX`, `CN`, `TW` or `WORLD` | – |
| `hs4` | Product code, zero-padded (e.g. `0101`) | – |
| `date` | First day of the month | date |
| `value_usd` | US general imports from the partner | USD |
| `year` | Calendar year | – |

## `clean/imports_hs4_annual.csv` (SQLite: `imports_annual`)

One row per product × year (11,142 rows).

| Column | Description | Unit |
|---|---|---|
| `hs4` | Product code | – |
| `year` | Calendar year | – |
| `MX`, `CN`, `TW`, `WORLD` | US imports from each partner | USD |
| `ROW` | Rest of world: `WORLD − MX − CN − TW` | USD |
| `share_MX`, `share_CN`, `share_TW`, `share_ROW` | Partner imports ÷ world imports for that product | 0–1 |
| `period` | `baseline`, `pandemic`, `transition` or `recent` | – |

## `clean/hs4_features.csv` (SQLite: `hs4_features`)

One row per product (1,238 rows), comparing baseline (2017–19) and recent (2023–25) values summed over three years.
In the columns below, `{p}` stands for `MX`, `CN`, `TW` or `ROW`.

| Column | Description | Unit |
|---|---|---|
| `hs4`, `hs4_desc` | Product code and description (latest year) | – |
| `hs2`, `sector` | HS chapter and sector | – |
| `share_{p}_base`, `share_{p}_recent` | Partner's share of US imports of the product in each period | 0–1 |
| `d_share_{p}_pp` | Recent share minus baseline share | percentage points |
| `contrib_{p}_pp` | Contribution to the partner's aggregate share change (sums across products to the total) | percentage points |
| `within_{p}_pp` | Part of the contribution from the partner's share change inside the product | percentage points |
| `mix_{p}_pp` | Part of the contribution from the product growing or shrinking in total imports | percentage points |
| `gain_{p}_bn_yr` | Average annual change in imports from the partner (recent minus baseline, ÷ 3) | USD bn per year |
| `world_base_bn`, `world_recent_bn` | Total US imports of the product in each period (three-year totals) | USD bn |
| `world_cagr` | Annual growth of US imports of the product between the two periods (six years between midpoints) | rate |
| `material` | `True` if baseline imports are at least USD 100M; used to filter share analysis | boolean |
| `hs_revision_flag` | `True` if the code exists in only one period (2022 HS revision) | boolean |

## `reference/sector_map.csv` (SQLite: `sector_map`)

One row per HS2 chapter. Chapters are grouped into 13 sectors by chapter range (e.g. 01–24 Agri-food, 84 Machinery,
85 Electrical & electronics, 87 Vehicles & parts).

| Column | Description |
|---|---|
| `hs2` | HS chapter (2 digits) |
| `hs2_desc` | Chapter description from the Census API |
| `sector` | Sector assigned by the project |

## `nearshoring.db`

A SQLite database (about 23 MB) with the four tables above: `imports_monthly`, `imports_annual`, `hs4_features`,
`sector_map`. Example query:

```sql
SELECT year, ROUND(SUM(MX)/1e9, 1) AS mexico_bn, ROUND(SUM(CN)/1e9, 1) AS china_bn
FROM imports_annual
GROUP BY year
ORDER BY year;
```
