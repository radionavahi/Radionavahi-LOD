# Curation notes

This document records the design decisions, scope choices, and validation procedures behind the Radionavahi micro-graph (`radionavahi-micro-graph.rdf`). It is intended as a paper-trail-quality justification for choices that may not be self-evident from the file itself.

## Curation principles

The graph was assembled under five guiding principles:

### 1. Cultural plausibility over literal extraction

The Radionavahi archive contains 555 canonical taxonomy terms after Stage 1 normalization, of which only 73 appear in this micro-graph. The selection was governed not by frequency in the source but by **cultural representativeness** — covering the major regional traditions of Iranian folk music (southern coast, Khorasan and the eastern frontier, the western Zagros, the Caspian rim) and the principal organology, rituals, and genres associated with each. Where the corpus had ambiguous or thinly attested entities, they were excluded. Where domain knowledge could supply a culturally important entity that the corpus did not document well (e.g., the `Tar`, `Setar`, `Santur` of the classical radif tradition, or `TaziehPassion` as a ritual class), it was added.

### 2. Preservation of emic categories

The model deliberately preserves culture-specific distinctions:

- **Khorasan** is treated as a cultural-historical region (the source of *maqami* music), with the modern administrative split into `NorthKhorasan` and `RazaviKhorasan` represented as separate entities. The graph uses `rdfs:seeAlso` rather than `owl:sameAs` so that consumers can follow the link to Wikidata's modern administrative entity without forcing the local sense to identify with it.
- **Bandari** music is kept as a distinct genre rather than collapsed into "Iranian folk music," because the local conceptual category foregrounds Persian Gulf coastal identity.
- **Sharveh**, **Kar-Ava**, and **Mowyeh** are kept as distinct ritual/genre individuals despite all being lament-adjacent, because the local terminology marks meaningful distinctions of social context.
- **Khorasan Maqami** is given its own genre-individual rather than treated as a sub-classification of `PersianRadif`, which would erase the modal-system difference that defines it.

### 3. Lightweight LOD integration

The graph uses `rdfs:seeAlso` (not `owl:sameAs`) for all external references. This is a deliberate epistemic choice. `owl:sameAs` is a strong logical claim — that two URIs denote the *same* individual and that all assertions about one apply to the other. For cultural-heritage data, this is rarely safe: the Radionavahi sense of "Bushehr" includes its musical-cultural identity (sharveh, ney-anban, the dammam ensemble), while the Wikidata sense includes the city's economic statistics, mayor, and air-quality measurements. `rdfs:seeAlso` is a softer claim — "for related information, see the linked resource" — and is the right primitive for cross-referencing without identity collapse.

### 4. Visual legibility in graph viewers

The model targets WebVOWL and Protégé OntoGraf as primary visualization tools. This shaped several decisions:

- **Sub-class hierarchies are kept shallow** (one level deep). Deeper hierarchies render poorly in WebVOWL.
- **Property labels are short and natural** ("plays instrument", "originates in", "born in") so they fit on edges in a node-link diagram without truncation.
- **The number of properties is kept modest** (11 object + 3 datatype). A property explosion would clutter the WebVOWL panel without proportional gain in expressiveness.
- **Hub-and-spoke topology is encouraged**: regions are hubs (with many incoming edges), artists are leaves. This produces a layout that the force-directed algorithms in WebVOWL/OntoGraf handle well.

### 5. Scope honesty

The Radionavahi archive includes a small number of comparative ethnomusicology recordings — Tuareg music, Inuit throat-singing, Bororo recordings — used in a comparative-method context rather than as Iranian musical material. These appeared in Stage 3 reconciliation (where they were correctly linked to Wikidata), but they were excluded from the micro-graph because including them as `EthnicGroup` individuals creates orphan nodes (no `nativeTo` Iranian region, no `practicedBy` Iranian ritual) and dilutes the model's focus. The reconciled entries remain in the full Stage 3 dataset; the micro-graph scopes itself to Iranian and Iran-adjacent musics.

## Design decisions worth flagging

### Multi-domain properties via `owl:unionOf`

Two object properties — `belongsToGenre` and `isPlayedIn` — apply to subjects of more than one class:

- `belongsToGenre`: an `Instrument` *or* a `Ritual` can belong to a `MusicalGenre`.
- `isPlayedIn`: an `Instrument` *or* a `Ritual` can be played in a `Region`.

The OWL idiom for this is `owl:unionOf`. The alternative — splitting each into two narrower properties (`instrumentBelongsToGenre`, `ritualBelongsToGenre`, etc.) — would double the property count without adding semantic content. Both Protégé and WebVOWL render union-domain properties cleanly, so the union approach is preferred.

### `belongsToGenre` and `usesInstrument` are partly redundant

`belongsToGenre` (Instrument → Genre) and `usesInstrument` (Genre → Instrument) express the same underlying relation in opposite directions. A purer ontology would declare them as `owl:inverseOf` each other and assert only one direction. The graph keeps both directions explicitly because:

