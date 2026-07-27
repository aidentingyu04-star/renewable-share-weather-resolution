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

```

These samples show the expected formats but are not the complete Denmark
dataset used for the reported results.

## Experimental design

The full comparison contains:

- 19 power systems
- 4 weather-grid resolutions
- 2 spatial weighting methods
- 2 classification models
- 2 regression models
- 152 country-resolution-weighting configurations
- 608 paired model evaluations

Each evaluation compares calendar-only and calendar-plus-weather versions of
the same algorithm.

### Feature sets

**Calendar**

- Hour sine and cosine
- Month sine and cosine
- Weekend indicator

**Calendar + weather**

- All calendar features
- 100-m wind speed
- Surface solar radiation
- 2-m temperature

### Models

Classification:

- Logistic regression
- Random forest

Regression:

- Gradient boosting
- LightGBM

### Evaluation

Every country uses a chronological split. The first 80% of observations are
used for training and the final 20% for testing. Data are not shuffled, and
`StandardScaler` is fitted only on training data.

Classification is evaluated with:

- Area under the ROC curve (AUC)
- Brier score

Regression is evaluated with:

- Coefficient of determination (R²)
- Mean absolute error (MAE)

Positive weather-added values always mean that weather helped:

- `auc_gain = AUC(calendar + weather) - AUC(calendar)`
- `brier_reduction = Brier(calendar) - Brier(calendar + weather)`
- `r2_gain = R²(calendar + weather) - R²(calendar)`
- `mae_reduction = MAE(calendar) - MAE(calendar + weather)`

## Spatial-resolution experiment

Two national weather representations are compared:

- **Capacity weighted:** grid cells receive more weight when they represent
  more known wind or solar capacity.
- **Uniform:** all national grid cells receive equal weight.

Weather processing is measured using point-hours, preparation time, and peak
memory. Independent country, resolution, and weighting configurations can run
in parallel across CPU cores or cluster jobs.

## Block bootstrap

The uncertainty analysis resamples 2,000 complete test-set weeks with
replacement. Whole weeks are used instead of individual hours so that related
neighboring observations remain together.

For each resample, the analysis recomputes:

- AUC improvement
- Brier-score reduction
- R² improvement
- MAE reduction
- Correlation between weather gain and wind-minus-solar share

The bootstrap resamples unseen test predictions rather than retraining the
models. Its intervals therefore measure test-period sampling uncertainty
conditional on the fitted models.

## Repository organization

```text
data/       Data instructions and small Denmark samples
figures/    Poster and manuscript figures
results/    Compact result tables and bootstrap summaries
scripts/    Data preparation, modeling, bootstrap, and plotting code
poster/     Final poster when available
```

Large ERA5, Energy-Charts, and generated weather files are excluded from
version control.

## Installation

Python 3.11 or newer is recommended.

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

## Reproducing the analysis

Preview the 2022–2026 workflow:

```bash
python scripts/run_era_spatial_resolution.py \
  --eras post \
  --resolutions 0.25 0.5 1.0 2.0 \
  --schemes capacity uniform \
  --workers 8 \
  --dry-run
```

Run the complete workflow:

```bash
python scripts/run_era_spatial_resolution.py \
  --eras post \
  --resolutions 0.25 0.5 1.0 2.0 \
  --schemes capacity uniform \
  --workers 8
```

Run the whole-week bootstrap:

```bash
python scripts/bootstrap_post_covid_all_models.py \
  --n-resamples 2000 \
  --confidence 0.95 \
  --workers 8
```

Regenerate the principal figures:

```bash
python scripts/generate_original_study_figures.py --skip-refit
python scripts/generate_paper_placeholder_figures.py
python scripts/plot_post_covid_bootstrap.py \
  --metric-family error \
  --formats png pdf
```

## Important limitations

- ERA5 is same-hour retrospective reanalysis, not an operational forecast.
- The calendar baseline does not include recent renewable-generation history.
- Facility maps may omit rooftop solar and smaller installations.
- Facility coverage and commissioning dates vary among countries.
- Results are limited to the four evaluated models and 19 European systems.
- Ireland and Serbia are excluded from wind-minus-solar correlations because
  usable solar observations were unavailable.
- Point-hours measure processed data volume rather than end-to-end cluster
  cost.
- Bootstrap intervals do not include model-retraining variability.

Although the numerical findings are specific to these models and countries,
the workflow can be extended to other algorithms, regions, weather datasets,
and prediction targets.

## Data availability

This repository contains code, derived result tables, figures, and small
Denmark samples. Original electricity, ERA5, and facility datasets are not
included. Obtain them directly from Energy-Charts, the Copernicus Climate Data
Store, and Global Energy Monitor.

## Poster

The final SC26 research-poster PDF will be added under `poster/`.

## AI disclosure

OpenAI Codex assisted with research planning, code organization, debugging,
figure formatting, and wording. The authors reviewed the analysis design,
executed the experiments, verified the results, and are responsible for the
scientific interpretation.

## Contact

For questions or reproducibility issues, open a GitHub issue in this
repository.
