# GDX — Graph Data eXchange Specification

- **Status:** Draft v0.11.0 (working draft, pre-1.0)
- **Date:** 2026-05-20
- **Editors:** 
- **Authors:** Julian Mussi


## Abstract
This document defines the GDX abstract data model, the GDX prefix vocabulary and composition rules, the canonical text form, and the schema and validation mechanism. It is independent of any particular wire format. Companion specifications cover the Compact and Binary Forms.


## Status of This Document

This is the v0.9 working draft.

Changes in v0.9.7 from v0.9.6: §1 (Introduction) restructured into §1.1 What GDX Is, §1.2 Existing Approaches, §1.3 GDX Design Goals, §1.4 Where GDX Fits, §1.5 Scope. Conventions and Terminology promoted back to top-level §2, scoped to term definitions only (Core Terms, Conformance Layers, Vocabulary Terms, Identifier Schemes). The GDX Element and Core Data Model renumbered to §3 and §4. §6 renamed to "Element Kinds." Field Sigils and Separators (previously in §2.7 of Conventions) absorbed into §3.4. No normative semantics change.

Changes in v0.9.8 from v0.9.7: the technical term "Kind" (naming the closed enumeration of Object, Record, Edge, etc.) renamed to "Role" throughout. "Kind" was ambiguous against generic English usage and competed with "value-kind." §6 renamed from "Element Kinds" to "Element Roles"; §6.3 from "Kinds in Detail" to "Roles in Detail." §4.1 renamed from "Field Roles" to "Element Fields" to avoid collision with the new technical term; its column header changed from "Role" to "Purpose." The §3.4 sigil table column header changed from "Role" to "Purpose" for the same reason. Sub-subsection numbering in §4.3 corrected from the stale §3.3.x to §4.3.x. The term "value-kind" (the value-category of a Resource) is preserved unchanged; only the structural-position concept renamed. No normative semantics change.

Changes in v0.9.9 from v0.9.8 sharpen the conceptual / data-model / text-form boundary across §3, §4, §5. Each section now has a distinct focus, signaled by its title: §3 "GDX Concepts" defines what GDX expresses; §4 "The GDX Abstract Data Model" defines what an implementation represents at runtime; §5 "Canonical Text Form" defines how elements are written. Substantial content moved between sections to eliminate the three-way overlap that had accumulated. The Sigils and Separators subsection moved from §3.4 to §5.1.1 (sigils are textual mechanics, not conceptual). The Real-Number Representation subsection from §4.3.2 merged into §5.2.3 Float Literals (suffix mechanics are textual). The two textual forms of null values (bare and empty-value) moved from §4.3.4 to a new §5.1.3, while the three-state Property semantics (Absent/Null/Present-with-value) remain in §4. A new §3.4 "The Implicit Parent-Child Relationship" promotes a conceptual point previously buried in §3.2's closing paragraph. Cross-references throughout the spec updated accordingly. No normative semantics change.

Changes in v0.9.10 from v0.9.9 are corrections: (1) §7.4 Resource Rules erroneously listed DEF prefixes (`DEFSTR`, `DEFINT`, etc.) as Resource-bearing; Templates declare a contract for instances and do not carry an instance value themselves. The DEF prefixes have been removed from the resource-family bullets and added to the list of roles that do not carry Resources. (2) Appendix C.2 was reframed from "Reference Cycle Detection" (a generic implementation-correctness concern) to "Reference Cycle Exploitation," emphasizing the adversarial framing consistent with the rest of Appendix C. (3) Several Quick Start examples in Appendix D (§D.1, §D.2, §D.3, §D.4, §D.6, §D.7, §D.8) were updated to wrap their content in an Object or Template root; the previous examples placed bare Properties, Records, or Sets at the document root, which violates the conformance requirement that a document's root MUST be an Object, Template, REL declaration, or Package (§7). No normative semantics change.

Changes in v0.10.0 from v0.9.10 are substantial:

(1) **Reference operator `@` added.** The `@` prefix produces typed references, joining the existing modifier vocabulary. The `@KIND` family composes universally with role bases and modifier-bearing bases (subject to a `@@` prohibition: references cannot target other references). The composition reads as "a reference to a KIND-element." Closed-vocabulary expansion: the §6 specification describes the composition pattern rather than enumerating every cell; valid `@KIND` prefixes are derived from the pattern. This adds the reflexivity primitive — any addressable element can be referenced by any other element through a typed pointer.

(2) **Set notation moved to suffix.** The Set marker `*` is now a suffix: `STR*`, `REC*`, `*INT` notation is replaced by `INT*`. This is a breaking change. The suffix position reflects Set's structural distinction from the refinement modifiers — `*` is value-kind-redirecting (creates a collection) rather than value-kind-preserving. The suffix also unlocks compositions previously forbidden by the no-modifier-stacking rule: `.STR*` (Set of Qualifiers), `_INT*` (Set of Extensions), `@STR*` (reference to a Set) are all valid.

(3) **§3 trio framing added.** New §3.6 "Embedding, Referencing, and Graph Relationships" articulates the three distinct mechanisms for involving an addressable element from within another: `%X` for inline embedding, `@X` for typed references, EDGE for graph statements. Each has a distinct structural shape and use case.

(4) **§6 restructured around the composition pattern.** The §6 intro introduces the prefix pattern `[@?][refinement-modifier?][base][*?]`. §6.1 (Canonical/Minified Forms) and §6.2 (Base Vocabulary) cover the core role vocabulary. §6.3 (Special Operators) introduces the `*` suffix and `@` operator. §6.4 (Refinement Modifiers) covers `.`, `_`, `%`, `N`. §6.5 (Roles in Detail) is the detailed per-role treatment, renumbered from old §6.3.x with `@KIND` added as §6.5.16. The base vocabulary is grouped by category: value primitives, structural roles, declaration roles, transport roles.

(5) **Reference literal grammar reserves `$` for path expressions.** The reference literal grammar accepts `@Identifier` (normative in this revision) and reserves `$PathExpression` for a future revision. Path-mode resolution is an open issue (§11).

(6) **§6.5.4 EDGE clarified.** A note distinguishes EDGE (structured graph-statement element with its own children) from `@OBJ` (leaf-like typed pointer). Both carry references to Objects but have different structural shapes.

(7) **§7.1 Composition Matrix expanded.** New Reference column and row reflect the leaf-like nature of `@KIND` elements (Qualifiers and Annotations as children only). REL and Template rows admit Reference children for vocabulary metadata.

(8) **§7.2 rules updated.** Rule 12 reframed to describe the refinement-modifier-non-stacking constraint precisely. New rules 13 (Set as suffix), 14 (universal `@` composition), 15 (`@@` prohibition), 16 (References are leaf-like).

(9) **Migration impact.** All `*PREFIX` notation in existing documents must migrate to `PREFIX*`. The `@KIND` family is additive and does not affect existing documents. EDGE is unchanged. Templates and RELs are unchanged.

(10) **Appendix split.** The previously-included Appendices B (Format Comparison), C (Security Considerations), and D (Quick Start) have been extracted into separate companion specifications, joining the existing Grammar, Compact Form, and Cookbook companions. The previously-included Appendix A (Conformance) remains in the core specification as normative content. Cross-references throughout the spec updated to point at the new companion documents. The split reduces the core specification by approximately 700 lines and clarifies the boundary between normative content (this specification, Appendix A, Grammar) and non-normative content (Format Comparison, Security Considerations, Quick Start, Cookbook).

Changes in v0.11.0 from v0.10.0:

(1) **Path-mode and key-mode reference resolution are now normative.** The `$PathExpression` form reserved in v0.10.0 §11.4 is fully specified in a new companion document, the GDX Path Expressions Specification (`GDX-Path.md`). The path grammar supports three segment kinds — `IdSegment` (bareword, matches `Id`), `RelSegment` (`#R`, matches `Rel`), and `OrdinalSegment` (`[N]`, matches by 1-based document order). The `&Key` reference literal provides key-based addressing through the location-stable `Key` anchor. Five reference-literal forms are recognized: `IdRef` (`@Anchor`), `PathRef` (`$PathExpression`), `KeyRef` (`&Key`), `IdPathRef` (`@Anchor$PathExpression`), and `IdKeyRef` (`@Anchor&Key`). The composed forms express cross-document addressing. The `@Anchor` production carries an optional role marker (`@OBJ:id`, `@REL:id`, `@DEFOBJ:id`, etc.) drawn from the closed prefix vocabulary; when omitted, composed references default to an Object anchor. The marker enables composed references to non-Object anchors (RELs, Templates, transport-layer containers) without requiring resolver inference.

(2) **§11.4 closes.** Path-mode reference resolution is no longer an open issue. References to §11.4 in earlier revisions are satisfied by the companion specification.

(3) **`/` removed from `IdStartChar` (§5.4.1).** Breaking change. Identifiers may no longer begin with `/`. The `IdContChar` set is unchanged, so identifiers may continue to contain `/` (e.g., IRIs like `https://acme.com/people/alice` remain valid). The narrowing makes the `$/...` path-mode form unambiguous and the `@Identifier$PathExpression` boundary lexically clean. Path-style local identifiers such as `<OBJ /people/alice>` MUST be migrated; the recommended migrations are documented in `GDX-Path.md` Appendix C.

(4) **§5.4.2 identifier-forms table updated.** The "Path" row is removed; path-style local naming is replaced by path-mode reference resolution. The remaining identifier forms (Local name, Single letter, CURIE, Dotted CURIE local, GUID/opaque, IRI) are unchanged.

(5) **§5.2.6 reference literal grammar updated.** The five-form `ReferenceLiteral` production replaces the placeholder. Address-mode singularity is incorporated by reference to `GDX-Path.md` §7.

(6) **§7.1 composition matrix expands the resolvable target set.** `@KIND` references composed with a `PathRef`, `KeyRef`, `IdPathRef`, or `IdKeyRef` resource may now resolve against Records, Blocks, value Properties, Qualifiers, Extensions, and Embedded elements. Parent-context restrictions on where `@KIND` reference elements may appear are unchanged; only the addressability of the target expands.

(7) **`Key` field reframed from Template-slot identifier to location-stable anchor.** The §3.2 description of `Key` is rewritten: Keys are an opt-in named-anchor mechanism providing stable reference targets for elements that lack global identifiers. The Template-alignment justification is removed. A new §5.4.5 defines Key uniqueness rules (scoped to the enclosing Object, or to the document at transport-layer position). The numeric Key form is retained as a lexical convenience but no longer carries Template-slot semantics.

(8) **§8.2 Template-instance alignment pinned.** A new normative paragraph specifies that alignment is by `Id` matching (case-sensitive ordinal), with `Rel` matching as a cross-vocabulary fallback. The `Key` field plays no role in alignment.

(9) **§5.1.1 sigils table updated.** The `$` row is updated from "reserved, grammar deferred" to "introduces a path-mode reference literal (§5.2.6, `GDX-Path.md`)."

(10) **Migration impact.** Existing documents using leading-slash identifiers MUST be migrated (drop the leading slash, retaining any interior `/`). Existing documents using only `@Identifier` references are unaffected. Existing documents that used `&K` numeric Keys for Template-slot alignment MUST be migrated to named-Id Template slots; the `Key` field continues to be lexically valid but no longer carries slot-alignment semantics. Templates that previously used `&1`, `&2` positional slots are migrated to named slots (`firstName`, `lastName`, etc.). Existing documents that used a non-normative `$`-prefixed literal under v0.10.0 §11.4's implementation-defined window MUST conform to the v0.11.0 grammar.

(11) **§1.3 adds the substrate-with-pluggable-constraints positioning.** A new design-goal paragraph names GDX as a structural schema (the prefix vocabulary, element model, Template structure, composition axioms) with no constraint implementation. Constraint vocabularies layer on top as Constraint Extensions whose identifiers GDX records but never interprets.

(12) **§2.2 decomposes Layer 2 conformance.** The single "Schema Conformance" layer splits into Layer 2a (Template structural conformance, owned by GDX) and Layer 2b (constraint-vocabulary conformance, owned by the constraint vocabulary). The decomposition makes explicit that an instance may be Layer 2a-conforming while Layer 2b-non-conforming for one vocabulary and Layer 2b-conforming for another; verdicts are vocabulary-relative above the primitive floor. Appendix A is updated to mirror the split.

(13) **§7.5 closes with a primitive-axioms statement.** A new closing paragraph names the §7.5 validation requirements as the GDX-native floor (Layer 1) and points to §8 for the schema layer.

(14) **§8 gains a layered-intro paragraph.** A new paragraph at the top of §8 frames the section in three layers — Template structure, Template-instance alignment, and Constraint Extensions — and ties each to the §2.2 conformance layers.

(15) **§8.4 renamed and rewritten.** Previous title "Validation Extensions" becomes "Constraint Extensions" to reflect that GDX hosts, vocabularies validate. New normative MUST: GDX implementations MUST NOT interpret constraint semantics; constraint identifiers are opaque to GDX. A new §8.4.1 "Parallel Constraint Vocabularies" formalizes the multi-vocabulary coexistence property and gives a worked example showing XSD, JSON Schema, and a vendor vocabulary on the same Template. §8.4.2 covers validator behavior on recognized vs unrecognized identifiers.

(16) **§8.6 updates Common Vocabulary framing.** The Common Vocabulary is positioned as one vocabulary among many — it ships with GDX as a convenience and reference but has no privileged status at the GDX layer. `gdx:` constraint identifiers are opaque to GDX in the same way as `xsd:`, `json:`, or any other.

(17) **§8.2 expands into subsections covering the universal claim rule and materialization.** §8.2.1 (Slot Alignment) carries forward the Id-and-Rel alignment rule. §8.2.2 (Cross-Role and Transport Restrictions) carries forward the prefix-matching rules. §8.2.3 (The Universal Claim Rule) is new: a four-case matrix governs how Template-declared and instance-declared values reconcile on any claim-bearing field (Rel, Key, default Resource, nested `--type`). Differing claims are conflicts; matching claims are confirmed; silent-on-one-side defers to the other. §8.2.4 (Materialization) is new: defines the build-time operation of writing Template-supplied values into the instance as explicit fields, motivated by the searchability and self-description requirements of persisted documents. Materialization is RECOMMENDED for persistence and OPTIONAL for in-memory processing.

(18) **§8.2 distinguishes named type identity from observable conformance.** Two new paragraphs in §8.2: (a) `--type` is an identity claim anchored to the Template name; two Templates with identical shapes are different types, and an instance of one is not silently an instance of the other; (b) cross-type conformance — an instance of type A structurally satisfying Template B — is a separately observable relation, useful for validation against multiple schemas, version-compatibility checks, and migration tooling. Conformance does not redefine identity. Consumers MUST NOT silently substitute one type for another even when their instances mutually conform; any cross-conformance is surfaced as a derived observation, not a rewrite of the original `--type` claim.

(19) **§1.1 reframes GDX as an abstract data model with concrete representations.** A new opening paragraph establishes that GDX is the data model — the abstract content — and that Standard Form, Compact Form, Binary Form, and programmatic in-memory form are alternative concrete representations of that model. This clarifies the boundary "what is GDX?" and positions the various forms as faithful realizations rather than separate things.

(20) **§4 (Abstract Data Model) intro reinforces the model/representation distinction.** New paragraph ties the data model defined in §4 to its concrete realizations: each preserves the same elements, prefixes, field semantics, and composition relationships; conversions transform surface syntax without changing content.

(21) **§2.1 adds Core / Extension definitional pair.** New terminology entries formally distinguish the **core** (what GDX itself defines and interprets — prefix vocabulary, element model, grammars, composition, Template-instance alignment, claim rule, primitive value-kind axioms) from **extension** (vocabulary-defined material that rides on top of the core — Rel identifiers, Type identifiers, Constraint Extension content). The pair names the **interpretation** boundary between what GDX understands and what GDX hosts. The terms *core layer* and *extension layer* are introduced as a convenient shorthand for the two sides of the boundary.

(22) **§3.7 articulates the optional-elaboration principle.** New subsection: elements are valid at their minimum form (Prefix alone); elaboration is opt-in and proceeds along two structurally distinct patterns. Identity elaboration is orthogonal across three independent axes (`Id`, `Key`, `Rel`). Value elaboration is layered across five layers (category, value, structural identity, constraint attachment, constraint semantics), where each layer is optional given the layer above and the layers live in different places (instance, Template, vocabulary). The principle organizes how to read the rest of §3 and explains why simple GDX stays simple while richer cases grow without restructuring.

(23) **§1.3 adds integration-surface, not-LCD framing.** A clause in the substrate-paragraph clarifies that GDX is the integration surface where multiple vocabularies coexist, not a lowest-common-denominator subset of them. The structural schema is opinionated and complete; what's neutral is the constraint layer.

(24) **Targeted wording sweep: core/extension terminology adopted across §2.2 and §8.4.** Replaced phrases like "the GDX layer" and "hosted on top" with "core layer" and "extension layer" where the new terminology disambiguates the interpretation boundary from the mechanism boundary. Other places — where wording was already clear — were left as-is to avoid churn.

(25) **§1 restructured.** §1.1 (What GDX Is) trimmed for redundancy: removed the duplicate prose paragraph (with `strutural`/`resouce` typos), pulled the "single repeating element shape" paragraph forward from §1.3's tail, and framed the small example explicitly as Standard Form (one realization of the abstract model). §1.2 and §1.3 swapped positions: design goals now lead, format comparison follows. The rhetorical flow shifts from "survey landscape, explain choices" to "introduce model, state goals, validate against alternatives."

(26) **§1.2 (Design Goals) rewritten with nine model-focused goals.** Replaced the previous mixed list (which combined model goals with serialization-format virtues) with nine goals that each articulate a capability GDX commits to and a corresponding gap in established formats or data models. The new goals: Typed elements baked into the form; Inline semantic alignment; Schema is data; Structural schema with pluggable constraint vocabularies; Unified document and graph model; Self-describing instances without the schema; Reflexivity through typed references; Layered validity with clear ownership; Minimum-is-enough elaboration. Serialization-format virtues (round-trip stability, encoding efficiency, canonical content addressing) are explicitly relegated to §5 as properties of Standard Form rather than goals of the data model.

(27) **§1.3 (Comparison with Existing Formats) absorbed the side-by-side example.** The composite-record side-by-side comparison block (showing the same record in GDX Standard, GDX Compact, JSON, YAML, and XML) moved from the tail of the old §1.3 to the tail of the new §1.3 (the comparison section). It now functions as a synthesizing example after each format's individual discussion rather than as an interruption of the design-goals list. The new §1.3 also gets typo cleanup (`unweildly`, `loosing`, `Write and Read`) and a sharpened opening that frames the section as validation of the §1.2 claims against the landscape.

(28) **Three cross-references updated for the §1.2/§1.3 swap.** §3.1 ("responds to the problems §1.2 named" → "§1.3 names"); §8 layered-model paragraph ("(§1.3)" → "(§1.2)"); the new §1.3 closing references "(§1.2)" for the layering/pluralism design goals. Historical changelog entries that reference §1.3 at the time of their item are left as-is; they correctly describe the change at the point it was made.


## Table of Contents

