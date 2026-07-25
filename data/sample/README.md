# Denmark example data

This folder provides a small example of the processed data used by the study.
It is intended for schema inspection and smoke tests, not for reproducing the
reported multi-year results.

## Files

- `denmark_hourly_sample.csv` contains 168 aligned hourly observations from
  January 1–7, 2022. It combines selected Energy-Charts electricity fields with
  farm-weighted ERA5 weather at 0.25-degree resolution.
- `denmark_capacity_weights_2022.csv` contains the 2022 Denmark wind and solar
  facility weights used to aggregate gridded weather.

The full analysis uses January 2022 through April 2026 for all 19 power
systems. Those larger files are intentionally excluded from the repository and
can be rebuilt using the scripts and source information documented in
`data/README.md`.
