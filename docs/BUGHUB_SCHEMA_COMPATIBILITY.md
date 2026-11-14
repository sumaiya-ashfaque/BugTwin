# BugTwin dataset schema compatibility correction

The Eclipse Platform BugHub source includes:

- `eclipse_platform.zip` containing `eclipse_platform.csv`, the canonical report data.
- `train.csv` and `test.csv`, two-column publisher split-label files (`Issue_id`, `Duplicate`).

The issue report adapter now accepts `Created_time` as `created_at` and
`Duplicated_issue` as `duplicate_of`. Split-label files are validated and written
separately to `bugtwin_runtime/data/normalized/official_split_labels.csv`.
They do not become search documents or duplicate links.

The notebook retains its independent chronological partitioning and direct-link
retrieval evaluation. Publisher split labels are preserved for source audits rather
than substituted into historical relevance judgments.

## Validation

Run the notebook on fixture data in CI, and perform the real BugHub download in
Colab. The regression suite adds three source-schema tests, including a synthetic
archive and matching train/test files. A fixture pass alone does not establish
that all real source records are semantically correct.

## Apply

Extract the ZIP in the root of the current BugTwin repository on a new fix branch.
The ZIP replaces only `BugTwin.ipynb` and adds this note under `docs/`.
Run `git diff --check` before committing, and inspect GitHub Actions before merging.
