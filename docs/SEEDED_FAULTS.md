# Seeded faults

## Feature test, written first (temp_x_workday)

Test: tests/test_leakage.py::test_temp_workday_interact_matches_handcomputed, written before the feature existed. Output of `python -m pytest tests/test_leakage.py -q`:

    E       AssertionError: temp_x_workday column missing from features
    E       assert 'temp_x_workday' in Index(['dteday', 'instant', 'season', 'yr', 'mnth', 'holiday', 'weekday', ... 'weather_severity', 'cnt_lag_1', 'cnt_lag_7', 'cnt_roll_7'], dtype='str')

    tests/test_leakage.py:84: AssertionError
    =========================== short test summary info ============================
    FAILED tests/test_leakage.py::test_temp_workday_interact_matches_handcomputed
    1 failed, 7 passed in 0.49s