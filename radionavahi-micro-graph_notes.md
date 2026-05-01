# Radionavahi micro-graph v2 — schema enrichment summary

**File:** `radionavahi-micro-v2.rdf`  
**Predecessor:** `radionavahi-micro.rdf` (v1)  
**Methodology:** Phase A of the scaling plan — schema enrichment + entity expansion from existing corpus, no new external data needed.

## Statistics comparison (WebVOWL panel)

| Metric | v1 | v2 | Change |
|---|---:|---:|---:|
| Classes (declared) | 5 | 14 | +9 |
| Object properties | 4 | 11 | +7 |
| Datatype properties | 0 | 3 | +3 |
| Individuals | 46 | 73 | +27 |
| Object-property assertions | 117 | 264 | +147 |
| Schema nodes (WebVOWL "Nodes") | 7 | ~17 | ~2.4× |
| Schema edges (WebVOWL "Edges") | 7 | ~25 | ~3.6× |

## What changed

### New classes (sub-class hierarchies)
- `Instrument` now has three sub-classes: `StringInstrument`, `WindInstrument`, `PercussionInstrument`.
- `Ritual` now has three sub-classes: `LifeCycleRitual`, `HealingRitual`, `DevotionalRitual`.
- `MusicalGenre` now has two sub-classes: `RegionalRepertoire`, `ClassicalRepertoire`.
- New top-level class: `EthnicGroup`.

These sub-classes are not cosmetic. The Iranian organology genuinely splits along these lines (string instruments belong to court/Sufi repertoires; wind instruments dominate southern coastal music; percussion is ceremonial across most regions), and the ritual sub-classes capture the major typological distinction in the literature (life-cycle vs. healing vs. devotional rituals).

### New object properties
- `originatesIn` (Genre → Region) — anchors regional repertoires to their geography.
- `usesInstrument` (Genre → Instrument) — instrumentation as a property of genre, complementing `belongsToGenre`'s reverse direction.
- `practicedBy` (Ritual → EthnicGroup) — connects ritual practice to its bearers.
- `nativeTo` (EthnicGroup → Region) — geographic anchor for ethnic groups.
- `performsGenre` (Artist → Genre) — direct artist-genre link.
- `bornIn` (Artist → Region) — biographical anchor.
- `influencedBy` (Genre → Genre) — captures the genealogical relations between regional musics (Bandari ← Southern Iran Music; Mazandarani ← Gilaki; etc.).

### New datatype properties
- `hasFloruitDate` (Artist → xsd:gYear) — biographical dating.
- `hasUNESCOInscription` (Genre/Ritual → xsd:string) — UNESCO ICH register numbers (Persian Radif = 00279, Khorasan Maqami = 00388, Taʿzieh = 00377).
- `hasLatinTransliteration` (any → xsd:string) — scholarly Latin form, distinct from English label.

### New individuals (27 added)
- **+7 Regions:** North Khorasan, Razavi Khorasan, Khuzestan, Fars, Semnan, Herat, Torbat-e Jam.
- **+2 Instruments:** Ney-labak (shepherd flute), Senj (cymbals).
- **+4 Rituals:** Therapeutic Music, Lullaby (Lalayi), Mowyeh, Noheh.
- **+3 Genres:** Mazandarani Music, Talesh Music, Qashqai Music.
- **+4 Ethnic Groups:** Bakhtiari, Qashqai, Hazara, Bedouins. (Tuareg and Inuit deliberately excluded as out-of-scope comparative references in the source archive.)
- **+7 Artists:** Gholam-Ali Pourataei, Osman Khafi, Ashur Geldi Garkazi, Ebrahim Sharifzadeh, Mohammad-Reza Eshaghi Gorji, Ashiq Yousef Ohanes, Taher Yarvaisi.

### Notable design decisions
- **`isPlayedIn` and `belongsToGenre` use `owl:unionOf` for multi-class domains.** This is the standard OWL idiom for "the property applies when the subject is *either* an Instrument *or* a Ritual." Protégé and WebVOWL both render union-domain properties cleanly.
- **Hazaragi misclassification (from Stage 1) is now corrected.** "هزارگی" was bucketed as a `ritual_context` during normalization. In v2, the Hazara ethnic group is properly typed as `EthnicGroup`, with `Q115473` Wikidata link, and `nativeTo Herat`.
- **Tuareg and Inuit deliberately removed from v2.** They appeared in the Radionavahi archive only as comparative ethnomusicology cross-references, not as participants in Iranian musical traditions. Including them in v1 created orphan nodes; v2 scopes the model strictly to Iranian and Iran-adjacent musics.
- **Datatype values are sparse on purpose.** Only 10 datatype assertions across the 73 individuals — enough to populate the WebVOWL panel's "Datatype prop." count and demonstrate the methodology, not so dense that they crowd the graph.

## What's *not* in v2 (deferred to Phase B)

- A `Recording` or `MusicalWork` class — would require specific recording titles with metadata (the highest-value Phase B input).
- Performer biographies beyond floruit dates — would need lived-dates, primary teachers, recorded works.
- Genre sub-classification beyond Regional/Classical (e.g., `SufiRepertoire`, `WomensRepertoire`) — possible but adds complexity without clear ethnomusicological consensus.
- More cross-archive linking (Mahoor Institute IDs, IRCICA references) — would need the parallel material.

## Validating in Protégé and WebVOWL

1. **Protégé:** `File → Open` and select `radionavahi-micro-v2.rdf`. The Classes tab will show the hierarchy with `Instrument`, `Ritual`, `MusicalGenre` as parent nodes containing their sub-classes. Object Properties tab shows 11 entries; Datatype Properties tab shows 3; Individuals tab shows 73 grouped under their most-specific class.
2. **WebVOWL:** [service.tib.eu/webvowl](https://service.tib.eu/webvowl) → Ontology → Custom Ontology → Upload. The Statistics panel should show Classes: 15+, Object prop.: 12, Datatype prop.: 3, Individuals: 73, Nodes ≈ 17, Edges ≈ 25.
