# Percent of Cohort BTC Supply still Unspent

A single-file, dependency-light interactive page that compares the **survival
of "new-buyer" cohorts across every Bitcoin halving epoch** with a stratified
Kaplan–Meier survival function. Each halving epoch gets one curve tracking how
much of that cohort's BTC (or USD cost basis) is still un-spent over time. The
chart auto-extends by one day each calendar day, and the current epoch is always
plotted up to today.

**Live demo:** https://bitcointerminal.net/Percent-of-Cohort-BTC-Supply-still-Unspent/

A cohort-analysis cousin of [HODL Waves](https://unchained.com/blog/hodl-waves-1/),
viewed one halving epoch at a time.

## What it shows

For each halving epoch:

- **Cohort** — every UTXO created inside the *onboarding window* (W days,
  optionally opening B days *before* the halving) that is still un-spent at the
  close of that window. Its size then is `N₀`.
- **Survival curve S(t)** — the fraction of that cohort still un-spent at
  follow-up day `t` after the onboarding close. `S(0) = 1`.
- A faint **BTC price overlay** per epoch (shared log scale, normalized to the
  onboarding-close price) with a **cycle-top marker** on each line.
- A **crosshair readout** giving, per visible cohort, the calendar window, the
  BTC price, and the % still un-spent at the hovered day.

## Controls

- **Days before halving** (input, 0–365, default **99**) — how far before the
  halving the cohort window opens. The default 99 is the gap from BlackRock's
  IBIT spot-ETF launch (2024-01-11) to the 2024 halving.
- **Onboarding window W** — `30 / 90 / 150 / 180 / 365` days. Every value sits on
  a BRK age-band edge, so the cohort denominator `N₀` has zero interpolation
  error. (Keyboard: `A`/`←` previous, `D`/`→` next.)
- **Weighting** — **BTC** (sat-weighted supply) or **USD** (realized cap /
  cost-basis dollars).
- **Right-censor** (default ON) — truncate every cohort to the follow-up the
  current epoch has reached, so all curves are directly comparable over a common
  horizon. OFF follows each cohort to today (full history, unequal lengths).
- **Legend** — click a cohort to show/hide it. Cohort 1 (2009–2012) is hidden by
  default; its tiny early supply makes the denominator unstable.
- **Language** — English, 日本語, Français, فارسی, 中文 (cycle button).

## Method

For a halving epoch with halving date `H`, window `W`, and pre-halving offset `B`,
evaluated at follow-up day `t`:

```
window opens at  H − B,   closes at  H − B + W
N₀   = supply aged ≤ W at the onboarding close                 (cohort)
N(t) = supply aged in [t, t+W] at observation (close + t)      (survivors)
S(t) = N(t) / N₀
```

Equivalently, in product-limit (Kaplan–Meier) form
`Ŝ(t) = ∏_{i: tᵢ ≤ t} (1 − dᵢ/nᵢ)`. On-chain spends are fully observed, so there
is no per-coin censoring and the two forms coincide. At observation
`d = close + t`, original-cohort UTXOs have age in `[t, t+W]`, which resolves
back to the same birth window — so the slice tracks exactly the original cohort.

## Data source

All inputs come from a Bitcoin Research Kit (BRK) deployment at
[bitview.space](https://bitview.space/api), daily resolution
(`/api/series/<name>/day1`). One endpoint family per weighting mode:

- `utxos_under_<BAND>_old_supply_sats` — BTC mode (1 sat = 1 unit)
- `utxos_under_<BAND>_old_realized_cap` — USD mode (cost-basis dollars)

plus the `price` series for the overlay. Each band series is the cumulative
quantity of UTXOs younger than `<BAND>` on that day. Bands:

```
1h 1w 1m 2m 3m 4m 5m 6m 1y 2y 3y 4y 5y 6y 7y 8y 10y 12y 15y
```

For follow-up days between band edges the cumulative quantity is linearly
interpolated.

## Accuracy / caveats

1. **Cohorts are reconstructed from age-band totals, not per-UTXO tracking.**
   The window choices sit on band edges, so `N₀` is exact, but the curve's tail
   is interpolated once a cohort's age passes the fine 30-day edges (gaps widen
   to 1 year beyond the 6-month mark). Read tail percentages as approximate
   (typically off by a low single-digit number of percentage points, biased
   slightly low); cross-cohort comparisons are more robust than any single
   absolute value.
2. **"New buyer" is a UTXO proxy, not a wallet/person proxy** — includes
   coinbase, change, and exchange-internal moves.
3. **No confidence bands / significance tests** — omitted for readability;
   large entities spending many UTXOs at once violate independence.
4. **Cycle bottoms are not marked** — the bear-market low falls outside the
   default window and the series minimum is the onboarding-start price, not a
   real bottom. Only cycle tops (captured within the window) are marked.

## Stack

- Pure HTML / CSS / vanilla JS — no build step, single file.
- CDN deps (pinned): [TradingView Lightweight Charts 5.2.0](https://github.com/tradingview/lightweight-charts)
  (Apache-2.0) and [MathJax 3.2.2](https://www.mathjax.org/) for the formulas.
- Live data from BRK at `bitview.space`.

## Local development

No build needed. Serve over HTTP (the page makes cross-origin fetches to
`bitview.space`, which browsers may block from `file://`):

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

## License

CC0 1.0 Universal — public domain. Free to use, modify, redistribute. See
[LICENSE](LICENSE).

## Credits

- **HODL Waves** lens — [Unchained Capital](https://unchained.com/blog/hodl-waves-1/)
  (Dhruv Bansal, 2018).
- Per-epoch cohort-analysis framing — *"Deciphering Bitcoin Blockchain Data by
  Cohort Analysis"*, Yulin Liu, Luyao Zhang & Yinhong Zhao (*Scientific Data*, 2022).
- **Bitcoin Research Kit (BRK)** — open-source Bitcoin indexer:
  [github.com/bitcoinresearchkit/brk](https://github.com/bitcoinresearchkit/brk).
- **TradingView Lightweight Charts** — open-source charting library.
- **Kaplan–Meier** — Edward L. Kaplan & Paul Meier (1958), JASA 53(282).
