# Optimising NYC Taxi Operations — Exploratory Data Analysis

An end-to-end EDA of ~37 million NYC Yellow Taxi trips from 2023, 
conducted to uncover demand patterns, operational inefficiencies and 
pricing dynamics that could inform strategy for a taxi operator 
entering the New York market.

---

## Problem Statement

Taxi companies in NYC operate in a dynamic environment where demand 
shifts dramatically by hour, day, season and neighbourhood. The goal 
of this analysis was to answer three business questions:

1. **Routing & dispatching** — when and where is demand highest, and 
   where are the operational inefficiencies?
2. **Fleet positioning** — which zones should cabs be positioned in, 
   and how does that change through the day?
3. **Pricing** — where is revenue being maximised, and is there room 
   to compete on price?

---

## Dataset

Source: [NYC Taxi & Limousine Commission (TLC) Trip Record Data](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

| Item | Detail |
|---|---|
| Files | 12 Parquet files (one per month, 2023) + taxi zones shapefile |
| Raw size | ~37 million trip records |
| Columns | 19 — vendor, timestamps, zones, distance, fare, tips, surcharges, payment type |
| Zones | 263 TLC taxi zones across 5 boroughs |

**Note:** the raw data is not committed to this repo due to file size 
limits. Download it from the TLC link above and place it in a `data/` 
folder to reproduce the analysis.

---

## Approach

### 1. Sampling

Loading 37M rows into memory is impractical. Rather than sampling 
randomly — which risks over-representing busy hours and 
under-representing quiet ones — a **stratified sample** was taken: 
the data was grouped by pickup *date* and *hour*, and 5% of trips 
were drawn from each date-hour group.

This guarantees every hour of every day across all 12 months is 
proportionally represented, preserving the temporal structure that 
the entire analysis depends on.

### 2. Data Cleaning

Several non-obvious data quality issues surfaced:

- **Duplicate airport fee columns** — January 2023 used `airport_fee` 
  while the other 11 months used `Airport_fee`. Concatenation produced 
  two half-empty columns, which were coalesced into one.

- **79 rows with negative surcharges** — all from a single vendor, 93.7% 
  cash payments, `fare_amount = 0`, and surcharge values that contradicted 
  the data dictionary (e.g. `improvement_surcharge` of -$1.00 where the 
  spec defines $0.30). Since surcharges cannot logically exist without a 
  metered fare, these were treated as invalid records and removed.

- **Systematic missing values** — five columns each had exactly 3.42% 
  missing values, all on the *same rows*, indicating a systematic recording 
  failure rather than random gaps. Imputed with mode where the underlying 
  trip was clearly valid.

- **Invalid sentinel values** — `RatecodeID = 99` (not in the data 
  dictionary's 1–6 range) and `payment_type = 0` (undefined) were 
  identified and handled separately from true nulls.

- **Physically impossible outliers** — including a 126,360-mile trip 
  (five times the Earth's circumference) and $300+ fares on sub-mile trips.

### 3. Analysis

- **Temporal** — pickup distributions by hour, day and month; weekday vs 
  weekend traffic; revenue trends and quarterly splits
- **Financial** — fare/distance/duration correlations, fare per mile by 
  time and vendor, distance-tiered vendor comparison, tip behaviour
- **Geospatial** — zone-level trip counts mapped via GeoPandas choropleths; 
  zone demand profiles broken down by time window

---

## Key Findings

**Demand is concentrated and highly time-dependent**
- JFK Airport is the single busiest pickup zone (~4,100 avg hourly pickups), 
  yet appears nowhere in the top dropoff zones — airport traffic is 
  strongly asymmetric.
- Evening rush (6–8 PM) drives peak volume; 3–5 AM is the quietest window.

**Fleet positioning must change through the day**
- Morning (6–10 AM): Upper East Side North leads (17,557 trips)
- Evening (4–7 PM): Midtown Center takes over as #1 (27,781 trips)
- Night (10 PM–3 AM): **East Village, West Village and Clinton East emerge 
  as top zones — none of which appear in any daytime list.** A driver 
  following daytime positioning logic would miss this demand entirely.

**Fare efficiency peaks in congestion, not in speed**
- Fare per mile peaks at **$9.42 at 6 PM** and bottoms out at **$6.50 at 
  5 AM**. The meter charges on both time *and* distance — slow traffic means 
  more time-based accumulation per mile covered.
- Wednesday/Thursday are the strongest days ($8.79/$8.75 per mile); Sunday 
  is the weakest ($7.59).

**Price is not a competitive lever**
- Creative Mobile Technologies and VeriFone track each other almost exactly 
  across all 24 hours (max divergence ~$0.15/mile). TLC rate regulation 
  standardises pricing — differentiation must come from service, not fares.

**Night operations are undervalued**
- Night contributes only 12.1% of revenue but earns **$29.47 per trip** vs 
  $28.78 by day, and carries the highest passenger count of any period 
  (1.46 avg at 3 AM) — meaning better vehicle utilisation per trip.

---

## Repository Structure

├── EDA_Optimising_NYC_Taxis_Sibtain_Raghib.ipynb   # Full analysis notebook

├── Report_NYC_Taxi_Operations.pdf                  # Findings & recommendations

├── README.md

└── .gitignore

---

## Tech Stack

| Purpose | Tools |
|---|---|
| Data manipulation | pandas, NumPy |
| Visualisation | Matplotlib, Seaborn |
| Geospatial | GeoPandas |
| File formats | PyArrow (Parquet), Shapefile |
| Environment | Python 3.13, Jupyter |

---

## Reproducing the Analysis

```bash
# Clone
git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git
cd YOUR_REPO

# Install dependencies
pip install pandas numpy matplotlib seaborn geopandas pyarrow jupyter

# Download the 2023 Yellow Taxi Parquet files and taxi_zones shapefile
# from the TLC website into a data/ folder, then run the notebook
jupyter notebook EDA_Optimising_NYC_Taxis_Sibtain_Raghib.ipynb
```

---

## Assumptions & Limitations

- Analysis is based on a 5% stratified sample; absolute trip counts are 
  scaled by 20× where real-world figures are quoted.
- Tip analysis is restricted to **credit card payments only** — the data 
  dictionary states cash tips are not recorded, so cash tipping behaviour 
  is unobservable in this dataset.
- Zone-level conclusions describe *what* the data shows (trip volumes per 
  zone), not *why* — inferring the character of a neighbourhood from taxi 
  volume alone would go beyond what the data supports.
- The `Airport_fee` column contains values of $1.75 that are not defined in 
  the data dictionary ($1.25 is the stated amount). This is flagged rather 
  than corrected, as the true cause is unknown.
