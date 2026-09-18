# The pull sequence

## 0. Account and the window

`list_marketplaces` first. Never print `account_id`.

**Ask for the stockout dates.** When did it go to zero, when did it come back.

The inventory snapshot shows **today only** — there is no historical inventory
anywhere in the MCP, so the tool cannot find the window for you.

If the user does not know, infer it from a weekly units collapse to zero or near
zero, and **say on the report that the dates were inferred from sales**, with
the weeks used. An inferred window that is a week out moves every number in the
report.

## 1. The four periods

| Period | Range | Why |
|---|---|---|
| **Baseline** | 4 weeks, ending **2 weeks before** the stockout | clean rate |
| **Run-up** | the 2 weeks before | usually already degraded — shown, not used |
| **Stockout** | zero to restock | the hole |
| **Recovery** | restock to today | the curve |

The gap between baseline and stockout is deliberate. The fortnight before a
stockout is almost never normal — stock was thin, availability patchy, ads
possibly throttled. Using it as the baseline understates the loss, sometimes by
a lot.

Show the run-up separately. It is often the most useful part of the report,
because it shows the stockout was visible before it happened.

## 2. One call per week

```
for each week in baseline + run-up + stockout + recovery:
    get_product_performance(account_id, start_date=<Sun>, end_date=<Sat>,
                            group_by='product', limit=200)
```

**`granularity="daily"` is accepted and silently ignored by this tool** — no
error, no `date` field, rows come back aggregated. There is no series to slice,
so the weekly picture is genuinely one call per week.

Never take a twelve-week aggregate and divide. A stockout is precisely the shape
that destroys, and a flat line through it will be believed.

Filter to the ASIN and **sum across its SKUs**. Then derive rates.

Fields: `revenue`, `units`, `sessions`, `orders`, `conversion_rate`.

## 3. The variation family — substitution

Pull the same weekly series for the **sibling ASINs** in the variation family.
If the 8.75oz went out and the 17.6oz rose, some of the lost demand was
captured, not lost.

There is no variation-family field in the MCP. Identify siblings by title
pattern and by the client's own knowledge of the catalogue — ask if unsure.
Oxylabs `get_product` on the parent will also show the variation set.

## 4. Advertising through the window

```
for each week:
    get_product_ads(account_id, start_date, end_date, limit=500)
```

Filter to the ASIN. This answers the question that makes the report land: **was
the brand still paying for clicks on a product it could not ship?**

Ad spend during a stockout window is pure waste and it is the easiest number in
the report to act on. If it is non-zero, lead with it.

`get_product_ads` carries `ad_id`, `campaign_id`, `ad_group_id` and `state`, so
the report can name exactly which ads kept running.

## 5. Stock now

```
get_inventory_snapshot(account_id, limit=100, offset=…)
```

Paginate. Compute cover **per ASIN, never per SKU** — see
`trackiq-restock-priority`. This answers whether the recovery is about to be
interrupted by a second stockout, which happens more often than anyone expects
because the restock quantity was set before the demand was understood.

## 6. Rank — not from the MCP

Neither rank tool has history:

- `get_bsr` returns one `tracked_date` whatever range is requested, and `price`
  is null
- `get_keyword_rank` returns the tracked keyword **roster** with `organic_rank`
  and `sponsored_rank` **null on every row**

So "BSR decay through the gap" cannot be reconstructed after the fact. It is
gone.

What can be done:

- sample rank **now** with Oxylabs `search_keyword` on the target terms, and
  again weekly through the recovery
- state the pre-stockout rank only if the client has it from their own records
- otherwise say rank history is unavailable and measure recovery on sessions

Do not present the keyword roster as if it were rank data.
