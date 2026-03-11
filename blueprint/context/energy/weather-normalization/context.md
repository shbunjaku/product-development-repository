# Weather Normalization — Context

> @-mention this file in Cursor when working on weather-adjusted energy metrics.

---

| Field        | Value |
|--------------|-------|
| Feature      | Weather Normalization |
| Product      | energy |
| Status       | Defining |
| Owner        | |
| Started      | 2026-03-11 |
| Shipped      | |
| Ticket       | |
| Figma        | |

---

## In one sentence

This feature lets energy managers compare energy performance across buildings by removing weather variability, so that true operational differences are surfaced at the portfolio level.

---

## Problem

Portfolio operators manage buildings spread across vastly different climate zones — Alaska, Florida, the Midwest, the Southwest. Raw EUI comparisons between these buildings are misleading because weather dominates energy consumption. A building in Miami running heavy cooling looks worse than one in Seattle simply because of climate, not because it's poorly operated.

Without weather normalization:
- **Building-to-building comparisons are unfair.** A well-run building in a hot climate will always appear to underperform a mediocre building in a mild climate.
- **Year-over-year trends are unreliable.** A 15% consumption increase could be a harsh winter or an operational problem — there's no way to tell.
- **Portfolio-wide benchmarks are meaningless.** Ranking buildings by raw EUI punishes geography, not poor performance. Energy managers cannot identify which buildings genuinely need attention.
- **Capital investment decisions are misinformed.** Without a level playing field, upgrade budgets get allocated based on weather noise instead of real operational gaps.

Energy managers feel this pain every time they try to compare buildings, prepare portfolio reports, or prioritize energy projects across regions.

---

## Users

**Energy Manager**
Responsible for energy performance, sustainability targets, and regulatory compliance across a portfolio. Compares buildings against benchmarks, tracks EUI trends, monitors utility bills, and prepares compliance reports. What they care about most: accuracy and auditability — energy data that can be defended to regulators and ownership.

In this feature, the Energy Manager is the primary user. They need weather-normalized EUI to make fair building-to-building comparisons across climate zones, track genuine year-over-year performance changes, and identify which buildings truly underperform — independent of geography.

---

## Domain context

| Term | What it means in this feature |
|------|------------------------------|
| **Weather normalization** | Statistical adjustment removing weather effects from energy data, enabling fair comparisons across climate zones and time periods |
| **HDD / CDD** | Heating / Cooling Degree Days — measures of how much heating or cooling weather demanded relative to a base temperature |
| **TMY** | Typical Meteorological Year — standard year of weather representing a location's typical climate, used as the normalization target |
| **Base temperature** | Outdoor temp at which a building needs neither heating nor cooling (default: 65°F / 18°C) |
| **EUI** | Energy Use Intensity — total energy per unit area (kBtu/sq ft/yr). The metric being normalized. See `energy/domain/concepts/eui.md` |
| **Site EUI / Source EUI** | Site = energy at the building boundary. Source = includes grid losses. Normalization applies to both |
| **Metric** | Batch-computed, entity-bound signal. Normalized EUI is a Metric. See `core/domain/glossary.md` |
| **KPI** | Metric + target + judgment. Normalized EUI compared to a benchmark becomes a KPI |
| **Benchmark** | Reference EUI (ASHRAE, ENERGY STAR, portfolio average, historical baseline) that normalized EUI is compared against. See `energy/domain/concepts/benchmarks.md` |
| **Baseline year** | The reference year against which current normalized performance is measured |
| **Peer group** | Buildings of the same type and climate zone used for percentile ranking |
| **Change-point regression** | Statistical method (ASHRAE Inverse Modeling) that models energy as a function of outdoor temperature, identifying the point where heating or cooling load begins |
| **Weather station mapping** | Assignment of each building to the nearest or most representative weather station for sourcing HDD/CDD data |
| **Normalization period** | The time window being normalized — typically rolling 12 months or calendar year |

**Constraints from the domain:**
- Weather data must be location-specific — each building must be mapped to a weather station based on its lat/lng
- Normalization requires at least 12 months of continuous meter data to build a reliable regression model
- TMY data must match the building's climate zone; using the wrong TMY invalidates the normalization
- Normalized EUI is a Metric (entity-bound, batch-computed) — not a Virtual Point
- Benchmarks used for comparison must also be weather-normalized or weather-independent to avoid double-counting
- Buildings with no weather-sensitive load (e.g. data centres with constant cooling) may not benefit from normalization and should be flagged

