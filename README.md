# Nearshoring: Real or Re-routed?

Is Mexico's rise in US imports a structural relocation of production, or mostly diversion away from China?
IS507E – Programming for Data Analytics, Rennes School of Business.

## Data
US Census Bureau International Trade API — monthly US imports by HS4 product, 2017–2025,
for Mexico, China, Taiwan and the world total.

## Structure
- `notebooks/01_extract.ipynb` – API pull → `data/raw/`
- `notebooks/02_transform_load.ipynb` – cleaning → `data/clean/` + SQLite
- `notebooks/03_analysis_models.ipynb` – EDA, charts, models

## How to run
1. `pip install -r requirements.txt`
2. Get a free Census API key and save it in `.env` as `CENSUS_KEY=...`
3. Run the notebooks in order.

## Team
- Axel Hernández – data engineering (extract, transform, load)
- Jorge Paulín – analysis and models