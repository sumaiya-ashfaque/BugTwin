# BugTwin implementation checkpoint

## Delivered

- Preserved the original M1 ingestion, canonicalization, duplicate-link auditing and regression suite.
- Added training-vocabulary TF-IDF and BM25 ranking with strict candidate-time gating.
- Added chronological, source-labeled Hit@10 / MRR@10 / Recall@10 / MAP@10 evaluation.
- Added a small Gradio search interface with source-backed report results.
- Added notebook fixture CI and a Git-first development workflow.

## Validation boundaries

- Synthetic fixture execution verifies notebook control flow and retrieval contracts, but is not a real-dataset benchmark.
- Real GitHub BugHub source parsing and observed retrieval metrics must be verified in Colab before scientific claims.
- Indexing the earliest 12,000 reports is a limited, time-preserving benchmark cohort, not the full historical index.
- This is an initial working lexical search release. Neural embeddings, reranking, user-uploaded corpus indexing, and production-level explanations are not yet implemented.

## Next engineering checkpoint

Confirm real-source data audit, inspect historical query counts and measured baseline metrics, then implement MiniLM/FAISS and compare it on an identical evaluation population.
