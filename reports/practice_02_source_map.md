# Practice 2 — Source map

## Source search strategy

- Primary databases: IntAct (via PSICQUIC), REPAIRtoire (HTML scraping), Reactome (REST API), UniProt/HGNC/NCBI (REST APIs).
- Search engines / platforms: PubMed for literature validation; ProteomeXchange for supplementary proteomics data.
- Keywords for initial seed list:
“DNA repair” + “protein-protein interaction” + “human”
“BER”, “NER”, “HRR”, “NHEJ”, “MMR”, “TLS”, “DDS”, “DDR” (REPAIRtoire pathways).

## Source groups

### Scientific papers

The current source map does not explicitly include a scientific_papers group.
However, primary literature is an essential indirect source.
The databases described in the databases group (e.g., IntAct, BioGRID) derive their content from publications, making
the original research articles a primary source for establishing confidence in individual interactions.
As a rule, each publications provides a single interaction fact, including interaction_context, precise conditions that
are not available for parsing but rather are retrieved by simply reading a paper.

### Supplementary materials

The supplementary materials group includes two sources, both proteomics datasets from ProteomeXchange. They provide
experimental interaction evidence not yet fully captured in primary interaction databases, and are therefore valuable
for adding novel protein pairs.
- **DDRAM** (supp_ddram): A multi‑conditional interaction network from 2023 that includes 336 proteins and 109 protein
assemblies. The data are available through ProteomeXchange (PXD037494) and are described by a DOI (
10.1016/j.cels.2023.04.007). The source is expected to contribute protein_a_id, protein_b_id, interaction_type,
interaction_confidence, and pmid fields. Data format is proteomics mass spectrometry (mzIdentML, mzTab).
- **Bromodomain proteins in DNA repair** (supp_pxd016673): A smaller‑scale affinity purification (AP‑MS) dataset
investigating 24 human bromodomain proteins relevant to DNA repair. Data are accessible via ProteomeXchange (PXD016673)
and are covered by DOI 10.1101/gad.331231.119. The source contributes protein_a_id, protein_b_id, detection_method, and
pmid fields.

### Datasets

The databases group forms the core of the dataset, providing both the primary interaction records (from IntAct) and the
mapping tables needed to normalise identifiers (UniProt, HGNC, NCBI Taxonomy) and to annotate DNA repair pathways (
REPAIRtoire, Reactome).
- **IntAct** (db_intact): A manually curated molecular interaction database, accessible via PSICQUIC, FTP, or REST API. It
is the primary source for interaction records, supplying source_db, source_db_id, protein_a_id, protein_b_id,
protein_a_gene, protein_b_gene, interaction_type, detection_method, interaction_confidence, pmid, taxid_a, taxid_b.
- **REPAIRtoire** (db_repairtoire): Provides DNA repair pathway annotations for individual proteins. The source supplies
protein_a_gene, protein_a_repair_pathways, and protein_a_id. This mapping is used to populate the
protein_a_repair_pathways and protein_b_repair_pathways fields in the final schema.
- **Reactome** (db_reactome): A curated pathway knowledgebase that provides both pathway membership and interaction
context. It supplies protein_a_id, protein_b_id, interaction_type, and interaction_context. It is also used as an
alternative source for pathway annotations.
- **UniProt** (db_uniprot): Provides identifier normalisation services. It supplies protein_a_id, protein_a_gene, and
taxid_a. Importantly, UniProt is used to convert gene symbols, Entrez IDs, or other identifiers to canonical UniProtKB
accession numbers.
- **HGNC** (db_hgnc): Supplies official gene symbols (protein_a_gene). It ensures that gene names are standardised to the
HGNC nomenclature.
- **NCBI Taxonomy** (db_ncbi_taxonomy): Supplies taxid_a. This is used to ensure that organism identifiers are
standardised to NCBI Taxonomy IDs (e.g., 9606 for Homo sapiens).

### Aggregators

The aggregators group provides secondary sources that aggregate data from multiple primary resources. These are
primarily used for cross‑validation, enrichment, and confidence scoring, rather than as primary sources for interaction
records.
- **Pathway Commons** (agg_pathway_commons): Integrates pathway data from multiple sources including Reactome, KEGG, and
others. It supplies protein_a_id and protein_a_repair_pathways, complementing REPAIRtoire annotations. Its main added
value is as a source of alternative pathway membership information, which is useful for conflict resolution and coverage
extension.
- **HitPredict** (agg_hitpredict): Aggregates PPI data from IntAct, BioGRID, and HPRD and assigns reliability scores to
each interaction. It supplies protein_a_id, protein_b_id, interaction_confidence, detection_method, and pmid. The
confidence scores from HitPredict are distinct from the per‑source scores available in IntAct (MIscore) or BioGRID,
offering an independent reliability metric. This source is particularly useful for filtering high‑confidence
interactions or for weighting interactions during network construction.

