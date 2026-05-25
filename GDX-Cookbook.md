# GDX — Cookbook of Canonical Patterns

- **Status:** Draft v0.13.0 (companion to GDX Core Specification v0.13.0, working draft, pre-1.0)
- **Date:** 2026-05-20
- **Editors:** 
- **Authors:** Julian Mussi


## Abstract

This document is a companion to the GDX Core Specification. It is non-normative: it documents idiomatic GDX shapes for common modeling tasks. The patterns demonstrate the structural and schema mechanisms defined by the core specification (§4 through §8); they do not introduce new semantics.

Section references of the form §4, §6, §8 etc. refer to the GDX Core Specification.

The vocabulary identifiers used in these examples (`gdx:`, `schema:`, `vocab:`, etc.) are illustrative. Specific vocabulary content is defined by the relevant vocabulary specification, not by this document.

## Status of This Document

This is the v0.13.0 working draft of the Cookbook, companion to the GDX Core Specification v0.13.0. The patterns documented here have stabilized alongside the core; future revisions will add patterns as common use cases emerge but will not remove existing entries except to merge or restructure them. v0.13.0 migrates all leaf-like `@KIND` references to the new `%KIND` REF family (Core Spec §7.16, entry 42 of the v0.13.0 changelog); Pattern 22 is rewritten as "Typed REF — Slot Holding a Handle to an Independent Element" and now includes a reification subsection (REF to an @-edge via `%@OBJ`/`%@REL`); Pattern 27 (Template Version Chain) is updated with the conceptual-shift note (Templates are vocabulary-layer, not graph-entities, so version chains are slot-bindings via `%DEFOBJ`, not graph statements). v0.12.0 migrates `<EDGE …>` examples to `<@OBJ …>` per the @-Edge family (Core Spec §7.4, entry 41 of the v0.12.0 changelog); Pattern 7 (Edge Between Entities) is rewritten to demonstrate the new family including an edge-as-substance example (axiom E6 identity). v0.11.0 migrates leading-slash identifiers in examples to the new hierarchical-local form (no leading `/`); adds Pattern 16 (Parallel Constraint Vocabularies on One Template) demonstrating Layer 2b vocabulary pluralism; adds patterns demonstrating path-mode references (Pattern 23) and key-mode anchor references (Pattern 24); adds Pattern 20 (Adjunct Extension for Non-Schema Metadata) demonstrating Extensions beyond schema constraints; and adds Pattern 26 (Complete Vocabulary Document) showing how RELs with predicate-semantic Extensions, concept Objects, and Templates compose into a self-contained vocabulary.

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
- **References and Vocabularies** (21–27): identifier schemes, typed pointers, path-mode addressing, key-mode anchors, vocabulary metadata, complete vocabulary documents, version chains.
- **Property State** (28): null, absent, and present.
- **Transport** (29–31): packaging, grouping, vocabulary export.
- **Use-Case Framings** (32–36): typed configuration through graph-structured data.

---

## 1. Named Object with Properties

The most common shape: an Object as the root, with Properties as direct children.

```
<OBJ Alice
    <STR name : "Alice">
    <INT age : 24>
    <BOOL active : true>
>
```

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

When a property has a shared semantic identity across documents, attach a `#rel`.

```
<STR #schema:givenName fname : "Alice">
```

Two documents using `#schema:givenName` are talking about the same attribute regardless of how they locally name it. The element above is semantically equivalent to one written elsewhere as:

```
<STR #schema:givenName firstName : "Alice">
```

## 6. Schema-Aligned Slot

A Template declares slots with `Id` fields; an instance satisfies the Template by giving its children the same `Id` values. Cross-vocabulary alignment uses `#Rel` to bridge differing local names.

```
<DEFREC schema:address
    <STR #schema:streetName street>
    <STR #schema:city city>
    <STR #schema:usStateCode state>
    <INT #schema:extendedUSZip zip>
>

<REC MyAddress --schema:address
    <STR street : "Broadway">
    <STR city : "Oakland">
    <STR state : "CA">
    <INT zip : 94610>
>
```

