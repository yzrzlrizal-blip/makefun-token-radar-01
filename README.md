# makefun-token-radar-01

[![make.fun · token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/yzrzlrizal-blip/makefun-token-radar-01/rank.svg)](https://make.fun/bounty/token-radar-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/yzrzlrizal-blip/makefun-token-radar-01/reward.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)

Token launch radar: ranks pre-token DeFi protocols by implied launch value from a sealed DefiLlama snapshot, with a sortable dashboard, 7-step farm plans and a dossier. It does not predict whether or when a token launches.

## Run it

```sh
mkdir -p out
MAKEFUN_OUTPUT_DIR=out node main.ts --input <path-to>/snapshot.json
```

---

<sub>Entered on [make.fun](https://make.fun/?ref=wdbz3sg5&utm_source=github&utm_medium=readme&utm_campaign=token-radar-01) · AI agents build software for cash rewards</sub>

**What it does:** reads the sealed DefiLlama snapshot (`input/snapshot.json`,
758 protocols as of 2026-11-14, watchlist floor $50,000,000), finds every
protocol that records no token and is not a wrapped/staked product of one
that trades, and ranks them by an implied value — what a launch would be
worth if the protocol's record stayed as it is. It does **not** predict
whether or when a token launches.

## The board

- **178 rows** — the 40-protocol watchlist in full, plus the 138 pre-token
  protocols below the floor that pass the watchlist's own clearing rule
  (no token on record; not a bridge, liquid-staking/restaking or CEX
  product; no wrapped/staked word in the name; no exchange prefix; no
  version marker). Wrapped and staked products of tokens that already
  trade — WBTC, Binance staked ETH, USDT0 — are off the board, as they
  are off the watchlist. Above the floor, no protocol the watchlist does
  not name is ranked: the snapshot cannot show whether DefiLlama records
  a parent protocol for it, so the radar does not guess.
- Live-token protocols are set aside with their market caps (the market's
  own value), never mixed into the ranking.

## The model — `radar-2` (`lib/model.ts`, function `impliedValue`)

Only the snapshot's own figures, the same function for every protocol,
no name ever special-cased:

```
implied = round( tvl_usd x 0.30
  x (1 + clamp(change_7d, -50, 150) / 100 x 0.60)
  x (0.65 + min(age_days, 1460) / 1460 x 0.35)
  x (1 + 0.02 x min(chains - 1, 5)) )
```

- **TVL × 0.30** — the size of the business a token would capitalise.
- **Seven-day growth** — momentum, clamped so one wild week cannot dominate.
- **Age with no token** — how long value has accrued without a claim on it
  (unknown listing date = 365 days).
- **Chain breadth** — TVL spread over more chains is stickier (at most ×1.10).

## Farm plans

Every row carries 7 farm steps (`farmSteps` in `lib/model.ts`), derived
from the protocol's category, chains, TVL tier and age — the action fits
the category, the capital fits the TVL ($50–$250), the hold fits the age
(21–60 days) — and every step states its cost and its time. A plan farms
the protocol's own activity; it promises no token and no airdrop, because
the snapshot records none, and it warns never to pay to claim anything.

## The dashboard

`dashboard/` is a static page with no network: `index.html`, `app.css`,
`app.js`, plus `data.js` (written by `main.ts` from the same board).
It has a sortable board, search, category + chain filters, a watchlist-only
toggle, a live-token tab, a detail view for every protocol (model
breakdown, recomputed implied value, 7-step farm plan), and a what-if
sandbox that recomputes implied value with the same function — defaults
match `board.json` exactly.

## Run it

```sh
mkdir -p out
MAKEFUN_OUTPUT_DIR=out node main.ts --input <path-to>/snapshot.json
```

Requires Node 22+. Writes `out/board.json`, `out/data.js`,
`out/dashboard/`, `out/screenshots/` and `out/dossier.json`.
Runs are deterministic: two runs write the same bytes.

## Files

| File | What it is |
| --- | --- |
| `main.ts` | Entrypoint: snapshot in, board + dashboard + dossier out |
| `lib/model.ts` | The radar-2 model, board builder and farm plans |
| `vendor/makefun-board.ts` | make.fun dossier library, copied unchanged from the kit |
| `dashboard/` | Static dashboard (no network, no framework) |
| `screenshots/` | 1-board.png, 2-detail.png, 3-filtered.png (captured with headless Chromium) |
| `board.json`, `data.js`, `dossier.json` | Outputs of a local run of `main.ts` |
| `LICENSE` | MIT |

## Status

Local build only. Not submitted. Submission still needs: a public GitHub
repository in Udin's linked GitHub account (created at/after the
competition opened, 2026-11-14), the agent transcripts of the build
sessions, and submission via the make.fun CLI before Mon 14 Dec 2026
17:00 WIB.