---

## What we are building

- **Weather data ingestion** — automatically pull HDD/CDD data for each building's location from a weather provider, mapped by lat/lng
- **Regression model per building** — build a change-point regression (ASHRAE Inverse Modeling) relating energy consumption to outdoor temperature for each building
- **Normalized EUI metric** — compute weather-normalized EUI for every building using TMY as the baseline climate, available as a standard Metric
- **Portfolio-wide normalized comparison** — rank and compare all buildings on a level playing field across climate zones, independent of geography
- **Weather impact breakdown** — show how much of each building's energy consumption is driven by weather vs. operations, so users can see where real savings opportunities exist

## What we are not building (this version)

- Predictive load forecasting based on weather forecasts
- Sub-daily (hourly) normalization — this version operates at monthly/annual granularity
- Custom TMY generation — we use standard TMY3 datasets
- User-driven weather station override — station mapping is automatic based on building lat/lng
- Normalization for non-energy metrics (water, waste, etc.)

---

## How it works

### Happy path

1. Building is onboarded with location (lat/lng) and has at least 12 months of continuous meter data
2. System automatically maps the building to the nearest weather station and pulls historical HDD/CDD data
3. System builds a change-point regression model for the building, relating monthly energy consumption to outdoor temperature
4. System computes weather-normalized EUI using TMY data as the baseline climate and publishes it as a Metric on the building entity
5. Energy manager opens the portfolio view and sees all buildings ranked by normalized EUI — comparable regardless of climate zone
6. Energy manager drills into a building to see the weather impact breakdown — how much energy is driven by weather vs. operations
7. Energy manager compares normalized EUI year-over-year to track genuine performance changes, independent of whether it was a hot or cold year

### Edge cases

| Situation | Behaviour |
|-----------|-----------|
| Fewer than 12 months of meter history | Show raw EUI only. Display a banner: "Weather normalization requires 12+ months of data — X months available." No regression model is built. |
| Weather data unavailable for a period | Interpolate from adjacent periods if gap is ≤ 30 days. If gap > 30 days, flag the normalized value as approximate with reduced confidence. |
| Building has no weather-sensitive load (e.g. data centre) | Regression model will show near-zero weather correlation. Flag the building as "weather-insensitive" and skip normalization — raw EUI is already a fair comparison. |
| Building location (lat/lng) is missing | Cannot map to a weather station. Show raw EUI only and prompt user to add location in the Building Graph. |
| Regression model has poor fit (low R²) | Flag the normalized value as low confidence. Display a warning: "Weather model is weak for this building — normalized EUI may be unreliable." |
| Multiple fuel types with partial data | Normalize only the fuel types with complete data. Show which fuel types are included and which are excluded from the normalized EUI. |
| Building changes use type mid-year | Use the most recent building type for benchmarking. Flag in the decision log that a use-type change occurred. |

---

## Design approach

**Principles:**
- **Show raw and normalized side by side** — never hide the raw data; let the user see both and understand the adjustment
- **Explain the weather effect, don't just adjust the number** — show how much weather contributed to consumption, not just the corrected result
- **Portfolio-first view** — the default view is all buildings ranked on a normalized basis; drill-down into individual buildings is secondary
- **Confidence transparency** — if the model is weak or data is incomplete, say so clearly rather than showing a misleading number

**Key screens / states:**

| Screen / state | What the user sees | Figma |
|---------------|-------------------|-------|
| Portfolio normalized ranking | All buildings ranked by weather-normalized EUI, with raw EUI shown alongside. Climate zone and confidence indicator per building. | — |
| Normalized EUI trend | Time-series chart showing raw EUI and normalized EUI side by side for a single building. Shaded area between the two lines represents the weather effect. | — |
| Weather impact summary | Breakdown of a building's energy into weather-driven and operations-driven portions. Shows HDD/CDD for the period, regression model fit (R²), and the magnitude of the weather adjustment. | — |
| Year-over-year comparison | Normalized EUI for current year vs. prior year(s). Highlights genuine performance improvement or degradation with weather removed. | — |
| Insufficient data state | Banner or card explaining why normalization is unavailable (< 12 months data, missing location, etc.) with guidance on what's needed. | — |

