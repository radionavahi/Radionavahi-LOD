# Changelog

All notable changes to the Radionavahi-LOD micro-graph are documented here. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the project adheres to semantic versioning at the schema level.

## [2.0.0] — 2026

### Summary

Schema-enriched release. Adds class hierarchy, the `EthnicGroup` class, seven new object properties, three datatype properties, and 27 new individuals. The schema view in WebVOWL grows from 7 nodes / 7 edges to approximately 17 nodes / 25 edges, while individual-level assertions more than double from 117 to 264.

### Statistics comparison

| Metric | v1 | v2 | Change |
|---|---:|---:|---:|
| Classes (declared) | 5 | 14 | +9 |
| Object properties | 4 | 11 | +7 |
| Datatype properties | 0 | 3 | +3 |
| Individuals | 46 | 73 | +27 |
| Object-property assertions | 117 | 264 | +147 |
| WebVOWL "Nodes" (schema view) | 7 | ~17 | ~2.4× |
| WebVOWL "Edges" (schema view) | 7 | ~25 | ~3.6× |

### Added — Class hierarchy

The `Instrument`, `Ritual`, and `MusicalGenre` classes now have sub-classes:

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

These sub-classes are not cosmetic. The Iranian organology genuinely splits along these lines (string instruments belong to court and Sufi repertoires; wind instruments dominate southern coastal music; percussion is ceremonial across most regions), and the ritual sub-classes capture the major typological distinction in the literature (life-cycle vs. healing vs. devotional rituals).

### Added — `EthnicGroup` class

A new top-level class for ethno-linguistic communities, joining `Artist`, `Instrument`, `Ritual`, `MusicalGenre`, and `Region` at the top of the hierarchy.

### Added — Object properties

Seven new object properties extend the model's expressivity:

- `originatesIn` — `MusicalGenre` → `Region`. Anchors regional repertoires to their geography.
- `usesInstrument` — `MusicalGenre` → `Instrument`. Captures instrumentation as a property of genre.
- `practicedBy` — `Ritual` → `EthnicGroup`. Connects ritual practice to its bearers.
- `nativeTo` — `EthnicGroup` → `Region`. Geographic anchor for ethnic groups.
- `performsGenre` — `Artist` → `MusicalGenre`. Direct artist-genre link.
- `bornIn` — `Artist` → `Region`. Biographical anchor.
- `influencedBy` — `MusicalGenre` → `MusicalGenre`. Captures genealogical relations between regional musics.

### Added — Datatype properties

Three datatype properties enable typed-literal assertions:

- `hasFloruitDate` (`xsd:gYear`) — biographical dating for artists.
- `hasUNESCOInscription` (`xsd:string`) — UNESCO ICH register numbers (Persian Radif 00279, Khorasan Maqami 00388, Taʿzieh 00377).
- `hasLatinTransliteration` (`xsd:string`) — scholarly Latin form, distinct from English label.

Datatype assertions are deliberately sparse in this release (approximately ten total): the schema is validated rather than densely populated.

### Added — Individuals

Twenty-seven new individuals across the expanded class hierarchy:

- **Regions (+7):** North Khorasan, Razavi Khorasan, Khuzestan, Fars, Semnan, Herat, Torbat-e Jam.
- **Instruments (+2):** Ney-labak (shepherd flute), Senj (cymbals).
- **Rituals (+4):** Therapeutic Music, Lullaby (Lalayi), Mowyeh, Noheh.
- **Genres (+3):** Mazandarani Music, Talesh Music, Qashqai Music.
- **Ethnic Groups (+4):** Bakhtiari, Qashqai, Hazara, Bedouins.
- **Artists (+7):** Gholam-Ali Pourataei, Osman Khafi, Ashur Geldi Garkazi, Ebrahim Sharifzadeh, Mohammad-Reza Eshaghi Gorji, Ashiq Yousef Ohanes, Taher Yarvaisi.

### Changed — Property domain/range declarations

All eleven object properties now carry explicit `rdfs:domain` and `rdfs:range`. In v1, only `isPlayedIn` had domain/range; the others were unrestricted. The result is a much richer schema view in WebVOWL — class-to-class edges now reflect every object property in the model, not just one.

The two multi-domain properties — `belongsToGenre` and `isPlayedIn` — use `owl:unionOf` to declare both `Instrument` and `Ritual` as valid subjects. This is the standard OWL idiom for "the property applies when the subject is *either* an Instrument *or* a Ritual." Protégé and WebVOWL both render union-domain properties cleanly.

