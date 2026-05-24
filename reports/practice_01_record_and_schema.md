# Practice 1 — Record definition and dataset schema

## Topic

Protein–protein interactions in DNA repair dataset

## Scientific task

Collect experimentally reported interactions between proteins involved in DNA damage recognition, repair, checkpoint
control, and genome stability.

## One-record definition

One **record** = one experimentally reported aptamer–protein binding measurement from a specific source (one row
in `data/processed/dataset.csv`). See `project.json` and `reports/practice_01_record_and_schema.md`.

Each record corresponds to one row in data/processed/dataset.csv and contains at least:

- UniProtKB identifiers for both proteins
- Interaction type (PSI‑MI controlled vocabulary)
- Detection method (PSI‑MI)
- Source database and its original interaction ID
- PubMed ID of the supporting publication

## Examples of records

| Example                                                                                                                                                                                                                                                                                                                                                                                  | Why it counts                                                                                                                                                                                                                                                |
|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| XRCC1 (P18887) – APEX1 (P27695) interaction from IntAct (EBI-4566492), detected by anti‑bait co‑immunoprecipitation (MI:0005), PMID: 19934257, taxid: 9606/9606, interaction type: physical association (MI:0915), confidence score: 0.4 (MI Score). Both proteins annotated to BER pathway via REPAIRtoire.                                                                             | Complete record with all required fields (protein_a_id, protein_b_id, source_db_id, detection_method as PSI‑MI term, pmid, taxid). Includes optional confidence score. Pathway context (BER) can be added via mapping.                                       |
| XRCC1 (P18887) – APTX (Q7Z2E3‑9 isoform) from IntAct (EBI-11161000), detected by anti‑tag co‑immunoprecipitation (MI:0007), PMID: 26496610, DOI: 10.1016/j.cell.2015.09.053, taxid: 9606/9606, interaction type: association (MI:0914) (close to physical association), confidence score: 0.35 (MI Score). XRCC1 belongs to BER pathway; APTX is involved in single‑strand break repair. | Valid record despite use of a splice variant for APTX. The interaction is experimentally validated with a physical method. Isoform identifier is retained as provided; optional normalisation to canonical UniProt AC (Q7Z2E3) can be noted in record_notes. |

Both records meet inclusion criteria: direct physical interaction, experimental detection method (co‑IP), traceable
source database ID, PubMed ID, and organism identifiers.

## Non-record examples

| Example                                                                                                                                                                                       | Why it is not a record                                                                                                                                                                                                                                                                                                                                                    |
|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| NPM1 (P06748) – XRCC1 (P18887) from IntAct (EBI-58762356), detected by proximity‑dependent biotin identification (BioID), PMID: 39251607, interaction type: proximity.                        | Detection method (proximity labeling) does not demonstrate direct physical binding; it only indicates spatial proximity (<20 nm) in the cell. According to inclusion criteria, only methods that directly measure physical interaction (e.g., co‑immunoprecipitation, affinity chromatography, FRET, crystallography) are accepted. Proximity‑based methods are excluded. |
| A BioGRID entry that lists only “Physical Association” but does not provide a specific detection method (e.g., “Affinity Capture‑MS” missing) and has no interaction confidence or PubMed ID. | Missing required fields: detection_method (must be a physical method PSI‑MI term) and pmid.                                                                                                                                                                                                                                                                               |
| “ATM and ATR function together in the DNA damage response” from a review article, no experimental method, no PubMed ID for the specific interaction, no source database ID.                   | No experimental detection method; no source_db_id; not a curated physical interaction record.                                                                                                                                                                                                                                                                             |

## Dataset fields

List of each schema field and how it is populated, see `specs/dataset_schema.json` for data schema.
TODO: указать для всех чёткий формат и способы валидации

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
 - Interaction between Protein A (human) and Protein B (mouse) – different taxids.   
Keep record. Include both taxid values (e.g., 9606 and 10090). As long as the interaction is experimentally shown (e.g., human protein expressed in mouse cells), it qualifies. Add note in record_notes: “cross‑species interaction”.
- Multiple detection methods for the same protein pair listed as separate entries in the source database.   	
Keep separate records (each with its own source_db_id). Do not merge. In final dataset for network construction, can be deduplicated by (protein_a_id, protein_b_id, pmid) if needed.
- Same interaction reported by two different databases (e.g., IntAct and BioGRID) with different interaction_type terms (e.g., “direct interaction” vs. “physical association”).   
Keep both records. Do not overwrite. In network analysis, the more specific term (direct interaction) may be preferred, but the dataset retains both.
- Confidence score is provided as a range (e.g., “0.7–0.9”) or text (“high”).    	
Store null in interaction_confidence. Record the range or text in record_notes. Do not convert to numeric.
- The protein is annotated with multiple DNA repair pathways in REPAIRtoire (e.g., BER and NER).
Use pipe separator: BER|NER. Both pathways are stored as a single string.
- The source database provides an interaction, but the protein identifiers are not UniProt AC (e.g., Entrez Gene ID).   
   Map to UniProt AC using UniProt ID mapping service. Store original identifier in record_notes if desired. The record is still valid after normalisation.
- The same protein pair with same detection method and same PMID appears twice in the same source database but with different source_db_id (possible duplicate curation).   
Keep both entries unless identical in all mandatory fields. If identical, deduplicate and note in record_notes. 
- Different interaction types for the same protein pair across different sources – e.g., IntAct says “direct interaction” (MI:0407), BioGRID says “physical association” (MI:0915) (or same source gives different records on interaction type).   
Keep both records. Do not overwrite one with the other. When aggregating for network analysis, the more specific term (direct interaction) may be preferred, but the raw dataset must retain both to maintain provenance. Document the discrepancy in record_notes if desired (e.g., “direct interaction vs. physical association”).