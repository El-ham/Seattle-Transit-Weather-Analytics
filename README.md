
# Seattle Transit & Weather Analytics Platform

A full data engineering pipeline that integrates Seattle's public transit (GTFS) data with local hourly weather forecasts to analyze how weather conditions impact transportation activity. Built using PySpark, Databricks, Delta Lake, and visualized through Databricks SQL Dashboards.

---

## Project Overview
This project ingests real-time and static GTFS transit feeds and enriches them with weather data from the National Weather Service. The goal is to explore correlations between transit behavior and weather patterns across time, location, and route types.

---

## Tech Stack
- **Databricks**
- **PySpark**
- **Delta Lake** for table versioning and partitioning
- **NOAA API** (National Weather Service)
- **GTFS Feeds** (King County Metro)
- **Databricks Dashboards** for visual exploration
- **GitHub** for version control and documentation

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
    ├── ...

README.md                # This file
```

---

## Pipeline Architecture
```
            ┌─────────────────────┐
            │  GTFS-Static (once) │◄────────────┐
            └─────────────────────┘             │
                       ▼                        │
     ┌────────────────────────────┐             │
     │  Real-Time GTFS Feed (RT)  │             │
     └────────────────────────────┘             │
                       ▼                        │
        ┌────────────────────────────┐          │
        │  Bronze → Silver → Gold    │◄───┐     │
        │  (Delta Lake Transform)    │    │     │
        └────────────────────────────┘    │     │
                       ▼                  │     │
              ┌────────────────┐          │     │
              │ NOAA Hourly API│──────────┘     │
              └────────────────┘                │
                       ▼                        │
                ┌──────────────┐                │
                │ Final Join   │◄───────────────┘
                └──────────────┘
```

---

## Data Lake Architecture: Bronze, Silver, and Gold Layers

This project follows the **medallion architecture pattern** to structure raw, cleaned, and enriched data for analytics and ML.

---

### Bronze Layer: Raw Ingested Data
**Purpose**: Stores raw data as received from source systems (GTFS static, GTFS real-time, and weather APIs).

Handled in:
- `01_ingest_gtfs_static.ipynb`
- `02_ingest_gtfs_rt.ipynb`
- `05_ingest_nws_weather.ipynb`

Paths:
- **GTFS Static**: `/bronze/gtfs_static/<date>` — Raw GTFS files (`routes`, `stops`, `trips`)
- **GTFS Real-Time**: `/bronze/gtfs_rt/<date>` — Real-time vehicle position updates
- **Weather**: `/bronze/weather/<date>` — NOAA hourly forecast snapshots

---

### Silver Layer: Cleaned and Transformed Data
**Purpose**: Applies schema validation, type casting, filtering, and basic enrichments.

Handled in:
- `03_transform_gtfs_rt.ipynb`
- `06_transform_nws_weather.ipynb`
- `10_transform_gtfs_static.ipynb`

Paths:
- **GTFS Static**: `/silver/gtfs_static/<date>` — Combines lat/lon, adds `ingestion_ts`
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

Each layer is written as a Delta table and partitioned by date for performance and scalability.

---
| From → To           | Dataset                  | Key Transformations                                                                                                         |
| ------------------- | ------------------------ | --------------------------------------------------------------------------------------------------------------------------- |
| **Bronze → Silver** | `gtfs_static`            | Type casting (`stop_lat`, `stop_lon`), added `location` struct, added `ingestion_ts`<br>📓 `10_transform_gtfs_static.ipynb` |
|                     | `gtfs_rt`                | Parsed timestamps (`event_ts`), added `event_date`, `ingestion_date`, validated schema<br>📓 `03_transform_gtfs_rt.ipynb`   |
|                     | `weather`                | Removed nulls, filtered unrealistic values, added `ingestion_date`<br>📓 `06_transform_nws_weather.ipynb`                   |
| **Silver → Gold**   | `gtfs_rt_enriched`       | Joined with static `routes` and `trips` for metadata enrichment<br>📓 `04_enrich_rt_with_static.ipynb`                      |
|                     | `gtfs_rt_weather_joined` | Matched real-time events with closest hourly weather snapshot using timestamp logic<br>📓 `07_join_rt_with_weather.ipynb`   |



---
## Automation & Scheduling

- Daily automated runs are scheduled using **Databricks Workflows**, ensuring up-to-date data ingestion, processing, and dashboard refresh every morning at 8:00 AM.
- Task dependencies are defined to preserve logical notebook execution order (e.g., static before enrichment, weather before joins).

---

## Dashboards & Visual Insights
Key insights extracted and visualized via Databricks Dashboards:

- **Vehicle Updates by Day**: Understand how daily transit activity varies
- **Temperature vs Transit**: Correlate cold/warm days with usage patterns
- **Route Type Distribution**: See usage by bus, rail, ferry, etc.
- **Hourly Activity**: Peak hours for vehicle location updates
- **Weather Condition Trends**: Most common forecast types

📷 PNG samples are included in the `dashboards/` folder.

---

## Sample Data
View sample outputs from the Silver tables in the `data/` folder to get a sense of the structured outputs:
- GTFS vehicle update rows
- Weather snapshots with temperature, conditions, wind
- Static stop locations and metadata

---

## Key Takeaways
- Fully automated ingestion and transformation workflow
- Uses partitioned Delta tables with versioned layers (Bronze → Silver → Gold)
- Scheduled dashboards update daily at 8:30 AM
- Modular notebooks with clear documentation and visual storytelling

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



# TO DO