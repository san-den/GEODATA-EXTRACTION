# CET Geodata Extraction (Page-Aware + LLM)

[![Python](https://img.shields.io/badge/python-3.10%2B-blue.svg)](#)
[![Colab](https://img.shields.io/badge/Run%20in-Colab-brightgreen.svg)](#)
[![Model](https://img.shields.io/badge/model-gpt--4o--mini-8A2BE2.svg)](#)

This project extracts **structured geology data** (no coordinates) from thesis PDFs into **page-scoped JSON** using a page-aware text pass and an LLM. Every record carries **page provenance** for traceability.

---

## Why LLM vs regex?

- **Robust to phrasing & layout:** LLMs handle free-form prose and variations (“U–Pb zircon dating indicates 2.1 Ga” vs “2.1 billion years (U-Pb)”) that brittle regex rules miss.
- **Schema-first output:** We force **strict JSON** (no free text), making validation and loading into DW/graph trivial.
- **Less rule maintenance:** New theses/styles need fewer regex tweaks. We still keep **light validators** (units, ranges, dedup) as guardrails.
- **Per-page provenance by design:** The model only sees one page at a time, so extracted facts are naturally traceable to `pdf_name + page_number`.

> We still use small regex/heuristics post-steps (e.g., unit normalization, sanity checks), but the LLM does the heavy lifting on heterogeneous prose.

---

## What changed (from the previous regex pipeline)

**Before:** Regex + dictionaries over full text; good for rigid patterns (assays, explicit ages), struggled with multi-sentence context and style drift.

**Now:** **Page-aware LLM extraction** with a fixed schema. Each page is cleaned, sent to the model, and returned as JSON. Post-processing applies **unit normalization** (e.g., Ga→Ma), **de-duplication**, and CSV export. The result is higher recall on prose and simpler maintenance, while keeping deterministic validation steps.

---

## Pipeline
1. **Setup & ingestion** – install libs, read OpenAI key from Colab Secret (`sandra`), move PDFs into `/content/pdfs`.
2. **Stage 2 – Page-aware text** – clean per page, drop Abstract on first pages, stop at References, **no re-split**.
3. **Collect pages** – build `all_docs_pages = {pdf_name: [(page, text), ...]}`.
4. **Stage 3 – LLM extraction** – page-referenced JSON against a fixed schema.
5. **Outputs** – one JSON per PDF in `/content/extracted_json/<stem>.extracted.json`.

---

## Quick start (Colab)
1. Upload PDFs to Colab `/content/` or directly to `/content/pdfs/`.
2. In **Colab → Secrets**, add your OpenAI key with name **`sandra`**.
3. Run the notebook top-to-bottom.

**Tip:** Set the model via code (`gpt-4o-mini`) and keep `temperature≈0.0–0.1` for stable outputs.

---

## Outputs
- `/content/extracted_json/<stem>.extracted.json` — list of per-page JSON objects following the schema.

---

## Notes
- If a page is scanned (no selectable text), `pdfplumber` returns empty text; add OCR later if needed.
- For ages reported in **Ga**, add a post-step to convert to **Ma** (×1000) or strengthen the prompt rule accordingly.

---

## Optional extras
- **Doc-level metadata pass** (pages 1–3) to attach `title/author/year/institution` to all page records.
- **Age normalization** (Ga → Ma) after Stage 3 (heuristic: if 1–10 and “Ga” on page, ×1000).
- **Flattening & DW/Graph export** (dim/fact), preserving `pdf_name` and `page_number` as provenance keys.
- **Gold-set evaluation**: 5–10 labeled pages per PDF to compute P/R/F1.

---

## Trade-offs & mitigations
- **Cost/latency:** API calls per page → keep temperature low, cap page text length, and cache results.
- **Determinism:** Not 100%; mitigate with low temperature, fixed model version, and schema validators.
- **Tables/figures:** Plain text only; add table agent/OCR to improve geochem recall on scanned or tabular pages.

