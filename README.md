
# Seattle Transit & Weather Analytics Platform

A full data engineering pipeline that integrates Seattle's public transit (GTFS) data with local hourly weather forecasts from NOAA to analyze how weather conditions impact transportation activity. Built using PySpark, Databricks, Delta Lake, and visualized through Databricks SQL Dashboards.

---

## Project Overview
This project ingests real-time and static GTFS transit feeds and enriches them with weather data from the National Weather Service. The goal is to explore correlations between transit behavior and weather patterns across time, location, and route types.

---

## Tech Stack
- **Databricks** (notebooks, job scheduling, dashboards)
- **PySpark**
- **Delta Lake** for table versioning and partitioning
- **NOAA API** (National Weather Service)
- **GTFS Feeds** (King County Metro)
- **Databricks Dashboards** for visual exploration
- **GitHub** for version control and documentation
- **Pandas, Seaborn, Matplotlib, SciPy, scikit-learn, XGBoost** for analytics and ML

---

## Folder Structure
```
📁 data/                  # Sample outputs from Silver tables
    ├── Real_Time_GTFS_sample_data.csv
    ├── Weather_sample_data.csv
    └── Static_GTFS_sample_data.csv

📁 dashboards/            # PNGs of Databricks dashboard charts
    ├── vehicle_updates_per_day.png
    ├── vehicle_count_by_weather.png
    ├── vehicle_updates_by_hour.png
    └── ...

📁 notebooks/             # All pipeline notebooks
    ├── 01_ingest_gtfs_static.ipynb
    ├── 02_ingest_gtfs_rt.ipynb
    ├── 03_transform_gtfs_rt.ipynb
    ├── 04_enrich_rt_with_static.ipynb
    ├── 05_ingest_nws_weather.ipynb
    ├── 06_transform_nws_weather.ipynb
    ├── 07_join_rt_with_weather.ipynb
    ├── 10_transform_gtfs_static.ipynb
    ├── 11_statistical_analysis.ipynb     
    ├── 12_predict_transit_volume.ipynb   
    ├── 13_platinum_static.ipynb
    ├── 14_create_platinum.ipynb
    ├── 96_one_time_dedupe.ipynb
    ├── 97_data_validation_tests.ipynb
    ├── 98_export_sample_data.ipynb
    └── 99_cleanup_silver_rt.ipynb

README.md                # This file
```

---

## Pipeline Architecture

```
Sources
========
┌──────────────────────────────┐   ┌──────────────────────────┐   ┌──────────────────────────┐
│ GTFS Static (routes/stops/   │   │  GTFS-RT Vehicle Feed    │   │   NOAA / NWS Hourly API  │
│ trips, SCD2 over time) (01)  │   │ (02)                     │   │ (05)                     │
└───────────────┬──────────────┘   └──────────────┬───────────┘   └──────────────┬───────────┘
                │                                 │                              │
                ▼                                 ▼                              ▼
          Bronze / gtfs_static              Bronze / gtfs_rt               Bronze / weather
                     (Delta)                        (Delta)                       (Delta)
                     │                                │                              │
             (10)    │                        (03)    │                        (06)  │
                     ▼                                ▼                              ▼
          Silver / gtfs_static                Silver / gtfs_rt                Silver / weather
                     │                                │                              │
                     │                                │                              │
     ┌───────────────┴───────────────┐                │                              │
     │   Used to ENRICH RT in Gold   │ <──────────────┘                              │
     │          (04 notebook)        │                                               │
     └───────────────┬───────────────┘                                               │
                     ▼                                                               │
            Gold / gtfs_rt_enriched  (04)                                            │
                     │                                                               │
                     │                    Join RT (Gold) to Weather (Silver)         │
                     └───────────────────────────────►  (07)  ◄──────────────────────┘
                                                       Gold / gtfs_rt_weather_joined
                                                                    │
            ┌───────────────────────────────────────────────────────│
            │                                                       │
            │            ┌──────────────────────────────────────────┴──────────────────────────────────────────┐
            │            │                                                                                     │
            │            │     Analytics & ML (consume Gold)                                                   │
            │            │     ────────────────────────────────────────────────────────────────────────────    │
            │            │     (11) Statistical Analysis  ──►  /analytics/ (summaries, charts, test results)   │
            │            │     (12) ML Prediction         ──►  /models/, /predictions/ (e.g., daily forecasts) │
            │            │                                                                                     │
            │            └─────────────────────────────────────────────────────────────────────────────────────┘
            │                                                       
            │
            └───────────────────────────────────────────────────► (14)  
                                                     Platinum / fact_transit_event
                                                       (joins to dims with SKs)

                 Platinum Dimensions (built biweekly)  (13)
                 ┌──────────────────────────────┐     (from **Bronze/SCD2** static)
                 │  Platinum / dim_route        │◄─────────┐
                 └──────────────────────────────┘          │
                 ┌──────────────────────────────┐          │
                 │  Platinum / dim_trip         │◄─────────┘
                 └──────────────────────────────┘

                                           BI Consumers
                                           ─────────────
                                           • Databricks SQL Dashboards
                                           • Power BI (reads Platinum via Unity Catalog)

                                    ┌──────────────────────────────────────────────────────┐
                                    │              Guardrails (cross-cutting)              │
                                    │  (96) One-time Dedupe  •  (97) Data Validation       │
                                    │  (99) Cleanup (e.g., 1970-01-01)                     │
                                    │  Applied to: Bronze | Silver | Gold | Platinum       │
                                    └──────────────────────────────────────────────────────┘

```

