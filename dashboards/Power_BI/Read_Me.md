## Power BI Visualizations

The Platinum fact table and three related dimension tables were accessed from the Unity Catalog in Databricks and imported into Power BI using a star schema (1-to-many relationships). Two visualizations previously created in Databricks dashboards were recreated in Power BI:

1. **Daily Transit Activity and Average Temperature**  
   A combined line and bar chart showing the relationship between vehicle counts and average daily temperature.

2. **Most Common Weather Conditions**  
   A donut chart displaying the distribution of recorded weather conditions, illustrating which conditions occur most frequently.

Both visuals were captured in a single screenshot and are included in this repository. The `.pbix` file is stored in the Databricks `dashboards/powerbi` folder for reference and reproducibility.
