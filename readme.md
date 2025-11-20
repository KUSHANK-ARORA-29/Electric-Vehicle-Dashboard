# Electric Vehicle Data Analysis

## Project overview

A Tableau dashboard and supporting dataset for exploring electric vehicle (EV) population statistics across the United States. The dashboard surfaces high-level KPIs (total vehicles, BEV/PHEV split, average electric range), trends by model year, geographic distribution by state, top manufacturers and models, and CAFV (clean-air/fuel vehicle) eligibility breakdown.

This README documents project files, data schema, how to reproduce the main visuals (Tableau or Python), the calculations used, quick interpretation of the dashboard, and suggestions to extend it.

---

## Files included (local paths)

* Tableau workbook: `/mnt/data/Electric Vehicle Dashboard.twb`
* Dataset (CSV): `/mnt/data/Electric_Vehicle_Population_Data.csv`
* Dashboard snapshot (PNG): `/mnt/data/dashboard.png.png`

> Note: those files are already present in the project workspace. Use the CSV as the single source of truth to re-create or modify visuals.

---

## Data schema (expected columns)

Below are the columns used by the dashboard. Use the CSV to verify exact names and types — adjust if slightly different.

* `Model Year` — numeric (e.g., 2020)
* `Make` — string (manufacturer, e.g., Tesla)
* `Model` — string (vehicle model)
* `EV Type` — categorical (`BEV`, `PHEV`, etc.)
* `State` — US state (postal or full name)
* `Total Vehicles` — integer (count of vehicles / registrations)
* `Electric Range` — numeric (miles; electric-only range)
* `CAFV Eligibility` — categorical (`Eligible`, `Not Eligible`, `Unknown`)
* (optional) any other columns such as `Zip`, `City`, `Trim`, `Year` that appear in the CSV

---

## Key calculations & metrics

These are the computed measures that appear as KPI tiles in the dashboard.

* **Total Vehicles**: `SUM([Total Vehicles])`
* **Total BEV Vehicles**: `SUM(IF [EV Type] = 'BEV' THEN [Total Vehicles] ELSE 0 END)`
* **Total PHEV Vehicles**: `SUM(IF [EV Type] = 'PHEV' THEN [Total Vehicles] ELSE 0 END)`
* **% of Total (for BEV/PHEV)**: `([BEV or PHEV Total] / SUM([Total Vehicles])) * 100`
* **Avg Electric Range**: `AVG([Electric Range])` — displayed as miles (round to 2 decimals)
* **Top N Makes**: rank by `SUM([Total Vehicles])` and limit to top N (dashboard control)

Time-series (Model Year): aggregate `SUM([Total Vehicles])` by `Model Year` and plot a line/area chart (show labels in K notation, e.g., 10.7K).

Map (State): choropleth using `SUM([Total Vehicles])` per state; add labels showing counts.

Donut/Pie (CAFV Eligibility): compute percentage of `SUM([Total Vehicles])` per `CAFV Eligibility` category.

Detail table: list of models with columns `Model | Make | EV Type | Total Vehicles | % of Total` sorted by `Total Vehicles`.

---

## How to reproduce (Tableau)

1. Open Tableau and connect to `Text file` → select `/mnt/data/Electric_Vehicle_Population_Data.csv`.
2. In the Data Source pane, ensure `Total Vehicles` and `Model Year` are set to numeric types.
3. Create calculated fields below (copy-paste names and formulas from *Key calculations*).
4. Build these worksheets:

   * KPIs: separate single-number sheets for `Total Vehicles`, `Avg Electric Range`, `BEV` and `PHEV` totals, formatted as large text.
   * Time series: `Model Year` (continuous) on Columns, `SUM(Total Vehicles)` on Rows; set mark to Area + Line; annotate points for labels.
   * State map: `State` on Detail, `SUM(Total Vehicles)` on Color, add `SUM(Total Vehicles)` on Label.
   * Top makes: horizontal bar chart showing `Make` vs `SUM(Total Vehicles)`; use parameter `Top N` to filter.
   * CAFV donut: create a pie chart and format inner size to create donut aesthetics.
   * Model table: build a text table with columns listed in the data schema.
5. Combine sheets into a dashboard and arrange similar to the snapshot (`dashboard.png.png`).
6. Add dashboard filters for `CAFV Eligibility`, `EV Type`, `Model`, and `State`. Make filters global where appropriate.

---

## How to reproduce (Python — Pandas + Plotly / Matplotlib)

This is a short outline; adapt to your preferred libraries.

1. Install dependencies:

   ```bash
   pip install pandas plotly matplotlib geopandas
   ```
2. Load data:

   ```python
   import pandas as pd
   df = pd.read_csv('/mnt/data/Electric_Vehicle_Population_Data.csv')
   ```
3. KPIs:

   ```python
   total = df['Total Vehicles'].sum()
   avg_range = df['Electric Range'].mean()
   bev_total = df[df['EV Type']=='BEV']['Total Vehicles'].sum()
   phev_total = df[df['EV Type']=='PHEV']['Total Vehicles'].sum()
   ```
4. Time series:

   ```python
   ts = df.groupby('Model Year')['Total Vehicles'].sum().sort_index()
   ts.plot(kind='area')
   ```
5. Map: use `geopandas` or Plotly choropleth keyed on state codes with `SUM(Total Vehicles)`.
6. Top makes & table: `df.groupby('Make')['Total Vehicles'].sum().nlargest(5)` and `df.groupby(['Model','Make','EV Type'])['Total Vehicles'].sum().reset_index()`.

---

## Dashboard analysis & interpretation (quick takeaways)

* **Total vehicles**: the dataset contains a six-figure count (e.g., ~150k). Confirm exact figure with `SUM(Total Vehicles)`.
* **BEV vs PHEV split**: BEVs typically dominate; compute percentage to quantify (e.g., ~77% BEV, 23% PHEV in the snapshot).
* **Model year trend**: strong recent growth for newer model years (notably 2022–2023 spikes), which may indicate registration surges or improved model availability.
* **Geography**: certain states concentrate most EVs (hover over map to find state counts). Use per-capita normalization if you want fair state comparisons.
* **Top manufacturers**: Tesla often leads by a large margin; verify `Top 5` to show market concentration.
* **CAFV Eligibility**: large `Unknown` category suggests missing eligibility metadata — consider cleaning or enriching the dataset.

---

## Suggestions & next steps

* **Data quality**: impute or enrich missing `Electric Range` and `CAFV Eligibility` values where possible.
* **Per-capita metrics**: add `Total Vehicles per 100k population` by joining state population.
* **Trend decomposition**: show rolling averages and year-over-year growth rates for model year trends.
* **Interactive filters**: allow user to filter by `Make`, `EV Type`, and `Model Year Range`.
* **Exportable reports**: add a button or action to export filtered results to CSV for further analysis.

---

## Notes on reproducibility

* The Tableau workbook (`.twb`) references the CSV—if you move files, re-point the data source.
* Save extracts in Tableau (`.hyper`) for faster performance on large datasets.

---

## Licensing & contact

This repository’s code and documentation are provided for educational purposes. If you want me to convert these instructions into a runnable Jupyter notebook or generate Python scripts that recreate the full dashboard visuals, reply and I will add them.

Contact: `Kushank Arora` (project owner)

---

*End of README*
