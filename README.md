# BugTwin

A compact single-notebook NLP project for finding potentially duplicated software bug reports.

## Run in Google Colab

Open `BugTwin.ipynb` and choose **Runtime → Run all**. The notebook installs Gradio and SentenceTransformers, loads 30 built-in fictional issue reports, compares TF-IDF and MiniLM, and launches a search interface. A Google Colab internet connection is needed the first time MiniLM is downloaded. No API key required.

## What it demonstrates

- TF-IDF + cosine similarity: traditional lexical retrieval.
- SentenceTransformer MiniLM + cosine similarity: semantic retrieval.
- Top-5 ranked matches and six illustrative Hit@3 evaluation queries.
- Gradio input form and results table.

**Limitations:** The embedded corpus and labeled examples are handcrafted demonstration data, not an external validation benchmark. Similarity is not a calibrated duplicate probability. This project offers human-review candidates, not automatic closure of bug tickets.

## Source and application structure

All executable application logic is in `BugTwin.ipynb`. The `.github` workflow runs a minimal offline TF-IDF smoke check without downloading MiniLM. Actual transformer inference must be verified in Colab.

## Use with your own data

Replace the `reports` DataFrame in the dataset cell with a CSV having `id`, `title`, and `description` columns, then rerun all cells. Avoid sensitive reports if using the public Gradio share link.
