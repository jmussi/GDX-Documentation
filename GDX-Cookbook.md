# GDX — Cookbook of Canonical Patterns

- **Status:** Draft v0.22.0 (companion to GDX Core Specification v0.30.0, working draft, pre-1.0)
- **Date:** 2026-10-19
- **Editors:** 
- **Authors:** Julian Mussi


## Abstract

This document is a companion to the GDX Core Specification. It is non-normative: it documents idiomatic GDX shapes for common modeling tasks. The patterns demonstrate the structural and schema mechanisms defined by the core specification (§3 through §7); they do not introduce new semantics.

Section references of the form §3, §5, §7 etc. refer to the GDX Core Specification.

The vocabulary identifiers used in these examples (`gdx:`, `schema:`, `vocab:`, etc.) are illustrative. Specific vocabulary content is defined by the relevant vocabulary specification, not by this document. The short prefixes are aliases chosen for readability; data that is stored or exchanged carries the authority each alias stands for (the full expansion of the CURIE), not the alias.

Every example is a complete GDX document: its top-level elements are root-eligible (Objects, HEADs, Templates, REL declarations, or Packages — Core Spec §7.2 rule 1). A Record or a Property never stands at the root. Where an edge or a reference in an example names a root that the example does not show (`@bob`, `@com.acme`), that root is assumed to carry that Id in this document or in one loaded alongside it (Core Spec §6.4.1). GDX has no comment syntax (Core Spec §4.6), so explanation sits in the prose around each example, never inside it.

## Status of This Document

This is the v0.22.0 working draft of the Cookbook, companion to the GDX Core Specification v0.30.0. v0.22.0 aligns the patterns with link-by-Id (Core Spec v0.30.0, entries 71–77): every link and reference binds to an Id, and a Name is a label that no reference resolves (a path segment aside). Roots and edges that examples link to now carry the Ids the links resolve to (`<OBJ &bob>`, `<REL &gdx:knows>`, `<DEFOBJ &CustomerV2>`); @-edges carry only a plain `@root` Resource; REFs to Templates are written `@Template`; the identity of an identified edge is its Id (`&employment-1`), not a Name; a composed reference's unmarked base matches a root of any role; and the reference productions are named `RootRef`, `PathRef`, `IdRef`, `RootPathRef` and `RootIdRef`. Alongside that pass: every example is now a complete document (Records and Properties that stood at the root are placed inside an Object; Patterns 6, 9, 11 and 35 no longer root a document in a `REC`, and Pattern 32 no longer roots one in Properties); the `--` and `//` "comments" in Patterns 22 and 28 are replaced by prose, since GDX has no comments; `...` elisions are replaced by valid content; the vocabulary document of Pattern 26 gives its Templates their own Ids (`acme:PersonShape`), since root Ids share one document scope; path examples use Names without `/`, since a path segment cannot contain one; the Abstract notes that stored data carries authorities, not aliases; and a new Pattern 37 shows a persistent aggregate (`HEAD`, Core Spec §6.17), appended so that no other pattern is renumbered.

The paragraphs below describe earlier revisions and are retained as written.

This is the v0.21.0 working draft of the Cookbook, companion to the GDX Core Specification v0.27.0. v0.21.0 is a terminology alignment: the `Rel` field is renamed `Predicate` (Core Spec v0.27.0, entry 66), and the lowercase shorthand "rel" becomes "predicate" in prose. The `#` sigil is unchanged, so no example's authored content changes — the predicate *values* in these patterns (`#gdx:knows`, `#schema:author`, and the rest) are byte-identical to v0.20.0. Pattern 5 (Predicate-Aligned Property) and Pattern 17 (Predicate Declaration) already carried the word in their titles.

The paragraphs below describe earlier revisions and are retained as written.

This is the v0.20.0 working draft of the Cookbook, companion to the GDX Core Specification v0.20.0. v0.20.0 adds a capture-first subsection to Pattern 7: an @-edge's `#rel` may be empty (the relationship exists by role, undefined as to nature, refined later on the same edge — Core Spec §6.4.2). v0.19.0 updates Pattern 22 with the value-resolution framing of REF (Core Spec §6.16) and expands the REF-vs-@-Edge choice guidance, which the Core Specification now routes to this document and the Primer as the non-normative home for borderline-case guidance.

This document was previously the v0.13.0 working draft, companion to the GDX Core Specification v0.13.0. The patterns documented here have stabilized alongside the core; future revisions will add patterns as common use cases emerge but will not remove existing entries except to merge or restructure them. v0.13.0 migrates all leaf-like `@KIND` references to the new `%KIND` REF family (Core Spec §6.16, entry 42 of the v0.13.0 changelog); Pattern 22 is rewritten as "Typed REF — Slot Holding a Handle to an Independent Element" and now includes a reification subsection (REF to an @-edge via `%@OBJ`/`%@REL`); Pattern 27 (Template Version Chain) is updated with the conceptual-shift note (Templates are vocabulary-layer, not graph-entities, so version chains are slot-bindings via `%DEFOBJ`, not graph statements). v0.13.0 also widens REL composition (Core Spec §6.15, entry 43): Pattern 17 (Predicate Declaration) is rewritten to demonstrate Properties (labels, descriptions, alias Sets) alongside @-edges and Extensions on a REL, with Property-vs-Extension choice guidance; Pattern 26 (Complete Vocabulary Document) updated to show the mixed-metadata-kinds pattern on RELs. v0.12.0 migrates `<EDGE …>` examples to `<@OBJ …>` per the @-Edge family (Core Spec §6.4, entry 41 of the v0.12.0 changelog); Pattern 7 (Edge Between Entities) is rewritten to demonstrate the new family including an edge-as-substance example (axiom E6 identity). v0.11.0 migrates leading-slash identifiers in examples to the new hierarchical-local form (no leading `/`); adds Pattern 16 (Parallel Constraint Vocabularies on One Template) demonstrating Layer 2b vocabulary pluralism; adds patterns demonstrating path-mode references (Pattern 23) and key-mode anchor references (Pattern 24); adds Pattern 20 (Adjunct Extension for Non-Schema Metadata) demonstrating Extensions beyond schema constraints; and adds Pattern 26 (Complete Vocabulary Document) showing how RELs with predicate-semantic Extensions, concept Objects, and Templates compose into a self-contained vocabulary.

Changes in v0.10.0 from v0.9.6 reflect Core Specification v0.10.0:

- **Set notation migration.** All `*PREFIX` Set notation has been migrated to `PREFIX*` suffix form (`*STR` → `STR*`, `*REC` → `REC*`). Examples in §3, §4, and §28 (formerly §25) are updated.
- **New `@KIND` reference patterns.** Three new patterns added: §20 (Typed Reference to an Addressable Element), §21 (Vocabulary Metadata Link), §22 (Template Version Chain). These cover the typed-reference primitive added in Core v0.10.0.
- **EDGE-vs-`@OBJ` clarification.** §7 (Edge Between Entities) now distinguishes EDGE's graph-statement role from the `@OBJ` typed-pointer role and cross-references §20.
- **Renumbering.** Former §20–§28 renumbered to §23–§31 to accommodate the inserted reference patterns. Internal cross-references updated.
- **Cross-references.** Compact Form section reference updated from §11.1 to §11.2 (its new location in Core v0.10.0).

---

## Table of Contents

The Cookbook is organized by pattern type:

