# Concepts and terminology

If you are new to electricity markets, read this before anything else.
The vocabulary is unusual, and confusion about basic terms wastes more
time than any other source of error in this case.

## Why electricity is different from other commodities

You cannot store electricity at industrial scale. Every megawatt-hour
that someone consumes must be produced at the same instant by someone,
somewhere on the connected grid. Demand and supply must match
continuously, and if they drift apart, the grid frequency moves away
from 50 Hz and the system operator has to intervene.

This physical constraint creates the structure of the electricity
market: a sequence of overlapping markets that trade electricity for
ever-shorter delivery windows as the delivery time approaches.

## The market sequence

For any given delivery hour or quarter-hour, traders can buy and sell
electricity in roughly this order:

1. **Year-ahead, month-ahead, week-ahead forward markets** — bilateral
   contracts, often outside organised exchanges. Not the focus of
   this case.

2. **Day-ahead market (DAM)** — a single daily auction that clears
   prices for every delivery period of the next day. In Europe this is
   coordinated through **SDAC** (Single Day-Ahead Coupling). Bulgaria's
   day-ahead segment participates in SDAC. Historically prices were
   cleared hourly; **from 1 October 2025 SDAC publishes 15-minute
   day-ahead prices**.

3. **Intraday market (IDM)** — opens after the day-ahead market closes
   and runs until shortly before delivery. Lets market participants
   adjust their positions as forecasts improve and reality unfolds.
   The intraday market has two flavours:

   - **Continuous intraday trading** — pairs of bids and offers match
     continuously, like a stock exchange. In Europe this is coordinated
     by **SIDC** (Single Intraday Coupling). Bulgaria has had
     **15-minute (quarter-hour, "QH") products on continuous intraday
     since 30 September 2022**.
   - **Intraday auctions (IDA1, IDA2, IDA3)** — three single
     auctions per day, scheduled at fixed times, each clearing prices
     for the remaining delivery periods.

4. **Balancing market** — runs in near-real-time, used by the system
   operator to keep the grid balanced. Not central to this case but
   relevant context.

The IBEX page that motivates this case
(<https://ibex.bg/markets/idm/idm-prices-volumes-with-qh/>) shows
**continuous intraday** prices and volumes at 15-minute resolution.

## Key terms

**Delivery period / MTU (Market Time Unit)**  
The time slot for which electricity is bought and sold. Historically
60 minutes; for Bulgarian intraday it's been 15 minutes since
September 2022; for SDAC day-ahead it's been 15 minutes since October
2025.

**Load (consumption / demand)**  
How much electricity is being consumed across the system at a given
moment. Usually reported as an average MW value over each MTU.

**Generation (supply / production)**  
How much electricity is being produced across the system. Reported
per production type (nuclear, lignite, gas, hydro, wind, solar, etc.).

**Production type ("psr_type" in ENTSO-E)**  
The technology category for a generating unit. ENTSO-E uses codes
like B14 (nuclear), B02 (lignite / brown coal), B16 (solar), B19
(wind onshore), B12 (hydro water reservoir), B11 (hydro
run-of-river). For Bulgaria the dominant types are nuclear,
lignite, hydro, and increasingly wind and solar.

**Net position**  
For a bidding zone, the net result of imports minus exports in a
given delivery period. Positive net position means the zone is a net
importer for that period.

**Cross-border physical flow**  
The actual measured electricity flow across an interconnector between
two zones, after the system operates. Different from scheduled
exchanges, which are the commercial schedules from market coupling.

**Scheduled exchange**  
The commercial schedule of imports and exports between two zones,
resulting from market coupling. The difference between scheduled and
physical flows reflects loop flows and balancing actions.

**Net Transfer Capacity (NTC)**  
The maximum power that can be transferred between two zones for a
given delivery period, taking grid security constraints into account.
Published as a forecast at day-ahead, week-ahead, month-ahead, and
year-ahead horizons. NTC limits how much one zone can export to or
import from another, and thus shapes prices in interconnected markets.

**Unavailability of generation / production units**  
Notifications of planned and forced outages of generating units,
published under the EU REMIT transparency regulation. Reduce the
available supply.

**Imbalance**  
The difference between scheduled and actual flows for a balance
responsible party. Settled at the imbalance price, which is closely
related to the actions taken by the system operator to balance the
grid.

**Bidding zone**  
A geographical area within which the electricity price is uniform.
For continental Europe most countries are a single bidding zone.
Bulgaria's code is **BG** in ENTSO-E and **10YCA-BULGARIA-R** in the
underlying EIC code system.

## Forecasting horizons in this case

**15 minutes ahead**  
Nowcasting territory. Persistence ("price next 15 min ≈ price now")
is a strong baseline. Useful for trading algorithms and real-time
operations.

**24 hours ahead**  
The horizon at which day-ahead markets clear. Most published forecasts
target this. Weather forecasts at this horizon are reasonably good,
which makes the modelling tractable.

**1 week ahead**  
Operational planning horizon. Weather forecasts deteriorate noticeably
beyond 3–4 days, so prediction quality drops. Useful for scheduling
maintenance, fuel deliveries, and ramping plans.

## "Generation" vs "supply" vs "production"

In this case we use the three terms loosely interchangeably for the
electricity-supply side of the market. Strictly:

- **Production** = what's generated by physical units
- **Generation** = same as production, ENTSO-E's preferred word
- **Supply** = production plus net imports, i.e. what's available to
  cover demand within the zone

If your team wants to be precise (recommended), pick one term and
define it explicitly in your write-up.

## Useful reading

- ENTSO-E Transparency Platform user guide:
  <https://transparency.entsoe.eu>
- IBEX market segments overview:
  <https://ibex.bg/en/markets/>
- SDAC and SIDC overviews from ENTSO-E and NEMO Committee, for the
  European market-coupling context.
