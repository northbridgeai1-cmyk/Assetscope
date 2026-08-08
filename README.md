# SwingScope

Crypto-only swing-trade **research** tool. Candle-based technical checklist, a volatility-sized stop-loss/take-profit suggestion for every coin, a probability projection, a way to discover coins by category, and a manual trade journal to help you enforce your own 1-week max-hold rule.

## What this is not

**SwingScope does not connect to Phantom or any other wallet, and cannot place, size, or execute a real trade.** Two separate reasons, both hard limits:

1. **I don't build tools that execute real trades or move funds on someone's behalf.** That's true regardless of how the request is framed — a rule, not a preference.
2. **It isn't technically possible to build "connect and it trades unattended" the way that phrase implies, even setting policy aside.** Phantom (like every non-custodial wallet) requires you to manually approve each transaction in its own popup, by design — specifically so a website can never move your assets without you personally clicking approve, every single time. A dApp can *request* a signature; it cannot silently trade on a schedule for a week on its own. Any product that claims otherwise is either lying or asking you to hand over a private key/session authority in a way that is a massive rug-pull/drain risk — doubly so for anything deployed as public, inspectable client-side code on GitHub Pages, where literally anyone can read every line.

What SwingScope does instead: the research, the checklist, and the numbers — stop-loss, take-profit, probability range — so you can place the actual order yourself, on whatever exchange or wallet you trade with. You stay the only one who can ever move your money.

**Not financial advice.** Crypto is highly volatile and speculative. Every score describes liquidity/technical/momentum characteristics, not a judgment on a project's merit, and nothing here is a guarantee.

## Setup

**None.** No API key, no signup, no wallet connection. Everything runs on CoinGecko's free public "keyless" API (~10–30 requests/minute, shared per IP — SwingScope paces its requests to stay well under that). Upload `index.html` + this `README.md` to a GitHub repo, turn on Pages, done — same deploy steps as QuantFolio/AssetScope if you need a refresher.

## How it works

### Fundamentals (4 points) — "is this tradeable, not a ghost coin?"
Market cap rank (top 300), liquidity (24h volume ÷ market cap ≥ 3%), supply dilution risk, exchange breadth (listed on 8+ exchanges). Filters out illiquid coins where a stop-loss might not even fill cleanly.

### Technical / Candle Signals (5 points) — computed from daily closing prices
| Check | What it means |
|---|---|
| Trend Structure | Price > 20-day SMA > 50-day SMA — a basic uptrend structure |
| RSI (14-day) | 40–70 — avoids chasing overbought (>70) or catching a falling knife (<40) |
| Near 20-Day High | Within 10% of the 20-day closing high — a breakout/strength read |
| Clear of 20-Day Low | More than 5% above the 20-day low — not sitting right at fresh weakness |
| Volume Confirmation | 5-day average volume rising vs. the prior 20 days |

**Honest limitation:** these are computed from CoinGecko's daily *closing prices*, not full intraday OHLC candle data. That's a genuine simplification — it's close-price technical analysis, not tick-level candle-body analysis (wicks, gaps, etc.). Good enough for a swing-trade lens on a 1-week horizon; not what a scalper or day-trader would want.

### Suggested Stop-Loss / Take-Profit
Sized from the coin's own historical weekly volatility (de-annualized from 6 months of daily price swings): stop ≈ 1.5× weekly volatility below current price, target ≈ 2× weekly volatility above. This is a mechanical sizing method tuned to how much *that specific coin* actually moves — not a prediction of where it's going. **You place these numbers yourself** on your exchange or wallet; SwingScope has no ability to submit an order. There's a "Prefill in Trade Journal" button in the Analyzer to save you retyping the numbers into your own record.

### Probability Projection (7-day, matches the max-hold horizon)
A Monte Carlo simulation (4,000 paths, geometric Brownian motion) using the coin's own historical mean daily return and volatility — not a CAPM/market-beta model like QuantFolio uses for stocks, because crypto has no clean equivalent to an equity risk premium. Shown as 5th/25th/median/75th/95th percentile outcomes plus a box-and-whisker chart, same visual language as QuantFolio's Monte Carlo tool. **Historical mean return is a genuinely noisy, backward-looking estimator** — more so than the CAPM approach even. Treat the whole projection as illustrating a plausible range under stated assumptions, never a forecast.

### Discover — browse by category
Live data from CoinGecko's own category list (`/coins/categories`), not a list I hand-typed — search "layer 1," "meme," "defi," "gaming," "AI," whatever, and it pulls the actual top coins in that category right now. This is more accurate than QuantFolio's stock-sector presets (which had to be hand-curated because there's no free sector-filter API for stocks) — CoinGecko genuinely supports category filtering, so Discover results stay current on their own.

### Trade Journal — manual, not connected to anything
Log a symbol, entry price, and your stop/target after you've placed a trade yourself, elsewhere. This is a **self-reported record** — it does not read any wallet or exchange, and logging an entry does not buy or sell anything. It exists purely to help you keep the 1-week discipline: open trades show "Day N" since entry, and anything at 7+ days gets flagged both in the journal and on the Dashboard as due for review. Closing a trade is you typing in the exit price for your own record-keeping — nothing more.

### Global search & sortable watchlist
Same conventions as QuantFolio/AssetScope: a header search box that jumps straight to the Analyzer for any coin, and clickable column headers on the Watchlist table to sort (click again to reverse).

## Rate-limit handling
Watchlist refreshes use a lighter score (skips the Monte Carlo projection) and space coins ~4.5 seconds apart; resolved coin IDs are cached after the first lookup so repeat refreshes don't re-hit CoinGecko's `/search` endpoint. The Analyzer's one-off "Analyze" always computes the full picture (checklist + stop/target + probability projection) — that's a handful of calls for one coin, not a rate-limit concern.

## Honest limitations
- **No wallet connection, no execution, ever** — see "What this is not" above. This is the core design choice, not a missing feature.
- **Technical signals use daily closes, not full OHLC candles** — a simplification, documented above.
- **Stop/target sizing is a volatility heuristic, not a support/resistance or order-book-depth analysis** — it doesn't know about actual liquidity walls, just how much the price has historically swung.
- **The Monte Carlo projection's "expected return" comes from the coin's own noisy historical average**, which is a weaker foundation than QuantFolio's CAPM approach for stocks (crypto has no clean equivalent).
- **CoinGecko's free tier can rate-limit you** — an external constraint; a paid Demo key would raise it but isn't wired in here.
- If you ever do want real execution, that has to be something you do yourself, in your own wallet/exchange, using their own official tools — not something built for you here.