---

## Data Lake Architecture: Bronze, Silver, Gold, and Platinum Layers

This project follows the **medallion architecture pattern** to structure raw, cleaned, and enriched data for analytics and ML.

---

### Bronze Layer: Raw Ingested Data
**Purpose**: Stores raw data as received from source systems (GTFS static, GTFS real-time, and weather APIs).

Handled in:
- `01_ingest_gtfs_static.ipynb`
- `02_ingest_gtfs_rt.ipynb`
- `05_ingest_nws_weather.ipynb`

Paths:
- **GTFS Static**: `/bronze/gtfs_static` — Raw GTFS files (`routes`, `stops`, `trips`)
- **GTFS Real-Time**: `/bronze/gtfs_rt` — Real-time vehicle positions
- **Weather**: `/bronze/weather` — NOAA hourly forecast snapshots

---

### Silver Layer: Cleaned and Transformed Data
**Purpose**: Applies schema validation, type casting, filtering, enrichments, and deduplication.

Handled in:
- `03_transform_gtfs_rt.ipynb`
- `06_transform_nws_weather.ipynb`
- `10_transform_gtfs_static.ipynb`

Paths:
- **GTFS Static**: `/silver/gtfs_static` — Combines lat/lon, adds `ingestion_ts`
- **GTFS Real-Time**: `/silver/gtfs_rt` — Adds `event_date`, filters nulls, formats timestamps
- **Weather**: `/silver/weather` — Drops nulls, filters unrealistic values, adds `ingestion_date`

---

### Gold Layer: Enriched and Joined Data
**Purpose**: Combines weather + GTFS RT for rich analytics and ML-ready features.

Handled in:
- `04_enrich_rt_with_static.ipynb`
- `07_join_rt_with_weather.ipynb`

Paths:
- **GTFS RT Enriched**: `/gold/gtfs_rt_enriched` — Joins GTFS RT with static route/trip info
- **GTFS RT + Weather**: `/gold/gtfs_rt_weather_joined` — Joins GTFS RT with nearest forecast time from NOAA

This tier powers:
- Dashboards (visualizations)
- Analytics and insight generation
- Statistical tests or ML tasks

---

### Platinum Layer – Star Schema for BI
Optimized for slicing/filtering in BI tools:
- `13_platinum_static.ipynb` – Builds SCD2 dimension tables (`dim_trip`, `dim_route`) with surrogate keys
- `14_create_platinum.ipynb` – Joins SKs into `fact_transit_event`

Paths:
- `/plat/dim_trip/`, `/plat/dim_route/`, `/plat/fact_transit_event/`

Each layer is written as a Delta table and partitioned by date for performance and scalability.

---

## Additional Analytics & Modeling
`11_statistical_analysis.ipynb` – Statistical Analysis of Seattle Transit Activity vs Weather
Goal: Identify patterns and relationships between transit activity and weather variables.

Data: Gold-layer table gtfs_rt_weather_joined.

### Key Analyses:

