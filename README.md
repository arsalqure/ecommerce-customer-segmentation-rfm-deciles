<div align="center">

```
██████╗ ███████╗███╗   ███╗
██╔══██╗██╔════╝████╗ ████║
██████╔╝█████╗  ██╔████╔██║
██╔══██╗██╔══╝  ██║╚██╔╝██║
██║  ██║██║     ██║ ╚═╝ ██║
╚═╝  ╚═╝╚═╝     ╚═╝     ╚═╝
  CUSTOMER ANALYTICS ENGINE
```

# E-commerce Customer Segmentation & Revenue Concentration

**RFM Modelling · Customer Deciles · Retention Playbooks · Budget Allocation**

![Python](https://img.shields.io/badge/Python-3.12-3776AB?style=flat-square&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-2.2-150458?style=flat-square&logo=pandas&logoColor=white)
![Seaborn](https://img.shields.io/badge/Seaborn-0.13-4C72B0?style=flat-square)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?style=flat-square&logo=jupyter&logoColor=white)
![Status](https://img.shields.io/badge/Status-Complete-2ea44f?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)

</div>

---

## What This Project Does

> *"Not all customers are equal — this project proves it with numbers, and turns that insight into action."*

This project applies **RFM analysis** (Recency · Frequency · Monetary) to a real-world e-commerce transaction dataset to answer one fundamental business question:

**Which customers drive the business — and what should we do about it?**

The pipeline goes from raw CSV to decision-ready marketing segments, validated by two independent concentration checks (Pareto + deciles), and closed with a blended budget allocation model.

---

## Results at a Glance

<table>
<tr>
<td width="50%">

### 📦 Dataset
| Metric | Value |
|--------|-------|
| Raw rows | 541,909 |
| Columns | 8 |
| Date range | Dec 2010 – Dec 2011 |
| Customers analysed | **4,334** |
| Cleaned revenue | **£8,767,752** |

</td>
<td width="50%">

### 🎯 Revenue Concentration
| Slice | Revenue Share |
|-------|--------------|
| Top 10% of customers | **61.4%** |
| Top 20% of customers | **74.5%** |
| Champions segment (7.4% of base) | **49.7%** |
| Decile 1 alone | **~45%+** |

</td>
</tr>
</table>

> **Core takeaway:** Revenue is hyper-concentrated. Protecting the top tier is worth more than acquiring new customers at scale.

---

## Analytical Pipeline

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         ANALYTICAL PIPELINE                                 │
├──────────┬───────────────┬──────────────┬──────────────┬────────────────────┤
│  INGEST  │    CLEAN      │  ENGINEER    │   ANALYSE    │      OUTPUT        │
├──────────┼───────────────┼──────────────┼──────────────┼────────────────────┤
│          │ • Drop dupes  │ • RFM        │ • Segments   │ • Retention plan   │
│ CSV Load │ • Remove NaN  │   metrics    │ • Pareto     │ • Win-back targets │
│ + Audit  │ • Cancel      │ • Scoring    │ • Deciles    │ • Budget model     │
│          │   invoices    │   (1–5)      │ • Top 20     │ • 9 visualisations │
│          │ • Qty/Price   │ • Tenure vs  │   customers  │                    │
│          │   filters     │   activity   │              │                    │
│          │ • Non-product │   rates      │              │                    │
│          │   codes       │              │              │                    │
└──────────┴───────────────┴──────────────┴──────────────┴────────────────────┘
```

---

## Segmentation Design

### Customer Segments & Business Logic

| Segment | Rules | Business Priority |
|---------|-------|:-----------------:|
| 🏆 **Champions** | Recency ≤ 30d · Freq ≥ P80 · Spend ≥ P90 | ⬆⬆ Protect |
| 💎 **High-Value New** | Recency ≤ 30d · Spend ≥ P90 | ⬆⬆ Nurture fast |
| 💚 **Loyal Customers** | Recency ≤ 60d · Freq ≥ P80 | ⬆ Maintain |
| 🆕 **Recent Customers** | Recency ≤ 30d | ↗ Convert to loyal |
| ⚠️ **At Risk** | Recency ≥ 120d · High freq or spend | ⬆ Win back urgently |
| 💤 **Hibernating** | Recency ≥ 180d · Freq ≤ 2 · Spend ≤ P25 | → Low priority |
| 🔔 **Needs Attention** | All other customers | ↗ Reactivate selectively |

### Threshold Derivation

Thresholds are **data-driven**, not hardcoded guesses:

```python
f_cut = rfm["Frequency"].quantile(0.80, interpolation="higher")   # Top-20% order frequency
m_p90 = rfm["Monetary"].quantile(0.90)    # Top-10% spend — VIP tier
m_p75 = rfm["Monetary"].quantile(0.75)    # Upper-quarter spend
m_p25 = rfm["Monetary"].quantile(0.25)    # Lower-quarter spend
```

This makes segments **reproducible and defensible** in stakeholder reviews — thresholds shift with the data, not with opinions.

---

## Feature Engineering

### Standard RFM

```
Recency   = SNAPSHOT_DATE − max(InvoiceDate) per customer  [days]
Frequency = count(distinct InvoiceNo) per customer          [orders]
Monetary  = sum(Quantity × UnitPrice) per customer          [£]
```

> **Snapshot date** is hardcoded (`2011-12-10`) — one day after the final transaction. This keeps Recency values **reproducible** across re-runs, regardless of when the notebook executes.

### Habit vs Burst Purchasing (Extended Metrics)

Standard RFM treats a customer with 10 orders in 1 month the same as one with 10 orders across 12 months. These two metrics separate them:

| Metric | Formula | Captures |
|--------|---------|---------|
| `OrdersPerMonth` | Frequency ÷ ActiveMonths | Intensity during active periods |
| `OrdersPerTenureMonth` | Frequency ÷ TenureMonths | Habit across full customer lifespan |

```
ActiveMonths   = months in which the customer placed ≥1 order
TenureMonths   = (last purchase month) − (first purchase month) + 1
```

**Why it matters:** Habit buyers belong in loyalty programmes. Burst buyers respond better to replenishment nudges and cross-sell.

---

## Revenue Concentration (Two Independent Checks)

### Check 1 — Customer-Level Pareto

```
Top 10% customers  →  61.4% of revenue
Top 20% customers  →  74.5% of revenue   ✓ Pareto confirmed (≥75% threshold)
```

### Check 2 — Decile Analysis

```
Decile  │  Revenue %  │  Cum. %  │  Interpretation
────────┼─────────────┼──────────┼──────────────────────────────
  D1    │    45%+     │   45%+   │  Top 10% spenders
  D2    │    ~15%     │   ~60%   │  Upper-mid tier
  D3    │    ~9%      │   ~69%   │  ← Growth ROI zone begins
  D4    │    ~6%      │   ~75%   │
  D5    │    ~5%      │   ~80%   │  ← 80% revenue threshold
 D6–D10 │    ~20%     │  100%    │  Bottom 50% — minimal spend
```

Both approaches independently confirm the same story: **a small minority of customers accounts for the majority of revenue.**

---

## Marketing Budget Allocation Model

Rather than allocating budget purely by current revenue (which over-protects D1 and ignores growth potential), the model blends two signals:

```
Budget_% = (55% × Revenue_%) + (45% × GrowthUplift_%)

where:
  GrowthUplift_D = 100% − CumRevenue%_(D-1)
  (= the revenue share not yet captured by deciles above D)
```

| Weight | Component | Effect |
|--------|-----------|--------|
| 55% | Revenue share | Protects Champions and Loyal customers |
| 45% | Growth-uplift potential | Shifts premium toward D3–D5 (highest upsell ROI) |

**Result:**
- **D1** receives slightly *less* budget than its revenue share → retention is cheaper than acquisition
- **D3–D5** receive a growth premium → highest expected ROI for conversion campaigns
- **D8–D10** receive minimal spend → low value and low growth potential

---

## Business Action Playbook

<details>
<summary><strong>🏆 Champions — Protect & reward (click to expand)</strong></summary>

**Who:** Recently active, very frequent, very high spend. 7.4% of customers, 49.7% of revenue.

**Actions:**
- Priority customer service lane
- Early access to new products / collections
- Personalised thank-you communication (not generic newsletters)
- Proactive outreach if Recency crosses 45 days (before they become At Risk)
- Dedicated account manager for top 50 by Monetary value
</details>

<details>
<summary><strong>💎 High-Value New — Fast-track to loyalty</strong></summary>

**Who:** Recent buyers with VIP-level spend but not yet frequent. The next Champions.

**Actions:**
- Onboarding sequence focused on product depth (show catalogue breadth)
- Second-purchase incentive within 30 days
- Monitor Recency weekly — intervene at 21 days
</details>

<details>
<summary><strong>⚠️ At Risk — Time-sensitive win-back</strong></summary>

**Who:** Previously high-value customers who have gone quiet (≥120 days lapsed).

**Actions:**
- Triggered win-back email at day 120, 150, and 180
- Selective incentive (not blanket discount) — use Monetary tier to calibrate offer value
- Survey to understand reason for lapse before discounting
- Suppress from general promotional sends to avoid training them to wait for discounts
</details>

<details>
<summary><strong>💤 Hibernating — Low investment, test-and-learn</strong></summary>

**Who:** Long lapsed, low frequency, low spend. Minimal revenue risk.

**Actions:**
- Single lightweight reactivation attempt (low-cost channel: push or email)
- No discounts — margin does not justify it at this spend level
- If no response, suppress from all campaigns and archive
</details>

---

## Visual Outputs

The notebook produces **9 charts** across four analytical dimensions:

```
Trend Analysis          │  Segment Analysis         │  Concentration         │  Allocation
────────────────────────┼───────────────────────────┼────────────────────────┼──────────────────
Monthly Revenue Trend   │  Customer Count/Segment   │  Decile Revenue Bar    │  Revenue vs
Hourly Purchase Pattern │  Revenue/Segment (bar)    │  Cumulative Pareto     │  Budget % (bar)
New vs Returning (line) │  R, F, M Distributions    │                        │
                        │  Freq × Monetary Scatter  │                        │
```

---

## Data Cleaning Audit Trail

Every removal step is logged with a count so the pipeline is fully auditable:

```
Step              │ What removed                            │ Why
──────────────────┼─────────────────────────────────────────┼─────────────────────────────
Exact duplicates  │ ~5,268 rows                             │ Inflate all RFM metrics
Missing CustomerID│ ~135K rows (~25%)                       │ Cannot assign customer
Cancelled invoices│ InvoiceNo starts with "C"               │ Not real completed sales
Invalid Qty/Price │ Quantity ≤ 0 or UnitPrice ≤ 0          │ Returns, errors, free items
Bad dates         │ NaT after pd.to_datetime coerce         │ Recency undefined
Non-product codes │ POST, D, M, PADS, DOT, CRUK, AMAZON FEE│ Services, not products
```

---

## Technical Highlights

### Performance — Vectorised Operations Throughout

| Pattern | Replaced | Speedup |
|---------|----------|---------|
| `np.where(condition, a, b)` | `apply(lambda)` for 2-way conditions | ~100× |
| `np.select(conditions, choices)` | `apply(segment_fn, axis=1)` for segmentation | ~100× |
| `groupby.transform("min")` | `groupby.apply()` for first-purchase date | ~50× |
| `(date2 - date1).dt.days` | `apply(lambda x: x.days)` for Recency | ~30× |

### Robustness — Defensive Coding

```python
# Tie-safe scoring (avoids pd.qcut crashing on duplicate bin edges)
ranked = series.rank(method="dense", ascending=higher_is_better)
binned = pd.qcut(ranked, q=k, labels=False, duplicates="drop")

# Data integrity assertions
assert rfm["Segment"].isna().sum() == 0          # All customers got a segment
assert (monthly_sum == true_unique).all()         # No double-counting in cohort chart

# Reproducible snapshot date — not datetime.today()
SNAPSHOT_DATE = pd.Timestamp("2011-12-10")        # Stable across all re-runs
```

---

## Project Structure

```
ecommerce-customer-segmentation-rfm-deciles/
│
├── 📓 Datacsvfinal_fixed.ipynb      ← Main analysis notebook
├── 📄 data.csv                      ← Source: UCI Online Retail Dataset
├── 📄 README.md                     ← This file
│
└── outputs/                         ← Generated charts (auto-created by notebook)
    ├── monthly_revenue_trend.png
    ├── hourly_patterns.png
    ├── new_vs_returning.png
    ├── segment_counts.png
    ├── segment_revenue.png
    ├── rfm_distributions.png
    ├── freq_monetary_map.png
    ├── decile_revenue.png
    └── cumulative_pareto.png
```

---

## Quick Start

```bash
# 1. Clone the repository
git clone https://github.com/arsalanahmed/ecommerce-customer-segmentation-rfm-deciles.git
cd ecommerce-customer-segmentation-rfm-deciles

# 2. Install dependencies
pip install pandas>=2.2.0 numpy>=1.26.0 matplotlib>=3.8.0 seaborn>=0.13.2

# 3. Place data file
# Ensure data.csv is in the project root directory
# Dataset: https://archive.ics.uci.edu/dataset/352/online+retail

# 4. Run the notebook
jupyter notebook Datacsvfinal_fixed.ipynb
```

> The notebook will auto-locate `data.csv` relative to its own directory.
> If it cannot find the file, it raises a clear `FileNotFoundError` with instructions.

---

## Assumptions & Scope

| Assumption | Detail |
|-----------|--------|
| Sales only | All cancelled invoices (`InvoiceNo` starts with `"C"`) excluded |
| Valid transactions | Quantity > 0 and UnitPrice > 0 required |
| UK-centric data | Dataset is primarily UK B2C / B2B retail |
| CustomerID grain | One RFM record per unique customer, not per invoice |
| Heavy-tailed spend | Log scaling used in monetary histogram and scatter plots |
| Snapshot fixed | `2011-12-10` = max(date) + 1 day, hardcoded for reproducibility |

---

## Roadmap

- [ ] Cohort retention curves — repeat-purchase rate by first-order month
- [ ] Margin-aware segmentation — if cost data becomes available
- [ ] Segment-level A/B test design — holdout groups for retention vs win-back
- [ ] Pipeline conversion — refactor notebook into `src/` scripts + CLI
- [ ] CLV modelling — BG/NBD or Pareto/NBD probabilistic lifetime value

---

## Tech Stack

<table>
<tr>
<td><img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white"/></td>
<td><img src="https://img.shields.io/badge/Pandas-150458?style=for-the-badge&logo=pandas&logoColor=white"/></td>
<td><img src="https://img.shields.io/badge/NumPy-013243?style=for-the-badge&logo=numpy&logoColor=white"/></td>
<td><img src="https://img.shields.io/badge/Matplotlib-11557c?style=for-the-badge"/></td>
<td><img src="https://img.shields.io/badge/Seaborn-4C72B0?style=for-the-badge"/></td>
<td><img src="https://img.shields.io/badge/Jupyter-F37626?style=for-the-badge&logo=jupyter&logoColor=white"/></td>
</tr>
</table>

---

## Data Source

**UCI Online Retail Dataset** — real transactions from a UK-based online retailer (Dec 2010 – Dec 2011).

- Source: [UCI Machine Learning Repository](https://archive.ics.uci.edu/dataset/352/online+retail)
- Citation: Daqing Chen, Sai Liang Sain, and Kun Guo, *Data mining for the online retail industry*, Journal of Database Marketing and Customer Strategy Management (2012)

---

<div align="center">

---

**Built by [Arsalan Ahmed](https://github.com/arsalanahmed)**

*If this project helped you, consider starring the repository ⭐*

</div>
