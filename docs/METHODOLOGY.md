# Methodology

This document explains the analytical approach, formulas, and assumptions used throughout the NHS GP Appointment Data Analysis project. It is intended to let a technical reviewer verify the analysis without needing to read the full notebook.

---

## 1. Data Cleaning Decisions

### Date Conversion
Each dataset stored dates differently, which required different handling:

- **actual_duration.csv** — dates were stored as strings in the format `01-Dec-21`. Converted using:
  ```python
  pd.to_datetime(df['appointment_date'], format='%d-%b-%y')
  ```

- **appointments_regional.csv** — `appointment_month` was stored as a `YYYY-MM` string. Converted directly to a pandas `Period` object using:
  ```python
  pd.to_datetime(df['appointment_month']).dt.to_period('M')
  ```

- **national_categories.xlsx** — Excel exports often carry timezone-aware datetime objects, which caused repeated `TypeError` failures when re-converted. The fix was to standardise to UTC first, then strip the timezone:
  ```python
  pd.to_datetime(df['appointment_date'], utc=True).dt.tz_localize(None)
  ```
  `appointment_month` was then **derived from** `appointment_date` rather than converted independently, avoiding the conflict entirely.

### Why No Rows Were Removed
All `Unknown`, `Unmapped`, and `Inconsistent Mapping` entries were **retained** rather than dropped. These are documented, structural limitations of NHS GP recording systems (confirmed by the official metadata file and the UK LLC NHS England coding documentation), not data entry errors. Removing them would understate the true scale of data quality issues and bias the DNA rate calculations.

### ICB Name Extraction
`appointments_regional.csv` only contains `icb_ons_code`, not a readable name. ICB names were recovered by extracting them from the `sub_icb_location_name` column in `national_categories.xlsx` using a regex pattern that strips the sub-ICB suffix:
```python
icb_map['icb_name'] = icb_map['sub_icb_location_name'].str.extract(r'^(.*?)\s*-\s*\w+$')[0]
```
This mapping was then merged into `appointments_regional.csv` by `icb_ons_code`.

---

## 2. Core Metric: DNA Rate

DNA (Did Not Attend) rate is the single most important metric in this analysis. It is calculated consistently throughout as:

```
DNA Rate (%) = (Sum of count_of_appointments where status = DNA)
               ÷ (Sum of count_of_appointments, all statuses)
               × 100
```

This was calculated at five different levels of granularity to identify where DNA is concentrated:
1. **Overall** (4.16% across all 742.8M appointments)
2. **By appointment mode** (Telephone 1.96% vs Face-to-Face 5.51%)
3. **By booking lead time** (Same Day 1.77% vs 28+ Days 8.83%)
4. **By HCP type** (GP vs Other Practice Staff)
5. **By ICB region** (range of 2.69% to 5.84% across 42 ICBs)

Note: `count_of_appointments` is a **sum** of appointment volumes, not a count of rows. This was confirmed in the official metadata file. All aggregations use `.sum()` accordingly, never `.count()`.

---

## 3. Composite ICB Performance Score

To rank all 42 ICBs on a single comparable scale, four metrics were normalised to a 0–100 range and combined using fixed weights.

### Metrics and Weights

| Metric | Weight | Rationale |
|--------|--------|-----------|
| DNA rate | 40% | Primary cost driver and the central focus of the NHS business question |
| Same-day booking rate | 30% | Direct evidence of booking system efficiency and the strongest predictor of low DNA |
| Telephone utilisation rate | 20% | Reflects operational flexibility and proven low-DNA-risk delivery mode |
| Unknown status rate | 10% | Secondary indicator of data and operational discipline at the ICB |

### Normalisation Formula

Each metric was scaled to 0–100 using min-max normalisation across the 42 ICBs:

```python
def normalise(series, invert=False):
    mn, mx = series.min(), series.max()
    norm = (series - mn) / (mx - mn) * 100
    return 100 - norm if invert else norm
```

`invert=True` was applied to DNA rate and Unknown rate, since for these metrics a **lower** value should produce a **higher** score.

### Composite Score Formula

