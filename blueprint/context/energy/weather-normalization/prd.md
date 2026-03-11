# Portfolio-Wide Weather Normalization — PRD


| Field        | Value         |
| ------------ | ------------- |
| Author       | Endrit Gojani |
| Status       | Draft         |
| Last updated | 2026-03-11    |
| Product      | energy        |
| Design       | —             |
| Spec         | —             |


## Problem

Portfolio operators manage buildings spread across vastly different climate zones — Alaska, Florida, the Midwest, the Southwest. Raw EUI comparisons between these buildings are misleading because weather dominates energy consumption. A building in Miami running heavy cooling looks worse than one in Seattle simply because of climate, not because it's poorly operated.

Without weather normalization, building-to-building comparisons are unfair, year-over-year trends are unreliable, portfolio-wide benchmarks are meaningless, and capital investment decisions are misinformed. Energy managers feel this pain every time they compare buildings, prepare portfolio reports, or prioritize energy projects across regions.

## Users

**Energy Manager** — Responsible for energy performance, sustainability targets, and regulatory compliance across a portfolio. Needs weather-normalized EUI to make fair comparisons across climate zones and track genuine year-over-year changes.

## Goals

- Energy manager can compare any two buildings in the portfolio on a weather-normalized basis, regardless of climate zone
- Year-over-year EUI trends reflect operational changes, not weather fluctuations
- Portfolio ranking identifies genuinely underperforming buildings, not buildings penalised by geography
- Weather's contribution to energy consumption is quantified and visible per building
- Normalization methodology is auditable — calculations can be traced back to source data, regression model, and TMY baseline

## Non-goals

- Predictive load forecasting based on weather forecasts
- Sub-daily (hourly) normalization — this version operates at monthly/annual granularity
- Custom TMY generation — we use standard TMY3 datasets
- User-driven weather station override — station mapping is automatic based on building lat/lng
- Normalization for non-energy metrics (water, waste, etc.)

## User stories

- As an Energy Manager, I want to see all my buildings ranked by weather-normalized EUI so that I can identify which buildings genuinely underperform, regardless of their climate zone
- As an Energy Manager, I want to compare a building's normalized EUI year-over-year so that I can tell whether operational changes actually improved performance or weather just got milder
- As an Energy Manager, I want to see how much of a building's energy consumption is driven by weather vs. operations so that I can quantify where real savings opportunities exist
- As an Energy Manager, I want to see raw EUI alongside normalized EUI so that I understand the magnitude of the weather adjustment and can explain it to ownership
- As an Energy Manager, I want to know when a building's normalization model is unreliable so that I don't make decisions based on misleading data

## Requirements

### Must have

- **NOAA weather data ingestion** — system automatically pulls HDD/CDD data for each building's mapped weather station
- **Weather station mapping** — each building is automatically mapped to the nearest NOAA weather station based on lat/lng
- **Change-point regression model** — system builds an ASHRAE Inverse Modeling regression for each building with 12+ months of meter data, relating monthly energy to outdoor temperature
- **Normalized EUI metric** — system computes weather-normalized EUI using TMY3 as the baseline climate and publishes it as a Metric on the building entity
- **Portfolio normalized ranking** — all buildings displayed in a ranked view by normalized EUI, with raw EUI shown alongside
- **Weather impact breakdown** — per-building breakdown showing weather-driven vs. operations-driven energy consumption
- **Year-over-year normalized comparison** — normalized EUI for current vs. prior year(s), highlighting genuine performance changes
- **Confidence indicator** — each normalized value shows model confidence (based on R² and data completeness)
- **Insufficient data handling** — buildings with < 12 months of data show raw EUI only, with a clear explanation of what's missing
- **Missing location handling** — buildings without lat/lng show raw EUI only, with a prompt to add location

### Should have

