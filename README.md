# BugTwin

Semantic duplicate bug-report search, reproducible historical data auditing, and baseline NLP retrieval in a single Colab notebook.

**Application code:** [`BugTwin.ipynb`](BugTwin.ipynb) — the repository's only application source file.

## Capabilities

- Provenance-aware ingestion of the public Eclipse Platform bug-report corpus from [LogPAI BugHub](https://github.com/logpai/bughub).
- Canonical issue records, duplicate-link audits, timestamp checks, and quarantining of invalid rows.
- Chronologically safe dataset partitions and verified direct-duplicate evaluation queries.
- TF-IDF and BM25 historical candidate retrieval with fixed training-derived corpus statistics.
- Gradio interface for searching existing reports, plus baseline evaluation summaries.
- Automated regression tests embedded in the notebook and executed in CI using synthetic fixtures.

## Run in Google Colab

1. Open `BugTwin.ipynb` in [Google Colab](https://colab.research.google.com/).
2. Select **Runtime → Run all**. The notebook installs the runtime dependencies itself.
3. The default source mode is `bughub`; ingestion requires internet access and a successful source-schema audit.
4. Review the source manifest and data quality gates. Then use the UI at the end of the notebook.
5. In Colab, a public Gradio share link is enabled by default **only for the public BugHub dataset or the built-in synthetic fixture**. Anyone with that URL may access the application while it runs. Set `BUGTWIN_SHARE_UI=0` if you do not want a public link; note that a local-only interface may not open in Colab. Local source mode never enables public sharing by default.

The default benchmark selects up to the earliest 12,000 source reports to keep CPU/RAM costs manageable. The M2 benchmark uses its own chronological split and frozen training vocabulary. These results are **not** full-corpus BugHub benchmark scores. Increase the limit only after verifying available memory and processing time.

## Offline / CI fixture mode

```powershell
$env:BUGTWIN_M1_SOURCE = 'fixture'
$env:BUGTWIN_SKIP_INSTALL = '1'
$env:BUGTWIN_LAUNCH_UI = '0'
python -m pip install -r requirements-ci.txt
python -c "import nbformat; from nbclient import NotebookClient; n=nbformat.read('BugTwin.ipynb',as_version=4); NotebookClient(n,timeout=180,kernel_name='python3').execute(); print('PASS')"
```

Fixture data tests the implementation, not the quality of a model on real bug reports. Do not report fixture metrics as scientific results.

## Reproducibility and licensing

- The public data is downloaded on demand and is **not committed** to this repository.
- Confirm the original source's research-use terms before using or redistributing results or source records.
- The M1 manifest records the pinned Git commit and SHA-256 hashes of downloaded artifacts.
- Queries with no verified direct historical duplicate link are classified as **unknown**, not negatives.
- Original issue resolution metadata is excluded from ranking features.
- Neural embeddings, hybrid retrieval and reranking are subsequent milestones; this repository currently implements lexical baselines.

## Development and GitHub workflow

See [docs/GITHUB_WORKFLOW.md](docs/GITHUB_WORKFLOW.md) for branch, commit, push, pull request and merge instructions.

## Documentation

- [Master specification](docs/BT-SPEC-001.md)
- [Architecture and stakeholder maps](docs/BT-ARCH-001.md)
- [M1 verification report](docs/M1-verification.md)
- [Implementation checkpoint](docs/IMPLEMENTATION_STATUS.md)
