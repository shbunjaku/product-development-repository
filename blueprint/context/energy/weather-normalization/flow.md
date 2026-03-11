# Weather Normalization — Design Flows

| Field        | Value |
|--------------|-------|
| Author       | Gresa Bytyqi |
| Status       | Draft |
| Last updated | 2026-03-11 |
| Product      | energy |
| PRD          | [prd.md](prd.md) |
| Figma        | — |

---

## Flow 1: Portfolio Normalized Ranking

### Overview

The primary flow. The Energy Manager opens the portfolio energy view and compares all buildings on a weather-normalized basis.

**Persona:** Energy Manager
**Entry point:** Portfolio energy dashboard → "Normalized view" toggle or tab
**Success end state:** Energy Manager sees all buildings ranked by normalized EUI, identifies underperformers, and understands the weather adjustment for each

### Flow diagram

```
Portfolio dashboard
       │
       ▼
┌─────────────────────┐
│ Toggle: Raw / Norm  │
└─────────────────────┘
       │
       ▼
┌─────────────────────────────────┐
│ Portfolio Normalized Ranking    │
│                                 │
│  Building A  ████████░░  62     │  ← Normalized EUI + raw EUI
│  Building B  █████████░  71     │     Climate zone badge
│  Building C  ██████████  89     │     Confidence indicator
│  Building D  ░░░░░░░░░░  --     │  ← Insufficient data
└─────────────────────────────────┘
       │
       ├──── Click building ──── ▶ Flow 2 (Building detail)
       │
       └──── Click "insufficient data" ──── ▶ Flow 4 (Insufficient data)
```

### Steps

#### 1. Open portfolio energy dashboard
- **User sees:** Portfolio energy dashboard with the current raw EUI ranking (existing view)
- **User does:** Clicks "Normalized" toggle/tab to switch to weather-normalized view
- **System does:** Fetches normalized EUI Metrics for all buildings in the portfolio. Falls back to raw EUI for buildings without normalization.
- **Edge cases:** If no buildings have normalization yet, show empty state with explanation of what's needed (12+ months data + location)

#### 2. Review normalized ranking
- **User sees:** All buildings ranked by normalized EUI (lowest = best performing). Each row shows:
  - Building name
  - Normalized EUI value
  - Raw EUI value (secondary, greyed)
  - Climate zone badge (e.g. "4A — Mixed-Humid")
  - Confidence indicator (high / medium / low)
  - Trend arrow (vs. prior period)
- **User does:** Scans the ranking to identify underperformers. May sort or filter by climate zone, confidence, or trend.
- **System does:** Ranks buildings by normalized EUI. Groups buildings without normalization at the bottom with an explanation.
- **Edge cases:** Mixed portfolio where some buildings have normalization and others don't — non-normalized buildings appear in a separate "Raw EUI only" section below the ranked list

#### 3. Drill into a building
- **User sees:** A building row that looks concerning (high normalized EUI or degrading trend)
- **User does:** Clicks the building row
- **System does:** Navigates to building-level weather normalization detail (Flow 2)
- **Edge cases:** If building has low-confidence normalization, show a subtle warning before drill-in

---

## Flow 2: Building Normalized EUI Detail

### Overview

The Energy Manager drills into a single building to understand its weather-adjusted performance in depth.

**Persona:** Energy Manager
**Entry point:** Click a building from the portfolio normalized ranking (Flow 1)
**Success end state:** Energy Manager understands the building's true performance, the weather adjustment magnitude, and the year-over-year trend

### Flow diagram

```
Building detail view
       │
       ▼
┌──────────────────────────────────────┐
│ Normalized EUI Trend                 │
│                                      │
│  ▓▓▓ Raw EUI    ███ Normalized EUI   │
│                                      │
│  100 ─ ▓                             │
│   90 ─ ▓▓    ▓▓                      │
│   80 ─ ██▓▓██▓▓▓                     │
│   70 ─ ████████████                  │
│        J F M A M J J A S O N D       │
│                                      │
│  Shaded area = weather effect        │
└──────────────────────────────────────┘
       │
       ▼
┌──────────────────────────────────────┐
│ Weather Impact Summary               │
│                                      │
│  Weather-driven:    38%  ████░░░░░░  │
│  Operations-driven: 62%  ██████░░░░  │
│                                      │
│  HDD this year: 4,200 (TMY: 4,800)  │
│  CDD this year: 1,100 (TMY: 950)    │
│  Model fit (R²): 0.87               │
└──────────────────────────────────────┘
       │
       ▼
┌──────────────────────────────────────┐
│ Year-over-Year Comparison            │ ──── Flow 3
└──────────────────────────────────────┘
```

### Steps

#### 1. View normalized EUI trend
- **User sees:** Time-series chart (monthly) showing raw EUI and normalized EUI as two lines. The area between them is shaded to represent the weather effect. Period selector (12M, 24M, 36M, YTD).
- **User does:** Reads the chart to understand how much weather inflates or deflates EUI each month. May change the time period.
- **System does:** Renders both raw and normalized EUI from Metrics. Computes the shaded weather-effect area as the difference.
- **Edge cases:** Months with missing data show a dashed line with a tooltip explaining the gap. Months before normalization is available show raw EUI only.

#### 2. Review weather impact summary
- **User sees:** A card showing:
  - Weather-driven energy percentage vs. operations-driven percentage (bar or donut)
  - Actual HDD/CDD for the period vs. TMY HDD/CDD
  - Regression model fit (R²)
  - Confidence level (derived from R² and data completeness)
- **User does:** Assesses whether weather is a major factor for this building. A high weather-driven % means the building is sensitive to climate; a low % means weather normalization changes little.
- **System does:** Pulls regression model metadata and weather data from the API. Computes weather-driven vs. operations-driven split.
- **Edge cases:** If R² is below threshold, show a warning banner: "Weather model is weak for this building — results may be unreliable."

