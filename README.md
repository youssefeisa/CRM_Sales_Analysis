# B2B Sales Pipeline — Exploratory Data Analysis Report
**Generated:** April 30, 2026  
**Data Period:** October 2016 – December 2017 (14 months)  
**Industry:** Computer Hardware (B2B)

---

## 1. Dataset Overview

| File | Rows | Columns | Role |
|---|---|---|---|
| `sales_pipeline.csv` | 8,800 | 13 | Main fact table |
| `sales_teams.csv` | 35 | 3 | Agent → Manager → Region |
| `products.csv` | 7 | 3 | Product catalog with list prices |
| `accounts.csv` | 85 | 7 | Customer account profiles |
| `data_dictionary.csv` | 21 | 3 | Field definitions |

---

## 2. Data Quality Assessment

### 2.1 Missing Values

| Table | Field | Missing | % | Explanation |
|---|---|---|---|---|
| sales_pipeline | `account` | 1,425 | 16.2% | All from Prospecting (337) and Engaging (1,088) stages — no account assigned yet |
| sales_pipeline | `engage_date` | 500 | 5.7% | All Prospecting rows — engagement not started |
| sales_pipeline | `close_date` | 2,089 | 23.7% | Prospecting (500) + Engaging (1,589) — deal not yet closed |
| sales_pipeline | `close_value` | 2,089 | 23.7% | Same rows as `close_date` |
| sales_pipeline | `deal_duration_days` | 2,089 | 23.7% | Calculated from engage→close, N/A for open deals |
| accounts | `subsidiary_of` | 70 | 82.4% | Most accounts are independent; 15 are subsidiaries |

**Verdict:** All missing values are structurally expected. No data corruption detected. The pipeline design intentionally leaves date/value fields blank for open opportunities.

### 2.2 Duplicate Check

- **Duplicate `opportunity_id`:** 0 — primary key is clean.
- **Unique agents in pipeline:** 30 of 35 from the team roster. Five agents (`Carol Thompson`, `Carl Lin`, `Mei-Mei Johns`, `Natalya Ivanova`, `Elizabeth Anderson`) appear in the teams file but have zero pipeline activity — likely new hires or inactive during the data window.

### 2.3 Data Type Notes

- `engage_date` and `close_date` stored as strings — require `pd.to_datetime()` for time-series analysis.
- `year`, `quarter`, `month` stored as float64 — safe for filtering but should be cast to int.
- `close_value` is 0.0 for all **Lost** deals (not null) — this is a design choice, not missing data. Lost deals have no recoverable revenue value.

---

## 3. Pipeline Structure & Deal Stages

| Stage | Count | % of Total | Notes |
|---|---|---|---|
| Won | 4,238 | 48.2% | Closed, revenue captured |
| Lost | 2,473 | 28.1% | Closed, zero value |
| Engaging | 1,589 | 18.1% | Active, in negotiation |
| Prospecting | 500 | 5.7% | Earliest stage, no account/dates yet |

**Pipeline funnel logic:** Prospecting → Engaging → Won or Lost  
**Active pipeline (Prospecting + Engaging):** 2,089 opportunities — represents future revenue potential.

---

## 4. Revenue & Deal Value Analysis

| Metric | Value |
|---|---|
| Total Won Revenue | $10,005,534 |
| Overall Win Rate | 63.2% (Won / Won+Lost) |
| Average Won Deal Value | $2,361 |
| Median Won Deal Value | $1,117 |
| Max Single Deal Value | $30,288 |

**Distribution skew:** The gap between mean ($2,361) and median ($1,117) confirms a right-skewed distribution. A small number of high-value GTK 500 and GTX Plus Pro deals pulls the mean up significantly.

**Close value vs. list price:** Products close almost exactly at their list price (ratio 0.99–1.00 across all 7 products). There is effectively **no discounting** in this pipeline. This is either policy (fixed-price selling) or the data represents catalog pricing rather than negotiated deals.

---

## 5. Product Analysis

| Product | Series | List Price | Opportunities | Win Rate | Total Revenue | Avg Won Value |
|---|---|---|---|---|---|---|
| GTX Basic | GTX | $550 | 1,866 | 63.7% | $499,263 | $546 |
| GTX Pro | GTX | $4,821 | 1,480 | 63.6% | $3,510,578 | $4,816 |
| MG Special | MG | $55 | 1,651 | 64.8% | $43,768 | $55 |
| MG Advanced | MG | $3,393 | 1,412 | 60.3% | $2,216,387 | $3,389 |
| GTX Plus Pro | GTX | $5,482 | 968 | 64.3% | $2,629,651 | $5,490 |
| GTX Plus Basic | GTX | $1,096 | 1,383 | 62.1% | $705,275 | $1,080 |
| GTK 500 | GTK | $26,768 | 40 | 60.0% | $400,612 | $26,707 |

**Key observations:**

