# Dataset card — Protein–protein interactions in DNA repair dataset

## Dataset title

Protein–protein interactions in DNA repair (v0.1.0)

## Dataset summary

Tabular collection of experimentally reported physical protein–protein interactions (PPI) involving proteins known to participate in DNA damage recognition, repair, checkpoint control, and genome stability. Each record links two interacting proteins via stable UniProt identifiers, includes the detection method (PSI‑MI controlled vocabulary), the original source database identifier, and—when available—the DNA repair pathway annotation derived from REPAIRtoire. 

## Scientific task

Support construction of a focused interaction network and identification of key proteins or interaction modules that may be important for DNA repair and cancer-related mechanisms.

## Record unit

One row = one experimentally reported physical interaction between two proteins that are involved in DNA damage recognition, repair, checkpoint control, or genome stability. The interaction must be directly supported by a publication and curated by a trusted source database.

## Data sources

Defined in `specs/source_map.json`: journal PDFs, supplementary tables, aptamer databases, metadata aggregators, GitHub releases, and optional ML dataset exports (with license review).

## Data extraction procedure

1. PDF: `scripts/extract_pdf.py` guided by `specs/pdf_extraction_manifest.json`
2. Web: `scripts/extract_web.py` guided by `specs/web_extraction_manifest.json`
3. Logs: `data/extracted/extraction_log.jsonl`

## Data cleaning and normalization

`scripts/build_dataset.py` merges extracts; `scripts/clean_dataset.py` normalizes sequences, units (to nM), missing values, and deduplicates per `specs/cleaning_pipeline.json`.

## Dataset schema

Field definitions, types, and examples: `specs/dataset_schema.json`. Final columns in `data/processed/dataset.csv`.

## Validation

Rules in `specs/validation_rules.json`; checks via `scripts/validate_project.py` and `tests/test_required_artifacts.py`.

## Known limitations

- Pathway annotations (protein_a_repair_pathway, protein_b_repair_pathway) are based on REPAIRtoire (or a static mapping file) and may not capture all possible pathway assignments for a given protein.
- The interaction_context field is rarely present in primary databases and will often be empty.
- Confidence scores (interaction_confidence) are kept in original database scales (e.g., IntAct MIscore) and are not normalised across sources.
- Only physical interaction types are retained; genetic or functional interactions are excluded by design.
- Example rows and fields are illustrative examples and are not directly verified against live sources or overall example structure.

## Recommended use

- Building a focused PPI network for DNA repair proteins.
- Identifying highly connected (“hub”) proteins or interaction modules in repair pathways.

## Not recommended use

- Meta‑analysis without re‑verifying the primary literature (due to possible curation errors or missing context).
- Any application that requires complete coverage of all known DNA repair interactions (the dataset is a focused subset).

## TODO: License

See `LICENSE` — replace placeholder before publication (e.g. CC-BY-4.0 or CC0-1.0 subject to upstream data licenses).

## Citation

See `CITATION.cff`.