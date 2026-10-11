# HIT140 Assessment 3 - Objective 2
## FIFA World Cup 2026: two pre-match regression models
**Darwin Group 46**  
 # Angel Shahi (s400420)
 # Sabraham Shrestha (s404389)
 # Akash Singh (s401643)
 # Nishant Shrestha (s400342)

## What this is

Two linear regression models built only from information available before kick-off.

| | Response | Rows | Explanatory variables |
|---|---|---|---|
| **2.1** | goal difference of a match | 104 | exactly 8, as differences between the two teams |
| **2.2** | goals scored by a team in a match | 208 | exactly 8, as levels plus the opponent's attributes |

Two quantities are shared between the models, against a permitted maximum of four.

## Run it

```bash
pip install -r requirements.txt
jupyter notebook WC2026_Objective2.ipynb      # then Run All
```

The notebook is committed with its outputs, so every figure and result is visible without running
it. Re-running reproduces them exactly: every seed is fixed. A full run takes about three and a half
minutes.

## The evaluation protocol

This is the part of the project worth reading first, because it is what the numbers mean.

**Nothing data-driven ever sees the rows it will be scored on.** The variable-selection procedure —
correlation screening, collinearity pruning at |r| = 0.85 and a matrix-rank guard — is one function
that reads only the rows it is handed, and it is re-run from scratch inside every cross-validation
fold. Hyperparameters are tuned by a further grouped cross-validation nested inside that fold's
training rows. Model 2.2's shared-variable cap is re-chosen inside every fold too, which is the
difference between estimating the procedure and estimating a procedure whose cap was already
settled; it costs 0.018 of R² and the notebook reports the cost.

**Twenty-one matches are held out and opened once.** They take no part in selecting variables,
choosing the shared-variable cap, tuning a hyperparameter or choosing an algorithm. The same 21
matches are held out for both models, so no match sits in one model's test set and the other's
training set.

**Every split is grouped by `match_id`,** so the two rows of a fixture are never separated.

**NRMSE** divides RMSE by one explicit named constant per model — `RANGE21` = 10.0 and `RANGE22` =
7.0, the range of that model's response across its training rows — passed into the metric rather
than inferred from whichever array is to hand. The per-fold alternative is computed alongside, and
the two do **not** agree closely: up to 0.004 apart on model 2.1 and 0.008 on model 2.2, about four
per cent of a typical NRMSE. A range is fixed by two observations, so a fold that drops the 7–0
group-stage result keeps a denominator of 5 or 6 instead of 7 — 30 of model 2.2's 150 folds — and
its NRMSE rises. The per-fold version is the less stable of the two, which is why the fixed one is
reported; the notebook prints the fold-range distribution so the instability is visible.

### Three things the pipeline is built to prevent

**Leakage from the match being predicted.** Form variables aggregate only a team's strictly earlier
matches, computed with a cumulative sum from which the current match is subtracted back out. The
notebook re-derives those totals independently for five teams and asserts they match.

**Information not yet public.** The FIFA ranking edition used is dated 11 June 2026, the same day as
the opening match, so a date comparison would not settle whether it was knowable in advance. FIFA's
announcement of that edition is timestamped 11 June 2026, 16:45 and describes it as published hours
before the tournament began; the opening match kicked off at 19:00 UTC. The notebook checks
publication time against kick-off time rather than date against date.

**Imputed values standing in for measurements.** A team's opening fixture has no tournament history.
Rather than fill those 48 rows with zero, they carry each team's form over their last ten
internationals before the tournament — all 480 of those fixtures listed individually with sources,
402 confirmed against a second source, none dated on or after the opening match of 11 June 2026.

**Selection that has seen the test rows.** The notebook measures this rather than asserting it is
absent: selecting once on all training rows and cross-validating that fixed set scored 0.259 on
model 2.1 against 0.280 for selection inside each fold, and the two were indistinguishable on model
2.2. The shortcut did not inflate these numbers. It is avoided anyway, because whether a variable
set chosen with the test rows in view flatters itself on those rows cannot be settled by any
procedure that keeps using them.

## Headline results

Both are ordinary multiple linear regression, the recommended model for both responses.

| | cross-validated R² | holdout sensitivity (200 draws) | the one designated holdout |
|---|---|---|---|
| **2.1** goal difference | 0.280 (sd 0.045) | 0.338 (sd 0.220) | 0.674 |
| **2.2** goals scored | 0.171 (sd 0.037) | 0.213 (sd 0.140) | 0.292 |

A single 21-match holdout has a standard deviation of 0.22 on model 2.1, so no number from one is
worth defending on its own. The defensible claims are ranges: roughly **0.28 to 0.34** for goal
difference and **0.17 to 0.21** for goals scored. The 200-draw column is a *sensitivity analysis* —
overlapping draws from the same 104 matches, measuring how much one holdout score depends on which
21 matches it contains — not an additional independent test set.

Six algorithms were compared on identical folds. Random forest leads model 2.1 by 0.007 over
multiple linear regression (16 of 30 repeats) and Ridge leads model 2.2 by 0.002
over it (an exact 15–15 split of the repeats). Each gap is far smaller than its own repeat-to-repeat
spread — 0.049 and 0.018 respectively — so we recommend the linear model on interpretability and
make no claim of superiority for any method.

**No confidence intervals or p-values are quoted from the repeats.** They share one training set and
differ only in where the fold boundaries fall, so they are not independent experiments: an interval
computed from them would not have the coverage its name implies. The notebook reports the mean
difference, its standard deviation, a 5th-to-95th-percentile range, whether that range covers zero,
and how the repeats split. Those are exploratory stability comparisons and nothing more.

## Files

```
WC2026_Objective2.ipynb              the complete analysis, executed
dataset_2_1_goal_difference.csv      104 rows · 1 ID · 8 explanatory · 1 response
dataset_2_2_goals_scored.csv         208 rows · 1 ID · 8 explanatory · 1 response
dataset/matches.csv                  104 fixtures, dates, venues, scorelines
dataset/team_pretournament.csv       48 teams: FIFA ranking, squad, qualifying, World Cup history
dataset/pre_tournament_form.csv      48 teams: form over their last 10 internationals before the tournament
dataset/pre_tournament_form_matches.csv   all 480 of those fixtures, individually sourced
dataset/rest.csv                     recovery intervals between fixtures
figures/                             the four report figures, written out by the notebook itself
prov/PROVENANCE.md                   sources, access dates, cross-check counts, known limitations
prov/form_validation_by_team.csv     per-team reconciliation record, one row per team
prov/form_match_log_full.csv         the 480 fixtures sorted by team, with source and cross-check status
                                     (409 confirmed on two sources; 71 single-sourced, each flagged)
requirements.txt
```

## Method

Weeks 1 to 9 of the unit: exploratory analysis and Pearson correlation, data visualisation, simple
and multiple linear regression, training and test splits, MAE / MSE / RMSE / NRMSE, R squared and
adjusted R squared, and the regression assumptions. For the competing-algorithm comparison the
notebook also fits Ridge, K-nearest neighbours, random forest and gradient boosting from
scikit-learn, with hyperparameters tuned by cross-validation nested inside the training folds.

## Data sources

FIFA Official (rankings, squad lists, fixtures, tournament archive) and FBref (match logs, squad and
club data), with Wikipedia used as a cross-check on the pre-tournament match log and as the source
for 58 fixtures absent from FBref. `prov/PROVENANCE.md` gives the full record, including the 13
fixtures that are single-sourced and why.
