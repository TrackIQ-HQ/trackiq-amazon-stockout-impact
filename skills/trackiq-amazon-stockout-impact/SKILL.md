---
name: trackiq-amazon-stockout-impact
description: Measures what a completed Amazon stockout actually cost — lost units and revenue against the pre-stockout run rate, how far sessions and conversion fell, whether they have come back, and how long recovery is taking — then produces a staged plan for restarting ads and rebuilding rank. Use when the user asks what a stockout cost, lost sales from being out of stock, stockout impact, recovery after a stockout, how long to recover rank, or why sales have not bounced back after restocking.
---

# Stockout Impact & Recovery

The skill you run **after** the thing you were trying to prevent.

**What did that stockout cost, and how do we get the rank back?**

Output is a branded HTML report: the cost, the recovery curve so far, and a
staged plan.

## Requires

- The TrackIQ MCP, for `list_marketplaces`, `get_product_performance`,
  `get_inventory_snapshot` and `get_product_ads`.
- **The stockout window** — when the ASIN went to zero and when it came back.
  Ask; the inventory snapshot only shows today. See non-negotiable 2.
- **The Oxylabs scraper**, if rank recovery is wanted. The MCP has no rank
  history.
- Nothing else. No filesystem, no shell, no internet.
- **Without the MCP:** works from weekly Business Report exports covering the
  weeks before, during and after.

## First run

Fill in a copy of `assets/account.example.md` saved as account.md beside the
skill. Every TrackIQ skill reads the same file, so an account already set up
for another TrackIQ report needs nothing added here.

If the runtime has no filesystem, print the same block and ask the user to
paste it into their project instructions once.

## Read first

- `assets/pulls.md` — the calls, week by week, and the three histories that do
  not exist
- `assets/method.md` — the counterfactual, and why recovery is measured on
  sessions
- `assets/checks.md` — what to verify before anything is sent

Copy `assets/report-template.html` and replace every `{{TOKEN}}`.

## Non-negotiables

1. **There is no daily series per ASIN.** `get_product_performance` accepts
   `granularity="daily"` and silently ignores it. Build the before / during /
   after picture from **separate calls over separate windows**, weekly. Never
   divide a long aggregate into days.
2. **The inventory snapshot has no history.** It shows stock now. It cannot tell
   you when an ASIN went to zero or when it came back. **Ask the user for the
   dates**, or infer them from a weekly units collapse and say plainly that they
   were inferred.
3. **The baseline excludes the run-up.** The two weeks before a stockout are
   usually abnormal — stock was already thin, ads may have been throttled,
   availability was patchy. Take the baseline from the **four weeks ending two
   weeks before** the stockout started, and say so.
4. **Lost revenue is a counterfactual, not a measurement.** It assumes the
   baseline rate would have held. Present it as an estimate with a range, never
   as a figure the brand definitely lost.
5. **Subtract substitution.** If a sibling ASIN's sales rose during the window,
   some of the "lost" demand was captured. Check the variation family and net it
   off; state the figure both ways.
6. **Recovery is measured on sessions, not revenue.** Revenue can be bought back
   with ad spend and tells you nothing about whether organic demand returned.
   Sessions at the pre-stockout rate is the recovery test.
7. **There is no rank history in the MCP.** `get_bsr` and `get_keyword_rank`
   both return one `tracked_date` regardless of the range requested, and
   `get_keyword_rank` returns null ranks. Rank recovery must be sampled from
   Oxylabs going forward, or left out and said to be unavailable. Never present
   a roster as a history.
8. **Recovery is staged, not switched.** Turning full ad spend back on the day
   stock lands wastes it against a rank that cannot hold. The stages are in
   `assets/method.md`.
9. **One stockout per report.** Several at once is a catalogue problem and
   belongs in `trackiq-restock-priority`.
10. **Never print `account_id`.**

## What it pairs with

`trackiq-restock-priority` exists so this skill never has to run.
`trackiq-inventory-ad-throttle` is what should have happened on the way down —
if ads ran at full spend into a zero, this report will show what that cost, and
that is the argument for installing the throttle.

## Delivery

The output is produced in the chat first. Delivery is the last step and the
method comes from the Delivery block in account.md — never ask per run.

| Method | What to do | Needs |
|---|---|---|
| `in-chat` | Return the report. The default, and the fallback for every other method. | nothing |
| `file` | Write it beside the skill, dated. | a filesystem |
| `slack` | Post the headline findings as text, then upload the file. | a connected Slack tool |
| `n8n` | POST it to the configured webhook. | network access |
| `email` | Hand it to the connected mail tool. | a connected mail tool |

Confirm before the first outward send of a session, fall back to in-chat
loudly when a method is unavailable, and never substitute a different
outward channel.

## Version

`trackiq-amazon-stockout-impact` v1.0.0 (2026-09-18).

If the user asks whether this skill is current, fetch
`https://trackiq.com/skills/registry.json`, compare the `version` field for
`trackiq-amazon-stockout-impact`, and if it is newer, give them the download link and
the one-line changelog. Do not fetch at any other time.
