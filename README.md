# IDS701_Causal_Report_Project

# Data Overview and Construction Summary

We construct a panel dataset that aligns stock return data across multiple AI-related firms and major U.S. trade war policy events. The objective is to evaluate whether hardware-dependent AI companies exhibit systematically stronger stock price reactions—measured via abnormal returns—to these policy shocks.

## Firms and Groups

- **Treatment group (T = 1):**
  - NVIDIA, AMD, Intel  
  Firms with heavy reliance on AI hardware (e.g., GPU/TPU).

- **Control group 1 (T = 0):**
  - Palantir, C3.ai, Snowflake Inc. 
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



# Empirical Strategy

## Baseline Cross-sectional DiD Model

We estimate the following regression model:

$$
\text{delta\_CAR}_{i,e} = \alpha + \beta \cdot \text{Treatment}_i + \epsilon_{i,e}
$$

Where:

- **$\text{delta\_CAR}_{i,e}$**: The change in cumulative abnormal return for firm *i* around event *e*, computed as:

  $$
  \text{delta\_CAR} = \text{CAR}_{\text{post}} - \text{CAR}_{\text{pre}}
  $$

- **$\text{Treatment}_i$**: An indicator variable equal to 1 if firm *i* is **hardware-dependent** (treatment group), and 0 if the firm belongs to the **control group**.

- **$\alpha$**: The intercept, representing the average delta CAR for control firms.

- **$\beta$**: The treatment effect — how much more (or less) the delta CAR is for hardware-dependent firms compared to control firms.

- **$\epsilon_{i,e}$**: The error term, capturing unexplained variation at the firm-event level.

### **Explanation of Results**

In the baseline regression, the **treatment coefficient ($\beta$)** represents the difference in abnormal returns between hardware-dependent firms (Treatment group) and software-based firms (Control group). 

- If $\beta$ is **positive** and **statistically significant**, this would suggest that hardware-dependent firms experienced a stronger positive reaction to policy events compared to the control group. 
- Conversely, if $\beta$ is **negative** or **not significant**, this suggests that the treatment group did not respond more strongly, or even responded weaker, than the control group in terms of abnormal returns.

The **intercept ($\alpha$)** represents the baseline abnormal return for the control group (software-focused firms with no hardware dependence), and is often expected to be close to zero if the control group’s response is neutral.

If your result shows that the treatment effect ($\beta$) is **significant**, it would provide evidence that hardware dependency influences firm reactions to policy changes. On the other hand, a **non-significant**