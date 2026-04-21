# European Defense Investment Screener
**SE370 — Computer Aided Systems Engineering | West Point**
Cadets David Kendig & Noah Ross | Spring 2026

---

## Project Overview
This tool ranks European countries by their attractiveness as defense investment targets,
combining military spending trends, GDP growth, and the strength of each country's defense
industry. The output is a scored, ranked dataset that feeds a Streamlit map application.

---

## Repository Structure

```
├── data_collection.ipynb           # Main data pipeline (run this first)
├── nyt_article_scraper.ipynb       # NYT Archive API scraper (NLP dataset, separate)
├── SIPRI_Companies_2024_Clean.csv  # SIPRI Top 100 defense companies
├── API_MS_MIL_XPND_GD_ZS_...csv   # World Bank: military expenditure % of GDP
├── GDP_percent.csv                 # World Bank: GDP growth by country
└── europe_defense_data.csv         # OUTPUT: final merged and scored dataset
```

---

## Data Sources

| # | Source | Method | What it provides |
|---|--------|--------|-----------------|
| 1 | [SIPRI Top 100](https://www.sipri.org/databases/milex) | CSV download | Top defense companies, country, total revenue |
| 2 | [World Bank — Mil. Expenditure](https://data.worldbank.org/indicator/MS.MIL.XPND.GD.ZS) | CSV download | Military spending as % of GDP (1960–2025) |
| 3 | [World Bank — GDP Growth](https://databank.worldbank.org/source/world-development-indicators) | CSV download | Average GDP growth by country |
| 4 | [Trading Economics](https://tradingeconomics.com/country-list/military-expenditure?continent=europe) | Web scrape (`pd.read_html`) | Military expenditure in USD millions |
| 5 | [Open-Meteo Geocoding API](https://geocoding-api.open-meteo.com/v1/search) | API call (`requests`) | Latitude & longitude per country |

---

## Pipeline — `data_collection.ipynb`

Running the notebook top to bottom produces `europe_defense_data.csv`.

1. **SIPRI** — Keep Company, Country, Total Revenue. Filter to Europe. Aggregate to one row per country: top 3 companies by revenue (comma-separated) and their average revenue.
2. **Military Expenditure % GDP** — Average the 2015–2024 columns into a single `Avg_MilExp_Pct_GDP` value per country.
3. **GDP Growth** — Keep Country and `GDP_Avg_Growth` only.
4. **Web Scrape** — Fetch the Trading Economics HTML table with `pd.read_html()`, keep Country and `MilExp_USD_Millions`.
5. **API Call** — Loop through all 43 European countries, call the Open-Meteo geocoding endpoint, collect `Latitude` and `Longitude`.
6. **Merge** — All five datasets merged on `Country` using `pd.merge()`.
7. **Fix Missing Data** — `fillna(0)` applied to all four numeric scoring columns.
8. **Scoring** — Each country is ranked 1–39 in four categories. The four ranks are summed into `Total_Score` and the rank columns are dropped, leaving original values intact.

---

## Output Schema — `europe_defense_data.csv`

| Column | Description |
|--------|-------------|
| `Country` | Country name |
| `Avg_MilExp_Pct_GDP` | Average military spend as % of GDP (2015–2024) |
| `GDP_Avg_Growth` | Average GDP growth rate |
| `Top_3_Companies` | Top 3 defense companies by revenue (comma-separated) |
| `Avg_Top3_Revenue_M` | Average total revenue of those 3 companies (USD millions) |
| `MilExp_USD_Millions` | Most recent military expenditure in USD millions |
| `Latitude` | Country centroid latitude |
| `Longitude` | Country centroid longitude |
| `Total_Score` | Composite investment score (sum of 4 category ranks, max 156) |

---

## Scoring Methodology
Each of the four numeric columns is ranked 1–39 (lowest to highest value = lowest to highest score). The four ranks are summed into `Total_Score`. Higher is better. Equal weighting is applied across all categories.

| Category | Column | Max Points |
|----------|--------|-----------|
| Military spend (% GDP) | `Avg_MilExp_Pct_GDP` | 39 |
| GDP growth | `GDP_Avg_Growth` | 39 |
| Military spend (USD) | `MilExp_USD_Millions` | 39 |
| Defense industry revenue | `Avg_Top3_Revenue_M` | 39 |
| **Total** | `Total_Score` | **156** |

> **Note:** Countries with no SIPRI company data score 0 in the revenue category.
> This will be improved in a future update with a hard-coded fallback company list.

---

## Dependencies
```
pandas
requests
time
```
Install with: `pip install pandas requests`

---

## Planned Next Steps
- Streamlit app with Folium map heatmap (David)
- Hard-coded fallback companies for the 29 countries not in SIPRI top 100 (Noah)
- Investment portfolio recommendation output