Instance children align to Template slots by `Id` matching. When local names differ from Template slot names (for example, a localized instance using `prenom` where the Template declares `firstName`), the `#Rel` field provides the alignment bridge:

```
<REC ProfileFR --schema:Person
    <STR #schema:givenName prenom : "Alice">
    <STR #schema:familyName nom : "Smith">
>
```

Here the instance children have local `Id`s `prenom` and `nom`, which do not match the Template's slot `Id`s. The shared `#Rel` values (`schema:givenName`, `schema:familyName`) align them to the corresponding slots.

The `&Key` field is NOT used for Template-instance alignment; Keys are a separate location-stable anchor mechanism for references (see Pattern 22).

## 7. Edge Between Entities

@-edges (`@OBJ`, `@REL`) express directed relationships between graph-entities.

```
<OBJ Alice
    <@OBJ #knows : @Bob>
>
```

This reads as: alice — knows → bob.

@-edges with attributes carry them as Record-like children:

```
<OBJ Alice
    <@OBJ #knows : @Bob
        <STR since : "2024-01-15">
        <FLOAT weight : 0.8>
    >
>
```

The edge attributes describe the *relationship*, not either endpoint.

An @-edge with identity is a persistent edge-entity — its substance endures across changes to predicate or attributes, and the edge can be referenced from other documents:

```
<OBJ Alice
    <@OBJ employmentEdge #worksFor : @com.acme
        <STR startDate : "2024-01-15">
        <FLOAT confidence : 0.95>
    >
>
```

The identifier `employmentEdge` is the edge's substance (per §7.4 axiom E6); the `#worksFor` predicate is its current assertion; the attributes describe the edge itself. Two edges with the same source, predicate, and target are distinct if their identities differ.

An @-edge asserts a graph statement and admits attribute children. When you want a leaf-like typed pointer instead (no graph-statement framing, no attribute children), use the Reference family (§22). Use an @-edge when the relationship matters and the target is a graph-entity (Object or REL); use a Reference for typed pointers to non-graph-entities (Templates, interior elements addressed by path or key).

## 8. Refining a Property with Constraints

Qualifiers attach metadata to a Property without changing its value.

```
<FLOAT temperature : 21.5
    <.STR unit : "celsius">
    <.BOOL observed : true>
>
```

Qualifiers refine the meaning of their parent and travel with the data. A consumer reading the temperature should consider the unit Qualifier when interpreting the value.

## 9. Schema Definition and Materialization

Templates define the shape; instances reference them via `--type`. The Template may declare values for slot metadata (Rel, default Resource) that the instance is silent about; those values are logically present on the instance via §8.2.3 of the Core Specification.

```
<DEFREC com.acme.Customer
    <STR firstName #schema:givenName>
    <STR lastName #schema:familyName>
    <STR locale : "en-US">
>

<REC alice --com.acme.Customer
    <STR firstName : "Alice">
    <STR lastName : "Smith">
>
```

The instance is silent about each child's `Rel`, and silent about `locale` entirely. By the universal claim rule (Core Spec §9.2.3): for `firstName` and `lastName`, the Template-supplied `Rel`s are the effective values; for `locale`, the Template's default `Resource` is the effective value. The instance logically carries `Rel`s of `schema:givenName` and `schema:familyName`, and a `locale` of `"en-US"`.

When the document is persisted, exchanged, or archived, **materialize** those Template-supplied values into the instance. The materialized form is what readers see when the schema is not available at read time:

```
<REC alice --com.acme.Customer
    <STR firstName #schema:givenName : "Alice">
    <STR lastName #schema:familyName : "Smith">
    <STR locale : "en-US">
>
```

The materialized instance is self-describing. A reader that does not load `com.acme.Customer` still sees the predicate alignments; a query for elements with `Rel` `schema:givenName` finds this instance without resolving the Template. Materialization is recommended for persistence; it is optional for in-memory processing where the Template is already available.

Materialization does not extrapolate beyond what the Template explicitly declares. Missing slots are not invented; Constraint Extensions on the Template (Core Spec §9.4) remain Template-scoped and are not copied into the instance.

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