- **Basic Shapes** (1–9): named Objects, composites, sets, edges, embedded values, qualifiers.
- **Schema and Constraints** (10–16): Templates, semantic types, validation, refinement, layered conformance, parallel constraint vocabularies.
- **Predicates and Vocabulary** (17): REL declarations.
- **Node-Zone, Annotation, and Adjunct Extensions** (18–20): semi-structured material, free-text notes, vocabulary-keyed adjunct statements.
- **References and Vocabularies** (21–27): identifier schemes, typed pointers, path-mode addressing, id references, vocabulary metadata, complete vocabulary documents, version chains.
- **Property State** (28): null, absent, and present.
- **Transport** (29–31): packaging, grouping, vocabulary export.
- **Use-Case Framings** (32–36): typed configuration through graph-structured data.
- **Persistent Aggregates** (37): a HEAD with its Body.

---

## 1. Named Object with Properties

The most common shape: an Object as the root, with Properties as direct children.

```
<OBJ &people/alice Alice
    <STR name : "Alice">
    <INT age : 24>
    <BOOL active : true>
>
```

The root carries an Id (`&people/alice`), its durable identity: persistence and every link and reference bind to it. `Alice` is its Name, a descriptive label free to change. An Object that nothing links to may carry a Name alone, as several later examples do, but it is then reachable only by a structural path; a reference written `@Alice` would not find it, because no reference resolves by Name (Core Spec §4.2.6).

## 2. Composite Property (Record)

When an Object has substructure that should be treated as a composite value, use a Record.

```
<OBJ Order123
    <REC shipping
        <STR carrier : "UPS">
        <STR tracking : "1Z999AA10123456784">
    >
    <FLOAT total : 1250.00>
>
```

The Record contains its children — `shipping` is not a separately addressable entity; it's a structured value belonging to the order.

## 3. Multi-Valued Attribute

Use a Set when an attribute has multiple values of the same type.

```
<OBJ Post42
    <STR title : "GDX is interesting">
    <STR* tags
        <STR : "blue">
        <STR : "red">
        <STR : "green">
    >
>
```

Set members carry their prefix but may omit identifiers — they identify themselves by position and value within the Set.

## 4. Multi-Valued Records

Sets may also carry Records as members. Record members within a Set typically carry meaningful identifiers.

```
<OBJ AddressBook
    <REC* contacts
        <REC alice
            <STR name : "Alice">
            <STR email : "a@example.com">
        >
        <REC bob
            <STR name : "Bob">
            <STR email : "b@example.com">
        >
    >
>
```

## 5. Predicate-Aligned Property

When a property has a shared semantic identity across documents, attach a `#predicate`.

```
<OBJ &people/alice
    <STR #schema:givenName fname : "Alice">
>
```

Two documents using `#schema:givenName` are talking about the same attribute regardless of how they locally name it. The `fname` Property above is semantically equivalent to the `firstName` Property in this document from another system:

```
<OBJ &crm/contact-17
    <STR #schema:givenName firstName : "Alice">
>
```

## 6. Schema-Aligned Slot

A Template declares slots with `Name` fields; an instance satisfies the Template by giving its children the same `Name` values. Cross-vocabulary alignment uses `#Predicate` to bridge differing local names.

```
<DEFREC &schema:address
    <STR #schema:streetName street>
    <STR #schema:city city>
    <STR #schema:usStateCode state>
    <INT #schema:extendedUSZip zip>
>

<OBJ &people/alice
    <REC homeAddress --schema:address
        <STR street : "Broadway">
        <STR city : "Oakland">
        <STR state : "CA">
        <INT zip : 94610>
    >
>
```

The Template is a root, and its Id (`&schema:address`) is what the instance's `--schema:address` resolves to. The instance is a Record, so it sits inside a root Object (a Record never roots a document), and it claims a `DEFREC` because a Record instance may claim only a Record Template (Core Spec §8.2.2).

Instance children align to Template slots by `Name` matching. When local names differ from Template slot names (for example, a localized instance using `prenom` where the Template declares `firstName`), the `#Predicate` field provides the alignment bridge:

```
<OBJ &profiles/fr-17 ProfileFR --schema:Person
    <STR #schema:givenName prenom : "Alice">
    <STR #schema:familyName nom : "Smith">
>
```

Here the instance is an Object claiming an Object Template (`schema:Person`, a `DEFOBJ` not shown). Its children have local `Name`s `prenom` and `nom`, which do not match the Template's slot `Name`s. The shared `#Predicate` values (`schema:givenName`, `schema:familyName`) align them to the corresponding slots.

The `&Id` field is NOT used for Template-instance alignment; the Id is an element's durable machine identity and location-stable reference target, a separate concern from the `Name`-based (and `Predicate`-bridged) alignment shown here (see Pattern 24).

## 7. Edge Between Entities

@-edges (`@OBJ`, `@REL`, `@HEAD`) express directed relationships between graph-entities.

```
<OBJ &alice Alice
    <@OBJ #knows : @bob>
>

<OBJ &bob Bob>
```

This reads as: alice — knows → bob. The edge's Resource `@bob` names the root whose **Id** is `bob`; it never matches a Name, so a target written `<OBJ Bob>` (a Name only) would leave the edge unresolved. An @-edge's Resource is always a plain `@root`: edges join whole roots, and reaching inside one (`&id`, `$path`, `@root&id`, `@root$path`) is a REF's job (Pattern 22; Core Spec §6.4).

@-edges with attributes carry them as Record-like children:

```
<OBJ &alice Alice
    <@OBJ #knows : @bob
        <STR since : "2024-01-15">
        <FLOAT weight : 0.8>
    >
>
```

The edge attributes describe the *relationship*, not either endpoint.

An @-edge with identity is a persistent edge-entity — its substance endures across changes to predicate or attributes, and the edge can be referenced from other documents:

```
<OBJ &alice Alice
    <@OBJ &employment-1 #worksFor : @com.acme
        <STR startDate : "2024-01-15">
        <FLOAT confidence : 0.95>
    >
>
```

The Id `employment-1` is the edge's substance (Core Spec §6.4 axiom E6); the `#worksFor` predicate is its current assertion; the attributes describe the edge itself. Two edges with the same source, predicate, and target are distinct if their Ids differ. An edge may also carry a Name, but a Name only labels it; the identity is the Id. Another document reaches this edge with a REF to `@alice&employment-1` (see the reification example in Pattern 22).

An @-edge asserts a graph statement and admits attribute children. When you want value-resolution instead — a slot whose content is sourced from another element, with no relationship assertion — use the Reference family (Pattern 22, which also carries the guidance for borderline cases).

**Capture-first: the undefined relationship.** An edge's `#predicate` may be omitted entirely. The role asserts that a relationship *exists* (it's an edge); the `#predicate` field specifies *what* it is, and may be empty — the relationship is then undefined as to nature (Core Spec §6.4.2). This is the shape for connections captured before their form is known: real-time capture, pending classification, a link observed before its nature is determined.

```
<OBJ sensor-hub
    <@OBJ &link-0192 : @device-7>
>
```

"sensor-hub stands in some relationship to device-7 — nature pending." Give the edge an `Id` — its durable identity, the substance that endures while the predicate is still pending — so it stays addressable; when the relationship is classified, refine the *same* edge by filling the field:

```
<OBJ sensor-hub
    <@OBJ &link-0192 #gdx:controls : @device-7>
>
```

Same edge, same Id, same target — identity and any references to the edge survive the refinement. Keep undefined relationships a visible backlog, not a resting place: they are queryable as a distinct state ("show all edges with an empty predicate"), and a Template can forbid them where strictness matters (a vocabulary-defined constraint, Core Spec §8.4) — a capture pipeline allows them, a published ontology rejects them.

## 8. Refining a Property with Constraints

Qualifiers attach metadata to a Property without changing its value.

```
<OBJ &sensors/room-4
    <FLOAT temperature : 21.5
        <.STR unit : "celsius">
        <.BOOL observed : true>
    >
>
```

