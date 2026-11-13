# BugTwin — Master Engineering Specification and Architecture Baseline

**Document ID:** BT-SPEC-001  
**Revision:** 1.1 (consolidated architecture edition)  
**Date:** 25 October 2026  
**Status:** Specification complete; application implementation not started  
**Associated architecture:** `BugTwin_Architecture_Stakeholder_Maps_v1.0.md` (BT-ARCH-001)  
**Planned implementation artifact:** `BugTwin.ipynb`

## Document control and scope

This consolidated specification preserves the existing BugTwin harness requirements and incorporates the architectural decisions, stakeholder maps, and acceptance gates in the accompanying BT-ARCH-001. It is a **design contract**, not a statement that data, models, evaluation, or UI already run. The two files constitute a single controlled design package. Technical detail in the architecture document prevails for implementation mechanics; product requirements here prevail for scope and acceptance.

**Product:** semantic duplicate bug report retrieval and explainable review. **Target users:** developers, QA engineers, issue maintainers, ML researchers, academic/portfolio reviewers. **Delivery:** fully functional, Python-only source code in one Colab-compatible notebook, with live Gradio UI. No separate maintained application source code or paid inference endpoint.

## S1. Problem statement, value, and measurable outcomes

Software teams regularly receive different descriptions of the same defect. Searching old issues by title or literal keywords misses paraphrases, while blindly using semantic similarity can conflate similar symptoms with genuinely different causes. BugTwin will rank older, traceable issue reports for a newly submitted report, showing source evidence and keeping the final duplicate decision with a human.

**User promise:** “Given a bug title and description, search my eligible historical issue corpus and show the best plausible existing reports, with transparent evidence and provenance.”

**Research goal:** compare lexical, dense, hybrid and optional reranking methods under realistic time-aware candidate constraints. **Product goal:** provide a usable single-notebook interface and exportable results. **Non-goals:** root-cause guarantee, auto-close/merge, full hosted SaaS, fine-tuning as a prerequisite, paid API reliance.

## S2. Fixed mandatory constraints

| ID | Non-negotiable condition | Acceptance evidence |
|---|---|---|
| C-01 | All maintained application source code is in one `.ipynb` | notebook contains implementation and launch cells |
| C-02 | Python is application programming language | no separately authored JS/backend/frontend source |
| C-03 | Runs in Google Colab | clean Run All on supported Colab profile |
| C-04 | Real working application, not prototype screenshot | interactive Gradio query and true retrieval |
| C-05 | Real software bug duplicate-discovery use case | ranked historical bug candidates |
| C-06 | Genuine reports and verified positives for evaluation | dataset/source and link audit |
| C-07 | Reproducible and scientifically valid evaluation | manifests, same eligible corpora, metric tests |
| C-08 | No separate maintained Python packages or backend | files generated only as data/cache/export |
| C-09 | No paid external inference API | open models, CPU search fallback |
| C-10 | Graceful predictable failure behavior | negative-path unit and end-to-end tests |

Dependencies and model weights may be downloaded from public package/model providers; cached embeddings, indexes, CSV/JSON exports, and logs may be generated at runtime. None may be a hidden prerequisite for a fresh notebook run.

## S3. Functional scope and priorities

**P0 required:** dataset loading and validation (FR-01); report submission (FR-02); input validation (FR-03); duplicate retrieval (FR-04); ranked candidates (FR-05); original candidate text/metadata (FR-06); selectable baseline/semantic methods (FR-07); evaluation results (FR-08).

**P1 intended complete release:** dataset exploration (FR-09); hybrid BM25+dense retrieval (FR-10); configurable reranking (FR-11); source-grounded evidence highlighting (FR-12); metadata filters (FR-13); user CSV input with schema mapping (FR-14); CSV/JSON export (FR-15); session-level reviewer feedback separate from ground truth (FR-16).

**P2 optional, gated:** calibrated duplicate decision confidence (FR-17), task-specific embedding fine-tuning (FR-18). Do not silently classify unlabeled pairs as nonduplicates to implement these features.

**Typical workflow:** load corpus → input report → choose mode/project/K → compute ranked candidates → inspect original report and score semantics → review candidate → export. UI must handle an empty result without asserting that no duplicate exists.

## S4. Stakeholders, ownership, and governance