- WebVOWL does not infer inverse properties from `owl:inverseOf` for the visualization.
- Different downstream queries naturally start from different sides of the relation. A query about an instrument's genre coverage starts from the instrument; a query about a genre's instrumentation starts from the genre. Materializing both directions makes both query patterns equally direct.
- Storage cost is trivial.

### Datatype assertions are sparse on purpose

Only ~10 datatype assertions appear in the graph (3 floruit dates, 3 UNESCO inscription numbers, 4 Latin transliterations). These are illustrative — enough to populate the WebVOWL "Datatype prop." count and to demonstrate the methodology, not so dense that they crowd the node-link diagram. The `hasFloruitDate`, `hasUNESCOInscription`, and `hasLatinTransliteration` properties are declared in the schema and ready to receive more assertions in future versions; the choice in v2 was to validate the schema rather than to densely populate it.

### Genre genealogy via `influencedBy`

Seven `influencedBy` assertions encode the most defensible genealogical relations among Iranian regional musics: `NorthKhorasanMusic ← KhorasanMaqami`, `Bandari ← SouthernIranMusic`, `LoriMusic ← BakhtiariMusic`, `MazandaraniMusic ← GilakiMusic`, `TaleshMusic ← GilakiMusic`, `QashqaiMusic ← BakhtiariMusic`, `PersianRadif ← KhorasanMaqami`. These reflect mainstream ethnomusicological consensus about influence directionality. The property is deliberately *not* used to capture every conceivable cross-influence — only the structural ones.

### Hazaragi correction

In Stage 1 normalization, the Persian term "هزارگی" (Hazaragi) was misclassified as `ritual_context` because of its morphological similarity to other terms in the corpus. The Stage 3 reconciliation step correctly identified it as the Hazara ethnic-linguistic group, with Wikidata QID Q115473. The micro-graph applies this correction by representing it as `Hazara` (`EthnicGroup` class), with `nativeTo Herat`. This is a case where the linked-data graph corrects an error in the source taxonomy.

## What the graph deliberately does not do

The following are *not* in scope for this micro-graph:

- **Audio recording metadata.** Individual recordings (titles, durations, recording dates, file URLs, performers per session) are deferred to a future `Recording` or `MusicalWork` class. Adding them is the principal extension planned for Phase B.
- **Region containment.** "Bushehr is part of Iran" is true but unmodeled. Adding region containment requires a `partOf` property and a national-scope individual, which is outside the micro-graph's scope.
- **Genre sub-classification beyond Regional/Classical.** Categories like `SufiRepertoire`, `WomensRepertoire`, or `OccupationalSongs` could be reasonable, but they would exceed the micro-graph's complexity budget and are deferred.
- **Performer biographies beyond floruit dates.** Lived dates, primary teachers, recorded works, and discographies are out of scope. A future biographical extension would either add datatype properties to `Artist` or introduce a separate `BiographicalEvent` pattern.
- **Cross-archive linking.** References to other archives (Mahoor Institute IDs, IRCICA references, BL Sound Archive numbers) are not present. They would be added as additional `rdfs:seeAlso` URIs once those archives' identifier schemes are confirmed.

## Validation procedures applied

The following validation steps were performed before commit:

1. **XML well-formedness.** The file parses without error using Python's `xml.etree.ElementTree`.
2. **Reference completeness.** Every IRI used as the subject, predicate, or object of any assertion is declared elsewhere in the file as a class, property, or named individual. No dangling references.
3. **Type consistency.** Every object-property assertion was checked at construction time against the property's declared `rdfs:domain` and `rdfs:range` (with class-hierarchy reasoning — e.g., a `StringInstrument` subject satisfies an `Instrument` domain). 264 / 264 assertions pass.
4. **Connectivity check.** Every `Instrument`, `Ritual`, and `Artist` individual has at least one outgoing object-property edge. Every `Region` and `MusicalGenre` individual has at least one incoming edge. No orphan nodes.
5. **Bilingual labels.** Every named individual carries both an `@en` and an `@fa` `rdfs:label`. 73 / 73 individuals validated.
6. **Sub-class typing.** Every individual is typed against the most specific applicable class (e.g., `Dotar` is `StringInstrument`, not just `Instrument`). The parent-class memberships are inferred via the schema.

## Provenance summary

| Provenance | Count | Notes |
|---|---:|---|
| Corpus (≥2 co-occurrences in source archive) | 8 | Direct co-occurrences of canonical terms in 82 audio-item recordings. |
| Domain knowledge | 256 | Culturally-defensible relations from standard Iranian ethnomusicology references. |
| **Total object-property assertions** | **264** | |

The brief explicitly authorized domain-knowledge enrichment ("improve or replace imperfect data; introduce missing but necessary entities; refine naming and structure"). Domain-knowledge assertions outnumber corpus assertions because the corpus signal at the granularity of the curated 73 individuals is intrinsically sparse: most corpus pairs involved generic regions ("Asia", "Iran") that were filtered as too broad to be culturally informative, or fine-grained terms outside the micro-ontology's scope. Every domain-knowledge assertion is defensible against standard ethnomusicological references — for example, "Ney-anban is played in Bushehr" or "Tanbur is played in Kurdistan and Lorestan" are statements no specialist in Iranian music would dispute.
