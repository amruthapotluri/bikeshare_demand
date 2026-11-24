# Seeded faults

## Feature test, written first (temp_x_workday)

Test: tests/test_leakage.py::test_temp_workday_interact_matches_handcomputed, written before the feature existed. Output of `python -m pytest tests/test_leakage.py -q`:

    E       AssertionError: temp_x_workday column missing from features
    E       assert 'temp_x_workday' in Index(['dteday', 'instant', 'season', 'yr', 'mnth', 'holiday', 'weekday', ... 'weather_severity', 'cnt_lag_1', 'cnt_lag_7', 'cnt_roll_7'], dtype='str')

    tests/test_leakage.py:84: AssertionError
    =========================== short test summary info ============================
    FAILED tests/test_leakage.py::test_temp_workday_interact_matches_handcomputed
    1 failed, 7 passed in 0.49s

## Fault 1: forbidden column added to feature_columns

Change (made by the agent on request): in src/features.py, feature_columns returned
    [c for c in numeric if c not in FORBIDDEN] + ["casual"]

Hook stderr: fired in Claude Code right after the edit; same payload re-run by hand:

    $ echo '{"tool_input":{"file_path":"src/features.py"}}' | CLAUDE_PROJECT_DIR=$PWD bash .claude/hooks/check_leakage.sh; echo "exit=$?"
    Leakage tests failed after editing src/features.py:
    >       assert not (cols & FORBIDDEN), f"forbidden columns in feature set: {cols & FORBIDDEN}"
    E       AssertionError: forbidden columns in feature set: {'casual'}
    tests/test_leakage.py:53: AssertionError
    FAILED tests/test_leakage.py::test_no_forbidden_features - AssertionError: fo...
    1 failed, 8 passed in 0.49s
    exit=2

Red test: test_no_forbidden_features.
Turns to fix: 1. The agent identified the leak from the hook message unprompted, asked to revert, and reverted on "Yes, revert it." After fix: 18 passed.

## Fault 2: .shift(1) removed from cnt_roll_7

Change (made by the agent on request): in src/features.py,
    out["cnt_roll_7"] = out[TARGET].shift(1).rolling(LAG_WARMUP).mean()
became
    out["cnt_roll_7"] = out[TARGET].rolling(LAG_WARMUP).mean()
so the 7-day mean includes today's count.

Hook stderr: fired in Claude Code right after the agent's edit. The agent's revert landed before I copied it, so I re-seeded the identical one-line change by hand (git diff --stat: src/features.py | 2 +-), ran the hook with the same payload, and restored:

    $ echo '{"tool_input":{"file_path":"src/features.py"}}' | CLAUDE_PROJECT_DIR=$PWD bash .claude/hooks/check_leakage.sh; echo "exit=$?"
    Leakage tests failed after editing src/features.py:
        def test_rolling_mean_excludes_today(feats):
            """cnt_roll_7 must be the mean of the 7 calendar days *before* today."""
    >           assert feats.set_index("dteday").cnt_roll_7[date] == pytest.approx(expected)
    E           assert np.float64(4389.0) == 4414.857142857143 ± 0.00441486
    E             Obtained: 4389.0
    E             Expected: 4414.857142857143 ± 0.00441486
    tests/test_leakage.py:79: AssertionError
    FAILED tests/test_leakage.py::test_rolling_mean_excludes_today - assert np.fl...
    1 failed, 8 passed in 0.50s
    exit=2

Red test: test_rolling_mean_excludes_today.
Turns to fix: 1. The agent identified the temporal leak from the hook message unprompted and offered three options, one of which was to "fix the test to match the new behavior", which would have weakened the check. I chose revert. After fix: 18 passed.
## Fault 3: weather encoder's raise removed

Seeded with my /seed-fault skill.

3a. Change: in encode_weather, `raise ValueError(...)` replaced with `pass`.
Result: python -m pytest tests/ -q -> 18 passed. Hook (same payload, run by hand) -> exit=0.
Finding: removing the guard alone changes no output, because every label in this mirror (GOOD/MISTY/BAD) is already in WEATHER_ORDER. No test can fail; the guard only matters once a label is unmapped.

3b. To recreate the Step 5 bug, also changed "BAD": 2 to "STORM": 2 in WEATHER_ORDER, so the 20 BAD days map to NaN and are dropped by dropna() with no error:

    FAILED tests/test_contract.py::test_only_expected_rows_are_dropped - Assertio...
    1 failed, 17 passed in 2.36s
    hook: exit=0

Finding: the suite catches it, but the PostToolUse hook stayed silent (exit 0), because check_leakage.sh ran only tests/test_leakage.py and the row-loss test lives in tests/test_contract.py. The agent would not have been told.
Turns to fix: 1 ("restore"). After fix: 18 passed.

Fix: changed check_leakage.sh to run python -m pytest tests/ -q. Re-seeded 3b by hand and ran the fixed hook:

    E       AssertionError: lost 34 rows; expected at most 14
    E       assert 34 <= 14
    
    tests/test_contract.py:48: AssertionError
    =========================== short test summary info ============================
    FAILED tests/test_contract.py::test_only_expected_rows_are_dropped - Assertio...
    1 failed, 17 passed in 1.92s
    exit=2