The complete register, influence/interest map, relationships, journeys and RACI table are in BT-ARCH-001 §A3. Stakeholders: developer/report submitter, QA/triage analyst, issue/project maintainer, notebook/ML implementer, academic evaluator, dataset steward, security reviewer, runtime/library providers, portfolio assessor.

**Decision rights:** the software/project maintainer decides whether to merge/close issues; NLP implementer decides architecture under these requirements; reviewer/evaluator can reject unsupported scientific claims; security reviewer can block unsafe public sharing. In a solo student project these are responsibility perspectives, not claims that outside organizations have signed off.

![Stakeholder power-interest map](BugTwin_Stakeholder_Power_Interest.png)

## S5. High-level architecture

![Logical architecture](BugTwin_System_Architecture.png)

Logical layering is **data acquisition → canonicalization and duplicate graph → chronological evaluation and indexing → lexical/dense retrieval → hybrid fusion / optional reranker → evidence construction → engine → Gradio UI**, all as Python definitions and cells in `BugTwin.ipynb`. The offline experiment path shares retrievers with the app but does not depend on UI state. The state machine and interface contracts are defined in BT-ARCH-001 §§A2–A7.

The runtime may generate `/content/bugtwin_runtime/` artifacts but cannot require users to download additional application source files. Optional Google Drive cache is versioned and bypassable.

## S6. Data and annotation requirements