When the slot itself is semantically aligned (the *slot* corresponds to a vocabulary predicate, distinct from the value's own alignment), put the `#rel` on the VAL:

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
<REC colorReading
    <VAL lab
        <OBJ <FLOAT l : 50.0> <FLOAT a : 25.3> <FLOAT b : -12.7>>
    >
    <VAL rgb
        <OBJ <INT r : 124> <INT g : 89> <INT b : 200>>
    >
>
```

Each VAL names the representation slot (`lab`, `rgb`); each Object occupant is itself a structured entity. The slot identifies which representation the value provides; the occupant carries the structured content. The two-level identity (slot-level on VAL, occupant-level on the contained Object) lets each representation be addressable as a slot in `colorReading` *and* as an Object in its own right when occupant identifiers are present.

## 12. Width-Bounded Numeric Values

When a numeric value must commit to a specific representation width, use a representation suffix.

```
<INT packetSize : 1500u16>
<INT temperature : 25i8>
<FLOAT angle : 1.5708f32>
<INT largeBalance : 9999999999999999999bigint>
```

The suffix is a Layer 1 (representation) assertion. The value MUST fit the suffix's range; the implementation MUST preserve the representation through round-trip.

## 13. Decimal Precision for Real Numbers

When a real number requires exact decimal representation, use a decimal suffix.

```
<FLOAT price : 25.50d>
<FLOAT taxRate : 0.0875d>
<FLOAT coordinate : 37.7749295d128>
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
<BYTE compactBlob : 0zSGVsbG8sIHdvcmxkIQ==>
```

## 15. Combining Layer 1 and Layer 2 Assertions

A single property can carry both a representation suffix (Layer 1) and a Template reference (Layer 2).

```
<INT count : 25i8 --vocab:positive-i8>
```

The suffix asserts representation (`i8` width). The Template reference asserts schema conformance (the value satisfies the `vocab:positive-i8` Template, which might constrain the value to be greater than zero).

The two layers operate independently and are validated separately. Per Core Spec §2.2, conformance is further decomposed: Layer 2 splits into Layer 2a (Template structural conformance — owned by GDX) and Layer 2b (constraint-vocabulary conformance — owned by external vocabularies). The next pattern shows how Layer 2b admits multiple constraint vocabularies on a single Template.

## 16. Parallel Constraint Vocabularies on One Template

A single Template MAY carry constraints expressed in multiple constraint vocabularies simultaneously. Each vocabulary's identifiers are opaque to GDX (Core Spec §9.4); GDX records them as well-formed Extensions and assigns no meaning. Consumers each read the vocabularies they understand and ignore the rest.

```
<DEFSTR UserHandle
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

Use `REL` to declare a predicate as a first-class entity.

```
<REL gdx:knows>

<REL gdx:manages
    <_STR vocab:inverse : "gdx:reportsTo">
>
```

The bare REL declares a predicate with no semantic refinements. The second form declares that `gdx:manages` has an inverse predicate `gdx:reportsTo` — a vocabulary-defined refinement that consumers may use to compute inverse-direction queries.

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

Extensions (§7.10) are not limited to schema-validation constraints on Templates. The same vocabulary-defined adjunct-statement mechanism is used wherever a consumer needs to attach a typed, vocabulary-keyed assertion to an element — provenance, presentation hints, migration directives, audit trails, application-specific metadata. Each Extension is opaque to GDX; the consumer that understands the vocabulary acts on it, others ignore it.

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
<INT score : 87
    <_STR ui:color : "green">
    <_STR ui:label : "Performance Score">
>
```

A migration directive attached to a Template:

```
<DEFOBJ CustomerV2
    <STR firstName>
    <STR lastName>
    <STR email>
    <%DEFOBJ #gdx:supersedes : @CustomerV1>
    <_STR migrate:from-v1 : "add-email-from-default">
    <_STR migrate:default-email : "unknown@example.com">
>
```

A confidence Extension on an @-edge:

```
<@OBJ #knows : @bob
    <_FLOAT analytics:confidence : 0.87>
    <_STR analytics:source : "co-authorship-graph">
>
```

In each case, GDX records the Extension as a well-formed element; nothing in GDX-core interprets the vocabulary identifier (`acme:source-system`, `ui:color`, `migrate:from-v1`, `analytics:confidence`). A consumer that understands the vocabulary processes the Extension; one that doesn't ignores it. Multiple vocabularies may coexist on the same element — the same Object, Property, or @-edge may carry `xsd:`, `acme:`, `ui:`, and `analytics:` Extensions side by side, each processed by its respective consumer.

**Extension vs Annotation.** Annotations (Pattern 19) are free-text without a vocabulary identifier; Extensions are vocabulary-keyed and typed. Use Annotation for human commentary that doesn't need machine interpretation; use Extension when a specific consumer needs to recognize the metadata by identifier and act on it programmatically.

**Extension vs Qualifier.** Qualifiers (Pattern 8) are *constitutive* refinements of the parent's value — removing a Qualifier changes what the parent means. Extensions are *adjunct* — removing an Extension does not change the parent's identity, value, or validity. Use Qualifier when the metadata is part of the value's meaning (a unit on a measurement); use Extension when the metadata is an assertion alongside the value (a provenance claim about the measurement).

## 21. Reference with Custom Identifier Scheme

Object identifiers are not constrained to any particular scheme. Use whatever fits your system.

```
<OBJ Alice ...>
<OBJ people/alice ...>
<OBJ 7f3a2b9c-4e21-4f88-91d5-abcdef012345 ...>
<OBJ gdx:Person ...>
<OBJ https://acme.com/people/alice ...>
```

@-edges to these Objects use the `@OBJ` prefix:

```
<@OBJ #knows : @Alice>
<@OBJ #references : @people/alice>
<@OBJ #describes : @7f3a2b9c-4e21-4f88-91d5-abcdef012345>
<@OBJ #cites : @https://acme.com/people/alice>
```

The resolver determines how each reference resolves; the format does not constrain the scheme. Identifiers may not begin with `/` (Core Spec §5.4.1, narrowed in v0.11.0); the leading-slash form is the path-mode reference syntax (see Pattern 21).

## 22. Typed REF — Slot Holding a Handle to an Independent Element

When you want a slot whose value is a typed handle to an independently-existing element — a configuration pointing at a service, a schema-bound instance pointing at its Template, a derived view pointing at an interior position in a source document — use the REF family (Core Spec §7.16). REFs are slot-fillers: they identify the target and stop, with no relationship assertion about the parent and no attribute children. REFs use the `%KIND` prefix, distinguishing them from @-edges (`@OBJ`, `@REL`) which assert relationships and belong to graph-entity targets.

Configuration / wiring:

```
<OBJ ServerConfig
    <STR name : "web-01">
    <%OBJ database : @primary-db>
    <%OBJ fallback : @secondary-db>
    <%DEFOBJ #conformsTo : @ServerConfigV2>
    <%REC sharedDefaults : @global-defaults$/server-defaults>
>
```

`%OBJ` is a REF whose target is an Object; `%DEFOBJ` targets a Template; `%REC` targets a Record at an interior path. The reader and validator know the target's kind from the prefix: `%OBJ` resolves to an Object, `%REC` to a Record, `%BLK` to a Block, `%.STR` to a string Qualifier, `%@OBJ` to an @-edge (reification — see Pattern 25), and so on.

The closed REF family is generated by a single rule: `%` wraps any role-prefix to produce a REF-to-that-role. Qualifiers may refine the slot binding:

```
<%OBJ database : @primary-db
    <.STR role : "read-write">
>
```

The Qualifier annotates the slot binding: "this pointer is the read-write database."

**REF vs @-Edge — choosing the right construct.** Both `%OBJ` and `@OBJ` carry a Resource pointing at an Object, but they mean different things:

- `<%OBJ database : @primary-db>` — "this slot named `database` holds a handle to `primary-db`." A slot-content assertion, with no claim about a relationship.
- `<@OBJ #usesBackend : @primary-db>` — "the parent stands in a `usesBackend` relationship to `primary-db`." A graph statement, asserting the relationship.

Pick `%` when you want a typed handle in a slot; pick `@` when you want GDX to record the relationship as a first-class graph statement.

**Reification — REF to an @-edge.** When an @-edge carries identity (per Core Spec §7.4 axiom E6), it becomes addressable as a persistent substance. A REF prefixed `%@OBJ` or `%@REL` is a handle to such an edge:

```
<OBJ alice
    <@OBJ knows-bob-2024 #knows : @bob>     -- @-edge with identity "knows-bob-2024"
>

<OBJ audit-record
    <%@OBJ source : @alice&knows-bob-2024>  -- REF to the edge by key
    <STR confidence : "verified">
    <STR auditedBy : "SecurityTeam">
>
```

The audit-record references the specific edge by its key; the edge itself remains owned by `alice` per axiom E2′. The REF asserts only "this slot holds a handle to that @OBJ-edge" — no claim about a relationship, no claim of ownership. Reification lets external documents make statements *about* a relationship without disturbing the relationship itself.

The two prefixes `%@OBJ` and `%@REL` are the reification forms: a REF whose target is an Object-edge or a REL-edge respectively. This is the canonical mechanism for making statements about graph relationships — the alternative (replicating an edge to make claims about it) loses the identity that makes the assertion specific.

## 23. Reference to a Non-Identified Element via Path

When the target of a reference is not a globally identifiable Object — when it's a Record inside an Object, a specific item in a list, or a Qualifier hanging off a Property — the identifier-mode reference cannot reach it. Path-mode references address elements by their structural location.

```
<OBJ orders/2026
    <REC shipping
        <STR street : "123 Main St">
        <STR city : "Oakland">
    >
    <REC billing
        <STR street : "PO Box 9">
        <STR city : "Berkeley">
    >
>

<OBJ orders/2026/preferences
    <%REC defaultShippingTo : $/orders/2026/shipping>
>
```

The Resource `$/orders/2026/shipping` is a path-mode reference literal. The `$` introduces path mode; the `/orders/2026/shipping` traverses from the document root into the Object `orders/2026` and then into its `shipping` Record. The reference resolves to that Record element.

Cross-document targets compose path-mode with identifier-mode:

```
<%REC alicesPreferredAddress : @people/alice$/addresses/[1]>
```

The `@people/alice` portion resolves the Object by name (per the resolver's scope walk); the `$/addresses/[1]` portion then traverses from that Object to its `addresses` child and then to that child's first element by document order.

Path segments select by different fields depending on the segment's first character:

- A bareword (`shipping`, `addresses`) matches by `Id`.
- A `#`-prefixed token (`#gdx:knows`) matches by `Rel`.
- A bracketed numeral (`[1]`, `[3]`) matches the *N*-th child in document order, 1-based.

A path used as a reference Resource MUST resolve to exactly one element of the matching role; zero matches, multiple matches, and matches of the wrong role all yield an unresolved reference. The GDX Path Expressions Specification (`GDX-Path.md`) gives the full grammar and resolution rules.

Key-based addressing — for elements that carry a `Key` field — is covered in Pattern 22; it provides location-stable references that survive document restructuring.

## 24. Reference to a Keyed Element via Anchor

When an element carries a `Key` field, it becomes a location-stable anchor — addressable independent of where it sits in the document tree. This is the recommended pattern for references that must survive document restructuring: long-lived archival references, cross-document audit links, and references into Sets of similar elements (log entries, transactions, events).

```
<OBJ system/audit-log
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

<OBJ compliance/incident-7
    <%REC originatingEvent : @system/audit-log&entry-12345>
>
```

The reference `@system/audit-log&entry-12345` is an `IdKeyRef`. The `@system/audit-log` portion resolves the audit-log Object by name; the `&entry-12345` portion then looks up the Keyed element within that Object's scope. The reference resolves to the Record with that Key.

If the audit log is later restructured — entries archived into nested categories, reorganized by severity, etc. — the reference continues to resolve as long as the Key is preserved. A path-based equivalent like `$/system/audit-log/entries/[1]` would break under any such change.

When the referencing element shares a document with the Keyed target, the `@Identifier` portion may be omitted; bare `&entry-12345` is a `KeyRef` that looks up within the document scope.

```
<OBJ compliance/internal-incident
    <%REC originating : &entry-12345>
>
```

Key uniqueness is scoped to the enclosing Object (or to the document for transport-layer elements); see Core Specification §5.4.5. Two different audit-log Objects MAY each have their own `&entry-12345` without collision, because Key resolution is scoped to one Object at a time.

When the anchor is not an Object — for example, a Keyed annotation on a REL declaration or a Keyed slot-documentation inside a Template — use an explicit anchor role marker. The marker is one of the closed prefix-vocabulary keywords (`OBJ`, `REL`, `DEFOBJ`, `PKG`, `GRP`, etc.) followed by `:`:

```
<REL gdx:knows
    <STR &description : "Asserts that the subject knows the object.">
>

<OBJ vocabulary/docs
    <%STR knowsDoc : @REL:gdx:knows&description>
>
```

The `@REL:gdx:knows` anchor explicitly resolves as a REL rather than an Object. The default anchor role in composed references is `OBJ`, so the marker is only needed when the target lives inside something else.

XML practitioners may recognize this pattern as the equivalent of `xml:id`. Convention: when converting GDX ↔ XML, GDX's `Key` corresponds to XML's `xml:id` attribute.

## 25. Vocabulary Metadata Link

REL declarations may reference other RELs to express vocabulary relationships — inverse, sameAs, subPropertyOf, and similar predicates that describe how predicates relate to each other.

```
<REL gdx:manages
    <@REL #gdx:inverse : @gdx:reportsTo>
>

<REL gdx:reportsTo>
```

The `@REL` child of `gdx:manages` declares that its inverse predicate is `gdx:reportsTo`. A consumer that understands the `gdx:inverse` vocabulary can traverse a "manages" edge in reverse by following this link.

Schema cross-references between predicates and Templates use the same mechanism:

```
<REL gdx:hasAuthor
    <%DEFOBJ #gdx:range : @Person>
>

<DEFOBJ Person
    <STR name>
>
```

The `%DEFOBJ` child of `gdx:hasAuthor` declares that the predicate's range is the `Person` Template — instances reached through `gdx:hasAuthor` edges should conform to the `Person` contract. Note: this is a REF (slot pointing at a Template, not a graph statement) because Templates are vocabulary-layer artifacts, not graph-entities; only `@OBJ` and `@REL` are in the @-Edge family. For domain/range pointing at *concept-Objects* (which are Objects and therefore graph-entities), use `@OBJ` instead: `<@OBJ #gdx:domain : @acme:Person>`.

## 26. Complete Vocabulary Document

A GDX vocabulary document defines the predicates, concepts, and schemas of a domain as a single self-contained document. Predicates are REL declarations; concepts are Objects with conceptual identifiers; schemas are Templates. Because all three are GDX, the entire vocabulary is parseable and queryable with the same tools as the instance data it describes.

```
<OBJ acme:Concept
    <STR rdfs:label : "Concept">
    <STR rdfs:comment : "Top-level concept in the acme: vocabulary.">
>

<OBJ acme:Person
    <STR rdfs:label : "Person">
    <@OBJ #rdfs:subClassOf : @acme:Concept>
>

<OBJ acme:Organization
    <STR rdfs:label : "Organization">
    <@OBJ #rdfs:subClassOf : @acme:Concept>
>

<REL acme:knows
    <@OBJ #acme:domain : @acme:Person>
    <@OBJ #acme:range  : @acme:Person>
    <_BOOL acme:symmetric : true>
    <STR rdfs:comment : "Asserts an acquaintance or social relationship.">
>

<REL acme:employs
    <@OBJ #acme:domain : @acme:Organization>
    <@OBJ #acme:range  : @acme:Person>
    <@REL #acme:inverse : @acme:employedBy>
>

<REL acme:employedBy
    <@OBJ #acme:domain : @acme:Person>
    <@OBJ #acme:range  : @acme:Organization>
>

<DEFOBJ acme:Person
    <STR #acme:givenName given>
    <STR #acme:familyName family>
    <REC* #acme:knows knowsList>
>

<DEFOBJ acme:Organization
    <STR #acme:legalName name>
    <REC* #acme:employs employees>
>
```

This vocabulary document defines:

- **Three concept Objects** (`acme:Concept`, `acme:Person`, `acme:Organization`), each addressable by other vocabularies and instances. The `acme:Concept` Object plays the role a Class or top-of-hierarchy would in OWL; `rdfs:subClassOf` links subordinate concepts to it.
- **Three REL declarations** (`acme:knows`, `acme:employs`, `acme:employedBy`) with domain/range Extensions pointing at the concept Objects, plus an inverse Reference between `employs` and `employedBy`.
- **Two Templates** (`acme:Person`, `acme:Organization`) that bind instance shape to the concepts. The Template slots carry `#rel` references aligning slot semantics to the vocabulary's predicates.

The document is one GDX file. A consumer reading it gets the predicates, the concept taxonomy, the schemas, and the alignment between them in a single parse. An instance document satisfying `acme:Person` can be validated against the Template, traversed through `acme:knows` edges, and aligned to other vocabularies through `#rdfs:subClassOf` and similar cross-references.

This pattern combines the smaller patterns (17 Predicate Declaration, 24 Vocabulary Metadata Link, schema patterns) into a coherent unit. The vocabulary itself is data; nothing in this document requires a separate schema language or external metadata layer.

## 27. Template Version Chain

Templates can reference other Templates to declare version succession, supersession, or compatibility. This makes vocabulary evolution self-describing.

```
<DEFOBJ CustomerV1
    <STR firstName>
    <STR lastName>
>

<DEFOBJ CustomerV2
    <STR firstName>
    <STR lastName>
    <STR email>
    <%DEFOBJ #gdx:supersedes : @CustomerV1>
>
```

The `%DEFOBJ` element inside `CustomerV2` declares that this Template's `supersedes` slot points at `CustomerV1`. A migration tool consuming both documents can follow the chain to understand which Templates are predecessors and which are successors.

The same mechanism works for marking deprecation:

```
<DEFOBJ Customer
    <%DEFOBJ #gdx:supersededBy : @CustomerV2>
    <STR firstName>
    <STR lastName>
>
```

**Note on the conceptual shift.** Template version chains are expressed as REFs (`%DEFOBJ`), not as @-edges. Under the GDX partition, the @-Edge family is restricted to targets that are independent graph-entities — Objects and REL declarations. Templates are vocabulary-layer artifacts; pointing at a Template is slot-filling (a slot whose value is a handle to a Template), not graph-statement-making. Authors who think of version chains as "relationships between Templates" should read the structure as "slots holding handles to predecessor/successor Templates, semantically aligned via `#rel` to the vocabulary's version predicates." The semantic interpretation is up to the consumer (migration tools, schema registries); GDX records the slot bindings, not graph statements about Templates.

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

The three-state distinction applies to all Resource-bearing kinds. For an integer:

```
// Absent
<OBJ Order123 <STR customer : "ACME">>

// Null - quantity is unknown
<OBJ Order123 <STR customer : "ACME"> <INT quantity>>

// Present with value (zero is a value, not empty)
<OBJ Order123 <STR customer : "ACME"> <INT quantity : 0>>
```

For string-typed Properties only, there is a fourth conceptual state — present with empty value:

```
<STR middleName : "">
```

The empty string is a legitimate string value, distinct from null. A consumer reading this form knows that middleName was considered and is known to be empty (the customer has no middle name, or has an empty middle name in their record). This empty state applies only to strings; for integers, floats, bools, bytes, and references, there is no "empty value" concept — `<INT count : 0>` is the integer zero (a value), not "empty integer".

## 29. Transport Wrapper for Multiple Objects

When transporting multiple Objects together with metadata about the transport, wrap them in a Package.

```
<PKG customer-import-2026-04-29
    <STR source : "crm-v3">
    <STR contentType : "customer-batch">
    <INT count : 3>
    
    <OBJ Alice
        <STR name : "Alice">
        <STR email : "alice@example.com">
    >
    <OBJ Bob
        <STR name : "Bob">
        <STR email : "bob@example.com">
    >
    <OBJ Carol
        <STR name : "Carol">
        <STR email : "carol@example.com">
    >
>
```

The Package metadata (source, contentType, count) describes the transport; the contained Objects are extracted and stored individually by the receiver. The Package itself is discarded after unpacking.

## 30. Transport Partitioning with Groups

When a Package conveys multiple distinct collections, partition them with named Groups.

```
<PKG color-libraries-2026-04-29
    <STR source : "color-management-system">
    <STR exportType : "full">
    
    <GRP pantone
        <STR libraryName : "Pantone Solid Coated">
        <STR version : "2023">
        <OBJ Pantone-485-C
            <FLOAT l : 47.2>
            <FLOAT a : 67.8>
            <FLOAT b : 54.1>
        >
        <OBJ Pantone-186-C ...>
    >
    
    <GRP ral
        <STR libraryName : "RAL Classic">
        <STR version : "2022">
        <OBJ RAL-3020 ...>
        <OBJ RAL-9010 ...>
    >
>
```

Each Group carries its own metadata (libraryName, version) describing the partition. Groups are unpacked alongside the Package; the contained Objects retain their independent identity and lifecycle.

## 31. Vocabulary Export with Groups

Vocabularies that contain many Templates and RELs can use Groups to organize related declarations.

```
<PKG gdx-standard-vocab-v1
    <STR vocabName : "gdx-standard">
    <STR version : "1.0.0">
    <REM : "GDX Common Vocabulary export">
    
    <GRP numerics
        <REM : "Width-bounded numeric types">
        <DEFINT gdx:i8 ...>
        <DEFINT gdx:i16 ...>
        <DEFFLOAT gdx:f64 ...>
    >
    
    <GRP predicates
        <REM : "Common relational predicates">
        <REL gdx:knows>
        <REL gdx:manages
            <_STR inverse : "gdx:reportsTo">
        >
    >
>
```

The receiver imports the vocabulary by extracting all contained Templates and RELs; the Group organization is informational and does not affect how the receiver stores or uses the imported items.

## 32. Use as Typed Configuration

GDX at its simplest is typed key-value configuration. Use Properties as direct top-level elements; no composition, no schema, no predicates. Comparable in shape to YAML or TOML configuration but with explicit types and no inference traps.

```
<STR databaseHost : "db.internal">
<INT databasePort : 5432>
<BOOL tlsEnabled : true>
<STR logLevel : "info">
<FLOAT timeoutSeconds : 30.0>
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

Adding `#rel` predicates to Properties enables cross-system semantic alignment without committing to a triple model.

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
<DEFREC vocab:Server
    <STR name>
    <STR address>
    <INT port>
>

<REC web01 --vocab:Server
    <STR name : "web-01">
    <STR address : "10.0.1.42">
    <INT port : 8080>
>
```

Templates declare the contract; instances claim to satisfy it; validators check. The Template can travel in the same document as the instances it describes, or in a separate vocabulary file that consumers resolve via reference.

## 36. Use as Graph-Structured Data

Adding @-edges and IRI-based Object identifiers yields fully graph-structured data with relationships, attributes on relationships, and external resolvability.

```
<OBJ https://infra.acme.com/server/web01
    <STR name : "web-01">
    <@OBJ #vocab:dependsOn : @https://infra.acme.com/db/primary
        <STR since : "2024-01-15">
    >
    <@OBJ #vocab:routesTo : @https://infra.acme.com/cache/edge>
>
```

Relationships are explicit, traversable, and queryable. The Object identifiers are full IRIs — globally resolvable and interoperable with RDF-aware systems. The same document can be parsed as a tree, stored as relational rows, traversed as a graph, or exchanged with any RDF-aware system that resolves the IRIs.

The progression across §27 through §31 demonstrates that the same format scales from typed configuration to graph-structured data without changing parsers, tooling, or syntax. Applications use the level of feature engagement that fits their problem.