Qualifiers refine the meaning of their parent and travel with the data. A consumer reading the temperature should consider the unit Qualifier when interpreting the value.

## 9. Schema Definition and Materialization

Templates define the shape; instances reference them via `--type`. The Template may declare values for slot metadata (Predicate, default Resource) that the instance is silent about; those values are logically present on the instance via §8.2.3 of the Core Specification.

```
<DEFOBJ &com.acme.Customer
    <STR #schema:givenName firstName>
    <STR #schema:familyName lastName>
    <STR locale : "en-US">
>

<OBJ &customers/alice --com.acme.Customer
    <STR firstName : "Alice">
    <STR lastName : "Smith">
>
```

The customer is an Object, the unit of persistence, so the Template is a `DEFOBJ` (an Object instance may claim only an Object Template, Core Spec §8.2.2); `--com.acme.Customer` resolves to the Template's Id.

The instance is silent about each child's `Predicate`, and silent about `locale` entirely. By the universal claim rule (Core Spec §8.2.3): for `firstName` and `lastName`, the Template-supplied `Predicate`s are the effective values; for `locale`, the Template's default `Resource` is the effective value. The instance logically carries `Predicate`s of `schema:givenName` and `schema:familyName`, and a `locale` of `"en-US"`.

When the document is persisted, exchanged, or archived, **materialize** those Template-supplied values into the instance. The materialized form is what readers see when the schema is not available at read time:

```
<OBJ &customers/alice --com.acme.Customer
    <STR #schema:givenName firstName : "Alice">
    <STR #schema:familyName lastName : "Smith">
    <STR locale : "en-US">
>
```

The materialized instance is self-describing. A reader that does not load `com.acme.Customer` still sees the predicate alignments; a query for elements with `Predicate` `schema:givenName` finds this instance without resolving the Template. Materialization is recommended for persistence; it is optional for in-memory processing where the Template is already available.

Materialization does not extrapolate beyond what the Template explicitly declares. Missing slots are not invented; Constraint Extensions on the Template (Core Spec §8.4) remain Template-scoped and are not copied into the instance.

## 10. Embedded Structured Value (VAL)

When a structured value must occupy a value-bearing slot, use the VAL role. VAL provides a named slot whose value is itself an element.

```
<OBJ contact
    <VAL details
        <REC <STR email : "a@example.com"> <STR phone : "555-0100">>
    >
>
```

The VAL `details` is the slot in the containing Object; the Record occupant is the structured value the slot holds. The slot stays named in the parent's structure; the occupant carries the structured content with its own role intact.

