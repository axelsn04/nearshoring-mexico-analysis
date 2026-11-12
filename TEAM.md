# Team

| Member | Role |
|---|---|
| **Axel Hernández Sanchez Navarro** | Data engineering: ETL pipeline and code |
| **Jorge Paulin** | Data analysis: interpretation of results and charts |

## What each member did

### Axel Hernández Sanchez Navarro: data engineering
- Built the extraction pipeline against the US Census International Trade API: tested the API, identified partner codes,
  designed the call strategy (one call per partner-year to avoid timeouts) and saved the raw JSON files
  (`01_extract.ipynb`).
- Built the cleaning and transformation pipeline: type casting, HS code standardisation, duplicate and outlier checks,
  the zeros-vs-missing treatment, the sector mapping, the annual tables, market shares and product features
  (`02_transform_load.ipynb`).
- Loaded the clean tables into CSV and a SQLite database and validated the totals against published Census figures.
- Wrote the code for the analysis and models (`03_analysis_models.ipynb`).
- Set up and maintained the GitHub repository and the project environment.

### Jorge Paulin: data analysis
- Analysed the results against the four hypotheses and interpreted what each finding means for the nearshoring narrative.
- Helped design and review the charts used in the analysis.

## How we worked

- **Split by pipeline stage:** one member owned the data pipeline, the other the analysis, so we never edited the same
  notebook at the same time.
- **Version control:** all work was committed to this GitHub repository, with a pull-before-work, push-after-work routine.
- **Checkpoints:** each stage was validated before moving on (row counts, totals against Census figures, shares ≤ 100%,
  contributions summing to the aggregate) so errors were caught early.
