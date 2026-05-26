# Wiley Indicator Suite — Threshold Calibration Guide

## Why this doc exists

The Echo (centered RSI) and Tango (centered, double-smoothed MFI) oscillators
ship with `Upper Threshold = 30` / `Lower Threshold = -30`. Those defaults are
tuned for **daily** charts. On other timeframes the oscillators compress or
expand, so a static ±30 either floods the chart with signals or — more often —
prints none at all.

If you load Echo or Tango and the line clearly oscillates but no diamonds are
plotting, the first thing to check is whether the line is actually crossing
the threshold rails. Open the indicator settings and adjust to the values
below for the current timeframe.

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

## Phase 2 — adaptive thresholds (not yet implemented)

A more robust long-term fix is to compute `upperThresh` and `lowerThresh`
dynamically as the 80th / 20th percentiles of the oscillator's last N bars
(e.g. 200), replacing the static inputs. This auto-calibrates per timeframe
and per ticker. Tracked as a Phase 2 enhancement — for now, this static
table is the recommended workflow.
