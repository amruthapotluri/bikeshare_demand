---
name: seed-fault
description: Seed one deliberate fault, record the red test and hook output, then restore. Use for Exercise 2 or any check that a test can fail.
allowed-tools: Read, Edit, Bash(make*), Bash(python -m pytest*), Bash(git diff*)
---

# Seeding a fault: $ARGUMENTS

1. Make exactly the one edit described above. Nothing else. Do not fix it.
2. Run `python -m pytest tests/ -q` and show the FAILED lines and the summary line verbatim.
   If nothing fails, say so plainly: that is a finding (the suite cannot see this fault), not something to hide.
3. Stop and wait. The user records the output in docs/SEEDED_FAULTS.md before anything is restored.
4. When the user says "restore", undo the edit, run `make test`, and show the summary line.
   It must match the pre-fault count. Then run `git diff --stat` and confirm it is empty.

Never edit a test to make a seeded fault pass.
