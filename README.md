# ORR Dashboard

A single-page, zero-build dashboard for the **Militia Long/Short Equity ETF (ORR)**.
It pulls the fund's published holdings from [militiaetf.com](https://militiaetf.com/), prices every
position through Yahoo Finance, and shows how each book is contributing to the fund today.

Live: https://cbay460704-web.github.io/orr-dashboard/

## Features

- **Long and short books** side by side, each with weight, live price, daily change and YTD change.
  Changes for shorts are shown from the fund's perspective (a falling short is a gain).
- **Summary bar**: long-book P&L, short-book P&L, net attribution (in bps of NAV), winners/losers,
  top mover, net long exposure and gross exposure (hover for the long/short split and L/S ratio).
- **ORR quote** in the header with daily change and YTD.
- **Filter box** that narrows both books by ticker or name (`/` to focus, `Esc` to clear).
- **Sortable columns**: click any column header to sort both books; click again to flip direction.
  The chosen sort is remembered.
- **Market-status badge** (pre-market, open, after hours, closed) and a countdown to the next
  auto-refresh. Refresh cadence: 60 s while the market is open, 2 min pre/after hours, 5 min closed.
- **Pause/resume** auto-refresh, and `R` to refresh immediately.
- **Dark mode** that follows the system setting, with a manual toggle that is remembered.
- **Yahoo Finance links** on every ticker, and a contribution-to-fund tooltip (in bps) on each change cell.
- **CSV export** of the full holdings table including prices, changes, YTD and contribution.
- **Offline resilience**: the last good holdings list is cached in `localStorage` and used if
  militiaetf.com is unreachable; YTD data is cached per session.
- Works on phones: the header wraps and the table columns tighten below 600 px.

## How it works

Everything lives in [index.html](index.html). There is no build step and no dependencies beyond
Google Fonts.

1. **Holdings** are scraped from the holdings table on militiaetf.com. Tickers with an exchange code
   (for example `7453 JP`) are mapped to Yahoo symbols via `YAHOO_SUFFIX_MAP`.
2. **Quotes** come from Yahoo's `v7/finance/quote` batch endpoint, with a per-symbol fallback to the
   `v8/finance/chart` endpoint. International names use the chart endpoint with a 5-day range.
3. **YTD** is computed against the last close of the prior year, fetched in batches of 10.
4. All requests go through a **CORS proxy**. The primary is a Cloudflare Worker (`WORKER_URL` in the
   script); if it fails, a list of public proxies is tried in order.

The debug console (the `DBG` button) shows every fetch and fallback.

## Deploying

The site is served by GitHub Pages from the `main` branch root, so pushing to `main` deploys.

To use your own proxy, deploy a Cloudflare Worker that forwards `?url=` to the target and adds
`Access-Control-Allow-Origin: *`, then set `WORKER_URL` at the top of the script in `index.html`.

## Local development

Any static server works:

```bash
python3 -m http.server 8791
```

Then open http://localhost:8791/. Note that Yahoo and militiaetf.com are fetched live through the
proxy, so you need network access.

## Disclaimer

For informational purposes only. Not investment advice. Prices are delayed roughly 15 minutes.