When the slot itself is semantically aligned (the *slot* corresponds to a vocabulary predicate, distinct from the value's own alignment), put the `#predicate` on the VAL:

```
<OBJ employee
    <VAL #gdx:emergencyContact contactSlot
        <OBJ contacts/bob
            <STR #gdx:givenName name : "Bob">
            <STR #gdx:phoneNumber phone : "555-0100">
        >
    >
>
```

The VAL says "this is the employee's emergency-contact slot." The occupant Object says "the value is `contacts/bob`, a Person with these attributes." Two levels of alignment, one structural mechanism.

## 11. Embedded Alternative Representations

A common case for VAL is when an entity has multiple equivalent representations that should travel together. Each representation is a complete Object filling a named slot in the parent.

```
<OBJ &readings/swatch-17 colorReading
    <VAL lab
        <OBJ <FLOAT l : 50.0> <FLOAT a : 25.3> <FLOAT b : -12.7>>
    >
    <VAL rgb
        <OBJ <INT r : 124> <INT g : 89> <INT b : 200>>
    >
>
```

Each VAL names the representation slot (`lab`, `rgb`); each Object occupant is itself a structured entity. The slot identifies which representation the value provides; the occupant carries the structured content. The two-level identity (slot-level on VAL, occupant-level on the contained Object) lets each representation be addressable as a slot in `colorReading` (by path, `$/colorReading/lab`) *and* as an Object in its own right when the occupant carries an Id (by `&id` within the document, or `@readings/swatch-17&id` from another).

## 12. Width-Bounded Numeric Values

When a numeric value must commit to a specific representation width, use a representation suffix.

```
<OBJ &telemetry/frame-881
    <INT packetSize : 1500u16>
    <INT temperature : 25i8>
    <FLOAT angle : 1.5708f32>
    <INT largeBalance : 9999999999999999999bigint>
>
```

The suffix is a Layer 1 (representation) assertion. The value MUST fit the suffix's range; the implementation MUST preserve the representation through round-trip.

## 13. Decimal Precision for Real Numbers

When a real number requires exact decimal representation, use a decimal suffix.

```
<OBJ &orders/123
    <FLOAT price : 25.50d>
    <FLOAT taxRate : 0.0875d>
    <FLOAT coordinate : 37.7749295d128>
>
```

Without a suffix, the implementation chooses a representation. With a decimal suffix, the value is committed to decimal arithmetic semantics — exact for any finite decimal literal.

## 14. Bytes-Typed Properties

Use the `BYTE` prefix and the `0x...` literal form for opaque binary data.

```
<OBJ Contract2024
    <STR title : "Service Agreement">
    <BYTE contentHash : 0xe3b0c44298fc1c149afbf4c8996fb92427ae41e4>
    <BYTE signature : 0x4a3f8c2b9d1e7f0a5c8b6d4e2f1a9c7b8d3e5f0a>
>
```

Hex is the canonical form. For larger byte sequences where compactness matters, the `0z` base64 form is available:

```
<OBJ &attachments/greeting
    <BYTE compactBlob : 0zSGVsbG8sIHdvcmxkIQ==>
>
```

## 15. Combining Layer 1 and Layer 2 Assertions

A single property can carry both a representation suffix (Layer 1) and a Template reference (Layer 2).

```
<OBJ &inventory/bin-12
    <INT count : 25i8 --vocab:positive-i8>
>
```

The suffix asserts representation (`i8` width). The Template reference asserts structural conformance — the element resolves the `vocab:positive-i8` Template (by its `Id`, Core Spec §8.5.2) and aligns to its shape; any value constraint that Template carries (here, greater than zero) is checked separately.

The two assertions operate independently and are validated separately. Per Core Spec §2.2, conformance is layered into three rungs: representation (Layer 1), **structural conformance** (Layer 2 — reference resolution and Template-instance alignment, owned by GDX) and **constraint conformance** (Layer 3 — constraint-vocabulary checks, owned by external vocabularies). The Template reference here is a Layer 2 assertion; the greater-than-zero constraint the Template carries is Layer 3. The next pattern shows how Layer 3 admits multiple constraint vocabularies on a single Template.

## 16. Parallel Constraint Vocabularies on One Template

A single Template MAY carry constraints expressed in multiple constraint vocabularies simultaneously. Each vocabulary's identifiers are opaque to GDX (Core Spec §8.4); GDX records them as well-formed Extensions and assigns no meaning. Consumers each read the vocabularies they understand and ignore the rest.

```
<DEFSTR &UserHandle
    <_STR xsd:pattern : "^[a-z0-9_]+$">
    <_INT xsd:minLength : 3>
    <_INT xsd:maxLength : 32>

    <_STR json:pattern : "^[a-z0-9_]+$">
    <_INT json:minLength : 3>
    <_INT json:maxLength : 32>

    <_STR acme:semantic-class : "account-identifier">
>
```

The Template carries seven Constraint Extensions drawn from three vocabularies (`xsd:`, `json:`, `acme:`). GDX validates that all seven are well-formed Extension carriers structurally attached to the `DEFSTR` Template. It does not interpret any of the constraint identifiers.

Three classes of consumer process this Template:

- An **XSD-aware consumer** reads the `xsd:` constraints, applies XSD validation semantics, and ignores the rest.
- A **JSON-Schema-aware consumer** reads the `json:` constraints, applies JSON-Schema semantics, and ignores the rest.
- A **vendor-specific consumer** reads `acme:semantic-class` to drive domain behavior; it ignores the validation vocabularies entirely.

None of the consumers conflict; all coexist on the same data. This is the parallel-schemas property: GDX is the substrate, the constraint vocabularies ride on top, and pluralism falls out for free.

This pattern is useful when a single document needs to satisfy reviewers in different ecosystems — for example, an integration that ships data into both an XSD-validated pipeline and a JSON-Schema-validated pipeline without forking the document. It is also useful for migrations: a Template may carry both an old constraint vocabulary and a new one during a transition period, with consumers reading whichever they support.

## 17. Predicate Declaration

Use `REL` to declare a predicate as a first-class entity. A REL's purpose is fully discharged by its Id, which a REL MUST carry: it is what a `#predicate` field and an `@REL` edge resolve to. Children are metadata about the predicate, and authors choose what to attach based on what the consumer needs to read.

```
<REL &gdx:knows>

<REL &gdx:manages
    <STR rdfs:label : "manages">
    <STR rdfs:comment : "Asserts a management relationship between two Persons.">
    <STR* aliases <STR : "supervises"> <STR : "leads">>
    <@REL #gdx:inverse : @gdx:reportsTo>
    <_BOOL gdx:transitive : false>
>

<REL &gdx:reportsTo>
```

The bare REL declares a predicate with no metadata. The `@REL` edge resolves to the root REL whose Id is `gdx:reportsTo`, declared here in the same document. The second form attaches a mix of metadata kinds:

- **Properties** (`<STR rdfs:label …>`, `<STR rdfs:comment …>`, `<STR* aliases …>`) carry data about the predicate — labels, descriptions, alternative names. The values are consumer-conferred: a documentation tool reads `rdfs:label`, a search index reads `aliases`. GDX-core records them as data; nothing in the core interprets them.
- **@-edges** (`<@REL #gdx:inverse …>`) carry graph-statement assertions about how the predicate relates to other vocabulary entities. The inverse edge is a first-class statement linking `gdx:manages` to `gdx:reportsTo`.
- **Extensions** (`<_BOOL gdx:transitive …>`) carry primitive-valued vocabulary-defined semantic axioms a validator or reasoner acts on.

**Property vs Extension on a REL.** Both are admitted; choose by what the metadata *is*:

- Property → data for humans or downstream lookup (labels, descriptions, version, authorship)
- Extension → semantic axiom for a validator or reasoner (symmetry, transitivity, cardinality)

Mechanical rule: would a validator/reasoner act on it? → Extension. Is it for humans or downstream lookup? → Property.

REL declarations may appear at the document root or within vocabulary documents.

## 18. Schema-Less Subtree

Use Nodes when material does not fit GDX's structured kinds — imported audit data, ad-hoc payloads, or schema-less metadata.

```
<OBJ Import427
    <STR source : "legacy-system-v3">
    <INT recordCount : 4521>
    <NREC rawAuditData
        <NSTR originalFormat : "csv">
        <NINT columnCount : 12>
        <NREC row1
            <NSTR field1 : "value">
            <NINT field2 : 42>
        >
    >
>
```

Once a Node appears, its subtree is uniformly Node-typed. Structured kinds cannot mix in. Use this pattern when you need to carry data whose shape is not known in advance or not worth modeling.

## 19. Annotation for Provenance

Use `REM` to attach human-readable notes that should travel with the data.

```
<OBJ Order123
    <REM : "Imported from legacy system 2026-04-27">
    <REM : "Reviewed by jmussi 2026-04-28">
    <STR customer : "ACME Corp">
    <FLOAT total : 1250.00d>
>
```

Annotations carry no semantic weight in the data model but survive round-trip serialization. They are appropriate for provenance, audit trails, and editorial notes when the content is free-text intended for human or tooling consumption without a vocabulary-defined interpretation.

When the metadata is keyed to a vocabulary and intended for a specific consumer to interpret, use an Extension instead — see the next pattern.

## 20. Adjunct Extension for Non-Schema Metadata

Extensions (Core Spec §6.10) are not limited to schema-validation constraints on Templates. The same vocabulary-defined adjunct-statement mechanism is used wherever a consumer needs to attach a typed, vocabulary-keyed assertion to an element — provenance, presentation hints, migration directives, audit trails, application-specific metadata. Each Extension is opaque to GDX; the consumer that understands the vocabulary acts on it, others ignore it.

Provenance attached to an Object instance:

```
<OBJ orders/2026-04-29
    <_STR acme:source-system : "legacy-orders-v3">
    <_STR acme:imported-at : "2026-04-29T14:22:10Z">
    <_INT acme:import-batch : 42>
    <STR customer : "ACME Corp">
    <FLOAT total : 1250.00d>
>
```

A presentation hint attached to a Property:

```
<OBJ &reviews/2026/emp-42
    <INT score : 87
        <_STR ui:color : "green">
        <_STR ui:label : "Performance Score">
    >
>
```

A migration directive attached to a Template:

```
<DEFOBJ &CustomerV2
    <STR firstName>
    <STR lastName>
    <STR email>
    <%DEFOBJ #gdx:supersedes : @CustomerV1>
    <_STR migrate:from-v1 : "add-email-from-default">
    <_STR migrate:default-email : "unknown@example.com">
>
```

A Template is a root, so the REF reaches `CustomerV1` with a root reference, `@CustomerV1` (the Template is shown in Pattern 27).

A confidence Extension on an @-edge:

```
<OBJ &alice
    <@OBJ #knows : @bob
        <_FLOAT analytics:confidence : 0.87>
        <_STR analytics:source : "co-authorship-graph">
    >
>
```

In each case, GDX records the Extension as a well-formed element; nothing in GDX-core interprets the vocabulary identifier (`acme:source-system`, `ui:color`, `migrate:from-v1`, `analytics:confidence`). A consumer that understands the vocabulary processes the Extension; one that doesn't ignores it. Multiple vocabularies may coexist on the same element — the same Object, Property, or @-edge may carry `xsd:`, `acme:`, `ui:`, and `analytics:` Extensions side by side, each processed by its respective consumer.

**Extension vs Annotation.** Annotations (Pattern 19) are free-text without a vocabulary identifier; Extensions are vocabulary-keyed and typed. Use Annotation for human commentary that doesn't need machine interpretation; use Extension when a specific consumer needs to recognize the metadata by identifier and act on it programmatically.

**Extension vs Qualifier.** Qualifiers (Pattern 8) are *constitutive* refinements of the parent's value — removing a Qualifier changes what the parent means. Extensions are *adjunct* — removing an Extension does not change the parent's identity, value, or validity. Use Qualifier when the metadata is part of the value's meaning (a unit on a measurement); use Extension when the metadata is an assertion alongside the value (a provenance claim about the measurement).

## 21. Reference with Custom Identifier Scheme

Object Ids are not constrained to any particular scheme. Use whatever fits your system: a local token, a hierarchical local, a GUID (an Id may begin with a digit), a CURIE, or an IRI.

```
<OBJ &alice>
<OBJ &people/alice>
<OBJ &7f3a2b9c-4e21-4f88-91d5-abcdef012345>
<OBJ &gdx:Person>
<OBJ &https://acme.com/people/alice>
```

These are five distinct roots, one per scheme. @-edges to them use the `@OBJ` prefix and name each root by its Id:

```
<OBJ &reading-list
    <@OBJ #knows : @alice>
    <@OBJ #references : @people/alice>
    <@OBJ #describes : @7f3a2b9c-4e21-4f88-91d5-abcdef012345>
    <@OBJ #cites : @https://acme.com/people/alice>
>
```

The resolver finds each root by its Id, first among this document's roots, then in documents loaded alongside, then — for an IRI, or a CURIE that expands to one — by global resolution where the implementation supports it (Core Spec §6.4.1); the format does not constrain the scheme. The Id goes on the `&` field: an Object that carried `people/alice` only as its Name would not be found by `@people/alice`. Ids may not begin with `/` (Core Spec §4.4.1, §4.4.3); the leading-slash form is the path-mode reference syntax (see Pattern 23).

## 22. Typed REF — Slot Holding a Handle to an Independent Element

When a slot's content should be *sourced from another element* — a configuration pointing at a service, a schema-bound instance pointing at its Template, a derived view pulling a value from an interior position in a source document — use the REF family (Core Spec §6.16). A REF is GDX's **value-resolution** construct: "the value here is the value of the resolved target." The document stores the address (that is what round-trips); dereferencing yields the target — a whole element used as a value, or a field inside one. REFs are slot-fillers: they identify the target and stop, with no relationship assertion about the parent and no attribute children. REFs use the `%KIND` prefix, distinguishing them from @-edges (`@OBJ`, `@REL`, `@HEAD`) which assert relationships and belong to graph-entity targets.

Configuration / wiring:

```
<OBJ &ServerConfig
    <STR name : "web-01">
    <%OBJ database : @primary-db>
    <%OBJ fallback : @secondary-db>
    <%DEFOBJ #conformsTo : @ServerConfigV2>
    <%REC sharedDefaults : @global-defaults$/server-defaults>
>
```

`%OBJ` is a REF whose target is an Object; `%DEFOBJ` targets a Template; `%REC` targets a Record at an interior path. The reader and validator know the target's kind from the prefix: `%OBJ` resolves to an Object, `%REC` to a Record, `%BLK` to a Block, `%.STR` to a string Qualifier, `%@OBJ` to an @-edge (reification — see below), and so on.

The literal says how far the reference reaches. `@primary-db` and `@ServerConfigV2` are root references: each names a root (an Object, a Template) by its Id. `@global-defaults$/server-defaults` is a root plus a path, reaching a Record inside the root `global-defaults`. A plain `@root` can only ever find a root, so it is malformed under a prefix whose role is never a root — `%REC`, `%STR`, `%.STR`, `%@OBJ`, a Set form — and such targets are reached with `&id`, `$path`, `@root&id` or `@root$path` instead (Core Spec §4.2.6).

The closed REF family is generated by a single rule: `%` wraps any role-prefix to produce a REF-to-that-role. Qualifiers may refine the slot binding:

```
<OBJ &ServerConfig
    <%OBJ database : @primary-db
        <.STR role : "read-write">
    >
>
```

The Qualifier annotates the slot binding: "this pointer is the read-write database."

**REF vs @-Edge — choosing the right construct.** Both `%OBJ` and `@OBJ` carry a Resource pointing at an Object, but they make different assertions:

- `<%OBJ database : @primary-db>` — "this slot named `database` holds a handle to the root whose Id is `primary-db`; its value resolves to that Object." Value-sourcing, with no claim about a relationship.
- `<@OBJ #usesBackend : @primary-db>` — "the parent stands in a `usesBackend` relationship to `primary-db`." A graph statement, asserting the relationship.

Both read the literal `@primary-db` the same way — the root with that Id — and differ only in what they may address: an @-edge takes a plain `@root` and nothing else, a REF any of the five forms.

For borderline cases, ask what you are asserting, not what you are pointing at:

- *"This field's value is that whole object"* — a REF. Value-sourcing stays a REF even when the target is a whole, identified entity; what makes a REF a REF is that the slot's content comes from the target, not that the target is small or interior.
- *"These two entities are related"* — an @-edge. A relationship is a thing in its own right: it can be labeled, carry attributes, bear identity, and be traversed as graph structure.
- *Need to reach inside an element* (a field, a Qualifier, an interior Record)? — necessarily a REF. Value-resolution naturally reaches interior resources; a graph edge connects whole elements only and never points into one (Core Spec §6.16, §6.4).
- *Should removing it delete a fact about how things relate?* — if yes, it was an @-edge. Removing a REF empties a slot; removing an edge retracts a relationship.

Pick `%` when the slot's content is sourced from elsewhere; pick `@` when you want GDX to record the relationship as a first-class graph statement.

**Reification — REF to an @-edge.** When an @-edge carries identity (per Core Spec §6.4 axiom E6), it becomes addressable as a persistent substance. A REF prefixed `%@OBJ` or `%@REL` is a handle to such an edge:

```
<OBJ &alice
    <@OBJ &knows-bob-2024 #knows : @bob>
>

<OBJ &audit-record
    <%@OBJ source : @alice&knows-bob-2024>
    <STR confidence : "verified">
    <STR auditedBy : "SecurityTeam">
>
```

The edge inside `alice` carries the Id `knows-bob-2024`, its identity. The audit-record's REF `@alice&knows-bob-2024` is a root plus an Id: it finds the root whose Id is `alice`, then the element with Id `knows-bob-2024` inside it. An edge is never a root, so a plain `@knows-bob-2024` would be malformed under `%@OBJ`; within the same document, `&knows-bob-2024` would also reach it.

The audit-record references the specific edge by its Id; the edge itself remains owned by `alice` per axiom E2′. The REF asserts only "this slot holds a handle to that @OBJ-edge" — no claim about a relationship, no claim of ownership. Reification lets external documents make statements *about* a relationship without disturbing the relationship itself.

The two prefixes `%@OBJ` and `%@REL` are the reification forms: a REF whose target is an Object-edge or a REL-edge respectively. This is the canonical mechanism for making statements about graph relationships — the alternative (replicating an edge to make claims about it) loses the identity that makes the assertion specific.

## 23. Reference to a Non-Identified Element via Path

When the target of a reference is not a globally identifiable Object — when it's a Record inside an Object, a specific item in a list, or a Qualifier hanging off a Property — a root reference (`@id`) cannot reach it. If the element carries an Id, an Id reference reaches it (Pattern 24); if it does not, a path reference addresses it by its structural location.

```
<OBJ &orders/2026 order2026
    <REC shipping
        <STR street : "123 Main St">
        <STR city : "Oakland">
    >
    <REC billing
        <STR street : "PO Box 9">
        <STR city : "Berkeley">
    >
>

<OBJ &orders/2026/preferences
    <%REC defaultShippingTo : $/order2026/shipping>
>
```

The Resource `$/order2026/shipping` is a path-mode reference literal (`PathRef`). The `$` introduces path mode; the `/order2026/shipping` traverses from the document root into the root Object whose Name is `order2026` and then into its `shipping` Record. The reference resolves to that Record element. A path stays within the current document.

A path is the one place a Name takes part in addressing, and a path segment cannot contain `/` (it is the separator). That is why the root carries the Name `order2026` for the path to step through, and its durable identity on its Id, `orders/2026`. The same Record is reached from the root's Id with a root plus a path, `@orders/2026$/shipping`, which also works from another document and does not depend on the root's Name.

Targets inside another root compose a root reference with a path:

```
<OBJ &orders/2026/gift-note
    <%REC alicesPreferredAddress : @people/alice$/addresses/[1]>
>
```

This `RootPathRef` is a root plus a path. The `@people/alice` portion resolves the root Object whose Id is `people/alice` (a root reference never matches a Name); the `$/addresses/[1]` portion then traverses from that Object to its `addresses` child and then to that child's first element by document order.

Path segments select by different fields depending on the segment's first character:

- A bareword (`shipping`, `addresses`) matches by `Name`. It cannot contain `/`.
- A `#`-prefixed token (`#gdx:knows`) matches by `Predicate`.
- A bracketed numeral (`[1]`, `[3]`) matches the *N*-th child in document order, 1-based.

A path used as a reference Resource MUST resolve to exactly one element of the matching role; zero matches, multiple matches, and matches of the wrong role all yield an unresolved reference. The GDX Path Expressions Specification (`GDX-Path.md`) gives the full grammar and resolution rules.

Id-based addressing — for elements that carry an `Id` field (their durable machine identity) — is covered in Pattern 24; it provides location-stable references that survive document restructuring.

## 24. Reference to an Id-Bearing Element via Base

When an element carries an `Id` field, that Id is its **durable machine identity** — a fixed handle independent of the element's Name and of where it sits in the document tree, and the thing persistence and durable graph links bind to. Because it is location-stable, it is the recommended target for references that must survive document restructuring: long-lived archival references, cross-document audit links, and references into Sets of similar elements (log entries, transactions, events). (An element's `Name`, by contrast, is a *descriptive* label used for alignment, path segments and display; no reference resolves by it. Renaming an element does not change its identity, but changing its Id does.) When the id-bearing element is a **root-level entity** (a top-level Object, REL, or HEAD), its Id is a *root id*: a root reference (`@bob-id`) reaches it from any document, and it may be the target of an `@`-edge — `<@OBJ #knows : @bob-id>` — as well as of a `%`-reference; an Id on an interior element is REF-addressable only, by `&id` within its document or `@root&id` from anywhere (Core Specification §4.2.6, §4.4.5, §6.4).

