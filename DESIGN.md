# DESIGN.md — gta6-twitch-ccu-forecast-01 v24

## The bar
- v2 (verified, direct-graded): practice **0.128297** (p00 0.1215 / p01
  0.1296 / p02 0.0907 / p03 0.1713), LOO-large 0.2113, LOO-mega 0.2137.
- v22 (direct-graded): practice **0.127258**, LOO-large 0.210621,
  LOO-mega 0.212838 — the first version in 22 rounds to beat v2 on
  BOTH practice and LOO.
- v23 (direct-graded): practice **0.126869** (p00 0.1227 / p01 0.1323 /
  p02 0.0896 / p03 0.1628), LOO-large **0.210446**, LOO-mega
  **0.212649** — a two-band refinement of v22's per-band asymmetric
  triple, found outside the systematic grid.
- House baseline: 0.387077. Live bar at 17 Nov ~17:27 WIB: #1 0.0975,
  top-10 ≈ 0.115, top-20 ≈ 0.122, ~3,127 entries. Prizes pay top 20.
- Owner's rule: keep improving until a candidate hits practice ≤ 0.100,
  then submit it. This version does not reach 0.100 — it is the third
  version ever to beat v2 on both gates, and the first to move the
  MEDIAN.

## What v24 is
The v23 construction (3-way weighted-mean blend for the spread, v23's
per-band asymmetric scales) with a **per-band median source** and
**per-band median offsets**:

| band | hours | median source | offset | spread scales |
| ---- | ----- | ------------- | ------ | ------------- |
| early | 2–6 | median(Y5, Z0, Y1) | +0.00 | (0.90, 1.10) |
| mid | 7–14 | 0.5·Y5 + 0.5·clone | −0.01 | (1.10, 0.80) |
| late | 15–24 | median(Y5, Z0, Y1) | +0.02 | (0.95, 0.85) |

Spread (all bands): 0.45·sig(Y5) + 0.35·sig(Z0) + 0.20·sig(clone).

Direct-graded practice **0.125709** (p00 0.1226 / p01 0.1316 / p02
0.0871 / p03 0.1615); LOO-large **0.210426**, LOO-mega **0.212649** →
**0.212453**. Median-only practice **0.194455** (LOO-large 0.322301)
— the first median-only practice improvement in 24 rounds; Y5 held
the record at 0.197518 since v4, and the best median-only LOO-large
remains the ensemble's 0.322272 (v24 is +0.000029 behind it there).

## How it was found
All experiments rescore v23's cached member rows (851 LOO rows, 92
practice rows; verified byte-identical to v22's rows where they
overlap), so every number below is a pure rescoring, and the three
direct grades confirm the packaged program reproduces them exactly.

**Idea 1 — systematic grid around the v23 cell: DEAD.** One band at
a time, each lo/hi ±0.05 from (0.90,1.10)/(1.10,0.80)/(0.95,0.85)
— 16 cells, **0 pass** the v23 gates. Closest: early (0.90,1.05)
practice 0.126881, LOO-large 0.210506 (fails LOO by +0.000060).
The systematic neighbourhood is exhausted; the v23 cell is a local
optimum of the one-band plane, and the two-band combinations had
nothing to combine (0 passing one-band cells).

**Idea 2 — new median levers: READY-BETTER.** Four sub-levers,
coordinate-wise, LOO-first:
- (2a) Per-band blend weights: 2 of 21 cells pass. Best:
  mid-band weights (0.55, 0.25, 0.20) → 0.126820 / 0.210438 /
  0.212640; early-band (0.35, 0.45, 0.20) → 0.126825 / 0.210386 /
  0.212579.
- (2b) Per-band median offsets: 2 of 18 pass. Mid −0.02 →
  0.126670 / 0.210046 / 0.212354; mid −0.04 → 0.126824 / 0.209793 /
  0.212216. Late +0.02 alone gives practice 0.125772 but fails LOO
  (0.210634) — the late band's medians run LOW on practice and
  the shift is worth ~0.0009 of practice.
- (2c) Per-band median source (3^3 = 26 cells, one passing):
  ('ens', 'hb', 'ens') → **0.126614 / 0.209835 / 0.212161**. The
  ensemble median wins early/late (it trims the member that is
  off), the half-blend wins mid (clone's median is the best
  mid-day anchor). Best LOO in the grid: ('hb','ens','ens')
  0.208073 — but practice 0.128560, far off.
- (2d) Hour-ramp clone weight: 0 of 4 pass. Positive slopes help
  practice (slope +0.3 → 0.126044) and hurt LOO monotonically
  (0.212361) — the same trade-off, no free lunch.
- Round 2 (combining the passing levers, 21 cells, 18 pass):
  2c + mid offset −0.01 → 0.126551 / 0.209627 / 0.212006.
