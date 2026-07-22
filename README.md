# NHL Time on Ice (TOI/GP): finding players worth a second look

Build an expected ice time model for NHL skaters, then use **actual vs predicted** `TOI/GP` to flag players who may deserve reevaluation. Some play less than the model expects from their profile. Others play more.

A second goal is interpretive: **the same metrics can treat players differently**. Feature importance and SHAP show why two similar looking lines of stats imply different roles and ice time expectations. That is useful for market search, where you care about role fit, not only raw counting numbers.

## Why this matters

Ice time is one of the strongest signals of a player's role. The residual

`actual TOI/GP − predicted TOI/GP`

is a practical mismatch detector:
- **Positive residual:** more ice than the model expected
- **Negative residual:** less ice than the model expected

Those gaps are a starting point for deeper scouting (coach usage, PP/PK, injury, linemates), not a final roster decision.

## Project structure

```
├── NHL Model TOi per GP-GitHub.ipynb   # main analysis notebook
├── Data/
│   ├── All Stats (Seasons 21-25).csv
│   ├── All Stats (Seasons 25-26).csv
│   └── For TOI-GP lag metric (Season 20-21) Github.csv
├── requirements.txt
└── README.md
```

## Data sources

Stats were collected from:
- [NHL.com Stats](https://www.nhl.com/stats)
- [HockeyStats](https://hockeystats.com/stats)

Not every collected metric enters the final model. Cleaning, appending seasons, and early prep were done in **Power Query**. Player seasons were filtered to a minimum of **30 games** and **100 total TOI minutes** per season (also in Power Query) to reduce noise from tiny samples. This repository focuses on modeling and interpretation in the notebook.

| File | Seasons | Role |
|------|---------|------|
| `All Stats (Seasons 21-25).csv` | 21-22 … 24-25 | Training history |
| `For TOI-GP lag metric (Season 20-21) Github.csv` | 20-21 | Prior season TOI for lag features |
| `All Stats (Seasons 25-26).csv` | 25-26 | Prediction and residual review |

Example feature families: goals/assists, shots, hits, blocks, possession (SAT%), zone starts, faceoffs, individual expected goals, on ice GF/xG rates, PDO, NHL Edge speed bursts / top speed.

**Target:** `TOI/GP` (minutes per game)

## Method (short)

1. EDA on multi season player data
2. Feature engineering: position group (L/R → W), season id, prior season TOI lag, missing lag flag, target encoding for shot side (`S/C`)
3. Model experiments: XGBoost and CatBoost (plus a simple ensemble) on train/val/test
4. Final CatBoost model on full 21-25 history
5. Predict 25/26, rank residuals, inspect global feature importance and local SHAP for a player comparison
6. Push actual vs expected results into a Power BI dashboard for league wide review

## Power BI dashboard

Model outputs (actual `TOI/GP`, predicted `TOI/GP`, residuals) were also used to build a **Power BI** dashboard for season **25/26**. The dashboard covers all skaters in that season and compares actual vs expected ice time so underplayed and overplayed players (relative to the model) are easy to spot.

The `.pbix` file is not included in this repository. This repo focuses on the modeling notebook and interpretation.

## Interpreting importance vs SHAP

- **Global feature importance:** what drives TOI/GP on average across the league
- **SHAP (local):** which features pushed *this player's* prediction up or down

Tree models do not have linear regression style coefficients. SHAP contributions are the right tool for explaining one player, and for showing why two players with similar headline stats can still get different expected ice time (different roles).

## How to run

```bash
# from this project folder
python -m venv .venv
# Windows:
.venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook "NHL Model TOi per GP-GitHub.ipynb"
```

Run all cells top to bottom.

## Key libraries

- pandas, numpy
- scikit-learn
- XGBoost, CatBoost
- matplotlib, seaborn

## Limitations

- Same season counting stats are used when estimating that season's TOI/GP. Treat results as **expected ice time given the season profile**, not a pure prior year only forecast.
- Target encoding for `S/C` is fit on the full final training history. Stricter cross validation would fit encoding inside each fold.
- Large residuals always need context before any personnel conclusion.
