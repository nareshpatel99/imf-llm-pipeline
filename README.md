# IMF Staff Report to Dashboard: LLM extraction, RAG and evaluation

A single notebook that reads an IMF country report (PDF) and turns it into structured JSON, a grounded question-answering index and a four-panel dashboard. Every step that uses a large language model is checked by a second model and by plain code.

**Report analysed:** IMF Country Report No. 24/340, *Seychelles: Third Reviews Under the EFF and RSF Arrangements and Request for Rephasing of Access under the RSF* (December 2024).

## What the notebook does

| Stage | Method | Output |
|---|---|---|
| PDF processing | `pypdf` extraction, statistical header/footer removal, paragraph rebuild, section detection, token-aware chunks (450 tokens, 50 overlap, `tiktoken`) | `outputs/json/sections.json`, `chunks.json` |
| Summaries | one summary per section (under 100 words, checked by code), executive summary bullets | `section_summaries.json`, `executive_summary.json` |
| Judge | a second model scores consistency, completeness and factual alignment; numbers are also checked against the source by code | `summary_judgements.json` |
| Themes | seven macro-financial themes counted by a keyword lexicon and by LLM chunk labels, compared with Spearman correlation | `theme_distribution.json` |
| Insights | strengths and vulnerabilities with quotes verified in the source text | `qualitative_insights.json` |
| Indicators | core indicator JSON in the target schema, hybrid (dense + keyword) retrieval, code checks, `null` fallbacks with reasons, judge audit | `core_indicators.json`, `core_indicators_detailed.json`, `indicator_audit.json` |
| Time series | table rows parsed from the PDF and verified on their page | `temporal_series.json` |
| RAG | `sentence-transformers/all-MiniLM-L6-v2` embeddings in a FAISS `IndexFlatL2`, answers with chunk citations | `rag_results.json` |
| Extension | Multi-Query RAG (query variants + reciprocal rank fusion) compared with single-query RAG under the same judge | `rag_comparison.json` |
| Dashboard | `pd.melt` to long format, four panels plus an executive text panel, 300 dpi | `outputs/figures/*.png` |

## How to run

**Google Colab (recommended).** Open `IMF_LLM_Pipeline.ipynb` in Colab and choose *Runtime > Run all*. If `CONFIG["repo_url"]` is set, the notebook clones this repository so `data/` and `cache/` are available; otherwise upload `data/seychelles_cr24340.pdf` to the Colab file panel first.

**Locally.**

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook IMF_LLM_Pipeline.ipynb
```

## LLM access and caching

Every model response is stored in `cache/llm/`, keyed by a SHA-256 hash of the model, system instruction, prompt, schema and temperature. The cache is part of the repository, so a rerun replays the same answers without any API key and reproduces the reported numbers.

When an answer is not cached, the notebook picks a backend automatically:

1. `GEMINI_API_KEY` in Colab Secrets or the environment: Gemini API.
2. Inside Colab without a key: Colab's built-in Gemini access (`google.colab.ai`). The shipped cache was produced this way, with Gemini 3.7 Flash as the extractor and Gemini 3.8 Flash as the judge.
3. Outside Colab: the Antigravity CLI (`agy`) if installed.

No key is stored in the notebook or the repository.

## Repository layout

```
IMF_LLM_Pipeline.ipynb   the notebook
data/                    the IMF report PDF
cache/llm/               cached model responses
cache/rag/               FAISS index
outputs/json/            all machine-readable outputs
outputs/figures/         dashboard and individual charts (300 dpi)
requirements.txt
LICENSE
```

## Licence

Code is released under the MIT Licence (see `LICENSE`). The IMF report is © International Monetary Fund and is included only so the notebook can be reproduced.
