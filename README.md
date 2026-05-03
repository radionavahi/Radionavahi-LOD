# Radionavahi-LOD
> A Linked Open Data knowledge graph of Iranian regional and ritual musics, derived from the [Radionavahi](https://radionavahi.com) audio archive.

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](LICENSE)
[![Format: RDF/XML (OWL 2)](https://img.shields.io/badge/Format-RDF%2FXML%20(OWL%202)-blue.svg)](radionavahi-micro-graph.rdf)
[![Visualize in WebVOWL](https://img.shields.io/badge/Visualize-WebVOWL-orange.svg)](https://service.tib.eu/webvowl/)

## Overview

This repository hosts the **Radionavahi micro-graph**, a curated conceptual knowledge graph that represents the universe of Iranian regional musics, instruments, ritual contexts, performers, and ethnic groups documented in the Radionavahi archive. It is designed to demonstrate the transition from a document-based archive (a WordPress site of audio recordings, metadata, and tags) to a Linked Open Data resource that participates in the wider semantic web.

The model follows the methodological principles of cultural-heritage knowledge engineering projects such as [ACUSTEME](http://ontology.acusteme.org/):

- **Preserve emic categories.** Regional repertoires, ritual contexts, and indigenous instrument names are kept in their culture-specific forms rather than collapsed into generic Western analogues.
- **Maintain semantic interoperability.** Every reconciled entity carries `rdfs:seeAlso` links to authority files (Wikidata, GeoNames) so the graph can participate in federated SPARQL queries across the LOD Cloud.
- **Avoid over-complex ontology design.** The micro-graph deliberately keeps the schema minimal and human-readable; it is intended as a teaching artefact and conceptual demonstration, not as a maximalist research vocabulary.

## Quick statistics

| Element | Count |
|---|---:|
| Classes | 14 (5 top-level + 9 sub-classes) |
| Object properties | 11 |
| Datatype properties | 3 |
| Named individuals | 73 |
| Object-property assertions | 264 |
| Individuals with Wikidata `rdfs:seeAlso` | 38 |
| Regions with GeoNames `rdfs:seeAlso` | 4 |

## Files

| File | Purpose |
|---|---|
| [`radionavahi-micro-graph.rdf`](radionavahi-micro-graph.rdf) | The knowledge graph itself, in RDF/XML (OWL 2) format. The single canonical artefact of this repository. |
| [`docs/curation.md`](docs/curation.md) | Curation notes: methodology, design decisions, scope choices, and validation procedures. |
| [`docs/methodology.md`](docs/methodology.md) | The four-stage pipeline (audit → ACUSTEME alignment → reconciliation → micro-graph) that produced the file. |
| [`docs/changelog.md`](docs/changelog.md) | Version history (v1 → v2). |
| [`LICENSE`](LICENSE) | CC BY 4.0 license terms. |

## Schema

### Top-level classes

| Class | Persian | Description |
|---|---|---|
| `Artist` | هنرمند | Performers, singers, and instrumentalists. |
| `Instrument` | ساز | Musical instruments (organology). Sub-classed by family. |
| `Ritual` | آیین | Ritual or ceremonial contexts. Sub-classed by social function. |
| `MusicalGenre` | گونهٔ موسیقایی | Repertoire categories. Sub-classed by tradition type. |
| `Region` | منطقه | Geographic locations (provinces, cities, cultural regions). |
| `EthnicGroup` | قوم | Ethno-linguistic communities. |

### Sub-class hierarchy

```
Instrument
├── StringInstrument        ساز زهی
├── WindInstrument          ساز بادی
└── PercussionInstrument    ساز کوبه‌ای

Ritual
├── LifeCycleRitual         آیین چرخهٔ زندگی
├── HealingRitual           آیین درمانی
└── DevotionalRitual        آیین آیینی-عرفانی

MusicalGenre
├── RegionalRepertoire      موسیقی نواحی
└── ClassicalRepertoire     موسیقی کلاسیک
```

### Object properties

| Property | Domain | Range |
|---|---|---|
| `belongsToGenre` | Instrument ∪ Ritual | MusicalGenre |
| `isPlayedIn` | Instrument ∪ Ritual | Region |
| `playsInstrument` | Artist | Instrument |
| `takesPlaceIn` | Ritual | Region |
| `originatesIn` | MusicalGenre | Region |
| `usesInstrument` | MusicalGenre | Instrument |
| `practicedBy` | Ritual | EthnicGroup |
| `nativeTo` | EthnicGroup | Region |
| `performsGenre` | Artist | MusicalGenre |
| `bornIn` | Artist | Region |
| `influencedBy` | MusicalGenre | MusicalGenre |

### Datatype properties

| Property | Range | Notes |
|---|---|---|
| `hasFloruitDate` | `xsd:gYear` | Documented period of activity for an artist. |
| `hasUNESCOInscription` | `xsd:string` | UNESCO Intangible Cultural Heritage register number. |
| `hasLatinTransliteration` | `xsd:string` | Scholarly Latin transliteration distinct from English label. |

## Linked Open Data integration

The graph integrates with the wider LOD Cloud via lightweight `rdfs:seeAlso` cross-references rather than `owl:sameAs` identity claims. This is a deliberate design choice: it allows downstream consumers to discover the corresponding Wikidata or GeoNames entity without committing the local resource to *being identical* to its external counterpart, which preserves nuance in cases where the local sense (e.g., the cultural-historical region of "Khorasan") does not exactly match the modern administrative entity.

| Authority | Coverage |
|---|---|
| Wikidata | 38 of 73 individuals (52%) |
| GeoNames | 4 of 15 regions |

## How to use this graph

### Visualize in WebVOWL

1. Open the [WebVOWL service](https://service.tib.eu/webvowl/).
2. Click **Ontology → Custom Ontology → Select ontology file**.
3. Upload `radionavahi-micro-graph.rdf`.
4. Use the **Filter** menu to toggle individuals on/off, switching between schema view and populated view.

### Open in Protégé

1. Install [Protégé 5.x](https://protege.stanford.edu/).
2. `File → Open` and select `radionavahi-micro-graph.rdf`.
3. The Classes, Object Properties, Datatype Properties, and Individuals tabs will populate automatically.
4. For graph visualization, use the built-in OntoGraf tab or install the WebVOWL plugin.
5. To see Persian labels alongside English, set Preferences → Renderer → Render labels in language: `fa, en` (in that order, or vice versa).

### Query with SPARQL

The graph can be loaded into any SPARQL-capable triplestore (Apache Jena Fuseki, GraphDB, Blazegraph, Stardog). Once loaded, queries like the following become possible:

```sparql
PREFIX test: <http://www.radionavahi.org/ontology/test#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

# Which instruments are played in Khorasan, and who plays them?
SELECT ?instrument ?instrumentLabel ?artist ?artistLabel
WHERE {
  ?instrument test:isPlayedIn test:Khorasan ;
              rdfs:label ?instrumentLabel .
  FILTER (lang(?instrumentLabel) = "en")
  OPTIONAL {
    ?artist test:playsInstrument ?instrument ;
            rdfs:label ?artistLabel .
    FILTER (lang(?artistLabel) = "en")
  }
}
```

A small collection of example SPARQL queries will be added under `examples/` in a future commit.

## Methodology in brief

The graph was produced by a four-stage pipeline applied to the Radionavahi WordPress export (505 items, 82 audio recordings, ~700 raw taxonomy terms):

1. **Audit & normalization** — resolved 638 percent-encoded slugs, 9 ZWNJ duplicates, 116 cross-taxonomy collisions, 23 unused tags, and 3 526 illegal XML control characters; re-projected all terms onto a 6-domain ethnomusicological ontology.
2. **ACUSTEME alignment** — mapped each canonical term to its ACUSTEME 1.0 class equivalent and serialized the full archive as RDF/Turtle (557 subjects).
3. **Reconciliation** — resolved 50 representative entities to Wikidata QIDs and GeoNames numeric IDs (62% Wikidata match rate; 100% for regions and ethnic groups).
4. **Micro-graph curation** — selected 73 individuals across 14 classes, derived ~120 corpus-attested co-occurrence assertions, supplemented with ~150 culturally-defensible domain-knowledge enrichments, and serialized as a Protégé-clean RDF/XML file (this repository).

For the full methodology, see [`docs/methodology.md`](docs/methodology.md).

## Citation

If you use this graph in academic work, please cite as:

> Radionavahi-LOD: a Linked Open Data micro-graph of Iranian regional and ritual musics. Version 2.0, 2026. Available at https://github.com/radionavahi/Radionavahi-LOD

A more formal citation will be added when the associated thesis is published.

## License

The data and schema in this repository are released under the **Creative Commons Attribution 4.0 International (CC BY 4.0)** license. You are free to share and adapt the material for any purpose, including commercially, as long as you provide appropriate credit. See [`LICENSE`](LICENSE) for the full terms.

External authority-file references (Wikidata QIDs, GeoNames IDs) are governed by their respective sources' licenses (CC0 for Wikidata, CC BY 4.0 for GeoNames).

## Acknowledgements

This graph builds on:

- The [Radionavahi](https://radionavahi.com) archive, the source of the underlying audio metadata.
- The [ACUSTEME ontology](http://ontology.acusteme.org/) by the CNRS Centre for Research on the Anthropology of Music, which informed the class-design methodology.
- [Wikidata](https://www.wikidata.org) and [GeoNames](https://www.geonames.org), which provide the authority-file references that link this graph to the wider LOD Cloud.

## Status

**Version 1.0** (current). Schema-enriched, with sub-class hierarchies and 11 object properties.

Future work (Phase B) will introduce a `Recording` or `MusicalWork` class to model individual audio items as first-class entities, extending the graph from a vocabulary-cum-gazetteer into a full archival catalogue.
## Thank You!
## Hossein Ebrahimi