- **GTX series dominates:** $7,344,767 (73.4% of total revenue) from 4 products.
- **MG Special anomaly:** 1,651 opportunities at only $55 each → $43,768 total revenue. This product generates high activity for minimal return. It may serve as a loss-leader or entry-point product.
- **GTK 500 anomaly:** Only 40 opportunities across 14 months despite having the highest list price ($26,768). Revenue per deal is excellent but pipeline volume is severely limited. Market fit, distribution, or sales motion may need investigation.
- **Win rates are tightly clustered:** All products fall between 60.0%–64.8%, suggesting win rate is more driven by sales team behavior than product characteristics.

---

## 6. Sales Team Performance

### 6.1 Regional Offices

| Region | Agents | Won Revenue | Revenue/Agent |
|---|---|---|---|
| West | 12 | $3,568,647 | $297,387 |
| Central | 11 | $3,346,293 | $304,208 |
| East | 12 | $3,090,594 | $257,550 |

West leads in total revenue; Central leads in revenue per agent efficiency.

### 6.2 Manager Win Rates

| Manager | Region | Win Rate | Agents |
|---|---|---|---|
| Cara Losch | East | 64.75% | 6 |
| Summer Sewald | West | 64.71% | 6 |
| Celia Rouche | West | 64.06% | 6 |
| Dustin Brinkmann | Central | 63.32% | 5 |
| Rocco Neubert | East | 62.45% | 6 |
| Melvin Marxen | Central | 61.79% | 6 |

Gap between top and bottom manager is only **2.96 percentage points** — relatively tight, suggesting consistent management quality across the organization.

### 6.3 Agent Performance (Top 10 by Revenue)

| Agent | Manager | Region | Opps | Win Rate | Revenue |
|---|---|---|---|---|---|
| Darcel Schlecht | Melvin Marxen | Central | 553 | 63.1% | $1,153,214 |
| Vicki Laflamme | Celia Rouche | West | 347 | 63.7% | $478,396 |
| Kary Hendrixson | Summer Sewald | West | 335 | 62.4% | $454,298 |
| Cassey Cress | Rocco Neubert | East | 261 | 62.5% | $450,489 |
| Donn Cantrell | Rocco Neubert | East | 275 | 57.5% | $445,860 |
| Reed Clapper | Rocco Neubert | East | 237 | 65.4% | $438,336 |
| Zane Levy | Summer Sewald | West | 261 | 61.7% | $430,068 |
| Corliss Cosme | Cara Losch | East | 229 | 65.5% | $421,036 |
| James Ascencio | Summer Sewald | West | 206 | 65.5% | $413,533 |
| Daniell Hammack | Rocco Neubert | East | 187 | 61.0% | $364,229 |

**Darcel Schlecht** is a clear outlier: 553 opportunities (the most of any agent), $1.15M revenue (2.4× the next-highest agent). His volume advantage, not a higher-than-average win rate, drives dominance.

### 6.4 Win Rate Distribution (All Agents)

| Tier | Win Rate Range | Agents | % |
|---|---|---|---|
| Excellent | ≥ 60% | 26 | 86.7% |
| Good | 55–59.9% | 4 | 13.3% |
| At Risk | < 55% | 0 | 0% |

**Bottom 5 agents by win rate:** Lajuana Vencill (55.0%), Markita Hansen (57.3%), Donn Cantrell (57.5%), Gladys Colclough (58.2%), Niesha Huffines (60.0%).

Note: Donn Cantrell ranks 5th in revenue ($445,860) despite a below-average win rate (57.5%) — high volume compensates for conversion inefficiency.

---

## 7. Account Analysis

### 7.1 Sector Distribution

| Sector | Accounts | Won Revenue |
|---|---|---|
| Retail | 17 | $1,867,528 |
| Technology | 12 | $1,515,487 |
| Medical | 12 | $1,359,595 |
| Software | 7 | $1,077,934 |
| Finance | 8 | $950,908 |
| Marketing | 8 | $922,321 |
| Entertainment | 6 | $689,007 |
| Telecommunications | 6 | $653,574 |
| Services | 5 | $533,006 |
| Employment | 4 | $436,174 |

Retail has the most accounts and highest revenue, but software (7 accounts) achieves disproportionately high revenue — highest revenue-per-account ratio among major sectors.

### 7.2 Top 10 Accounts by Revenue

| Rank | Account | Revenue |
|---|---|---|
| 1 | Kan-code | $341,455 |
| 2 | Konex | $269,245 |
| 3 | Condax | $206,410 |
| 4 | Cheers | $198,020 |
| 5 | Hottechi | $194,957 |
| 6 | Goodsilron | $182,522 |
| 7 | Treequote | $176,751 |
| 8 | Warephase | $170,046 |
| 9 | Xx-holding | $169,357 |
| 10 | Isdom | $164,683 |

### 7.3 Account Demographics

- **Revenue range:** $4.54M – $11,698M annual revenue
- **Employee range:** 9 – 34,288
- **Geography:** 71/85 accounts (83.5%) are US-based; 14 international accounts across 13 countries
- **Subsidiaries:** 15 of 85 accounts (17.6%) are subsidiaries of parent companies — relevant for account expansion strategy

