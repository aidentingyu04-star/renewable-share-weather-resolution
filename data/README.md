# Data

This directory contains the local inputs and derived hourly datasets used by the
renewable-share weather-resolution study. The repository does not distribute
the large raw or derived data files. They can be reconstructed from the
original providers using the scripts in `scripts/`.

## Study coverage

- Period: January 2022 through April 2026
- Resolution: hourly
- Systems: 19 European power systems
- Country codes: `at`, `be`, `bg`, `cz`, `de`, `dk`, `es`, `fr`, `gr`, `hr`,
  `ie`, `lt`, `lv`, `nl`, `pt`, `ro`, `rs`, `si`, and `sk`
- Approximately 36,000–38,000 aligned observations are available per country.
- All 19 systems are modeled. Wind-minus-solar correlation analyses use the 17
  systems with usable wind and solar observations.

## Original sources

| Source | Variables used | Role |
|---|---|---|
| Fraunhofer ISE Energy-Charts | Hourly national generation by source and load | Renewable-share prediction target |
| ECMWF/Copernicus ERA5 | 100-m wind, surface solar radiation, and 2-m temperature | Weather predictors |
| Global Energy Monitor | Locations, technologies, capacities, and commissioning years of known wind and solar facilities | Annual spatial weights |

Data remain subject to the original providers' licenses and terms. Retrieve raw
data from those providers rather than committing provider downloads to this
repository.

## Active post-2022 directories

| Directory | Contents | Approximate local size | Commit to GitHub? |
|---|---|---:|---|
| `sample/` | Seven-day Denmark example and 2022 facility weights | small | Yes |
| `energy_targets_source_by_era/` | Downloaded electricity and load tables | 200 MB | No |
| `energy_targets_by_era/post/` | Links exposing post-2022 targets to the evaluator | negligible | No |
| `era5_native_0p25deg/` | Native 0.25-degree country subsets | 8.3 GB | No |
| `era5_coarse_post_covid/` | Cached 0.25-, 0.5-, 1-, and 2-degree grids | 4.4 GB | No |
| `capacity_weights_post_by_year/` | Annual facility-weight tables for 2022–2026 | 2.9 MB | Rebuild instead |
| `capacity_weights_unknown_start_excluded/` | Sensitivity maps excluding unknown commissioning dates | 800 KB | Rebuild instead |
| `country_weather_post_covid/` | National hourly weather under each grid and weighting configuration | 429 MB | No |

Legacy pre-COVID, COVID-period, and HPC-scheduling directories are not used by
the final post-2022 analysis.

The repository-ready Denmark example is documented in `sample/README.md`.

## Main file schemas

### Electricity targets

Files are named `weather_energy_merged_<country>.csv`. Important fields include:

```text
timestamp
Load
Wind_onshore / other available wind columns
Solar
Renewable_share_of_load
```

Generation columns vary slightly across national systems. The modeling scripts
standardize the available country columns before evaluation.

### National weather features

Files follow names such as:

```text
weather_era5_bg_capacity_1p0deg.csv
weather_era5_bg_uniform_1p0deg.csv
```

Their common schema is:

```text
timestamp
wind_speed_100m
shortwave_radiation
temperature_2m
```

`capacity` means weather grid cells are weighted toward known wind and solar
facilities. `uniform` gives included grid cells equal weight.

### Annual facility weights

Files are stored as `<year>/<country>.csv` with:

```text
lat
lon
wind_mw
solar_mw
```

These tables represent known geolocated facilities, not a complete inventory of
every distributed or unreported installation.

## Processing flow

```text
Energy-Charts generation and load
                         \
ERA5 0.25-degree grids -> resolution caches -> national weather features
                         /
Annual wind/solar facility weights
                         |
                         v
        chronological classification and regression evaluation
```

Relevant scripts include:

- `stage_era5_arco.py`
- `prepare_era5_resolution_cache.py`
- `import_gem_capacity.py`
- `run_spatial_resolution_ladder.py`
- `evaluate_spatial_weather.py`
- `bootstrap_post_covid_all_models.py`

## Repository policy

Commit this README, analysis scripts, compact result summaries, and final
figures. Do not commit NetCDF/Zarr weather files, raw provider downloads,
country-level hourly feature tables, credentials, `.DS_Store`, lock files, or
machine-specific paths.