**Open design questions:**
| Question | Owner | Due |
|----------|-------|-----|
| Should the portfolio ranking default to normalized EUI or raw EUI? | Gresa Bytyqi | — |
| How do we visualize confidence levels — badge, colour, icon? | Gresa Bytyqi | — |
| Should weather-insensitive buildings be hidden from normalized views or shown with a distinct marker? | Gresa Bytyqi | — |

---

## Technical approach

**Architecture touchpoints:**

| Component | How it's involved |
|-----------|------------------|
| Building Graph | Provides building entity with location (lat/lng), floor area, building type, and climate zone. Normalized EUI Metric is written back to the building entity. |
| Timeseries Engine | Not directly involved — normalization uses batch meter data, not streaming data. |
| Rules Engine | Not used for normalization computation itself. Could be used downstream to fire findings when normalized EUI exceeds a benchmark threshold. |
| Library | Stores TMY3 datasets, standard base temperatures, regression model parameters, and benchmark reference values for normalized EUI comparison. |
| API Layer | Exposes normalized EUI metrics, portfolio rankings, weather impact breakdowns, and regression model metadata. Serves both frontend and AI agents. |
| Frontend | Portfolio ranking view, normalized EUI trend charts, weather impact breakdown, year-over-year comparison, and confidence indicators. |

**Key decisions:**
- Normalization runs as a batch Metric computation (Entity/Graph Engine), not in the Timeseries Engine — monthly/annual granularity, not real-time
- Change-point regression (ASHRAE Inverse Modeling) chosen as the normalization methodology — industry standard, auditable, well-understood by energy professionals
- TMY3 used as the standard climate baseline — widely available, no licensing cost, covers all US locations
- Weather station mapping is automatic (nearest station by lat/lng) — no manual override in v1

**ADRs:**
- None yet — regression methodology decision (ASHRAE IMT) may warrant a formal ADR if challenged

**Supporting documents:**
- [PRD](prd.md) — formal requirements, user stories, and acceptance criteria
- [Design Flows](flow.md) — step-by-step user journeys for all key flows

---

## Dependencies

| Dependency | Status | Owner |
|------------|--------|-------|
| 12+ months of meter data per building | Required | DataHub |
| Building location (lat/lng) in the Building Graph | Required | BMS |
| NOAA weather data integration (HDD/CDD by station) | To be built | DataHub / Engineering |
| TMY3 datasets loaded into Library | To be built | Library / Domain |
| ASHRAE Inverse Modeling Toolkit implementation | To be built | Engineering |
| Meter data pipeline (energy/utility-data feature) | Defining | DataHub |

---

## Open questions

| Question | Owner | Due | Answer |
|----------|-------|-----|--------|
| Which weather data provider? | Endrit Gojani | — | **Decided: NOAA** — free, US-only, public dataset. Global coverage deferred to future version. |
| What regression methodology? | Endrit Gojani | — | **Decided: ASHRAE Inverse Modeling Toolkit (IMT)** — industry standard change-point regression. |
| How frequently should normalized EUI be recomputed? | Endrit Gojani | — | |
| Should we expose regression model details (R², coefficients) to the user or keep it behind the scenes? | Gresa Bytyqi | — | |

---

## Knowledge contributions

- [ ] **Glossary** — "Weather normalization", "HDD/CDD", "TMY", "Change-point regression" may warrant entries in `core/domain/glossary.md`
- [ ] **Domain concepts** — Weather normalization is deep enough for its own concept file in `energy/domain/concepts/weather-normalization.md`
- [ ] **Architecture** — NOAA integration adds a new external data source not yet documented in `core/engineering/architecture.md`
- [ ] **Personas** — No new persona needs identified
- [ ] **Design patterns** — "Raw vs. normalized side-by-side" and "confidence indicator" could become reusable patterns in `core/design/design-system.md`

---

## Decision log

- **2026-03-10** Feature scoped.
- **2026-03-11** Feature kickoff. Portfolio-wide focus defined. Problem, users, domain context, scope, happy path, edge cases, design principles, key screens, architecture touchpoints, and key technical decisions documented. Weather provider decided: NOAA. Regression methodology decided: ASHRAE IMT. Owners assigned: Gresa Bytyqi (Design), Endrit Gojani (Tech Lead). PRD and design flows created.
