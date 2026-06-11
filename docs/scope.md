# Scope

What this case asks for, what it doesn't, and what's left open.

## What you must deliver

1. A **consumption forecast model** (Layer 1) for Bulgarian
   electricity load at 15-minute, 24-hour, and 1-week horizons.
2. A **supply forecast model** (Layer 2) for Bulgarian electricity
   generation at the same horizons.
3. An **attempt at a price forecast** (Layer 3) at the same horizons,
   using the outputs of Layers 1 and 2 along with historical prices
   and any other relevant features.
4. A **working data pipeline** that produces a coherent joined
   dataset from the raw sources, with documented timezones,
   resolutions, and units.
5. A **clear comparison against baselines** (persistence,
   seasonal-naïve) on a held-out test set, with multiple metrics.
6. A **presentation** of what you did, what worked, what didn't, and
   what you'd do next.

"Attempt at" in point 3 is deliberate. A serious, honest attempt at
Layer 3 that doesn't outperform baselines is a perfectly acceptable
outcome and a more interesting story than a model that mysteriously
beats persistence by a large margin (see the warning in
`docs/practices.md`).

## What you're free to redefine

- **The exact set of features** you build into each layer. The
  README lists *minimum* required inputs; everything else is your
  choice to defend.
- **The metric you optimise.** Different metrics encode different
  values. Whatever you pick, justify it and report alongside the
  common ones (MAE, RMSE, MAPE).
- **The time window of your training data.** Bulgaria's intraday
  market structure changed in late 2022, the energy crisis shifted
  price regimes in 2022, day-ahead went 15-min in October 2025. Pick
  a window that suits your modelling approach and explain why.
- **The level of granularity you model at.** You can model total
  generation, or generation per type and sum. You can model the
  country load, or model it as a sum of regional sub-loads if you
  have the data.
- **The forecasting paradigm.** Point forecasts, probabilistic
  forecasts, scenario ensembles — pick what makes sense and what
  your team can execute.
- **The role split** within the team (the README's suggested split
  is a default, not a requirement).

## What's explicitly out of scope

- **Real-time deployment.** You're not building a production trading
  system. Your code should be reproducible and reasonably engineered,
  but it doesn't need to be a service.
- **A paid data feed** (Nord Pool, EPEX, EEX intraday products,
  Bloomberg). The case is designed to be solvable with free sources.
- **Forecasting balancing market prices.** Out of scope; conceptually
  related but a different mechanism.
- **Forecasting individual generator output**, except as a step
  within the supply model if you choose.
- **Building a recommender for which delivery period to trade.** You
  forecast; trading strategy is a different problem.

## Optional directions (for teams who finish the layers and want more)

If you complete the three layers and the supporting infrastructure
with time left, here are directions the case authors find interesting.
None are required. Pick one — picking three and doing none well is a
classic trap.

- **Probabilistic forecasts and trading-cost-relevant metrics.**
  Convert your point forecast into a distributional forecast (quantile
  regression, conformal prediction, ensemble of seeds). Compute
  pinball loss. Discuss which forecast is "better" depending on whether
  the cost of error is symmetric.
- **Cross-border effects.** Bulgaria is interconnected with five
  neighbours. Model how their prices, loads, and physical flows
  influence Bulgarian prices. Does adding Greek prices to your price
  model help? Does Romanian wind generation affect Bulgarian
  intraday prices?
- **Regime detection.** Use a hidden Markov model or change-point
  detection to identify regime shifts in price (calm vs. volatile
  periods). Train separate models per regime, or feed the regime
  indicator as a feature.
- **Causal exploration vs. predictive modelling.** Predictive features
  may not be causal. Pick one or two relationships you find
  interesting (e.g. "do outages of unit X *cause* price spikes?")
  and explore them with proper causal-inference tools
  (instrumental variables, regression discontinuity, synthetic
  control).
- **Adversarial backtesting.** Pretend you're a sceptical reviewer
  trying to break your own model. What's the most embarrassing
  failure mode you can find? Examples: peak days, holidays, the
  spring/autumn DST transitions, days with extreme weather, days
  with major outages.
- **A graphical model of the system.** A structural model where load,
  generation, cross-border flows, and price are connected by causal
  arrows, fit with the data — Bayesian network, structural VAR, or
  a graph neural network if you're feeling ambitious. The school
  covers graph neural networks; this is a natural application.
- **Compare your supply forecast to ENTSO-E's published one.**
  ENTSO-E publishes a day-ahead generation forecast for Bulgaria.
  Is your forecast better? Where? When?
- **Energy-only model vs. capacity-aware model.** Build one model
  that only uses energy variables (load, generation, weather, prices)
  and one that adds capacity-side data (installed capacity, outages,
  NTC). How much does the second help?
- **Long-term retrospective.** ENTSO-E goes back many years. Build a
  model of *hourly* load and price over the last 5+ years (Bulgarian
  day-ahead prices were hourly until October 2025; quarter-hour
  intraday products exist since September 2022). The hourly world is
  bigger and lets you study longer-run effects (the 2022 energy
  crisis, Russia-Ukraine impacts, post-COVID recovery).

## What success looks like

There is no rubric and no grading scheme attached to this case. As a
loose definition: at the end of the work, the team should be able to
hold a 20-minute conversation with a sceptical electricity-market
analyst about your methodology, your data choices, your baselines,
your evaluation, and your honest assessment of what your model can
and cannot do. If you can hold that conversation, you've succeeded.
