# Real Estate USA — Data Analysis Project

An end-to-end data analysis project covering US residential real estate sales. The pipeline runs from raw data ingestion and cleaning in BigQuery SQL through exploratory analysis to interactive Power BI dashboards.

---

## Project Overview

The dataset contains residential property listings and sales records across the northeastern United States, spanning from the early 1990s through 2023. The analysis focuses on sales volume, pricing trends, property characteristics, and geographic distribution across states and cities.
<img width="1914" height="1072" alt="Screenshot 2026-03-20 140324" src="https://github.com/user-attachments/assets/8b932fd9-1ecd-4daf-a0b9-dfe8352d0b49" />
**Tools used:** Google BigQuery, SQL, Power BI


---

## Repository Structure

```
├── Data_cleaning.sql               # Full data cleaning pipeline in BigQuery SQL
├── Exploring_data.sql              # Exploratory queries by state, city, bedrooms, bathrooms
├── Quered_data_property_sold.csv   # Aggregated data exported for Power BI (by year)
├── Quered_data_year_and_month.csv  # Aggregated data exported for Power BI (by year and month)
├── Real_Estate_USA_Dashboards.pbix # Power BI report file
└── Real-Estate_USA_Dashboards.pdf  # Dashboard export (static)
```

---

## Data Cleaning

Performed in `Data_cleaning.sql` using Google BigQuery. Key steps:

- **Removed duplicates** — the raw dataset was reduced by approximately 9x after deduplication using `SELECT DISTINCT` and window functions (`ROW_NUMBER() OVER PARTITION BY`)
- **Column type casting** — `bed`, `bath`, and `zip_code` were cast to appropriate types (`INT64`, `STRING`)
- **Filtered low-quality records** — removed `ready_to_build` status entries (277 rows with no sold date), states with very few records (Virginia, Georgia, South Carolina, Tennessee, Wyoming, West Virginia), and properties priced below $5,000 (51 rows)
- **Fixed null city values** — 23 null city entries resolved using `CASE WHEN` logic matched against street address
- **Standardised city names** — multiple spellings of New York (`New York City`, `Nyc`, `Ny`) unified using `REPLACE()`
- **Corrected data entry errors** — three properties with clearly incorrect prices, bedroom/bathroom counts, or missing sizes were manually corrected against public realtor data
- **Unit conversions** — added `house_size_m2` (from sq ft) and `hectare_lot` (from acres) columns
- **Separated property types** — land plots (no bedrooms, bathrooms, or house size) separated into their own table (`re_us_plots`)
- **Extracted year** — added `year` column using `EXTRACT(YEAR FROM sold_date)`
- **Null handling** — `sold_date` nulls retained in the general property table (`re_us_property`); a separate clean table (`re_us_sold`) contains only records with a known sold date for time-series analysis

---

## Data Exploration

Performed in `Exploring_data.sql`. Queries cover:

- Property sales count by year
- State-level aggregations: count, average/min/max price, average/min/max size, average/min/max lot size
- City-level aggregations with the same metrics
- Bedroom and bathroom distribution, cross-referenced with state and price
- Year and month extraction for time-series queries
- Separate exploration of off-market (null sold date) records

Two query outputs were exported to CSV for use in Power BI:
- `Quered_data_property_sold.csv` — grouped by state, city, year, bedrooms, bathrooms
- `Quered_data_year_and_month.csv` — same, with month added

---

## Dashboards

The Power BI report (`Real_Estate_USA_Dashboards.pbix`) contains three dashboard pages.

### Page 1 — Sales Stats Overview
<img width="1914" height="1081" alt="Screenshot 2026-03-20 140517" src="https://github.com/user-attachments/assets/fd56181f-5c4b-4189-a5cd-d13dec3ea274" />

High-level summary across all states and years.

| Metric | Value |
|---|---|
| Total properties sold | ~57,000 |
| Average price | $811K |
| Minimal price | $7.5K |
| Maximal price | $135M |
| Average property size | 212.49 m² |
| Average lot size | 4.19 HA |

Key visuals: property count by state (bar chart), market size by state (treemap), bedroom and bathroom distribution (donut charts), top cities by sales volume.

Top states by volume: New Jersey (20,405), New York (15,116), Connecticut (7,831), Pennsylvania (6,042), Massachusetts (3,980).

### Page 2 — Price Trends by Year
<img width="1914" height="1069" alt="Screenshot 2026-03-20 140313" src="https://github.com/user-attachments/assets/a00f0070-371a-4f86-ba54-f1bc4d09f68c" />

Filters: state, city, number of bedrooms, year range (2000–2023).

Key visuals: average price over time (line chart), property sold count by year (bar chart). Total properties sold across the filtered period: 47K. Average price fluctuates between approximately $690K and $950K depending on year.

### Page 3 — Sales by State and Month
<img width="1914" height="1072" alt="Screenshot 2026-03-20 140324" src="https://github.com/user-attachments/assets/439167c8-82cd-43ea-bca1-c99e89a1ef25" />

Filters: state, year.

Key visuals: sales trend lines by state over time (1990–2023), monthly seasonality chart. Sales peak in summer months (July–August) and dip in February. New Jersey shows the sharpest growth trend post-2015.

---

## Key Findings

- **2023** recorded the highest number of property sales in the dataset; **1901** recorded the lowest
- **New Jersey** leads in total properties sold; **New York** leads in total market size ($21bn)
- The most common property configuration is **3 bedrooms** (34.63%) and **2 bathrooms** (35.67%)
- Sales follow a clear seasonal pattern, peaking in **summer** and declining in **winter**
- A significant portion of records (54,092) have no `sold_date`, indicating off-market or unlisted-date properties — these were retained separately for general analysis but excluded from time-series work