## Priority sources
1. IntAct (db_intact) – Primary source for experimental PPIs. Provides all core fields: protein_a_id, protein_b_id, interaction_type, detection_method, pmid, taxid, interaction_confidence. Highest yield.  
2. REPAIRtoire (db_repairtoire) – Essential for pathway annotation (protein_a_repair_pathways). Controlled vocabulary (BER, NER, HRR, etc.). Medium yield, high specificity.  
3. UniProt (db_uniprot) & HGNC (db_hgnc) – Normalisation of protein identifiers to canonical UniProt AC and HGNC symbols. Not interaction sources, but required for merging.  
4. Reactome (db_reactome) – Adds interaction_context and alternative pathway membership. Enrichment only.  
5. NCBI Taxonomy (db_ncbi_taxonomy) – Supplies taxid_a / taxid_b for organisms. Easily obtained via mapping.  
6. HitPredict (agg_hitpredict) – Independent confidence scores for validation; no novel PPIs.  
7. Pathway Commons (agg_pathway_commons) – Alternative pathway annotations (redundant with REPAIRtoire/Reactome).  
8. Supplementary proteomics datasets (supp_ddram, supp_pxd016673) – Novel PPIs not yet in primary databases. Lowest priority due to extraction effort and small additional yield.  
9. Execution order: IntAct → REPAIRtoire → UniProt/HGNC normalisation → Reactome enrichment → supplementary datasets.

## Access conditions

For each source, access conditions are summarised below, including paywalls, registration requirements, API key needs,
and institutional access requirements. The access_status, access_method, and access_date for each source are recorded in
the source map (version 0.2.0).