#### 3. Scroll to year-over-year comparison
- **User sees:** Year-over-year section (see Flow 3)
- **User does:** Continues scrolling or clicks to expand
- **System does:** Loads comparison data

---

## Flow 3: Year-over-Year Normalized Comparison

### Overview

The Energy Manager compares a building's normalized performance across years to assess genuine improvement or degradation.

**Persona:** Energy Manager
**Entry point:** Year-over-year section within building detail (Flow 2) or direct link
**Success end state:** Energy Manager confirms whether a building's performance genuinely improved or degraded, independent of weather

### Flow diagram

```
┌──────────────────────────────────────────┐
│ Year-over-Year Comparison                │
│                                          │
│           2024        2025       Δ       │
│ Raw EUI   92          98        +6.5%    │
│ Norm EUI  78          81        +3.8%    │
│ Weather   Mild year   Harsh     —        │
│                                          │
│ Verdict: ⚠ Performance degraded 3.8%     │
│          (weather explains 2.7% of the   │
│           raw increase)                  │
└──────────────────────────────────────────┘
```

### Steps

#### 1. View year-over-year table
- **User sees:** Table showing raw EUI and normalized EUI for each year, with the delta between them. A plain-language verdict: "Performance improved X%" or "Performance degraded X%". Weather context: whether the year was milder or harsher than TMY.
- **User does:** Reads the verdict. Understands that the raw EUI change is partially weather, partially operational.
- **System does:** Pulls normalized EUI Metrics for each year. Computes deltas. Generates a plain-language verdict based on the normalized delta direction and magnitude.
- **Edge cases:** First year of data — no comparison available, show "Baseline year — comparison available next year." Partial year — show YTD with a note that the comparison is incomplete.

#### 2. Expand detail (optional)
- **User sees:** Expandable section with monthly breakdown for each year
- **User does:** Clicks to expand if they want to identify which months drove the change
- **System does:** Renders a monthly comparison table or small multiples chart
- **Edge cases:** Missing months in one year but not the other — align by calendar month, show gaps

---

## Flow 4: Insufficient Data State

### Overview

A building cannot be normalized. The system explains why and tells the user what to do.

**Persona:** Energy Manager
**Entry point:** Click on a building marked as "insufficient data" in the portfolio ranking, or land on a building detail page where normalization is unavailable
**Success end state:** Energy Manager understands what's missing and knows the next action to get normalization working

### Flow diagram

```
┌──────────────────────────────────────────┐
│ ⚠ Weather Normalization Unavailable      │
│                                          │
│ This building cannot be weather-          │
│ normalized because:                      │
│                                          │
│  ☐ Location (lat/lng) is missing         │
│    → Add location in Building Settings   │
│                                          │
│  ☐ Only 7 of 12 required months of      │
│    meter data available                  │
│    → 5 more months needed (est. Aug 26)  │
│                                          │
│  [Add Location]  [View Raw EUI]          │
└──────────────────────────────────────────┘
```

### Steps

#### 1. See the blockers
- **User sees:** A card listing each blocker with a clear explanation:
  - Missing location → "Add location in Building Settings" with a direct link
  - Insufficient meter data → "X of 12 months available — estimated availability date"
  - Weather data unavailable → "Weather data cannot be retrieved — contact support"
- **User does:** Reads the blockers and decides which to act on
- **System does:** Checks each prerequisite (lat/lng exists, months of meter data, weather station reachable) and renders the status of each
- **Edge cases:** Multiple blockers at once — show all of them, ordered by actionability (user-fixable first, system-dependent second)

#### 2. Take action
- **User sees:** Action buttons — "Add Location" (links to Building Graph settings), "View Raw EUI" (dismisses the banner and shows raw data)
- **User does:** Either fixes the blocker or decides to work with raw EUI for now
- **System does:** If location is added, queues the building for weather station mapping on the next batch run. If meter data threshold is met later, normalization will appear automatically.
- **Edge cases:** User adds location but weather station is unreachable — show a different error: "Location set, but weather data is temporarily unavailable"

---

## Error paths

| Flow | Error | What happens |
|------|-------|-------------|
| Flow 1 | API timeout loading normalized metrics | Show cached values with a "Last updated: [timestamp]" label. Retry in background. |
| Flow 1 | No buildings in portfolio have normalization | Show empty state: "No buildings have weather normalization yet. Buildings need a location and 12+ months of meter data." |
| Flow 2 | Regression model not yet computed | Show a loading state: "Weather model is being computed — check back in [estimated time]." |
| Flow 2 | Weather data fetch from NOAA fails | Show raw EUI with a banner: "Weather data temporarily unavailable. Normalized values may be stale." |
| Flow 3 | Only one year of data available | Show single year with note: "Baseline year — year-over-year comparison available next year." |
| Flow 4 | User adds location but no weather station within threshold distance | Show warning: "Nearest weather station is [X] miles away — normalization may be less accurate." |

---

## Open questions

| Question | Owner | Due |
|----------|-------|-----|
| Should Flow 1 default to the normalized view or require an explicit toggle? | Gresa Bytyqi | — |
| In Flow 2, should the weather impact summary be a donut chart, stacked bar, or simple percentage cards? | Gresa Bytyqi | — |
| In Flow 3, what language should the verdict use? Should it include a severity (minor / significant / critical)? | Gresa Bytyqi | — |
| In Flow 4, should "estimated availability date" be computed from current meter data ingestion rate? | Endrit Gojani | — |
| Should there be a Flow 5 for exporting normalized data for compliance reporting? | Gresa Bytyqi | — |
