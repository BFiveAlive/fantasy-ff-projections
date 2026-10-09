# Fantasy Football Projection Analysis

Personal analytics for a single 8-team Yahoo fantasy football league: head-to-head
matchup projections, optimal-lineup analysis, and Monte Carlo playoff odds, built on
Yahoo's projections and a variance model fit to historical results.

**Read-only. One user, one league, non-commercial.** No redistribution of provider data.

> Scaffolding stage — the tool runs locally today and the source is being published
> here incrementally. This repository currently holds project documentation and
> configuration only.

## What it does

**Historical baseline (2013–2026).** Preseason projections joined to final half-PPR
actuals, 9,517 player-seasons, used to measure how accurate projections actually are
before trusting them for anything.

**League analysis.** Reads league settings, teams, rosters, schedule and weekly
projections, then for each team and week:

- solves the **optimal lineup** exactly — with FLEX slots this is an assignment
  problem rather than a greedy per-position fill, so it runs a Hungarian solver over
  an eligibility matrix
- reports each manager's **bench gap**, the points left unplayed
- projects every head-to-head matchup, locking completed weeks to actual results

**Monte Carlo.** Win probabilities, expected values, and playoff and title odds.
Each player is drawn from a gamma distribution with a spike at zero, calibrated on
5,100 real player-weeks rather than an assumed variance.

## What the analysis found

Each of these was measured, and each one changed how the tool works:

| finding | evidence |
|---|---|
| Preseason DST projections carry no signal | r = 0.02 with actuals, 2015–2025 |
| Running backs score ~nothing in 22% of weeks | roughly 3× the quarterback rate |
| Projections run low at every position | DEF +10–19%, TE +7–11%, K +8–10%, three seasons running |
| A player's past projection gap does **not** predict his future gap | r = −0.09 across seasons, +0.08 within one |
| Position-level bias **does** carry forward | slope +0.72 |
| In-season projections already absorb early results | WR: R² 0.770 alone, 0.770 with weeks 1–2 added |
| Two games deserve ~20–30% weight, not 100% | measured against the preseason projection |

The last two are why rankings run on projections rather than recent form: adding a
"he's been hot" adjustment double-counts what the provider has already priced in.

## Planned structure

Auth and league retrieval; optimal lineups and matchup projections; Monte Carlo
simulation; variance and bias calibration; an early-season predictiveness study; a
free-agent upgrade finder; a scheduled watcher for projection and availability
changes; and a standalone HTML report.

## Configuration

Copy `.env.example` to `.env` and fill in your own credentials:

```
YAHOO_CLIENT_ID=
YAHOO_CLIENT_SECRET=
```

Credentials are gitignored, as is all league data — rosters and manager names belong
to a private league and are not published here.

## Data handling

Responses are cached to disk and not re-requested unless something changed, requests
are rate-paced, and the scheduled job runs a few times a day at roughly 30 requests
per run. Scripts that talk to a quota-limited API track spend against the daily cap
and stop rather than exceed it.
