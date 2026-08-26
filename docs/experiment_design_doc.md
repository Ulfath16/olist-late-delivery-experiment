# Experiment Design Doc: Proactive Late-Delivery Notification

**Status:** Proposed

## 1. Background & Problem

Analysis of historical order data shows that late deliveries are strongly associated with
negative customer reviews:

- On-time orders: 9.2% negative review rate (score <= 2)
- Late orders: 54.1% negative review rate

This ~45pt gap makes late delivery one of the largest single drivers of dissatisfaction we can
observe in the data. Today, when a shipment is going to miss its estimated delivery date, the
customer has no way of knowing until they check the tracking page or the package simply doesn't
arrive. We hypothesize that proactively notifying the customer once a delay is detected (rather
than leaving them to discover it) will reduce the damage to satisfaction, even though the
delivery itself is still late.

## 2. Hypothesis

Proactively notifying customers about an in-progress delivery delay reduces the negative review
rate among late orders, relative to no notification, without materially increasing cancellations.

## 3. Population & Eligibility

- Eligible orders: orders confirmed and shipped, where the system detects the delivery is now
  projected to miss order_estimated_delivery_date.
- Excluded: orders that end up canceled, returned, or lost in transit before a delay is even
  detectable.

## 4. Randomization

- Unit of randomization: customer (customer_unique_id), not order.
  Rationale: if a customer has multiple late orders during the test window and we randomized by
  order, they could receive the notification for one and not the other, contaminating their
  reaction. Checked in the data: only 0.7% of customers with a late order have more than one
  late order in this window, so randomizing by customer costs almost no statistical power while
  eliminating this contamination risk entirely.
- Split: 50/50 treatment (notification) vs. control (no notification, current experience).

## 5. Metrics

| Type | Metric | Definition |
|---|---|---|
| Primary (North Star) | Negative review rate | % of orders with review score <= 2 |
| Secondary | Repurchase rate | % of customers placing another order within 90 days |
| Guardrail | Cancellation rate | % of eligible orders canceled before delivery |
| Guardrail | Review response rate | % of eligible orders that receive any review at all |

## 6. Baseline & Minimum Detectable Effect

- Baseline negative review rate among late orders: 54.1%
- MDE: 8 percentage points (target: reduce to ~46%)
  - Smaller effects (3-5pt) would require 3,100-8,700 orders, implying over 1-2 years of runtime
    at current late-order volume (~319/month) -- not practically actionable.
  - An 8-10pt reduction is both statistically detectable in ~2-3 months and large enough to
    represent a real business win in retention/support-cost terms.

## 7. Sample Size & Duration

- Required sample: ~612 customers per arm (~1,224 total), at alpha = 0.05, power = 0.80,
  two-sided test.
- At ~319 late orders/month, expect to reach required sample in ~4 months.

## 8. Risks & Considerations

- Notification fatigue / false alarms: if the delay-detection system flags a shipment as late
  and it then arrives on time, the notification could create unnecessary anxiety.
- Novelty effect: an early positive result could partly reflect reaction to a new touchpoint
  rather than the substance of the message.
- Estimated dates are already conservative: median actual delivery is ~12 days earlier than the
  estimate, meaning Olist pads its promises.

## 9. Decision Criteria

Ship the notification feature if, at the end of the test:
- Negative review rate among late orders drops by >= 8pts with statistical significance (p<0.05), and
- Cancellation rate does not increase beyond a pre-agreed non-inferiority margin (e.g. +1pt).
