# How Much Weather Is Enough for Predicting Renewable Grid Share?

This project tests how much spatial detail is actually needed when using
large weather datasets to predict hourly renewable-energy share. It combines
national electricity data with ERA5 weather and geolocated wind and solar
capacity across 19 European power systems.

The main experiment compares four ERA5 grid resolutions and two ways of
averaging weather. The goal is to measure the tradeoff between predictive
value and the amount of weather data that must be stored, moved, and
processed.

## Main findings

- Adding weather improved renewable-share prediction in most countries.
- Weather helped most in wind-heavy power systems. Across the four models,
  the correlation between weather-added gain and wind share minus solar share
  ranged from **0.809 to 0.881** across the 17 countries with complete wind
  and solar observations.
- Weighting weather toward known wind and solar facility locations
  outperformed treating all national grid cells equally. Mean improvements
  were **0.013–0.017 AUC** for classification and **0.043–0.047 R²** for
  regression.
- Compared with the 0.25° grid, the 0.5°, 1°, and 2° grids processed
  approximately **25.9%, 7.0%, and 2.0%** as many weather point-hours.
  Predictive results remained relatively stable under moderate coarsening.
- Whole-week block bootstrap results showed clear prediction-error reductions
  in **14–17 of 19 countries**, depending on the model.

## Research question

ERA5 is an HPC-generated global reanalysis. Fine spatial grids increase data
volume, storage, data movement, memory use, and preprocessing work. This
project asks:

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

ERA5 is evaluated at 0.25°, 0.5°, 1°, and 2° grid resolutions.

### Wind and solar facilities

Facility locations and capacities come from Global Energy Monitor's
[Global Wind Power Tracker](https://globalenergymonitor.org/projects/global-wind-power-tracker)
and
[Global Solar Power Tracker](https://globalenergymonitor.org/projects/global-solar-power-tracker),
February 2026 releases.

These records are used to weight national weather toward known wind and solar
facilities. The repository does not redistribute the original tracker
workbooks; download them from Global Energy Monitor and follow the preparation
instructions in `data/README.md`.

## Experimental design

The full comparison contains:

- 19 power systems
- 4 weather-grid resolutions
- 2 spatial weighting methods
- 2 classification models
- 2 regression models
- 152 weather configurations
- 608 model evaluations

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

Every country uses a chronological split: the first 80% of observations are
used for training and the final 20% for testing. Data are not shuffled.
`StandardScaler` is fitted on the training period only.

Classification is evaluated with:

- Area under the ROC curve (AUC)
- Brier score

Regression is evaluated with:

- Coefficient of determination (R²)
- Mean absolute error (MAE)

Weather-added gain is the difference between the calendar-only model and the
calendar-plus-weather model. Positive values always mean that adding weather
helped.

## Spatial-resolution experiment

Two national weather representations are compared:

- **Facility weighted:** grid cells receive more weight when they contain more
  known wind or solar capacity.
- **Uniform:** all national grid cells receive equal weight.

Weather processing is measured using point-hours, preparation time, and peak
memory. Country, resolution, and weighting configurations are independent and
can be distributed across CPU cores or cluster array jobs.

## Block bootstrap

The uncertainty analysis resamples 2,000 complete test-set weeks with
replacement. Complete weeks are used instead of individual hours so that
related neighboring hours remain together.

For each resample, the analysis recomputes:

- AUC improvement
- Brier-score reduction
- R² improvement
- MAE reduction
- Cross-country correlation between weather gain and wind-minus-solar share

The bootstrap conditions on the fitted models; it resamples their held-out
predictions rather than retraining every model.

## Repository organization

```text
data/       Data-access instructions and local generated inputs
figures/    Poster and manuscript figures
results/    Compact result tables and bootstrap summaries
scripts/    Data preparation, modeling, bootstrap, and plotting code
```

Large ERA5, Energy-Charts, and generated weather files are intentionally
excluded from version control.

## Installation

Python 3.11 or newer is recommended.

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

## Reproducing the analysis

All scripts anchor paths to the project directory rather than the current
working directory.

Preview the post-2022 workflow:

```bash
python scripts/run_era_spatial_resolution.py \
  --eras post \
  --resolutions 0.25 0.5 1.0 2.0 \
  --schemes capacity uniform \
  --workers 8 \
  --dry-run
```

Run the workflow after the required source data are available:

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

- ERA5 is a reanalysis, not an operational weather forecast.
- Facility weighting represents known geolocated utility-scale capacity and
  does not capture every small or distributed installation.
- Facility coverage and commissioning dates vary among countries.
- National load and generation accounting conventions may differ.
- Ireland and Serbia are modeled but excluded from wind-minus-solar
  correlations because usable solar observations were unavailable.
- Bootstrap confidence intervals describe held-out prediction uncertainty
  conditional on the fitted models; they do not include model-retraining
  variability.

## Data availability

This repository contains code, derived result tables, and figures. Original
electricity, ERA5, and facility datasets are not included. Users should obtain
them directly from Energy-Charts, the Copernicus Climate Data Store, and
Global Energy Monitor.

## Poster

The final SC26 research-poster PDF will be added under `poster/`.

## AI disclosure

Generative-AI tools assisted with code organization, debugging, figure
formatting, and wording. The authors reviewed the analysis design, executed
the experiments, checked the generated results, and are responsible for the
scientific interpretation.

## Contact

For questions or reproducibility issues, open a GitHub issue in this
repository.