**Primary candidate source:** [LogPAI BugHub Eclipse Platform](https://github.com/logpai/bughub). The source README lists 85,156 issues and 14,404 duplicates for Eclipse Platform; a separate reported chronological train/test table uses different positive counts. These are source-published figures with different definitions, **not** yet validated canonical ingest counts. M1 must inspect actual archives, schema, link direction, timestamps, provenance and permission terms before proceeding. Secondary projects are optional after initial reproduction.

**Canonical fields:** `(project, report_id)` unique key; title, description, created_at (UTC-aware); optional product/component/severity/source_url; source provenance, text snapshot timestamp if known, derived lexical/semantic text. **Ground-truth edge:** source issue, target issue, source annotation, optional edge-label time and validity status. **User-upload mode:** supports custom CSV with required key/title/text; if duplicate annotations are missing, run retrieval only and disable definitive evaluation.

**Rules:** preserve exception names, version strings, paths, error codes and function identifiers. Track invalid rows, missing targets, out-of-domain or unsupported labels. Do not use `duplicate_of`, resolution/status/closing metadata, or future comments as features. Report limitations if only post-edited issue snapshots are available. Never treat missing duplicate links as reliable negative labels.

## S7. Temporal protocol and leakage controls

**Primary experimental setting:** query `q` at time `t` can search only reports with `created_at < t` under chosen project constraints, excluding itself. Positive relevance derives from verified historical duplicate links/groups only where justified. Queries with no historical verified positive have a separate status and are not automatically counted as failed positive retrievals.

**Hard invariant:** filter candidate eligibility **before** lexical/dense top-K retrieval, not after. Exact dense similarity over the eligible prefix is acceptable for historical experiments even when the application uses a reusable FAISS index for current-corpus search. Split report/query dates chronologically; tune only on validation; lock final test metrics and manifests. If text snapshots include edits that happened later, clearly disclose residual hindsight leakage.

## S8. Preprocessing and representations

Separate lexical and semantic text streams. Lexical may normalize case/whitespace and select useful tokens, without erasing technical symbols. Semantic combines title and bounded high-information description content; store original text as evidence. Track sequence truncation and include a title-only vs title+description ablation.

Initial semantic model `sentence-transformers/all-MiniLM-L6-v2` yields 384-dimensional embeddings and has a documented 256-token maximum; model revision and actual batch size must be recorded. Full description understanding is not guaranteed beyond truncation.

## S9. NLP experiment matrix and selection

| ID | Method | Purpose |
|---|---|---|
| E1 | TF-IDF + cosine | baseline |
| E2 | BM25 | strong lexical baseline |
| E3 | MiniLM dense cosine | semantic baseline |
| E4 | BM25 + MiniLM via RRF | hybrid proposal |
| E5 | E4 + MiniLM MS MARCO cross-encoder | optional reranking |
| E6 | task-tuned embedding encoder | optional fine-tuning |

Initial *untuned* parameters: lexical top 50, semantic top 50, RRF k=60, reranker depth 10-30. Validate choices before declaring a winner. Raw cosine/RRF/cross-encoder scores are **not duplicate probabilities**. Use exact eligible-corpus scores in offline historical evaluation, with optional FAISS normalized inner-product index for interactive current-corpus search.

## S10. Search output and evidence

`SearchResponse` holds query summary, method and index version, eligible count, ordered candidate keys and ranks, score breakdown, original title and description excerpts, available source URL, warnings, latency, and run provenance. No self-matches or duplicate candidate IDs. Stable ties broken deterministically. Source excerpts must be traceable verbatim to original records; highlighted matching terms are indicators, not claims about actual transformer attention or reasoning.

## S11. Interactive UI specification

Use Gradio, launched from the notebook. Required controls include title, description, project/filter, method selection and top-K. Required results include ranked cards with issue keys, text snippets, score labels, and source links. Additional tabs: Dataset Explorer, Model Comparison, Benchmark Dashboard, Review & Export. Handle READY/INDEXING/SEARCHING/DEGRADED/ERROR states; present readable errors without private stack traces. Reviewer labels are human feedback data, kept distinct from verified source labels and never silently used to score the held-out test.

## S12. Primary metrics and experimental studies

- **Hit@1/5/10:** fraction of eligible-positive queries with ≥1 relevant report in top K.
- **Set Recall@K:** mean or micro fraction of eligible relevant reports retrieved; state aggregation.
- **MRR:** reciprocal rank of first relevant issue, averaged over eligible queries.
- **MAP@K:** precision-recall ranking quality when multiple relevant issues exist, with exact denominator stated.
- **Latency:** warm first-stage, reranking and end-to-end p50/p95 under named device/corpus conditions.
- **If verified negatives exist:** precision/recall/F1/PR-AUC and false-positive rate for a separately defined binary decision task.

Required ablations: title vs full description, TF-IDF vs BM25, lexical vs dense, dense vs hybrid, hybrid vs reranking, with/without technical-token retention and optional metadata constraints. Report query count, candidate counts, positive coverage, broken-link rates, runtime settings, seed and paired query-level uncertainty when feasible.

**Do not** equate paired classification accuracy with large-corpus retrieval quality; do not claim that an embedding score of 0.89 implies 89% duplication probability; do not report statistically meaningful differences without appropriate paired analysis.

## S13. Testing and quality assurance

**Unit cases UT-01…UT-16:** schema validation, duplicate IDs, technical token retention, deterministic normalization, embedding shape/normalization, cosine/FAISS agreement, unique top-K IDs, time gating, RRF tie behavior, self-match prevention, missing positive targets, metric fixture correctness, result serialization, fallback and CSV validation.

**Integration:** raw input to canonical store; duplicate graph to split builder; retrieval to evidence cards; experiment results to manifests; CSV upload to engine; engine to UI callback; exports round-trip safely.

**E2E:** known historical duplicate is retrievable; unrelated report yields sensible candidates or no-match state; malformed CSV never crashes UI; core CPU profile works; fresh Colab Run All reconstructs the system. Include negative-path security checks and regression cases for each resolved bug.

## S14. Error handling, reliability and security

Errors: bad/empty user text; missing source; malformed CSV and timestamps; unavailable model weights; Colab CPU/GPU variation; short/zero eligible corpus; reranker OOM; invalid cache; output export errors. The system should remain lexically searchable where possible if neural components fail, with visible degraded-mode label and audit record. Avoid blanket `except` and silent recovery.

Privacy/security: treat uploaded descriptions as untrusted data; never execute text/CSV contents; escape HTML render; neutralize spreadsheet formula injection on export; avoid printing sensitive raw reports in logs; do not embed credentials; take care with Gradio publicly shared links. BugTwin is a human-in-the-loop recommendation tool and cannot close issues automatically.

## S15. Performance and resource targets

Supported notebook profile: core CPU-compatible Colab environment, larger optional GPU experiments. Targets—not observations—include at least 10,000 validated candidate issues in the standard core corpus, median warm first-stage query under 2 seconds, and optional reranked query median under 10 seconds on a defined reference runtime. Record cold model loading separately. Measure embedding build time, TF-IDF/BM25 memory, float32 dense matrix/index size, GPU memory, p50/p95 query stages, warnings and failures; reduce batch size or corpus while preserving positive links when constrained.

## S16. Notebook internal structure

1. Overview and Run All instructions  
2. Dependency pinning and Colab environment inspection  
3. Typed configuration and manifest schema  
4. Dataset download/adapters and checksum  
5. Canonical data validator and duplicate graph  
6. EDA and timestamp/query population checks  
7. Chronological split and eligibility gate  
8. Text processing and lexical baselines  
9. Sentence embeddings and dense search  
10. Hybrid RRF and optional reranker  
11. Evidence and `BugTwinEngine` APIs  
12. Retrieval metrics, ablations and error analysis  
13. Unit/integration/regression tests  
14. Gradio interface and live examples  
15. Export, manifests, and limitations.

Everything is implemented in notebook cells; generated data/cache files are never treated as external source code. Expensive processing only happens in explicit run cells, not during class definitions.

## S17. Milestones, dependencies and gates

| Stage | Objective | Acceptance gate |
|---|---|---|
| M0 | Freeze BT-SPEC-001 and BT-ARCH-001 | requirements/roles/ADRs are coherent |
| M1 | Verify source archives and build canonical data + graph | real source labels/time assumptions validated |
| M2 | Baseline lexical search + tested evaluator | measured chronological retrieval scores |
| M3 | Dense/hybrid/reranker + ablations | same candidate protocol, validation-based selection |
| M4 | Working Gradio product + exports | end-to-end user workflow functional |
| M5 | Safety/reliability + clean Colab checks | required tests pass, fallback verified |
| M6 | Final `BugTwin.ipynb` release | Run All, manifest, documented evidence, no missing modules |

A milestone report must distinguish implemented, tested, verified, deferred, and blocked items. Do not claim a stage completed based on code generation alone.

## S18. Risk register and controlled open decisions

**Critical risks:** incomplete duplicate links; mismatch between published dataset statistics and actual usable records; data leakage through future candidates or post-edited text; approximate full-index time filtering; domain mismatch in the MS MARCO reranker; too-long bug reports and MiniLM token limits; Colab session eviction and memory pressure; untrusted upload data; public demo exposure. Each has a specific owner/control in BT-ARCH-001 §A14.

**Open pending M1/M3:** raw BugHub artifact format and license details; duplicate grouping policy; earliest accessible report snapshot; corpus sampling profile; exact dependency lock; actual supported latency; whether a neural reranker outperforms the no-rerank baseline; whether confidence thresholds are scientifically supportable.

## S19. Definition of Done

1. Exactly one maintained Colab-compatible `.ipynb` contains all app code.
2. Fresh runtime can build/rebuild data, models and indexes without secret keys.
3. Authentic source reports are loaded; schema and duplicate relations are audited.
4. TF-IDF/BM25, dense and hybrid methods are usable and benchmarked.
5. Historic evaluation filters candidate eligibility before ranking.
6. Scores, Hit@K, set Recall@K, MRR/MAP and runtimes are correctly labeled.
7. Gradio supports report input, ranked evidence, method selection and errors.
8. Required P1 review/export features function or are explicitly recorded as not delivered.
9. Unit/integration/end-to-end tests, security checks and CPU fallback pass.
10. Experiment and data/model provenance exports exist; observed failures/limitations are documented.
11. No invented model claims, duplicate probability percentages or benchmark outcomes.
12. M6 run-all verification is recorded with a real Colab environment manifest.

## S20. Architecture governance and sources

Changes to major algorithms, labels, test splits, UX decisions, storage or dependency policies require updated architecture decision record and affected test assertions. Versions are coordinated: `BT-SPEC-001 v1.1 ↔ BT-ARCH-001 v1.0`.

Sources and documentation checked 25 October 2026:

- LogPAI BugHub: https://github.com/logpai/bughub
- SentenceTransformers max sequence length: https://www.sbert.net/examples/sentence_transformer/applications/computing-embeddings/README.html
- SentenceTransformer model: https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2
- Candidate cross-encoder: https://huggingface.co/cross-encoder/ms-marco-MiniLM-L6-v2
- Gradio public sharing: https://www.gradio.app/guides/sharing-your-app

**Final standing:** This package authorizes implementation planning. It does not certify dataset parsing, model performance, working UI, or Colab run-all results. Those verifications occur in M1-M6.
