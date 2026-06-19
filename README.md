# NHS GP Appointment Data Analysis
### End-to-End Data Analytics Project | Python | Pandas | Matplotlib | Seaborn

[![Python](https://img.shields.io/badge/Python-3.10+-blue.svg)](https://www.python.org/)
[![Pandas](https://img.shields.io/badge/Pandas-1.5+-green.svg)](https://pandas.pydata.org/)
[![Matplotlib](https://img.shields.io/badge/Matplotlib-3.6+-orange.svg)](https://matplotlib.org/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Complete-brightgreen.svg)]()

---

## Project Overview

This project delivers a full end-to-end data analytics investigation into NHS GP appointment data across **42 Integrated Care Boards (ICBs)** in England, covering **742.8 million appointments** over a 30-month period (January 2020 to June 2022).

The analysis answers two core business questions posed by NHS England:

> **1. Has there been adequate staff and capacity in the networks?**
> **2. What was the actual utilisation of resources?**

The central finding challenges the conventional narrative: **this is not a capacity problem requiring capital investment, it is an operational problem that can be solved through smarter booking policy, expanded telephone appointments, and targeted ICB-level interventions.**

---

## Business Context

The NHS incurs significant, potentially avoidable, costs when patients miss GP appointments. As BMA chair Professor Philip Banfield stated, the reasons why this happens should be investigated rather than simply resorting to punishing patients, since financial penalties disproportionately affect the poorest and most vulnerable communities. This project takes a data-driven approach to understanding those reasons and translating them into actionable, costed recommendations.

---

## Key Findings

| Metric | Value |
|--------|-------|
| Total appointments analysed | 742.8 million |
| DNA (missed) appointments | 30.9 million (4.16%) |
| Estimated annual DNA cost | GBP 371 million |
| Same-day DNA rate | 1.77% |
| 28+ day booking DNA rate | 8.83% (5x higher) |
| Telephone DNA rate | 1.96% |
| Face-to-face DNA rate | 5.51% (2.8x higher) |
| Best performing ICB | Northamptonshire (Score: 82.7) |
| Worst performing ICB | Greater Manchester (Score: 27.2) |
| Recoverable savings (OpEx only) | GBP 196 million per year |

---

## The Core Insight: OpEx, Not CapEx

Three pieces of evidence prove the NHS does not need to build more, it needs to operate smarter:

1. **46.1% of appointments are already same-day** with a DNA rate of just 1.77%. Physical capacity exists.
2. **Telephone appointments grew 151% during COVID** (3.5M to 8.8M per month) with zero new infrastructure. Operational flexibility exists.
3. **DNA rates vary by 3.15 percentage points across 42 ICBs** using identical national infrastructure. Same buildings, very different results. That is an operational gap, not a capital one.

The vicious cycle identified: limited appointment slots force patients to book far in advance, long-wait bookings generate DNA rates nearly five times higher than same-day bookings, wasted slots reduce effective capacity further, and the cycle repeats. Breaking this cycle requires operational change rather than new buildings.

---

## What Was Analysed

### Datasets

| File | Description | Rows | Period |
|------|-------------|------|--------|
| `actual_duration.csv` | Appointment duration at sub-ICB level | 137,793 | Dec 2021 to Jun 2022 |
| `appointments_regional.csv` | Status, mode, HCP type, booking lead time by ICB | 596,821 | Jan 2020 to Jun 2022 |
| `national_categories.xlsx` | Service setting, context type, national category | 817,394 | Aug 2021 to Jun 2022 |
| `tweets.csv` | NHS-related Twitter sample data | 1,174 | N/A |

> **Note:** The original dataset is not included in this repository due to licensing/course restrictions. The notebook expects these files to be placed in a local `data/` folder before running.

### Data Quality Notes
- `count_of_appointments` is a **sum** of appointment numbers, not a row count
- Totals are **estimates** because not all GP practices are included
- **Unknown appointment mode**: Cegedim system practices cannot supply mode data, increasing Unknown entries from July 2019
- **HCP type**: Only the GP category is reliably captured; all other roles are grouped as Other Practice Staff
- **Duration data**: Only available from December 2021; durations under 1 minute or over 60 minutes are labelled Unknown
- **Unmapped/Unknown categories**: Known structural limitations of GP recording systems, not analysis errors

---

## Analysis Modules

### Segment 1 — Planning and Business Context
- Business question framing using the NHS assignment brief
- Evidence review from BJGP research and NHS England publications (5% of appointments missed annually, costing GBP 216M)
- OpEx vs CapEx investment thesis established upfront

### Segment 2 — Data Import and Quality Assessment
- Import using pandas with explicit date format handling
- Data quality documentation across all four datasets
- Descriptive statistics and unique value checks for every categorical column

### Segment 3 — Data Wrangling and Cross-Analysis
- Date column conversion with timezone conflict resolution (a recurring technical issue solved during the project)
- DNA rate calculated across four dimensions: appointment mode, booking lead time, HCP type, and ICB region
- Booking lead time identified as the strongest DNA predictor

### Segment 4 — Visualisation and Trend Analysis
- Charts built using the NHS official colour palette (Blue #005EB8, Green #009639, Red #DA291C)
- Audience labels on every chart (Business, Technical, or Both)
- COVID-19 impact analysis with annotated monthly trend showing a 40%+ drop and full recovery

### Segment 5 — Twitter and Social Media Analysis
- Hashtag extraction using Python regex (`re.findall(r'#(\w+)', text)`)
- Theme classification: NHS/Healthcare, Mental Health, Workforce
- Ethical limitations documented (Twitter users not representative of the general population, supplement not replace primary data)

### Segment 6 — Business Questions, ICB Ranking, Recommendations
- Vicious cycle analysis: limited slots leads to long waits leads to high DNA leads to reduced capacity
- Composite performance scoring built for all 42 ICBs
- OpEx vs CapEx investment framework with annualised savings estimates
- 8 ranked recommendations with financial impact

---

## ICB Performance Ranking

A composite performance score (0 to 100) was calculated for all 42 ICBs using four weighted metrics:

| Metric | Weight | Direction |
|--------|--------|-----------|
| DNA rate | 40% | Lower is better |
| Same-day booking rate | 30% | Higher is better |
| Telephone utilisation rate | 20% | Higher is better |
| Unknown status rate | 10% | Lower is better |

**Top 5 ICBs:** Northamptonshire, Cambridgeshire and Peterborough, Herefordshire and Worcestershire, Coventry and Warwickshire, Frimley

**Bottom 5 ICBs:** South East London, Dorset, South Yorkshire, Black Country, Greater Manchester

**Key financial insight:** If Greater Manchester alone matched Northamptonshire's DNA rate, 933,808 appointments would be recovered, saving approximately GBP 28 million from that one ICB alone.

A quadrant analysis (DNA rate vs same-day booking rate) further classified each ICB by problem type, identifying which ICBs need urgent operational intervention versus which have access issues versus booking policy issues.

---

## Recommendations

| Priority | Type | Action | Evidence | Annual Impact |
|----------|------|--------|----------|--------------|
| 1 | OpEx | Reduce forward booking time to same-day or next-day | DNA 1.77% (same-day) vs 8.83% (28+ days) | GBP 19.5M |
| 2 | OpEx | Expand telephone for routine and follow-up care | Telephone DNA 1.96% vs face-to-face 5.51% | GBP 196M potential |
| 3 | OpEx | SMS reminders 24 to 48 hours before appointments | Sussex ICB reduced DNA from 6.5% to 5.5% | GBP 15-30M |
| 4 | OpEx | Target high-DNA ICBs with operational review | 3.15pp gap across 42 ICBs, identical infrastructure | GBP 50M+ |
| 5 | OpEx | Link DNA rates to deprivation index (IMD) | Most deprived patients 10-20% more likely to miss | Equity focused |
| 6 | OpEx | Improve data recording quality | 4.6% Unknown status limits reporting accuracy | Enables future insight |
| 7 | CapEx | Scale video and online appointment infrastructure | Proven during COVID, only 0.49% of appointments | Medium-term |
| 8 | CapEx | New GP surgery capacity if OpEx gap persists | Reassess after 12-24 months of OpEx improvements | Deferred |

---

## Deliverables Produced

1. **Jupyter Notebook** (`Sista_Sharath_DA201_Assignment_Notebook.ipynb`) — Full end-to-end analysis across all six modules, with code, markdown commentary, and 15+ charts
2. **Technical Report** (Word/PDF, ~1,100 words) — Aimed at technical stakeholders for validating and reproducing results
3. **Business Presentation** (PowerPoint, 14 slides) — Aimed at business stakeholders, NHS colour palette, data-driven storytelling
4. **Reusable Python Modules** — `data_loader.py`, `metrics.py`, `visualisations.py` for clean, modular, production-style code
5. **15 Chart Outputs** (PNG) — All visualisations saved at high resolution for reuse in reports and presentations

---

## Charts Produced

| # | Chart | Description |
|---|-------|-------------|
| 1 | Appointment Status Overview | Bar and donut chart of Attended / DNA / Unknown split |
| 2 | Monthly Appointment Trend | COVID-19 dip and recovery, annotated peak and trough |
| 3 | Appointment Mode Trends | Face-to-Face vs Telephone vs Video/Online over time |
| 4 | DNA Rate by Booking Lead Time | The key finding: 1.77% same-day vs 8.83% at 28+ days |
| 5 | DNA Rate by Mode | Telephone (1.96%) vs Face-to-Face (5.51%) |
| 6 | Monthly DNA Rate Trend | DNA rate and absolute count over time |
| 7 | Service Setting and Context Type | Breakdown of appointment categories |
| 8 | Top National Categories | Clinical activity types by volume |
| 9 | Appointment Duration Distribution | Consultation length analysis |
| 10 | Twitter Hashtag Analysis | Top hashtags and engagement patterns |
| 11 | OpEx Evidence Dashboard | Same-day volume, COVID telephone surge, ICB variation |
| 12 | Utilisation Savings Dashboard | Mode breakdown and OpEx savings potential |
| 13 | Vicious Cycle Diagram | Visual explanation of the capacity-DNA feedback loop |
| 14 | ICB Full Performance Ranking | All 42 ICBs ranked by composite score |
| 15 | ICB Performance Quadrant | DNA rate vs same-day rate scatter plot |

---

## Technical Stack

```
pandas>=1.5.3
numpy>=1.24.0
matplotlib>=3.7.0
seaborn>=0.12.2
openpyxl>=3.1.0
re (standard library)
collections (standard library)
```

---

## Skills Demonstrated

- **Data Wrangling** — handling timezone conflicts, mixed date formats, missing values, multi-file merging
- **Exploratory Data Analysis** — cross-dimensional analysis across 742M+ records
- **Data Visualisation** — 15 charts with a consistent design system and explicit audience targeting
- **Statistical Analysis** — normalised composite scoring, DNA rate modelling, savings calculations
- **Business Communication** — translating data findings into financial impact and investment recommendations
- **External Data Integration** — social media (Twitter) analysis with ethical framing
- **Domain Knowledge** — NHS ICB structure, GP appointment systems, OpEx vs CapEx investment framing
- **Technical Troubleshooting** — resolved multiple pandas datetime and timezone conversion errors during development

---

## Author

**Sharath Sista**
Data Analyst | LSE DA201

[![GitHub](https://img.shields.io/badge/GitHub-sharathsista-black.svg)](https://github.com/sharathsista)

---

## License

This project is licensed under the MIT License.

> **Disclaimer:** The scenario used in this analysis is based on a fictitious assignment brief. NHS appointment data used is publicly available from NHS England. Twitter data is provided as an educational example only and is not representative of the general population.
