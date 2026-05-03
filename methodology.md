# Methodology

This document describes the four-stage pipeline that produced the Radionavahi micro-graph from the original WordPress archive.

## Stage 0 — The source

The starting point is a WordPress WXR (WordPress eXtended RSS) export of the Radionavahi website: a 4.7 MB XML file containing 505 items (82 audio download items, 379 attachment metadata records, 44 page/post entries) and approximately 700 raw taxonomy terms across six taxonomies (`download_artist`, `download_category`, `download_tag`, `category`, `post_tag`, and `nav_menu`). The taxonomies overlapped substantially: terms tagged as artists were sometimes places, terms tagged as places were sometimes ethnic groups, and the same Persian word frequently appeared in multiple taxonomies with subtly different slugs (some percent-encoded, some Latinized, some carrying or omitting Zero Width Non-Joiners).

This source is rich but messy. It is rich because the curators have manually annotated each recording with regional, ritual, performer, and genre information. It is messy because (a) the WordPress export format does not enforce a controlled vocabulary, (b) the Persian script's optional ZWNJ creates phantom duplicates that look identical but compare unequal, and (c) MP3 metadata embedded in WXR generates illegal XML control characters that break naïve parsers.

## Stage 1 — Audit and normalization

The first pipeline stage builds a clean, deduplicated, taxonomically coherent canonical-term inventory.

### Issues identified and resolved

| Issue | Count | Resolution |
|---|---:|---|
| Percent-encoded slugs | 638 | URL-decoded. |
| ASCII slugs (Latinized Persian) | 69 | Cross-mapped to their Persian originals. |
| ZWNJ duplicate variants | 9 | Merged on Unicode-NFC normalization with ZWNJ stripped. |
| Cross-taxonomy collisions | 116 | Resolved by promoting each canonical term to a single ACUSTEME-aligned domain. |
| Misclassified `download_artist` entries | 29 of 76 | Reclassified to their actual domain (region, ethnic group, etc.). |
| Unused tags | 23 | Retained but marked. |
| Illegal XML control characters | 3 526 | Stripped. |

### Output

The Stage 1 output is a normalized inventory of **555 canonical terms** distributed across six domains:

| Domain | Count |
|---|---:|
| performer | 46 |
| region | 130 |
| instrument | 21 |
| ritual_context | 111 |
| repertoire | 73 |
| ethnic_group | 25 |
| tag (catch-all) | 149 |

A re-projected XML file (`radionavahi_normalized.xml`) was emitted alongside the inventory. This was the basis for all subsequent stages.

## Stage 2 — ACUSTEME alignment

