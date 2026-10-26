# StockenShares

**A global paper trading simulator. Every user gets $10,000 in virtual cash and trades real, live priced assets across world markets. No brokerage account, no email verification, no payment info. Live app: <https://stockenshares.onrender.com/>**

## Why it exists

Most paper trading apps fail on one of three things:

- They are paywalled or require a real brokerage account.
- They only support US stocks.
- They simulate prices instead of using real market data.

StockenShares fixes all three. Trade `RELIANCE.NS` (NSE), `AAPL` (NASDAQ), `BTC-USD`, gold futures, `SAP.DE`, Toyota (Tokyo), and more, all at real prices, the moment you sign up. 37 tradable symbols across 9 markets, all free.

## Features

- **Global coverage**: 9 markets, NSE, NASDAQ, NYSE, LSE, Frankfurt, Tokyo, Hong Kong, ASX, and TSX, with BSE Sensex quoted as an index.
- **Natural language search**: type `reliance` or `crude oil` and Groq's LLaMA 3.3 resolves the exact ticker. If the AI call fails, the server falls back to suffix matching so the app never hard stops.
- **Multi currency**: 17 live exchange rates refresh from Yahoo Finance and are cached for five minutes. All internal math runs in USD, with fallback rates baked in for API outages.
- **Live charts**: TradingView widgets for US tickers, a custom Lightweight Charts pipeline with 20, 50, and 200 day moving average overlays for everything else (Yahoo's symbol formats do not fit TradingView outside the US).
- **Parallel ticker tape**: 14 live prices refresh every 90 seconds through a thread pool, so the front page shows movement without waiting on sequential requests.
- **Brokerage style accounting**: buys record a real weighted average cost basis, so gains and losses compute like a real brokerage would.
- **Live leaderboard**: every portfolio revalues against current market prices on each page load, and users rank by real time return. Sessions last 30 days, no email walls.
- **6 themes**: Raspberry, Matrix (with canvas rain animation), Terminal, Light, Pink, and Red.

## How a trade works

1. You search "reliance" and the AI resolves it to `RELIANCE.NS`.
2. The server fetches the live price and the INR per USD rate, then quotes you in your currency.
3. You confirm the buy. The server re-fetches the live rate (no stale sellers), deducts the USD equivalent from your balance, and stores the holding plus trade history in MongoDB.
4. As prices move, returns, gains, and the leaderboard all recompute against live quotes on every load, so nothing fakes its way upward.

## The stack

| Layer | Technology |
|---|---|
| Backend | Flask, one file (`app.py`) |
| Frontend | Vanilla JS in one template, no framework |
| Database | MongoDB Atlas, free tier |
| Market data | yfinance (quotes and FX), Finnhub |
| AI ticker resolution | Groq (LLaMA 3.3 70B) |
| Charts | TradingView widget + TradingView Lightweight Charts |
| Session store | Flask cookies, 30 day lifetime |
| Hosting | Render free tier |

## Setup

```bash
git clone https://github.com/nithilan-manivannan/StockenShares
cd StockenShares
pip install -r requirements.txt
```

Create a `.env` (or set these in your host dashboard):

```
GROQ_API_KEY=
FINNHUB_API_KEY=
MONGO_URI=
SECRET_KEY=
```

Then run it:

```bash
python app.py
```

Deploy as is to Render with the `Procfile`, or to any host that runs Flask apps.

## What happens under the hood

- Quotes and FX rates come straight from Yahoo Finance through yfinance; FX uses the proper normalizations for EUR, GBP, AUD, CAD, and CHF pairs, whose Yahoo tickers quote USD per unit rather than units per USD.
- Buy and sell flows re-fetch live prices at execution time, so the price you see is the price you pay.
- All holdings store cost basis per share, and returns derive from that rather than naive shares times price.
- Currency conversion is applied once at the trade boundary, keeping the rest of the app in USD units, which keeps multi currency math honest and simple.

## Known limitations

Honest to list them:

- Passwords are stored as plaintext. Fine for a demo, not for production.
- The leaderboard re-fetches live quotes per user per page load; it stops being practical past a few hundred users without a caching layer.
- Render's free tier cold-starts after 15 minutes idle, so the first page load can take 20 to 40 seconds.
- TradingView charts only work for US tickers. Non-US symbols rely on the custom chart pipeline instead (by design).
- Flask sessions are cookie based and will not survive multi instance load balancing without Redis.
- Market data depends on Yahoo's availability and their symbol universe, so some exotic instruments may quote stale or nothing at all.

## Repository structure

```
StockenShares/
├── app.py                # the whole backend: auth, quotes, trades, leaderboard
├── templates/index.html  # the whole frontend: dashboard, search, charts
├── requirements.txt      # Flask, requests, yfinance, OpenAI, pymongo
├── Procfile              # Render entrypoint
└── README.md
```
