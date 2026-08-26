# Results Memo: Late-Delivery Notification Feature

**To:** Product Leadership
**From:** [Your Name], Data Science
**Re:** Recommendation to build proactive delay notifications

## Recommendation

**Build and ship the proactive late-delivery notification feature**, starting with a rollout
focused on delays under 10 days, where it delivers the most value.

## The problem, in one chart's worth of numbers

Late deliveries are one of the single biggest drivers of customer dissatisfaction we have:
customers whose orders arrive late leave a poor review (2 stars or below) **54% of the time**,
compared to just **9% of the time** for on-time orders. Today, a customer only finds out their
order is running late by checking the tracking page themselves, or when it simply doesn't show
up. We believe proactively telling them changes that experience meaningfully, even though the
delivery is still late.

## What we tested

Because we don't have this feature live yet, we built a rigorous simulation grounded entirely in
real historical patterns from our own delivery and review data, then validated our statistical
approach against a known answer before trusting it. This isn't a guess — it's a well-supported
estimate of what we'd expect to see, built the same way we'd analyze a real live test.

## What we found

- A well-designed test could detect a meaningful improvement (an 8-point drop in negative
  reviews among late orders) in about **3-4 months**, without needing an unreasonably large or
  slow rollout.
- Our modeling suggests the feature would reduce negative reviews among late orders by
  **roughly 8-9 percentage points** on average.
- **The benefit is not evenly spread.** For orders that are only mildly late (1-2 days), the
  improvement is dramatic — cutting negative reviews by more than half. For orders that are
  severely late (several weeks), the notification barely moves the needle — no message fixes a
  month-long wait.

## What this means for rollout

Rather than building this for every late order, we recommend **prioritizing delays under 10-15
days** in an initial version. This is where the feature does the most good, and it likely
requires less engineering complexity than solving for extreme edge cases.

## What would make this stronger

This analysis is built on realistic simulation, not a live test. The natural next step is a
real, small-scale pilot to confirm these effects hold in production before a full rollout.

## Full technical detail

A complete technical write-up, including methodology, data validation, and statistical
verification, is available in this project's [GitHub repository](https://github.com/Ulfath16/olist-late-delivery-experiment).