- Round 3 (mid × late offsets, 18 cells, 9 pass): 2c + mid −0.01
  + late +0.02 → **0.125709 / 0.210426 / 0.212453**, the packaged
  cell. The late +0.02 gain is broad (p01 0.1323→0.1316,
  p02 0.0896→0.0871, p03 0.1628→0.1615) and the mid −0.01 buys
  back exactly the LOO-large the late shift costs.
- Round 4 (fine grid around the winner + late-scale re-tune,
  14 cells, 8 pass): nothing beats the packaged cell on
  practice; the flattest neighbour (mid −0.015 / late +0.02,
  0.125720 / 0.210324 / 0.212377) has 6× more LOO-large
  headroom for +0.000011 of practice — direct-graded as the
  alternate below. Late-scale re-tune at the new offsets:
  (1.00, 0.85) gives LOO-large 0.209602 but practice 0.126122.

**Idea 3 — per-band asymmetry on half-blend / ensemble at base
scale 1.0 (v23's base, not 0.81): DEAD.** Full 729-cell grids,
0 pass the v23 gates in all four variants (half-blend and
ensemble, base 1.0 and 0.81). Best practice in any grid:
half-blend ×0.81 at 0.124219 — but LOO-large 0.226167, hopeless.
The corner v22/v23 found does not transfer to the other
median families: their LOO failure is in the median, not the
spread shape.

## Direct grades (3 variants graded, max 3)
| candidate | practice (direct) | per-case | LOO-large | LOO-mega |
|---|---|---|---|---|
| **v24 (packaged)** mid −0.01 / late +0.02 | **0.125709** | .1226/.1316/.0871/.1615 | 0.210426 | 0.212453 |
| alternate mid −0.015 / late +0.02 | **0.125720** | .1226/.1324/.0869/.1609 | 0.210324 | 0.212377 |
| 2c base, no offsets | **0.126614** | .1238/.1332/.0879/.1616 | 0.209835 | 0.212161 |

All three direct grades match their harness numbers exactly
(per-case identical to 6 decimals). The packaged program was
re-graded after its final edit and reproduces 0.125709 exactly.

## Why 0.100 still held (twenty-fourth round)
The median moved for the first time — median-only practice
0.1975 → 0.1945 — but the gap to 0.100 is still ~0.095 of
median loss, and the per-hour OLS residual sd is still
0.35–0.45 log. The late-band +0.02 offset is the single
biggest honest practice gain since v22 (−0.0009), and it cost
LOO-large almost exactly what the mid −0.01 offset repays:
the frontier is still a frontier. Every median family,
blend, spread construction, and now per-band median source
has been built or measured; the remaining moves are
sub-0.001 refinements of the same corner.

## Honest caveat (read before trusting the number)
The band parameters (sources, offsets, scales) were chosen on
LOO-large AND on practice, so NEITHER gate is a clean holdout
for the packaged cell — the LOO-large margin (−0.000020) is
essentially zero and optimistically biased by selection, and
the practice margin (−0.001160 vs v23) is the cleaner but
still selected evidence. The neighbourhood holds together
(18 of 21 round-2 cells and 9 of 18 round-3 cells pass the
v23 gates; both direct-graded neighbours beat v23 on all
three numbers), and the per-case gains are broad rather than
one-case luck — but the final is a new launch in a new regime
(lnV1 ≈ 14+, beyond the library max 13.68), and practice is
only 4 launches. Treat this as the best verified fallback,
not a guarantee. The alternate (mid −0.015) is the more
LOO-robust packaging of the same idea if the LOO margin
matters more than 0.000011 of practice.

## Files
- `work/h24.py` — fresh harness (row rescoring, candidate
  builders, direct run_program, calibration: v23 cell
  0.126869 / 0.210446 / 0.212649 reproduced exactly)
- `work/exp24.py` — ideas 1–3 (3,001 cells) +
  `work/exp24-results.json`
- `work/exp24b.py` — round 2: combining passing median levers
  (21 cells) + `work/exp24b-results.json`
- `work/exp24c.py` — round 3: mid × late offsets (18 cells) +
  `work/exp24c-results.json`
- `work/exp24d.py` — round 4: fine grid + late-scale re-tune
  (14 cells) + `work/exp24d-results.json`
- `work/rows24.json`, `work/prows24.json` — cached LOO/practice
  member rows (851 + 92 rows; copied from v23's rows23/prows23)
- `work/sully-p00..p03.json` — SullyGnome chart data (copied data)
- `work/main24-alt.py` — the mid −0.015 alternate, direct-graded
  0.125720; `work/main24-base.py` — the 2c base with no offsets,
  direct-graded 0.126614
- Kit: `kit/` (RULES.txt read fully; kit.zip = downloaded kit)
- `repo/` — packaged entry: main.py (direct-graded 0.125709 after
  the final edit, 2 runs byte-identical, ~0.17 s/case),
  README.md (3 live badges), LICENSE (MIT + SPDX), DESIGN.md,
  .makefun/owner.json. Commit <SHA>, pushed, GitHub About set
  (homepage + makefun topic).
- v1–v23 dirs untouched. No submission, no recording started
  (transcript bundles only, as briefed).
