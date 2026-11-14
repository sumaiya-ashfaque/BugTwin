# BugHub Eclipse dataset: verified source contract

The Eclipse Platform source consists of an issue archive and two publisher-defined
split/reference CSVs. This document describes how BugTwin handles these artifacts.

## Source roles

- `eclipse_platform.zip` contains report rows with `Issue_id`, `Title`,
  `Description`, `Created_time`, and `Duplicated_issue`.
- `train.csv` and `test.csv` contain `Issue_id` and `Duplicate` columns.
  `Duplicate` is **not a binary flag**: values can be empty, a single numeric issue
  ID, or multiple numeric issue IDs delimited by semicolons.
- Auxiliary source references are retained separately from canonical report
  records and cannot silently become retrieval ground truth.

## Normalization

A split CSV row with `Duplicate=101150;101224;101443` expands into three
reference records. Each retains the original source file, row, raw cell value,
ordinal within the cell, reference issue ID, and split membership. A blank field
produces one auditable `no_reference` record. Non-numeric tokens are rejected.

The main archive is the source of report timestamps and direct report-to-report
references for the chronological retrieval benchmark. Publisher labels are audited
for canonical issue-ID alignment and split overlap, not misclassified as reports.

## Regression coverage

The notebook tests include the exact seven-ID and eight-ID semicolon patterns
observed in a real Eclipse Platform run, invalid-member rejection, empty
references, source-row provenance, repeated IDs, and a three-file integration
fixture matching the archive + train.csv + test.csv format.

## Validation limitations

The test suite has been executed locally on realistic fixtures. Real-source
validation is performed by the separate GitHub Actions workflow and must pass
on the actual publisher archive before a real-data acceptance claim is made.
Do not confuse these two validation levels.
