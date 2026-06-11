# Data sources

This document is your map of what data exists, where it lives, what
the provided tools fetch, and which traps to watch out for.

## The three primary sources

### IBEX — Independent Bulgarian Energy Exchange

**URL:** <https://ibex.bg>  
**Provided tool:** `tools/scrape_ibex_idm_15min.py`  
**Seed data:** `data/ibex/`

IBEX operates the Bulgarian day-ahead and intraday electricity
exchanges. The page you'll be scraping is:

<https://ibex.bg/markets/idm/idm-prices-volumes-with-qh/>

This page shows **continuous intraday (SIDC) statistics at 15-minute
resolution**: weighted-average price, max price, min price, last price,
and traded volume for each quarter-hour of the chosen delivery date.

**Important limitations:**

- **Data is limited to the past three months.** This is enforced by
  the IBEX backend, not just the user interface. There is no public
  way to get older IBEX continuous-intraday QH data without a paid
  Nord Pool API contract.
- The page is JavaScript-rendered and gated by an anti-bot challenge.
  The provided scraper passes the challenge once with a headless
  browser, then makes plain HTTP requests for the actual data.
- The HTML response contains **both** the 60-minute and the 15-minute
  tables; the front-end toggle just switches which is visible. The
  scraper extracts the 15-minute one.

If you need historical IBEX-style continuous-intraday statistics
beyond 3 months, **they aren't publicly available**. The closest
public substitute for an intraday price signal is the day-ahead price
series from ENTSO-E (long history, 15-minute since October 2025) —
a different market, but strongly related.

### ENTSO-E — Transparency Platform

**URL:** <https://transparency.entsoe.eu>  
**Provided tool:** `tools/scrape_entsoe_bulgaria.py`  
**Seed data:** `data/entsoe/`

The ENTSO-E Transparency Platform is the single most important data
source for European electricity markets. It publishes, for every
bidding zone in Europe:

- Day-ahead prices (hourly historically, 15-min from October 2025)
- Intraday auction prices (IDA1/2/3) for many zones — **but not for
  Bulgaria**: as of mid-2026 the API returns no per-zone IDA prices
  for BG; IBEX publishes its IDA results only on its own website
- Load: actual and day-ahead forecast
- Generation per production type (actual and day-ahead forecast)
- Wind and solar generation forecasts
- Net position
- Cross-border physical flows (one series per neighbour, per direction)
- Scheduled exchanges (one series per neighbour, per direction)
- Net Transfer Capacity (NTC) at four horizons (day-, week-, month-,
  year-ahead)
- Imbalance prices and volumes
- Unavailability of generation and production units (planned and
  forced outages)

**To use it:** register a free account at the URL above, then email
`transparency@entsoe.eu` with the subject `Restful API access` and
the body `I want to request access to the Restful API`. Reply takes
a few working days. Once you have the token, export it as
`ENTSOE_API_KEY` and run the provided scraper.

**Bulgaria coverage:** very good for the core series. Verified missing
for BG: imbalance prices/volumes, production-unit outages
(generation-unit outages ARE published), IDA prices, and week-/year-
ahead NTC on some non-EU borders. The scraper logs and skips these
gracefully and records them in its `_summary.json`.

**Bulgaria's electrical neighbours:** Romania (RO), Greece (GR),
Serbia (RS), North Macedonia (MK), Turkey (TR). The scraper queries
all five for cross-border data.

**Historical depth:** most series go back many years. 15-minute
resolution for intraday is from ~September 2022. Day-ahead resolution
is hourly before October 2025 and 15-min after.

### Open-Meteo — Weather data

**URL:** <https://open-meteo.com>  
**Provided tool:** `tools/scrape_weather_bulgaria.py`  
**Seed data:** `data/weather/`

Free, no token, hourly weather data sourced from ERA5 reanalysis
(Copernicus / ECMWF). The provided scraper fetches:

- Five major cities: Sofia, Plovdiv, Varna, Burgas, Ruse
- A simple country-average series across the five
- Variables: temperature at 2m, wind speed at 10m and 100m, wind
  direction at 100m, shortwave radiation (GHI), direct normal
  irradiance, cloud cover, precipitation, relative humidity at 2m.

