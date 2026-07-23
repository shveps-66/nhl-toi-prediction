# Glossary

Definitions for columns in the `Data/` CSVs and for features created in `NHL Model TOi per GP-GitHub.ipynb`.

Data sources: [NHL.com Stats](https://www.nhl.com/stats), [HockeyStats](https://hockeystats.com/stats).  
Cleaning and season filters were applied in Power Query before modeling.

**Sample filter:** at least **30 games** and **100 total TOI minutes** per player season.

Unless noted, values are **per player season**.

---

## Identity and context

| Metric | Meaning |
|--------|---------|
| `Index` | Player ID used to track the same skater across seasons (internal join key, not a public NHL ID) |
| `Season` | NHL season label (e.g. `21-22`, `25-26`) |
| `S/C` | Shoots (skaters): `L` or `R` |
| `Pos` | Position: `C`, `L`, `R`, `D` |

---

## Counting offense and physical play

| Metric | Meaning |
|--------|---------|
| `G` | Goals |
| `A` | Assists |
| `A1` | Primary (first) assists |
| `S` | Shots on goal |
| `Hits` | Hits delivered |
| `HitsT` | Hits taken |
| `BkS` | Blocked shots |
| `GvA` | Giveaways |
| `TkA` | Takeaways |
| `Pen Drawn` | Penalties drawn |
| `Pen Taken` | Penalties taken |
| `FOW` | Faceoffs won (mostly relevant for centers) |
| `FOL` | Faceoffs lost (mostly relevant for centers) |
| `ixG` | Individual expected goals |
| `iCF` | Individual Corsi For (individual shot attempts) |

---

## Size, ice time, and games

| Metric | Meaning |
|--------|---------|
| `Ht` | Height (inches in this dataset) |
| `Wt` | Weight (pounds in this dataset) |
| `TOI` | Total time on ice for the season (minutes) |
| `GP` | Games played |
| `TOI/GP` | Time on ice per game (**model target**, minutes per game) |

---

## Possession and zone starts

| Metric | Meaning |
|--------|---------|
| `SAT%` | Shot attempts for % (Corsi): on ice share of shot attempts |
| `SAT% Relative` | Shot attempts % relative to team baseline |
| `USAT %` | Unblocked shot attempts for % (Fenwick) |
| `OZ Ratio` | Offensive zone start ratio |
| `OZ Start%` | Share of starts in the offensive zone |
| `NZ Start%` | Share of starts in the neutral zone |
| `DZ Start%` | Share of starts in the defensive zone |

---

## On ice rates

| Metric | Meaning |
|--------|---------|
| `GF` | On ice goals for |
| `GA` | On ice goals against |
| `xGF` | On ice expected goals for |
| `xGA` | On ice expected goals against |
| `On-Ice GF %` | On ice goals for % |
| `oiSh%` | On ice shooting % |
| `oiSv%` | On ice save % |
| `PDO` | `oiSh% + oiSv%` (often used as a luck / sustainability signal) |

---

## NHL Edge skating

| Metric | Meaning |
|--------|---------|
| `18+ MPH Speed Bursts` | Count of speed bursts at 18+ mph |
| `20+ MPH Speed Bursts` | Count of speed bursts at 20+ mph |
| `22+ MPH Speed Bursts` | Count of speed bursts at 22+ mph |
| `Top Speed (MPH)` | Maximum recorded skating speed |

---

## Engineered features (created in the notebook)

These columns are built from the raw data before training and prediction.

| Metric | Meaning | How it is created |
|--------|---------|-------------------|
| `Pos_group` | Coarse position group | Map `L` / `R` → `W`; keep `C` and `D` |
| `Pos_C`, `Pos_D`, `Pos_W` | One hot position flags | From `Pos_group` via `pd.get_dummies` |
| `Season id` | Numeric season order | e.g. `21-22` → 1, `22-23` → 2, ... |
| `TOI/GP lag` | Previous season ice time | Prior season `TOI/GP` for the same `Index`. For `21-22`, filled from the 20-21 lag file when available. Missing lags are imputed with mean `TOI/GP` by `Pos_group` |
| `TOI_lag_missing` | Missing lag flag | `1` if no real prior season TOI was available, else `0` |
| `S_C_te` | Target encoded shot side | Mean `TOI/GP` by `S/C` on training history (with global mean fallback) |

Not every raw column enters the final model. Some fields (for example total `TOI`, `GP`, `USAT %`, `OZ Ratio`) are kept for context or prep only.

---

## Model outputs (not training inputs)

| Metric | Meaning |
|--------|---------|
| `pred_TOI/GP` | Model expected ice time per game |
| `residual` | Actual `TOI/GP` minus `pred_TOI/GP` (positive = more ice than expected; negative = less) |
| `abs_residual` | Absolute value of `residual` (used to rank mismatch size) |

---

## Quick read for scouting

1. **Raw stats** describe production, usage context, and skating.
2. **Engineered features** add role history (`TOI/GP lag`) and structure the model can use (`Pos_group`, `Season id`, `S_C_te`).
3. **Residuals** turn the model into a reevaluation tool: who is over or underplayed relative to their profile.
