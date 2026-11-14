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
- The initial lexical release is preserved. M3 adds pretrained neural embeddings and optional reranking; user-uploaded corpus indexing and production-level explanations remain out of scope.

## Next engineering checkpoint

Confirm real-source data audit and run the pretrained-model benchmark in Colab before making performance claims.


## M3 — Semantic and hybrid retrieval

- Implemented: MiniLM encoder adapter, cached normalized embeddings, FAISS exact index / eligible-first cosine retrieval, BM25 + MiniLM reciprocal rank fusion, optional cross-encoder, comparative evaluation, UI selection, and twelve additional regression tests.
- Verification: synthetic-fixture execution is automated through the existing GitHub Actions notebook workflow. Real pretrained weights, FAISS availability, and real BugHub data must be verified in Colab before claiming real NLP benchmarks.