### Changed — Hazaragi correction

In v1, the Persian term "هزارگی" (Hazaragi) was carried over from Stage 1 normalization, where it had been misclassified as a `ritual_context`. In v2, this is corrected: the Hazara ethnic group is properly typed as `EthnicGroup`, with Wikidata `rdfs:seeAlso` to Q115473, and `nativeTo Herat`. The correction was identified during Stage 3 reconciliation and applied at the micro-graph layer.

### Removed — Tuareg and Inuit

The Tuareg and Inuit ethnic groups, present in v1 as orphan nodes, are removed from v2. They appeared in the Radionavahi archive only as comparative ethnomusicology cross-references (the curators included a small number of comparative recordings to contextualize Iranian material against neighbouring or distant traditions), not as participants in Iranian musical life. Their inclusion in v1 created visually disconnected nodes and diluted the model's focus. Their reconciled entries remain in the upstream Stage 3 dataset.

### Validation

All assertions in v2 were type-checked at construction time against the property's declared `rdfs:domain` and `rdfs:range`, with class-hierarchy reasoning (a `StringInstrument` subject satisfies an `Instrument` domain). 264 of 264 assertions passed. XML well-formedness, reference completeness, connectivity (no orphan nodes), and bilingual-label completeness (73 of 73 individuals carry both `@en` and `@fa` labels) were also validated programmatically before commit.

### Deferred to a future release (Phase B)

The following are explicitly out of scope for v2 and are planned for a subsequent release once additional source material is available:

- A `Recording` or `MusicalWork` class to model individual audio items as first-class entities.
- Performer biographies beyond floruit dates (lived dates, primary teachers, recorded works).
- Genre sub-classification beyond Regional / Classical (e.g., `SufiRepertoire`, `WomensRepertoire`).
- Cross-archive linking to Mahoor Institute, IRCICA, BL Sound Archive identifiers.
- Region containment (e.g., "Bushehr is part of Iran") via a `partOf` property.

---

## [1.0.0] — 2026

### Summary

Initial public release. A minimal Protégé-style micro-graph mirroring the structure of a reference example file, designed for visualization and conceptual demonstration.

### Schema

- 5 classes: `Artist`, `Instrument`, `MusicalGenre`, `Region`, `Ritual`.
- 4 object properties: `belongsToGenre`, `isPlayedIn`, `playsInstrument`, `takesPlaceIn`.
- No datatype properties.
- Only `isPlayedIn` carried explicit `rdfs:domain` / `rdfs:range` declarations.

### Individuals

- 46 named individuals: 8 regions, 14 instruments, 8 rituals, 8 genres, 8 artists.
- Bilingual labels (`@en` and `@fa`) on every individual.
- 24 individuals carried Wikidata `rdfs:seeAlso`; 4 regions also carried GeoNames `rdfs:seeAlso`.

### Assertions

- 117 object-property assertions: 8 corpus-derived (≥2 co-occurrences in source archive), 109 domain-knowledge.
- Distribution: 48 `belongsToGenre`, 32 `isPlayedIn`, 25 `takesPlaceIn`, 12 `playsInstrument`.

### Linking strategy

`rdfs:seeAlso` rather than `owl:sameAs` for external authority-file references — a deliberate choice to allow soft cross-referencing without identity collapse. See [`curation.md`](curation.md) for the full rationale.

---

## Versioning notes

The major version (2.x) reflects schema-level changes. A breaking schema change (renaming classes or properties, removing existing terms) would warrant version 3.0; non-breaking additions (new sub-classes, new properties, new individuals) increment the minor version (2.1, 2.2, …); pure data corrections without schema impact increment the patch version (2.0.1, 2.0.2, …).

## Planned

### [2.1.0] (planned)

- Cosmetic: rename `Sistan_Baluchestan` → `SistanBaluchestan` for consistency with `NorthKhorasan` / `RazaviKhorasan` (no underscore convention).
- Add an `examples/` folder with sample SPARQL queries demonstrating common access patterns.

### [3.0.0] (Phase B; pending source material)

- Introduce `Recording` (or `MusicalWork`) class to model individual audio items.
- Extend `Artist` with biographical datatype properties.
- Cross-archive linking to external identifier schemes.
