# Decisions

One line per methodological decision: what you chose, why, and what it changed.
Agents append here via the `/new-feature` skill; you append here by hand.

- Exclude `casual`, `registered`: they sum to `cnt` exactly. Leaky model R^2 1.000 -> honest ridge 0.583.
- Exclude `cnt_2d_bfr`: a shipped 2-day lag we cannot test the alignment of. Build our own lags instead.
- Exclude `instant`, `yr`, `days_since_2011`: time indexes let a tree predict a flat constant past 2011 and a linear model extrapolate a trend we have no grounds for.
- Split 2011 / H1 2012 / H2 2012 by date. Test set touched once, after all feature work.
- Report mean and persistence baselines with every score. Persistence (test R^2 0.510, MAE 840) is within 0.07 R^2 of ridge and beats it on MAE; that caveat goes in the report.
- 2026-11-23: Track reports/ (metrics.json, audit.jsonl) in git so graded outputs are committed; data/processed/ and .venv/ stay ignored.
- 2026-11-23: Changed the new-feature skill's feature-list injection to `make features`; the starter's `python -c` command is not in the skill's allowed-tools, so the skill failed outside auto mode.
- 2026-11-23: Added temp_x_workday = temp * is_workday (availability: same-day weather and the calendar are both known for date d, like weather_severity). Validation: ridge R² 0.6475 -> 0.6611 (+0.0136), MAE 832.9 -> 809.2; hgb R² 0.2185 -> 0.2011. Kept: ridge is the deployed model and clears the 0.01 floor, though the gain is small for 182 validation days. Test set not consulted.
- 2026-11-23: Added SOURCE_COLS to src/config.py and a contract test that fails if any raw CSV column is not in FORBIDDEN, SOURCE_COLS, the feature set, or dteday (so a new raw column cannot enter silently). Verified red by dropping weather from SOURCE_COLS.
- 2026-11-23: Added test_no_feature_restates_target: no 1- or 2-column subset of features may reproduce cnt (lstsq residual <= 1 ride). Catches a leaked column under an innocent name, which test_no_forbidden_features cannot. Clean minimum residual ~3936 rides; casual+registered renamed gives ~4e-12.
- 2026-11-23: Added test_no_feature_restates_target: no 1- or 2-column subset of features may reproduce cnt (lstsq residual <= 1 ride). Catches a leaked column under an innocent name, which test_no_forbidden_features cannot. Clean minimum residual ~3936 rides; casual+registered renamed gives ~4e-12.
- 2026-11-23: Wrote the seed-fault skill (takes $ARGUMENTS) for the break/record/restore workflow I repeated for faults 1-2; used it for fault 3.
