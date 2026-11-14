# Chronological benchmark sampling correction

BugTwin's full-source chronological partitions are authoritative. The retrieval
smoke corpus must not be constructed by taking the earliest N records and then
repartitioning that truncated set, because many early records can share one
calendar day.

`select_benchmark_corpus` now samples across all three existing partitions,
reserves a bounded number of verified historical query–target pairs, and fills
remaining positions deterministically. Search eligibility remains strictly
pre-query and intra-project. Train-only lexical vocabulary fitting remains in place.

## Interpretation

The bounded sample is deliberately link-anchored for functional testing. Its
retrieval scores are **diagnostic** and are not an unbiased estimate of
full-corpus duplicate retrieval performance. The output now distinguishes a
real-data sampled evaluation from an evaluated full-source benchmark.

The sampler exports `reports/lexical/corpus_sampling.json` with source and sample
sizes, date coverage and partition counts. Four regression tests cover
same-day-heavy data, known historical-positive preservation, reproducibility,
and corruption of source partitions.

## Validation

`Notebook CI` runs the synthetic fixture. `BugHub real-data smoke test` now also
triggers when the notebook is pushed directly to `main`. It runs the actual
Eclipse ingestion and lexical-only search pipeline and uploads the source audit.
Neither workflow alone verifies the Colab GPU or real MiniLM checkpoint.

The real-data smoke workflow additionally asserts sample date coverage and at
least one labeled validation and test query. These checks run from direct
`main` pushes and report a specific acceptance failure rather than silently
producing empty model-comparison tables.
