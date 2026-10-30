# crossfit

Heath's CrossFit data and tools, in one place.

## Apps

- `apps/pace-clock/` — the Pace Clock. Live at https://hsinclair6140.github.io/crossfit/apps/pace-clock/
  (https://hsinclair6140.github.io/crossfit/ redirects there, `#plan=` links included).
  - Plans live in `apps/pace-clock/plans.txt`. The clock reads that file from `main` through the GitHub API
    every time it opens, so editing it goes live within seconds with no Pages rebuild. Only changes to
    `index.html` need a rebuild.

New helper apps go in their own folder under `apps/`.

## Data

- `data/training-log/` — the training log. `results.csv` gets a dated row for every reported result
  (load, reps, score, estimated 1RM). `sessions.csv`, `session_components.csv`, `summary.md` and
  `training_log.xlsx` were parsed from the coach spreadsheet (Aug 2024 to Oct 2025).
- `data/pr-board/efforts.csv|json` — export of the PR Board artifact's `efforts` collection.
- `data/pace-sheet/paces.csv|json` — export of the Pace Sheet artifact's `paces` collection
  (expected pace per movement at easy / moderate / hard / sprint).

The artifacts are what Heath uses at the gym; this folder is the permanent, versioned copy.