Correlation tests (Pearson & Spearman) between average temperature and daily vehicle updates.

Weekday vs weekend comparison using t-tests.

Visualization of trends using Seaborn and Matplotlib.

### Insights:

Found weak positive correlations between temperature and vehicle update counts (not statistically significant).

Strong, statistically significant difference between weekday and weekend activity levels.

`12_predict_transit_volume.ipynb` – Machine Learning Prediction of Transit Volume
Goal: Predict daily transit volume using weather and time-based features.

Data: Gold-layer table gtfs_rt_weather_joined (aggregated to daily level).

### Approach:

Feature engineering: Extract day of week, clean wind speed, one-hot encode weather conditions.

Models compared: Linear Regression, Random Forest, XGBoost.

### Results:

Random Forest achieved R² = 0.95; XGBoost achieved R² = 0.97.

Most important predictor: day_of_week, followed by wind_speed and certain weather condition categories.

### Use Cases:

Forecasting peak days for transit service demand.

Supporting operational decision-making under varying weather conditions.

---

## Automation & Scheduling

- Daily automated runs are scheduled using **Databricks Workflows**, ensuring up-to-date data ingestion, processing, and dashboard refresh every morning at 8:00 AM.
- Task dependencies are defined to preserve logical notebook execution order (e.g., static before enrichment, weather before joins).

### Daily Pipeline (8:00 AM)
Runs Notebooks:
- `02`, `03`, `05`, `06`, `04`, `07`, `97`, `14`
- Ingests real-time GTFS and weather
- Refreshes Databricks Dashboards by **8:30 AM**

### Biweekly Pipeline (Every 15 Days)
Runs Notebooks:
- `01`, `13`, `10`
- Updates SCD2-based static GTFS and rebuilds Platinum dimensions and fact table

### Biweekly Pipeline (Every 2 weeks)
Runs Notebooks:
- `11`, `12`
- Updates statistical analysis and ML models

---

## Data Validation & Cleanup

- `97_data_validation_tests.ipynb`  
  - Checks for nulls, timestamp logic, temperature bounds, and uniqueness
- `99_cleanup_silver_rt.ipynb`  
  - Removes records with `event_date = '1970-01-01'` or missing weather timestamps
- `96_one_time_dedupe.ipynb`  
  - Drops duplicates in historical Bronze, Silver, and Gold tables

---

## Dashboards & Visual Insights

### Databricks Dashboard:

- **Vehicle Updates by Day**: Understand how daily transit activity varies
- **Temperature vs Transit**: Correlate cold/warm days with usage patterns
- **Route Type Distribution**: See usage by bus, rail, ferry, etc.
- **Hourly Activity**: Peak hours for vehicle location updates
- **Weather Condition Trends**: Most common forecast types

PNG samples are included in the `dashboards/` folder.

### Power BI

In addition to Databricks dashboards, selected insights were recreated in Power BI to demonstrate proficiency with external BI tools.  
Data was accessed from the Unity Catalog in Databricks, using the Platinum fact table and three related dimension tables in a star schema (1-to-many relationships).  

Two visuals — *Daily Transit Activity and Average Temperature* and *Most Common Weather Conditions* — were combined into a single screenshot for presentation.  
The Power BI `.pbix` file and the screenshot are stored in the `dashboards/Power_BI` folder for reference and reproducibility.

---

## Sample Data
View sample outputs from all the tables in the `data/` folder to get a sense of the structured outputs.

---

## Key Takeaways

- Medallion architecture: Bronze → Silver → Gold → Platinum
- Automated ingestion, transformation, and dashboard refresh
- SCD2 dimensional modeling with surrogate keys
- Built-in data validation and cleanup notebooks
- Production-style pipelines with partitioned Delta tables

---

## Future Improvements
- Add geospatial mapping (e.g., Folium or Kepler.gl)
- Consider using **Apache Airflow** for advanced orchestration, notifications, or multi-project dependencies
- Add alerts for anomalies (e.g., weather spikes, route delays)
- Extend pipeline to include ridership counts (if available)

---

## About the Author
**Elham Afruzi**  
Data Scientist
Seattle, WA  
[LinkedIn](https://www.linkedin.com/in/elham-afruzi/)  |  [GitHub](https://github.com/El-ham)

---
