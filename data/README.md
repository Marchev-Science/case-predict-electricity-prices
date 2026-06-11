# Provided data

Seed snapshots from the three primary sources, so your team can start
modelling before any API tokens arrive. **This data goes stale** — it
is a snapshot taken at case-preparation time. Refresh it early using
the scrapers in `../tools/` (and request the ENTSO-E token on day one;
it takes a few working days).

## Folder contents

### `ibex/`

IBEX continuous-intraday statistics at 15-minute resolution, from
<https://ibex.bg/markets/idm/idm-prices-volumes-with-qh/>.

One CSV, one row per quarter-hour per delivery date:

| Column | Meaning |
| --- | --- |
| `delivery_date` | Delivery day |
| `Product` | Quarter-hour product code (`QH-YYMMDD-NN`, NN = 01…96) |
| `Weighted Average Price (EUR/MWh)` | Volume-weighted average traded price |
| `Max Price (EUR/MWh)` / `Min Price (EUR/MWh)` | Extremes over the trading session |
| `Last Price (EUR/MWh)` | Last trade before gate closure |
| `Volume (MW)` | Traded volume |

Covers ~3 months — the maximum IBEX makes public.

### `entsoe/`

One CSV per ENTSO-E dataset for the Bulgarian bidding zone, plus
`_summary.json` describing every file (rows, date range, columns).
The main families:

- `prices_day_ahead.csv` — hourly before 2025-10-01, 15-minute after
- `load_actual.csv`, `load_forecast_day_ahead.csv`
- `generation_per_type.csv` (one column per production type),
  `generation_forecast_day_ahead.csv`, `wind_solar_forecast.csv`
- `net_position.csv`
- `scheduled_exchanges_<A>_to_<B>.csv`, `physical_flows_<A>_to_<B>.csv`
  — per neighbour, per direction
- `transfer_capacity_<horizon>_<A>_to_<B>.csv` — NTC forecasts at
  day-/week-/month-/year-ahead horizons where published
- `unavailability_generation_units.csv` — event-style outage records
  (one row per notification, with start/end and capacity columns)

Not present, because ENTSO-E doesn't publish them for Bulgaria:
imbalance prices/volumes, production-unit outages, IDA intraday
auction prices, and some NTC horizons on non-EU borders.

### `weather/`

Hourly weather from Open-Meteo (ERA5 reanalysis, gap-filled with the
historical-forecast archive for the most recent days — see the
`source` column):

- `weather_Sofia.csv`, `weather_Plovdiv.csv`, `weather_Varna.csv`,
  `weather_Burgas.csv`, `weather_Ruse.csv`
- `weather_bg_total.csv` — unweighted average of the five cities
  (wind direction averaged as a speed-weighted vector mean)

Variables: `temperature_2m` (°C), `wind_speed_10m` / `wind_speed_100m`
(km/h), `wind_direction_100m` (°), `shortwave_radiation` and
`direct_normal_irradiance` (W/m²), `cloud_cover` (%), `precipitation`
(mm), `relative_humidity_2m` (%).

## Timezone warning

The three sources do not share a timezone convention out of the box:
the ENTSO-E CSVs carry timezone-aware timestamps (Europe/Sofia), the
weather CSVs are in Europe/Sofia local time, and the IBEX product
codes encode local delivery time. **Pick one canonical timezone for
your project and convert everything to it on load.** See
`../docs/data.md` for the full list of joining pitfalls (DST,
resolution mixing, publication lags).

---

# Source quick-reference

## Primary sources

| Source | URL | What's there | Tool |
| --- | --- | --- | --- |
| IBEX | <https://ibex.bg/markets/idm/idm-prices-volumes-with-qh/> | Continuous-intraday QH prices and volumes, rolling 3 months | `tools/scrape_ibex_idm_15min.py` |
| ENTSO-E | <https://transparency.entsoe.eu> | Prices, load, generation, cross-border, NTC, outages | `tools/scrape_entsoe_bulgaria.py` |
| Open-Meteo | <https://open-meteo.com> | Hourly weather, historical + recent | `tools/scrape_weather_bulgaria.py` |

## Useful secondary sources

| Source | Where | What's there | Notes |
| --- | --- | --- | --- |
| Bulgarian holidays | `pip install holidays` | National and religious holidays | Strong load signal |
| TTF gas (EU benchmark) | ICE / financial aggregators | Front-month gas price | Daily; drives thermal marginal cost |
| EU ETS carbon | <https://www.eex.com/en/market-data/environmental-markets> | EUA price | Daily |
| IRENA renewable capacity | <https://www.irena.org/Statistics> | Installed wind/solar capacity per country | Annual; for capacity-factor normalisation |
| ESO (Bulgarian TSO) | <https://www.eso.bg> | National grid reports | Partly Bulgarian-only |
| National Statistical Institute | <https://www.nsi.bg/en> | Industrial production, GDP, demographics | Monthly / quarterly |
| IBEX IDA results | <https://ibex.bg/markets/idm/intraday-auctions/> | Intraday auction outcomes | Rolling 3 months, website only |

## ENTSO-E bidding-zone codes

| Country | `entsoe-py` code | EIC code |
| --- | --- | --- |
| Bulgaria | `BG` | `10YCA-BULGARIA-R` |
| Romania | `RO` | `10YRO-TEL------P` |
| Greece | `GR` | `10YGR-HTSO-----Y` |
| Serbia | `RS` | `10YCS-SERBIATSOV` |
| North Macedonia | `MK` | `10YMK-MEPSO----8` |
| Turkey | `TR` | `10YTR-TEIAS----W` |

## ENTSO-E production-type codes (the ones that matter for Bulgaria)

| Code | Type |
| --- | --- |
| B01 | Biomass |
| B02 | Fossil brown coal / lignite |
| B04 | Fossil gas |
| B05 | Fossil hard coal |
| B10 | Hydro pumped storage |
| B11 | Hydro run-of-river and poundage |
| B12 | Hydro water reservoir |
| B14 | Nuclear |
| B16 | Solar |
| B19 | Wind onshore |

Bulgaria's generation mix is dominated by nuclear (Kozloduy), lignite,
and hydro, with fast-growing solar and a modest wind fleet.

## Getting-started checklist

1. Request an ENTSO-E API token (see `../tools/README.md`) — it takes
   a few working days, so do this first.
2. Explore the seed data in this folder; read `_summary.json` in
   `entsoe/` to see what exists.
3. Run the IBEX and weather scrapers (no token needed) to bring those
   snapshots up to date.
4. When the token arrives, refresh the ENTSO-E data.
5. Build your joined dataset: one canonical timezone, one canonical
   resolution per layer, documented schema and units.
6. Establish baselines (persistence, seasonal-naïve) per layer on a
   held-out test set you will not touch again until the end.
