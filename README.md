# Auto Dealer Causal Analysis

Root-cause and causal analysis of an auto-dealer sales pipeline (bookings → allocations → invoices), using a synthetic panel of 300 dealers over 36 months (10,800 dealer-months). The project covers data quality auditing, invoice lag estimation, seasonal decomposition, and a causal framework for estimating the cost of allocation delay.

> **Status: work in progress.** Phase 1 data foundations and cohort-based lag estimation are complete. Seasonality decomposition, regression-based lag estimation and causal estimation are underway.

**Tools:** Python, pandas, NumPy, statsmodels, matplotlib, Jupyter

---

## Business problem

A dealer asks: *"My invoices fell. Why, and what would fix it?"*

Answering this reliably requires three checks before attributing a drop to any business cause:

1. **Data reliability:** does the period reflect trading, or an ingestion anomaly?
2. **Timing:** how long after a cause (allocation) does its effect (invoice) appear?
3. **Seasonality:** how much of a drop is expected seasonal movement versus a structural change?

## Process model

Two views are used in this project.

**1. Booking flow (accounting identity).** Every booking ends each month in exactly one state:

```
bookings = cancellations + allocations + open
```

Allocations are later invoiced with a delay.

**2. Declared causal map (`dag.json`).** The business process allows these directions of influence:

```
policy_active ──► held ◄──► bookings
held, bookings ──► cancellations, allocations
avg_allocation_lag_days ──► cancellations
cancellations ──► allocations ──► invoices
```

An edge means the process allows a cause in that direction; it does not mean the effect is real, linear or immediate. The bookings ↔ held pair forms a loop, which a standard causal graph does not allow. This is flagged for validation rather than removed.

## Evidence levels

Each finding is labelled with the strength of evidence behind it:

| Level | Meaning | Allowed wording |
|---|---|---|
| Descriptive | A total split into parts it already contained | "accounts for" |
| Associative | Variables that move together | "is associated with" |
| Causal (model-based) | An effect estimated under stated assumptions | "would change by", plus assumptions |

---

## Key findings

| # | Finding | Evidence level | Result | 
|---|---|---|---|---|
| 1 | June 2025 volume is consistent with a data ingestion anomaly, not trading | Descriptive | 2,409 bookings vs. a monthly mean of 17,093 excluding June (−85.9%). Ratio to median 0.141; every other month ≥ 0.85 |
| 2 | Oct–Dec 2025 invoice conversion is not a business decline: these cohorts are immature (right-censored) | Descriptive | Cohort ages 2, 1, 0 months vs. a data-derived maturity age of 3 months. Observed invoice rates 84.4%, 54.9%, 0.0%. 20,037 invoices not yet issued |
| 3 | Allocation → invoice lag distribution (cohort method) | Descriptive | Lag 1: 54.9%, Lag 2: 29.5%, Lag 3: 15.6%. No invoicing in the allocation month; none after lag 3 | Complete |
| 4 | Allocation → invoice lag distribution (distributed-lag regression, dealer and month effects) | Associative | – |
| 5 | Seasonal vs. structural decline in invoices, late 2023 – early 2024 | – | STL (in-progress) |

### 1. Data quality: identifying an unreliable period

Bookings, allocations and invoices all collapse in June 2025 and recover immediately in July. A drop of this size, in every stage at once, is not consistent with trading and is treated as an ingestion anomaly. The month is excluded from downstream analysis.

![June 2025 anomaly](reports/figures/monthly_volumes.pdf)

### 2. Cohort maturity: why recent conversion looks poor

Cohort invoice rate rises with cohort age and plateaus at 100% by month 3, so the maturity age is derived from the oldest cohorts rather than assumed. The three most recent cohorts are younger than that, so their rates are incomplete. Reading them as final would suggest a conversion collapse that has not happened.

![Cohort invoice rate by age](reports/figures/cohort_invoice_rate_by_age.pdf)

**Answer to "what changed in Oct–Dec 2025?":** nothing has shown up as changed yet. The apparent decline is right-censoring.

### 3. Lag estimation: cohort method

The lag distribution is the first difference of the cumulative cohort invoice rate (share at lag *k* = rate at age *k* − rate at age *k−1*).

![Invoice lag distribution](reports/figures/invoice_lag_profile.pdf)

### 4. Lag estimation: regression method (in progress)

Monthly invoices are regressed on current and lagged allocations (t, t−1, t−2, t−3), with dealer and month fixed effects. The coefficients are compared with the cohort-based shares to check agreement and explain any differences.

---

## Roadmap

**Phase 1: Data Foundations & Baseline Analysis**
- [x] Data quality audit: unreliable periods and immature cohorts
- [x] Invoice lag estimation from cohort data
- [ ] Invoice lag estimation via distributed-lag regression, reconciled with the cohort result
- [ ] Seasonality and trend decomposition (STL), with attribution of the structural dip to process steps
- [ ] Marginal effect of allocation delay on invoices

**Phase 2: Advanced Analysis**
- [ ] Sensitivity analysis: robustness to unmeasured confounders
- [ ] Policy impact evaluation: finance scheme effect on hold rate and invoices
- [ ] Heterogeneous effects: which dealer segments benefit most
- [ ] Process map validation: testing each link against the data
- [ ] Panel vs. aggregate modelling

**Phase 3: Experimentation & Future Work**
- [ ] Rollout experiment design with power analysis

---

## Author

Saikruba Krishnan
