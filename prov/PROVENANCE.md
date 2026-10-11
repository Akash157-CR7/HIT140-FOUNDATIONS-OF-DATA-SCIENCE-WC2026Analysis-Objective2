# Provenance and validation record

Objective 2, HIT140 Assessment 3, Darwin Group 46.
Every figure in the report traces to one of the files listed here. This document exists because the
pre-tournament form variables are the only part of the dataset we assembled ourselves rather than
downloading whole, so they are the part that needs an audit trail.

## Approved sources

| Source | What we took from it |
|---|---|
| FIFA Official (fifa.com) | the 104 fixtures, scorelines, dates and venues; the pre-tournament FIFA ranking and points; the tournament archive used for appearances, titles and best finish |
| FBref (fbref.com) | squad lists, ages, caps and club affiliations; the pre-tournament international match logs |
| Wikipedia | independent cross-check on 409 of the 480 pre-tournament fixtures, and the primary source for the 58 absent from FBref |

Access date for the form files: **2026-10-09**. Per-team source URLs are in the
`source_url` column of `dataset/pre_tournament_form.csv` and in
`prov/form_validation_by_team.csv`; several teams required two or three URLs because FBref splits a
national team's log across season pages.

## The pre-tournament form window

For each of the 48 teams we recorded that team's last **10** completed senior internationals before
the tournament: 480 fixtures in total, individually listed in
`dataset/pre_tournament_form_matches.csv` and `prov/form_match_log_full.csv` with date, opponent,
competition, score, source and cross-check status.

Earliest fixture 2025-06-05, latest 2026-06-10. The tournament opened **2026-06-11**, so no
fixture in the window falls on or after the first match. The notebook asserts this.

Scores are the 90-minute result. Where a fixture went to extra time or penalties the note column
records it (28 fixtures carry a note) and the 90-minute score is the one used, so every
fixture in the window measures the same quantity.

## Cross-check status of the 480 fixtures

| Status | Fixtures |
|---|---|
| Confirmed on Wikipedia against FBref | 409 |
| Wikipedia only — fixture missing from FBref | 58 |
| FBref only — Wikipedia page not machine-readable for that date | 13 |
| **Total** | **480** |

409 of 480 fixtures (85 per cent) were read from two independent sources and agreed on both
the score and the date. The remaining 71 (15 per cent) are single-sourced and we do not
claim otherwise: 13 are FBref-only, because the Wikipedia results page was not machine-readable for
that date, and 58 are Wikipedia-only, because FBref does not carry the fixture at all - mostly
minor-confederation friendlies and invitational tournaments. Every row states which case it is in
its `cross_check` column, so the counts above are reproducible by grouping that column.

## Reconciliation

`prov/form_validation_by_team.csv` has one row per team and is the machine-readable validation
record. For each team it reports the aggregate in `pre_tournament_form.csv` beside the sum over that
team's rows in the match log, with an agreement flag for matches played, goals for and goals against,
the window dates, the leakage check, the access date, the source URLs and the count of fixtures in
each cross-check category.

All three aggregates reconcile for all 48 teams: **True**. The notebook repeats this check at
load time as one of its 11 assertions and will not run if it fails.

## Variables taken whole from a single source

The columns of `dataset/team_pretournament.csv` were taken from the three approved sources and
merged. For the three World Cup history fields we held a second reading from FBref alongside the FIFA
figure — `alt_fbref_top_clubs`, `alt_fbref_wc_appearances`, `alt_fbref_best_finish_ordinal` — and the
`sources_disagree` flag marks the teams where the two readings differ. The merge did not carry
per-column source URLs into this file; the sources are the three in the table above, accessed on the
same date as the form files.

## What a reader can check without rerunning anything

- Any single fixture: find the team in `prov/form_match_log_full.csv`, read the date, opponent and
  score, and compare against the `source` named on that row.
- Any team's aggregate: one row of `prov/form_validation_by_team.csv`.
- The leakage claim: the `window_closes_before_opening_match` column, or the notebook's assertion.
- Every number in the report: run the notebook. All seeds are fixed.
