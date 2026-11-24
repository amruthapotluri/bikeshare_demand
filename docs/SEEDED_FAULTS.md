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