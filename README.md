# Live Gold Rates

A single static page showing the live gold rate per gram across 31 countries.
No backend, no build step, no API keys.

## How it works

Gold has one global spot price (XAU/USD). Every country's rate is derived:

```
rate_per_gram = (spot_USD_per_oz / 31.1035) x FX(USD->local) x purity x (1 + tax%)
```

24K = 999 fine. 22K = 91.67%.

### Data sources

| Purpose  | Endpoint                            | Freshness | Key |
|----------|-------------------------------------|-----------|-----|
| Spot XAU | `api.gold-api.com/price/XAU`        | live      | no  |
| FX       | `open.er-api.com/v6/latest/USD`     | daily     | no  |
| Fallback | jsDelivr `currency-api/xau.json`    | daily     | no  |

Both primary endpoints send `Access-Control-Allow-Origin: *`, so the browser
calls them directly. If the primary feeds fail the page falls back to the
jsDelivr source; if everything is unreachable it shows the last rates that
browser loaded, labelled as cached. Each degraded state shows a banner.

## Editing the data

Everything you'd want to change lives in `countries.json`:

```json
{ "name": "Dubai (UAE)", "cc": "AE", "currency": "AED", "taxPct": 10.00 }
```

- `taxPct` — your tax + clearance markup, as a percent. `0` shows a dash.
- `currency` — must be a valid ISO 4217 code present in the FX feed.
- `cc` — ISO 3166-1 alpha-2, used only to draw the flag.

Add or remove countries by editing that array. No code changes needed.

## Deploying (GitHub Pages)

Push to `main`, then in the repo: **Settings -> Pages -> Source: Deploy from a
branch -> `main` / `root`**. The site is live at
`https://albatrosshub.github.io/live_rate/` a minute later.

## Local preview

```
python3 -m http.server 8912
```

Then open <http://localhost:8912>. Opening `index.html` as a `file://` URL will
not work — `countries.json` is fetched, which requires a real origin.

## Caveats

- FX updates once a day; gold spot is live. Intraday FX needs a paid key.
- These are spot-derived reference rates, not dealer quotes. Real retail prices
  also carry making charges and dealer premiums beyond the tax percentage.
- `Data - Sheet1.csv` is the original input, kept for reference only. The page
  does not read it — `countries.json` is the source of truth.
