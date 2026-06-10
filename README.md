# California Healthcare Analytics Intelligence Dashboard

> A 4-page interactive Power BI dashboard analyzing California healthcare measures across all 58 counties (2018–2023), covering chronic disease burden, emergency department utilization, payer disparities, and demographic equity.

---

## The Problem

California's health plan enrollment data contains a wealth of information — but it is fragmented across dozens of measures, geographies, age bands, payer types, and years. Without a centralized analytical layer, it is difficult for health analysts and policymakers to:

- Identify which counties carry the highest chronic disease burden
- Understand how emergency department utilization differs by payer type and age
- Spot geographic and demographic inequities in health outcomes
- Track whether conditions are improving or worsening year over year

## The Solution

This project transforms raw California Health Plan Data (HPD) into a structured, interactive Power BI dashboard. The data was modeled using a star schema and enriched with custom DAX measures to deliver four purpose-built analytical views — each scoped to a different dimension of healthcare intelligence.

---

## Dataset

| File | Description |
|---|---|
| `hpd_measures_data_2018-2023_refresh.csv` | 523,584 rows of measure data across counties, age bands, sexes, payer types, and years |
| `measure-descriptions-july2025.csv` | Plain-language definitions for all 37 measures |
| `data-dictionary-measures-july2025-update.csv` | Field-level data dictionary |
| `measures-tech-note-august-2025-refresh-final-completed_ada.pdf` | Official technical methodology note |

**Source:** California Office of Health Care Affordability (OHCA) — Health Plan Data  
**Coverage:** 2018–2023 | 58 California Counties + 8 LA SPAs | 3 Payer Types | 6 Age Bands | 2 Sexes  
**Measures:** 37 across three categories — Demographics, Health Conditions, Utilization

### Measure Categories

| Category | Measure IDs | Examples |
|---|---|---|
| Demographics | 1–6, 37 | Medical Member Count, Enrollment Rates, Dual Enrollment |
| Health Conditions | 7–29, 36 | Diabetes, Hypertension, COPD, Cancer, Chronic Condition Prevalence |
| Utilization | 30–35 | ED Visit Rate, Inpatient Rate, Potentially Avoidable ED Visits |

### Suppression
44.1% of rows are suppressed per California privacy rules (small cell suppression). Suppressed rows have `suppression_ind = 'Y'` and null numerators — this is handled in the DAX layer to avoid treating nulls as zeros.

---

## Data Model

A star schema was built in Power BI with `Fact_Healthcare` at the center:

```
Fact_Healthcare
    ├── Dim_Geography    (county / LA SPA, full geography label)
    ├── Dim_AgeBand      (0–17, 18–34, 35–49, 50–64, 65–74, 75+)
    ├── Dim_Sex          (Female, Male)
    ├── Dim_Payer        (Commercial, Medi-Cal, Medicare)
    ├── dim_year         (Reporting Year)
    ├── dim_measure      (Measure ID, Name, Category)
    └── Measures_dax     (all DAX calculated measures)
```

**Key design decisions:**
- Scaling factors (1, 100, 12,000) are retained in the fact table and used in the `Standardized Rate` measure to normalize across measure types
- LA County and LA SPA rows are kept separate in `Dim_Geography` to prevent double-counting (per official methodology)
- `dim_year` is a calculated dimension derived from the fact table

---

## DAX Measures

| Measure | Purpose |
|---|---|
| `Total Population` | Sum of medical member-years (Measure 1) |
| `Standardized Rate` | `DIVIDE(numerator × scaling_factor, denominator)` — normalizes prevalence % and utilization per 1,000 |
| `Avg Healthcare Rate` | Average standardized rate across all selected measures |
| `Avg Chronic Disease Rate` | Average rate scoped to Health Conditions measures |
| `Avg Utilization Rate` | Average rate scoped to Utilization measures |
| `Highest Risk County` | County with highest average chronic disease rate |
| `Highest Burden Region` | Covered California region with highest disease burden |
| `Highest Utilization Region` | Region with highest average utilization rate |
| `Most Utilized Payer` | Payer type with highest ED/inpatient utilization |
| `Dominant Payer` | Payer type with highest enrollment share |
| `Largest Population Region` | Region with highest total member-years |
| `Total Counties` | Distinct count of counties in current filter context |
| `Total Regions` | Distinct count of Covered CA regions |
| `Total Measures` | Distinct count of measures in scope |
| `Total Chronic Measures` | Count of Health Conditions measures |
| `Total Utilization Measures` | Count of Utilization measures |

