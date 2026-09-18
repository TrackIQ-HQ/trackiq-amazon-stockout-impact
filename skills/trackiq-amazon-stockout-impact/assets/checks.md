# Before you send it

## 1. The window

- The stockout dates are stated, with **whether they came from the client or
  were inferred from sales**.
- If inferred, the weeks used are shown, and the report says every figure moves
  if the dates are wrong.
- The baseline is **four weeks ending two weeks before** the stockout, and the
  gap is explained.
- The run-up fortnight is shown separately and is **not** part of the baseline.

## 2. The weekly data is real

- **One call per week.** No week's figure came from dividing a longer aggregate.
- Check for identical weeks: two weeks equal to the cent is the signature of a
  divided aggregate.
- SKUs were summed to the ASIN before any rate was derived.

## 3. The counterfactual is labelled

- Lost units and lost revenue are presented as **estimates with a range**.
- The word "lost" is never used without the assumption beside it.
- Units actually sold during the window were subtracted — the figure is rarely
  zero.
- No single-figure total appears without its range.

## 4. Substitution

- Sibling ASINs were checked.
- **Gross and net lost revenue are both shown**, and the difference is
  explained.
- Only plausible substitutes were netted off — same product, different size or
  pack. Not an unrelated product that happened to rise.

## 5. The ad spend

- Ad spend during the stockout window is computed and shown.
- If non-zero, it **leads the report** and the specific ads are named.
- The report points at `trackiq-inventory-ad-throttle` as the fix.

## 6. Recovery

- Recovery is measured on **sessions**, not revenue.
- The organic-session estimate is labelled approximate and not quoted to a
  percentage point.
- The recovery reading (rank held / partial / rebuild / flat) is stated with the
  number behind it.
- Current stock cover was checked — the report says whether a second stockout is
  coming.

## 7. Rank

- **No rank history is presented.** If rank appears, it is a sample taken now
  with Oxylabs, labelled with its date.
- The keyword roster from `get_keyword_rank` is not shown as rank data anywhere.
- If pre-stockout rank came from the client's own records, it is attributed to
  them.

## 8. The plan

- The current stage (confirm / restart / rebuild / normalise) is named.
- Stage 3's elevated bids are **capped at 14 days** and the cap is stated.
- A conversion rate still below baseline on recovered sessions is flagged as
  *not* a stockout effect, pointing at listing, price or reviews.

## 9. Render check

```js
({ overflows: document.documentElement.scrollWidth > document.documentElement.clientWidth,
   tables: document.querySelectorAll('table').length,
   weeks: document.querySelectorAll('table')[0].querySelectorAll('tbody tr').length,
   logos: [...document.images].map(i => i.naturalWidth > 0),
   tokens: (document.body.innerHTML.match(/\{\{[A-Z0-9_]+\}\}/g) || []).length,
   // the estimate must never appear without a range beside it
   ranges: (document.body.innerText.match(/estimate|range|assumes/gi) || []).length })
```

`overflows` false, `logos` all true, `tokens` zero, `ranges` at least 2. Then
look at it; if it will not paint, say the check was structural.

## 10. Ship

Save as `<client>-stockout-impact-<ASIN>-<YYYY-MM-DD>.html`.

Lead the message with the wasted ad spend if there was any — it is the one
number in the report that needs no assumptions and has an obvious fix. Lead with
the net lost revenue otherwise, and say it is an estimate in the same sentence.
