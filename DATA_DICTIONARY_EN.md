# Data Dictionary — Table Tennis Elo Ratings Dataset

*(English version of `DATA_DICTIONARY.md`. Same content.)*

All identifiers (`player_id`, `match_id`) are stable opaque hashes: the same player
carries the same `player_id` in every file, which makes the files joinable without
exposing any source identifier.

**Note on column names:** the CSV headers are in French, as shipped. The table below
gives the English meaning of each one.

---

## `matches.csv` — main file, 51,397 rows, 13 columns

| Column | Type | Description |
|---|---|---|
| `match_id` | string | Opaque match identifier. Unique, no duplicates. |
| `date` | date | Day the match was played (YYYY-MM-DD). |
| `league` | string | Competition name. 69 distinct values. |
| `player_a` | string | `player_id` of player A. |
| `player_b` | string | `player_id` of player B. |
| `elo_a_pre` | float | Player A's Elo rating **before** the match. No future leakage. |
| `elo_b_pre` | float | Player B's Elo rating **before** the match. |
| `sol_a_pre` | float [0–1] | Player A's solidity **before** the match. |
| `sol_b_pre` | float [0–1] | Player B's solidity **before** the match. |
| `p_model` | float [0–1] | Model probability that A wins, computed before the match. |
| `winner` | `A` / `B` | Actual winner. |
| `sets_a` | int | Sets won by A. |
| `sets_b` | int | Sets won by B. |

**Cold start.** The earliest matches in a player's history carry the initial 1500
rating, and `p_model` is exactly 0.500 for the 623 rows where both players are at
1500. Predictive power appears once ratings are established. Filter with
`m[(m.elo_a_pre != 1500) & (m.elo_b_pre != 1500)]`.

---

## `players.csv` — 1,662 rows

| Column (as shipped) | English | Type | Description |
|---|---|---|---|
| `player_id` | player id | string | Stable opaque player identifier. |
| `elo` | elo | float | Current Elo rating (set-points Elo — see methodology). |
| `solidite` | solidity | float [0–1] | Share of wins that are clean wins (≥2 sets margin). |
| `matchs_joues` | matches played | int | Number of matches played. |
| `nom_reel` | real name | string | Player name. |

Median matches played: 44. 1,202 players have 20 or more matches.

---

## `ratings_history.csv` — 7,982 rows

Rating trajectory: one point per player per ISO week.

| Column (as shipped) | English | Type | Description |
|---|---|---|---|
| `player_id` | player id | string | Stable opaque player identifier. |
| `semaine` | week | string | ISO week, format `YYYY-Sww` (e.g. `2026-S30`). |
| `elo` | elo | float | Rating at the end of that week. |

Weeks present: `2026-S27` through `2026-S34` (8 weeks).

---

## `calibration.csv` — 7 rows

Calibration table: for each announced-confidence band, the win frequency actually
observed. Use it to turn a raw model score into an empirical probability.

| Column (as shipped) | English | Type | Description |
|---|---|---|---|
| `confidence_annoncee` | announced confidence | string | Confidence band, in percent (e.g. `65-70`). |
| `frequence_observee` | observed frequency | float | Observed win rate in that band, in percent. |

| Announced | Observed |
|---|---|
| 50–55 | 52.4 |
| 55–60 | 55.7 |
| 60–65 | 60.6 |
| 65–70 | 70.4 |
| 70–75 | 75.9 |
| 75–80 | 86.4 |
| 80–85 | 89.6 |

The model is well calibrated up to about 65% confidence and **under**-confident above
it: in the 75–80 band it actually wins 86.4% of the time. Applying this table is what
turns the raw score into a usable probability.
