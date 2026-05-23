# Practice 1 — Record definition and dataset schema

> Replace template text with your project decisions. Keep this report aligned with `project.json` and `specs/dataset_schema.json`.

## Topic

Aptamer–protein binding dataset (example template).

## Scientific task

Collect experimentally reported aptamer–protein binding measurements for comparing affinity values and assay conditions across sources.

## One-record definition

**One record** = one experimentally reported aptamer–protein binding measurement from a specific source (one row in `data/processed/dataset.csv`).

## Examples of records

| Example | Why it counts |
|---------|----------------|
| Kd = 0.5 nM for sequence GGTTGGTGTGGTTGG vs thrombin from Table 2 in Green 2018 | Single measurement + sequence + target + source |
| IC50 from supplementary table for one aptamer–lysozyme pair | One numeric binding outcome tied to one pair |

## Non-record examples

| Example | Why it is not a record |
|---------|-------------------------|
| General review paragraph on SELEX without numeric binding data | No measurement |
| Full list of 50 sequences without per-sequence affinity | Not one measurement per row (unless split) |
| Predicted docking score without experimental citation | Out of scope if only experimental data allowed |

## Dataset fields

List each schema field and how you will populate it. Update `specs/dataset_schema.json` when fields change.

| Field                      | Data type | Is required | How is populated                                                                                                                                                                                |
|----------------------------|-----------|-------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `record_id`                | String    | Yes         | Generated automatically: rec_{protein_a_gene}_{protein_b_gene}_{source_db}_{source_db_id}. <br/> Must be unique.                                                                                |
| `source_db`                | String    | Yes         | Taken from the source database name. Controlled vocabulary: BioGRID, IntAct, Reactome, etc.                                                                                                     |
| `source_db_id`             | String    | Yes         | Directly copied from the interaction identifier in the original database <br/> (e.g., BioGRID interaction ID, IntAct accession).                                                                |
| `protein_a_id`             | String    | Yes         | UniProtKB AC from the source. If missing, map via UniProt ID mapping service <br/> using gene symbol or other identifiers.                                                                      |
| `protein_b_id`             | String    | Yes         | UniProtKB AC from the source. If missing, map via UniProt ID mapping service <br/> using gene symbol or other identifiers.                                                                      |
| `protein_a_gene`           | String    | No          | HGNC symbol – either taken from source or added via UniProt mapping.                                                                                                                            |
| `protein_b_gene`           | String    | No          | HGNC symbol – either taken from source or added via UniProt mapping.                                                                                                                            |
| `protein_a_repair_pathway` | String    | No          | From REPAIRtoire. Use the eight controlled terms: BER, NER, HRR, NHEJ, MMR, TLS, DDS, DDR. Multiple pathways separated by `\|`. Leave empty if not annotated.                                   |
| `protein_b_repair_pathway` | String    | No          | From REPAIRtoire. Use the eight controlled terms: BER, NER, HRR, NHEJ, MMR, TLS, DDS, DDR. Multiple pathways separated by `\|`. Leave empty if not annotated.                                   |                                                                                                                                              |
| `interaction_type`         | String    | Yes         | Use PSI‑MI term from source (e.g., MI:0407 for direct interaction). If source only says “physical association” use MI:0915.                                                                     |
| `detection_method`         | String    | Yes         | PSI‑MI term for the experimental method. Keep only physical methods (co‑IP, affinity chromatography, FRET, crystallography, etc.).<br/> Exclude genetic, computational, or text‑mining methods. |
| `interaction_confidence`   | Float     | No          | If source provides a numeric score (e.g., IntAct MIscore, BioGRID score), keep as float. If not available, is left null.                                                                        |
| `pmid`                     | Integer   | Yes         | Integer from the source. If a record has no PubMed ID, it is excluded.                                                                                                                          |
| `taxid_a`                  | Integer   | Yes         | NCBI Taxonomy ID. Default for human = 9606. For other organisms, use ID from source. Required. Is specified, since PPI databases also provide different species' interactions                   |
| `taxid_b`                  | Integer   | Yes         | NCBI Taxonomy ID. Default for human = 9606. For other organisms, use ID from source. Required. Is specified, since PPI databases also provide different species' interactions                   |
| `interaction_context`      | String    | No          | Free text extracted from publication or source annotation (e.g., cell line, tissue, treatment). If not available, is left empty.                                                                |
| `record_notes`             | String    | No          | Curator notes. Examples: “Pathway from REPAIRtoire mapping”, “Merged from BioGRID and IntAct – same PMID and method”, “Confidence score not provided”.                                          |

## Ambiguous cases

Document decisions here, for example:

- Multiple Kd values for the same aptamer under different buffers → separate records or one record with notes?
- Range reported as “0.1–1 nM” → store null + note, or midpoint?
- Duplicate sequence in paper and database → deduplication rule in Practice 5.
