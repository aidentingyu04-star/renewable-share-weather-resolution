# How Much Weather Is Enough for Predicting Renewable Grid Share?

This project tests how much spatial detail is needed when using large weather
datasets to predict hourly renewable-energy share. It combines national
electricity data with ERA5 weather and mapped wind and solar capacity across
19 European power systems.

The main experiment compares four ERA5 grid resolutions and two ways of
averaging weather. The goal is to measure the tradeoff between predictive
value and the amount of weather data that must be stored, moved, and processed.

## Main findings

- Weather reduced prediction error on unseen test data in **14–17 of 19
  countries**, depending on the model and error metric.
- Weather helped most in wind-heavy power systems. Across the four primary
  model metrics, the correlation between weather-added gain and wind share
  minus solar share ranged from **0.809 to 0.881** across the 17 countries
  with complete wind and solar observations.
- Capacity weighting improved performance **on average**, but not in every
  country. Relative to uniform averaging, mean improvements were
  **0.013–0.017 AUC** and **0.043–0.047 R²** at 0.25° resolution.
- Compared with 0.25°, the 0.5°, 1°, and 2° grids processed approximately
  **25.9%, 7.0%, and 2.0%** as many weather point-hours.
- Moving from 0.25° to 0.5° or 1° changed median weather-added gain by at most
  **0.003**. Some countries experienced larger losses at 2°.
- In a high-gain example, Denmark’s gradient-boosting R² increased from
  **−0.03 to 0.80**, while MAE fell from **28.7 to 11.4 percentage points**.

## Research question

ERA5 is a global atmospheric reanalysis produced and processed using
large-scale computing systems. Fine spatial grids increase storage, data
movement, memory use, and preprocessing work. This project asks:

> How much ERA5 spatial detail is needed to preserve the predictive value of
> weather for national renewable-share prediction?

## Data

The study covers 19 European power systems from January 2022 through April
2026. After aligning electricity and weather records and removing missing
observations, each country contains approximately 36,000–38,000 usable hourly
records.

### Electricity

Hourly national generation and load were obtained from the
[Fraunhofer ISE Energy-Charts API](https://api.energy-charts.info/).

The prediction target is renewable generation divided by national load. It is
evaluated in two forms:

- **Classification:** whether renewable share exceeded 50%.
- **Regression:** the continuous renewable share of load.

### Weather

Weather variables come from
[ERA5 hourly data on single levels](https://cds.climate.copernicus.eu/datasets/reanalysis-era5-single-levels?tab=overview):

- 100-m wind speed
- Surface solar radiation
- 2-m temperature

ERA5 is evaluated at 0.25°, 0.5°, 1°, and 2° grid resolutions. Spatial
resolution is the size of each weather grid cell.

### Wind and solar facilities

Facility locations and capacities come from Global Energy Monitor’s
[Global Wind Power Tracker](https://globalenergymonitor.org/projects/global-wind-power-tracker)
and
[Global Solar Power Tracker](https://globalenergymonitor.org/projects/global-solar-power-tracker),
February 2026 releases.

These records are used to weight national weather toward known utility-scale
wind and solar facilities. The repository does not redistribute the original
tracker workbooks. Download them from Global Energy Monitor and follow the
preparation instructions in `data/README.md`.

### Sample data

Small Denmark samples are included so users can inspect the expected schemas
without downloading the full datasets:

```text
data/sample/denmark_hourly_sample.csv
data/sample/denmark_capacity_weights_2022.csv
data/sample/README.md
