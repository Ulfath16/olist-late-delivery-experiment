# Decisions Log: Olist Late-Delivery Experiment Project

| # | Decision | Rationale |
|---|---|---|
| 1 | Population = order_status == 'delivered' with non-null delivered + estimated dates | 2,971 orders dropped (non-delivered or missing dates -- can't compute lateness without both) |
| 2 | Duplicate reviews (547 orders) -> keep latest review_answer_timestamp | Validated across all 547 cases: 63% identical score (likely duplicate survey pings), 37% genuine score changes, roughly symmetric direction (113 down, 88 up) -- no directional bias introduced |
| 3 | 646 orders with no review excluded from review-based metrics (not imputed) | No response does not equal a neutral opinion; imputing would fabricate signal |
| 4 | is_late defined as delivered_customer_date > estimated_delivery_date | Standard, matches what a customer actually experiences |
| 5 | North star metric = negative review rate (score <= 2) | Real baseline: 9.2% on-time vs 54.1% late -- confirms lateness is a major driver, justifying the feature |
| 6 | customer_unique_id used for any repeat-customer logic, never customer_id | customer_id is generated fresh per order in this dataset; using it would make repurchase rate look like 0% for everyone |
| 7 | Item-level and payment-level tables aggregated to order level (sum price/freight, count items) before joining to orders | order_items and order_payments have multiple rows per order; joining directly without aggregating first would silently duplicate order-level rows |
| 8 | Minimum Detectable Effect = 8 percentage points (54.1% -> ~46%) | Smaller effects (3-5pt) require 3,000-8,700 orders (over 1-2 years runtime) -- not practically actionable. 8-10pt is detectable in ~2-3 months and matters to the business. |
| 9 | Required sample: ~612 orders per arm (~1,224 total) | At 5% significance, 80% power, two-sided test |
| 10 | Randomization unit = customer_unique_id, not order_id | Only 0.7% of customers with a late order have more than one late order in the data -- randomizing by customer avoids contamination (same customer in both arms) at almost no sample cost |
| 11 | Ground-truth baseline probability modeled as a logistic function of delay days, not a flat rate | Empirical buckets showed negative-review rate climbing from ~15% (0-2 days late) to ~80% (10-50 days late) -- a flat baseline would have been unrealistic |
| 12 | Treatment effect modeled as exponential decay (largest for mild delays, near-zero for severe delays) | Encodes the hypothesis that a heads-up notification helps most when the delay still feels recoverable to the customer |
| 13 | Kept ground-truth probabilities alongside simulated binary outcomes | Enables validating that the statistical test recovers something close to the true planted effect -- most simulation projects skip this check |
| 14 | Used two-proportion z-test (not t-test) on the binary outcome | Negative-review is a binary indicator; a t-test would be a common but technically incorrect choice here |
| 15 | Validated the testing pipeline against ground truth before trusting it on real (unknown-truth) data | True ATE (-9.50pts) fell inside the observed 95% CI (-10.77, -6.35) -- confirms the test isn't biased before relying on it |
