# Geo Facts Extractor (Offline)

An LLM-free pipeline to mine **structured geological facts** from PDFs:
ages (Ma/Ga), assays (ppm/%/g/t), oxides, and **tectonism events (D1–D4)**.
It uses regex + dictionaries + heuristics (no internet/API), then applies
confidence gating, normalisation, deduplication, and **attaches provenance**
(section title, citations, figure/table IDs). Outputs a master CSV/Parquet
and an Excel workbook with per-category sheets.

**Highlights**
- Offline, deterministic (Colab/Notebook-friendly)
- Unit normalisation (Ma/Ga/ppm/%/g/t, etc.)
- Confidence scoring + clean/review/drop splits
- Dedup with unit-aware tolerances (e.g., ±2 Ma)
- Provenance: `PDF_NAME#p{page}:{block_id}`, section titles, citations
- Tectonism detection: D1–D4 stages and common structural terms

## Pipeline 
1. **PDF windowing** – pick useful pages; skip front matter/refs.
2. **Block extraction** – sentences/table rows/captions with IDs.
3. **Candidate finder** – geo keywords + numeric patterns (±, ranges, units).
4. **Structured facts** – classify (geochronology/assay/… + D1–D4), tag context.
5. **Packaging** – per-category CSVs + lookups (ages/assays) + catalog JSONL.
6. **Noise gating** – penalties/bonuses → `conf_final`; clean/review/drop.
7. **Provenance** – section titles, author-year cites, figure/table IDs.

## Quickstart
- Put your PDF at `/content/YourThesis.pdf`, set `PDF_PATH`.
- Run notebooks/cells in order: Steps 1 → 8.
- Find outputs in `/content/step5_offline`, `/content/facts_step6_*`,
  `/content/facts_step8_provenance_offline.csv`.