```
Composite Score = (DNA Score × 0.40)
                 + (Same-Day Score × 0.30)
                 + (Telephone Score × 0.20)
                 + (Unknown Score × 0.10)
```

Scores range from 0 to 100. ICBs were then ranked descending, and grouped into four bands for visualisation:
- 65+ : High performing
- 50–64 : Good performing
- 40–49 : Below average
- Under 40 : Poor performing

### Limitation
The weighting (40/30/20/10) is a reasoned judgement, not a statistically derived optimum. It reflects the priority order established by the business question (DNA cost first, then booking efficiency, then mode flexibility, then data quality). A sensitivity analysis varying these weights was not performed and is listed as a recommended next step.

---

## 4. Financial Savings Estimation

All financial figures use a constant cost assumption of **GBP 30 per missed GP appointment**, sourced from NHS England's published estimate of the cost of a missed appointment.

### Annualisation
The dataset spans 30 months. To convert a 30-month total into an annual figure:
```
Annual Saving = (30-Month Saving) ÷ 2.5
```

### Scenario 1 — Fixing Booking Lead Time
Calculates the appointments that would be recovered if 28+ day bookings had the same DNA rate as same-day bookings:
```python
recoverable_booking = long_wait_appointments * (dna_rate_28plus - dna_rate_sameday)
saving = recoverable_booking * 30  # GBP per appointment
annual_saving = saving / 2.5
```
Result: ~1.63M appointments recovered, ~GBP 19.5M per year.

### Scenario 2 — Matching All Modes to Telephone's DNA Rate
Calculates the appointments that would be recovered if every appointment mode performed as well as telephone (1.96% DNA):
```python
recoverable_mode = total_appointments * (current_overall_dna_rate - telephone_dna_rate)
saving = recoverable_mode * 30
annual_saving = saving / 2.5
```
Result: ~16.35M appointments recovered, ~GBP 196M per year.

### Limitation
Both scenarios assume the *marginal* DNA rate for shifted appointments would match the target rate exactly. In reality, some patients may have higher inherent DNA risk regardless of lead time or mode (for example, due to deprivation or clinical complexity), so realised savings would likely be lower than the theoretical maximum. This is flagged as an assumption, not a guarantee.

---

## 5. Twitter Hashtag Extraction

Hashtags were extracted directly from raw tweet text using regex, rather than relying solely on the `tweet_entities_hashtags` field, because a portion of tweets had a null value in that field despite containing hashtags in the visible text:

```python
def extract_hashtags(text):
    if pd.isna(text):
        return []
    return re.findall(r'#(\w+)', str(text), re.IGNORECASE)
```

All hashtags were lowercased before counting to avoid `#NHS` and `#nhs` being treated as different tags. Theme classification (NHS/Healthcare, Mental Health, Workforce, Other) was applied using simple keyword matching against the hashtag text, for visualisation purposes only, not formal NLP classification.

### Limitation
This is frequency-based hashtag counting, not sentiment analysis. No claim is made about whether tweets are positive or negative toward the NHS. A recommended next step is applying a proper sentiment model (e.g. VADER or a transformer-based classifier) to the tweet text.

---

## 6. General Assumptions and Limitations

- All appointment totals are **NHS England estimates**, not a full census, since not all GP practices submit data
- The dataset only covers **England**; findings should not be generalised to other UK nations
- `actual_duration.csv` only covers December 2021 to June 2022, so duration-based findings have a shorter and more recent time window than the rest of the analysis
- The composite ICB score does not include deprivation (IMD) data, population size, or rurality, all of which could explain part of the performance variation; this is the top recommended extension to the model
- Cost estimates use a flat GBP 30 figure and do not vary by HCP type, region, or appointment complexity

---

## 7. Tools and Versions

| Tool | Purpose |
|------|---------|
| pandas | Data loading, cleaning, aggregation |
| numpy | Numerical operations, normalisation |
| matplotlib | Chart rendering |
| seaborn | Chart styling and themes |
| openpyxl | Reading the `.xlsx` source file |
| re (standard library) | Hashtag extraction |
| collections.Counter (standard library) | Hashtag frequency counting |

See `requirements.txt` for minimum version numbers.