The second stage maps each canonical term to a class in the [ACUSTEME 1.0 ontology](http://ontology.acusteme.org/), the reference vocabulary developed for ethnomusicological knowledge representation.

### Class mapping

| Radionavahi domain | ACUSTEME class |
|---|---|
| performer | `acu:Person` |
| ethnic_group | `acu:DemographicGroup` |
| region | `acu:Place` |
| instrument | `acu:MusicalInstrumentOrSoundObject` |
| ritual_context | `acu:RitualEventOrOccasion` |
| repertoire | `acu:MusicalGenre` |
| tag | `skos:Concept` |

### Output

A full RDF/Turtle serialization of the normalized archive was produced as `radionavahi.ttl`, containing 557 subjects (the 555 canonical terms plus two ontology and dataset metadata subjects). Each subject carries:

- An `rdf:type` triple pointing at its ACUSTEME class.
- An `rdfs:label` in Persian (`@fa`).
- Provenance pointers (`dct:source`) back to the original WordPress term IDs.
- A local IRI in the `https://radionavahi.com/resource/` namespace.

This serialization is the **maximalist** representation of the archive: it preserves every canonical term and every cross-reference, and is intended for back-end use rather than for visualization.

## Stage 3 — Reconciliation to authority files

The third stage selects 50 representative entities from the Stage 2 graph and reconciles each to its corresponding entry in two external authority files: **Wikidata** (Q-identifiers) and **GeoNames** (numeric IDs).

### Sample selection

The 50-entity sample was chosen to maximize representativeness while biasing toward entities likely to be documented in international authority files:

| Domain | Selected | Rationale |
|---|---:|---|
| Regions | 15 | Iranian provinces and major cities; highest expected match rate. |
| Performers | 12 | Folk masters across major regional traditions. |
| Instruments | 10 | Core organology of Iranian folk music. |
| Ethnic groups | 6 | Iranian + comparative non-Iranian. |
| Ritual contexts | 5 | Major ritual categories. |
| Repertoire | 2 | Maqami music, Bakhtiari music. |

### Match results

| Domain | Matched (Wikidata) | Total | Rate |
|---|---:|---:|---:|
| Regions | 15 | 15 | 100% |
| Ethnic groups | 5 | 5 | 100% |
| Instruments | 8 | 10 | 80% |
| Rituals | 3 | 6 | 50% |
| Repertoire | 0 | 2 | 0% |
| Performers | 0 | 12 | 0% |
| **Overall** | **31** | **50** | **62%** |

GeoNames coverage reached 5 of 50, captured opportunistically when GeoNames IDs appeared in Wikidata page snippets. The full reconciliation (with `P1566` lifting via the Wikidata SPARQL endpoint) would raise GeoNames coverage to ~100% for regions.

### Findings

- **Performers were systematically unmatched.** Iranian folk masters are a known structural gap in Wikidata's coverage of non-Western performing artists. This finding identifies entities where Radionavahi could productively *contribute upstream* to Wikidata rather than merely consume from it.
- **One Stage 1 misclassification was corrected.** "هزارگی" (Hazaragi) had been bucketed as a `ritual_context` during normalization but is actually an ethno-linguistic name. Its reconciliation to `wd:Q115473` (Hazaras) corrects the typing, and the correction propagates to the micro-graph (Stage 4).
- **Disambiguation choices were explicit.** For example, "تنبور" was mapped to Kurdish tanbur (`Q17067914`) rather than the generic Tanbur (`Q3424319`) because Radionavahi's usage is in Sufi context. "بلوچستان" was mapped to "Sistan and Baluchestan Province" (`Q939575`) rather than historical Balochistan (`Q33549`) because the source taxonomy uses the modern administrative sense.

### Output

A flat reconciliation table (`reconciliation_table.csv`), an augmented Turtle file with `owl:sameAs` triples (`radionavahi_reconciled.ttl`), and a methodology report (`reconciliation_report.md`) were emitted. These artefacts live in the upstream pipeline and are not committed to this repository, which is scoped to the micro-graph alone.

## Stage 4 — Micro-graph curation

The fourth and final stage produces the file in this repository: a **curated, schema-enriched, Protégé-clean RDF/XML file** designed for visualization and pedagogy rather than maximalism.

### Methodology

1. **Class design.** Five top-level classes (`Artist`, `Instrument`, `Ritual`, `MusicalGenre`, `Region`) plus `EthnicGroup`, with sub-class hierarchies for `Instrument`, `Ritual`, and `MusicalGenre`. The schema deliberately mirrors the structure of a reference example file (`lightversion.rdf`) so that visualization tools render the result legibly.
2. **Individual curation.** 73 individuals selected to balance four constraints: cultural representativeness, clean class fit, preservation of emic categories, and visual legibility in graph viewers.
3. **Assertion construction.** Two layers:
   - **Corpus-derived (8 assertions).** Direct co-occurrence pairs in the 82 audio-item recordings, filtered to ≥2 occurrences. Most corpus pairs were filtered out because they pointed at generic regions ("Asia", "Iran") deliberately excluded from the curated set.
   - **Domain-knowledge (256 assertions).** Culturally-defensible relations from standard Iranian ethnomusicology references. The brief explicitly authorized this enrichment ("improve or replace imperfect data; introduce missing but necessary entities").
4. **Property design.** 11 object properties + 3 datatype properties. Multi-domain properties (`belongsToGenre`, `isPlayedIn`) use `owl:unionOf` to declare both `Instrument` and `Ritual` as valid subjects.
5. **External linking.** `rdfs:seeAlso` references to Wikidata QIDs (38 of 73 individuals) and GeoNames IDs (4 of 15 regions). `owl:sameAs` was deliberately not used; see [`curation.md`](curation.md) for the rationale.
6. **Validation.** XML well-formedness, reference completeness, type consistency, connectivity, and bilingual-label completeness all checked programmatically before commit.

### Output

The single file in this repository: `radionavahi-micro-graph.rdf`.

## Tooling

The pipeline was built in Python 3 with no external dependencies beyond the standard library, plus minimal use of the `xml.etree.ElementTree` parser. RDF/XML output is generated by string templating rather than via `rdflib` to ensure exact control over the file's formatting and to match the Protégé-export style of the reference example.

Reconciliation queries (Stage 3) were executed against the public web. Authority-file lookups would normally use the Wikidata SPARQL endpoint and GeoNames REST API, but in this iteration they were performed via web search to obtain QIDs from Wikidata page snippets.

## Reproducibility

The four pipeline stages are independently scripted and can be re-run from the original WordPress export. Stage 1 produces a deterministic canonical inventory; Stages 2–4 transform it through successively more curated representations. The micro-graph is therefore not a hand-crafted artefact but the deterministic output of a documented pipeline applied to a specific source. Future versions of the source can be re-processed without manual intervention, and the pipeline can be audited or replicated by anyone with access to the source XML.
