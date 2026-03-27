# Airbnb NYC — Pricing Inefficiency & Demand Analysis

SQL-based case study analyzing **48,895 Airbnb listings** across New York City's 5 boroughs to identify overpriced neighborhoods, high-efficiency listings, and revenue optimization opportunities using Databricks.

---

## Business Questions

| # | Question | Key Finding |
|---|----------|-------------|
| Q1 | Which neighborhoods are overpriced relative to demand? | Manhattan holds **68% of all overpriced listings** — averaging $320/night with only ~4 reviews |
| Q2 | Which listings generate highest engagement relative to availability? | East Elmhurst (Queens) leads listing efficiency at **1.62** — 10x higher than Midtown Manhattan |
| Q3 | Where should hosts adjust pricing? | Queens/Bronx are **underpriced by 10–15%**; Manhattan premium segment is **overpriced by 15–20%** |

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| **Databricks** | SQL notebook, Spark cluster compute, Delta Lake tables |
| **SQL** | CTEs, window functions (`RANK`, `PERCENTILE_APPROX`), CASE-based feature engineering |
| **PySpark** | CSV ingestion with escape handling for 7,031 comma-containing listing names |
| **Data Source** | [Inside Airbnb](http://insideairbnb.com/new-york-city) — NYC, 48,895 listings, 16 features |

---

## Project Structure

```
Airbnb-NYC-Pricing-Analysis/
├── notebooks/
│   ├── airbnb_analysis.ipynb        # Full Databricks SQL notebook (32 cells + markdown insights)
│   └── airbnb_analysis.html         # Static HTML export for viewing without Databricks
├── data/
│   └── raw/
│       └── Airbnb_NYC.csv           # Dataset (48,895 × 16)
├── screenshots/
│   ├── 01_data_quality_audit.png
│   ├── 02_price_by_borough.png
│   ├── 03_efficiency_ranking.png
│   ├── 04_revenue_top_neighborhoods.png
│   ├── 05_overpriced_listings.png
├── README.md
└── .gitignore
```

---

## Dataset Overview

Single CSV — `Airbnb_NYC.csv` — 48,895 listings across 221 neighborhoods in 5 boroughs.

| Column | Type | Description |
|--------|------|-------------|
| `id` | int | Unique listing identifier |
| `name` | string | Listing title (7,031 contain commas — required escape handling) |
| `host_id` / `host_name` | int / string | Host identifier and name |
| `neighbourhood_group` | string | Borough: Manhattan, Brooklyn, Queens, Bronx, Staten Island |
| `neighbourhood` | string | Specific neighborhood (221 unique values) |
| `latitude` / `longitude` | double | Geolocation coordinates |
| `room_type` | string | Entire home/apt, Private room, Shared room |
| `price` | int | Nightly price in USD |
| `minimum_nights` | int | Minimum booking length required |
| `number_of_reviews` | int | Total lifetime review count |
| `last_review` | date | Date of most recent review |
| `reviews_per_month` | double | Average monthly review frequency |
| `calculated_host_listings_count` | int | Number of listings this host operates |
| `availability_365` | int | Days available in the next 365 days |

---

## Data Preprocessing

| Issue | Count | Resolution |
|-------|-------|------------|
| Null listing names | 16 | Filled → `'Unnamed Listing'` |
| Null host names | 21 | Filled → `'Unknown Host'` |
| Null `reviews_per_month` | 10,052 | Filled → `0` (all correspond to 0-review listings — systematic, not random) |
| Price = $0 | 11 | Removed — invalid listings |
| Minimum nights > 365 | 14 | Removed — long-term leases, not short-term rentals |
| Listing names with commas | 7,031 | Handled via PySpark `escape` + `multiLine` CSV options |

**Rows removed:** 25 out of 48,895 (<0.05%). No meaningful data loss.

**Outlier handling:** No price outliers were removed. Since this is a descriptive SQL analysis (not ML), all valid prices were retained. Median-based metrics are used throughout to ensure outlier-resistant central tendency.

---

## Feature Engineering

5 SQL-derived columns built via `CREATE TABLE ... AS SELECT` — each maps to a specific business question:

| Feature | SQL Formula | Business Purpose |
|---------|-------------|------------------|
| `listing_efficiency` | `number_of_reviews / price` | Demand per dollar — higher = better price-demand alignment. Core metric for Q1. |
| `revenue_proxy` | `price × (365 - availability_365)` | Estimated gross revenue from booked nights. Core metric for Q2. |
| `price_category` | `CASE: Budget (<$70) / Mid-Range / Premium / Luxury (>$300)` | Cohort segmentation for pricing analysis |
| `engagement_level` | `CASE: No Reviews / Low / Medium / High / Very High` | Demand intensity classification |
| `demand_bucket` | `CASE: Fully Booked / High Demand / Moderate / Low / Very Low` | Booking pressure segmentation based on availability |

---

## Key Findings

### 1. Manhattan Is Systematically Overpriced

Manhattan's mean price ($197) sits **31% above its median** ($150) — the largest mean-median gap of any borough, driven by luxury listings inflating averages without proportional demand. Its listing efficiency (0.18) is the **lowest across all boroughs**. 1,545 Manhattan listings are classified as overpriced (above P75 price, below P25 reviews).

### 2. Queens Dominates Efficiency Rankings

**9 of the top 10** most efficient neighborhoods are in Queens. East Elmhurst leads at **1.62 efficiency** — generating 1.62 reviews per dollar of nightly price, compared to Midtown Manhattan at ~0.15. That's a **10x efficiency gap**. These listings average $80–$95/night with 30–88 reviews, hitting the optimal price-demand sweet spot.

### 3. "Fully Booked" Is Misleading

35.9% of listings show zero availability, but their average review count is only **7.93** — far below genuinely high-demand listings (28–47 reviews). Most zero-availability listings are **inactive or delisted**, not sold out. Any analysis equating zero availability with high demand is fundamentally flawed.

### 4. Revenue Concentration Is Extreme

Manhattan and Brooklyn capture **92% of total estimated revenue** (~$1.15B out of $1.26B). But Brooklyn achieves higher average booked nights (222 vs 196) at lower prices — making it the better risk-adjusted play. Queens represents the largest untapped supply opportunity: strong demand signals but limited inventory.

### 5. Corporate Hosts Underperform on Efficiency

Sonder (207 listings, $270 avg) generates only **0.024 efficiency**, while individual host Melissa (34 listings, $53 avg) achieves **0.315** — a 13x gap. Scale alone doesn't win; pricing strategy and listing quality determine per-dollar engagement.

---

## Recommendations

### For Hosts

| Segment | Action | Data Backing |
|---------|--------|-------------|
| Manhattan hosts with <10 reviews | Reduce price by 15–20% | 1,545 overpriced listings avg $320 with ~4 reviews |
| Queens/Bronx hosts with 50+ reviews | Raise prices 10–15% | Efficiency scores (0.5–1.6) prove demand absorbs the increase |
| Brooklyn hosts at ~$65/night with 100+ reviews | Raise prices 10–15% immediately | Proven sustained demand — money left on the table |
| Listings with availability > 300 days | Improve photos, descriptions, response rate | "Very Low Demand" bucket avgs $184/night — high price + low quality = no bookings |
| Multi-listing hosts with price std dev > $50 | Standardize pricing across portfolio | Inconsistent pricing signals mismanagement |

### For Airbnb Platform

| Action | Expected Impact |
|--------|----------------|
| Promote high-efficiency neighborhoods (East Elmhurst, Flushing, Jamaica) in search | Better guest satisfaction per dollar |
| Deploy Smart Pricing nudges on 1,545 overpriced Manhattan listings | Unlock idle capacity in oversupplied borough |
| Surface revenue potential data to Queens/Bronx hosts | Stimulate supply growth where demand exists but inventory is thin |
| Flag "Fully Booked" + 0 reviews listings as inactive | Clean search results; remove ghost listings |

---

## Analysis Sections

The notebook is structured into 5 analytical sections, each addressing a dimension of the pricing-demand relationship:

| Section | Focus | Key Output |
|---------|-------|------------|
| **Section 1: Pricing Distribution** | Avg/median price by borough, room type × borough, top expensive neighborhoods, price category distribution | Mean-median gap reveals Manhattan's pricing distortion |
| **Section 2: Demand Analysis** | Reviews by borough, engagement levels, high-demand neighborhoods, demand bucket distribution | Outer boroughs outperform Manhattan on per-listing engagement |
| **Section 3: Efficiency Metrics** | `listing_efficiency` rankings by neighborhood, borough, and room type × borough | Queens dominates — 9 of top 10 most efficient neighborhoods |
| **Section 4: Revenue Proxy** | Revenue by neighborhood and borough, top individual listings | Tribeca wins per-unit ($72K); Chelsea wins at scale ($44M total) |
| **Section 5: Optimization** | Overpriced/underpriced classification, neighborhood optimization matrix, multi-host analysis | Actionable pricing recommendations per neighborhood |

---

## How to Run

### Option 1: View on GitHub (No Setup Required)

Click on `notebooks/airbnb_analysis.ipynb` — GitHub renders Jupyter notebooks natively with all SQL cells, outputs, and markdown insights visible.

### Option 2: Run in Databricks

1. Sign up for [Databricks Community Edition](https://community.cloud.databricks.com/) (free) or use any Databricks workspace
2. Create a Volume and upload `Airbnb_NYC.csv`:
   - Catalog → your catalog → default schema → create Volume `raw_data` → upload CSV
3. Import the notebook: Workspace → Import → upload `airbnb_analysis.ipynb`
4. Attach a cluster (Single Node, latest LTS runtime)
5. Update the CSV path in Cell 2 to match your Volume path:
   ```python
   .load("/Volumes/<your_catalog>/default/raw_data/Airbnb NYC.csv")
   ```
6. Run all cells sequentially (Shift+Enter)

### Note on CSV Parsing

The dataset contains **7,031 listing names with commas** and **150 with quotes**. The notebook uses PySpark with `multiLine` and `escape` options to parse correctly. Loading via the Databricks UI table creator without these options will shift columns — always use the PySpark cell (Cell 2) for initial ingestion.

---

