# E-commerce Customer Segmentation & Revenue Concentration (RFM + Deciles)

Business-focused customer analytics project using transaction data to **segment customers**, quantify **revenue concentration**, and translate findings into **retention / win-back actions**.

---

## Executive Summary

Using cleaned transaction-level retail data, I built an **RFM (Recency–Frequency–Monetary)** model, created **business-meaningful customer segments**, and validated revenue concentration using **customer-level Pareto** and **decile analysis**.

### Key Results (from this dataset)
- **Dataset size (raw):** 541,909 rows × 8 columns  
- **Customers analyzed (post-cleaning):** 4,334  
- **Total cleaned revenue:** **£8,767,752.65**
- **Revenue concentration (customer-level Pareto):**
  - **Top 10% customers → 61.4% of revenue**
  - **Top 20% customers → 74.5% of revenue**
- **Segment concentration:**
  - **Champions = 7.4% of customers → 49.7% of revenue**

**Takeaway:** Revenue is **highly concentrated**, so retention and churn prevention for the highest-value customers is the highest ROI lever.

---

## Business Questions Answered

1) **Who are the highest-value customers and how concentrated is revenue?**  
2) **Which customer groups should we retain, convert, or win back?**  
3) **How do we separate “habit buyers” vs “burst buyers”?** (Tenure vs Active Months)  
4) **How much of revenue comes from each decile of customer spend?**

---

## Segmentation Approach

### 1) Data Cleaning (audit-friendly rules)
- Dropped rows with **missing CustomerID** (cannot assign customer behavior)
- Removed **canceled invoices** (`InvoiceNo` starts with `"C"`)
- Removed invalid sales lines: **Quantity ≤ 0** or **UnitPrice ≤ 0**
- Created a single **Revenue** field = `Quantity × UnitPrice`
- Removed non-product/service lines (postage/fees/manual adjustments)

### 2) RFM Construction
- **Snapshot date** = `max(InvoiceDate) + 1 day`
- **Recency** = days since last purchase  
- **Frequency** = unique invoices  
- **Monetary** = total revenue

### 3) Segmentation (business-unit thresholds)
Segments are created using:
- Recency thresholds (e.g., 30 / 60 / 120 / 180 days)
- Frequency **P80** cutoff for “high frequency”
- Monetary percentiles (**P25 / P75 / P90**) for value tiers

Segments:
- **Champions**
- **Loyal Customers**
- **Recent Customers**
- **At Risk**
- **Hibernating**
- **Needs Attention**

### 4) Revenue Concentration Validation (2 independent views)
- **Customer-level Pareto**: Top 10% / Top 20% contribution  
- **Decile analysis**: D1 (top 10% spenders) → D10 (bottom 10%)

---

## Key Insights & Business Actions

### Insight 1 — Extreme revenue concentration
- Top 10% customers contribute **61.4%** of revenue  
- Top 20% customers contribute **74.5%** of revenue  

**Action:** Prioritize retention programs and churn prevention for high-value customers before spending on broad discounts.

### Insight 2 — “Champions” drive outsized impact
- Champions are ~**7.4%** of customers but generate **49.7%** of revenue

**Action:** VIP retention playbook (priority support, exclusives, proactive reactivation before churn).

### Insight 3 — Win-back is targeted, not mass
“At Risk” customers are a small share but have meaningful revenue contribution.

**Action:** Triggered win-back campaigns (recency-based) with selective incentives, not blanket discounting.

### Insight 4 — Habit vs burst purchasing (rate metrics)
Added:
- **ActiveMonths** (months with ≥1 purchase)
- **TenureMonths** (months between first and last purchase)
- **OrdersPerTenureMonth** to distinguish repeat behavior vs one-time spikes

**Action:** Route “habit buyers” to loyalty programs; route “burst buyers” to replenishment nudges and cross-sell.

---

## Visual Outputs
The notebook includes:
- Monthly revenue trend
- Hourly purchasing patterns
- R/F/M distributions (log scale for heavy tail)
- Frequency vs Monetary value map by segment
- Decile revenue distribution + cumulative Pareto curve
- Suggested budget allocation by decile (based on revenue contribution)

---

## Skills Demonstrated (what this project proves)
- **Data cleaning & audit trail:** null handling, cancellation logic, invalid transaction filtering
- **Feature engineering:** RFM, tenure, activity rate metrics
- **Segmentation design:** business thresholds + percentiles (defensible in stakeholder reviews)
- **Revenue concentration analysis:** Pareto + deciles with sanity checks
- **Exploratory analytics & visualization:** trends, distributions, heavy-tail handling, segmentation plots
- **Communication:** decisions → outputs → business actions (retention / win-back)

---

## Tech Stack
- **Python:** pandas, numpy
- **Visualization:** matplotlib, seaborn
- **Environment:** Jupyter Notebook / VS Code

---

## Notes / Assumptions
The analysis focuses on **completed sales**:
- Canceled invoices (`InvoiceNo` starting with `"C"`) are excluded
- Invalid transactions (Quantity ≤ 0 or UnitPrice ≤ 0) are excluded

RFM is computed at the **CustomerID grain**:
- **Recency:** days since last purchase (relative to snapshot date)
- **Frequency:** unique invoices (transaction count)
- **Monetary:** total revenue (£)

Spend behavior is **heavy-tailed** (few customers contribute outsized revenue), so some plots use **log scaling** for interpretability.

---

## Next Improvements (optional, future work)
- Cohort retention analysis by first purchase month (repeat behavior tracking)
- Profit-aware segmentation if margin/cost data becomes available
- Segment-level A/B testing plan for retention and win-back campaigns
- Convert notebook into a reproducible pipeline (script + outputs + report)

---

## Author
**Arsalan Ahmed**  
Repository: `ecommerce-customer-segmentation-rfm-deciles`