---

## 8. Quarterly & Monthly Trends

### 8.1 Quarterly Summary

| Quarter | Revenue | Opportunities | Won | Win Rate | QoQ Revenue |
|---|---|---|---|---|---|
| Q4 2016 | $523,531 | 358 | 250 | 82.0% | — |
| Q1 2017 | $2,170,446 | 1,619 | 898 | 67.9% | +314.8% |
| Q2 2017 | $2,968,256 | 2,388 | 1,246 | 61.6% | +36.8% |
| Q3 2017 | $2,792,733 | 2,770 | 1,207 | 60.0% | -5.9% |
| Q4 2017 | $1,550,568 | 1,165 | 637 | 60.7% | -44.5% |

### 8.2 Monthly Won Revenue

| Month | Revenue |
|---|---|
| Mar 2017 | $1,134,672 |
| Apr 2017 | $721,932 |
| May 2017 | $1,025,713 |
| Jun 2017 | $1,338,466 |
| Jul 2017 | $696,932 |
| Aug 2017 | $1,050,059 |
| Sep 2017 | $1,235,264 |
| Oct 2017 | $731,980 |
| Nov 2017 | $938,943 |
| Dec 2017 | $1,131,573 |

Monthly revenue oscillates in a wave pattern (~$700K troughs, ~$1.3M peaks) with no clean seasonal signal — the quarterly drop in Q4 2017 is driven primarily by fewer opportunities entering the pipeline, not a conversion failure.

### 8.3 Win Rate Erosion

Win rate declined from **82.0% → 60.0%** between Q4 2016 and Q3 2017 then stabilized. The Q4 2016 rate is artificially elevated — the company was still selectively pursuing the highest-confidence deals early in its pipeline expansion. As volume scaled aggressively in Q1–Q2 2017, lower-quality leads diluted the win rate. Stabilization at ~60% from Q3 2017 onward suggests the pipeline has matured.

---

## 9. Deal Duration Analysis

| Stage | Mean (days) | Median (days) | Range |
|---|---|---|---|
| Won | 51.8 | 57.0 | 1–138 |
| Lost | 41.5 | 14.0 | 1–138 |

**Won deals take longer to close** (median 57d vs 14d for Lost). This counter-intuitive pattern means:
- Deals that close quickly are more likely to be Lost (perhaps the buyer decides quickly it's not a fit)
- Won deals require more nurturing, negotiation, and relationship-building time
- A deal still active after 30+ days is actually a positive signal

**Actionable implication:** Agents should not rush to close early-stage deals. Sustainable pipeline management favors 45–90 day engagement cycles for won outcomes.

---

## 10. Key Findings Summary

### ✅ Strengths
1. **Clean, well-structured data** with no duplicates, consistent IDs, and all missing values structurally explained.
2. **Strong overall win rate** (63.2%) with 86.7% of agents in the "Excellent" tier (≥60%).
3. **Fixed-price discipline:** Close values match list prices precisely — no discounting behavior in the data.
4. **Win rate stabilized** at ~60% in H2 2017 after rapid pipeline expansion.

### ⚠️ Risks & Anomalies
1. **Revenue concentration:** Darcel Schlecht = 11.5% of total revenue; top 10 accounts = ~33% of revenue. Key-person and key-account risk.
2. **GTK 500 underutilization:** Only 40 opportunities in 14 months for the highest-priced product. Massive revenue upside if sales motion can be improved.
3. **MG Special ROI:** 1,651 opportunities generating only $43,768 in revenue. High agent time cost for minimal return.
4. **Q4 2017 revenue cliff:** -44.5% QoQ. The pipeline volume drop (2,770 → 1,165 opps) is the primary driver — likely seasonal, but needs monitoring.
5. **5 inactive agents** in the teams roster — confirm onboarding/attrition status.

### 🔍 Open Questions for Stakeholders
1. Is the MG Special intended as a loss-leader? If not, it may warrant discontinuation or repositioning.
2. What is blocking GTK 500 pipeline volume? Is it a training, targeting, or product-market fit issue?
3. Are the Q4 2017 pipeline entry declines seasonal or structural? If structural, Q1 2018 forecasts should be adjusted downward.
4. What is the status of the 5 agents in the teams file with zero pipeline activity?

---

## 11. Recommended Next Steps

**Immediate analyses:**
- Agent cohort analysis: Compare win rates by manager to identify coaching opportunities
- Account penetration: Cross-reference active accounts against all 85 to find untapped relationships
- GTK 500 sales motion: Isolate which agents/regions have attempted GTK 500 and what drove the 15 wins

**Forecasting (in progress):**
- Use 5 quarterly data points with Prophet or Exponential Smoothing
- Forecast Q1–Q2 2018 revenue with honest confidence intervals
- Export forecast CSV for Power BI Page 5 integration

**Power BI:**
- All DAX measures validated against confirmed totals in this report
- DateTable relationship to `close_date` confirmed as correct join key
- Q4 2017 revenue cliff should be annotated on trend visuals for stakeholder context

