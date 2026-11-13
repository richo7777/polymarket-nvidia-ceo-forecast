# Polymarket NVIDIA CEO Price Forecaster

Entry for make.fun's `polymarket-nvidia-ceo-forecast-01` — forecast the Yes
price of every market in Polymarket's "Next CEO of NVIDIA?" event at every
minute of the 4 hours after each checkpoint, as five quantiles
(0.05, 0.25, 0.5, 0.75, 0.95) per minute.

## Method

For each market, a random-walk quantile forecast around its price at the
checkpoint:

```
q_tau(h) = clamp(x + z_tau * sigma * h**0.35, 0, 1)
```

- `x` — the market's price at the checkpoint.
- `z_tau` — the standard normal's quantile at each level.
- `sigma` — the market's own realized 1-minute volatility: the standard
  deviation of its 1-minute price changes over the last 120 minutes,
  floored at 0.0003 and capped at 0.015.
- `h**0.35` — the spread grows slower than a random walk's sqrt(h),
  because these prices mean-revert around a level. Tuned on the practice
  cases; 0.35 was the optimum.
- No drift — Polymarket prices are martingales at this horizon; every
  drift variant tested made the practice loss worse.

## Why it beats the house baseline

The baseline uses one fixed sigma (0.0025) and sqrt(h) for every market.
Dead markets barely move (true 1-min sd ~0.0001) while the live ones move
10–50x more, so the fixed sigma is far too wide for the former and far too
narrow for the latter. A 120-minute window tracks the current regime (the
market is settling, so longer windows are contaminated by the early
discovery phase when prices fell from 0.485).

## Local practice score

Pinball loss over the 6 practice cases (lower is better), scored against
Polymarket's public price history:

| | loss |
|---|---|
| house baseline | 0.347 |
| this entry | **0.254** |

It beats the baseline on every one of the 6 practice cases. Deterministic
(no randomness, fixed constants), ~0.05 s per case. Python 3.13, standard
library only, no network.

## License

MIT. See LICENSE.
