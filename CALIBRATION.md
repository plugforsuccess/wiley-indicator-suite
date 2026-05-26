# Wiley Indicator Suite — Threshold Calibration Guide

## TL;DR

As of the adaptive-thresholds release, Echo and Tango ship with
**Adaptive Thresholds = on**. The rails self-calibrate per ticker and
timeframe to the 80th / 20th percentile of the oscillator's last 200 bars,
so you should not need to touch settings for any new chart. This document
exists to cover the static fallback case — what to set if you toggle
adaptive off.

## When the static fallback applies

The "Static Upper/Lower Threshold" inputs are used only when:

- The **Adaptive Thresholds** toggle is off, OR
- The chart has fewer bars than the adaptive lookback (warmup phase), in
  which case adaptive silently falls back to the static values.

In either case, the per-timeframe values below are the recommended starting
points.

## Recommended thresholds by timeframe

| Timeframe       | Upper Threshold | Lower Threshold | Notes                                              |
| --------------- | --------------- | --------------- | -------------------------------------------------- |
| Intraday 5m–15m | +35 to +40      | -35 to -40      | Tighter to filter noise on fast oscillator swings. |
| Intraday 1H     | +35             | -35             | Slightly tighter than daily default.               |
| Daily           | +30             | -30             | Default. Tuned against SPY/QQQ daily 2022–2025.    |
| **Weekly**      | **+20**         | **-20**         | Weekly readings compress — ±30 rarely prints.      |
| Monthly         | +15             | -15             | Same compression as weekly, more pronounced.       |

These values apply to **both** Echo and Tango — they share the same ±axis
structure by design, so calibrate them together when you change timeframes.

## How to apply

1. On the chart, click the gear icon next to **Echo** in the indicator list.
2. Set `Upper Threshold` and `Lower Threshold` to the values from the table.
3. Click **OK**.
4. Repeat for **Tango**.
5. Diamonds for past cross events on the visible chart should now appear.

Hardening picks up its inputs from Echo's and Tango's `_bull` / `_bear`
streams, so once those plot diamonds, Hardening will start scoring confluences
automatically — no separate change needed there.

## Worked example — ServiceNow (NOW) weekly, May 2026

Reported in the 2026-05-26 validation pass:

- Echo oscillates between roughly +25 and -22 across 2024–2026 — never
  touches ±30, so zero diamonds fire.
- Tango peaks at roughly +30 and bottoms near -10 — grazes the upper rail
  without producing a clean cross-back-below, and never reaches the lower
  rail, so zero diamonds fire.
- Bravo fires normally throughout (it does not depend on these thresholds).

**Resolution:** set Echo and Tango to ±20. Both pillars then print diamonds at
their past extremes on the visible chart, Hardening begins scoring triple
confluences where Bravo overlaps, and the validation pass can proceed.

## When to revisit

These numbers are starting points, not gospel. If a particular ticker runs
hot (oscillator regularly slams the rails) or cold (oscillator barely budges
off zero) at the recommended setting, shift the threshold ±5 in the
appropriate direction. The goal is for the oscillator to spend most of its
time between the rails and visit the extremes a handful of times per visible
chart period — not every bar, not never.

## Adaptive thresholds (now the default)

Both pillars now compute rolling-percentile rails out of the box:

- `Adaptive Thresholds` (default: **on**) — when on, the rails track the
  oscillator's own recent percentiles, replacing the static values.
- `Adaptive Lookback (bars)` (default: **200**) — how far back the
  percentile window extends. Roughly 10 months on daily, 4 years on weekly.
- `Adaptive Upper Percentile` (default: **80**) and `Adaptive Lower
  Percentile` (default: **20**) — the bands themselves. Tighter bands
  (e.g. 90/10) produce rarer signals; wider bands (70/30) produce more.

Because the rails track the oscillator's own distribution, the line will
spend roughly the configured fraction of its time outside each rail
regardless of how compressed or expanded the oscillator is on a given
chart. No per-ticker tuning required.

The visible threshold lines are now `stepline` plots rather than `hline`s
(hline requires a const value; the adaptive rails are series-valued). Color
and placement are unchanged.
