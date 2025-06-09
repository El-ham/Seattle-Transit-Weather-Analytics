
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