- **Raw + normalized side-by-side trend** — time-series chart with shaded area showing the weather effect
- **Weather-insensitive building detection** — automatically flag buildings with near-zero weather correlation and skip normalization
- **Regression model metadata** — R², coefficients, base temperature, and change-point available for audit
- **Climate zone display** — show each building's climate zone in the portfolio view for context
- **Data gap handling** — interpolate weather data for gaps ≤ 30 days; flag values as approximate for gaps > 30 days

### Won't have (this version)

- Predictive load forecasting
- Hourly normalization
- Custom TMY generation
- Manual weather station override
- Non-energy normalization

## Acceptance criteria

- Given a building with 12+ months of meter data and a valid lat/lng, when the normalization batch runs, then a weather-normalized EUI Metric is published on the building entity
- Given a portfolio of buildings across different climate zones, when the Energy Manager opens the portfolio ranking, then buildings are ranked by normalized EUI with raw EUI visible alongside
- Given a building with < 12 months of meter data, when the portfolio ranking loads, then the building shows raw EUI only with a banner explaining why normalization is unavailable
- Given a building with no lat/lng, when the portfolio ranking loads, then the building shows raw EUI only with a prompt to add location
- Given a building whose regression model has R² < 0.5, when the normalized EUI is displayed, then a low-confidence indicator is visible with an explanatory tooltip
- Given a building with near-zero weather correlation (R² < 0.1 on weather terms), when normalization runs, then the building is flagged as "weather-insensitive" and raw EUI is used
- Given a building with 2+ years of data, when the Energy Manager views year-over-year comparison, then both years show normalized EUI and the delta reflects operational change only
- Given a weather data gap of ≤ 30 days, when normalization runs, then the gap is interpolated and the result is not flagged
- Given a weather data gap of > 30 days, when normalization runs, then the normalized value is flagged as approximate with reduced confidence
- Given any normalized EUI value, when the Energy Manager inspects it, then the calculation is traceable to source meter data, weather data, regression model, and TMY baseline

## Open questions


| Question                                                                                               | Owner         | Due |
| ------------------------------------------------------------------------------------------------------ | ------------- | --- |
| How frequently should normalized EUI be recomputed? (Monthly batch? On-demand?)                        | Endrit Gojani | —   |
| Should we expose regression model details (R², coefficients) to the user or keep it behind the scenes? | Gresa Bytyqi  | —   |
| What R² threshold defines "low confidence" vs. acceptable? (0.5 proposed)                              | Endrit Gojani | —   |
| What R² threshold defines "weather-insensitive"? (0.1 proposed)                                        | Endrit Gojani | —   |
| How do we handle buildings that straddle two climate zones?                                            | Endrit Gojani | —   |


## Dependencies


| Dependency                                         | Status      | Owner                 |
| -------------------------------------------------- | ----------- | --------------------- |
| 12+ months of meter data per building              | Required    | DataHub               |
| Building location (lat/lng) in the Building Graph  | Required    | BMS                   |
| NOAA weather data integration (HDD/CDD by station) | To be built | DataHub / Engineering |
| TMY3 datasets loaded into Library                  | To be built | Library / Domain      |
| ASHRAE Inverse Modeling Toolkit implementation     | To be built | Engineering           |
| Meter data pipeline (energy/utility-data feature)  | Defining    | DataHub               |


## References

- Context file: [context.md](context.md)
- Design flow: [flow.md](flow.md)
- EUI concept: `energy/domain/concepts/eui.md`
- Benchmarking concept: `energy/domain/concepts/benchmarks.md`
- ASHRAE Guideline 14 — Measurement of Energy, Demand, and Water Savings (inverse modeling methodology)
- NOAA Climate Data Online: [https://www.ncdc.noaa.gov/cdo-web/](https://www.ncdc.noaa.gov/cdo-web/)
- TMY3 datasets: [https://rredc.nrel.gov/solar/old_data/nsrdb/1991-2005/tmy3/](https://rredc.nrel.gov/solar/old_data/nsrdb/1991-2005/tmy3/)