**One gotcha worth knowing:** the ERA5 archive lags real time by
roughly five days. The provided scraper stitches in Open-Meteo's
"historical forecast" archive (high-resolution model output) for the
last few days to fill the gap. There's a `source` column in the output
so you can see which rows came from where. The two sources differ
slightly — ERA5 assimilates observations, the forecast archive is
pure model output — usually not enough to matter, but worth knowing
if your model behaves oddly on the most recent days.

## Secondary sources worth considering

The case explicitly invites you to add data you can argue for. Some
possibilities, in rough order of usefulness:

- **Bulgarian public holidays and school calendar** — strong load
  signal. The `holidays` Python library covers Bulgarian holidays.
- **Neighbour-country prices and load** — Bulgaria is strongly
  interconnected with Romania, Greece, Serbia, North Macedonia, and
  Turkey. ENTSO-E publishes their data too. Spillover effects on price
  are real.
- **Fuel prices** — TTF natural gas, EU ETS carbon, API2/API4 coal.
  Day-ahead clearing prices are sensitive to fuel costs because they
  set the marginal generator's cost. Free daily data is available from
  EEX, ICE, and various financial data aggregators.
- **Solar / wind installed capacity** — published in IRENA's renewable
  capacity statistics and in ENTSO-E's installed capacity figures.
  Useful for normalising production to capacity-factor terms.
- **Macroeconomic indicators** — industrial production index, GDP
  proxies. Slower-moving but explains long-run load trends.
- **News / events** — large industrial customer outages, political
  events affecting cross-border trade. Hard to operationalise but
  occasionally important.

## Data hygiene — the silent killer

Joining electricity-market data across sources is the single most
common place where projects in this domain go wrong. The pitfalls:

1. **Timezones.** ENTSO-E publishes some series in UTC, the
   `entsoe-py` library converts to local time for you, IBEX is in
   Europe/Sofia, Open-Meteo can be in either depending on the request
   parameter. **Pick one canonical timezone for your project (UTC is
   safest) and convert everything to it on load.**

2. **Daylight saving time.** Bulgaria observes DST. On the spring
   transition you'll have a missing hour; on the autumn one a doubled
   hour. ENTSO-E handles this correctly in its XML, but some tools
   strip the DST flag and the doubled hour looks like duplicate
   timestamps. Inspect the spring/autumn DST days specifically.

3. **Resolution mixing.** Don't naively join a 15-minute series with
   a 60-minute one. Decide whether to upsample the hourly to
   15-minute (forward-fill or interpolate) or downsample the 15-min
   to hourly (average or sum, depending on the variable — *average*
   for power [MW], *sum* for energy [MWh]).

4. **Publication lag.** ENTSO-E publishes some series with a delay
   of minutes (load), hours (prices), or days (imbalance). If your
   "as-of" timestamp is wrong, you may accidentally use information
   that wouldn't have been available at forecast time. This is the
   #1 cause of suspiciously-good forecasts.

5. **Revisions.** ENTSO-E republishes some series as data is
   corrected. The version of a series you fetched yesterday may
   differ from today's version for the same historical period. For
   model training that's mostly fine; for honest backtesting it's a
   problem.

6. **Units.** Power vs. energy, MW vs. MWh, °C vs. K, EUR/MWh vs.
   BGN/MWh. Document units explicitly in your dataset schema.

7. **Missing data.** Some series have gaps. Decide on a policy
   (interpolate? forward-fill? mark missing and skip?) and apply it
   consistently.

## What "raw data" means in this case

For your work, *raw data* = whatever lands directly from a scraper or
download, before any cleaning. **Keep it.** Store it separately from
your processed datasets, never overwrite it, and treat your cleaning
pipeline as code that turns raw into processed. If someone (including
future you) needs to question a number in your final report, you must
be able to trace it back to the raw cell it came from.

A small note from the case authors: the seed data provided in
`data/` is itself a snapshot of these sources at a point in time, and
it will go stale if you leave it sitting. Refresh from the live
sources on day one.
