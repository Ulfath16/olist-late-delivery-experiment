# Proactive Late-Delivery Notification: An A/B Test Case Study

An end-to-end experimentation case study built on the Olist Brazilian E-Commerce dataset,
covering data cleaning, power analysis, and a simulated randomized experiment with a
heterogeneous treatment effect.

## The problem

Analysis of ~96,000 delivered orders shows a massive gap in customer satisfaction between
on-time and late deliveries:

| | Negative review rate (score <= 2) |
|---|---|
| On-time orders | 9.2% |
| Late orders | 54.1% |

Today, customers only discover a delivery delay when they check tracking themselves or the
package simply doesn't show up. This project designs, powers, and simulates an experiment
testing whether **proactively notifying customers about an in-progress delay** reduces the
damage to satisfaction, even though the delivery is still late.

## What's in this repo

| File | What it covers |
|---|---|
| `docs/experiment_design_doc.md` | Full experiment proposal: hypothesis, metrics, randomization, MDE, decision criteria |
| `docs/decisions_log.md` | Every judgment call made in this project, with the reasoning behind it |
| `notebooks/01_data_exploration.ipynb` | Loads and validates all 9 Olist tables, builds a clean order-level dataset, computes the baseline finding above |
| `notebooks/02_power_analysis.ipynb` | Derives minimum detectable effect and required sample size from the real baseline rate |
| `notebooks/03_simulate_experiment.ipynb` | Simulates the experiment with a realistic, heterogeneous treatment effect; validates a two-proportion z-test against known ground truth; segment analysis |
| `notebooks/04_cuped_variance_reduction.ipynb` | Applies CUPED variance reduction using delay severity as a pre-treatment covariate; compares confidence interval width with and without the adjustment |

## Key methodology decisions

- **Randomized by customer, not order** (`customer_unique_id`), to avoid contaminating results
  when the same customer has multiple late orders during a test window.
- **Modeled a heterogeneous treatment effect** rather than a flat one: the notification is
  hypothesized (and simulated) to help most for mild delays and to matter far less for severe
  ones, where no message can undo a month-long wait.
- **Validated the statistical pipeline against known ground truth** before trusting it: since
  real experiments never let you check your estimate against the true effect, this project
  simulates data with a known planted effect and confirms the standard two-proportion z-test
  recovers it within its confidence interval.

## Results summary

- Chosen target: an 8 percentage point reduction in negative review rate among late orders,
  requiring ~1,224 orders total and an estimated ~3.7 months of runtime at current order volume.
- Simulated experiment result: **-8.56pt effect, 95% CI [-10.77, -6.35], p < 0.000001** -- meets
  the pre-registered decision criterion to ship.
- Segment analysis shows the average effect hides real heterogeneity: ~15pt improvement for
  delays under 2 days, shrinking to ~4-5pts for delays over 10 days.
- CUPED variance reduction using delay severity as covariate: outcome variance reduced by 10.6%,
  narrowing the 95% confidence interval by 5.5%, with no additional data collection required.

## Data

This project uses the [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
(via Kaggle). Raw CSVs are not included in this repo -- download them from Kaggle and place
them in `data/raw/` to reproduce the analysis.

## Setup

```bash
conda create -n olist python=3.9 -y
conda activate olist
pip install -r requirements.txt
python -m notebook
```

Run the notebooks in order: `01_data_exploration` -> `02_power_analysis` -> `03_simulate_experiment` -> `04_cuped_variance_reduction`.