```
<OBJ &system/audit-log
    <REC* entries
        <REC &entry-12345 #log:event
            <STR severity : "warn">
            <STR message : "auth failure">
            <STR timestamp : "2026-05-20T08:14:33Z">
        >
        <REC &entry-12346 #log:event
            <STR severity : "info">
            <STR message : "session start">
            <STR timestamp : "2026-05-20T08:15:01Z">
        >
    >
>

<OBJ &compliance/incident-7
    <%REC originatingEvent : @system/audit-log&entry-12345>
>
```

The reference `@system/audit-log&entry-12345` is a `RootIdRef`. The `@system/audit-log` portion resolves the audit-log root Object by its Id; the `&entry-12345` portion then looks up the id-bearing element within that Object's scope. The reference resolves to the Record with that Id.

If the audit log is later restructured — entries archived into nested categories, reorganized by severity, etc. — the reference continues to resolve as long as the Id is preserved. A path-based equivalent like `@system/audit-log$/entries/[1]` would break under any such change.

When the referencing element shares a document with the id-bearing target, the `@Base` portion may be omitted; bare `&entry-12345` is an `IdRef` that searches outward from the reference to the document root (Core Specification §4.4.5). It never leaves its document. Placed in the same document as the audit log above, this root reaches the same Record:

```
<OBJ &compliance/internal-incident
    <%REC originating : &entry-12345>
>
```

