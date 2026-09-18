# Method

## The counterfactual

```
baseline_units_per_day   = baseline units / baseline days
baseline_revenue_per_day = baseline revenue / baseline days

lost_units   = baseline_units_per_day   x stockout_days - units sold during
lost_revenue = baseline_revenue_per_day x stockout_days - revenue during
```

"Units sold during" is rarely zero — there is usually a tail from a remnant of
stock, an FBM listing, or a seller-fulfilled fallback. Subtract it.

**This is an estimate, not a measurement.** It assumes the baseline rate would
have held through the window. Say that, and give a range:

```
sigma = stdev(weekly units across the baseline weeks) / 7      # per day
low   = lost_units - sigma x stockout_days
high  = lost_units + sigma x stockout_days
```

Never present a single figure as what the brand lost. A client who repeats an
over-precise number to their finance team will come back with questions the data
cannot answer.

## Substitution — net it off

If a sibling ASIN rose during the window, that demand was captured:

```
sibling_lift = sibling revenue during - (sibling baseline rate x stockout_days)
net_lost     = lost_revenue - max(0, sibling_lift)
```

**Show both numbers.** Gross lost revenue is the honest headline for "what the
stockout did to this ASIN"; net is the honest answer to "what did it cost the
brand". They are different questions and both get asked.

Only count lift that is plausibly substitution — a sibling size or pack count of
the same product. A different product rising in the same weeks is not
substitution, it is a coincidence, and netting it off understates the damage.

## Ad spend during the window

```
wasted_ad_spend = ad spend on the ASIN during the stockout window
```

This is the cleanest, most actionable number in the report: money spent on
clicks for a product that could not be shipped. It needs no counterfactual and
no assumptions.

If it is non-zero, **lead the report with it**, name the ads that kept running
(`get_product_ads` carries the IDs), and point at
`trackiq-inventory-ad-throttle` as the fix.

## Recovery — measured on sessions

Revenue can be bought back with ad spend, which tells you nothing about whether
organic demand returned. **Sessions is the recovery test.**

```
recovery_pct[w] = sessions in recovery week w / baseline sessions per week
```

| Recovery | Reading |
|---|---|
| **>= 95% within 2 weeks** | rank held. Short stockouts on strong products often cost nothing durable. |
| **70–95% after 4 weeks** | partial. Rank slipped and is climbing back. |
| **< 70% after 4 weeks** | rank was lost. This is a rebuild, not a recovery. |
| **flat for 3+ weeks** | it is not coming back on its own. |

Also track **organic session share**: sessions not attributable to ads. If total
sessions recovered but only because ad spend doubled, the product has not
recovered — it is being carried.

```
implied_organic = sessions - (ad clicks in the same week)
```

Rough — ad clicks and sessions are not the same unit and a click can produce
more than one session. Use it as a direction, say it is approximate, and never
quote it to a percentage point.

## Conversion through the window

Conversion usually **rises** during a partial stockout — the remaining buyers
are the determined ones — then falls below baseline after restock while the
listing re-earns its position. A conversion rate below baseline four weeks after
restock, on recovered sessions, points at something other than the stockout:
price, reviews, or a listing change. Check `trackiq-listing-monitor`.

## The staged recovery plan

Turning full spend back on the day stock lands wastes it against a rank that
cannot hold it yet.

| Stage | When | What |
|---|---|---|
| **1. Confirm** | day stock lands | check availability is live, buy box held, price right |
| **2. Restart defensively** | days 1–3 | branded and exact-match terms only, bids at pre-stockout levels |
| **3. Rebuild** | days 4–14 | broad and auto back on, bids up to 20% above pre-stockout to re-enter auctions |
| **4. Normalise** | day 15+ | back to standard bids; if sessions are not at 90%, it is a rank rebuild, not an ad problem |

State which stage the ASIN is in on the day the report is run.

Stage 3's higher bids are deliberate and temporary: re-entering an auction costs
more than staying in it. Cap it at 14 days and say so, or it quietly becomes the
new normal.

## What this skill does not do

- **No rank history.** Neither `get_bsr` nor `get_keyword_rank` carries any.
- **No daily series.** The tool ignores `granularity="daily"`.
- **No prediction of recovery time.** Report the curve so far and the reading.
- **One stockout per report.** A catalogue-wide problem is
  `trackiq-restock-priority`.
