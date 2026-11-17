# GTA VI launch-day Twitch CCU forecaster — v24

[![make.fun · gta6-twitch-ccu-forecast-01](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/AndryXSad/gta6-twitch-ccu-forecast-01-v24/rank.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/AndryXSad/gta6-twitch-ccu-forecast-01-v24/reward.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)

A make.fun `gta6-twitch-ccu-forecast-01` entry. It forecasts, for each of
hours 2–24 of Grand Theft Auto VI's release day (2026-11-19 UTC), five
quantiles (5%, 25%, 50%, 75%, 95%) of the concurrent Twitch viewers in the
"Grand Theft Auto VI" category, from the first hour's print and the
launch-day curves of past big game launches shipped inside each case.

## How it works

`main.py` (Python 3.13, standard library only, no network) builds four
member predictors in log space from the 37 comparable past launches:

1. **Y5** — tier-filtered per-hour OLS on `[lnV1, g12]` blended
   (0.5/0.2/0.3) with console-only and all-launch one-factor shape views.
   `g12` is the "unlock momentum" feature: lnV1 minus log viewers 12 hours
   before release day.
2. **Z0** — the same construction with the naive early-access filter
   (launches whose notes contain "early access" are excluded).
3. **Y1** — the same construction, naive filter, features `[lnV1, lnPREM]`.
4. **Clone** — whole-library shape views, sample-sd spreads, and a
   weighted-mean blend (0.6 OLS / 0.2 console-shape / 0.2 all-shape) whose
   spread is 0.81× the weighted mean of the view spreads.

The **log-median is chosen per hour band** — the new lever this round,
because the median is the bottleneck (best median-only practice ever
before this round was 0.1975):

| band | hours | median source | offset |
| ---- | ----- | ------------- | ------ |
| early | 2–6 | median of Y5 / Z0 / Y1 log-medians | +0.00 |
| mid | 7–14 | half-blend 0.5·Y5 + 0.5·clone | −0.01 |
| late | 15–24 | median of Y5 / Z0 / Y1 log-medians | +0.02 |

The **log-spread** is the 3-way weighted mean of the member spreads
(45% Y5 + 35% Z0 + 20% clone), shaped per band with separate
lower/upper scales: early (0.90, 1.10), mid (1.10, 0.80),
late (0.95, 0.85). Quantiles are lognormal.

## Results

Direct-graded with the exact RULES scorer on the four practice cases
("Average viewers" answers): **practice pinball loss 0.125709**
(p00 0.1226 / p01 0.1316 / p02 0.0871 / p03 0.1615),
LOO-large **0.210426**, LOO-mega **0.212453**. That beats v23
(0.126869 / 0.210446 / 0.212649) on practice and on LOO-mega, with
LOO-large essentially tied (−0.000020). Median-only practice is
**0.194455** — the first median improvement in 24 rounds (Y5 held
0.1975). See `DESIGN.md` for the full reasoning record and the honest
caveats. Not submitted: the owner's rule is to submit only a candidate
at practice ≤ 0.100.

## Run it

```sh
MAKEFUN_INPUT_DIR=<dir with *.in> MAKEFUN_OUTPUT_DIR=<dir> python3 main.py
```

Each `*.in` case produces a matching `*.out`:
`{"hours": [{"hour": h, "q": [q05, q25, q50, q75, q95]}]}` for h = 2..24.
