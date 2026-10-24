# Table Tennis Pre-Match Elo Ratings

103,561 table tennis matches where every row carries the Elo ratings of **both
players as they stood before the match was played**, together with the model's
pre-match win probability. A free 2,000-row sample lives in this repo.

No look-ahead leakage: the rating columns are frozen at the state that existed
before the result was known, so the file is usable in a backtest exactly as
shipped.

```
data/tt_ratings_sample_2000.csv      2,000 rows, 13 columns, CC BY 4.0
notebooks/01_calibration_check.ipynb recomputes the calibration from scratch
DATA_DICTIONARY.md / _EN.md          every column, documented
```

## The claim, and how to check it

Stated confidence against the frequency actually observed, measured on the
**full 103,561-match file** (102,787 rows once cold-start matches are excluded):

| Model said | Matches | Actually won | Gap |
|---|---|---|---|
| 50–55% | 43,521 | 51.7% | −0.8 |
| 55–60% | 28,944 | 55.0% | −2.5 |
| 60–65% | 15,848 | 58.9% | −3.6 |
| 65–70% | 7,706 | 63.3% | −4.2 |
| 70–75% | 3,642 | 69.3% | −3.2 |
| 75–80% | 1,801 | 75.0% | −2.5 |
| 80%+ | 1,325 | 79.1% | — |

**Every gap is negative.** The model is systematically over-confident above 60%,
by up to 4.2 points. An earlier release measured 2.9 points on half the data; on
twice the data it is 4.2. That is the honest number, and it is why `p_model`
should be treated as a baseline rather than an edge.

The bias is concentrated rather than uniform. On matches stated at 65% or above:

| League | Matches | Observed | Stated | Gap |
|---|---|---|---|---|
| TT Elite Series | 8,143 | 70.6% | 72.6% | −2.0 |
| Setka Cup | 4,166 | 63.8% | 70.0% | −6.2 |
| TT Cup | 1,293 | 63.0% | 68.6% | −5.7 |
| Czech Liga Pro | 503 | 55.9% | 67.3% | −11.4 |

Which league you work on matters more than the aggregate does.

Brier on the full file: **0.2436** against **0.2500** for a flat 50/50.

Run [`notebooks/01_calibration_check.ipynb`](notebooks/01_calibration_check.ipynb)
to reproduce all of this from the sample in this repo, with Wilson 95% intervals
so you can see which bands the sample can actually resolve.

Prefer to run it without installing anything? The same file, and the same
notebook, live on two hosted platforms:

- Kaggle —
  [dataset](https://www.kaggle.com/datasets/thierryseverinlamien/table-tennis-pre-match-elo-ratings-sample)
  · [notebook](https://www.kaggle.com/code/thierryseverinlamien/is-this-table-tennis-elo-model-actually-calibrated)
- Hugging Face —
  [dataset with an interactive table viewer](https://huggingface.co/datasets/lathise/table-tennis-pre-match-elo-ratings)

## Quick start

```python
import pandas as pd

m = pd.read_csv("data/tt_ratings_sample_2000.csv", parse_dates=["date"])
m[["date", "league", "elo_a_pre", "elo_b_pre", "p_model", "winner"]].head()
```

## Schema

`matches.csv` — 13 columns:

| Column | Type | Meaning |
|---|---|---|
| `match_id` | string | Opaque match id, unique |
| `date` | date | Match day, `YYYY-MM-DD` |
| `league` | string | Competition name |
| `player_a`, `player_b` | string | Opaque, stable player ids |
| `elo_a_pre`, `elo_b_pre` | float | Elo **before** the match |
| `sol_a_pre`, `sol_b_pre` | float | Solidity index before the match |
| `p_model` | float | Model probability that A wins, pre-match |
| `winner` | `A` / `B` | Actual winner |
| `sets_a`, `sets_b` | int | Set score |

## How the rating works

Elo computed at the **set** level rather than the match level: updated once per
set, weighted by the point margin within that set (k=3, α=1). A 3–0 and a 3–2
are the same result to a match-level Elo; they are plainly not the same
performance.

The match probability is derived from the set rating, then adjusted by a
solidity differential — the share of a player's wins taken by 2 or more sets —
with weights learned by logistic regression.

Glicko-2 was tested and rejected at Brier 0.2540.

## Releases

Rebuilt and republished roughly every 10 days, larger each time. Current release:
**25 June – 21 October 2026**, 119 days, 134 competitions, 17 weeks of weekly
rating history.

Every rebuild runs an automatic leakage check: if predictive accuracy exceeds
70%, implausibly high for table tennis, the release is blocked rather than
published. This release scores 55.9%.

## Sample vs. full dataset

| | sample (free) | full release |
|---|---|---|
| Matches | 2,000 | 103,561 |
| Players | 1,316 | 2,120 |
| Competitions | 39 | 134 |
| Rating history | — | 17 weeks |
| Players file | — | yes |
| Calibration table | — | yes |
| License | CC BY 4.0 | single-user commercial |

**Full dataset:** https://lathiselam.gumroad.com/l/tt-ratings

## Honest limits

- Four months of data, not a multi-season archive.
- Four competitions are 95.4% of the file.
- 774 matches (0.7%) are cold-start rows where both players still sit at the
  opening 1500 rating and the model outputs exactly 0.500.
- The edge over a match-level Elo at K=32 is **+0.0052 Brier** on a 13,892-match
  holdout. The methodology shows K=32 is not that baseline's optimum (interior
  optimum near K=14), so the edge over a fully tuned match-level Elo is smaller
  than that and is not quantified.
- `p_model` is over-confident above 60%, as measured above. The durable value
  here is the four leak-free rating columns as inputs to your own classifier.

## License

The 2,000-row sample in this repo is released under CC BY 4.0 — free to use,
including commercially, with attribution. The full dataset ships under a
separate single-user commercial license. This license covers the sample only.
