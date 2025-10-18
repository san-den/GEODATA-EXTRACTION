# CET Geodata Extraction (Page-Aware + LLM)

[![Python](https://img.shields.io/badge/python-3.10%2B-blue.svg)](#)
[![Colab](https://img.shields.io/badge/Run%20in-Colab-brightgreen.svg)](#)
[![Model](https://img.shields.io/badge/model-gpt--4o--mini-8A2BE2.svg)](#)

Extract **structured geology data** (no coordinates) from thesis PDFs into **page-scoped JSON** using a page-aware text pass and a constrained LLM. Every record keeps **page provenance** (`pdf_name + page_number`) for traceability.

---

## Approaches in this repo

### Final (default): Schema-first, page-aware LLM
- Clean text **one page at a time** (no global re-split), stop at **References**.
- LLM returns **strict JSON** per page (fixed schema: metadata, geology, geochronology, geochemistry, metallogeny).
- Post-steps: **unit normalization** (e.g., Ga→Ma), **dedup**, optional CSV export.

### Earlier variant (kept): Hybrid **regex + LLM**
- **Regex/dictionary pass** finds likely facts (ages, assays, methods, rock terms) per page.
- Only those **candidate snippets** go to the LLM, which **validates & structures** them into the **same schema**.
- Helpful for table-heavy PDFs and for **token/cost control**.

> Both variants are included. Use the schema-first pipeline by default; run the hybrid notebook/script to reproduce the presentation results.

---

## Why LLM vs regex?

- **Robust to phrasing & layout** (free-form prose and varied wording).
- **Schema-first output** (JSON-only → easy validation & DW/graph loading).
- **Less rule maintenance** (keep small validators for ranges/units/dedup).
- **Built-in provenance** via page-level extraction.

Regex still adds value as a **candidate generator** (hybrid mode) and **post-validator**.

---

## What changed from the original rules pipeline?

**Before:** Rules over full text → OK for rigid patterns, weak on multi-sentence context and author style drift.  
**Now:** **Page-aware LLM** with fixed schema → better recall on narrative geology, simpler maintenance, deterministic validation after the model.

---

## Pipeline (default schema-first LLM)

1. **Setup & ingestion** – install libs, read OpenAI key from Colab Secret **`sandra`**, move PDFs into `/content/pdfs`.
2. **Stage 2 – Page-aware text** – clean per page, drop Abstract on first pages, stop at References, **no re-split**.
3. **Collect pages** – build `all_docs_pages = {pdf_name: [(page, text), ...]}`.
4. **Stage 3 – LLM extraction** – send each page to **gpt-4o-mini**; receive JSON per page (temperature ~0.0–0.1).
5. **Outputs** – one JSON per PDF in `/content/extracted_json/<stem>.extracted.json`; optional CSV flatteners.

### Running the hybrid regex + LLM
Open the hybrid notebook/script committed in the repo (e.g., `notebooks/hybrid_regex_llm.ipynb`).  
It performs the regex candidate pass, then calls the same schema LLM. Outputs are saved beside the default pipeline for side-by-side comparison.

---

## Quick start (Colab)

1. Upload PDFs to `/content/` or `/content/pdfs/`.  
2. In **Colab → Secrets**, create **`sandra`** with your OpenAI API key.  
3. Run the notebook top-to-bottom (schema-first).  
   - To run the hybrid, execute the hybrid notebook/script as well.

**Tip:** keep page text ≤ ~5,500 chars, temperature 0.0–0.1, and cache responses.

---

## Outputs

- `/content/extracted_json/<stem>.extracted.json` — list of per-page JSON objects following the schema.  
- Optional CSV projections per category (geology, geochronology, geochemistry, metallogeny).

---

## Notes

- **Scanned pages** (no selectable text) come back empty with `pdfplumber`; add OCR in a follow-up.  
- Convert **Ga → Ma** after extraction or enforce in the prompt.  
- Table-heavy pages benefit from the **hybrid** path or a future table/OCR agent.

---

## Trade-offs & mitigations

- **Cost/latency:** page-level API calls → clip long pages, cache, consider hybrid to reduce tokens.  
- **Determinism:** low temperature, fixed model version, JSON schema validators.  
- **Tables/figures:** plain text only; add table/OCR to lift geochem recall.

---


