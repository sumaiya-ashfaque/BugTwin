# BugTwin M3 verification and release notes

## Scope

This patch updates the single executable application notebook, README, and implementation status. It preserves the original M1/M2 functionality and the corrected GitHub Actions workflow. No dataset files, model weights, cached embeddings, or external Python modules are committed.

## New behavior

- `MiniLMEncoder`: pretrained `sentence-transformers/all-MiniLM-L6-v2` inference, normalized float32 vectors.
- `SemanticIndex`: document fingerprint and npz embedding cache; FAISS `IndexFlatIP` for unfiltered search, exact prefiltered cosine scores for time-restricted historical evaluation.
- `HybridIndex`: BM25 + dense candidate union, reciprocal rank fusion, optional cross-encoder reranking.
- `M3 evaluation`: paired eligible queries; Hit@10, MRR@10, Recall@10, MAP@10, median query latency; manifest and CSV outputs.
- `Gradio`: method selection and visible retrieval model/source metadata.

## Scientific safeguards

The CI fixture uses a deterministic token-hashing **test double**; it is not a pretrained NLP model. A failed pretrained-model load disables semantic/hybrid methods rather than silently switching to a different model. All restricted queries filter future and project-ineligible reports before ranking. RRF and cross-encoder scores are not probabilities. Results on a capped source corpus are not full-corpus results.

## Required remaining acceptance checks

1. Run `BugTwin.ipynb` in Google Colab using BugHub Eclipse source data and record `m1_manifest.json`.
2. Confirm `m3/manifest.json` identifies `pretrained_sentence_transformer`, not `synthetic_fixture`.
3. Inspect matching reports, links, and dated labels, plus the exported per-query metrics.
4. Confirm the new GitHub Actions fixture-validation check succeeds on the M3 pull request.
5. Review and merge using **Create a merge commit** after tests pass.

## Local execution evidence — 14 November 2026

- Executed all **33 notebook cells** through `nbclient` with `BUGTWIN_M1_SOURCE=fixture`, `BUGTWIN_SKIP_INSTALL=1`, and `BUGTWIN_LAUNCH_UI=0`.
- Original M1 suite: **15/15 passed**.
- Existing lexical retrieval suite: **12/12 passed**.
- New M3 suite: **12/12 passed**.
- Repeated the end-to-end notebook execution with `BUGTWIN_ENCODER_MODE=off`; lexical-only fallback passed.
- The local runtime did not have `faiss-cpu` or `sentence-transformers` installed. FAISS and actual pretrained-model inference therefore remain **unverified locally**; the FAISS comparison test returned early and did not exercise the FAISS backend.
- The execution used generated **fixture data**, not real BugHub Eclipse records. These results establish functional integrity of the tested code paths, not measured duplicate-retrieval quality or Colab runtime acceptance.
