# Bequeathed-Runner-Adjusted ERA

Traditional ERA has a well-known flaw: when a starter leaves with runners on
base (**bequeathed runners**), whether those runners score is decided by the
reliever, yet the runs are charged to the starter. A starter who hands
the bullpen a runner on 2nd with 2 outs is charged a full run if the reliever
gives up a bloop single and zero if the reliever gets a strikeout, even though
neither outcome reflects the starter's pitching.

This project recomputes ERA by replacing the actual downstream fate of a
starter's bequeathed runners with their expected value under
league-average pitching, given the base-out state at the moment of the
pitching change.

## Method

At each handoff we charge the starter an *expected* number of bequeathed-runner
runs instead of the actual outcome. That expectation is computed two ways:

| Method | What gets charged | Interpretation |
|---|---|---|
| **A — Marginal run expectancy** | `RE(state, outs) − RE(bases empty, outs)` | The marginal run value of the situation left behind (RE24 framework) |
| **B — Per-runner scoring probability** | `Σ P(runner on base b scores \| state, outs)` | Direct analog of ERA's own logic: "did *this specific runner* score?" |

Each method draws its expectation table from **two sources** — computed
empirically from the season's own play-by-play, and a standard published
league-average table — giving four adjusted-ERA variants to compare.

The adjustment per starter:

```
adjERA = 9 × (ER − actual_bequeathed_runs + expected_bequeathed_runs) / IP
```

A pitcher whose bequeathed runners scored *more* than the state-average
(bad luck / weak relief) sees his ERA come down; one who was repeatedly bailed
out sees it go up.

## Data

- **Play-by-play:** pitch-level Statcast via [`pybaseball`](https://github.com/jldbc/pybaseball),
  collapsed to plate-appearance level (state at PA start, outcome at PA end).
- **Season pitching stats:** the [MLB Stats API](https://statsapi.mlb.com)
  (`statsapi.mlb.com`), which returns MLBAM player IDs natively.

Run-expectancy and scoring-probability tables are estimated from innings 1–8
only, to avoid distortion from walk-off-truncated 9ths and the automatic
runner in extras.

## Setup

```bash
pip install pybaseball pandas numpy matplotlib requests
```

Then open `adjusted_era_2025.ipynb` and run top to bottom. The first run pulls
a full season of Statcast (~20–60 min); `pybaseball`'s cache makes subsequent
runs near-instant.

## Caveats

- **Earned vs. unearned:** Statcast has no earned-run flag, so `adjERA`
  assumes bequeathed-runner runs are earned. This makes it slightly
  inconsistent (official `ER` excludes error-driven runs, but the actual/
  expected bequeathed terms don't). Use **`adjRA9`** — computed alongside on
  total runs allowed — where exact accounting matters.
- **Swingmen:** handoffs are only detected in games a pitcher started; relief
  appearances are not adjusted. Exact for pure starters.
- **Runner tracking:** pinch-runner substitutions and rare
  lead-runner-out-while-trail-runner-scores plays are mis-assigned. Both are a
  small fraction of a percent of runner-PAs and roughly unbiased.

## Possible extensions

- Charge the starter at each baserunner event rather than only at handoff
  (full RE24 pitching lines).
- Mirror-image reliever credit for *inherited* runners (falls out of the same
  handoff table).
- Multi-season stability: does the adjustment persist year to year (a
  repeatable effect) or regress to zero (luck)?