| Source                               | Paywall                                                                                   | Registration / API Key                                                                           | Access Date                                                                |
|--------------------------------------|-------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------|
| IntAct                               | Open access; no paywall                                                                   | None; free public API via PSICQUIC                                                               | PSI‑MITAB (tab‑separated), PSI‑MI XML                                      |
| BioGRID                              | Open access; no paywall                                                                   | API key required for REST API (free academic registration at https://webservice.thebiogrid.org/) | MITAB, PSI‑XML, TAB                                                        |
| REPAIRtoire                          | Open access; no paywall                                                                   | Web scraping                                                                                     | None; direct HTML parsing                                                  |
| Reactome                             | Open access; no paywall (license: CC‑BY‑4.0 for API; FTP access may require paid license) | REST API / BioPAX / SBML                                                                         | None for REST API; no API key required                                     |
| UniProt                              | Open access; no paywall                                                                   | REST API                                                                                         | None; free REST API                                                        |
| HGNC                                 | Open access; no paywall                                                                   | REST API                                                                                         | None; free REST API                                                        |
| NCBI Taxonomy                        | Open access; no paywall                                                                   | E‑utilities / REST API                                                                           | API key recommended for higher rate limits (free at NCBI account settings) |
| HitPredict                           | Open access; no paywall                                                                   | Bulk download                                                                                    | None; FTP download and web interface                                       |
| Pathway Commons                      | Open access; no paywall                                                                   | REST API                                                                                         | None; free REST API                                                        |
| ProteomeXchange (DDRAM, bromodomain) | Open access; no paywall                                                                   | FTP                                                                                              | None; ProteomeXchange FTP                                                  |

## Expected data types

| Source                               | Data type                                            | Access Method            | Format                                |
|--------------------------------------|------------------------------------------------------|--------------------------|---------------------------------------|
| IntAct                               | Molecular interactions (PPI)                         | PSICQUIC / REST API      | PSI‑MITAB (tab‑separated), PSI‑MI XML |
| BioGRID                              | Molecular interactions, PTMs, CRISPR                 | REST API                 | MITAB, PSI‑XML, TAB                   |
| REPAIRtoire                          | DNA repair pathways, protein membership, diseases    | Web scraping             | HTML tables                           |
| Reactome                             | Pathways, reactions, physical entities, interactions | REST API / BioPAX / SBML | JSON (API), BioPAX, SBML, MySQL dump  |
| UniProt                              | Protein annotations, identifiers, taxonomy           | REST API                 | JSON, XML, TSV                        |
| HGNC                                 | Gene symbols, cross‑references                       | REST API                 | JSON                                  |
| NCBI Taxonomy                        | Taxonomy IDs, scientific names                       | E‑utilities / REST API   | XML, JSON                             |
| HitPredict                           | High‑confidence PPIs, reliability scores             | Bulk download            | TAB‑separated files                   |
| Pathway Commons                      | Pathway membership, interactions                     | REST API                 | BioPAX, GSEA GMT                      |
| ProteomeXchange (DDRAM, bromodomain) | Proteomics interactions                              | FTP                      | mzIdentML, mzTab, AP‑MS tables        |

## Expected conflicts and overlaps

1. **Overlap:** IntAct vs. BioGRID  
Both databases contain largely overlapping sets of physical PPIs, because they both extract from the same primary literature.     
*Resolution*: Keep IntAct as a primary source.   
2. **Overlap:** Reactome vs. Pathway Commons  
Pathway Commons aggregates pathway data from Reactome along with other sources (KEGG, PID, others). For the DNA repair pathways of interest, there is substantial overlap, but Pathway Commons may also include interactions that Reactome does not (via its integration of other databases) or conversely, Reactome may contain curated pathway details not fully captured in Pathway Commons.  
*Resolution*: Pathway Commons is treated as a secondary, alternative source for pathway annotation. For the interaction_context field, Reactome is preferred because it provides more granular reaction‑level descriptions. For pathway membership (protein_a_repair_pathways), both sources are queried;    
if they disagree, Reactome is prioritised because it is the more granular, manually curated source.   
3. **Overlap:** HitPredict vs. IntAct/BioGRID confidence scores    
HitPredict assigns its own interaction_confidence (reliability score) to interactions derived from IntAct and BioGRID. The same interaction present in IntAct (with its original MIscore) may also appear in HitPredict with a different numerical score, because HitPredict recomputes confidence based on its own algorithm.  
*Resolution*: Use HitPredict score and mark it as the source for interaction.    
4.  **Conflict:** Contradictory interaction types for the same protein pair from different sources  
For the same protein pair, one source may report direct interaction (MI:0407) while another reports physical association (MI:0915) or co‑immunoprecipitation (MI:0005). The most specific, reliable term is kept, but the dispute is recorded.  
*Resolution*: If multiple interaction_type or detection_method values exist for the same interaction, the most specific experimental method is retained. Specifically: crystallography (MI:0018) > affinity chromatography (MI:0004) > co‑immunoprecipitation (MI:0005) > computational prediction (MI:0661, excluded). The alternative values are recorded in record_notes.  
5. **Overlap:** REPAIRtoire vs. Reactome pathway assignments  
A given protein may be assigned to a DNA repair pathway by REPAIRtoire but not by Reactome, or vice versa. REPAIRtoire explicitly provides the eight curated pathways that are the controlled vocabulary for protein_a_repair_pathways. Reactome may not assign the same protein to the same pathway or may assign it to a broader pathway class.   
*Resolution*: REPAIRtoire is treated as the authoritative source for protein_a_repair_pathways because it was specifically designed for DNA repair systems biology. Reactome pathway assignments are only used as an alternative if REPAIRtoire does not provide an annotation for a given protein. The source of the pathway annotation is recorded in record_notes (e.g., “pathway from REPAIRtoire” or “pathway from Reactome”).  

## Coverage gaps

1. **Missing source types**  
Scientific papers are absent as a source type in the current source map.  
This is not a deficiency because the databases already derive their content from papers. 
However, there may be recent (2024–2026) publications containing PPI data that are not yet incorporated into IntAct or BioGRID.   
The dataset should be periodically updated to incorporate newly curated interactions.
2. **Gaps in coverage by protein**  
Non‑human organisms are not prioritised. The dataset currently focuses on human (taxid:9606) interactions because the downstream application (DNA repair and cancer‑related mechanisms) is primarily human‑centric. 
If the use case expands, interactions for mouse (10090), yeast (4932), and other model organisms should be incorporated.
3. **Interaction context (interaction_context)**  
Interaction context (interaction_context)  is not systematically available for most interactions, so only those records which have it specified are kept. Only Reactome provides this field; most other sources do not. 
4. **Gaps in coverage by time (recent literature)**
IntAct and BioGRID are updated periodically.   
IntAct is updated frequently; BioGRID is updated on the 2nd of each month.   
Interactions from very recent publications (e.g., 2025–2026) may not yet be curated. Future updates will require re‑extracting from all sources.

### Plan for addressing coverage gaps
1. Implement periodic updates (e.g., quarterly) to capture newly curated interactions, particularly from recent publications.  
2. Document missing interaction_context in the dataset description as a known limitation.  
3. Evaluate inclusion of mouse and yeast interactions after the human dataset is complete, if resources permit and the use case expands.  