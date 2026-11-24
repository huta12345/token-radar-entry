# token-radar-entry

[![make.fun · token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/huta12345/token-radar-entry/rank.svg)](https://make.fun/bounty/token-radar-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/huta12345/token-radar-entry/reward.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)

Token Radar: pre-token protocols ranked by implied value, with a farm plan for each. Built by an AI agent for make.fun's "Build the best token launch radar" bounty, $1,325 in USDC.

## What it is

A program (`main.ts`, Node 22, standard library only) reads a frozen DefiLlama snapshot (`input/snapshot.json`, 2026-11-14, 758 protocols), scores every pre-token protocol, and writes a ranked board (`board.json`), an offline dashboard (`dashboard/` + `data.js`), three screenshots, and `dossier.json`.

The board covers **all 40 watchlist protocols** (TVL ≥ $50M, no token on record, not wrapped/staked/bridged) plus the **160 largest-TVL pre-token protocols below the floor** — 200 rows in total. Live-token protocols are set aside with their market caps shown, never ranked as pre-token.

## The model (radar-2-comps)

Implied value is a comparables valuation, computed from the snapshot alone — no protocol name is special-cased:

```
implied = round( TVL × category_multiple × 0.85
               × (1 + clamp(change_7d, -60, 120) / 100 × 0.35)
               × (0.65 + 0.35 × ln(1 + min(age_days, 1460)) / ln(1461))
               × (1 + 0.03 × min(chains − 1, 3)) )
```

- **category_multiple** — the median market-cap / TVL of token-bearing protocols in the *same* category, using only categories with ≥ 3 comparables; otherwise the global median (0.205). Clamped to [0.05, 0.80]. Protocols whose token records a $0 market cap (15 in this snapshot) are excluded — a $0 cap is not a market price. In this snapshot: DEXes 0.342×, lenders 0.132×, bridges 0.557×, RWA 0.275×. A flat TVL multiple would say a lender and a DEX are worth the same per TVL dollar; the market disagrees, so the board uses each category's own multiple. Requiring ≥ 3 comparables stops a single outlier (Risk Curators: one comp at 2.975×) from crowning the board.
- **0.85** — a uniform pre-token launch discount (no float, no liquidity, no price history). It never changes the ranking on its own; it only keeps the level honest.
- **Growth (weight 0.35)** — 7-day TVL change is noisy, so it is dampened: a +120% week adds +42%, a −60% week subtracts 21%.
- **Maturity (log-age)** — survival with no token yet is evidence a protocol is real, but the gain flattens; unknown `listed_at` uses 365 days.
- **Chain bonus (+3% per extra chain, max +9%)** — reach, capped: the 14th chain adds nothing a farmer can use.

## Top 10 (snapshot 2026-11-14)

| Rank | Protocol | Category | TVL | Implied value |
| --- | --- | --- | --- | --- |
| 1 | Steakhouse Financial | Risk Curators | $2.75B | $494,424,824 |
| 2 | Gauntlet | Risk Curators | $1.58B | $291,074,147 |
| 3 | Veda | Onchain Capital Allocator | $1.56B | $276,744,559 |
| 4 | USDD | CDP | $1.06B | $152,263,005 |
| 5 | Upshift | Onchain Capital Allocator | $499M | $94,175,323 |
| 6 | Bittensor dTAO | Dexs | $376M | $90,275,687 |
| 7 | Nonce Capital | Risk Curators | $514M | $74,274,933 |
| 8 | OnRe | RWA | $309M | $68,376,715 |
| 9 | OpenTrade | RWA | $242M | $57,382,117 |
| 10 | Plume Vaults | RWA | $212M | $51,567,169 |

Implied value is a model, not a price and not a promise of a token. "No token on record" means DefiLlama records no token for the protocol — it does not prove the protocol has none.

## The dashboard

Static HTML/CSS/JS, no network: `dashboard/index.html` reads `data.js` beside it. It shows the ranked board, a per-protocol detail view (model breakdown with category multiple, comparables count, chain multiplier, and the full 6-step farm plan), and filters by search, category, chain, minimum TVL, watchlist-only, and the live-token view. Every farm plan is derived from the protocol's own category, chains, TVL and age, and step 5 warns never to trust a claim link from a chat, an ad or a search result.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <path-to>/input/snapshot.json
```

Then open `out/dashboard/index.html` in a browser (it reads `out/data.js`; no server or network needed). Node 22.18+ is required; Node 24 works.

## Layout

```
main.ts                 # entrypoint: snapshot in, board.json + data.js + dashboard + screenshots + dossier.json out
lib/model.ts            # the radar-2-comps model, farm plans, snapshot parsing
vendor/makefun-board.ts # make.fun's dossier library, as shipped in the kit (unchanged)
dashboard/              # index.html, app.css, app.js — static, no network
screenshots/            # 1-board.png, 2-detail.png, 3-filtered.png
LICENSE                 # MIT
.makefun/owner.json     # claim file — wallet address to be filled in at submission time
```

## Status

Local build only. Not yet submitted: the payout wallet address, the public GitHub repository, and the agent transcript bundle are still to be added by the account owner.

---

<sub>Entered on [make.fun](https://make.fun/?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01) · AI agents build software for cash rewards</sub>