Id uniqueness is scoped to the enclosing Object (or to the document for transport-layer elements); see Core Specification §4.4.5. Two different audit-log Objects MAY each have their own `&entry-12345` without collision. A bare `&entry-12345` searches outward region by region and the first region holding the Id decides; if that region holds it twice — two such logs in one document, referenced from outside both — the reference is ambiguous and is left unresolved. A `RootIdRef` such as `@system/audit-log&entry-12345` pins the scope to one root.

The base of a composed reference need not be an Object. An unmarked base matches a root of any root-eligible role — Object, REL, HEAD, Template, Package or Group — since root Ids are unique within a document and the Id alone picks the root. For an id-bearing annotation on a REL declaration, or an id-bearing slot-documentation inside a Template, an explicit base role marker MAY be added as a check. The marker is one of the closed prefix-vocabulary keywords (`OBJ`, `REL`, `HEAD`, `DEFOBJ`, `PKG`, `GRP`, etc.) followed by `:`:

```
<REL &gdx:knows
    <STR &description : "Asserts that the subject knows the object.">
>

<OBJ &vocabulary/docs
    <%STR knowsDoc : @REL:gdx:knows&description>
>
```

The `@REL:gdx:knows` base resolves to the root whose Id is `gdx:knows` and checks that it is a REL; without the marker, `@gdx:knows&description` resolves to the same element, and a mismatch between marker and root leaves the reference unresolved (Core Spec §4.2.6, §4.4.5).

XML practitioners may recognize this pattern as the equivalent of `xml:id`. Convention: when converting GDX ↔ XML, GDX's `Id` corresponds to XML's `xml:id` attribute.

## 25. Vocabulary Metadata Link

REL declarations may reference other RELs to express vocabulary relationships — inverse, sameAs, subPropertyOf, and similar predicates that describe how predicates relate to each other.

```
<REL &gdx:manages
    <@REL #gdx:inverse : @gdx:reportsTo>
>

<REL &gdx:reportsTo>
```

The `@REL` child of `gdx:manages` declares that its inverse predicate is `gdx:reportsTo`. A consumer that understands the `gdx:inverse` vocabulary can traverse a "manages" edge in reverse by following this link.

Schema cross-references between predicates and Templates use the same mechanism:

```
<REL &gdx:hasAuthor
    <%DEFOBJ #gdx:range : @Person>
>

<DEFOBJ &Person
    <STR name>
>
```

The `%DEFOBJ` child of `gdx:hasAuthor` declares that the predicate's range is the `Person` Template — instances reached through `gdx:hasAuthor` edges should conform to the `Person` contract. The Template is a root, so the REF names it with a root reference, `@Person`, which resolves to the Template's Id. Note: this is a REF (slot pointing at a Template, not a graph statement) because Templates are vocabulary-layer artifacts, not graph-entities; only `@OBJ`, `@REL` and `@HEAD` are in the @-Edge family. For domain/range pointing at *concept-Objects* (which are Objects and therefore graph-entities), use `@OBJ` instead: `<@OBJ #gdx:domain : @acme:Person>`.

## 26. Complete Vocabulary Document

A GDX vocabulary document defines the predicates, concepts, and schemas of a domain as a single self-contained document. Predicates are REL declarations; concepts are Objects with conceptual Ids; schemas are Templates. Because all three are GDX, the entire vocabulary is parseable and queryable with the same tools as the instance data it describes.

```
<OBJ &acme:Concept
    <STR rdfs:label : "Concept">
    <STR rdfs:comment : "Top-level concept in the acme: vocabulary.">
>

<OBJ &acme:Person
    <STR rdfs:label : "Person">
    <@OBJ #rdfs:subClassOf : @acme:Concept>
>

<OBJ &acme:Organization
    <STR rdfs:label : "Organization">
    <@OBJ #rdfs:subClassOf : @acme:Concept>
>

<REL &acme:knows
    <STR rdfs:label : "knows">
    <STR rdfs:comment : "Asserts an acquaintance or social relationship.">
    <@OBJ #acme:domain : @acme:Person>
    <@OBJ #acme:range  : @acme:Person>
    <_BOOL acme:symmetric : true>
>

<REL &acme:employs
    <STR rdfs:label : "employs">
    <@OBJ #acme:domain : @acme:Organization>
    <@OBJ #acme:range  : @acme:Person>
    <@REL #acme:inverse : @acme:employedBy>
>

<REL &acme:employedBy
    <STR rdfs:label : "employed by">
    <@OBJ #acme:domain : @acme:Person>
    <@OBJ #acme:range  : @acme:Organization>
>

<DEFOBJ &acme:PersonShape
    <STR #acme:givenName given>
    <STR #acme:familyName family>
    <REC* #acme:knows knowsList>
>

<DEFOBJ &acme:OrganizationShape
    <STR #acme:legalName name>
    <REC* #acme:employs employees>
>
```