---

## Dashboard Pages

### Page 1 — Executive Healthcare Overview

**Purpose:** A high-level, cross-category summary for executives and stakeholders who need a single-pane view of California healthcare.

**KPI Cards:**
- Total Population (member-years)
- Standardized Rate (blended across all measures)
- Total Counties
- Total Measures
- Avg Healthcare Rate

**Visuals:**
- **California map** — geographic distribution of healthcare activity by county
- **Trend line by year** — Standardized Rate over 2018–2023, broken out by Measure Category (Demographics, Health Conditions, Utilization)
- **Bar chart by region** — Standardized Rate by Covered California region, segmented by category

**Design note:** No page-level filter is applied here, giving a full cross-category view.

---

### Page 2 — Chronic Disease Intelligence

**Purpose:** Drill into the burden of chronic and acute health conditions across geographies, payers, and age groups.

**Page filter:** `Measure Category = Health Conditions`

**KPI Cards:**
- Avg Chronic Disease Rate
- Total Chronic Measures
- Highest Burden Region
- Total Population
- Highest Risk County

**Visuals:**
- **Disease trend chart** — Prevalence rate over time by disease (line chart with measure slicer)
- **Disease map** — County-level disease prevalence bubbles
- **Payer analysis** — Chronic condition rates by payer type (Commercial vs. Medi-Cal vs. Medicare)
- **Age-band analysis** — How disease rates shift across age groups
- **County ranking** — Top counties by standardized chronic disease rate

**Slicers:** Year, Measure Name, Age Band, Sex, Payer Type

---

### Page 3 — Healthcare Utilization & Emergency Department Intelligence

**Purpose:** Analyze emergency department and inpatient utilization patterns — identifying overuse, avoidable visits, and high-burden populations.

**Page filter:** `Measure Category = Utilization`

**KPI Cards:**
- Avg Utilization Rate (per 1,000 members/year)
- Total Utilization Measures
- Highest Utilization Region
- Most Utilized Payer
- Total Population

**Visuals:**
- **Utilization trend** — Rate per 1,000 over 2018–2023 by measure type (ED, inpatient, avoidable ED)
- **Geographic map** — County-level utilization intensity
- **Payer utilization** — How utilization differs by Commercial / Medi-Cal / Medicare
- **Age-based utilization** — Clustered bar showing utilization rates by age band
- **Top counties** — County ranking by utilization rate

**Slicers:** Year, Measure Name, Payer Type, Age Band, Sex

---

### Page 4 — Healthcare Equity & Demographic Analysis

**Purpose:** Examine who is covered and how population composition varies by geography, sex, age, and payer — surfacing equity gaps.

**Page filter:** `Measure Category = Demographics`

**KPI Cards:**
- Total Population
- Total Regions
- Total Counties
- Largest Population Region
- Dominant Payer

**Visuals:**
- **Gender analysis** — Total population by sex (column chart)
- **Age distribution** — Population by age band (bar chart)
- **Payer distribution** — Enrollment share by payer type (donut chart)
- **Regional equity** — Population by Covered California region (clustered bar)
- **Demographic heatmap** — Age × Sex matrix (pivot table) showing member-year counts

---

## Tools & Technologies

| Tool | Use |
|---|---|
| Power BI Desktop | Dashboard development, data modeling, DAX |
| DAX | Calculated measures and KPIs |
| Power Query (M) | Data transformation and loading |
| Star Schema | Data model architecture |
| Azure Maps | Geographic visualization |
| GitHub | Version control and portfolio hosting |

---

## How to Open

1. Download `HD.pbix`
2. Open with [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (free)
3. The data is embedded — no additional connection required
4. Use slicers to filter by year, age band, sex, payer type, and measure

---

## Project Structure

```
california-healthcare-analytics-intelligence-dashboard/
│
├── HD.pbix                                          # Power BI dashboard file
├── hpd_measures_data_2018-2023_refresh.csv          # Raw source data (523K rows)
├── measure-descriptions-july2025.csv                # Measure definitions
├── data-dictionary-measures-july2025-update.csv     # Field-level data dictionary
├── measures-tech-note-august-2025-refresh-final-completed_ada.pdf  # Methodology
└── README.md
```

---

## Author

**Veekshith Reddy Ravula**  
[GitHub](https://github.com/veekshith21)

---

## License

Data sourced from California OHCA (public dataset). Dashboard and analysis by Veekshith.
