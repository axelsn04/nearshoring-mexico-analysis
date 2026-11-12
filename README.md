# Nearshoring: Real or Re-routed?

**Is Mexico's rise in US imports a genuine relocation of production, or mostly a side effect of China's decline?**

An end-to-end data project (extraction, cleaning, analysis and modelling) on monthly US import data, 2017–2025.
IS507E – Programming for Data Analytics, Rennes School of Business, November 2026.

---

## Why this project: a Mexican perspective

We are two Mexican students. In financial circles back home, nearshoring is pitched as Mexico's opportunity of the decade:
companies leaving China, factories moving to Mexico, and booming demand for industrial real estate, manufacturing and logistics.

We wanted to test that pitch against the data. Is the shift of US supply chains toward Mexico really happening?
How big is it, and in which products? If nearshoring is the opportunity, where exactly is it?

## The issue

In 2023, Mexico overtook China as the largest source of US goods imports, and "nearshoring" became the dominant story
about Mexico's economy. But a headline crossing can happen for very different reasons:

- companies genuinely moving production from China to Mexico (nearshoring);
- China shrinking, with many countries filling the gap and Mexico only one of them;
- Mexico growing in sectors it already dominated (autos, since NAFTA), unrelated to China;
- Chinese goods passing through Mexico on their way to the US (re-routing).

This project uses trade data to separate these explanations.

## Why these countries

| Partner | Role in the analysis |
|---|---|
| **United States** | The market: the world's largest importer and Mexico's main export destination. All figures are US imports. |
| **Mexico** | The subject: the country the nearshoring narrative is about. |
| **China** | The country losing share: the source of the supply-chain shift. |
| **Taiwan** | The control: the other major "China+1" beneficiary. If Taiwan gains as much as Mexico in the same products, the shift is not Mexico-specific. |
| **Rest of world** | Everything else (world total minus the three), so no part of China's lost share goes unaccounted for. |

## What we expected

| # | Hypothesis | Supported if… |
|---|---|---|
| H1 | Mexico's gains are concentrated in a few products | A handful of products explain the net gain |
| H2 | Mexico substitutes China | Mexico gains share in the products where China loses it |
| H3 | It is "China+1", not Mexico-specific | Taiwan and other countries absorb as much as, or more than, Mexico |
| H4 | Part of it is re-routing | Chinese exports to Mexico rise in the products Mexico ships to the US (not testable with this data; kept as a limitation) |

## The short answer

**Real, but narrower than the narrative, and partly cyclical.** Mexico captured about a fifth of the share China lost
in US imports (2017–19 vs 2023–25), and that proportion holds even without 2025. But the gain is concentrated in
18 products, Taiwan and other countries absorbed far more of China's losses, and the most recent surge is driven by
computers and the AI-server boom. Full findings: [`docs/report.md`](docs/report.md).

## Where to find everything

| What | Where | Contents |
|---|---|---|
| Report | [`docs/report.md`](docs/report.md) | Why it matters, findings by hypothesis, investment implications, limitations |
| Methodology | [`docs/methodology.md`](docs/methodology.md) | Pipeline, tools, metrics, models and the decisions behind them |
| Team | [`TEAM.md`](TEAM.md) | Who we are and what each member did |
| Extraction | [`notebooks/01_extract.ipynb`](notebooks/01_extract.ipynb) | US Census API → raw JSON (36 calls) |
| Cleaning and loading | [`notebooks/02_transform_load.ipynb`](notebooks/02_transform_load.ipynb) | Raw JSON → clean tables in CSV and SQLite |
| Analysis and models | [`notebooks/03_analysis_models.ipynb`](notebooks/03_analysis_models.ipynb) | Hypothesis tests, charts, K-means, regression, Random Forest, conclusions |
| Clean data | [`data/clean/`](data/clean/) | Monthly, annual and product-level tables (CSV) |
| Database | `data/nearshoring.db` | The same tables in SQLite |
| Data dictionary | [`data/DATA_DICTIONARY.md`](data/DATA_DICTIONARY.md) | Every table and column, with units |
| Reference data | [`data/reference/sector_map.csv`](data/reference/sector_map.csv) | HS chapter → sector mapping |
| Charts | [`figures/`](figures/) | The 8 charts used in the analysis |

## How to run it

1. Install Python 3.12 and the dependencies:
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate        # Windows: .venv\Scripts\activate
   pip install -r requirements.txt
   ```
2. To re-download the data, get a free US Census API key at https://api.census.gov/data/key_signup.html
   and save it in a `.env` file in the project root: `CENSUS_KEY=your_key`.
3. Run the notebooks in order: `01_extract` (about 10–20 minutes), `02_transform_load`, `03_analysis_models`.

The clean CSVs and the SQLite database are included, so `03_analysis_models` runs on its own without an API key.

## Data source

US Census Bureau, International Trade API: monthly general imports by HS4 product and partner country, 2017–2025.
[API guide](https://www.census.gov/foreign-trade/reference/guides/Guide_to_International_Trade_Datasets.pdf).

## Team

Axel Hernández Sanchez Navarro (data engineering) and Jorge Paulin (data analysis). Details in [`TEAM.md`](TEAM.md).

*This is an academic analysis of trade data, not investment advice.*
