# Lexical retrieval verification — 13 November 2026

## Reproducibility

The delivered notebook was executed end-to-end in a local Jupyter kernel using:

- `BUGTWIN_M1_SOURCE=fixture`
- `BUGTWIN_SKIP_INSTALL=1`
- `BUGTWIN_LAUNCH_UI=0`

The synthetic dataset contains ten bug reports, five earlier-target direct duplicate links, and is used only as a software fixture.

## Verified outcomes

| Check | Result |
|---|---|
| Original dataset-engineering regression tests | 15 passed / 15 |
| Added lexical retrieval regression tests | 12 passed / 12 |
| Notebook execution without pip cell | Passed (27 notebook cells) |
| Canonical data/audit export | Passed |
| Frozen-training TF-IDF / BM25 index build | Passed |
| Time-eligible retrieval and holdout evaluation | Passed |
| Gradio component construction | Passed |
| Launching an externally accessible Gradio session | Not tested |
| Real BugHub Eclipse download/schema acceptance | Not tested in this environment |
| Full-scale Colab memory/performance benchmark | Not tested |

The development fixture produced measured retrieval scores, but those scores are intentionally excluded from claims of real-world model performance.

## Source limitations

The environment that generated this package cannot access `api.github.com` or `raw.githubusercontent.com`. The notebook's built-in BugHub acquisition must be verified in an internet-enabled Colab runtime before treating the real-data ingest and metrics as accepted.

## CI expectations

The GitHub Actions workflow uses the fixture mode and tests the notebook as a complete unit. This is a software-regression gate, not an independent source-dataset availability or scientific-performance gate.