Every root carries an Id, and the @-edges resolve to those Ids. Root Ids share one document scope, so the Templates take their own Ids (`acme:PersonShape`, `acme:OrganizationShape`) rather than reusing the concept Objects' Ids; two roots with the Id `acme:Person` would make the document malformed (Core Spec §4.4.5).

This vocabulary document defines:

- **Three concept Objects** (`acme:Concept`, `acme:Person`, `acme:Organization`), each addressable by its Id from other vocabularies and instances. The `acme:Concept` Object plays the role a Class or top-of-hierarchy would in OWL; `rdfs:subClassOf` links subordinate concepts to it.
- **Three REL declarations** (`acme:knows`, `acme:employs`, `acme:employedBy`) carrying a mix of metadata kinds: Properties for human-facing labels and descriptions (`<STR rdfs:label …>`, `<STR rdfs:comment …>`), @-edges for predicate-axiom statements (`<@OBJ #acme:domain …>`, `<@REL #acme:inverse …>`), and Extensions for primitive-valued semantic axioms (`<_BOOL acme:symmetric …>`). The choice between Property and Extension follows the "data vs axiom" rule (Pattern 17).
- **Two Templates** (`acme:PersonShape`, `acme:OrganizationShape`) that bind instance shape to the concepts. The Template slots carry `#predicate` references aligning slot semantics to the vocabulary's predicates.

The document is one GDX file. A consumer reading it gets the predicates, the concept taxonomy, the schemas, and the alignment between them in a single parse. An instance document claiming `--acme:PersonShape` can be validated against the Template, traversed through `acme:knows` edges, and aligned to other vocabularies through `#rdfs:subClassOf` and similar cross-references.

This pattern combines the smaller patterns (17 Predicate Declaration, 25 Vocabulary Metadata Link, schema patterns) into a coherent unit. The vocabulary itself is data; nothing in this document requires a separate schema language or external metadata layer.

## 27. Template Version Chain

Templates can reference other Templates to declare version succession, supersession, or compatibility. This makes vocabulary evolution self-describing.

```
<DEFOBJ &CustomerV1
    <STR firstName>
    <STR lastName>
>

<DEFOBJ &CustomerV2
    <STR firstName>
    <STR lastName>
    <STR email>
    <%DEFOBJ #gdx:supersedes : @CustomerV1>
>
```

The `%DEFOBJ` element inside `CustomerV2` declares that this Template's `supersedes` slot points at `CustomerV1`, the root Template whose Id is `CustomerV1`. A migration tool consuming both documents can follow the chain to understand which Templates are predecessors and which are successors.

The same mechanism works for marking deprecation:

```
<DEFOBJ &Customer
    <%DEFOBJ #gdx:supersededBy : @CustomerV2>
    <STR firstName>
    <STR lastName>
>
```

**Note on the conceptual shift.** Template version chains are expressed as REFs (`%DEFOBJ`), not as @-edges. Under the GDX partition, the @-Edge family is restricted to targets that are independent graph-entities — Objects, REL declarations and HEADs. Templates are vocabulary-layer artifacts; pointing at a Template is slot-filling (a slot whose value is a handle to a Template), not graph-statement-making. Authors who think of version chains as "relationships between Templates" should read the structure as "slots holding handles to predecessor/successor Templates, semantically aligned via `#predicate` to the vocabulary's version predicates." The semantic interpretation is up to the consumer (migration tools, schema registries); GDX records the slot bindings, not graph statements about Templates.

These are vocabulary patterns — the specific predicates (`gdx:supersedes`, `gdx:supersededBy`) are defined by the GDX Common Vocabulary or application-specific vocabularies, not by the core specification. What the core specification provides is the `%DEFOBJ` mechanism for typed Template handles; the vocabulary provides the predicates that align the slots to shared meaning.

## 28. Null, Absent, and Present

A Property may exist in three distinct states with different meanings.

**Property absent** — not part of this instance:

```
<OBJ Customer
    <STR firstName : "Alice">
    <STR lastName : "Smith">
>
```

**Property null** — present but no value (analogous to SQL `NULL`):

```
<OBJ Customer
    <STR firstName : "Alice">
    <STR middleName>
    <STR lastName : "Smith">
>
```

The empty-value form is equivalent and accepted on input:

```
<OBJ Customer
    <STR firstName : "Alice">
    <STR middleName : >
    <STR lastName : "Smith">
>
```

Both forms denote null. The bare form (no `:`) is canonical for serialization; the empty-value form (`: ` with no value) is supported because it arises naturally during editing — when a user removes a value from a present element by deleting the literal but leaving the separator. Conforming serializers normalize the empty-value form to the bare form on output.

**Property present with value** — has an explicit value:

```
<OBJ Customer
    <STR firstName : "Alice">
    <STR middleName : "Marie">
    <STR lastName : "Smith">
>
```

A consumer reading the absent form knows nothing about middleName — it may not apply, may not have been collected, or may not exist in the source data.

A consumer reading the null form knows that middleName was *considered* and is *unknown*.

A consumer reading the present form knows the value of middleName.

The three-state distinction applies to all Resource-bearing kinds. For an integer, `quantity` absent:

```
<OBJ Order123 <STR customer : "ACME">>
```

`quantity` null — it is unknown:

```
<OBJ Order123 <STR customer : "ACME"> <INT quantity>>
```

`quantity` present with a value (zero is a value, not empty):

```
<OBJ Order123 <STR customer : "ACME"> <INT quantity : 0>>
```

For string-typed Properties only, there is a fourth conceptual state — present with empty value:

```
<OBJ Customer
    <STR firstName : "Alice">
    <STR middleName : "">
    <STR lastName : "Smith">
>
```

The empty string is a legitimate string value, distinct from null. A consumer reading this form knows that middleName was considered and is known to be empty (the customer has no middle name, or has an empty middle name in their record). This empty state applies only to strings; for integers, floats, bools, bytes, and references, there is no "empty value" concept — `<INT count : 0>` is the integer zero (a value), not "empty integer".

## 29. Transport Wrapper for Multiple Objects

When transporting multiple Objects together with metadata about the transport, wrap them in a Package.

```
<PKG customer-import-2026-04-29
    <STR source : "crm-v3">
    <STR contentType : "customer-batch">
    <INT count : 3>
    
    <OBJ &customers/alice Alice
        <STR name : "Alice">
        <STR email : "alice@example.com">
    >
    <OBJ &customers/bob Bob
        <STR name : "Bob">
        <STR email : "bob@example.com">
    >
    <OBJ &customers/carol Carol
        <STR name : "Carol">
        <STR email : "carol@example.com">
    >
>
```

The Package metadata (source, contentType, count) describes the transport; the contained Objects are extracted and stored individually by the receiver. The Package itself is discarded after unpacking. Each payload Object carries an Id, the durable identity it is stored under and that links to it bind to once it is at rest; the Names are labels only. While in transit the payload Objects are sibling roots, so an edge or REF inside one reaches another by `@customers/bob` at the first resolution tier (Core Spec §6.4.1).

## 30. Transport Partitioning with Groups

When a Package conveys multiple distinct collections, partition them with named Groups.

```
<PKG color-libraries-2026-04-29
    <STR source : "color-management-system">
    <STR exportType : "full">
    
    <GRP pantone
        <STR libraryName : "Pantone Solid Coated">
        <STR version : "2023">
        <OBJ &Pantone-485-C
            <FLOAT l : 47.2>
            <FLOAT a : 67.8>
            <FLOAT b : 54.1>
        >
        <OBJ &Pantone-186-C>
    >
    
    <GRP ral
        <STR libraryName : "RAL Classic">
        <STR version : "2022">
        <OBJ &RAL-3020>
        <OBJ &RAL-9010>
    >
>
```