1. [Introduction](#1-introduction)
2. [Conventions and Terminology](#2-conventions-and-terminology)
3. [GDX Concepts](#3-gdx-concepts)
4. [The GDX Abstract Data Model](#4-the-gdx-abstract-data-model)
5. [Canonical Text Form](#5-canonical-text-form)
6. [Element Roles](#6-element-roles)
7. [Composition Axioms](#7-composition-axioms)
8. [Schemas and Validation (Templates)](#8-schemas-and-validation-templates)
9. [Transport Semantics](#9-transport-semantics)
10. [Data Persistence](#10-data-persistence)
11. [Open Issues](#11-open-issues)
- [Appendix A — Conformance](#appendix-a--conformance)

---

## 1. Introduction

### 1.1 What GDX Is

GDX (Graph Data eXchange) is a typed graph data model with a closed structural schema and a family of concrete representations. The data model is the abstract content: elements with prefixes declaring their role and value-kind, identity fields, typed resource values, schema-type claims, and composition into trees with graph references. The concrete representations are how that abstract content is realized for human reading, machine exchange, and in-memory processing — Standard Form (§5 of this specification), Compact Form (companion specification), Binary Form (planned companion), and the programmatic in-memory form a runtime exposes. A GDX document, regardless of which representation it is serialized in, conveys the same underlying data; tools convert between representations without changing the model content.

The model is built on a single repeating element shape: a prefix declaring the element's role, optional identity fields, an optional typed value, and optional children that compose into trees and graph relationships. The same element shape expresses Properties, Records, Objects, Edges, Sets, Templates, References — every role GDX defines. This shape is the load-bearing design choice from which the rest of the specification follows; §3 develops the element as a concept and §4 defines it formally.

The following small example, written in Standard Form, illustrates how the data model surfaces in text:

```
<OBJ Alice
    <STR firstName : "Alice">
    <INT age : 24>
    <BOOL active : true>
    <EDGE #knows : @Bob>
>
```

This declares an Object identified as `Alice` with three typed properties (string, integer, boolean) and an Edge expressing a `knows` relationship to another Object identified as `Bob`. Every element states its role, its value-kind, and its content explicitly. No external schema is required to know that `age` is an integer, that `active` is a boolean, or that the Edge points to another entity — the data model itself carries that information on each element.

GDX is designed to be easily read, modified, and created by humans. The same data is just as easily produced by tools, parsed by machines, and traversed by graph queries. It carries structured application data, semantic relationships, and stable identity within a single uniform model — properties that established formats and data models typically split across several incompatible designs.

### 1.2 GDX Design Goals

GDX's design choices target capabilities that established formats and data models leave on the table. The goals below name what the data model itself commits to. Properties of any particular concrete representation (round-trip stability of the text form, encoding efficiency, canonical form for content addressing) are properties of Standard Form and are defined in §5 and its companion specifications.

**Typed elements baked into the form.** Every element declares its role and value-kind as part of its identity, not through external annotation or value-shape inference. `<INT age : 24>` is unambiguously an integer-valued property; `<STR version : "1.10">` is unambiguously a string. JSON's number-but-no-int-vs-float, YAML's permissive type inference (`yes`/`no`/`NO` parsing surprises), and XML's everything-is-text-with-attributes all leave types implicit. GDX makes the type part of the element's primary identity — every element in the data model carries an explicit value-kind, and readers and tools never have to infer it from the value's appearance.

**Inline semantic alignment.** The optional `Rel` field on any element names a shared predicate that aligns the element's local identity to a cross-document vocabulary. `<STR #schema:givenName firstName : "Alice">` is locally named `firstName` but semantically identifies as `schema:givenName`. Two systems using different local names can recognize the same concept by matching the predicate. The alignment travels with the data, not in a separate mapping layer. This is RDF's predicate-alignment property without RDF's triple-model overhead; JSON and YAML have no semantic layer at all.

**Schema is data.** A Template is a GDX element with a `DEF…` prefix. It uses the same syntax as the instances it describes; the same parser reads both; the same query mechanisms address both. There is no separate schema language, no schema compilation, no schema/instance impedance. A Template can travel in the same document as the instances that satisfy it — common in vocabulary documents, integration tests, and self-contained exchange formats. XSD is a separate language from XML; JSON Schema is a separate convention layered over JSON; GDX collapses the distinction.

**Structural schema with pluggable constraint vocabularies.** GDX defines a complete structural schema — the prefix vocabulary, the element model with its identity and value fields, Template-based slot declarations, the universal claim rule, and the composition axioms. This structural schema is what GDX *is*, and it is fixed by this specification. What GDX deliberately does *not* define is constraint implementation: numeric bounds, string patterns, regex matches, semantic types, and similar value-level constraints have no normative semantics in the core. Constraint vocabularies (XSD, JSON Schema, OWL profiles, bespoke domain vocabularies) ride on top as Extension elements whose identifiers GDX records but never interprets. This lets GDX serve as an integration surface where multiple constraint vocabularies coexist — XSD and JSON Schema on the same Template, each interpreted by its own consumer. GDX is not a lowest-common-denominator subset of these vocabularies; its structural schema is opinionated and complete, while the constraint layer is deliberately neutral. No other widely-used format hosts multiple constraint vocabularies neutrally.

**Unified document and graph model.** A GDX document is a tree, but tree elements may carry Edges and typed references to other elements anywhere in the document or beyond it. Object identifiers can be local names, hierarchical paths, GUIDs, or full IRIs. The same data is readable as a document, traversable as a graph, and storable as relational rows (§10) — each access pattern uses the same underlying data with no separate format. Document databases and graph databases currently force a choice between which view the data lives in; GDX makes both natural and lets the consumer pick.

**Self-describing instances without the schema.** When a Template declares metadata for a slot (the slot's `Rel`, a default value, a Key anchor), an instance may omit the metadata and rely on the Template to supply it. Before persistence, materialization (§8.2.4) writes those Template-supplied values into the instance as explicit fields. A materialized instance carries its full identity without needing the Template at read time — searchable by `Rel`, queryable by predicate, parseable in isolation. This solves a problem schema-validated formats have traditionally had: data written against a schema becomes opaque to queries when the schema isn't loaded.

**Reflexivity through typed references.** Any addressable element can be the target of a typed reference (`@KIND`) from any other element. References compose with path expressions (`$path`) and Key anchors (`&key`) to address elements that lack global identifiers. This is the model-level expression of "everything is data": vocabulary documents reference Templates and predicates, configuration documents reference Objects, instances reference each other. The reference is itself a GDX element, validated like any other. JSON's `$ref` convention and XML's `IDREF` are bolt-ons; GDX's references are first-class typed elements.

**Layered validity with clear ownership.** Validity is layered, not monolithic. Three rungs with three different owners (§2.2): GDX-native primitive axioms (an `INT` is an integer — owned by the core), Template structural conformance (instance children align to declared slots — owned by GDX's alignment algorithm), and constraint-vocabulary conformance (instance values satisfy vocabulary-defined constraints — owned by the vocabulary). An element can be GDX-valid, Template-conforming, and constraint-invalid for one vocabulary while constraint-valid for another; the verdicts are independent and the boundaries are visible. XSD, JSON Schema, and OWL each conflate these into a single valid-or-not answer; GDX separates them by ownership.

**Minimum-is-enough elaboration.** Every element is valid with just its prefix; identity, value, type, and constraint affordances are all opt-in (§3.7). Identity grows along three orthogonal axes (`Id`, `Key`, `Rel`); value elaboration is layered across instance, Template, and external vocabulary. Simple cases stay simple — a property with just a name and a value is one line — and richer cases grow without restructuring. Schema-first typed formats require ceremony even for trivial values; GDX lets the simplicity match the situation.

### 1.3 Comparison with Existing Formats

The design goals above name capabilities GDX commits to. This section sets those claims against the data format landscape — JSON, YAML, XML, RDF/OWL, and binary protocols — to show where each existing format does what GDX does and where it doesn't.

**JSON** is simple, ubiquitous, and supported by every modern programming environment. Its weaknesses follow from its simplicity: numbers have no integer-vs-float distinction, binary data has no native form, semantic alignment requires external convention, and validation requires a separate schema language. JSON is the right answer when universality matters more than richness.

**YAML** is friendly for human authoring and widely used for configuration. Its type inference is permissive — `yes`, `no`, `on`, `off` may parse as booleans depending on YAML version, version numbers like `1.10` may parse as floats, country codes like `NO` may parse as the boolean false. Round-trip stability requires careful quoting discipline. YAML documents also become unwieldy as they grow more complex, losing the readability benefits they were designed for.

**XML** provides hierarchical structure, namespaces, and a mature schema language (XSD). Its weaknesses are verbosity and the attribute-versus-element ambiguity (some data goes in attributes, some in elements, with no principled rule). As a mechanism for markup XML is excellent but it is less efficient for pure information exchange.

**RDF/OWL** models data as a graph of subject-predicate-object triples and supports semantic interoperability through shared vocabularies. RDF triples function as the basis for ontology building. Its strengths come at the cost of high complexity (multiple serializations, blank nodes, named graphs, OWL inference) and ergonomics that many application developers find off-putting.

**Binary protocols** (Protobuf, FlatBuffers, Cap'n Proto, Avro) optimize wire format size and parse speed at the cost of human readability and schema portability. Each requires schema definition in its own language and ahead-of-time compilation. These are the right answer for high-frequency machine-to-machine traffic.

Each format's development can be traced to a primary motivation: XML for markup, YAML for human writing and reading, RDF for ontology, JSON for object serialization, binary protocols for wire efficiency. Each does its job well within its domain. The gap GDX fills is for applications needing *several* of these properties together: explicit types like a binary protocol, semantic alignment like RDF, schema validation like XML, document readability like JSON, and the layering / pluralism properties (§1.2) that none of them provide.

A short side-by-side example illustrates the contrast. The same composite record across formats:

**GDX (Standard Form):**

```
<REC customer
    <STR firstName : "Alice">
    <INT age : 24>
    <BOOL active : true>
>
```

**GDX (Compact Form):**

```
REC customer
  STR firstName : "Alice"
  INT age : 24
  BOOL active : true
```

**JSON:**

```json
{
  "customer": {
    "firstName": "Alice",
    "age": 24,
    "active": true
  }
}
```

**YAML:**

```yaml
customer:
  firstName: Alice
  age: 24
  active: true
```

**XML:**

```xml
<customer>
  <firstName>Alice</firstName>
  <age>24</age>
  <active>true</active>
</customer>
```

The shapes are similar; the information each carries is not. JSON, YAML, and XML each leave several things implicit: that `age` is an integer (not a string `"24"`), that `active` is a boolean (not the string `"true"`), and (in a richer example) that `firstName` aligns with the `schema:givenName` predicate. The receiver must already know — through external schema, convention, or out-of-band documentation — what types and meanings to assign. The GDX form makes each of these explicit on the element itself.

A more extensive comparison covering predicate alignment, schema validation, and graph-structured data is in the Format Comparison companion specification.

### 1.4 Where GDX Fits

GDX targets a range of problems where existing formats produce friction. Within this band, GDX combines properties no single alternative provides; outside it, an established format is usually the better choice.

GDX is well-suited to:

**Multi-tenant operational systems where tenants share semantics but not naming.** When two tenants both have a "customer" concept but call the field different things, GDX's `#rel` predicate puts the alignment directly on the data — local names stay tenant-specific while shared meaning travels with the field.

**Configuration and infrastructure systems with strong typing requirements.** YAML and JSON give readable configuration but no type safety; switching to a typed language gives safety but loses the data-as-data property. GDX provides both — readable like YAML, typed like a programming language, with Templates as the schema layer.

**Storage systems that need both document and graph semantics.** GDX maps cleanly to relational rows for document-style storage (§10.2) and supports graph traversal through Edges. This is a niche that document databases and graph databases currently force a choice between.

**Internal service interchange where contracts must be readable and validated.** Service-to-service messages where developers want JSON-grade readability for debugging, schema-grade validation for contracts, and predicate-style semantics for evolution.

GDX may **not** the right tool for:

- Public web APIs, where JSON's universality matters more than any feature GDX adds.
- Document publishing with mixed text and markup, where XML's mixed-content model is the right shape.
- High-frequency binary wire formats where every byte matters, where Protobuf, FlatBuffers, or Cap'n Proto will outperform GDX's binary form.

The common shape of GDX's target domain: data that is graph-shaped but predominantly hierarchical, where semantic alignment matters but full ontology reasoning does not, and where producer and consumer are within an organizational boundary loose enough that "everyone agrees on the schema" is not trivially true.

### 1.5 Scope of This Document

This specification defines the GDX core: the data model, the prefix vocabulary, composition rules, the canonical text form, the identifier and literal grammars, and the Template-based schema mechanism. It does not define standardized predicate vocabularies, semantic types, validation extension catalogs, or the binary wire format. These are the subject of companion specifications and the *GDX Common Vocabulary*.

The core specification is intentionally minimal. Concerns that are likely to evolve over time — predicate semantics, domain-specific types, validation extensions, mapping conventions — live outside the core in versioned vocabularies that may evolve without revising the structural specification.

This document specifies:

- The conceptual element form (§3).
- The abstract structure of a GDX element (§4).
- The canonical text form (§5).
- The closed prefix vocabulary, structural roles, and modifiers (§6).
- The composition rules that govern legal nesting (§7).
- The Template-based schema and validation mechanism (§8).
- Transport-layer semantics for exchange between systems (§9).
- Persistence and interchange considerations (§10).
- Conformance requirements (Appendix A).
- Comparison with other data formats (the Format Comparison companion specification).


---

## 2. Conventions and Terminology

This section defines the terms used throughout the specification. The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are interpreted in the sense established by RFC 2119.

### 2.1 Core Terms

- **element** — a single GDX node. The unit of structure; every element has a prefix and may have identity fields, a resource, and children. The term is generic and applies to any role.

- **Prefix** — the leading token that declares an element's role. Belongs to the closed vocabulary defined in §6.

- **Role** — the structural position an element occupies in the data model: Object, Record, Block, Edge, Property, Set, Node, Qualifier, Extension, Template, Relation Declaration, Annotation, Package, or Group. Obtained from a prefix by stripping any modifier. See §6.

- **Modifier** — a sigil or letter prefix-fragment that adapts a base role without changing its composition rules: `*`(Set), `%`(Embedded), `.`(Qualifier), `_`(Extension), `N`(Node), `DEF`(Template).

- **Resource** — the typed scalar value carried by an element, when present. Resource value-kinds are `string`, `int`, `float`, `bool`, `bytes`, and `reference` (§4.3).

- **Value-Kind** — the type category of a Resource. Distinct from Role (which describes structural position). A Property of prefix `STR` carries a Resource of value-kind `string`.

- **Identifier** — any of the three identity fields (`Id`, `Key`, `Rel`) that name or link an element. Some roles require at least one (§4.2).

- **Document** — a single rooted GDX tree, or a sequence of root-eligible elements treated as a unit of serialization. The root(s) of a document MUST be Objects, Templates, REL declarations, or Packages (§9).

- **Data layer** — the set of GDX roles representing persistent, addressable, schema-validatable content: Objects, Records, Blocks, Properties, Sets, Nodes, Edges, Qualifiers, Extensions, Templates, Relation Declarations, and Annotations.

- **Transport layer** — the set of GDX roles representing transient organizational and exchange constructs: Package and Group (§9). Transport-layer constructs are unpacked by receivers and are not persisted as such.

- **Core** — the structural and semantic content normatively defined by this specification: the prefix vocabulary, the element model, the identifier and literal grammars, the composition axioms, the Template structure and instance-alignment mechanism, the universal claim rule, and the primitive value-kind axioms. The core is what GDX *is*; it is fixed by this specification and does not change when extensions are added. (Distinguish "the core" — the concept — from "the Core Specification" — the document name. The latter is this document.)

- **Extension** — vocabulary-defined material that rides on top of the core through GDX's extension-point mechanisms. Extensions include: predicate identifiers in the `Rel` field, Template names in the `Type` field, and Constraint Extension elements (§8.4) carrying constraint identifiers from external vocabularies. The core records extensions structurally and validates their well-formedness; their *semantic content* is interpreted externally by consumers that understand the vocabulary in question. The core/extension distinction is the **interpretation** boundary between what GDX knows and what GDX merely carries.

- **Core layer / Extension layer** — when a discussion needs to distinguish what GDX understands from what GDX hosts, the terms *core layer* and *extension layer* refer to those two sides of the interpretation boundary. The core layer is owned by this specification; the extension layer is owned by external vocabularies and their consumers.

### 2.2 Conformance Layers

GDX defines a layered conformance model. Each layer is independent, evaluated by a different agent, and reports its own kind of error.

- **Representation Conformance (Layer 1)** — assertions about how a value is encoded. Includes prefix-to-value-kind matching, literal grammar conformance, representation tags (§5.2), and the round-trip text equivalence requirement (Appendix A). Owned by GDX. Evaluable at parse time without external context. Errors at this layer are *malformed document* errors and are absolute.

- **Schema Conformance (Layer 2)** — assertions about how data aligns to declared shapes and constraints. Decomposes into two sub-layers with different ownership:

  - **Layer 2a — Template structural conformance.** Whether an instance's children align to a referenced Template's declared slots (by `Id`, with `Rel` as cross-vocabulary bridge; see §8.2). Owned by GDX: the validation algorithm is part of this specification. Evaluable by any consumer that can resolve Templates. Errors at this layer are *schema violation* errors.

  - **Layer 2b — Constraint-vocabulary conformance.** Whether the values within a structurally-conforming instance satisfy the constraints expressed by a particular constraint vocabulary (e.g., `xsd:`, `json:`, a domain-specific vocabulary). Owned by the constraint vocabulary, not by GDX. Evaluable only by a consumer that understands the specific vocabulary's role identifiers. GDX records Constraint Extension elements (§8.4) but renders no verdict on them.

The layers are independent. A document may be Layer 1-conforming but Layer 2a-non-conforming (well-formed but does not satisfy its Template's structure). A document may be Layer 2a-conforming but Layer 2b-non-conforming for one vocabulary and Layer 2b-conforming for another (same data; one vocabulary's constraints pass, another's fail). All four verdicts MUST be reported and handled distinctly by conforming implementations.

The decomposition reflects the core/extension boundary (§2.1). Layer 1 and Layer 2a are the **core layer** — what GDX itself defines, owns, and evaluates. Layer 2b is the **extension layer** — what external constraint vocabularies define and what their consumers evaluate. GDX records extension-carried content but does not interpret it. This boundary is what allows multiple constraint vocabularies to coexist on the same data without GDX taking a side; see §8.4.

### 2.3 Vocabulary Terms

- **Vocabulary** — a versioned collection of standardized identifiers, Templates, and Extensions defining shared semantics for a domain. Vocabularies are layered on top of the core specification through the existing extension and template mechanisms.

- **Common Vocabulary** — the GDX Common Vocabulary, a companion specification shipping with the core. Defines standardized representation-tag suffixes, common Templates (numeric width, hash types, semantic types), and common predicate refinement extensions.

- **Representation Tag** — a literal-level annotation declaring how a value is encoded. Drawn from a closed vocabulary defined in §5.2 and referenced by suffix (e.g., `i8`, `f64`, `d`) or prefix (e.g., `0x`, `0b`).

### 2.4 Identifier Schemes

- **Local Identifier** — an identifier whose meaning is scoped to the document containing it. Property and Record names are local identifiers.

- **Resource Identifier** — an identifier whose meaning extends beyond the document. Object identifiers are resource identifiers. The specification does not constrain the identifier scheme; CURIEs, paths, GUIDs, and other forms are all valid.

- **CURIE** — a Compact URI, of the form `prefix:local`, where `prefix` is a vocabulary alias and `local` is the local identifier within that vocabulary. Example: `gdx:Person`.

The grammar for identifiers is uniform across all identifier-bearing positions (§5.4). The semantic interpretation differs by context but the lexical form does not.

---

## 3. GDX Concepts

This section defines the concepts that the rest of the specification builds on: what an element is, what its fields express, how elements relate to each other, and how the same form expresses both hierarchical structure and graph relationships.

The focus is conceptual. Programming-model concerns — how an implementation represents elements at runtime, what an implementation MUST enforce — are in §4. Text-form concerns — sigils, whitespace, literal grammars — are in §5.

### 3.1 The Self-Describing Element

A GDX element states its own type, its own role, and its own content. No external schema is required to determine that a value is an integer rather than a string, that a reference points to another entity, or that a field aligns with a shared predicate. The element carries this information on itself.

This is the design choice that responds to the problems §1.3 names. Type ambiguity dissolves when the element declares its value-kind through its prefix. Semantic drift dissolves when the element carries its own predicate alignment. Schema separation dissolves when the schema is itself written in the same form as the data. Document-vs-graph tension dissolves when the same element form expresses both hierarchical containment (parent-child) and graph relationships (Edges). Round-trip instability dissolves when the form is canonical enough that parsing and re-emitting produce identical text.

None of these dissolutions requires an external interpretation layer. The element is enough on its own.

### 3.2 Anatomy of an Element

An element has six defined positions: Prefix, Key, Rel, Id, Resource, Type. A richly-fielded example showing all six:

```
<STR &given-name #gdx:givenName fname : "Alice" --gdx:string>
```

This example is contrived to show all six fields together; in practice an element typically carries a subset. Each field plays a distinct conceptual role:

**Prefix** declares the element's role. It is the only required field. The Prefix tells consumers what kind of structural entity this element is — a Property, Record, Edge, Set, Object, and so on — and from the Prefix alone, a consumer knows the element's structural position, the value-kind of any Resource it carries, and what compositions are allowed. Prefixes are drawn from a closed vocabulary defined in §6.

**Key** is an optional named anchor on an element. When present, it identifies the element within its enclosing Object's scope (or within the document for elements at transport-layer position), independent of the element's path within that scope. Keys provide location-stable reference targets for elements that lack global identifiers — particularly non-Object elements whose tree position may change. References to a Keyed element use the `&Key` reference literal (§5.2.6) or the composed `@Identifier&Key` form for cross-Object addressing. Keys MAY be numeric or named identifiers; Key uniqueness rules are stated in §5.4.5. Keys play no role in Template-instance alignment (§8.2).

**Rel** names the predicate that aligns this element to a shared vocabulary. The Rel field is GDX's mechanism for cross-document semantic alignment. Two documents using the same Rel are asserting the same conceptual attribute, even when their local identifiers differ. The alignment travels with the data, not in a separate mapping layer.

**Id** is the element's name in the document. For Properties and Records, the Id is a local identifier — a field name meaningful within the document. For Objects, the Id is a resource identifier — a path, CURIE, GUID, or IRI that may be referenced from elsewhere. Whether an Id is required depends on the element's role and position; see §4.2 for the runtime rules.

**Resource** is the typed value the element carries. The Resource follows the typed value-kind appropriate to the Prefix family: a `STR` element's Resource is a string; an `INT` element's Resource is an integer; an `EDGE` element's Resource is a reference to another Object. The Resource is optional — an element may declare a slot or a null value with no Resource at all (§4.3.3).

**Type** is a reference to a Template the element claims to satisfy. The Type field is the element's schema-level claim about its content. When present, it triggers schema validation against the named Template. When absent, the element is interpreted purely at the structural level. Templates are defined in §8.

Together these six fields are sufficient to express identity, alignment, value, and schema-claim — every conceptual concern at the element level. Higher-order concerns (containment, schema definition, transport) are expressed through composition of multiple elements rather than additional fields.

### 3.3 The Identity / Value Split

The `:` separator divides every element into two sides. The left side carries identity — what this element is in the document. The right side carries content — what it actually says.

```
<Prefix Key Rel Id : Resource Type>
 ──────identity───── : ───value────
```

This split is a deliberate commitment about how typed structured data is shaped. Formats that don't make this split tend to tangle the two: XML's choice between attributes and child elements is a recurring source of confusion because both can serve identity or content roles depending on convention. JSON's bare key-value pairs put identity (the key) and content (the value) on the same syntactic level with no way to distinguish "this element claims to be of type X" from "this element has a value of X." RDF expresses identity (subject/predicate) and content (object) as separate triples that must be assembled to recover an element's full meaning.

GDX makes the split explicit. The identity portion (Prefix, Key, Rel, Id) tells a reader *what role the element plays*; the value portion (Resource, Type) tells them *what content the element carries and what schema it claims*. A consumer can stop after reading identity and know everything about the element's place in the document; they read further to learn what the element actually says. The textual mechanism that marks this split — the `:` separator — is defined in §5.

### 3.4 The Implicit Parent-Child Relationship

When one element appears as a child of another, the relationship between them is implicit — GDX does not label parent-child edges. The role of the child element determines what kind of attribute it represents to the parent. A `STR` child of a `REC` parent is a string-valued property of that record; an `EDGE` child of an `OBJ` parent is a directed relationship from that object; a `REC` child of an `OBJ` parent is a structured composite property of that object.

This is a deliberate design choice. Naming the parent-child relation on every element would add a field that the prefix already determines. By making the relationship implicit, GDX avoids redundant labeling while preserving the information: a consumer always knows, from the child's prefix and the parent's role, what the containment relationship means.

Which roles may contain which roles is not a conceptual question but a composition question. The composition rules are defined in §7.

### 3.5 Elements as Graph Statements

The same six-field form supports GDX's graph dimension. The subject of a statement is the parent element; the object is either a child Property's Resource (a literal value) or a child Edge's Resource (a reference to another Object); the predicate is the child's Rel field when present. This produces two forms of graph statement:

- **Literal-valued statement.** `<OBJ Alice>` containing `<STR #gdx:givenName fname : "Alice">` expresses *Alice — gdx:givenName — "Alice"*.
- **Edge-valued statement.** `<OBJ Alice>` containing `<EDGE #knows : @Bob>` expresses *Alice — knows — Bob*.

The document remains a tree while expressing graph relationships. A reader can navigate it as a tree, traverse it as a graph, or store it as relational rows (§10.2) — all without changing the underlying form. Comparison with RDF and labeled property graphs is in the Format Comparison companion specification.

### 3.6 Embedding, Referencing, and Graph Relationships

GDX provides three distinct mechanisms for involving an addressable element from within another element's content. The three answer different questions and have different structural shapes:

**Embedding** (the `%` modifier on a Property prefix) puts the target's content directly into the parent's structure. The embedded element appears in property-value position, carrying its full content inline. The host element holds the target's content. Use embedding when the content belongs *here*.

**Referencing** (the `@` operator on any base) creates a typed pointer to the target. The reference element is leaf-like — it carries an identifier that names the target and may be refined by Qualifiers, but it does not carry its own substructure. The target lives elsewhere; this element is just the pointer. Use referencing when you want to identify the target without including its content.

**Graph relationships** (the `EDGE` role) assert a labeled directed relationship between the parent Object and a target Object. EDGE elements are themselves structured — they may carry their own Properties expressing edge metadata such as weight, confidence, or timestamp. The edge is not just a pointer; it is a thing about the relationship, with attributes of its own. Use EDGE when the relationship itself is a meaningful structural unit in the graph.

The three mechanisms answer different questions:

- **Embedding** answers "what content does this property hold?"
- **Referencing** answers "what should this point at?"
- **Graph relationships** answer "what relationship exists between these entities, and what attributes does that relationship have?"

A reference to an Object (`@OBJ`) and an EDGE to an Object both carry an `@target` reference value, but they are structurally distinct. The reference is a pointer; the edge is a graph-statement element with its own children. Choose based on whether the link itself is a structural participant in your data (use EDGE) or merely an identifier of a target (use `@OBJ`).

### 3.7 Optional Elaboration

A GDX element is valid at its minimum form: the Prefix alone (e.g., `<STR>`, `<REC>`, `<OBJ>`) is sufficient for a well-formed element wherever its role is permitted. Every other field — `Id`, `Key`, `Rel`, `Resource`, `Type` — and the element's children are optional. An element with only a Prefix already declares its structural role and value-kind; richer claims are added only where the use case demands them.

Elaboration beyond the minimum follows two structurally distinct patterns.

**Identity is orthogonal.** The three identity fields — `Id`, `Key`, `Rel` — are independent dimensions. An element may carry any combination of them in any order, and the presence or absence of one places no constraint on the others. An element may have an `Id` and no `Key`; a `Key` and no `Id`; a `Rel` with both, with one, or with neither. Each axis serves a distinct addressing purpose:

- `Id` names the element within its scope (locally meaningful for Properties and Records; globally meaningful for Objects).
- `Key` provides a location-stable anchor for references (§5.4.5).
- `Rel` aligns the element to a cross-document vocabulary predicate (§3.6, §6.5.4).

The independence of these axes is what lets a single element be addressable in multiple ways simultaneously without conflict.

**Value elaboration is layered.** When an element carries content beyond its bare role, elaboration proceeds through nested layers. Each layer adds narrowing detail without altering what the prior layer said. The layers live in different places — some on the instance, some on the Template the instance references, some in the external vocabulary that defines a constraint identifier:

1. **Category** (instance) — the Prefix declares the element's structural role and, for value-bearing roles, its value-kind. The Prefix is the minimum.
2. **Value** (instance) — the Resource carries the concrete value, conforming to the value-kind declared by the Prefix. A value-bearing element without a Resource declares the slot but contributes no value (§4.3.3); with a Resource, it declares both.
3. **Structural identity** (instance) — a `--type` reference asserts that the element's content conforms to a named Template (§8.2). This narrows the value from "any value of this value-kind" to "a value satisfying this Template's slot structure."
4. **Constraint attachment** (Template, not instance) — Constraint Extensions on the referenced Template (§8.4) carry vocabulary-defined restrictions. These do not appear on the instance directly; they apply through the Template claim.
5. **Constraint semantics** (external vocabulary) — the meaning of each constraint identifier (`xsd:minInclusive`, `json:pattern`, etc.) is defined by the vocabulary that owns the identifier, evaluated externally by consumers that understand that vocabulary.

Each layer is optional given the layer above it: an element with a value need not have a `--type`; an element with a `--type` need not be constrained beyond the Template's structural declarations; constraints need not exist for every Template; no consumer is required to understand every constraint vocabulary. The minimum (Prefix alone) is at one end of this spectrum; a fully-elaborated element with full identity, value, type, and constraint-bearing Template is at the other.

This minimal-by-default discipline is what keeps simple GDX simple. A Property with just a name and a value is one line; the same element can grow identity and schema affordances as the use case demands, without restructuring.

---

## 4. The GDX Abstract Data Model

This section defines GDX at runtime: the abstract structure an implementation must represent, the rules an implementation must enforce, and the semantic states an implementation must distinguish. Concepts are in §3; text form is in §5.

The data model defined here is the single authoritative content that every concrete GDX representation faithfully realizes (§1.1). Standard Form (§5), Compact Form (companion specification), Binary Form (planned companion), and the programmatic in-memory form a runtime exposes are alternative realizations of the same model: each preserves the same elements, the same prefixes, the same field semantics, the same composition relationships. Conversions between representations transform the surface syntax without changing the model content; a round-trip through any combination of representations yields the same abstract document.

A GDX element is the tuple

```
(Prefix, Key?, Rel?, Id?, Resource?, Type?, Children*)
```

All fields except Prefix are optional at the data-model level. Whether a given role requires a particular field is governed by §6 (Element Roles) and §7 (Composition Axioms).

### 4.1 Element Fields

An element has six defined fields. The Prefix is required; the others are optional at the data-model level. Whether a particular field is required for an element depends on the element's role (§6) and its position relative to its parent (§4.2).

| Field      | Purpose |
|------------|---------|
| `Prefix`   | Declares the element's role. Required. Drawn from the closed vocabulary in §6. |
| `Key`      | An optional named anchor identifying the element within its enclosing Object's scope, independent of its tree position. Used as a target for `&Key` reference literals (§5.2.6). |
| `Rel`      | A predicate-style label naming the relationship between this element and its parent. |
| `Id`       | A local or resource identifier — the element's name in the document. |
| `Resource` | The element's typed scalar value. |
| `Type`     | A reference to a Template the element claims to satisfy. |

All identity fields (`Id`, `Key`, `Rel`) use case-sensitive ordinal comparison.

Implementations MUST represent these fields as distinct elements of an element's runtime structure. The order in which fields are stored or accessed is implementation-defined; only the children list has a normative order (§4.4).

### 4.2 Identifier Requirements

Identifier requirements apply to elements in *named-attribute positions* — positions where the element represents a distinct named property of its parent.

The following roles, when in named-attribute position, MUST carry at least one of `Id`, `Key`, or `Rel`:

- Object (`OBJ`/`O`, `DEFOBJ`)
- Record (`REC`/`R`, `%REC`/`%R`, `DEFREC`, `NREC`/`NR`) when not in Set member position
- Property (`STR`/`S`, `INT`/`I`, `FLOAT`/`F`, `BOOL`/`L`, `BYTE`/`B`, and their embedded/template variants) when not in Set member position
- Template (any `DEF…`)

Elements appearing as members of a Set (§6.5.13) are in *positional* rather than named-attribute position. They are not required to carry identifiers; they identify themselves by position and value within the Set. Identifiers on Set members are valid and MAY be used when meaningful (for example, to identify individual Records within a Record Set).

A Prefix is REQUIRED on every element regardless of position. Set members MUST carry their prefix even when their identifier is omitted, because the prefix distinguishes Set members from Qualifiers, Annotations, Extensions, and other elements that may appear within a Set's children.

A Property, Record, Edge, or other Resource-bearing element MAY appear with an identifier but no Resource section. The element is well-formed in this state. The semantic interpretation depends on context: in Template context, the form declares a slot without a default value; in instance context, the form represents a null value (§4.3.3).

The lexical grammar for identifiers is uniform across all identifier-bearing positions; see §5.4. The semantic interpretation of an identifier depends on the role that carries it:

- Identifiers on Property, Record, Edge, and Template elements name a relationship within the document. They are functionally equivalent to a field name, label, or variable name in a programming language.

- Identifiers on Object elements identify a resource and are used to resolve references to that Object. The specification places no constraint on the identifier scheme; paths (`/people/alice`), CURIEs (`gdx:Person`), GUIDs, and IRIs (`https://acme.com/people/alice`) are all valid.

This semantic distinction is not reflected in the grammar. A property identifier and an Object identifier draw from the same lexical production (§5.4); they differ only in how consumers interpret them.

### 4.3 Resource Value Kinds

A Resource, when present, MUST be one of:

| Value-Kind  | Description                                                                |
|-------------|----------------------------------------------------------------------------|
| `string`    | UTF-16 character sequence; quoted in text.                                 |
| `int`       | Mathematical integer (any whole number).                                   |
| `float`     | Real number. Representation is determined by suffix (§5.2).                |
| `bool`      | `true` or `false`.                                                         |
| `bytes`     | Opaque sequence of octets.                                                 |
| `reference` | A reference to another Object by identifier.                               |

The Resource value-kind MUST be consistent with the element's prefix family (string-family prefixes carry strings, int-family prefixes carry ints, etc.). Coercion between value-kinds is implementation-defined and SHOULD be avoided.

#### 4.3.1 Numeric Value-Kinds

The `int` value-kind represents any whole number. The `float` value-kind represents any real number. Neither value-kind constrains the numeric range or precision at the model layer. Constraints on range, width, or representation are expressed at the representation layer through suffix tags (§5.2) or at the schema layer through Templates (§8).

A conforming implementation MUST round-trip every representation-conforming numeric literal through serialization (Appendix A). An implementation that cannot represent a particular literal in its internal numeric type MUST reject the literal at parse time as a representation-conformance failure; it MUST NOT silently truncate, round, or coerce.

#### 4.3.2 Reference Value-Kind

A `reference` resource is a directional pointer to another Object by identifier. References are interpreted by a resolver, not by the GDX parser; the parser validates only that the literal is well-formed (§5.2.6).

The resolution scheme is system-defined. Common approaches include local-document resolution (the reference targets an Object within the same document), namespace-relative resolution (the reference targets an Object within a known scope), and global resolution (the reference is a fully qualified URI). The specification does not mandate any particular scheme; resolvers SHOULD attempt local resolution before broader scopes.

For the special case of Edge elements (§6.5.4), the Resource is always a reference value-kind, and the `#rel` field names the edge type.

#### 4.3.3 Absence and Null Values

A Property element exhibits one of three distinct states with respect to its value:

- **Absent**: the element does not appear in the document. Whether absence is permitted is a schema concern (§8).

- **Null**: the element is present but carries no Resource value. The Property exists structurally but has no value. In Template context, this form declares a slot without a default value; in instance context, this form represents a null value (analogous to SQL `NULL` or JSON `null`).

- **Present with value**: the element appears with an explicit Resource. The Property carries the indicated value.

These three states are structurally and semantically distinct. Implementations MUST represent them as distinct values at runtime; consumers MUST NOT conflate null with present-with-value, and SHOULD NOT conflate either with absent.

For string-typed Properties, the present-with-value state includes the empty string. The empty string is a legitimate string value, distinct from null — an explicit empty string is a value the Property carries, while a null Property carries no value at all. For other value-kinds, there is no "empty value" analogue — the integer zero is a value, not an "empty integer"; the boolean false is a value, not an "empty bool". The distinction between empty and null applies only to strings.

The following table summarizes the states and their typical use:

| State                | Semantic                                                  | When to use                                         |
|----------------------|-----------------------------------------------------------|-----------------------------------------------------|
| Absent               | The attribute does not apply or has not been determined   | The schema declares the field as optional; the value is genuinely unknown or inapplicable |
| Null                 | The attribute applies but has no value                    | The value is known to be missing (e.g., a person with no middle name) |
| Present with value   | The attribute has the indicated value                     | The value is known and is the indicated content     |
| Empty string (strings only) | The attribute has the empty-string value           | The value is known to be the empty string (rare in practice; usually indicates a default or placeholder value rather than a true absence) |

Distinguishing absent from null matters most when consumers act on the difference. Absent typically means "no information"; null typically means "information that the value is missing"; present-with-value means "this is the value." A schema designer who treats all three as equivalent is generally creating bugs that surface in transit between systems with different default behaviors.

The textual representation of these states — the bare-versus-empty-value forms a null element may take in serialization — is defined in §5.1.

### 4.4 Children

Children form an ordered list. Order is preserved through serialization and is significant for round-trip equivalence (Appendix A). The semantic significance of order is determined by the parent role and any Template the element claims to satisfy.

Set elements (§6.5.13) carry their members in document order. Set membership does not imply unordered semantics in the data-structure sense; the term refers to the multi-valued nature of the collection rather than to any ordering property.

Every child has exactly one parent. A node detached from its parent and not reattached is no longer part of the document.

---

## 5. Canonical Text Form

This section defines how GDX elements are represented as text: the syntax for writing an element, the textual markers (sigils and separators) that introduce its fields, the whitespace rules, and the literal grammars for each value-kind. Concepts are in §3; the data-model rules an implementation enforces are in §4.

Other text forms (compact, binary, alternative tab-indented) MAY be defined elsewhere and remain conforming so long as they preserve the fields, ordering, and values defined in §4.

### 5.1 Element Syntax

An element is written as a bracketed token sequence between `<` and `>`. The general form is:

```
<Prefix [&Key] [#Rel] [Id] [: Resource] [--Type]>
    [child elements ...]
>
```

Field write order is **Prefix · Key · Rel · Id · Resource · Type**. The Prefix is required; all other fields are optional.

A leaf element MAY be written on a single line:

```
<STR name : "Alice">
```

A non-leaf element places its children between its open and close brackets. The closing `>` matches the most recently opened unclosed `<`. Implementations MAY indent children for readability:

```
<REC address
    <STR street : "Main St">
    <INT number : 100>
>
```

#### 5.1.1 Sigils and Separators

A *sigil* is a single non-alphabetic character (or short character sequence) that introduces a field. The sigils used in field positions are:

| Sigil | Purpose |
|-------|---------|
| `&`   | Introduces the Key field |
| `#`   | Introduces the Rel field |
| `--`  | Introduces the Type field |
| `@`   | Introduces an identifier-mode reference literal (§5.2.6) |
| `$`   | Introduces a path-mode reference literal (§5.2.6); grammar specified in `GDX-Path.md` |

The character `:` is a *separator*, not a sigil. It divides an element into an identity portion (Prefix, Key, Rel, Id) and a value portion (Resource, Type). The Resource itself has no sigil; it is identified by appearing on the value side of `:`.

The sigils `&`, `#`, and `--` MUST be immediately followed by their field content with no intervening whitespace. Whitespace MAY appear before a sigil to separate it from the preceding token. Whitespace MAY appear on either side of `:`.

When the `:` separator appears with no identity field on the left in a context where an identifier is required, the element is malformed; conforming parsers MUST reject it. When the `--` sigil appears without a type identifier following, the element is malformed.

**Prefix-position characters** (`%`, `.`, `_`, `N`, `DEF`, `@`) and the **suffix-position character** (`*`) are not field sigils — they are part of the prefix token itself and are defined in §6.

#### 5.1.2 Whitespace Rules

Tokens within an element are separated by whitespace. The following rules apply:

- Whitespace MUST separate the Prefix from a following Id when no sigil-introduced field intervenes (so `<REC address>` not `<Raddress>`).
- The `:` separator MAY have whitespace on either side. The forms `<STR name:"Alice">`, `<STR name :"Alice">`, `<STR name: "Alice">`, and `<STR name : "Alice">` are all valid and equivalent.
- Whitespace MAY appear immediately after `<` and immediately before `>`.
- Whitespace within an element is otherwise not significant.

The canonical form used in this specification's examples places a single space between adjacent fields and around the `:` separator. Implementations producing GDX text MAY choose more compact forms (e.g., `<STR name:"Alice">`) where appropriate to context.

#### 5.1.3 Null Forms

The null state of a Property element (§4.3.3) has two equivalent textual forms:

- **Bare form** (canonical): `<STR middleName>`, `<INT age>`, `<FLOAT weight>`. The Resource section is omitted entirely.
- **Empty-value form**: `<STR middleName : >`, `<INT age : >`, `<FLOAT weight : >`. The `:` separator is present but no Resource value follows.

Both forms denote the same null state. The bare form is canonical for serialization (Appendix A.5); a conforming parser MUST accept either input form, but a conforming serializer SHOULD output the bare form. The empty-value form exists primarily to support natural human editing — when a user removes a value from a present element by deleting the literal but not the separator, the element remains valid rather than becoming malformed.

The same equivalence applies when a Type reference is present with no Resource value: `<STR name --gdx:string>` and `<STR name : --gdx:string>` are equivalent. The bare form (no `:`) is canonical.

### 5.2 Resource Literals

The Resource of an element, when present, is one of six value-kinds. Each value-kind has a defined literal form in the canonical text representation.

| Value-Kind  | Literal Form                                                             | Example                          |
|-------------|--------------------------------------------------------------------------|----------------------------------|
| `string`    | Double-quoted string with backslash escapes                              | `"Alice"`                        |
| `int`       | Decimal digits with optional sign and optional width suffix; alternative bases via `0b`/`0o` prefix | `42`, `25i8`, `0b1010`, `0o17` |
| `float`     | Decimal with point, optional width/representation suffix; or keyword     | `3.14`, `25.50d`, `1.5708f32`    |
| `bool`      | `true` or `false`                                                        | `true`                           |
| `bytes`     | Hex prefix form (canonical) or alternative encoding prefix               | `0x4a3f8c`                       |
| `reference` | At-prefixed identifier, optionally composed with a path                  | `@people/bob`, `$/people/[1]`    |

#### 5.2.1 String Literals

A string literal is a sequence of characters enclosed in double quotes. The following escape sequences are recognized:

| Escape | Meaning           |
|--------|-------------------|
| `\"`   | Literal `"`       |
| `\\`   | Literal `\`       |
| `\n`   | Newline           |
| `\t`   | Tab               |
| `\r`   | Carriage return   |
| `\u{X}` | Unicode codepoint, where `X` is one or more hex digits |

All other characters appear literally. The string content is interpreted as a UTF-16 character sequence.

```
<STR greeting : "Hello, world">
<STR path : "C:\\Users\\alice">
<STR quoted : "She said \"hello\"">
```

#### 5.2.2 Integer Literals

An integer literal is a sequence of decimal digits, optionally preceded by `-` or `+`, optionally followed by a width suffix.

The width suffix, when present, declares a representation-layer commitment to a specific integer width. The closed suffix vocabulary is:

| Suffix    | Representation                          |
|-----------|-----------------------------------------|
| `i8`      | 8-bit signed integer, range -128 to 127 |
| `i16`     | 16-bit signed integer                   |
| `i32`     | 32-bit signed integer                   |
| `i64`     | 64-bit signed integer                   |
| `u8`      | 8-bit unsigned integer, range 0 to 255  |
| `u16`     | 16-bit unsigned integer                 |
| `u32`     | 32-bit unsigned integer                 |
| `u64`     | 64-bit unsigned integer                 |
| `bigint`  | Arbitrary-precision integer             |

Suffix tokens MUST be immediately adjacent to the numeric portion with no intervening whitespace. A literal whose value exceeds the suffix's range MUST be rejected at parse time as a representation-conformance failure.

Untagged integer literals carry no representation commitment; the implementation chooses an internal representation sufficient to round-trip the literal (Appendix A).

```
<INT count : 42>
<INT temperature : 25i8>
<INT packetSize : 1500u16>
<INT balance : 9999999999999999999bigint>
```

Alternative bases are supported through prefix forms:

| Prefix | Base    | Example      |
|--------|---------|--------------|
| `0b`   | Binary  | `0b1010`     |
| `0o`   | Octal   | `0o17`       |

Width suffixes MAY follow base-prefix literals: `0b11111111u8` is a valid representation of 255 as an unsigned 8-bit integer.

#### 5.2.3 Float Literals

A float literal represents a real number. Three forms are defined:

**Numeric form:** decimal digits with a decimal point, optional sign, optional exponent (`e` or `E` followed by signed digits), and optional representation suffix.

**Integer-form:** decimal digits with optional sign and required representation suffix, or decimal digits with optional sign and optional exponent. An integer-form literal in a float context represents the corresponding real-valued number (`5` represents the real number 5).

**Keyword form:** the keywords `nan`, `inf`, and `-inf` representing IEEE 754 special values.

The closed representation suffix vocabulary for floats is:

| Suffix  | Representation                                    |
|---------|---------------------------------------------------|
| `f32`   | IEEE 754 binary32 (single-precision binary float) |
| `f64`   | IEEE 754 binary64 (double-precision binary float) |
| `d`     | Arbitrary-precision decimal                       |
| `d32`   | IEEE 754-2008 decimal32                           |
| `d64`   | IEEE 754-2008 decimal64                           |
| `d128`  | IEEE 754-2008 decimal128                          |

The suffix commits the literal to a specific representation:

- Untagged literals (e.g., `3.14`) carry no representation commitment. The implementation chooses a representation sufficient to satisfy round-trip text equivalence.
- Binary float suffixes (`f32`, `f64`) commit the value to IEEE 754 binary floating-point at the indicated width.
- Decimal suffixes (`d`, `d32`, `d64`, `d128`) commit the value to a decimal representation, exact for any finite decimal literal.

The choice of representation has consequences for arithmetic exactness, round-trip precision, and inter-system interchange. Applications carrying values where decimal precision matters (currency, exact-precision measurement) SHOULD use a decimal suffix or bind the property to a Template-defined semantic type that constrains representation.

```
<FLOAT temperature : 21.5>
<FLOAT angle : 1.5708f32>
<FLOAT price : 25.50d>
<FLOAT precise : 9.999999999999999d128>
<FLOAT infinity : inf>
<FLOAT undefined : nan>
<FLOAT count : 5>
<FLOAT angleAtRight : 90f32>
<FLOAT balance : 1000d>
```

The integer-form examples (`5`, `90f32`, `1000d`) are valid in float-prefix positions because integers are real numbers; the prefix declares the value-kind and the literal value is representable as a real.

A literal whose precision exceeds the representable precision of the suffix MUST be rejected at parse time as a representation-conformance failure.

Numeric literals are culture-invariant. `.` is the decimal separator. `e` or `E` introduces a base-10 exponent.

**Asymmetry with integer literals.** Float literals (literals containing a decimal point, an exponent, a float suffix, or a float keyword) are NOT valid in integer-prefix positions. A literal with a decimal point implies a real-valued representation regardless of whether the value is integer-valued: `<INT count : 5.0>` is malformed even though 5.0 represents the integer 5. The asymmetry is intentional — integers are a subset of reals, but reals are not a subset of integers, and the literal form signals the intended representation domain.

#### 5.2.4 Boolean Literals

A boolean literal is one of the following forms:

| Form        | Value |
|-------------|-------|
| `true`      | true  |
| `false`     | false |
| `t`         | true  |
| `f`         | false |

Boolean literals are case-insensitive on read. The forms `TRUE`, `True`, `T`, `FALSE`, `False`, and `F` are all valid input. Conforming implementations write boolean values in the lowercase spelled-out form (`true`, `false`); the single-letter forms exist to support input from systems that use abbreviated boolean conventions (PostgreSQL `t`/`f`, certain command-line tools) and to accommodate compact human entry.

```
<BOOL active : true>
<BOOL deleted : false>
<BOOL verified : t>
<BOOL hidden : F>
```

The single-letter forms are unambiguous in Resource position because the prefix declares the value-kind. `<BOOL flag : t>` parses `t` as the boolean true; the same input outside a bool-prefix position would parse as the identifier `t`.

#### 5.2.5 Bytes Literals

A bytes literal is an opaque sequence of octets. Two encoding forms are defined:

| Prefix | Encoding | Example                          |
|--------|----------|----------------------------------|
| `0x`   | Hex (canonical)        | `0x4a3f8c`         |
| `0z`   | Base64 (alternative)   | `0zSj+M`           |

The hex form is the canonical representation. The base64 form is provided as an alternative for cases where compactness matters.

Hex digits are case-insensitive on read; conforming implementations write hex bytes in lowercase. The total number of hex digits MUST be even.

Base64 uses the standard alphabet (RFC 4648). Padding (`=` characters) is required to align the encoded length to a multiple of four characters.

```
<BYTE contentHash : 0xe3b0c44298fc1c149afbf4c8996fb92427ae41e4>
<BYTE publicKey : 0x4a3f8c2b>
<BYTE compactBlob : 0zSGVsbG8sIHdvcmxkIQ==>
```

A bytes literal with no following digits (`0x` alone) is malformed. A hex literal with an odd number of digits is malformed. A base64 literal with invalid padding is malformed.

#### 5.2.6 Reference Literals

A reference literal designates another addressable element. Five forms are recognized, combining the three operations of name resolution (`@`), structural traversal (`$`), and key lookup (`&`):

```ebnf
ReferenceLiteral  ::= IdRef | PathRef | KeyRef | IdPathRef | IdKeyRef
IdRef             ::= "@" Anchor
PathRef           ::= "$" PathExpression
KeyRef            ::= "&" KeyToken
IdPathRef         ::= "@" Anchor "$" PathExpression
IdKeyRef          ::= "@" Anchor "&" KeyToken

Anchor            ::= (AnchorRole ":")? Identifier
AnchorRole        ::= "OBJ"  | "REL"  | "PKG"  | "GRP"
                    | "DEFOBJ" | "DEFREC" | "DEFSTR"
                    | "DEFINT" | "DEFFLOAT" | "DEFBOOL" | "DEFBYTE"
```

The `Anchor` production carries an optional role marker — a closed-vocabulary keyword drawn from §6.2 followed by `:` — that disambiguates the role of the resolved element. The marker is most useful in composed references where the anchor's role differs from the terminal target's role; in plain `@id` references and in composed references with Object anchors, the marker is optional. When omitted in a composed reference, the anchor defaults to an Object (the most common case).

The `PathExpression` and `KeyToken` grammars and the resolution algorithms are specified in the companion GDX Path Expressions Specification (`GDX-Path.md`). Implementations MUST conform to that specification for the contents of any `$`- or `&`-introduced literal and for the resolution semantics of role markers.

**Identifier-mode** (`IdRef`) is an `@` sigil followed by an identifier:

```
@people/bob
@gdx:Person
@7f3a2b9c-4e21-4f88-91d5-abcdef012345
@https://acme.com/people/alice
```

The identifier following `@` follows the lexical rules in §5.4. The reference is resolved by the implementation's reference resolver; the parser validates only that the literal is well-formed. The `@` sigil MUST be immediately followed by the identifier with no intervening whitespace.

**Path-mode** (`PathRef`) is a `$` sigil followed by a path expression. The path traverses the structural tree of the current document, starting at the document root, addressing an element by its location:

```
$/people/alice/mailing
$/orders/[3]/#gdx:postalAddress
```

The `$` sigil MUST be immediately followed by the path expression with no intervening whitespace.

**Key-mode** (`KeyRef`) is an `&` sigil followed by a Key token. The Key is looked up within the scope of the reference (the enclosing Object, or the document for transport-layer positions; see §5.4.5):

```
&entry-12345
&shippingAddress
&7f3a2b9c-4e21-4f88-91d5-abcdef012345
```

The `&` sigil MUST be immediately followed by the Key token with no intervening whitespace.

**Composed name + path** (`IdPathRef`) concatenates identifier-mode with path-mode. The `@Identifier` portion resolves to a starting element by scope walk; the `$PathExpression` portion is then evaluated rooted at that starting element:

```
@gdx:bob$/contacts/[1]
@https://acme.com/people/alice$/address/[2]
```

**Composed name + key** (`IdKeyRef`) concatenates identifier-mode with key-mode. The `@Anchor` portion resolves to a starting element; the `&Key` portion is then looked up within that element's scope. When the anchor carries no role marker, the starting element defaults to an Object:

```
@gdx:bob&shippingAddress                 // Object anchor (default)
@system/audit-log&entry-12345            // Object anchor (default)
@REL:gdx:knows&annotation-3              // REL anchor (explicit)
@DEFOBJ:com.acme.Customer&firstName-doc  // Template anchor (explicit)
```

The two composed forms are the canonical mechanisms for cross-document addressing. The `$` and `&` operators do not combine within a single reference literal; pick `IdPathRef` for structural traversal or `IdKeyRef` for key-based anchoring.

Because `$` and `&` are not members of `IdContChar`, the boundary between the identifier portion and the path or key portion is lexically unambiguous. The optional `AnchorRole` keyword before `:` is distinguished from a CURIE prefix (`gdx:`, `schema:`) by lookup against the closed `AnchorRole` set: only `OBJ`, `REL`, `PKG`, `GRP`, `DEFOBJ`, `DEFREC`, `DEFSTR`, `DEFINT`, `DEFFLOAT`, `DEFBOOL`, `DEFBYTE` are recognized as role markers.

**Parse-time vs. resolution-time validation of role markers.** The parser MUST reject an unknown role keyword (one not in the closed set). At resolution time, the resolver MUST verify that the named element actually has the claimed role; mismatch yields an unresolved reference.

References are commonly used as the Resource of Edge elements (§6.5.4) or of `@KIND` reference elements (§6.5.16):

```
<OBJ alice
    <EDGE #knows : @people/bob>
    <@OBJ employer : @com.acme>
    <@REC mailing : $/addresses/[1]>
    <@REC home    : &homeAddress>
>
```

**Address-mode singularity.** A reference literal whose Resource is a `PathRef`, `KeyRef`, `IdPathRef`, or `IdKeyRef` resolves to a single element. The reference is resolved if and only if path or key evaluation yields exactly one element whose role matches the enclosing prefix; zero matches, multiple matches, and a single match of non-matching role all yield an unresolved reference. Resolvers MUST NOT silently select a first match in the multi-match case. See `GDX-Path.md` §7 for the normative definition.

The five reference forms serve different addressing strategies. Identifier-mode looks up a target by name in the appropriate scope. Path-mode traverses a structural path to reach an element that may not have a global identifier. Key-mode looks up an element by its location-stable Key anchor. The composed forms combine name resolution with one of structural traversal or key lookup. The choice among them is the author's, based on what the target is, where it lives, and how stably it is identified. When both a Key and a path could address the target, prefer the Key form — Keys are location-stable, paths break under restructuring.

### 5.3 Type References

A Type reference attaches to an element via the `--` sigil and follows the Resource (or the Id, if no Resource is present):

```
<REC address : --com.acme.Address
    <STR street : "Main St">
    <INT number : 100>
>

<DEFOBJ MyAddress --schema:address
    <STR street : "Broadway">
    <STR city : "Oakland">
>
```

The `--` sigil MUST be preceded by whitespace separating it from the previous token, and MUST be immediately followed by a Type identifier with no intervening whitespace. The Type identifier follows the lexical rules in §5.4.

A Type reference asserts that the element's contents satisfy the named Template. Resolution and validation are described in §8.

### 5.4 Identifier Lexical Rules

A single identifier grammar applies to all identifier-bearing positions: the `Id` field, the contents of the `Key` field (when not numeric), the contents of the `Rel` field, and the contents of the `Type` field.

#### 5.4.1 Identifier Grammar

```
Id              ::= IdStartChar IdContChar*

IdStartChar     ::= ALPHA | "_"
IdContChar      ::= ALPHA | DIGIT | "_" | "-" | "." | "/" | ":"

ALPHA           ::= "A".."Z" | "a".."z"
DIGIT           ::= "0".."9"
```

An identifier MUST begin with a letter or an underscore. It MAY continue with any combination of letters, digits, underscores, hyphens, periods, forward slashes, and colons.

The forward slash (`/`) is admitted as a continuation character so that IRIs (e.g., `https://acme.com/people/alice`) remain valid identifiers. The forward slash is **not** admitted as a start character: identifiers may not begin with `/`. This narrowing (introduced in v0.11.0) reserves the leading-slash syntactic slot for path-mode reference literals (§5.2.6 and the companion GDX Path Expressions Specification). Documents conforming to v0.10.0 or earlier that use leading-slash identifiers MUST be migrated; see `GDX-Path.md` Appendix C for migration patterns.

Identifiers MUST NOT begin with a digit (which would be lexically ambiguous with numeric literals). Identifiers MUST NOT contain whitespace, brackets, sigil characters (other than as listed above), or other punctuation.

Single-letter identifiers are valid. Position parsing disambiguates them from prefix tokens: the first non-whitespace token after `<` (and any modifier sigil) is always the prefix; subsequent unsigiled tokens in identifier position are identifiers.

#### 5.4.2 Identifier Forms

The grammar is uniform but admits several conventional forms. The following are all valid and serve common purposes:

| Form               | Example                                | Common use                                  |
|--------------------|----------------------------------------|---------------------------------------------|
| Local name         | `name`, `firstName`, `customer_id`     | Property and Record identifiers             |
| Single letter      | `r`, `g`, `b`, `x`, `y`                | Mathematical or abbreviated conventions     |
| Hierarchical local | `people/alice`, `orders/123`           | Object identifiers within a hierarchy (no leading `/`) |
| CURIE              | `gdx:Person`, `schema:givenName`       | Object IDs, Type refs, Rel refs             |
| Dotted CURIE local | `gdx:com.acme.Person`                  | Namespace-qualified types                   |
| GUID / opaque      | `7f3a2b9c-4e21-4f88-91d5-abcdef012345` | System-generated identifiers                |
| IRI                | `https://acme.com/people/alice`        | Globally resolvable identifiers             |

All forms work in any identifier position because the grammar accepts the union of their characters. An identifier `gdx:Person` may appear as a Property identifier (unusual but valid), an Object identifier, a Type reference, or a Rel reference. Consumers interpret the identifier in context.

CURIEs are typically expanded to full IRIs by the resolver. The choice between IRI and CURIE is a tradeoff between explicitness (IRI) and compactness (CURIE):

- IRI form: `<OBJ https://acme.com/people/alice ...>`
- CURIE form: `<OBJ acme:people/alice ...>` (where `acme:` resolves to `https://acme.com/`)

This identity model parallels RDF's: globally resolvable identifiers are the canonical scheme; CURIEs and other syntactic forms (paths, local names, GUIDs) are conveniences for specific use cases.

When an IRI appears in `Rel` position, the lexer recognizes it by the `://` scheme separator and parses it as a URI reference following RFC 3986 syntax. In other identifier positions, the same input is parsed as a maximal identifier (which incidentally captures the URI-like form because all its characters are valid IdContChars). Either way, the value carried is the same identifier; the distinction is lexical.

#### 5.4.3 Key Field Values

The `Key` field, when introduced by `&`, MAY take a purely numeric form in addition to the standard identifier form. Both forms are valid anchor values:

```
<STR &1 : "Broadway">
<REC &shippingAddress
    <STR street : "Main St">
>
<REC &7f3a2b9c-4e21-4f88-91d5-abcdef012345 #log:event ...>
```

A numeric Key is one or more decimal digits. It is distinguished from an identifier by being all-digits; an identifier may not begin with a digit, so the two forms are unambiguous.

The Key field provides a location-stable anchor for references (§5.2.6, §3.2). The choice between numeric and named Keys is the author's; numeric Keys are convenient for sequential identifiers (log entries, append-only records) while named Keys are convenient for semantic anchors (named addresses, designated child records). Keys are NOT slot positions for Template-instance alignment; that alignment uses the `Id` field (§8.2).

#### 5.4.4 Identifier Length and Collisions

The identifier grammar imposes no maximum length. In practice, identifiers should be long enough to be meaningful and short enough to keep documents readable.

Practical guidance:

- Local identifiers (Property names, Record fields) typically fit comfortably in 1-50 characters. Longer names accumulate visual weight without adding clarity.
- Hierarchical local identifiers (`orders/2026/04/29/customer-12345`) may run longer when they encode hierarchy; readability remains the constraint. Such identifiers do not begin with `/` (see §5.4.1); the leading-slash form is the path-mode reference syntax (§5.2.6).
- IRI-form identifiers may run to several hundred characters when full URIs are used. This is acceptable but compact identifiers (CURIEs, GUIDs) are preferred for documents that are read or edited by humans.
- Implementations SHOULD impose an upper limit on identifier length (a reasonable default is 1024 characters) to bound memory use and prevent denial-of-service via pathologically long identifiers in untrusted documents.

Identifier collisions across documents are an application concern, not a parser concern. Two documents using `gdx:Person` are referencing the same conceptual identifier; whether they refer to the same Template depends on the resolver's vocabulary mapping (§8.5.2). Within a single document, identifiers in the same scope MUST be unique among elements that compete for resolution (e.g., two top-level Objects with the same Id is malformed); identifiers in different scopes (different parents) are independent.

#### 5.4.5 Key Uniqueness and Scope

When an element carries a `Key` field, the Key value participates in a scope-local namespace used for `KeyRef` and `IdKeyRef` reference resolution (§5.2.6).

Key scope is determined by the element's position in the document:

- **Object-scoped.** An element whose nearest enclosing addressable container is an Object has its Key scoped within that Object. Two elements inside the same Object MUST NOT share a Key value.
- **Document-scoped.** An element at transport-layer position — a direct child of a Package, Group, or document root — has its Key scoped within the document. Two elements at this level MUST NOT share a Key value.

The enclosing-Object rule means that two different Objects MAY each contain an element with the same Key (`&entry-12345` in two different log Objects is permitted), because Key resolution within each Object is independent. The composed `IdKeyRef` form (`@anchor-id&key`) pins the scope explicitly when addressing across Object boundaries.

**Default anchor role in composed references.** In a composed reference (`IdPathRef`, `IdKeyRef`), if the `@Anchor` portion carries no role marker, the resolved element MUST be of role Object. This default reflects the common case for cross-document addressing. To anchor a composed reference at a non-Object element (a REL, a Template, a Package, or a Group), the author MUST use an explicit role marker (§5.2.6).

Key collisions within the same scope make the document malformed; conforming parsers SHOULD emit a diagnostic, and conforming resolvers MUST NOT silently pick one matching element over another.

The numeric Key form (§5.4.3) participates in the same scope rules as the named Key form. Numeric Keys are compared as strings (ordinal equality); `&01` and `&1` are distinct Keys.

Keys are OPTIONAL. An element MAY omit the Key field entirely; in that case the element is addressable only by `Id` (if present) or by structural path. The decision to carry a Key is the author's, based on whether the element needs to be a stable reference target.

### 5.5 Whitespace Between Elements

§5.1.2 covers whitespace within an element. This section covers whitespace between elements.

The canonical form for single-line elements uses a single space between adjacent tokens and around the `:` separator. Multi-line elements indent children for readability. The amount of indentation is implementation choice; conforming implementations preserve element structure but MAY normalize whitespace during round-trip.

Newlines, carriage returns, and tabs are equivalent to spaces for the purpose of token separation. A line break does not terminate an element; element termination is signaled exclusively by the matching `>` bracket.

### 5.6 Comments

GDX has no comment syntax in the canonical text form. Documentation that should travel with the data is expressed through Annotation elements (`REM`, §6.5.6). Documentation that should not travel with the data is expressed in surrounding application or build context, not in the GDX document itself.

### 5.7 Use-Case Adaptability

The same text form scales across a wide range of complexity without changing format. An application that needs only typed key-value data uses Properties and nothing else; an application that needs full graph semantics with schema validation uses the same parser and the same syntax with more features engaged. Choosing which features to use is an application decision, not a maturity progression.

The simplest form is typed key-value:

```
<STR databaseHost : "db.internal">
<INT databasePort : 5432>
<BOOL tlsEnabled : true>
```

A richer form adds composition, predicates, and schema:

```
<OBJ CurrentConfig
    <REC Server : --vocab:ServerConfig
        <STR #vocab:hostName databaseHost : "db.internal">
        <INT #vocab:portNumber databasePort : 5432>
        <STR #vocab:primaryIP ipAddress : 192.168.1.1 --vocab:ipv4format>
        <BOOL tlsEnabled : true>
        <EDGE #vocab:dependsOn cache : @cache-primary>
    >
>
```

Both are valid GDX. The simple form is not a stripped-down version of the rich form; both use the same syntax and the same parser. Applications pay for the features they use and not for the ones they don't. The GDX Cookbook (companion specification) demonstrates idiomatic usage at each level of feature engagement.

---

## 6. Element Roles

An element's **role** is its structural position in the data model. The role is declared by the element's prefix; the closed set of roles is enumerated by the closed set of prefixes. This section introduces the prefix vocabulary in three layers — the base roles (§6.2), the set suffix and reference operator (§6.3), and the refinement modifiers (§6.4) — then provides detailed treatment of each role (§6.5).

The prefix vocabulary is **closed** and committed against future change. Implementations MUST NOT introduce new prefixes; extensibility is provided through Templates (§8), Extensions, and the predicate vocabulary mechanism. New primitive types, new structural roles, or new domain concerns are expressed by composing existing prefixes with Templates, Extensions, and vocabulary identifiers — never by adding a new prefix.

A prefix in GDX is composed according to the pattern:

```
[@?] [refinement-modifier?] [base] [*?]
```

Where:
- **base** is one of the core role prefixes (§6.2)
- **`*` suffix** marks the element as a Set (§6.3.1)
- **`@` prefix** marks the element as a typed reference (§6.3.2)
- **refinement-modifier** is one of `.`, `_`, `%`, `N` (§6.4) or the `DEF` declaration-construct (§6.5.14)

All four slots are independent. The base is required; the others are optional. Day-to-day GDX use involves the base prefixes plus the `*` suffix when a collection is needed. The `@` operator and the refinement modifiers are specialized constructs used in particular contexts (typed references, vocabulary metadata, schema declarations).

### 6.1 Canonical and Minified Forms

GDX prefix tokens are textual renderings of underlying type tags. The same data model is expressible in two equivalent textual forms:

- **Canonical form** uses three-letter (or longer where natural) prefix names. Canonical is the default for authoring, storage, content addressing, and inter-system exchange where readability matters.
- **Minified form** uses single-letter prefix names. Minified is for transport-efficiency contexts (high-frequency machine-to-machine traffic, embedded systems, size-constrained storage) where every byte matters.

Both forms are equivalent at the data-model level. Conversion is deterministic; round-tripping a document through canonical and minified forms preserves the data exactly.

**Recognition rules:**

- Implementations MUST recognize both canonical and minified forms on input.
- Prefix tokens are case-insensitive. `STR`, `Str`, `str`, and `sTR` are all recognized; the canonical case for output is uppercase.
- Mixed forms within a document are accepted on input. The serializer normalizes to the configured output style.
- Identifiers (Id, Key, Rel) remain case-sensitive (§5.4). The case-insensitive rule applies only to prefix tokens at prefix position.

**Form is style, not data.** Round-trip preserves data and configured output style, not authorial form choices. Tools serializing in canonical form produce uppercase three-letter prefixes regardless of input case or form.

**Strict canonical for content addressing.** The strict canonical form (Appendix A.5), used for content addressing and signing, uses canonical prefix names. Minified output is valid GDX but does not produce strict canonical bytes.

**Prefixes without minified forms.** Some prefixes are canonical-only — they are already three or more letters and represent infrequent declarative items where minification would save little: `REL`, `REM`, `PKG`, `BLK`, `GRP`, and all `DEF…` Template prefixes. These are identity-mapped between canonical and minified output: a minified document still contains `<REL gdx:knows>` and `<DEFREC Customer>`.

### 6.2 Base Vocabulary

The base prefixes are the core role vocabulary. They group into four categories:

**Value primitives** — typed scalar values:

| Canonical | Minified | Role     | Resource value-kind | Detail   |
|-----------|----------|----------|---------------------|----------|
| `STR`     | `S`      | Property | `string`            | §6.5.3   |
| `INT`     | `I`      | Property | `int`               | §6.5.3   |
| `FLOAT`   | `F`      | Property | `float`             | §6.5.3   |
| `BOOL`    | `L`      | Property | `bool`              | §6.5.3   |
| `BYTE`    | `B`      | Property | `bytes`             | §6.5.3   |

**Structural roles** — composite entities and organization:

| Canonical | Minified | Role   | Resource value-kind | Detail   |
|-----------|----------|--------|---------------------|----------|
| `OBJ`     | `O`      | Object | —                   | §6.5.1   |
| `REC`     | `R`      | Record | —                   | §6.5.2   |
| `BLK`     | —        | Block  | —                   | §6.5.5   |
| `EDGE`    | `E`      | Edge   | `reference`         | §6.5.4   |

**Declaration roles** — vocabulary and schema declarations:

| Canonical | Minified | Role                 | Detail   |
|-----------|----------|----------------------|----------|
| `REL`     | —        | Relation Declaration | §6.5.15  |
| `DEF…`    | —        | Template (see §6.5.14) | §6.5.14 |

**Transport roles** — exchange-layer wrappers:

| Canonical | Minified | Role    | Detail   |
|-----------|----------|---------|----------|
| `REM`     | —        | Annotation | §6.5.6 |
| `PKG`     | —        | Package | §6.5.7   |
| `GRP`     | —        | Group   | §6.5.8   |

### 6.3 Special Operators

Two operators compose with the base vocabulary to produce new roles. They are structurally distinct from the refinement modifiers (§6.4) because they change what value-kind the resulting element carries.

#### 6.3.1 The Set Suffix `*`

The `*` suffix marks an element as a Set: a multi-valued collection whose members share the base's role. Set composition reads naturally as "base, plural":

```
<STR* tags
    <STR : "data">
    <STR : "format">
>

<REC* contacts
    <REC alice <STR email : "a@example.com">>
    <REC bob   <STR email : "b@example.com">>
>
```

The Set's value-kind is "collection of base-instances," distinct from the base's single-valued instance form. A `STR*` element holds multiple strings; an individual `STR` element holds one string.

The `*` suffix composes with value primitives (`STR*`, `INT*`, `FLOAT*`, `BOOL*`, `BYTE*`) and with the Record structural role (`REC*`). It does not compose with Objects directly — multiple Objects appear as siblings rather than as members of a Set.

The `*` suffix MAY combine with refinement modifiers: `.STR*` is a Set of string Qualifiers, `_INT*` is a Set of integer Extensions, and so on. See §6.4.

Detailed treatment of Sets, including empty-vs-absent semantics, is in §6.5.13.

#### 6.3.2 The Reference Operator `@`

The `@` prefix marks an element as a typed reference: an element whose Resource is an identifier naming another addressable element. The base names *what kind of thing* is being referenced:

```
<@OBJ employer : @com.acme>          -- reference to an Object
<@REL #gdx:inverse : @gdx:reportsTo> -- reference to a REL declaration
<@DEFOBJ #conformsTo : @Customer>    -- reference to a Template
```

The `@` operator composes universally: it may prefix any base or modifier-base composition. The composition `@KIND` reads as "a reference to a KIND-element." Examples:

- `@OBJ` — reference to an Object
- `@REL` — reference to a REL declaration
- `@DEFSTR` — reference to a string-Template
- `@.STR` — reference to a string Qualifier
- `@STR*` — reference to a Set of strings
- `@.STR*` — reference to a Set of string Qualifiers

The composition is constrained by two rules:

- **No reference-to-reference.** `@@KIND` compositions are malformed. A reference points at an addressable element directly, not at another reference.
- **Modifier exemption.** Unlike the refinement modifiers (§6.4), `@` may compose with bases that themselves carry one of those modifiers. `@.STR` (reference to a Qualifier) and `@_BOOL` (reference to an Extension) are valid.

**Reference elements are leaf-like.** A `@KIND` element carries its target identifier as its Resource and MAY carry Qualifiers (and Annotations) as children. It does NOT carry Properties or other structural children. Reference elements are typed pointers — they identify a target without having their own substructure. This distinguishes them from EDGE (§6.5.4), which is a structured graph-statement element with its own Properties expressing edge metadata.

Detailed treatment of the `@KIND` family, including resolution rules and use cases, is in §6.5.16.

### 6.4 Refinement Modifiers

Four refinement modifiers compose with value primitives (and in some cases Records and Objects) to produce elements that carry the base's value-kind but function in a refined contextual role. They do not change what value an element carries; they change what the element *does* with that value.

| Modifier | Name      | Role                                                  | Composes with               |
|----------|-----------|-------------------------------------------------------|-----------------------------|
| `.`      | Qualifier | Refines the meaning of the parent element             | value primitives            |
| `_`      | Extension | Declares a schema constraint on a Template            | value primitives            |
| `%`      | Embedded  | Places a structural value in property-value position  | value primitives, REC, OBJ  |
| `N`      | Node-zone | Marks a semi-structured subtree                       | value primitives, REC       |

Examples:

```
<FLOAT temperature : 21.5
    <.STR unit : "celsius">             -- Qualifier refining what 21.5 means
>

<DEFSTR Email
    <_STR pattern : "^[^@]+@[^@]+$">    -- Extension constraining Email instances
>

<STR contact
    <%REC details                       -- Embedded Record in property-value position
        <STR email : "a@example.com">
    >
>

<NREC audit                             -- Node-zone container for ad-hoc data
    <NSTR source : "import">
>
```

**Refinement modifiers do not stack with each other.** `*%STR`, `%REC*`, `.NSTR`, `N.STR`, `%_STR` and similar combinations are all malformed. Each element carries at most one refinement modifier.

**Refinement modifiers MAY combine with `*` and `@`.** `.STR*` is a Set of Qualifiers; `_STR*` is a Set of Extensions; `%REC*` is a Set of Embedded Records. The reference operator `@` may prefix any modifier-base composition: `@.STR`, `@_BOOL`, `@%REC`, `@NSTR`. The pattern `[@?][refinement-modifier?][base][*?]` governs which combinations are well-formed.

**The `DEF` declaration construct** (§6.5.14) shares the prefix-position slot with the refinement modifiers but is structurally different — it declares a Template rather than refining an element's contextual role. See §6.5.14 for details.

### 6.5 Roles in Detail

This section provides detailed treatment of each role introduced in §6.2, §6.3, and §6.4. Each subsection covers one role, its prefix(es), composition rules, identifier requirements, and worked examples.

#### 6.5.1 Object — `OBJ`

| Canonical | Minified | Resource? | Identifier |
|-----------|----------|-----------|------------|
| `OBJ`     | `O`      | —         | required (resource identifier) |

An Object is an identifiable entity that may be referenced by other Objects. Objects are the primary type for data that will be persisted and exchanged. An Object's identifier is a resource identifier: a path, CURIE, GUID, IRI, or other globally meaningful identifier. The specification places no constraint on the scheme.

The root of a GDX document MUST be one or more Objects, Templates, REL declarations, or Packages. Objects may not contain other Objects except through the Embedded modifier (§6.5.11) or as transport-layer payload (§6.5.7, §9).

```
<OBJ people/alice
    <STR firstName : "Alice">
    <INT age : 24>
>
```

#### 6.5.2 Record — `REC`

| Canonical | Minified | Resource? | Identifier |
|-----------|----------|-----------|------------|
| `REC`     | `R`      | —         | required (local) when in named-attribute position |

A Record is a composite of named properties — the canonical class instance or struct equivalent in GDX. A Record *contains* its children, in contrast to a Block which merely *labels* a region of children (§6.5.5). A Record's identifier is a local identifier, meaningful within the document.

A Record may contain Properties, Sets, Edges, nested Records, Qualifiers, Nodes, and Annotations. Records may not contain Objects (except through the Embedded modifier, §6.5.11), Groups, or Extensions.

```
<REC address
    <STR street : "Main St">
    <INT number : 100>
>
```

#### 6.5.3 Property — `STR`, `INT`, `FLOAT`, `BOOL`, `BYTE`

Value primitives carry typed scalar values as their Resource. The Resource value-kind MUST match the prefix family.

| Canonical | Minified | Resource value-kind |
|-----------|----------|---------------------|
| `STR`     | `S`      | `string`            |
| `INT`     | `I`      | `int`               |
| `FLOAT`   | `F`      | `float`             |
| `BOOL`    | `L`      | `bool`              |
| `BYTE`    | `B`      | `bytes`             |

Properties require an identifier when in named-attribute position and may carry Annotations, Qualifiers, Extensions, and Nodes — but not other structural children. The three-state semantics (Absent, Null, Present-with-value) are defined in §4.3.3.

```
<STR name : "Alice">
<INT age : 24>
<FLOAT weight : 5.7>
<BOOL active : true>
<BYTE contentHash : 0xe3b0c44298fc1c14>
```

#### 6.5.4 Edge — `EDGE`

| Canonical | Minified | Resource value-kind | Identifier |
|-----------|----------|---------------------|------------|
| `EDGE`    | `E`      | `reference`         | `#rel` field required |

An Edge expresses a directed relation between two entities. Edges are GDX's representation of labeled property graph (LPG) edges: the parent element is the source, the Resource is a reference (`@target-id`) designating the target Object, and the `#rel` field names the edge type.

```
<OBJ alice
    <EDGE #knows : @people/bob>
>
```

This reads as: alice — knows → bob. The edge is unidirectional in the data model.

Edges accept the same children as Records and MAY carry properties as edge attributes. An edge with attributes is a labeled property graph edge in the full sense:

```
<EDGE #knows : @people/bob
    <STR since : "2024-01-15">
    <FLOAT weight : 0.8>
>
```

The `#rel` field references a predicate. A predicate may be declared as a first-class `REL` element (§6.5.15) when domain, range, inverse, or other semantic properties need to be expressed. The core specification makes no claim about the symmetry, transitivity, or other semantic properties of any predicate; those are declared at the REL level and computed by consumers.

Multiple edges MAY originate from the same source referencing the same target with different `#rel` tags, allowing multiple forms of relationship between the same pair of entities.

**Edge versus `@OBJ`.** Both EDGE and `@OBJ` (§6.5.16) carry references to Object targets, but they are structurally distinct. EDGE is a graph-statement element with its own children, including Properties that express edge metadata (weight, confidence, timestamp). `@OBJ` is a leaf-like typed pointer — it identifies a target without having its own substructure beyond Qualifiers. Use EDGE when the link is a structural participant in the graph with attributes of its own; use `@OBJ` when the link is just an identifier. See §3.6 for the full comparison.

##### 6.5.4.1 Edge Reference Resolution

An Edge's Resource is a reference to a target Object. Reference resolution determines whether the named target exists and what Object it identifies. Resolution behavior parallels Template reference resolution (§8.5.2) but operates on Object identifiers rather than Template names.

The default resolution order for Edge references is normative:

1. **Local document.** Object identifiers declared within the same document as the Edge are checked first. An `@bob` reference is satisfied by an `<OBJ bob>` declaration in the same document if one exists.
2. **Loaded document context.** When the Edge appears in a document being processed alongside other documents (e.g., in a batch import or registry context), Objects from those documents are checked in the order they were loaded.
3. **Globally-resolvable IRIs.** Object references expressed as full IRIs (e.g., `@https://acme.com/people/alice`) MAY be resolved by a network-aware resolver if the implementation supports it. Implementations are not required to perform network resolution.
4. **Resolution failure.** If no scope yields a target, the reference is unresolved.

When an Edge reference cannot be resolved:

- The Edge is reported as schema-non-conforming (Layer 2). The Edge itself is structurally well-formed (Layer 1); it is the schema-level claim that the reference points to an existing Object that fails.
- The error MUST identify the unresolved reference and the Edge that carries it.
- Validation of the rest of the document continues; unresolved Edge references do not halt validation.

A resolution failure means the target named by the reference does not exist in any resolvable scope. The unresolved reference does not become a placeholder, virtual Object, or tombstone in the data model — the data is exactly what was authored, and the validator's report indicates that the reference does not resolve. Applications that want placeholder behavior must construct it at the application layer.

Implementations MAY support modes where unresolved references are treated as warnings rather than errors, useful for ingesting partial data sets or for documents that intentionally reference targets in other systems. When such modes are supported, the strict mode (errors on unresolved references) SHOULD remain the default.

#### 6.5.5 Block — `BLK`

| Canonical | Minified | Resource? | Identifier |
|-----------|----------|-----------|------------|
| `BLK`     | —        | —         | required (local) |

A Block labels a region of an Object's children with a shared purpose or context. Where a Record asks "what is this composite value?", a Block asks "what context do these children belong to?". A Block is not a data type — it does not represent a structured composite that the parent *has*; it labels a region of the parent's children, asserting that they share a common purpose. BLK has no minified form.

Blocks accept the same children as Objects and may only appear under an Object or another Block. The children of a Block are addressed through the Block identifier (e.g., `/article-42/head/title`).

```
<OBJ Article42
    <BLK head
        <STR title : "Introducing GDX">
        <STR author : "Julian Mussi">
        <INT publishedYear : 2026>
    >
    <BLK body
        <REC intro>
        <REC section>
    >
>
```

A Block label says something about its children's purpose in the Object, but is not considered "data" in the sense a Record is. Pick a Record when the children together form a single conceptual value; pick a Block when they share context but do not form a composite.

#### 6.5.6 Annotation — `REM`

| Canonical | Minified | Resource value-kind |
|-----------|----------|---------------------|
| `REM`     | —        | `string`            |

A `REM` element is a leaf-only metadata note. REM elements may appear under any role and never have children. The Resource is a string carrying the annotation text. REM has no minified form.

```
<OBJ Order123
    <REM : "Imported from legacy system 2026-04-27">
    <STR customer : "ACME Corp">
    <FLOAT total : 1250.00d>
>
```

Annotations survive round-trip serialization and are part of the document, but carry no semantic weight in the data model. Tooling MAY surface them; data interpretation SHOULD NOT depend on them.

#### 6.5.7 Package — `PKG`

| Canonical | Minified | Resource? | Identifier |
|-----------|----------|-----------|------------|
| `PKG`     | —        | —         | optional   |

A Package is a transport-layer construct (§9). It wraps Objects, Templates, REL declarations, and metadata for transport between systems. Packages are not persisted as such; receivers unpack them and store the contents. PKG has no minified form.

```
<PKG color-export-2026-04-29
    <STR source : "color-system-v3">
    <OBJ Pantone-485-C ...>
    <OBJ Pantone-186-C ...>
>
```

Packages MUST NOT contain other Packages. See §7.2 rule 10 and §9 for transport semantics.

#### 6.5.8 Group — `GRP`

| Canonical | Minified | Resource? | Identifier |
|-----------|----------|-----------|------------|
| `GRP`     | —        | —         | required (local) |

A Group is a transport-layer construct (§9) that partitions a Package's contents into named subsets with optional metadata. Groups may only appear as children of Packages or other Groups. Like Packages, Groups are not designed to be persisted. GRP has no minified form.

```
<PKG color-export-2026-04-29
    <GRP pantone
        <STR libraryName : "Pantone Solid Coated">
        <OBJ Pantone-485-C ...>
    >
    <GRP ral
        <STR libraryName : "RAL Classic">
        <OBJ RAL-3020 ...>
    >
>
```

#### 6.5.9 Qualifier — `.`

The `.` modifier marks an element as a Qualifier. Qualifiers carry constraints, units, modes, confidence levels, or other metadata that refines the meaning of their parent. They contribute to data semantics and travel with the data. Qualifiers may contain Annotations only.

Qualifier is best understood by contrast with Extension (§6.5.10): a **Qualifier** modifies *instance data* (it travels with the value, contributes to interpretation, and is part of the data model); an **Extension** defines *schema rules* (it travels with the Template, contributes to validation, and is NOT part of the data model). Removing a Qualifier changes the meaning of the data; removing an Extension may relax validation but does not change instance data semantics. The `.` sigil signals "this element refines what its parent *means*"; the `_` sigil signals "this element constrains what instances *must satisfy*."

| Canonical | Minified | Resource value-kind |
|-----------|----------|---------------------|
| `.STR`    | `.S`     | `string`            |
| `.INT`    | `.I`     | `int`               |
| `.FLOAT`  | `.F`     | `float`             |
| `.BOOL`   | `.L`     | `bool`              |
| `.BYTE`   | `.B`     | `bytes`             |

```
<FLOAT temperature : 21.5
    <.STR unit : "celsius">
    <.BOOL observed : true>
>
```

#### 6.5.10 Extension — `_`

The `_` modifier marks an element as an Extension — a schema-only attachment that appears as a child of a Template (§6.5.14). Extensions are NOT part of the data model. The structural restriction — Extensions valid only under Templates — is enforced by the composition matrix (§7.1). An Extension appearing outside a Template is malformed.

See §6.5.9 for the contrast with Qualifier — both attach to elements, but Extension defines schema rules while Qualifier refines instance data.

| Canonical | Minified | Resource value-kind |
|-----------|----------|---------------------|
| `_STR`    | `_S`     | `string`            |
| `_INT`    | `_I`     | `int`               |
| `_FLOAT`  | `_F`     | `float`             |
| `_BOOL`   | `_L`     | `bool`              |
| `_BYTE`   | `_B`     | `bytes`             |

```
<DEFSTR Email
    <_BOOL required : true>
    <_STR pattern : "^[^@]+@[^@]+$">
>
```

The same value-kind may appear as a Qualifier or as an Extension depending on what is being said:

```
<FLOAT temperature : 21.5
    <.STR unit : "celsius">          ← Qualifier: this temperature is in celsius
>

<DEFFLOAT TemperatureReading
    <_STR unit : "celsius">          ← Extension: instances of this Template are constrained to celsius
>
```

The first is data ("the temperature value, expressed in celsius"); the second is schema ("any instance of TemperatureReading is required to be in celsius"). The grammar makes the distinction explicit at the prefix level.

#### 6.5.11 Embedded — `%`

The `%` modifier marks a Property, Record, or Object as eligible to appear in a property-value position — where a scalar value would otherwise sit. The embedded element retains its base role's children and identifier rules.

Embedded is the mechanism by which structured values can occupy positions normally reserved for scalars. Without `%`, a Property's value is its Resource (a typed scalar); with `%`, a Property may carry a structured value as its content.

| Canonical | Minified | Base role |
|-----------|----------|-----------|
| `%STR`    | `%S`     | Property  |
| `%INT`    | `%I`     | Property  |
| `%FLOAT`  | `%F`     | Property  |
| `%BOOL`   | `%L`     | Property  |
| `%BYTE`   | `%B`     | Property  |
| `%REC`    | `%R`     | Record    |
| `%OBJ`    | `%O`     | Object    |

```
<STR contact
    <%REC details
        <STR email : "a@example.com">
        <STR phone : "555-0100">
    >
>
```

The Embedded modifier is particularly useful when a structured value should retain its full GDX identity rather than being serialized to a string and re-parsed:

```
<REC colorReading
    <%OBJ lab
        <FLOAT l : 50.0>
        <FLOAT a : 25.3>
        <FLOAT b : -12.7>
    >
    <%OBJ rgb
        <INT r : 124>
        <INT g : 89>
        <INT b : 200>
    >
>
```

#### 6.5.12 Node — `N`

The `N` letter-modifier marks an element as establishing or participating in a Node-zone subtree — a semi-structured container for ad-hoc or schema-less material. The Node modifier composes with primitive bases or with `REC`. The `N` is a letter rather than a sigil because Node-zone elements have different composition rules than the rest of GDX (isolated subtrees — see §7.2 rule 5).

| Canonical | Minified | Carries Resource? | Resource value-kind |
|-----------|----------|-------------------|---------------------|
| `NREC`    | `NR`     | No                | —                   |
| `NSTR`    | `NS`     | Yes               | `string`            |
| `NINT`    | `NI`     | Yes               | `int`               |
| `NFLOAT`  | `NF`     | Yes               | `float`             |
| `NBOOL`   | `NL`     | Yes               | `bool`              |
| `NBYTE`   | `NB`     | Yes               | `bytes`             |

Node-zone elements are permissive in what may *contain* them — any structural role may have a Node-zone child — but restrictive in what they may *contain*: a Node-zone element's children MUST themselves be Node-zone elements (or Annotations). This isolation rule keeps validation and tooling tractable: above the Node, structured rules apply; within the Node, structure is informal.

The Node modifier always composes with a base — there is no bare Node prefix. `NREC` is the no-Resource form (Node-zone container analogous to a Record); it replaces the bare `<N>` form used in earlier versions. Earlier-version documents using bare `<N>` MUST be migrated to `<NREC>` for v0.9 conformance.

```
<NREC audit
    <NSTR source : "import">
    <NINT lineNumber : 42>
    <NREC raw
        <NSTR originalFormat : "csv">
        <NINT columnCount : 12>
    >
>
```

Use Node-zone elements sparingly. Their permissiveness in containment positions and restrictiveness on internal structure make them appropriate for material that doesn't fit GDX's structured roles — imported audit data with unknown shape, blobs of metadata from external systems, ad-hoc payloads where a Template would be premature. When the shape of formerly-Node content becomes well-understood, migrating it to structured roles is straightforward: replace the Node subtree with a Record, the Node-typed payloads with their structured equivalents, and define a Template to validate. The migration is destructive (the subtree is replaced, not augmented) and is a deliberate decision to formalize what was provisional.

#### 6.5.13 Set — `*`

The `*` suffix promotes a value-bearing primitive or Record into a multi-valued collection. A Set is a role in its own right, not a transparent overlay on its base role: Set composition rules differ from Property composition rules — a Set may contain its members; a Property cannot.

The `*` is a suffix because Set construction is structurally distinct from the refinement modifiers. A Set's value-kind is "collection of base-instances" rather than the base's single-value-kind. Reading `STR*` as "STR, plural" makes the collection semantics legible at the prefix level. See §6.3.1 for the suffix's role in the overall prefix vocabulary.

| Canonical | Minified | Member type      |
|-----------|----------|------------------|
| `STR*`    | `S*`     | string members   |
| `INT*`    | `I*`     | int members      |
| `FLOAT*`  | `F*`     | float members    |
| `BOOL*`   | `L*`     | bool members     |
| `BYTE*`   | `B*`     | bytes members    |
| `REC*`    | `R*`     | Record members   |

A Set element holds multiple members of its declared element type. Members appear in document order; the term "set" refers to the multi-valued nature of the collection, not to any unordered semantics. Member order is preserved through serialization and is significant for round-trip equivalence. Set members carry their prefix (matching the Set's element type) and MAY omit identifiers when their position and value are sufficient.

```
<STR* tags
    <STR : "blue">
    <STR : "red">
    <STR : "green">
>

<REC* contacts
    <REC alice <STR email : "a@example.com">>
    <REC bob   <STR email : "b@example.com">>
>
```

##### 6.5.13.1 Empty and Absent States for Sets

Sets have two distinguishable states (in contrast to Properties, which have three per §4.3.3). The simpler taxonomy reflects the fact that a Set's content is determined by its child members rather than by a Resource value:

- **Absent.** The Set element does not appear in the parent at all. The collection is undeclared; consumers cannot determine whether members exist or not.
- **Present.** The Set element appears in the parent, with zero or more member children. The collection is declared to exist; the members are exactly what is present.

A Present Set with zero members — `<STR* tags>` — is an **empty Set**. It is semantically distinct from an Absent Set: the empty form asserts "this collection exists and contains no members"; absence asserts nothing about the collection.

This distinction is significant for database mapping, query semantics, and API design. A `Customer` Object with `<STR* tags>` (no children) and a `Customer` Object with no `tags` element at all are not equivalent. The first declares an empty tag list (which can be serialized, stored, queried as "tag count = 0"); the second declares nothing about tags (which may indicate the field doesn't apply, hasn't been computed, or is unknown).

There is no third "null Set" state analogous to a null Property. Sets do not carry a Resource value of their own — their content is fully expressed through their children — so the present-but-null state has no meaningful encoding for Sets.

Templates that declare a Set property MAY use the `gdx:required` and `gdx:optional` extensions (Common Vocabulary §5.1) to constrain whether the Set element must be present:

- **Required Set.** The Set element MUST appear; it MAY be empty.
- **Optional Set.** The Set element MAY be absent; if present, it MAY be empty.

When emptiness itself is meaningful for the application, the Template MAY use `gdx:minItems` to require at least one member:

- **Required non-empty Set.** The Set element MUST appear and MUST contain at least one member.

#### 6.5.14 Template — `DEF`

The `DEF` prefix-fragment marks an element as a Template definition — a type contract rather than an instance (§8). A Template MUST carry an identifier (the type name) and follows the composition rules of its base role. Templates are the only role that may carry Extension children. Template prefixes are canonical-only; they have no minified form.

| Canonical  | Base role |
|------------|-----------|
| `DEFOBJ`   | Object    |
| `DEFREC`   | Record    |
| `DEFSTR`   | Property  |
| `DEFINT`   | Property  |
| `DEFFLOAT` | Property  |
| `DEFBOOL`  | Property  |
| `DEFBYTE`  | Property  |

Property Templates (`DEFSTR`, `DEFINT`, `DEFFLOAT`, `DEFBOOL`, `DEFBYTE`) typically declare semantic types or representation refinements on top of a base value-kind. A Template does not change the underlying value-kind; it adds interpretation or validation rules that consumers may apply.

A Template's *fields* are described using regular prefixes (not `DEF…` prefixes). The DEF prefix appears only at the Template's declaration root; the children are field descriptions using the standard structural and value prefixes.

```
<DEFREC com.acme.Address
    <STR street>
    <INT number>
>

<DEFSTR schema:csv
    <_STR separator : ",">
    <_BOOL hasHeader : true>
>

<DEFINT gdx:i8
    <_INT gdx:min : -128>
    <_INT gdx:max : 127>
>
```

#### 6.5.15 Relation Declaration — `REL`

| Canonical | Minified | Resource? | Identifier |
|-----------|----------|-----------|------------|
| `REL`     | —        | —         | required (predicate identifier) |

A `REL` element declares a predicate as a first-class entity. Predicates declared via REL are referenced from Edges (and other relation-bearing positions) through the `#rel` field. REL has no minified form.

The fields a REL carries — domain, range, inverse, symmetry, cardinality, and other semantic properties — are determined by the vocabulary the predicate participates in. The core specification defines REL only as a structural role; the predicate semantics live in vocabulary specifications. The GDX Common Vocabulary defines a base set of predicate refinement extensions. Applications may use additional vocabulary-defined extensions or define their own.

```
<REL gdx:knows>

<REL gdx:manages
    <@REL #gdx:inverse : @gdx:reportsTo>
>
```

A REL declaration MAY appear at the document root (alongside or instead of a primary Object) or within a vocabulary document. Resolution of `#rel` references follows the same mechanism as Type reference resolution (§8.5).

#### 6.5.16 References — `@KIND`

The `@` operator (§6.3.2) composes with any base or modifier-base composition to produce a typed reference. The resulting element is a leaf-like typed pointer whose Resource is an identifier naming the target.

**Element shape.** A `@KIND` element MAY carry an identifier (the name of this reference within its parent's scope), MAY carry a `#rel` field (a predicate naming the link's meaning, useful for vocabulary metadata), and carries its target as a reference literal in Resource position. Children, if any, are limited to Qualifiers and Annotations.

```
<@OBJ employer : @com.acme>                       -- a typed pointer to an Object
<@REL #gdx:inverse : @gdx:reportsTo>              -- vocabulary metadata link
<@DEFOBJ #conformsTo : @Customer>                 -- pointer to a Template
<@STR* allowedTags : @global-tag-set>             -- pointer to a Set of strings

<@OBJ employer : @com.acme
    <.STR confidence : "verified">                -- Qualifier refining the reference
>
```

**Universal composition.** The `@` operator composes with any role base (or modifier-base composition) except another `@KIND`. The closed prefix pattern is:

```
@ [refinement-modifier?] [base] [*?]
```

This produces references like `@OBJ`, `@REL`, `@DEFSTR`, `@.STR`, `@_BOOL`, `@%REC`, `@NSTR`, `@STR*`, `@DEFOBJ*`, `@.STR*`, and so on. The composition reads as "a reference to a [KIND] element," where KIND is the rest of the prefix interpreted as a role.

**The `@@` prohibition.** A reference cannot target another reference. `@@STR`, `@@OBJ`, `@@DEFOBJ`, and similar compositions are malformed. References resolve to addressable elements directly, not to other references.

**Resolution.** A reference's Resource is a reference literal (`@identifier`) naming the target. The resolver uses the target-kind declared by the prefix to scope the search: a `@REL` reference resolves against the REL declaration scope; a `@DEFOBJ` reference resolves against the Template scope; a `@OBJ` reference resolves against the Object scope. The resolution algorithm parallels Edge Reference Resolution (§6.5.4.1) but is scoped by target-kind.

Identifier-based reference resolution (`@id`) is normative in this specification. Path-based reference resolution (`$path`, `@anchor$path`) and key-based reference resolution (`&key`, `@anchor&key`) are specified in the companion GDX Path Expressions Specification (`GDX-Path.md`). The composed forms `@Anchor$PathExpression` and `@Anchor&Key` express cross-document addressing: the anchor portion resolves a starting element by scope walk, and the path or key portion locates the target within that element's scope.

The `Anchor` production carries an optional role marker (`@OBJ:id`, `@REL:id`, `@DEFOBJ:id`, etc.) that disambiguates the role of the resolved element. When the marker is present, the scope walk is restricted to elements of the claimed role. When the marker is absent, the role is inferred from the enclosing `@KIND` reference element's prefix (for plain `@id` references) or defaults to `OBJ` (for composed references). The resolver MUST verify that the resolved element actually has the inferred or claimed role; mismatch yields an unresolved reference.

A reference whose Resource is any non-`IdRef` literal MUST resolve to exactly one element of the matching role (see `GDX-Path.md` §7).

**Distinguishing `@OBJ` from EDGE.** Both `@OBJ` and EDGE carry an Object reference, but they are structurally distinct. An `@OBJ` element is a leaf-like typed pointer — it identifies a target without having its own substructure beyond Qualifiers. An EDGE element is a graph-statement element with its own children, including Properties that express edge metadata (weight, confidence, timestamp). Use `@OBJ` when the link is just an identifier; use EDGE when the link itself is a structural participant in the graph with attributes of its own. The full comparison is in §3.6.

**Reflexivity.** The `@` operator gives GDX a referential dimension of self-description: any addressable element can be the target of a reference from any other element. Vocabulary documents can declare relationships between predicates; configuration documents can declare typed pointers to Objects; Templates can declare version-chains to predecessor Templates. The format is self-describing not only structurally (the schema is GDX, transport is GDX) but referentially (elements can refer to other elements by typed identifier).

**Use cases.** Common patterns include:

- **Vocabulary metadata** — RELs referring to other RELs (inverse, sameAs, subPropertyOf) and to Templates (domain, range): `<@REL #gdx:inverse : @gdx:reportsTo>`
- **Typed configuration** — Objects referring to other Objects by role: `<@OBJ database : @primary-db>`, `<@OBJ fallback : @secondary-db>`
- **Schema cross-references** — Templates referring to other Templates (version chains, supersedes-by): `<@DEFOBJ #supersededBy : @CustomerV2>`
- **Registry patterns** — Sets of typed references: `<@OBJ* acceptedInputs <@OBJ : @type-a> <@OBJ : @type-b>>`

---

## 7. Composition Axioms

GDX nesting is governed by a single matrix indexed by **role**. The table below is the normative source: any prefix is admitted as a child if its role appears in the parent role's row.

Modifier-bearing and operator-bearing prefixes are validated as their base role with two refinements:

- **Refinement modifiers** (`.`, `_`, `%`, `N`) and the **`DEF` declaration construct** do not change which roles may contain or be contained by the element, with two exceptions: Extensions (`_X`) may only appear under Templates (rule 7), and Node-zone elements (`NX`) have isolated-subtree semantics (rule 5).

- **The `*` suffix** (Set) and **the `@` operator** (Reference) produce roles whose composition rules are distinct from their bases. A Set follows the Set row; a Reference follows the Reference row.

### 7.1 Allowed Children Matrix

| Parent ↓ \ Child →  | Object | Record | Block | Property | Set | Node | Edge | Reference | Qualifier | Extension | Annotation | REL | Group |
|---------------------|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| **Object**          |   | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |   | ✓ |   |   |
| **Record**          |   | ✓ |   | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |   | ✓ |   |   |
| **Block**           |   | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |   | ✓ |   |   |
| **Property**        |   |   |   |   |   | ✓ |   |   | ✓ |   | ✓ |   |   |
| **Set**             |   |   |   | ✓ |   | ✓ |   |   | ✓ |   | ✓ |   |   |
| **Node**            |   |   |   |   |   | ✓ |   |   |   |   | ✓ |   |   |
| **Edge**            |   | ✓ |   | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |   | ✓ |   |   |
| **Reference**       |   |   |   |   |   |   |   |   | ✓ |   | ✓ |   |   |
| **Qualifier**       |   |   |   |   |   |   |   |   |   |   | ✓ |   |   |
| **Extension**       |   |   |   |   |   |   |   |   |   |   | ✓ |   |   |
| **Annotation**      |   |   |   |   |   |   |   |   |   |   |   |   |   |
| **Template**        |   | ✓ |   | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |   |   |
| **REL**             |   |   |   |   |   |   |   | ✓ |   | ✓ | ✓ |   |   |
| **Package**         | ✓ | ✓ |   | ✓ | ✓ |   |   |   |   |   | ✓ | ✓ | ✓ |
| **Group**           | ✓ | ✓ |   | ✓ | ✓ |   |   |   |   |   | ✓ | ✓ | ✓ |
| **Document root**   | ✓ |   |   |   |   |   |   |   |   |   |   | ✓ |   |

The Template row covers all `DEF…` prefixes, validated by their base role's row except where Extensions are admitted (Templates are the only parent role that admits Extension children). Templates also admit Reference children, which support cross-references to other Templates (e.g., for version chains or supersedes-by relationships).

The Reference row covers all `@…` prefixes. References are leaf-like: they admit only Qualifiers and Annotations as children. They do NOT carry Properties, Edges, Sets, or other structural children of their own. The element identifies its target via the Resource (a reference literal); the target lives elsewhere and is reached through resolution.

The REL row admits Extensions (for predicate semantics: domain, range, inverse, symmetry) and References (for vocabulary metadata such as `@REL` pointing to inverse predicates or `@DEFOBJ` pointing to domain/range Templates).

The Package row admits Templates (any `DEF…` prefix) and REL declarations as payload along with Objects. For clarity: a Package may directly contain Object instances (`OBJ`/`O`), Template definitions (`DEF…`), REL declarations (`REL`), Properties (metadata), Records (metadata), Sets (metadata), Annotations, and Groups for transport-internal organization. A Group admits the same children as a Package except for Package itself.

The Document root row covers what may appear as the root of a GDX document: an Object, a Template, a REL, or a Package. A document MAY contain multiple top-level elements when each is a recognized root-eligible type.

### 7.2 Derived Rules

The following rules are consequences of the matrix and are repeated here for clarity:

1. **Root rule.** A document root MUST be an Object, a Template, a REL declaration, or a Package. A document MAY contain multiple top-level elements; each top-level element MUST independently be a root-eligible type.

2. **Object exclusion.** No role other than the Embedded modifier (`%O`, §6.5.11) and the transport-layer roles (Package, Group) may contain another Object. The `%O` form expresses Object-as-property-value within data-layer composition; Package and Group contain Objects as transport-layer payload, not as ownership relationships.

3. **Block containment.** A Block may only appear under an Object or another Block. Blocks do not appear in transport-layer roles.

4. **Annotations are leaves.** Annotations never have children and are permitted under every role except other Annotations.

5. **Nodes are isolated subtrees.** A Node-zone element's children MUST themselves be Node-zone elements (or Annotations). Once a Node-zone element appears in a document, its subtree is uniformly Node-typed. Structured roles (Properties, Records, Sets, Edges, etc.) cannot appear within a Node subtree. Use `NREC` for the no-Resource Node-zone container that establishes such a subtree.

6. **Sets contain members of their declared element type.** A Set's children MUST be elements whose prefix matches the Set's element type. A `STR*` Set contains `STR` members; a `REC*` Set contains `REC` members. Qualifiers, Extensions (where permitted), and Annotations may also appear as Set children to refine or annotate the Set itself.

7. **Extensions are schema-only.** Extensions MAY appear only under Template (`DEF…`) elements. An Extension under any other parent role is malformed.

8. **REL declares predicates.** A REL element may carry Extension children that declare predicate semantics (domain, range, inverse, symmetry, cardinality, etc.). The vocabulary of those Extensions is defined by the GDX Common Vocabulary or by application vocabularies.

9. **Modifier transparency.** Embedded (`%X`), Template (`DEF…`), Qualifier (`.X`), Extension (`_X`), and Node (`NX`) modifier-bearing prefixes are validated as their base role for composition purposes. The modifier never changes what is allowed inside or what may contain the element, with two exceptions: Extensions are restricted to Template parents (rule 7), and Node-zone elements have isolated-subtree semantics (rule 5).

10. **Transport non-nesting.** Packages MUST NOT contain other Packages. Groups MUST NOT contain Packages. Transport-layer roles form a controlled structure: a Package may contain Groups and data-layer items; a Group may contain other Groups and data-layer items.

11. **Transport non-persistence.** Package and Group elements are not persisted as such by storage layers. Their contents (Objects, Templates, REL declarations) are extracted and stored individually; transport-layer metadata MAY be retained as provenance information but the transport wrapping is discarded. See §9 and §10.4.

12. **Refinement modifiers do not stack.** An element carries at most one refinement modifier (`.`, `_`, `%`, `N`) or the `DEF` construct. The following are NOT valid prefixes:
    - `*%STR` — Set notation is now a suffix (see rule 13); the form would be `%STR*` if it were valid, but `%` and `*` may combine
    - `.NSTR` — Qualifier of a Node-zone element
    - `_%STR` — Extension of an Embedded element
    - `DEFN.STR` — DEF-on-modifier-on-modifier

    The reading pattern for any prefix is `[@?][refinement-modifier?][base][*?]`. At most one of `.`, `_`, `%`, `N`, `DEF` may appear in the prefix-position-element slot; the slot is filled by at most one of these.

    Where multi-valued semantics on an embedded or template-defined value are needed, the structure is expressed through normal nesting:

    ```
    <REC* tagSets
        <REC group1
            <STR* tags
                <STR : "red">
                <STR : "blue">
            >
        >
        <REC group2
            <STR* tags
                <STR : "green">
            >
        >
    >
    ```

    This restriction keeps the prefix vocabulary closed and bounded. The combinations expressible through nested structure are sufficient for all use cases the specification anticipates.

13. **Set is a suffix.** The `*` Set marker is a suffix on the base prefix: `STR*`, `INT*`, `REC*`, etc. The `*` suffix MAY combine with refinement modifiers (`.STR*`, `_INT*`, `%REC*`, `NSTR*`) and with the `@` reference operator (`@STR*`, `@.STR*`, `@DEFOBJ*`). The `*` suffix does not compose with `DEF` (Templates are not collected as a set in normal usage).

14. **Reference operator is universal.** The `@` operator MAY prefix any base or refinement-modifier-bearing base to produce a typed reference. The `@KIND` composition reads as "a reference to a KIND-element." The `@` operator MAY combine with the `*` suffix: `@KIND*` is a reference to a Set of KIND-elements.

15. **No reference-to-reference.** The `@` operator MUST NOT compose with another `@KIND` prefix. `@@STR`, `@@OBJ`, and similar compositions are malformed. References resolve to addressable elements directly, not to other references.

16. **References are leaf-like.** A `@KIND` element MAY carry Qualifiers and Annotations as children but MUST NOT carry Properties, Edges, Sets, or other structural children. The reference's content is its target identifier, not a substructure. This distinguishes References from Edges: EDGE elements are structured graph-statement elements with their own Properties; `@OBJ` elements are leaf-like typed pointers. See §3.6 and §6.5.4.

### 7.3 Identifier Rules

Per §4.2, the following roles MUST carry at least one identifier (`Id`, `Key`, or `Rel`) when in named-attribute position:

- Object (`OBJ`/`O`, `DEFOBJ`)
- Record (`REC`/`R`, `%REC`/`%R`, `DEFREC`, `NREC`/`NR`) when not in Set member position
- Property (`STR`/`S`, `INT`/`I`, `FLOAT`/`F`, `BOOL`/`L`, `BYTE`/`B` and their variants) when not in Set member position
- Template (any `DEF…`)
- REL (`REL`)
- Block (`BLK`) — Blocks always require an identifier; the identifier names the partition
- Group (`GRP`) — Groups always require an identifier; the identifier names the partition within a Package

Set members are exempt from the identifier requirement but MUST carry their prefix.

Package (`PKG`) carries an optional identifier. When present, the identifier is used for transport correlation (request/response matching, deduplication, audit) and is local to the transport context.

Other roles MAY carry identifiers but do not require them.

### 7.4 Resource Rules

Per §4.3, when a Resource is present, its value-kind MUST be consistent with the element's prefix family:

- String-family prefixes (`STR`/`S`, `STR*`/`S*`, `%STR`/`%S`, `.STR`/`.S`, `_STR`/`_S`, `NSTR`/`NS`) carry `string` resources.
- Int-family prefixes (`INT`/`I`, `INT*`/`I*`, `%INT`/`%I`, `.INT`/`.I`, `_INT`/`_I`, `NINT`/`NI`) carry `int` resources.
- Float-family prefixes (`FLOAT`/`F`, `FLOAT*`/`F*`, `%FLOAT`/`%F`, `.FLOAT`/`.F`, `_FLOAT`/`_F`, `NFLOAT`/`NF`) carry `float` resources.
- Bool-family prefixes (`BOOL`/`L`, `BOOL*`/`L*`, `%BOOL`/`%L`, `.BOOL`/`.L`, `_BOOL`/`_L`, `NBOOL`/`NL`) carry `bool` resources.
- Bytes-family prefixes (`BYTE`/`B`, `BYTE*`/`B*`, `%BYTE`/`%B`, `.BYTE`/`.B`, `_BYTE`/`_B`, `NBYTE`/`NB`) carry `bytes` resources.
- Edge prefixes (`EDGE`/`E`) carry `reference` resources.
- Reference prefixes (`@KIND` family) carry `reference` resources. The Resource is a reference literal whose target-kind is declared by the prefix (see §6.5.16).
- Annotation prefix (`REM`) carries `string` resources.

The Resource itself is optional on Resource-bearing roles; an element with no Resource represents a null value (instance context) or a slot declaration (Template context). See §4.3.3.

Object, Block, Record, Node-Record (`NREC`/`NR`), Set, Template (all `DEF…`), Relation Declaration (`REL`), Package, and Group elements do not carry Resources at the element level. Their content is conveyed through their children. A Template (`DEFSTR`, `DEFREC`, etc.) declares a contract for instances; it does not carry an instance value itself.

### 7.5 Validation

A conforming parser MUST reject any document containing:

- An element whose prefix is not in the closed vocabulary (§6.2).
- A child element whose role is not admitted by the parent role's row in §7.1.
- A required identifier that is missing (§7.3).
- A Resource whose value-kind does not match the prefix family (§7.4).
- An Extension appearing outside a Template parent.
- A Node containing a non-Node, non-Annotation child.
- A Set containing a member whose prefix does not match the Set's element type.
- An Object appearing as a non-root child in a position other than `%O` Embedded form, a Package payload, or a Group payload.
- A Package containing another Package.
- A Group containing a Package.
- A Block at the document root or in any position other than as a child of an Object or another Block.
- A Block missing its required identifier.
- A Group missing its required identifier.

A conforming parser MUST accept any element with an identifier but no Resource section. Such elements are well-formed and represent null values (instance context) or slot declarations (Template context).

The above are the *primitive axioms* of GDX validity — Layer 1 (§2.2). They are owned by GDX, evaluated at parse time, and absolute. Constraint validity beyond these axioms (numeric bounds, string patterns, regex matches, semantic types) is vocabulary-defined and evaluated externally; GDX hosts those constraints as Extension elements but does not interpret them. See §8 for the schema-layer validation model (Layer 2a — Template structural conformance) and Constraint Extensions (Layer 2b — vocabulary-defined constraint conformance).

---

## 8. Schemas and Validation (Templates)

> **Status:** Sections 9.1 through 9.5 are stable. Section 9.6 describes mechanisms that are committed in principle but require companion specifications for full normative treatment.

Schemas in GDX are themselves GDX documents. This section defines the schema mechanism in three layers, mirroring the conformance model of §2.2.

- **Template structure (§8.1, §8.3).** Templates are GDX elements with `DEF…` prefixes. They declare the structural shape an instance is expected to satisfy: which child slots exist, what role each carries, what Rel each aligns to, what Type each claims. Templates are GDX-native: this specification defines their grammar, their composition rules, and how they are resolved.

- **Template-instance alignment (§8.2).** An instance carrying a `--type` reference asserts that its structure aligns to the referenced Template. Slot alignment is by `Id` with `Rel` as a cross-vocabulary bridge; the universal claim rule (§8.2.3) governs how Template-declared and instance-declared values reconcile on aligned slots; materialization (§8.2.4) is the recommended persistence form. This is Layer 2a (§2.2).

- **Constraint Extensions (§8.4).** Constraint vocabularies — XSD, JSON Schema, OWL profiles, bespoke domain vocabularies — are expressed as Extension elements on Template slots. The Extension carriers (`_STR`, `_INT`, etc.) are GDX-native. The constraint identifiers they carry (`xsd:minInclusive`, `json:pattern`, etc.) are vocabulary-defined and opaque to GDX. This is Layer 2b (§2.2). Multiple constraint vocabularies MAY coexist on the same Template; each is interpreted independently by consumers that understand its identifiers.

This layered model is what makes GDX a structural schema with pluggable constraint vocabularies (§1.2). The structural surface — what Templates declare, how slots align, how Extensions attach — is fixed by this specification. The constraint surface is open: any vocabulary may define its own constraint identifiers and ride atop the GDX carrier mechanism without negotiating with GDX or with other vocabularies.

### 8.1 Templates

A Template is an element with a `TD…` prefix. It declares an *interface* — a set of properties, records, edges, and constraints — that an instance element claims to satisfy via its `--type` field.

```
<DEFREC com.acme.Address
    <STR street>
    <INT number>
    <STR country
        <_BOOL optional : true>
    >
>
```

A Template's children describe the *expected shape* of conforming instances. By default, every declared child is required. Optional fields are marked with the `_L optional` Extension (or other vocabulary-defined optionality markers).

Templates are themselves GDX elements. They use the same composition rules, the same identifier grammar, and the same syntactic conventions as data elements. The schema is data. Tools that read GDX read Templates the same way they read instances; tools that write GDX write Templates the same way they write instances. There is no separate schema language.

A Template MUST carry an identifier — the Template's name. The identifier follows the same lexical rules as any other identifier (§5.4) and typically takes a CURIE form (`gdx:Address`, `com.acme.Person`) for cross-vocabulary clarity.

### 8.2 Conformance via `--type`

An instance element references a Template by name in its `--type` field. The validator resolves the reference (§8.5) and checks that the instance's children satisfy the Template's declared shape.

```
<REC address --com.acme.Address
    <STR street : "Main St">
    <INT number : 100>
>
```

The `--type` reference is a Layer 2 (schema) assertion (§2.2). Resolving the reference and validating the instance against the Template are operations performed after parsing and structural validation. An instance may be representation-conforming (Layer 1) without being schema-conforming (Layer 2): the structure is well-formed, but the contents may not satisfy the claimed Template.

**Type identity is named, not structural.** An instance carrying `--com.acme.Customer` asserts that it is of type `com.acme.Customer`. The type's identity is the Template name; two Templates that happen to declare the same shape are nonetheless distinct types, and an instance of one is not silently an instance of the other.

**Conformance is a separately observable relation.** An instance of type A MAY structurally conform to a Template B — its children align to B's slots, its values satisfy B's declared metadata — without B being its type. This cross-type conformance is observable and useful: validators can check an instance against multiple schemas, versioning tools can detect that `acme:swatch.v1` conforms to `acme:swatch.v2`, migration tooling can identify candidate transformations between related types. But conformance does not redefine identity: the `--type` claim is what travels with the data. Consumers MUST NOT silently substitute one type for another even when their instances mutually conform; the original type claim is preserved and any cross-conformance is surfaced as a derived observation, not a rewrite.

#### 8.2.1 Slot Alignment

An instance child satisfies a Template slot by matching the slot's `Id` field, using case-sensitive ordinal comparison (§4.2). When a Template slot declares a `Rel`, an instance child carrying the same `Rel` MAY align to the slot even if its local `Id` differs; in that case the `Rel` is the alignment key, supporting cross-vocabulary instances whose local names differ from the Template's slot names.

The `Key` field plays no role in Template-instance alignment. Keys are an independent named-anchor mechanism (§3.2, §5.4.5) used by `KeyRef` and `IdKeyRef` reference literals; they are not consulted during schema validation.

#### 8.2.2 Cross-Role and Transport Restrictions

A `--type` reference MUST resolve to a Template whose base role matches the instance's role:

- An `STR`/`S` instance may reference `DEFSTR` only.
- An `INT`/`I` instance may reference `DEFINT` only.
- A `FLOAT`/`F` instance may reference `DEFFLOAT` only.
- A `BOOL`/`L` instance may reference `DEFBOOL` only.
- A `BYTE`/`B` instance may reference `DEFBYTE` only.
- A `REC`/`R` instance may reference `DEFREC` only.
- An `OBJ`/`O` instance may reference `DEFOBJ` only.

Cross-role references (e.g., a Record instance referencing an Object Template) are schema-conformance failures.

Transport-layer roles (Package, Group) are not subjects of Templates in the core specification. Their structure is defined entirely by §7 composition rules and §9 transport semantics. Vocabularies MAY define Templates for transport-construct validation purposes, but support for such Templates is a vocabulary concern, not a core requirement.

#### 8.2.3 The Universal Claim Rule

A Template MAY declare values for fields that the instance can also carry: the `Rel` field, the `Key` field, the `--type` field (in nested slots), and a default `Resource` value. When both the Template and the instance carry a value for the same field on the same aligned slot, the values are reconciled by a uniform rule that applies independently to each claim-bearing field.

| Template state | Instance state | Effective value | Disposition |
|---|---|---|---|
| silent | silent | none | No claim on this field for this slot. |
| silent | claimed | instance | The instance freely supplies a value; the Template imposes no constraint. |
| claimed | silent | Template | The Template's value is the effective value, implicitly carried by the instance. |
| claimed | claimed (matching) | confirmed | Both sides agree. The claim is affirmed. |
| claimed | claimed (differing) | — | **Conflict.** The instance is Layer 2a non-conforming. |

The rule applies independently to each field. An instance MAY satisfy the rule for `Rel` (case 3 — Template-supplied) while supplying its own value for `Key` (case 2 — free instance claim), with both outcomes being valid.

The differing-claims case (the conflict row) MUST be reported as a Layer 2a non-conformance. Implementations MUST NOT silently prefer one side over the other.

**Scope of "default."** Templates declaring a default `Resource` value declare the value as a fallback: when the instance is silent, the default applies; when the instance carries a value, the instance's value is the effective value. This is case 3 of the rule. A future revision MAY define a "fixed" claim scope, in which a Template's value is required (and a differing instance is a conflict) rather than a default; the distinction is one of claim scope, but the four-case matrix above is unchanged.

#### 8.2.4 Materialization

Per §8.2.3 case 3, a Template-supplied value is an implicit claim of the instance. The value is logically present, but it is not visibly written in the instance's text. An instance read in isolation — without resolving the Template — does not show that claim.

This creates a real difference between the instance's logical content and its persisted form. A query "find all elements with Rel `schema:givenName`" returns the instance only if the resolver has loaded the Template. Without the Template, the same data is invisible to the query. The instance's identity drifts depending on whether the schema is available.

**Materialization** is the build-time operation that resolves §8.2.3 case 3 in the persisted form: it writes the Template-supplied values into the instance as explicit fields. The materialized instance carries its full identity without needing the Template at read time.

```
# Template:
<DEFREC com.acme.Customer
    <STR firstName #schema:givenName>
    <STR locale : "en-US">
>

# Authored instance (Template values are implicit):
<REC alice --com.acme.Customer
    <STR firstName : "Alice">
>

# Materialized instance (Template values are explicit):
<REC alice --com.acme.Customer
    <STR firstName #schema:givenName : "Alice">    # Rel materialized from Template
    <STR locale : "en-US">                          # default Resource materialized
>
```

Materialization applies to the claim-bearing fields named in §8.2.3: `Rel`, `Key`, `Resource` defaults, and nested `--type` claims. It does NOT extrapolate beyond what the Template explicitly declares; missing slots are not added, and Constraint Extensions (§8.4) are not copied from Template to instance (constraints are vocabulary-defined and remain Template-scoped).

**Materialization is RECOMMENDED for persistence and exchange.** Documents that will be persisted, transmitted, archived, or signed SHOULD be materialized before they are written. This preserves the search and self-description properties of the data when the Template is not accessible at read time. Materialization is OPTIONAL for in-memory processing; tools MAY apply the §8.2.3 claim rule on-the-fly during processing without rewriting the document.

A materialized instance is semantically equivalent to its non-materialized form: per §8.2.3 case 4, every materialized field's value matches the Template's claim, producing affirmed conformance. A receiver of a materialized document MUST treat it as equivalent to the corresponding non-materialized document.

### 8.3 Property Templates and Semantic Types

Property Templates (`DEFSTR`, `DEFINT`, `DEFFLOAT`, `DEFBOOL`, `DEFBYTE`) declare semantic types or representation refinements on top of a base value-kind. A Property Template does not change the underlying value-kind; it carries vocabulary-defined constraint and interpretation rules that consumers process externally per §8.4.

A typical Property Template uses Extensions to express constraints:

```
<DEFSTR gdx:email
    <_STR gdx:pattern : "^[^@]+@[^@]+$">
    <_INT gdx:maxLength : 254>
>

<STR contact : "alice@example.com" --gdx:email>
```

The same pattern serves several common needs: width-bounded numerics (`<DEFINT gdx:i8>` with min/max constraints), format-bounded strings (patterns, length limits), semantic interpretation (a `gdx:csv` Template signaling structured string content), and domain typing (a `gdx:uuid` Template constraining a string to UUID format). The constraint vocabulary on the Template determines what each Template means.

The interpretation rules are carried by Extensions on the Template. The Extension vocabulary is application-extensible; the GDX Common Vocabulary defines a base set (§8.6).

The specific identifiers used in examples (`gdx:email`, `gdx:i8`, `gdx:csv`, `gdx:uuid`) are illustrative. Whether these specific Templates exist, and what their constraints are, is a vocabulary concern, not a core specification concern. The patterns are general; their realization in any particular vocabulary is documented by that vocabulary.

### 8.4 Constraint Extensions

Extensions on Template properties carry vocabulary-defined constraints that instances may be evaluated against. The structural carriers — the closed set of Extension prefixes (`_STR`/`_S`, `_INT`/`_I`, `_FLOAT`/`_F`, `_BOOL`/`_L`, `_BYTE`/`_B`) — are defined by this specification (§6.5.10). The *vocabulary* of constraint identifiers (such as `xsd:minInclusive`, `json:pattern`, `gdx:maxLength`, or any application-defined equivalents) is defined by external vocabularies, not by this core specification.

**GDX records but does not interpret.** Constraint identifiers are extension-layer content (§2.1): the core records them as well-formed elements but assigns no meaning. The parser validates that each Constraint Extension is structurally well-formed (correct carrier prefix, valid value-kind, attached at a position where Extensions are admitted per §7.1). It does not interpret the identifier itself. Implementations MUST NOT interpret constraint semantics at the core layer; meaning is supplied by the vocabulary that defines the identifier, evaluated by a consumer that understands that vocabulary.

This discipline is structural: by holding constraint semantics out of the core, GDX ensures that no constraint-semantic conflict can originate at the core layer. Constraint vocabularies (XSD, JSON Schema, OWL profiles, bespoke domain vocabularies) own their respective semantics independently and evolve without coordination with GDX or with each other.

The general form of a Constraint Extension on a Template slot is:

```
<DEFX TemplateName
    <X slotName
        <_X constraintIdentifier : constraintValue>
    >
>
```

Where `_X` is one of the Extension carrier prefixes (`_STR`/`_S`, `_INT`/`_I`, `_FLOAT`/`_F`, `_BOOL`/`_L`, `_BYTE`/`_B`) and `constraintIdentifier` is drawn from a constraint vocabulary.

#### 8.4.1 Parallel Constraint Vocabularies

Because constraint identifiers are opaque to GDX, multiple constraint vocabularies MAY coexist on the same Template. A single Template MAY carry equivalent or complementary constraints expressed in different vocabularies; each is evaluated independently by consumers that understand its identifiers.

A worked example showing XSD, JSON Schema, and a bespoke vendor vocabulary on the same Template:

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

Seven Constraint Extensions; three vocabularies. The core validates that all seven are well-formed Extension carriers attached to a Property Template. It assigns no meaning to `xsd:pattern`, `json:minLength`, or `acme:semantic-class`. Three classes of consumer process the Template:

- An **XSD-aware consumer** reads the `xsd:` identifiers and applies XSD validation semantics. It ignores the `json:` and `acme:` identifiers.
- A **JSON-Schema-aware consumer** reads the `json:` identifiers and applies JSON-Schema semantics. It ignores the others.
- An **`acme:`-vocabulary-aware consumer** reads `acme:semantic-class` to drive its domain-specific behavior.

None conflicts; all coexist; the core hosts all three without knowing what any of them mean. The Template is one document; the consumers each read their own slice of it.

**This pluralism is a direct consequence of GDX holding constraint semantics out of the core.** If core validation extended even to a primitive like `min` or `pattern`, the core would have its own constraint opinion competing with the extension-layer vocabularies, and parallel-vocabulary coexistence would break — a core `min` and an `xsd:minInclusive` might disagree, raising the question of which wins. By keeping core validation at only the primitive axioms (Layer 1, §2.2) and treating all constraints as extension-layer content (Layer 2b), the core never competes with any vocabulary. The discipline is what enables the pluralism.

#### 8.4.2 Validator Behavior

A validator processing a Template applies the Constraint Extensions it recognizes and ignores those it does not:

- If the validator recognizes a constraint and the instance satisfies it, validation continues.
- If the validator recognizes a constraint and the instance fails it, the instance is reported as Layer 2b non-conforming with respect to that vocabulary.
- If the validator does not recognize a constraint identifier, the validator MAY emit a warning (to indicate that a constraint was present but not checked) and MUST NOT fail validation on the unrecognized identifier alone.

This permissive behavior on unrecognized identifiers allows vocabularies to evolve without breaking implementations that have not been updated, and allows multiple vocabularies to coexist as described above. An implementation that supports vocabulary version N reads documents using vocabulary version N+1 successfully, treating unknown identifiers as advisory. An implementation that supports only XSD reads a document carrying both XSD and JSON-Schema constraints successfully, applying XSD semantics and warning (or silently ignoring, per the implementation's mode — §8.5.1) on the JSON-Schema portion.

An implementation MAY claim conformance to one or more constraint vocabularies. A vocabulary-conforming implementation honors that vocabulary's defined constraints according to the vocabulary's specification. The core specification does not require any particular constraint vocabulary to be supported; implementations choose which vocabularies they speak.

### 8.5 Conformance Modes, Type Resolution, and Error Reporting

#### 8.5.1 Conformance Modes

Validators MAY operate in **strict** mode or **lenient** mode:

- **Strict mode** (default): Every declared Template child MUST be present and conforming on the instance. Undeclared children on the instance are an error. Unrecognized extensions produce warnings.
- **Lenient mode**: Missing optional fields produce warnings. Undeclared children on the instance are accepted. Unrecognized extensions are silently ignored.

Implementations declare their default mode and provide configuration to override it.

#### 8.5.2 Type Resolution

A `--type` reference is resolved to a Template by walking a defined sequence of resolution scopes. The default resolution order is normative:

1. **Local document.** Templates declared within the same document as the reference are checked first. A `--Person` reference is satisfied by a `<DEFREC Person>` declaration in the same document if one exists.
2. **Loaded vocabularies.** Templates from explicitly-loaded vocabularies are checked next, in the order the vocabularies were loaded. CURIE prefixes (`gdx:`, `vocab:`) expand to their declared IRI bases via the resolver's vocabulary mapping table.
3. **Globally-resolvable IRIs.** Templates referenced by full IRI (e.g., `--https://acme.com/vocab/Person`) MAY be resolved by a network-aware resolver if the implementation supports it. Implementations are not required to perform network resolution.
4. **Resolution failure.** If no scope yields a Template, the reference is unresolved.

Implementations MUST follow this order by default. Implementations MAY provide configuration to alter the order — for example, to disable network resolution, to prepend an in-memory registry, or to invert the local/vocabulary precedence — but the spec-defined default applies when no explicit configuration is provided.

The default order (local first, then declared vocabularies, then global) reflects the principle that closer scopes are more specific and authoritative. A Template defined within a document is always intended for that document; vocabulary-defined Templates are shared definitions; globally-addressable Templates are the most general fallback.

Resolvers SHOULD cache resolved Templates to avoid repeated lookups. Cache invalidation is implementation-defined; resolvers MAY treat resolved Templates as stable for the lifetime of a process or until an explicit reload.

#### 8.5.3 Resolution Failure Behavior

When a `--type` reference cannot be resolved:

- The element carrying the reference is reported as schema-non-conforming.
- The error MUST identify the unresolved reference and the element that carries it.
- Validation of the rest of the document continues; resolution failures do not halt validation.

A resolution failure is distinct from a Template-mismatch error. The former means "the reference does not name any known Template"; the latter means "the reference names a Template, but the instance does not satisfy it." Implementations should distinguish these in error reports.

#### 8.5.4 Error Accumulation

Validators SHOULD accumulate validation errors and report them as a set, rather than failing at the first error encountered. A single validation pass over a document SHOULD produce a complete list of all conformance violations the validator can detect.

This recommendation exists because fail-fast validation forces users to fix one error, re-run, fix the next, re-run, etc. — a workflow that wastes time on documents with multiple independent issues. Accumulating errors lets users address all issues in one round.

Implementations MAY provide a fail-fast mode for use cases where early termination is preferred (e.g., performance-critical pipelines validating known-good data). When both modes are supported, accumulating mode SHOULD be the default.

Error reports SHOULD include:

- The element where the violation occurred (path or location)
- The nature of the violation (resolution failure, constraint failure, etc.)
- A reference to the relevant Template constraint when applicable
- A severity indicator (error, warning, info)

The format of error reports is implementation-defined; the recommendation is for completeness and clarity, not for a specific schema.

### 8.6 Templates Are Flat; Companion Vocabulary

GDX Templates are flat shape declarations. A Template's effective contract is exactly what the Template declares; Templates do not inherit fields or constraints from other Templates. Shared structure across Templates is achieved through application convention, vocabulary versioning, or tooling — not through a Template-level inheritance mechanism.

This decision is deliberate. Inheritance in distributed-vocabulary environments — where Templates from different vocabularies may reference each other across publication boundaries, without compile-time checking to catch breaks — creates failure modes (silent constraint propagation, transitive dependency brittleness, schema-evolution surprises) that outweigh the authoring conveniences inheritance provides. Applications needing shared structure across Templates should publish Templates as flat declarations, possibly generated by tooling from a common source.

A Template's relationship to other Templates is expressed through `--type` references on its child elements. A Customer Template that has an `address` Record whose shape conforms to an Address Template uses `<REC address --Address>` — a structural reference at the field level, not at the Template level. This composition pattern at the field level provides the shared-shape benefit without the propagation problems that Template-level inheritance would introduce.

**The GDX Common Vocabulary.** The GDX Common Vocabulary is a companion specification, separately versioned from this core specification. It defines a base set of widely useful Templates and Constraint Extensions covering numeric width and representation refinements, common semantic types (identifiers, dates, durations, currency), cryptographic and hash types, constraint identifiers, and predicate refinement extensions for use on REL declarations.

The Common Vocabulary is one vocabulary among many. It ships with GDX as a convenience and as a reference vocabulary for the patterns described in this specification, but it has no privileged status at the core layer: per §8.4, constraint identifiers from `gdx:` are extension-layer content in exactly the same way that identifiers from `xsd:`, `json:`, or any application-defined vocabulary are. A document using `xsd:minInclusive` is no less GDX-conformant than a document using `gdx:min`; consumers select which vocabularies they speak.

The Common Vocabulary is versioned independently of the core specification. Vocabulary additions and refinements may occur without revising the core; structural changes to GDX require core revision. Conforming implementations SHOULD support the Common Vocabulary in addition to the core; an implementation that supports only the core is conforming but limited, since common types and constraints will require explicit declaration in every document that uses them.

---

## 9. Transport Semantics

Package (§6.5.7) and Group (§6.5.8) are the two roles reserved for transport: they wrap and partition data items during exchange between systems. This section defines how transport-layer wrappers behave operationally — how to populate them with metadata and payload, how receivers unpack them, and what survives serialization. Definitions of the prefix tokens, composition rules, and required identifiers live in §6.5.7 and §6.5.8.

### 9.1 Package Patterns

A Package carries Objects, Templates, and REL declarations as payload, alongside Properties, Records, Sets, and Annotations as transport metadata. The identifier (when present) is used for transport correlation: request/response matching, deduplication, audit.

A Package with transport metadata and payload:

```
<PKG customer-import-2026-04-29
    <STR source : "crm-v3">
    <STR contentType : "customer-batch">
    <INT count : 2>
    <REM : "Daily customer sync">
    
    <OBJ Alice ...>
    <OBJ Bob ...>
>
```

A vocabulary export Package carrying Templates and RELs:

```
<PKG color-vocab-v1
    <STR vocabName : "color-science">
    <STR version : "1.0.0">
    
    <DEFREC ColorReading
        <FLOAT l>
        <FLOAT a>
        <FLOAT b>
    >
    
    <REL color:measuredBy>
    <REL color:hasReading>
>
```

### 9.2 Group Patterns

A Group identifier is local to its containing Package. Two different Packages may both contain a Group named `pantone`; these are independent. Groups are not globally addressable; their identifiers exist only within the transport scope of their Package.

Multiple color libraries in a single Package:

```
<PKG color-export-2026-04-29
    <STR source : "color-system-v3">
    
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

A choice-of-one transport pattern, where Package metadata signals the receiver to apply only one of the contained Groups:

```
<PKG config-update-2026-04-29
    <STR choose : "one">
    
    <GRP production-config
        <STR environment : "production">
        <OBJ Server-1 ...>
        <OBJ Server-2 ...>
    >
    
    <GRP staging-config
        <STR environment : "staging">
        <OBJ Server-1 ...>
        <OBJ Server-2 ...>
    >
>
```

### 9.3 Unpacking Behavior

A receiver processing a transport-layer wrapper:

1. Reads transport-level metadata (Package and Group identifiers, Properties, Records, Annotations) for any provenance or processing context.
2. Extracts the data-layer payload (Objects, Templates, REL declarations) from the wrapper.
3. Stores or processes the extracted data items according to the receiver's data-layer rules.
4. Discards the transport wrapping; Package and Group elements are not part of what the receiver retains as data.

Receivers MAY log transport metadata as audit information and MAY use Group identifiers to correlate processing of contained items. Receivers MUST NOT treat transport identifiers as part of the data layer; the identifier `pantone` on a Group does not become part of the identity of any contained Object.

### 9.4 Round-Trip Considerations

Transport-layer wrappers do not survive a parse-store-serialize round-trip in their original form. A receiver that imports a Package and later exports the same data items will not necessarily produce a Package; the items might be exported bare or as a different Package with metadata reflecting the export rather than the original import.

Persistence layers MAY retain transport metadata as provenance attached to the imported data items (for example, as a Qualifier or Annotation noting the source Package's identifier and timestamp). Such retention is application-defined and is not part of the core specification's round-trip contract. The data items themselves (Objects, Templates, REL declarations) round-trip per Appendix A regardless of the transport wrapper they arrived in.

---

## 10. Data Persistence

This section addresses how GDX data is stored and how stored data round-trips through serialization. It is informative; the core specification defines the data model and canonical text form, while persistence concerns are layered on top.

### 10.1 Wire Formats

The Standard Form defined in §5 is the normative reference. Two additional wire formats are defined in companion specifications:

- **Compact Form** — a line-oriented form using indentation rather than brackets to convey structure, designed for human authoring and for environments where bracket noise is unwanted. Each element occupies one line; child elements are indented under their parents. The Compact Form preserves all fields of the Standard Form; only the structural delimiters differ.
- **Binary Form** — a binary encoding optimized for fast wire transfer, compact size, and efficient parsing. The Binary Form preserves the data model of §4 with no information loss; round-trip through the Binary Form produces a document equivalent to the original Standard Form input.

A wire format is *conforming* if it preserves the data model defined in §4 and admits round-trip equivalence with the canonical form (Appendix A). The full grammars and encoding rules for the Compact Form and Binary Form are defined in their respective companion specifications.

### 10.2 Database Persistence

GDX maps cleanly to relational storage with a row-to-element mapping. This is one of several persistence strategies; document stores, graph databases, and key-value stores can also serve as GDX backends.

A single table can store an arbitrary GDX document with the following column structure:

| Column         | Purpose                                                            |
|----------------|--------------------------------------------------------------------|
| `RowId`        | Unique identifier for this element row                             |
| `RootId`       | The RowId of the document's root element                           |
| `ParentId`     | The RowId of this element's parent (NULL for the root)             |
| `NextSiblingId`| The RowId of the next sibling, for ordered child traversal         |
| `Prefix`       | The element's prefix token                                         |
| `Key`          | The element's Key field (NULL if not present)                      |
| `Rel`          | The element's Rel field (NULL if not present)                      |
| `Id`           | The element's Id field (NULL if not present)                       |
| `ValueKind`    | The value-kind of the Resource (NULL if not present)               |
| `ResourceValue`| The Resource value, encoded per the value-kind                     |
| `Type`         | The element's Type reference (NULL if not present)                 |

This single-schema approach allows GDX storage without per-document schema migration. Adding a new Object role, Property, or Template requires no schema changes — just additional rows.

Edge references stored as `ResourceValue` strings can be resolved at query time by joining against the `Id` column. For systems where the reference target is known at write time, the reference can be denormalized to a `TargetRowId` column for direct lookup. The resolution strategy is implementation choice; both approaches are conforming.

Other persistence strategies (document store, graph database, key-value store) follow analogous principles: preserve the data model, preserve element ordering, and preserve the round-trip contract (§10.4).

### 10.3 Data Interchange

GDX maps to and from common data interchange formats (JSON, YAML, XML, RDF). Each mapping has trade-offs: GDX features without direct equivalents in the target format (predicate alignment, native bytes, schema-as-data) require convention-based encoding or are lost in conversion. Implementations performing format conversion SHOULD document their conventions.

Side-by-side comparisons between GDX and other formats, along with discussion of mapping considerations, are in the Format Comparison companion specification.

### 10.4 Persistence Round-Trip

A persistence layer is *round-trip conforming* if a document parsed, stored, and serialized back produces a document equivalent to the original under Appendix A's structural equivalence rules.

Specifically, persistence layers MUST preserve:

1. Element prefixes
2. Identity fields (Id, Key, Rel) with case-sensitive ordinal comparison
3. Resource value-kinds and values
4. Type references
5. Child element order
6. Annotations (REM elements)
7. Null state: an element with no Resource value MUST round-trip as an element with no Resource value. An element with a Resource MUST round-trip with the same Resource value, including the empty string for string-typed Properties. The semantic distinction between null and present-with-value MUST be preserved.

Persistence layers MAY normalize:

- Whitespace within and between elements (per §5.1.2)
- Hex case in bytes literals (canonical form is lowercase)
- Boolean literal forms (canonical form is `true`/`false`; `t`/`f` and uppercase variants normalize to lowercase spelled-out form)
- Null Property forms (canonical form is the bare `<STR name>`; the empty-value form `<STR name : >` normalizes to the bare form)
- Indentation in Standard or Compact forms

Persistence layers MUST NOT alter:

- Numeric representation suffixes (a value stored as `25i8` is retrieved as `25i8`)
- String content within quotes (escape sequences are part of the literal)
- Element identity or ordering
- The semantic distinction between null and present-with-value

Transport-layer roles (Package, Group) are not subjects of the round-trip contract; see §9.4.

---

## 11. Open Issues

This section enumerates issues that the v1.0 specification SHOULD resolve before final publication. Each entry describes the issue and references the affected section.

### 11.1 Common Vocabulary Specification

**Section:** §8.6

The GDX Common Vocabulary is a companion specification, separately versioned. The current draft (v0.2) defines a Core / Standard / Extended level structure with approximately 70 normatively-defined identifiers across Core and Standard levels and a sketch of Extended-level content. Full population of Extended-level content, refinement based on implementer experience, and additional domain-specific vocabularies remain ongoing work.

### 11.2 Compact Form Grammar

**Section:** §10.1

The Compact Form is described conceptually and demonstrated by example in this specification. A companion grammar specification (v0.2) now defines the indentation rules, line-continuation handling, escape conventions, multi-line literals, and the precise mapping to Standard Form. 

### 11.3 Three-State Constraint Vocabulary

**Section:** §4.3.3

The introduction of distinct absent/null/present states implies three independent constraint dimensions for Templates: required-vs-optional (whether the Property may be absent), nullable-vs-non-nullable (whether a present Property may have no Resource), and value constraints (what values are permitted when a Resource is present). The Common Vocabulary v0.2 begins addressing this through `gdx:required`, `gdx:optional`, and value-constraint extensions; the full taxonomy and the relationship between optionality and nullability may benefit from further refinement based on implementer experience.

### 11.4 Path-Mode Reference Resolution — Resolved

**Section:** §5.2.6, §6.5.16, §7.1

**Status:** Closed as of v0.11.0.

Path-mode reference resolution is normatively specified by the companion GDX Path Expressions Specification (`GDX-Path.md`). All sub-items previously open under this issue are addressed:

- **The path expression grammar.** Specified in `GDX-Path.md` §3 and §4. Three segment kinds — `IdSegment`, `RelSegment`, `OrdinalSegment` — separated by `/`. Key-based addressing is provided through the `KeyRef` and `IdKeyRef` reference-literal forms rather than as a path segment.
- **The resolution algorithm.** Specified in `GDX-Path.md` §6. Left-to-right segment traversal from a defined starting element, with per-segment match semantics and explicit handling of zero, one, and multi-match outcomes.
- **The composition matrix expansion.** Specified in `GDX-Path.md` §8.5 (changes required of this specification). `@REC`, `@BLK`, `@.STR`, `@_INT`, `@%REC`, value-Property targets, and other previously-unresolvable `@KIND` forms become resolvable when the resource is a `PathRef` or `IdPathRef`. Parent-context restrictions are unchanged; only target-addressability expands.
- **Modifier-stacking interaction.** The `@`-with-modifier-bases pattern (`@.STR`, `@_INT`) now resolves through path-mode references.

The composed reference form `@Identifier$PathExpression` is the canonical mechanism for cross-document addressing: the identifier portion resolves a starting element by scope walk (per §6.5.16), and the path portion traverses structurally from there. This form replaces the speculative path-style identifier convention (e.g., `@/people/alice`) used in earlier revisions; with the v0.11.0 narrowing of `IdStartChar` (§5.4.1), identifiers no longer begin with `/`.

---

## Appendix A — Conformance

This appendix defines what it means for a serialization, a parser, and a document to be conforming to this specification. Conformance is layered: a document may be representation-conforming without being schema-conforming, and an implementation may support one or both layers.

### A.1 Conformance Layers

Conformance is layered, corresponding to the layers introduced in §2.2:

**Representation Conformance (Layer 1)** addresses how data is encoded. It covers the structural rules of §4 through §7: prefix vocabulary, identifier grammar, literal forms, composition rules, and resource value-kind matching. Owned by GDX. Decidable at parse time without external context.

**Schema Conformance (Layer 2)** addresses how data aligns to declared shapes and constraints, and decomposes into two sub-layers:

- **Layer 2a — Template structural conformance.** Covers the Template-based alignment of §8.1, §8.2, §8.3: instance children align to Template slots by `Id` (with `Rel` as cross-vocabulary bridge), and `--type` references resolve. Owned by GDX. Decidable by any consumer that can resolve Templates.

- **Layer 2b — Constraint-vocabulary conformance.** Covers the Constraint Extensions of §8.4: whether instance values satisfy the constraints expressed by a particular constraint vocabulary (XSD, JSON Schema, the GDX Common Vocabulary, or any other). Owned by the constraint vocabulary, not by GDX. Decidable only by a consumer that understands the vocabulary's constraint identifiers. A document MAY be Layer 2b-conforming with respect to one vocabulary and Layer 2b-non-conforming with respect to another; the verdicts are independent and vocabulary-relative.

A document is **fully conforming** at Layer 1 + Layer 2a if it is representation-conforming and Template-structurally-conforming with respect to all Templates referenced via `--type`. A document with no Template references is fully conforming at Layer 1 + Layer 2a if it is representation-conforming. **Layer 2b conformance is always vocabulary-qualified**: a document is "Layer 2b-conforming with respect to vocabulary V" when consumed by a V-aware consumer that finds no constraint violations. Claims of Layer 2b conformance MUST name the vocabulary against which the claim is made.

### A.2 Serialization Conformance

A serialization is **representation-conforming** if and only if a round-trip through it preserves, for every element in the document:

1. The Prefix.
2. The presence and value of `Key`, `Rel`, `Id`, and `Type` (string-equal, ordinal comparison).
3. The Resource — including:
   - The null state (an element with no Resource value MUST round-trip as a null-state element).
   - The value-kind of the Resource (per §4.3).
   - The value of the Resource (per §4.3 and §5.2).
   - Any representation tag carried by the Resource (numeric width suffixes, bytes encoding prefixes).
4. The ordered list of children (recursively conforming).
5. Annotation (REM) elements in their original positions.

A serialization MAY normalize:

- Whitespace within and between elements (per §5.1.2).
- Hex digit case in bytes literals (canonical form is lowercase).
- Boolean literal forms (canonical form is `true`/`false`; serializers normalize `t`/`f` and uppercase variants to lowercase spelled-out form).
- Null Property forms (canonical form is the bare `<STR name>`; serializers normalize the empty-value form `<STR name : >` to the bare form).
- Indentation in Standard or Compact forms.
- The choice of Standard, Compact, or Binary form (so long as the chosen form is itself conforming).

A serialization MUST NOT alter:

- Numeric representation suffixes. A value stored as `25i8` is retrieved as `25i8`, not as `25` or `25i64`.
- String content within quotes. Escape sequences are part of the literal and round-trip exactly.
- Element identity, order, or structure.
- The semantic distinction between null and present-with-value. The bare and empty-value forms of null are interchangeable; null and present-with-value are not.

### A.3 Parser Conformance

A parser is **representation-conforming** if it:

1. Accepts every document well-formed under §4 through §8.
2. Rejects every document that violates a MUST clause in §4 through §8 and §5.4.
3. Reports rejection as a *malformed document* error, distinct from schema validation errors.
4. Preserves all information required for representation-conforming round-trip serialization (§A.2).

A parser MAY additionally implement schema conformance (Layer 2). A parser that implements only representation conformance is conforming for the purposes of this specification; schema validation may be performed by a separate component or implementation.

### A.4 Document Conformance

A document is **representation-conforming** if a representation-conforming parser accepts it without error.

A document is **schema-conforming** with respect to a set of Templates if, for every element carrying a `--type` reference resolvable to one of those Templates, the element satisfies the Template's declared shape and constraints ( §8).

A document MAY claim conformance to one or more vocabularies (§8.6). A document is **vocabulary-conforming** to a vocabulary if its use of that vocabulary's identifiers (in `Rel`, `Type`, and Extension positions) is consistent with the vocabulary's specification.

The three conformance claims (representation, schema, vocabulary) are independent. A document may be representation-conforming without being schema-conforming, and either without being vocabulary-conforming.

A document containing transport-layer roles (Package, Group) is conforming under the same rules as any other document. Conforming receivers unpack transport wrappers per §9.3 before applying schema or vocabulary validation to the contained data items. Schema and vocabulary conformance apply to the data items extracted from transport wrappers, not to the wrappers themselves.

### A.5 Strict Canonical Form

For applications requiring byte-for-byte stability (content-addressable storage, signed payloads, cryptographic hashing), this specification defines a **strict canonical form**. A document in strict canonical form satisfies the following constraints in addition to all representation-conformance requirements:

1. A single ASCII space character separates adjacent fields within an element.
2. A single ASCII space character appears on each side of the `:` separator when a Resource is present.
3. No whitespace appears immediately after `<` or immediately before `>`.
4. Children of multi-line elements are indented with four ASCII space characters per nesting level.
5. Each child element appears on its own line.
6. No trailing whitespace appears on any line.
7. Lines are terminated by a single line-feed character (`\n`).
8. Hex digits in bytes literals appear in lowercase.
9. Boolean values appear as `true` or `false` (lowercase spelled-out form). The single-letter forms (`t`, `f`) and uppercase variants are NOT used in canonical output.
10. Null Properties appear in the bare form (`<STR name>`), not the empty-value form (`<STR name : >`). The empty-value form is accepted on input but normalized to the bare form on canonical output.
11. Field write order is exactly Prefix · Key · Rel · Id · Resource · Type.
12. Prefixes appear in **canonical form** — three-letter (or longer where natural) names in uppercase: `STR`, `INT`, `BOOL`, `FLOAT`, `BYTE`, `REC`, `OBJ`, `EDGE`, `GRP`, `STR*`, `INT*`, `.STR`, `_STR`, `%STR`, `%REC`, `%OBJ`, `NSTR`, `NINT`, `NBOOL`, `NFLOAT`, `NBYTE`, `NREC`, `DEFSTR`, `DEFINT`, `DEFBOOL`, `DEFFLOAT`, `DEFBYTE`, `DEFREC`, `DEFOBJ`, `REL`, `REM`, `PKG`, `BLK`. The minified single-letter forms (`S`, `I`, `L`, `F`, `B`, `R`, `O`, `E`, `G`, `S*`, `.S`, `_S`, `%S`, `NS`, etc.) are valid GDX but do not produce strict canonical bytes; conversion to canonical form is required before content addressing.

Two strict-canonical-form documents that represent the same data model produce byte-identical serializations. This property allows the strict canonical form to serve as the input to content-addressing functions.

The strict canonical form is OPTIONAL. Implementations not requiring byte-stability may produce any representation-conforming serialization, including the minified form for transport efficiency.

**Minified form output.** Implementations MAY produce minified form output for transport-efficiency contexts. The minified form replaces canonical prefix names with their single-letter equivalents where they exist (`STR` → `S`, `STR*` → `S*`, `NSTR` → `NS`, etc.). Prefixes without minified equivalents (`REL`, `REM`, `PKG`, `BLK`, `DEFSTR`, `DEFREC`, etc.) are emitted unchanged. Minified output produces a different byte sequence than canonical output for the same data; content addresses computed against minified output are NOT comparable to addresses computed against canonical output.

### A.6 Implementation Claims

An implementation claiming conformance to this specification SHOULD declare:

- Which conformance layer(s) it implements (representation only, or representation and schema).
- Which wire format(s) it supports (Standard, Compact, Binary).
- Whether it produces strict canonical form output.
- Which vocabularies it supports, with version identifiers.

The specification does not define a formal conformance certification process. Claims are self-attested.

### A.7 Conformance Test Suite Outline

A conforming implementation SHOULD verify its behavior against a test suite covering the cases enumerated below. This appendix outlines the categories of tests; the actual test corpus is a separate deliverable maintained alongside the specification.

**Lexical and grammatical tests (Layer 1):**

- Valid documents at every prefix in §6 (one positive test per prefix).
- Identifier forms: local names, single letters, paths, CURIEs, GUIDs, IRIs (one positive test per form per applicable position).
- Literal forms: each value-kind including representation suffixes (`i8` through `u64`, `f32`/`f64`, `decimal` family), boolean variants (`true`/`false`/`t`/`f` and case variants), bytes (`0x` and `0z` forms), null forms (bare and empty-value).
- Sigil tightness: positive tests for valid `&`, `#`, `--`, `@` usage; negative tests for whitespace violations.
- Whitespace handling: documents with various legal whitespace patterns; round-trip equivalence to canonical form.
- Comments: file-level `#` comments in Compact Form (positive); rejection of comment-position `#` in Standard Form (negative).

**Composition tests (Layer 1):**

- Each row of the §8.1 composition matrix represented as a positive test (parent role contains valid child role) and at least one negative test (parent role contains invalid child role).
- Top-level element restriction: positive tests for Object/Template/REL/Package as document root; negative tests for other roles at document root.
- Set member rules: positive tests for homogeneous Set members; negative tests for heterogeneous members where applicable.
- Node subtree isolation: positive tests for Node-only subtrees; negative tests for structured roles appearing within Node subtrees.

**Resource and value tests (Layer 1):**

- Each value-kind with edge values: zero, negative numbers, max/min values per width, infinity, NaN, very long strings, empty strings, empty bytes.
- Reference resolution within a single document: positive tests where targets exist; tests for unresolved references (which produce Layer 2 errors per §6.5.4.1, not Layer 1).
- Round-trip preservation: a document parsed and serialized to canonical form, then re-parsed, produces an identical data model.

**Schema validation tests (Layer 2):**

- Template conformance: instances claiming `--type` references that resolve and satisfy / fail the Template's constraints.
- Resolution order: tests confirming the §8.5.2 default order is followed (local document first, then loaded vocabularies, then global IRI).
- Resolution failure: tests confirming unresolved type references produce Layer 2 errors per §8.5.3.
- Validator modes: strict and lenient mode behavior on the same input documents.
- Error accumulation: validator returns multiple errors from a single pass (per §8.5.4).
- Edge reference resolution: tests confirming §6.5.4.1 behavior for resolved and unresolved Edge `@` references.

**Canonical form tests:**

- Documents in non-canonical form serialize to byte-identical canonical form per Appendix A.5.
- Two structurally-equivalent documents produce the same canonical form.
- Boolean normalization (`t`/`f` and uppercase forms normalize to lowercase spelled-out).
- Hex-bytes case normalization (uppercase normalizes to lowercase).
- Null form normalization (empty-value form normalizes to bare form).

**Resilience and security tests:**

- Documents at the recommended depth limit (positive); documents exceeding it (rejected per §1 of the Security Considerations companion).
- Documents at the recommended element-count limit (positive); documents exceeding it (rejected).
- Numeric literals at maximum length (positive); numeric literals exceeding it (rejected per §1 of the Security Considerations companion).
- Reference cycles in Edge graphs: cycles are valid data; tests confirm parsers accept them and traversal-based operations terminate.

**Compact Form tests (when supported):**

- Equivalence: every Standard Form test document, converted to Compact Form, should parse to the same data model.
- Indentation: documents with various consistent indent units; rejection of mixed indents.
- Multi-line strings (`|` and `|<` forms).
- Round-trip: Compact → Standard → Compact (data-model preservation; comment loss is expected per Compact Form spec).

The test suite is exhaustive in concept and bounded in practice. Each category above expands to dozens of specific tests when fully populated. A conformance test suite covering these categories is a substantial deliverable; implementations may bootstrap from a subset and expand coverage over time. The intent is that an implementation passing the full suite has demonstrably correct behavior on the cases that matter for interoperability.

---

