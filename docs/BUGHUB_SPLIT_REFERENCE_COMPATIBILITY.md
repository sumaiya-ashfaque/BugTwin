# BugHub Eclipse Platform source compatibility

The Eclipse Platform source includes a full issue archive and auxiliary two-column
`train.csv` and `test.csv` files. These auxiliary files contain `Issue_id` and
`Duplicate` issue-reference data, not Boolean duplicate flags.

## Contract

- Parse `Issue_id` and `Duplicate` as issue identifiers, with documented null sentinels.
- Preserve original publisher file/split membership and source row.
- Allow multiple relations for one `Issue_id`, including across publisher files.
- Audit missing source and referenced issues, self references, repeated exact pairs,
  and overlap between publisher files.
- Do not assume that the source's two-column rows are equivalent to binary
  duplicate labels, a mutually exclusive partition by unique issue, or
  chronologically eligible direct-positive links.
- Keep the current independently generated chronological benchmark and the
  main report archive's `Duplicated_issue` links unchanged.
- Export auxiliary rows to `bugtwin_runtime/data/normalized/official_split_labels.csv`
  for compatibility with the previous artifact path; the contents now have
  `record_key`, `reference_key`, `reference_kind`, `official_split`, `source_file`,
  `source_row` and per-row audit fields instead of `duplicate_flag`.

## Verification

Fixture-mode notebook execution checks schema adaptation, reference preservation,
source-file overlap, and previous M1–M3 regression coverage. A real Eclipse dataset
run remains required to confirm population counts, referential integrity, and
semantic alignment. Publisher split rows are *not* promoted into the historical
retrieval ground truth without further evidence on direction and timestamp eligibility.
