# IDS701_Causal_Report_Project

# Data Overview and Construction Summary

We construct a panel dataset that aligns stock return data across multiple AI-related firms and major U.S. trade war policy events. The objective is to evaluate whether hardware-dependent AI companies exhibit systematically stronger stock price reactions—measured via abnormal returns—to these policy shocks.

## Firms and Groups

- **Treatment group (T = 1):**
  - NVIDIA, AMD, Intel  
  Firms with heavy reliance on AI hardware (e.g., GPU/TPU).

- **Control group 1 (T = 0):**
  - Palantir, C3.ai, SoundHound.ai  
  AI-native software/SaaS firms.

- **Control group 2 (for robustness):**
  - Salesforce, Oracle, Adobe  
  Large-cap software/cloud service companies.

## Market Benchmark

- **S&P 500 Index (^GSPC)** is used as the proxy for market return, allowing for potential CAPM-based modeling.

---

## Step-by-Step Construction Process

### 1. Daily Return Calculation
- Downloaded historical daily closing prices for all firms.
- Computed daily returns using percentage change.

### 2. Expected Return Estimation (Historical Average)
- For each stock on each date, estimated expected return using average return from a pre-event estimation window: **[–90, –30] days**.
- Avoids market contamination and provides firm-specific baselines.

### 3. Abnormal Return (AR)
- Computed as:  
  `AR_it = Actual Return - Expected Return`

### 4. Event Alignment
- Defined 6 key U.S. policy events from 2022–2024 (e.g., export controls, investment bans).
- Tagged each firm-date with:
  - `event_id`
  - `event_date`
  - `event_time` (days relative to each event)
- Only dates within ±10-day window of each event are considered for alignment.

### 5. Cumulative Abnormal Return (CAR) Computation
For each firm-event pair:
- **CAR_pre**: Sum of ARs over **[–10, –1]**
- **CAR_post**: Sum of ARs over **[0, +10]**
- **delta_CAR**: `CAR_post - CAR_pre`
- **post**: Indicator (1 if `Date >= event_date`, else 0)

---

## Final Dataset Structure (`full_df`)

| Column Name       | Description |
|-------------------|-------------|
| `Date`            | Daily observation date |
| `ticker`          | Stock ticker |
| `Close`           | Closing price |
| `Return`          | Daily return |
| `Treatment`       | 1 if hardware-dependent firm, 0 otherwise |
| `expected_return` | Expected return based on historical average |
| `abnormal_return` | Actual – Expected return |
| `event_id`        | Associated policy event identifier |
| `event_date`      | Date of the policy event |
| `event_time`      | Days relative to event (0 = event date) |
| `CAR_pre`         | Cumulative abnormal return before event |
| `CAR_post`        | Cumulative abnormal return after event |
| `delta_CAR`       | Post – Pre CAR difference |
| `post`            | 1 if date is after event date, 0 otherwise |

---

## Next Steps

- Use `delta_CAR` as the dependent variable in a **cross-sectional DiD regression**.
- Use the full event-time data to estimate **dynamic treatment effects** in an **event-time DiD model**, and visualize treatment coefficients over time.
