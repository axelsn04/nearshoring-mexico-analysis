# Methodology

How the analysis works, and why each decision was made.

## 1. Pipeline

| Notebook | Input | What it does | Output |
|---|---|---|---|
| `01_extract.ipynb` | US Census International Trade API | Tests the API, looks up partner codes, builds the HS2 sector map, downloads monthly imports by HS4 product for Mexico, China, Taiwan and the world total, 2017–2025 (36 calls), and validates totals against published figures | `data/raw/*.json`, `data/reference/sector_map.csv` |
| `02_transform_load.ipynb` | Raw JSON | Cleans, reshapes and enriches the data; computes shares, period features, contributions and the within/mix split | `data/clean/*.csv`, `data/nearshoring.db` |
| `03_analysis_models.ipynb` | SQLite database | Summary statistics, hypothesis tests, charts, K-means, regressions, Random Forest, robustness check, conclusions | `figures/*.png` |

## 2. Tools

| Tool | Used for | Why |
|---|---|---|
| `requests`, `json` | Calling the API and saving responses | Standard HTTP client; raw files saved so the project never depends on the API again |
| `python-dotenv` | Reading the API key from `.env` | Keeps the key out of the code and out of GitHub |
| `pandas`, `numpy` | Cleaning, reshaping, pivots, joins, features | The standard tools for tabular data |
| `sqlite3` | Storing the clean tables and querying them with SQL | A real database in one file, no server needed |
| `matplotlib`, `seaborn` | All charts | Full control over layout (matplotlib) and statistical charts such as heatmaps (seaborn) |
| `scikit-learn` | K-means, scaling, silhouette score, linear regression, Random Forest, cross-validation | The standard machine-learning library |
| `statsmodels` | Weighted regressions with p-values | Gives coefficients, p-values and R², which scikit-learn does not |
| `git`, GitHub | Version control and delivery | Shared history between team members, one place for the deliverables |

## 3. Key decisions

**Shares instead of dollars.** US imports from Mexico grew 71% in dollars between 2017 and 2025, but prices and total
US imports grew too. A partner's share of US imports removes inflation and market growth, and answers the real question:
did Mexico gain relative to everyone else?

**Three-year periods.** Baseline 2017–19 (before the trade war fully hit) vs recent 2023–25, each summed over three
years so that a single large shipment cannot distort the result. 2020–21 (pandemic) and 2022 (transition) are kept in
the data but left out of the comparison.

**Rest of world.** World total minus Mexico, China and Taiwan. Because the four shares always add up to 100%, their
changes always add up to zero: every point China loses is accounted for.

**Zeros vs missing values.** The API only returns product-months with trade. A missing record means no trade, so the
full partner × product × month grid is built and filled with 0 (71,832 zeros added).

**Materiality filter.** Products with less than USD 100M of US imports over the baseline are excluded from share
analysis: in tiny markets one shipment can move a share by 50 points. The filter removes 20% of products but only 0.7%
of trade value.

**HS revisions.** Ten product codes exist in only one period because the 2022 Harmonized System revision created,
split or merged them. Their share change reflects reclassification, not trade, so they are flagged and excluded.

**Weighting.** Correlations and regressions are weighted by trade value, so computers count far more than tin scrap.

## 4. Metrics

**Share change (pp).** A partner's share of US imports of a product in the recent period minus its share in the
baseline, in percentage points. Useful per product, but blind to size.

**Contribution (pp).** Imports of the product from the partner as a share of *total* US imports, recent minus baseline.
It weights each product by its size, and the contributions of all products add up exactly to the partner's aggregate
share change, which notebook 02 verifies.

**Within / mix split (shift-share decomposition).** The contribution is split in two:
- *Within:* the partner's share change inside the product, at the product's baseline weight. This is the
  competitiveness signal.
- *Mix:* the product growing or shrinking as a part of total US imports (TVs getting cheaper, for example).

## 5. Models

**K-means clustering.** Groups products by trajectory using five standardised features: Mexico's, China's and Taiwan's
share changes, product size (log) and demand growth. Outliers are capped at the 1st and 99th percentiles. K = 5 was
chosen by the silhouette score; its level (0.21) means the segments describe a continuum rather than sharply separated groups.

**Weighted regression.** One product per row: partner's share change = β × China's share change. β reads as "for each
point China loses, the partner gains β points". Run for Mexico, Taiwan and the rest of the world, the three βs must sum
to −1. A second version adds sector dummies to compare products within sectors. Results are associations, not causes.

**Linear regression vs Random Forest.** Both try to predict Mexico's share change from product characteristics, scored
with 5-fold cross-validated R², unweighted and weighted. This answers the course question of how different models behave
on the same data. Both score below zero, so feature importances are not reported.

**Robustness check.** The key numbers are recomputed with 2022–24 as the recent period, to test whether the results
depend on 2025.

## 6. Issues found and fixed

| Issue | How it was found | Fix |
|---|---|---|
| API timeouts on large calls | Census documentation | One call per partner-year, with retries and resume |
| The API repeats a column used as a filter (`CTY_CODE`) | `pd.concat` failed with a duplicate-index error | Duplicate columns dropped before concatenating |
| Share rankings dominated by tiny markets (boats, tin scrap) | First top-10 made no economic sense | Materiality filter and the value-weighted contribution metric |
| `pd.cut` made the recent period 2022–25 instead of 2023–25 | Contributions contradicted share changes for cars | Explicit period function; baseline and recent both span three years |
| Negative contributions where Mexico gained share (TVs) | Losers table | Within/mix decomposition |
| Results sensitive to the "legacy" threshold | Threshold test from 20% to 50% | Reported as a range (25–74%) instead of a single number |