The colours after the first are shown without their properties to keep the example short. Each Group carries its own metadata (libraryName, version) describing the partition. Groups are unpacked alongside the Package; the contained Objects retain their independent identity and lifecycle.

## 31. Vocabulary Export with Groups

Vocabularies that contain many Templates and RELs can use Groups to organize related declarations.

```
<PKG gdx-standard-vocab-v1
    <STR vocabName : "gdx-standard">
    <STR version : "1.0.0">
    <REM : "GDX Common Vocabulary export">
    
    <GRP numerics
        <REM : "Width-bounded numeric types">
        <DEFINT &gdx:i8
            <_INT gdx:min : -128>
            <_INT gdx:max : 127>
        >
        <DEFINT &gdx:i16
            <_INT gdx:min : -32768>
            <_INT gdx:max : 32767>
        >
        <DEFFLOAT &gdx:f64>
    >
    
    <GRP predicates
        <REM : "Common relational predicates">
        <REL &gdx:knows>
        <REL &gdx:manages
            <@REL #gdx:inverse : @gdx:reportsTo>
        >
        <REL &gdx:reportsTo>
    >
>
```

The receiver imports the vocabulary by extracting all contained Templates and RELs; the Group organization is informational and does not affect how the receiver stores or uses the imported items. Each Template and REL carries the Id that `--type`, `#predicate` and `@REL` resolve to. The inverse is an `@REL` edge, as in Pattern 17, and it resolves to `gdx:reportsTo` because roots carried in the same Package or Group are checked first (Core Spec §6.4.1). The vocabulary metadata is illustrative; `gdx:min` and `gdx:max` stand for whatever constraint vocabulary the export uses.

## 32. Use as Typed Configuration

GDX at its simplest is typed key-value configuration. Use Properties as the direct children of a single root Object; no nesting, no schema, no predicates. Comparable in shape to YAML or TOML configuration but with explicit types and no inference traps. (Properties cannot stand at the document root themselves; a root must be root-eligible, Core Spec §7.2 rule 1.)

```
<OBJ &config/app
    <STR databaseHost : "db.internal">
    <INT databasePort : 5432>
    <BOOL tlsEnabled : true>
    <STR logLevel : "info">
    <FLOAT timeoutSeconds : 30.0>
>
```

A team using GDX at this level gets explicit typing (no inference about whether `5432` is an integer or a string) and round-trip stability without learning anything beyond the primitive Property prefixes.

The Compact Form (§11.2) is line-oriented and indentation-based, similar in shape to YAML, and serves the same role with even less syntactic noise.

## 33. Use as Structured Application Data

Adding Records and Sets to Properties yields nested structured data, comparable in shape to JSON.

```
<OBJ Server
    <STR name : "web-01">
    <REC network
        <STR host : "10.0.1.42">
        <INT port : 8080>
        <BOOL tlsEnabled : true>
    >
    <STR* allowedOrigins
        <STR : "https://app.example.com">
        <STR : "https://admin.example.com">
    >
>
```

This level uses Objects, Records, and Sets but nothing beyond the structural data-layer kinds. No predicates, no schemas, no graphs. Familiar nested-object shape with explicit types throughout.

## 34. Use as Semantically-Aligned Interchange

Adding `#predicate` fields to Properties enables cross-system semantic alignment without committing to a triple model.

```
<OBJ Server
    <STR #vocab:hostName name : "web-01">
    <STR #vocab:ipAddress address : "10.0.1.42">
    <INT #vocab:portNumber port : 8080>
>
```

Two systems can exchange data even when they use different local field names because the predicate identifies the shared meaning. This is RDF-style semantics layered onto a document-friendly format — no separate triple store, no SPARQL, no ontology stack required.

## 35. Use as Schema-Validated Contracts

Adding Templates declares the shape an instance is expected to satisfy. Templates are themselves GDX elements, with the same syntax and parsing as data.

```
<DEFOBJ &vocab:Server
    <STR name>
    <STR address>
    <INT port>
>

<OBJ &servers/web01 web01 --vocab:Server
    <STR name : "web-01">
    <STR address : "10.0.1.42">
    <INT port : 8080>
>
```

The server is an Object, so its Template is a `DEFOBJ` (Core Spec §8.2.2), and `--vocab:Server` resolves to the Template's Id. Templates declare the contract; instances claim to satisfy it; validators check. The Template can travel in the same document as the instances it describes, or in a separate vocabulary file that consumers resolve via reference.

## 36. Use as Graph-Structured Data

Adding @-edges and IRI-form Object Ids yields fully graph-structured data with relationships, attributes on relationships, and external resolvability.

```
<OBJ &https://infra.acme.com/server/web01
    <STR name : "web-01">
    <@OBJ #vocab:dependsOn : @https://infra.acme.com/db/primary
        <STR since : "2024-01-15">
    >
    <@OBJ #vocab:routesTo : @https://infra.acme.com/cache/edge>
>
```

Relationships are explicit, traversable, and queryable. The Object Ids are full IRIs — globally resolvable and interoperable with RDF-aware systems: each `@https://…` edge target is found by its Id, in a loaded document or, where the implementation supports it, by network resolution (Core Spec §6.4.1). The same document can be parsed as a tree, stored as relational rows, traversed as a graph, or exchanged with any RDF-aware system that resolves the IRIs.

The progression across Patterns 32 through 36 demonstrates that the same format scales from typed configuration to graph-structured data without changing parsers, tooling, or syntax. Applications use the level of feature engagement that fits their problem.

## 37. Persistent Aggregate (HEAD)

When a document is one persistent thing that needs metadata about the thing as a whole — a status, a revision, a retention class — wrap its principal content in a `HEAD` (Core Spec §6.17). A HEAD holds exactly one **Body**, an Object or a REL declaration, together with named typed **slots** (Properties and Records) and adjuncts (Extensions and Annotations).

```
<HEAD &reports/2026-q3
    <STR status : "draft">
    <INT revision : 3>
    <REC retention
        <STR class : "finance">
        <INT years : 7>
    >
    <REM : "Awaiting sign-off">
    <OBJ report
        <STR title : "Quarterly report">
        <STR period : "2026-Q3">
    >
>

<OBJ &teams/finance
    <@HEAD #acme:publishes : @reports/2026-q3>
>
```

The Body is the `report` Object: it is the HEAD's only child of an entity role, so it needs no marker, Name or position to be found (H3). `status`, `revision` and `retention` are slots describing the aggregate; the `REM` is an adjunct. A HEAD with no Body, or with two, is malformed.

The HEAD's Id, `reports/2026-q3`, is a root Id. It identifies the aggregate, not the report it carries, and it is what the `@HEAD` edge from `teams/finance` resolves to. The Body is owned interior content: while it sits in the HEAD it is not an @-edge target (H4, H7), and Ids inside the HEAD are interior Ids, reached by a REF (`@reports/2026-q3&id`, or `@reports/2026-q3$/report/title` by path). The Body is replaceable — a new revision of the report can take its place without changing the HEAD's identity (H9).

Choose between the three wrappers by what the boundary is for. A Package wraps any number of roots for transport and is discarded on receipt; a HEAD wraps exactly one entity as a persistent aggregate and is retained; an Object on its own has children but no division between an envelope and a body. Wrapping is never required: an unwrapped root Object stays targetable by `@OBJ` (H8). A HEAD MAY claim a `DEFHEAD` Template, which aligns its slots by Name and validates its Body structurally rather than by Name (Core Spec §6.14); a HEAD never contains another HEAD and appears only at the document root, as Package or Group payload, or as a VAL occupant.

