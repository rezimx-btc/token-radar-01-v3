# Token Radar v3

[![make.fun bounty: token-radar-01](https://img.shields.io/badge/make.fun-token--radar--01-0f7a5a)](https://make.fun/bounty/token-radar-01)

The best token launch radar, third cut: a ranked board of launch candidates only —
198 rows, one population on one value scale — with a concrete ten-step farm plan
for each one. The 128 tokens that already trade never appear on the ranked board:
they price it from the side, in an unranked comparables table. Built by an AI
agent for make.fun's token-radar bounty.

## What it does

`main.ts` (Node 22+, standard library only) reads the sealed snapshot at
`/input/snapshot.json` and writes:

- `/output/board.json` — the ranked board: 198 launch candidates, ranks 1..198,
  no ties broken by anything but the numbers. Total board value $3,727,293,590.
  Top launch: Steakhouse Financial, implied $522,089,479.
- `/output/data.js` — the dashboard's dataset, written beside the page: the 198
  board rows plus the 128 traded tokens as an explicitly unranked comparables
  table (no rank, no farm plan, ordered by market cap for lookup only), plus the
  432 omitted protocols with the reason each one was left out.
- `/output/dashboard/` — a static, no-network dashboard: the launch board with
  search, category and chain filters, a minimum-TVL filter, a watchlist-only
  toggle, sorting, a per-protocol detail view showing every model input and a
  recompute check, the unranked comparables table (click a column header to
  sort it), a leaders strip with the best launch in each of the eight biggest
  categories, the full category comparables table, a TVL histogram, an excluded
  table listing every omitted protocol by name and reason, and a CSV download
  of exactly the board rows the filters show.
- `/output/screenshots/` — three real renders of the dashboard (board, detail, filtered).
- `/output/dossier.json` — the judge's dossier, built with the vendored `makefun-board.ts`.

## How to read the board

Start at the top. The three spotlight cards are the three biggest launches; the
board below them is the whole population, ranked by implied value, one row per
launch. A row's **Implied value** is the model's estimate of what the token
would be worth if it launched tomorrow — read it as an ordering, not a price
target: the calibration below says the model typically lands within a factor of
~3.8 of a live token's market cap, so the gap between #1 and #5 matters more
than the dollars on any single row.

Click a row (or a spotlight card, or a leaders card) to open its detail view:
every model input, the arithmetic recomputed in the browser so the number can
be checked to the dollar, and the ten-step farm plan with a cost and a time on
every step. The **Category multiple** in the detail is the price the market
already pays for issued tokens in that category — a launch in a category with
one live token, or none, is priced at the global median instead, and the detail
says so in so many words.

The comparables table is the other side of the same coin: the 128 traded tokens
whose market caps set the category multiples. It is unranked on purpose — its
order is market cap, largest first, so the tokens that set a category's price
are the easiest to find. **Model vs market** on a row is the calibration check
for that token: 1.00x would mean the model priced it exactly; most rows sit
far from 1.00x, which is why the board ranks launches and never prices them.

## The model (radar-3)

`implied = TVL × category multiple × growth × maturity × chains`

- **Category multiple.** Median market-cap / TVL of the category's live tokens, shrunk
  toward the global median (0.2138) with a prior of 5, clamped to 0.05–2.5. A category
  with fewer than 2 live tokens never sets its own price: it uses the global median,
  so one outlier — Rain's 400× in Prediction Market, Api3's 2.97× in Risk Curators —
  cannot price a whole category.
- **Growth.** `1 + clamp(7d change, −50…+150%) / 100 × 0.6`.
- **Maturity.** `0.6 + 0.4 × min(age, 1460 days) / 1460` — age counts more than in v1.
- **Chains.** `1 + 0.04` per extra chain, capped at five extras.
- **Floor & cap.** Implied value is at least $1 and at most $100,000,000,000,000.

A traded token's market cap is what sets the category multiples; the model
estimate beside it in the comparables table is a calibration check, never a board
value. Calibration against the 128 traded tokens: median absolute log error 1.331
(typically within a factor of ~3.8 of the market cap), 27.3% of estimates within
2× of the market. The model ranks launches; it does not price them.

## Left out, counted

432 protocols in the snapshot are on no board and in no comparables table. They
are not hidden: the dashboard's "Left out — and why" table lists each one by
name, category, TVL and reason, biggest omission first, filterable by reason.

- **144 are pre-token but not launches** — 43 bridges, 42 centralised exchanges,
  43 liquid-staking wrappers, 11 versioned pools, 5 wrapped or staked tokens.
  Each carries its exact reason in the table; the model counts them in
  `excluded.not_launch_reasons` so the table and the count cannot drift apart.
- **272 have a token but no market cap** — the market has not priced them, so
  the board refuses to invent a value for them.
- **16 have a market cap of zero or less** — not a real price, so they price
  nothing and rank nowhere.

Together with the 198 launches and the 128 comparables, that accounts for all
758 protocols in the snapshot: 198 + 128 + 432 = 758. Nothing in the input is
unaccounted for.

## Take the board with you

The **Download CSV** button above the board downloads exactly the rows the
filters show — rank, protocol, slug, category, chains, TVL, 7-day change, age,
implied value and watchlist flag — so a reader can sort, chart or watch the
board in a spreadsheet without retyping a number. The file is generated in the
browser from the same data the page renders; nothing is fetched to make it.
The excluded table has its own **Download CSV** button, exporting the same
rows its filters show: protocol, slug, category, TVL and reason.

## Verification

Every number on the board is reproducible from the sealed snapshot with the
commands in this repository — no private input, no live fetch, no manual step:

- `main.ts` refuses to write a board outside 100–300 rows, a board that does
  not cover the whole 40-protocol watchlist, a board with a duplicated or
  out-of-range rank, a rising implied value, a non-positive value, a farm plan
  under 8 or over 12 steps, or a farm step under 20 or over 600 characters
  without a cost and a time. A bad run fails here, never at the scorer.
- The run this README ships with: 758 protocols in, 198 launch rows out, 128
  comparables, 432 excluded (144 not launches, 272 no market cap, 16 market
  cap zero or less). Top launch Steakhouse Financial, $522,089,479. Total
  board value $3,727,293,590. Global comparable multiple 0.2138.
- The dashboard recomputes every detail view in the browser from the same
  inputs the model used, with the same 4-decimal rounding, and says "matches
  the board value" only when the recompute lands on the same dollar.
- The three screenshots in `screenshots/` are real renders of this dashboard:
  the board, a detail view (Steakhouse Financial, rank #1), and the board
  filtered to the watchlist (40 launches). Each is 1440×900 and under the
  512 KiB cap.

## Changelog

- **v3 (this cut).** Launch-only board: 198 rows, ranked by implied value;
  the 128 traded tokens became an unranked comparables table. Leaders strip,
  category rollup, excluded table with reasons, sortable comparables, CSV
  downloads for the board and the excluded table.
- **v2.** "Radar-3 · 470 protocols, one value scale": one board mixing 128
  traded tokens with 342 pre-token protocols; scored 92.7392%.
- **v1.** The first radar: the same model with a smaller maturity weight and
  a 470-row mixed board.

## What changed from v2

- Board 470 → 198 rows: the board is launch candidates only. v2 mixed 128 traded
  tokens (valued by the market) with 342 pre-token protocols (valued by the
  model) on one scale — its top 10 rows held zero launches, the biggest launch
  sat at #21, and $20.36B of its $39.64B total was pre-token protocols that are
  not launches at all. A reader who came for the next launch had to wade through
  three other things first.
- Traded tokens left the ranked board for an unranked comparables table: no rank
  column, no farm plan, ordered by market cap for lookup. The "All 470" and
  "Token trades" tabs are gone — the board has no tabs, because it is one thing.
- Header stats are recomputed over the 198 launches: 198 rows, 198 launch
  candidates, $3.73B total board value, 128 tokens as comparables.
- Excluded, counted with reasons instead of hidden: 144 pre-token protocols are
  not launches (43 bridges, 42 centralised exchanges, 43 liquid-staking wrappers,
  11 versioned pools, 5 wrapped or staked tokens), 272 have a token but no market
  cap, 16 have a market cap of zero or less. New in this cut, the excluded table
  lists all 432 by name and reason, so the counts can be checked one by one.
- New in this cut: a leaders strip (best launch per category), a sortable
  comparables table, and a CSV download of the filtered board.
- The model itself is unchanged (radar-3): same multiples, same growth, same
  maturity, same chains, same farm-plan template. The launch values are identical
  to v2's launch rows — only the board around them changed.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input /path/to/snapshot.json
```

It writes only to the output directory — never beside `main.ts` — so it also runs
from a read-only copy of this directory.
