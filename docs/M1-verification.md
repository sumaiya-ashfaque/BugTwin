# BugTwin — M1 Implementation and Verification Report

**Document:** BT-M1-REP-001  
**Version:** 1.0  
**Date:** 26 October 2026  
**Parent design:** BT-SPEC-001 v1.1 / BT-ARCH-001 v1.0  
**Executable source:** `BugTwin.ipynb` (one notebook; no dependent application source files)

## 1. Implemented scope

| Capability | Implementation | Evidence |
|---|---|---|
| Source discovery | GitHub tree API, commit/tree cross-check, immutable raw-file URLs | Source acquisition functions |
| Dataset provenance | SHA-256 checksum for each source artifact, runtime manifest | `m1_manifest.json` |
| Multiple formats | CSV, TSV, JSON, JSONL, XML, ZIP, TAR, GZ; safe member validation | Reader tests |
| Canonical schema | Unique `(project, report_id)` key, UTC creation time, source fields, searchable text | Canonical adapter and validator |
| Invalid-row quarantine | Missing identifiers, missing text and malformed timestamps | Quarantine CSV and unit test |
| Duplicate relationship audit | Direct source-linked positives, missing/future/tied/self targets | Edge audit and tests |
| Chronological partitions | Train, validation, test; creation-date boundaries; deterministic sorting | Split builder and tests |
| Historical eligibility | Strict `candidate.created_at < query.created_at` | Eligibility API and tests |
| Query registry | Historical positive target keys separated from unknown/unconfirmed records | Query registry and tests |
| Linked demo cohort | Most recent seed reports plus eligible earlier targets | Cohort export and tests |
| EDA and exports | Cohort plot, edge status chart, CSV tables, manifest, test report | Generated on notebook execution |
| Regression tests | 15 tests for schema, parsing, graph integrity, splits, and archive safety | Test execution |

**Not implemented in M1:** TF-IDF, BM25, dense embedding retrieval, neural reranking, quantitative retrieval benchmark metrics, and Gradio UI. These belong to later project milestones.

## 2. Validation executed in this development environment

- Executed all non-install notebook code cells in an isolated kernel with `BUGTWIN_M1_SOURCE=fixture` using `nbclient`.
- Result: **success, eight executable code cells, zero errors**.
- M1 regression suite: **15 passed, zero failed**.
- Synthetic test dataset: **10 test bug reports, five direct earlier-target duplicate links**.
- Verified export generation and source-grounded query labels on that fixture.

**Important distinction:** Fixture results establish internal pipeline and test behavior. They are neither real dataset statistics nor BugTwin model-performance results.

## 3. Real-source status and remaining acceptance gate

The BugHub repository's README advertises an Eclipse Platform historical bug corpus and duplicate detection data. Public source listing and methodology were reviewed, but this execution environment cannot connect directly to GitHub's data API or download the actual data files.

As a result, the notebook defaults to **real BugHub acquisition** and performs the real-data audit *when run in Google Colab*. The real-source M1 acceptance gate is **not yet certified**.

The notebook enforces meaningful integrity checks, including minimum real report and historical-positive counts, low quarantine rate, and a pinned source revision. It refuses to claim benchmark readiness from synthetic fixtures or from an incomplete, unverifiable source.

If BugHub's current file schema differs from the recognized aliases, the notebook raises an explicit ingest error. Inspect the actual input columns and update the documented adapter rather than fabricating or silently discarding labels.

## 4. Run in Google Colab

1. Upload `BugTwin.ipynb` to Google Colab.
2. Select **Runtime → Run all**. The notebook's default source is `bughub`.
3. Examine the **Source audit**, **Quarantine**, **Duplicate-link audit**, and **M1 Manifest**.
4. Confirm `benchmark_ready` is `true` and inspect edge-status counts and available positive queries in every planned benchmark partition.
5. Download the generated data tables under `/content/bugtwin_runtime/data/normalized/` and reports under `/content/bugtwin_runtime/reports/m1/` if needed.

For offline pipeline testing only, set `BUGTWIN_M1_SOURCE=fixture` before the configuration cell runs. This mode must never be used to claim a real dataset benchmark.

For a local data source, set `BUGTWIN_M1_SOURCE=local` and `BUGTWIN_M1_LOCAL_ROOT` to a file or directory accessible in the runtime. Local source authenticity and license must be independently confirmed.

## 5. Data and evaluation integrity notes

- Only direct, source-annotated duplicate edges constitute M1 positives. Transitive group membership is not inferred automatically.
- Duplicate annotations whose linked report did not exist at query creation time are retained in the audit as future targets, not counted as retrievable historical positives.
- Unknown/unlinked reports are not treated as reliable negative pairs.
- The historical eligibility method is designed to filter *before* retrieval top-K selection in M2/M3.
- Potential post-creation edits in source snapshots remain an acknowledged limitation when source history is unavailable.
- Training/validation/test are query cohorts; earlier reports in any cohort can be candidates for later queries.
- BugHub states its datasets are available for research; do not redistribute raw downloaded data without checking source terms.

## 6. Required next step

Run the notebook in Google Colab against the pinned BugHub source, review the actual source schema, and confirm data/positive-query counts and integrity gates. Once that has been demonstrated, freeze the M1 manifest and begin M2 lexical baselines using the generated canonical reports and query registry.

## References

- LogPAI BugHub: https://github.com/logpai/bughub
- BugTwin architecture baseline: `BugTwin_Architecture_Stakeholder_Maps_v1.0.md`
- BugTwin master specification: `BugTwin_Master_Specification_v1.1.md`
