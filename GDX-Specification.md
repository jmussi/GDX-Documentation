# GDX — Graph Data eXchange Specification

- **Status:** Draft v0.22.0 (working draft, pre-1.0)
- **Date:** 2026-07-07
- **Editors:** 
- **Authors:** Julian Mussi


## Abstract
This document defines the GDX abstract data model, the GDX prefix vocabulary and composition rules, the canonical text form, and the schema and validation mechanism. It is independent of any particular wire format. Companion specifications cover the Compact and Binary Forms.


## Status of This Document

This is a pre-1.0 working draft. The specification is under active refinement; revisions may include breaking changes until v1.0.

### Revision History

| Version | Date       |
|---------|------------|
| v0.22.0 | 2026-07-07 |
| v0.21.0 | 2026-06-09 |
| v0.20.0 | 2026-06-09 |
| v0.19.0 | 2026-06-09 |
| v0.18.0 | 2026-05-25 |
| v0.17.0 | 2026-05-25 |
| v0.16.0 | 2026-05-25 |
| v0.15.0 | 2026-05-25 |
| v0.14.0 | 2026-05-25 |
| v0.13.0 | 2026-05-25 |
| v0.12.0 | 2026-05-25 |
| v0.11.0 | 2026-05-20 |
| v0.10.0 | —          |
| v0.9.10 | —          |
| v0.9.9  | —          |
| v0.9.8  | —          |
| v0.9.7  | —          |

Detailed change descriptions for each revision are collected in **Appendix B — Detailed Revision History** at the end of this document.


## Table of Contents

1. [Introduction](#1-introduction)
2. [Conventions and Terminology](#2-conventions-and-terminology)
3. [The GDX Abstract Data Model](#3-the-gdx-abstract-data-model)
4. [Canonical Text Form](#4-canonical-text-form)
5. [Prefix Vocabulary](#5-prefix-vocabulary)
6. [Element Roles](#6-element-roles)
7. [Composition Axioms](#7-composition-axioms)
8. [Schemas and Validation (Templates)](#8-schemas-and-validation-templates)
9. [Transport Semantics](#9-transport-semantics)
10. [Data Persistence](#10-data-persistence)
11. [Open Issues](#11-open-issues)
- [Appendix A — Conformance](#appendix-a--conformance)
- [Appendix B — Detailed Revision History](#appendix-b--detailed-revision-history)

---

## 1. Introduction

### 1.1 What GDX Is

GDX (Graph Data eXchange) is a typed graph data model with a closed structural schema. The model is the primary thing: an abstract content of elements, each with a prefix declaring its role and value-kind, optional identity fields, an optional typed value, an optional schema-type claim, and optional children that compose into trees with graph references. Everything else — the text you write, the bytes on a wire, the objects in a running program — is a *realization* of that model: a concrete form the same abstract content takes for human reading, machine exchange, or in-memory processing. Standard Form (§4), the Compact and Binary Forms (companion specifications), and a runtime's in-memory form are alternative realizations; a GDX document conveys the same model regardless of which one carries it.

This distinction governs how the specification is organized. The model (§3) is authoritative; the canonical text form (§4) and the other realizations are defined as faithful renderings of it, and conversions between realizations transform surface syntax without changing the model content. What this document specifies, then, is the model together with the rules a realization must satisfy to carry that model without loss — most fully for Standard Form, the canonical realization defined here; the Compact and Binary Forms are defined in companion specifications. Conformance is defined in Appendix A and is layered: it ranges from representation conformance (a document is well-formed against the structural and encoding rules) up through schema conformance. The orienting point for a reader new to the document is that GDX is not, at bottom, "a file format" — it is a model, and a realization conforms by carrying that model faithfully.

A short example in Standard Form:

```
<OBJ Alice
    <STR firstName : "Alice">
    <INT age : 24>
    <BOOL active : true>
    <@OBJ #knows : @Bob>
>
```

This declares an Object `Alice` with three typed properties and an @-edge expressing a `knows` relationship to `Bob`. Every element states its role, value-kind, and content explicitly; no external schema is needed to know that `age` is an integer or that the @-edge points to another entity.

This specification is the **normative** definition of GDX: the data model (§3), the canonical text form (§4), the prefix vocabulary and roles (§5–§6), the composition rules (§7), the schema mechanism (§8), transport (§9), and conformance (Appendix A). It is written for implementers and for adjudicating conformance.

A reader new to GDX — wanting to understand what it is, why it is shaped this way, and how to think in it — should start with the **GDX Primer** (companion, non-normative), which introduces the model by example, states the design goals, and compares GDX against JSON, YAML, XML, RDF/OWL, and binary protocols. The Primer teaches; this document specifies. The two are cross-referenced: the Primer cites the specification sections that define each concept normatively.


### 1.2 Scope of This Document

This specification defines the GDX core: the data model, the prefix vocabulary, composition rules, the canonical text form, the identifier and literal grammars, and the Template-based schema mechanism. It does not define standardized predicate vocabularies, semantic types, validation extension catalogs, or the binary wire format. These are the subject of companion specifications and the *GDX Common Vocabulary*.

The core specification is intentionally minimal. Concerns that are likely to evolve over time — predicate semantics, domain-specific types, validation extensions, mapping conventions — live outside the core in versioned vocabularies that may evolve without revising the structural specification.

This document specifies:

- The element model and field semantics (§3); the conceptual introduction is in the GDX Primer.
- The abstract structure of a GDX element (§3).
- The canonical text form (§4).
- The closed prefix vocabulary, structural roles, and modifiers (§5).
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

- **Prefix** — the leading token that declares an element's role. Belongs to the closed vocabulary defined in §5.

- **Role** — the structural position an element occupies in the data model *and* the conceptual interpretation that position carries. A role names both the structural shape (what fields and children are admitted) and the meaning (how the element is to be interpreted in relation to its parent and the wider document). The closed enumeration of roles — Object, Record, Block, Property, Set, Node, VAL, @-Edge family (`@OBJ`, `@REL`), Reference family, Qualifier, Extension, Template, Relation Declaration, Annotation, Package, Group — covers both axes; each role's detail is in §6. Obtained from a prefix by stripping any modifier.

- **Modifier** — a sigil or letter prefix-fragment that adapts a base role without changing its composition rules: `*`(Set), `.`(Qualifier), `_`(Extension), `N`(Node), `DEF`(Template).

- **Resource** — the typed scalar value carried by an element, when present. Resource value-kinds are `string`, `int`, `float`, `bool`, `bytes`, and `reference` (§3.3).

- **Value-Kind** — the type category of a Resource. Distinct from Role (which describes structural position). A Property of prefix `STR` carries a Resource of value-kind `string`.

- **Identifier** — any of the three identity fields (`Id`, `Key`, `Rel`) that name or link an element. Some roles require at least one (§3.2).

- **Document** — a single rooted GDX tree, or a sequence of root-eligible elements treated as a unit of serialization. Which roles may root a document is fixed normatively by the root rule (§7.2); informally, the root(s) are Objects, Templates, REL declarations, or Packages.

- **Data layer** — the set of GDX roles representing persistent, addressable, schema-validatable content: Objects, Records, Blocks, Properties, Sets, Nodes, Edges, Qualifiers, Extensions, Templates, Relation Declarations, and Annotations.

- **Transport layer** — the set of GDX roles representing transient organizational and exchange constructs: Package and Group (§9). Transport-layer constructs are unpacked by receivers and are not persisted as such.

- **Core** — the structural and semantic content normatively defined by this specification: the prefix vocabulary, the element model, the identifier and literal grammars, the composition axioms, the Template structure and instance-alignment mechanism, the universal claim rule, and the primitive value-kind axioms. The core is what GDX *is*; it is fixed by this specification and does not change when extensions are added. (Distinguish "the core" — the concept — from "the Core Specification" — the document name. The latter is this document.)

- **Extension** — vocabulary-defined material that rides on top of the core through GDX's extension-point mechanisms. Extensions include: predicate identifiers in the `Rel` field, Template names in the `Type` field, and Constraint Extension elements (§8.4) carrying constraint identifiers from external vocabularies. The core records extensions structurally and validates their well-formedness; their *semantic content* is interpreted externally by consumers that understand the vocabulary in question. The core/extension distinction is the **interpretation** boundary between what GDX knows and what GDX merely carries.

- **Core layer / Extension layer** — when a discussion needs to distinguish what GDX understands from what GDX hosts, the terms *core layer* and *extension layer* refer to those two sides of the interpretation boundary. The core layer is owned by this specification; the extension layer is owned by external vocabularies and their consumers.

### 2.2 Conformance Layers

GDX defines a layered conformance model: each layer is independent, evaluated by a different agent, and reports its own kind of error. This subsection introduces the model conceptually and records the decision behind its shape; the full normative statement of the layers — what each covers and what conforming implementations MUST do — is **Appendix A.1**, which is authoritative. (A future revision may relocate the normative layer model to a standalone early section; until then A.1 is its home and this subsection defers to it.)

In outline, there are three rungs with three owners:

- **Representation Conformance (Layer 1)** — how a value is encoded (prefix-to-value-kind matching, literal grammar, representation tags, round-trip text equivalence). Owned by GDX; evaluable at parse time; errors are *malformed document* errors and are absolute.
- **Schema Conformance, Layer 2a — Template structural conformance** — whether an instance's children align to a referenced Template's declared slots. Owned by GDX; evaluable by any consumer that can resolve Templates.
- **Schema Conformance, Layer 2b — Constraint-vocabulary conformance** — whether values satisfy a particular constraint vocabulary's constraints (`xsd:`, `json:`, a domain vocabulary). Owned by the constraint vocabulary, not GDX; evaluable only by a consumer that understands that vocabulary. GDX records Constraint Extension elements (§8.4) but renders no verdict on them.

The layers are independent and the verdicts are vocabulary-relative above the primitive floor; Appendix A.1 states the conformance combinations and the reporting requirement normatively.

**Why decompose this way (decision).** The decomposition reflects the core/extension boundary (§2.1): Layer 1 and Layer 2a are the **core layer** — what GDX itself defines, owns, and evaluates — while Layer 2b is the **extension layer**, owned by external constraint vocabularies and evaluated by their consumers. The alternative considered was a single, monolithic valid/invalid verdict in the manner of XSD, JSON Schema, and OWL, each of which conflates these rungs into one answer. That was rejected: separating the layers by *owner* is what lets multiple constraint vocabularies coexist on the same data without GDX taking a side (§8.4), and what lets a document be core-valid yet constraint-invalid for one vocabulary and constraint-valid for another. A monolithic verdict cannot express that independence.

### 2.3 Vocabulary Terms

- **Vocabulary** — a versioned collection of standardized identifiers, Templates, and Extensions defining shared semantics for a domain. Vocabularies are layered on top of the core specification through the existing extension and template mechanisms.

- **Common Vocabulary** — the GDX Common Vocabulary, a companion specification shipping with the core. Defines standardized representation-tag suffixes, common Templates (numeric width, hash types, semantic types), and common predicate refinement extensions.

- **Representation Tag** — a literal-level annotation declaring how a value is encoded. Drawn from a closed vocabulary defined in §4.2 and referenced by suffix (e.g., `i8`, `f64`, `d`) or prefix (e.g., `0x`, `0b`).

### 2.4 Identifier Schemes

- **Local Identifier** — an identifier whose meaning is scoped to the document containing it. Property and Record names are local identifiers.

- **Resource Identifier** — an identifier whose meaning extends beyond the document. Object identifiers are resource identifiers. The specification does not constrain the identifier scheme; CURIEs, paths, GUIDs, and other forms are all valid.

- **CURIE** — a Compact URI, of the form `prefix:local`, where `prefix` is a vocabulary alias and `local` is the local identifier within that vocabulary. Example: `gdx:Person`.

The grammar for identifiers is uniform across all identifier-bearing positions (§4.4). The semantic interpretation differs by context but the lexical form does not.

---

## 3. The GDX Abstract Data Model

This section defines GDX at runtime: the abstract structure an implementation must represent, the rules an implementation must enforce, and the semantic states an implementation must distinguish. The conceptual introduction is in the GDX Primer; the text form is in §4.

The data model defined here is the single authoritative content that every concrete GDX representation faithfully realizes (§1.1). Standard Form (§4), Compact Form (companion specification), Binary Form (planned companion), and the programmatic in-memory form a runtime exposes are alternative realizations of the same model: each preserves the same elements, the same prefixes, the same field semantics, the same composition relationships. Conversions between representations transform the surface syntax without changing the model content; a round-trip through any combination of representations yields the same abstract document.

A GDX element is the tuple

```
(Prefix, Key?, Rel?, Id?, Resource?, Type?, Children*, Metadata?)
```

All fields except Prefix are optional at the data-model level. Whether a given role requires a particular field is governed by §6 (Element Roles) and §7 (Composition Axioms). `Metadata` is set apart from the other fields: it is realization-layer carriage annotation, explicitly **not** part of the model content, and is defined in §4.8.

### 3.1 Element Fields

An element has six model fields plus an out-of-model metadata channel. The Prefix is required; the other model fields are optional at the data-model level. The metadata channel (§4.8) is not a model field — it carries realization-layer annotation and does not participate in model content, equality, or conformance. Whether a particular field is required for an element depends on the element's role (§5) and its position relative to its parent (§3.2).

| Field      | Purpose |
|------------|---------|
| `Prefix`   | Declares the element's role. Required. Drawn from the closed vocabulary in §5. |
| `Key`      | An optional named anchor identifying the element within its enclosing Object's scope, independent of its tree position. Used as a target for `&Key` reference literals (§4.2.6). |
| `Rel`      | A predicate-style label naming the relationship between this element and its parent. |
| `Id`       | A local or resource identifier — the element's name in the document. |
| `Resource` | The element's typed scalar value. |
| `Type`     | A reference to a Template the element claims to satisfy. |
| `Metadata` | *(Out-of-model.)* Realization-layer carriage annotation — flat key/value pairs about how the element is stored or transmitted, not about what it means. Not model content; see §4.8. |

All identity fields (`Id`, `Key`, `Rel`) use case-sensitive ordinal comparison.

On an @-Edge element (§6.4), an existence/specification split applies to the `Rel` field: the *role* asserts that a relationship exists (an @-edge is, by its role, a relationship), while the `Rel` field specifies *what* the relationship is — and MAY be empty. An empty Rel parses to null and means the relationship is **undefined as to nature**: it exists (the role settled that), but its form is not yet specified (§6.4.2). The empty Rel is the field's ordinary null/absent state — no special syntax and no distinct construct are involved.

Implementations MUST represent these fields as distinct elements of an element's runtime structure. The order in which fields are stored or accessed is implementation-defined; only the children list has a normative order (§3.4).

**Decision: every element divides into an identity portion and a value portion.** The six fields partition into identity (`Prefix`, `Key`, `Rel`, `Id`) and value (`Resource`, `Type`), with the `:` separator marking the boundary in the text form (§4). *Alternatives considered:* JSON-style bare key-value pairs (identity and content on one syntactic level, with no way to distinguish "this element claims to be of type X" from "this element has value X"); XML-style attribute-versus-element placement (where either can carry identity or content by convention); RDF-style separate triples that must be reassembled to recover an element's full meaning. *Rejected because:* collapsing identity and value forces every richer claim — type, predicate alignment, stable anchor — to be bolted on by external convention rather than stated on the element itself. The explicit split is what makes an element self-describing (the self-describing-element concept; see the GDX Primer): a consumer can read the identity portion and know the element's role, type, and alignment, then read the value portion for its content, with no external schema. This is the structural basis for the typed-elements and inline-semantic-alignment goals (§1.2). The split is introduced conceptually in the GDX Primer; this is its authoritative basis in the data model.

### 3.2 Identifier Requirements

Identifier requirements apply to elements in *named-attribute positions* — positions where the element represents a distinct named property of its parent.

The following roles, when in named-attribute position, MUST carry at least one of `Id`, `Key`, or `Rel`:

- Object (`OBJ`/`O`, `DEFOBJ`)
- Record (`REC`/`R`, `DEFREC`, `NREC`/`NR`) when not in Set member position
- Property (`STR`/`S`, `INT`/`I`, `FLOAT`/`F`, `BOOL`/`L`, `BYTE`/`B`, and their template variants) when not in Set member position
- Template (any `DEF…`)
- VAL (`VAL`/`V`) when not in Set member position (rare; VAL is typically a named slot)

Elements appearing as members of a Set (§6.13) are in *positional* rather than named-attribute position. They are not required to carry identifiers; they identify themselves by position and value within the Set. Identifiers on Set members are valid and MAY be used when meaningful (for example, to identify individual Records within a Record Set).

An element appearing as the **occupant of a VAL slot** (§6.11) is similarly in a positional rather than named-attribute position with respect to the VAL: the slot is the named-attribute position, and the occupant fills it. The occupant MAY carry its own `Id`, `Key`, or `Rel` for its own purposes (occupant-level identity, addressable from references), but it is NOT required to carry an identifier merely because its role would otherwise require one in a named-attribute position. The two identity levels (slot-level on the VAL, occupant-level on the child) are independent; this separation is what distinguishes VAL from a plain Property whose Resource is the value.

A Prefix is REQUIRED on every element regardless of position. Set members MUST carry their prefix even when their identifier is omitted, because the prefix distinguishes Set members from Qualifiers, Annotations, Extensions, and other elements that may appear within a Set's children.

A Property, Record, Edge, or other Resource-bearing element MAY appear with an identifier but no Resource section. The element is well-formed in this state. The semantic interpretation depends on context: in Template context, the form declares a slot without a default value; in instance context, the form represents a null value (§3.3.3).

The lexical grammar for identifiers is uniform across all identifier-bearing positions; see §4.4. The semantic interpretation of an identifier depends on the role that carries it:

- Identifiers on Property, Record, Edge, and Template elements name a relationship within the document. They are functionally equivalent to a field name, label, or variable name in a programming language.

- Identifiers on Object elements identify a resource and are used to resolve references to that Object. The specification places no constraint on the identifier scheme; paths (`/people/alice`), CURIEs (`gdx:Person`), GUIDs, and IRIs (`https://acme.com/people/alice`) are all valid.

This semantic distinction is not reflected in the grammar. A property identifier and an Object identifier draw from the same lexical production (§4.4); they differ only in how consumers interpret them.

### 3.3 Resource Value Kinds

A Resource, when present, MUST be one of:

| Value-Kind  | Description                                                                |
|-------------|----------------------------------------------------------------------------|
| `string`    | UTF-16 character sequence; quoted in text.                                 |
| `int`       | Mathematical integer (any whole number).                                   |
| `float`     | Real number. Representation is determined by suffix (§4.2).                |
| `bool`      | `true` or `false`.                                                         |
| `bytes`     | Opaque sequence of octets.                                                 |
| `reference` | A reference to another Object by identifier.                               |

The Resource value-kind MUST be consistent with the element's prefix family (string-family prefixes carry strings, int-family prefixes carry ints, etc.). Coercion between value-kinds is implementation-defined and SHOULD be avoided.

#### 3.3.1 Numeric Value-Kinds

The `int` value-kind represents any whole number. The `float` value-kind represents any real number. Neither value-kind constrains the numeric range or precision at the model layer. Constraints on range, width, or representation are expressed at the representation layer through suffix tags (§4.2) or at the schema layer through Templates (§8).

A conforming implementation MUST round-trip every representation-conforming numeric literal through serialization (Appendix A). An implementation that cannot represent a particular literal in its internal numeric type MUST reject the literal at parse time as a representation-conformance failure; it MUST NOT silently truncate, round, or coerce.

#### 3.3.2 Reference Value-Kind

A `reference` resource is a directional pointer to another addressable element by identifier (or by path or key, per §4.2.6). References are interpreted by a resolver, not by the GDX parser; the parser validates only that the literal is well-formed (§4.2.6). The kind of target a reference resource may point at is determined by the prefix of the bearing element — `@OBJ` and `@REL` are the @-Edge family whose resource targets an Object or a REL declaration respectively (§6.4); `%KIND` is the REF family whose resource targets an element of role KIND (§6.16).

The resolution scheme is system-defined. Common approaches include local-document resolution (the reference targets an Object within the same document), namespace-relative resolution (the reference targets an Object within a known scope), and global resolution (the reference is a fully qualified URI). The specification does not mandate any particular scheme; resolvers SHOULD attempt local resolution before broader scopes.

For the special case of @-Edge elements (`@OBJ`, `@REL`, §6.4), the Resource is always a reference value-kind, and the `#rel` field names the edge type.

#### 3.3.3 Absence and Null Values

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

The textual representation of these states — the bare-versus-empty-value forms a null element may take in serialization — is defined in §4.1.

### 3.4 Children

Children form an ordered list. Order is preserved through serialization and is significant for round-trip equivalence (Appendix A). The semantic significance of order is determined by the parent role and any Template the element claims to satisfy.

Set elements (§6.13) carry their members in document order. Set membership does not imply unordered semantics in the data-structure sense; the term refers to the multi-valued nature of the collection rather than to any ordering property.

Every child has exactly one parent. A node detached from its parent and not reattached is no longer part of the document.

---

## 4. Canonical Text Form

This section defines how GDX elements are represented as text: the syntax for writing an element, the textual markers (sigils and separators) that introduce its fields, the whitespace rules, and the literal grammars for each value-kind. The conceptual introduction is in the GDX Primer; the data-model rules an implementation enforces are in §3.

Other text forms (compact, binary, alternative tab-indented) MAY be defined elsewhere and remain conforming so long as they preserve the fields, ordering, and values defined in §3.

### 4.1 Element Syntax

An element is written as a bracketed token sequence between `<` and `>`. The general form is:

```
<Prefix [&Key] [#Rel] [Id] [: Resource] [--Type] [{Metadata}]>
    [child elements ...]
>
```

Field write order is **Prefix · Key · Rel · Id · Resource · Type · Metadata**. The Prefix is required; all other fields are optional. The metadata block, when present, is written last — after the Type — so that a reader (or a stripping tool) consumes the element's entire model content before reaching the out-of-model annotation; see §4.8.

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

#### 4.1.1 Sigils and Separators

A *sigil* is a single non-alphabetic character (or short character sequence) that introduces a field. The sigils used in field positions are:

| Sigil | Purpose |
|-------|---------|
| `&`   | Introduces the Key field |
| `#`   | Introduces the Rel field |
| `--`  | Introduces the Type field |
| `{ }` | Delimit the metadata block (§4.8) — out-of-model carriage annotation |
| `@`   | Introduces an identifier-mode reference literal (§4.2.6) |
| `$`   | Introduces a path-mode reference literal (§4.2.6); grammar specified in `GDX-Path.md` |

The character `:` is a *separator*, not a sigil. It divides an element into an identity portion (Prefix, Key, Rel, Id) and a value portion (Resource, Type). The Resource itself has no sigil; it is identified by appearing on the value side of `:`.

The sigils `&`, `#`, and `--` MUST be immediately followed by their field content with no intervening whitespace. Whitespace MAY appear before a sigil to separate it from the preceding token. Whitespace MAY appear on either side of `:`. A bare sigil — `&`, `#`, or `--` followed by anything other than its field content — is malformed; an empty field is expressed by omitting the sigil entirely (for the significance of an omitted `#rel` on an @-Edge element, see §6.4.2).

When the `:` separator appears with no identity field on the left in a context where an identifier is required, the element is malformed; conforming parsers MUST reject it. When the `--` sigil appears without a type identifier following, the element is malformed.

**Prefix-position characters** (`.`, `_`, `N`, `DEF`, `@`) and the **suffix-position character** (`*`) are not field sigils — they are part of the prefix token itself and are defined in §5.

#### 4.1.2 Whitespace Rules

Tokens within an element are separated by whitespace. The following rules apply:

- Whitespace MUST separate the Prefix from a following Id when no sigil-introduced field intervenes (so `<REC address>` not `<Raddress>`).
- The `:` separator MAY have whitespace on either side. The forms `<STR name:"Alice">`, `<STR name :"Alice">`, `<STR name: "Alice">`, and `<STR name : "Alice">` are all valid and equivalent.
- Whitespace MAY appear immediately after `<` and immediately before `>`.
- Whitespace within an element is otherwise not significant.

The canonical form used in this specification's examples places a single space between adjacent fields and around the `:` separator. Implementations producing GDX text MAY choose more compact forms (e.g., `<STR name:"Alice">`) where appropriate to context.

#### 4.1.3 Null Forms

The null state of a Property element (§3.3.3) has two equivalent textual forms:

- **Bare form** (canonical): `<STR middleName>`, `<INT age>`, `<FLOAT weight>`. The Resource section is omitted entirely.
- **Empty-value form**: `<STR middleName : >`, `<INT age : >`, `<FLOAT weight : >`. The `:` separator is present but no Resource value follows.

Both forms denote the same null state. The bare form is canonical for serialization (Appendix A.5); a conforming parser MUST accept either input form, but a conforming serializer SHOULD output the bare form. The empty-value form exists primarily to support natural human editing — when a user removes a value from a present element by deleting the literal but not the separator, the element remains valid rather than becoming malformed.

The same equivalence applies when a Type reference is present with no Resource value: `<STR name --gdx:string>` and `<STR name : --gdx:string>` are equivalent. The bare form (no `:`) is canonical.

### 4.2 Resource Literals

The Resource of an element, when present, is one of six value-kinds. Each value-kind has a defined literal form in the canonical text representation.

| Value-Kind  | Literal Form                                                             | Example                          |
|-------------|--------------------------------------------------------------------------|----------------------------------|
| `string`    | Double-quoted string with backslash escapes                              | `"Alice"`                        |
| `int`       | Decimal digits with optional sign and optional width suffix; alternative bases via `0b`/`0o` prefix | `42`, `25i8`, `0b1010`, `0o17` |
| `float`     | Decimal with point, optional width/representation suffix; or keyword     | `3.14`, `25.50d`, `1.5708f32`    |
| `bool`      | `true` or `false`                                                        | `true`                           |
| `bytes`     | Hex prefix form (canonical) or alternative encoding prefix               | `0x4a3f8c`                       |
| `reference` | At-prefixed identifier, optionally composed with a path                  | `@people/bob`, `$/people/[1]`    |

#### 4.2.1 String Literals

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

#### 4.2.2 Integer Literals

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

#### 4.2.3 Float Literals

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

#### 4.2.4 Boolean Literals

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

#### 4.2.5 Bytes Literals

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

#### 4.2.6 Reference Literals

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

The `Anchor` production carries an optional role marker — a closed-vocabulary keyword drawn from §5.2 followed by `:` — that disambiguates the role of the resolved element. The marker is most useful in composed references where the anchor's role differs from the terminal target's role; in plain `@id` references and in composed references with Object anchors, the marker is optional. When omitted in a composed reference, the anchor defaults to an Object (the most common case).

The `PathExpression` grammar and the resolution algorithms are specified in the companion GDX Path Expressions Specification (`GDX-Path.md`); the `KeyToken` grammar is defined in §4.4.3 of this document. Implementations MUST conform to `GDX-Path.md` for the contents of any `$`-introduced literal and for the resolution semantics of role markers.

**Identifier-mode** (`IdRef`) is an `@` sigil followed by an identifier:

```
@people/bob
@gdx:Person
@7f3a2b9c-4e21-4f88-91d5-abcdef012345
@https://acme.com/people/alice
```

The identifier following `@` follows the lexical rules in §4.4. The reference is resolved by the implementation's reference resolver; the parser validates only that the literal is well-formed. The `@` sigil MUST be immediately followed by the identifier with no intervening whitespace.

**Path-mode** (`PathRef`) is a `$` sigil followed by a path expression. The path traverses the structural tree of the current document, starting at the document root, addressing an element by its location:

```
$/people/alice/mailing
$/orders/[3]/#gdx:postalAddress
```

The `$` sigil MUST be immediately followed by the path expression with no intervening whitespace.

**Key-mode** (`KeyRef`) is an `&` sigil followed by a Key token. The Key is looked up within the scope of the reference (the enclosing Object, or the document for transport-layer positions; see §4.4.5):

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

Reference literals are commonly used as the Resource of @-Edge elements (`@OBJ`, `@REL`, §6.4) and of REF elements (§6.16):

```
<OBJ alice
    <@OBJ #knows : @people/bob>           ← @-edge: alice — knows → bob
    <%REC mailing : $/addresses/[1]>      ← REF: handle to a Record via path
    <%REC home    : &homeAddress>         ← REF: handle to a Record via key
>
```

**Address-mode singularity.** A reference literal whose Resource is a `PathRef`, `KeyRef`, `IdPathRef`, or `IdKeyRef` resolves to a single element. The reference is resolved if and only if path or key evaluation yields exactly one element whose role matches the enclosing prefix; zero matches, multiple matches, and a single match of non-matching role all yield an unresolved reference. Resolvers MUST NOT silently select a first match in the multi-match case. See `GDX-Path.md` §7 for the normative definition.

The five reference forms serve different addressing strategies. Identifier-mode looks up a target by name in the appropriate scope. Path-mode traverses a structural path to reach an element that may not have a global identifier. Key-mode looks up an element by its location-stable Key anchor. The composed forms combine name resolution with one of structural traversal or key lookup. The choice among them is the author's, based on what the target is, where it lives, and how stably it is identified. When both a Key and a path could address the target, prefer the Key form — Keys are location-stable, paths break under restructuring.

### 4.3 Type References

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

The `--` sigil MUST be preceded by whitespace separating it from the previous token, and MUST be immediately followed by a Type identifier with no intervening whitespace. The Type identifier follows the lexical rules in §4.4.

A Type reference asserts that the element's contents satisfy the named Template. Resolution and validation are described in §8.

### 4.4 Identifier Lexical Rules

A single identifier grammar applies to all identifier-bearing positions: the `Id` field, the contents of the `Key` field (which additionally permits a leading digit — §4.4.3), the contents of the `Rel` field, and the contents of the `Type` field.

#### 4.4.1 Identifier Grammar

```
Id              ::= IdStartChar IdContChar*

IdStartChar     ::= ALPHA | "_"
IdContChar      ::= ALPHA | DIGIT | "_" | "-" | "." | "/" | ":"

ALPHA           ::= "A".."Z" | "a".."z"
DIGIT           ::= "0".."9"
```

An identifier MUST begin with a letter or an underscore. It MAY continue with any combination of letters, digits, underscores, hyphens, periods, forward slashes, and colons.

The forward slash (`/`) is admitted as a continuation character so that IRIs (e.g., `https://acme.com/people/alice`) remain valid identifiers. The forward slash is **not** admitted as a start character: identifiers may not begin with `/`. This narrowing (introduced in v0.11.0) reserves the leading-slash syntactic slot for path-mode reference literals (§4.2.6 and the companion GDX Path Expressions Specification). Documents conforming to v0.10.0 or earlier that use leading-slash identifiers MUST be migrated; see `GDX-Path.md` Appendix C for migration patterns.

**Decision: identifiers begin with a letter or underscore.** Restricting `IdStartChar` to `ALPHA | "_"` (and the broader restriction of identifiers to ASCII letters, digits, and a small punctuation set). *Alternatives considered:* permitting a leading digit, leading sigil characters, or Unicode identifier characters. *Rejected because:* a leading digit is lexically ambiguous with numeric literals, and a leading `/` is reserved for path-mode references — but beyond those mechanical reasons, the restriction is a deliberate human-factors commitment. Identifiers are read, typed, searched, and grepped by people; keeping them in keyboard-reachable ASCII means an identifier can always be produced from muscle memory and located by a plain text search, with no character-picker and no encoding surprises. This serves the human-authorability and round-trip-readability goals (§1.2) directly. A format that admitted arbitrary Unicode in identifier-start position would trade that everyday ergonomics for a generality GDX's target domain does not need.

Identifiers MUST NOT begin with a digit (which would be lexically ambiguous with numeric literals). Identifiers MUST NOT contain whitespace, brackets, sigil characters (other than as listed above), or other punctuation.

Single-letter identifiers are valid. Position parsing disambiguates them from prefix tokens: the first non-whitespace token after `<` (and any modifier sigil) is always the prefix; subsequent unsigiled tokens in identifier position are identifiers.

#### 4.4.2 Identifier Forms

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

#### 4.4.3 Key Field Values

The `Key` field carries a **Key token**: an identifier token whose grammar additionally permits a leading digit.

```ebnf
KeyToken     ::= KeyStartChar IdContChar*
KeyStartChar ::= ALPHA | DIGIT | "_"
```

All of the following are valid anchor values:

```
<STR &1 : "Broadway">
<REC &shippingAddress
    <STR street : "Main St">
>
<REC &7f3a2b9c-4e21-4f88-91d5-abcdef012345 #log:event ...>
```

The leading-digit relaxation is unambiguous because a Key token never appears in an open lexical position: it is always introduced by the `&` sigil, whether in field position (§4.1) or in a `KeyRef`/`IdKeyRef` reference literal (§4.2.6), so the digit-start ambiguity with numeric literals that motivates the `IdStartChar` restriction (§4.4.1) cannot arise. Key tokens are opaque: they carry NO numeric interpretation and are compared by case-sensitive ordinal equality like every other identity field (§3.2). An all-digit Key is a token, not a number — `&01` and `&1` are distinct Keys.

The digit-permissive start admits the two anchor idioms that plain identifiers exclude: sequential tokens (`&1`, `&2`, … for log entries and append-only records) and digit-leading GUID/UUID tokens (`&7f3a2b9c-…`). Named Keys remain the idiom for semantic anchors (named addresses, designated child records). The Key field provides a location-stable anchor for references (§4.2.6, §4.4.5). Keys are NOT slot positions for Template-instance alignment, and the `Key` field is excluded from the universal claim rule (§8.2.3): alignment uses the `Id` field, and Keys are never Template-declared.

#### 4.4.4 Identifier Length and Collisions

The identifier grammar imposes no maximum length. In practice, identifiers should be long enough to be meaningful and short enough to keep documents readable.

Practical guidance:

- Local identifiers (Property names, Record fields) typically fit comfortably in 1-50 characters. Longer names accumulate visual weight without adding clarity.
- Hierarchical local identifiers (`orders/2026/04/29/customer-12345`) may run longer when they encode hierarchy; readability remains the constraint. Such identifiers do not begin with `/` (see §4.4.1); the leading-slash form is the path-mode reference syntax (§4.2.6).
- IRI-form identifiers may run to several hundred characters when full URIs are used. This is acceptable but compact identifiers (CURIEs, GUIDs) are preferred for documents that are read or edited by humans.
- Implementations SHOULD impose an upper limit on identifier length (a reasonable default is 1024 characters) to bound memory use and prevent denial-of-service via pathologically long identifiers in untrusted documents.

Identifier collisions across documents are an application concern, not a parser concern. Two documents using `gdx:Person` are referencing the same conceptual identifier; whether they refer to the same Template depends on the resolver's vocabulary mapping (§8.5.2). Within a single document, identifiers in the same scope MUST be unique among elements that compete for resolution (e.g., two top-level Objects with the same Id is malformed); identifiers in different scopes (different parents) are independent.

#### 4.4.5 Key Uniqueness and Scope

When an element carries a `Key` field, the Key value participates in a scope-local namespace used for `KeyRef` and `IdKeyRef` reference resolution (§4.2.6).

Key scope is determined by the element's position in the document:

- **Object-scoped.** An element whose nearest enclosing addressable container is an Object has its Key scoped within that Object. Two elements inside the same Object MUST NOT share a Key value.
- **Document-scoped.** An element at transport-layer position — a direct child of a Package, Group, or document root — has its Key scoped within the document. Two elements at this level MUST NOT share a Key value.

The enclosing-Object rule means that two different Objects MAY each contain an element with the same Key (`&entry-12345` in two different log Objects is permitted), because Key resolution within each Object is independent. The composed `IdKeyRef` form (`@anchor-id&key`) pins the scope explicitly when addressing across Object boundaries.

**Embedded Objects open a fresh Key scope.** An Object occupying a VAL slot (§6.11) is an Object for the purposes of the nearest-enclosing-container rule: elements inside it are Key-scoped to the embedded Object, not to the host Object. The embedded Object's own Key, if it carries one, belongs to the host's scope, exactly as any other child of the host.

**Lookup consults exactly one scope.** A `KeyRef` is resolved against the single scope determined by the rules above — the scope of the *referencing* element. Resolution MUST NOT fall back to an enclosing scope when the Key is not found; there is no scope chain and no shadowing. A bare `&key` inside an embedded Object cannot see the host Object's Keys; addressing across the boundary uses the composed `IdKeyRef` form.

**Default anchor role in composed references.** In a composed reference (`IdPathRef`, `IdKeyRef`), if the `@Anchor` portion carries no role marker, the resolved element MUST be of role Object. This default reflects the common case for cross-document addressing. To anchor a composed reference at a non-Object element (a REL, a Template, a Package, or a Group), the author MUST use an explicit role marker (§4.2.6).

Key collisions within the same scope make the document malformed; conforming parsers SHOULD emit a diagnostic, and conforming resolvers MUST NOT silently pick one matching element over another.

All Key tokens — named, all-digit, and GUID-form alike (§4.4.3) — participate in the same scope rules and compare by ordinal equality; `&01` and `&1` are distinct Keys.

Keys are OPTIONAL. An element MAY omit the Key field entirely; in that case the element is addressable only by `Id` (if present) or by structural path. The decision to carry a Key is the author's, based on whether the element needs to be a stable reference target.

### 4.5 Whitespace Between Elements

§4.1.2 covers whitespace within an element. This section covers whitespace between elements.

The canonical form for single-line elements uses a single space between adjacent tokens and around the `:` separator. Multi-line elements indent children for readability. The amount of indentation is implementation choice; conforming implementations preserve element structure but MAY normalize whitespace during round-trip.

Newlines, carriage returns, and tabs are equivalent to spaces for the purpose of token separation. A line break does not terminate an element; element termination is signaled exclusively by the matching `>` bracket.

### 4.6 Comments

GDX has no comment syntax in the canonical text form. Documentation that should travel with the data is expressed through Annotation elements (`REM`, §6.6). Documentation that should not travel with the data is expressed in surrounding application or build context, not in the GDX document itself.

### 4.7 Use-Case Adaptability

The same text form scales from typed key-value data to full graph-and-schema use without changing format or parser: applications engage only the features they need. This is a property of the form, not a maturity progression. The worked simple-to-rich progression that illustrates it is introductory rather than normative and is presented in the introductory material and the GDX Cookbook (companion specification) rather than here.

### 4.8 Metadata

An element MAY carry a **metadata block** — a brace-delimited sequence of flat key/value pairs written last in the element, after the Type. The block records *realization-layer* information about how the element is stored or carried — a persistence identifier, a stream instance identifier, delta-handling state, and the like — as distinct from the element's model content, which is what the element *means*.

```
<STR name : "Bob" {sid = 455, pid = 6, porthol:rev = "12:34.54"}>
```

#### 4.8.1 Out-of-Model Status

The metadata block is **not model content**. This is the defining property of the channel, and it is a Layer 1 structural fact — decidable by the parser, requiring no vocabulary knowledge (contrast Extensions, §6.10, which are model content). Concretely:

- Metadata MUST NOT participate in model equality or canonical comparison (§A.5). Two documents that differ only in their metadata blocks carry the **same model** and are model-equal.
- A processor MAY add, modify, or strip metadata pairs at any point without affecting any conformance verdict (§A.1) the document earns. Stripping every metadata block yields a document with identical model content.
- Metadata is realization-layer annotation in the sense of §1.1: it belongs to a particular carriage of the element (a database row, a stream frame), not to the abstract element that survives across realizations. The database field table of §10.2 is the same information at rest; the metadata block is its serialization when the element travels.

This out-of-model status is *why* the channel is deliberately impoverished (§4.8.3): anything rich enough to need structure, references, or semantic typing is model content and belongs in an Extension (§6.10), a Record, or — for carriage of the whole package — transport-layer metadata on a Package or Group (§7.1).

#### 4.8.2 Grammar

```ebnf
Metadata     ::= "{" ( MetaPair ( "," MetaPair )* )? "}"
MetaPair     ::= MetaKey "=" MetaLiteral
MetaKey      ::= Identifier
MetaLiteral  ::= StringLiteral | IntLiteral | FloatLiteral
               | BoolLiteral   | BytesLiteral
```

- **Keys** are identifiers (§4.4.1); the CURIE form (`porthol:sid`) is admitted for namespacing and lexes by the same rule that admits `gdx:name` in element positions (`:` is an `IdContChar`; the pair separator `=` is not, so the key/value boundary is unambiguous even without surrounding whitespace).
- **Values** are Resource literals (§4.2) of the five value-kinds — string, integer, float, boolean, bytes. Reference literals are NOT admitted (metadata addresses nothing). A value MUST be a literal: a bare unquoted token that is not a recognized literal keyword is malformed, not an implicit string. Strings are quoted or they do not appear — this is what keeps the channel free of implicit-typing ambiguity.
- **Separators.** `=` separates a key from its value; `,` separates pairs. Whitespace MAY appear around either. The `=`/`,` pairing (rather than `:`/whitespace) is deliberate: it visually marks the block as annotation rather than element structure, and avoids adjacency between the CURIE `:` and a separator `:`.
- **Value-kind** is carried by the literal form exactly as in Resource position (§4.2): quoted → string, bare digits → integer, decimal-point/exponent/suffix/keyword → float, `true`/`false` → boolean, `0x`/`0z` → bytes. No prefix token is written; the pair is not an element and takes no role.

An empty block `{}` is permitted and is equivalent to no block. Duplicate keys within one block are malformed; conforming parsers SHOULD emit a diagnostic. Key comparison is case-sensitive ordinal, as for all identifiers (§3.2).

#### 4.8.3 What Metadata Is Not

The channel carries **representational** typing (the literal form) and explicitly no **semantic** typing. A metadata value has no `--type`, satisfies no Template, and participates in no vocabulary: `porthol:rev = "12:34.54"` travels as a string, and the consumer that owns the `rev` key supplies its interpretation — exactly the contract of an HTTP header value. The block admits no nesting, no elements, no references, and no Sets. These are not omissions to be filled by a later revision; they are the boundary that makes “this is not data” credible. A need for any of them is the signal that the information is model content and belongs in the model.

#### 4.8.4 Positions and Persistence

A metadata block MAY appear on any element in any position, including leaf Properties, Records, Objects, @-edges, and REFs. It is written after the Type field and, on a non-leaf element, before the opening of the child list.

```
<REC address {sid = 9021}
    <STR street : "Main St">
    <INT number : 100 {pid = 6}>
>
```

Because metadata is out-of-model, a realization is not required to carry it in order to be model-faithful (§1.1): a conversion that drops the block still preserves the model. Realizations SHOULD carry the channel and conversions SHOULD preserve it, but neither is required for model conformance. A pipeline that depends on metadata surviving — delta streaming, for instance — selects realizations that carry it; it does not rely on the model-conformance contract to guarantee preservation. The signing consequence follows directly: the canonical form (§A.5) excludes metadata, so a signature over canonical content covers model content only. An application that must sign carriage information signs the realization octets (the stored row, the stream frame), not the canonical form.

---

## 5. Prefix Vocabulary

An element's **role** is its position and interpretation in the data model. The role declares both what the element is structurally (what fields it carries, what children it admits) and how it is to be read conceptually (an attribute of its parent, a thing with identity, a graph statement, a refinement, and so on). Two roles may share a structural shape and still differ as roles because their conceptual interpretation differs — Object and Record are the load-bearing example (§6.1, §6.2). The role is declared by the element's prefix; the closed set of roles is enumerated by the closed set of prefixes. This section introduces the prefix vocabulary in three layers — the base roles (§5.2), the set suffix and reference operator (§5.3), and the refinement modifiers (§5.4). Detailed per-role treatment is in §6 (Element Roles), where the structural and conceptual aspects of each role are both stated.

The prefix vocabulary is **closed** and committed against future change. Implementations MUST NOT introduce new prefixes; extensibility is provided through Templates (§8), Extensions, and the predicate vocabulary mechanism. New primitive types, new structural roles, or new domain concerns are expressed by composing existing prefixes with Templates, Extensions, and vocabulary identifiers — never by adding a new prefix.

**Decision: the prefix vocabulary is closed.** The set of prefixes is fixed by this specification; extensibility is pushed out to Templates, Extensions, and vocabulary identifiers. *Alternative considered:* an open or extensible prefix set, letting applications or vocabularies mint new prefixes for new roles or types. *Rejected because:* a closed prefix set is what lets any conforming consumer parse and structurally validate any GDX document without first loading a vocabulary — the structural schema is fully known from the specification alone (§1.2, structural-schema-with-pluggable-constraints). An open prefix set would make structural validity vocabulary-dependent: a parser could not know whether an unrecognized prefix was malformed or merely from a vocabulary it had not loaded, collapsing the clean Layer 1 / Layer 2b boundary (Appendix A.1). Domain extension is real and necessary, but it belongs in the extension layer (Templates and vocabulary identifiers GDX records but does not interpret), not in the structural core that every consumer must agree on.

A prefix in GDX is composed according to a two-layer pattern:

```
[%?] [@?] [refinement-modifier?] [base] [*?]      (layer 1: role-prefix)
%  + [layer-1 role-prefix]                         (layer 2: REF wrapper)
```

Where (layer 1, the role-prefix):
- **base** is one of the core role prefixes (§5.2)
- **`*` suffix** marks the element as a Set (§5.3.1)
- **`@` prefix** marks the element as a member of the @-Edge family (§5.3.2); restricted to two members (`@OBJ`, `@REL`)
- **refinement-modifier** is one of `.`, `_`, `N` (§5.4) or the `DEF` declaration-construct (§6.14)

And (layer 2, the REF wrapper):
- **`%` prefix** wraps any layer-1 role-prefix to produce a `%KIND` REF — a slot-filler whose Resource is a typed handle to a KIND-element (§5.3.3, §6.16).

The layer-1 slots are independent. The base is required; the others are optional. Day-to-day GDX use involves the base prefixes plus the `*` suffix when a collection is needed. The `@` sigil constructs @-edges; the `%` sigil constructs REFs; the refinement modifiers construct Qualifiers, Extensions, Node-zone elements, and Templates. Three compositions are malformed: `@@KIND` and `@OBJ*`/`@REL*` (Edge-side; §5.3.2), `%%KIND` and `@%KIND` (REF/Edge-side; §5.3.3).

### 5.1 Canonical and Minified Forms

GDX prefix tokens are textual renderings of underlying type tags. The same data model is expressible in two equivalent textual forms:

- **Canonical form** uses three-letter (or longer where natural) prefix names. Canonical is the default for authoring, storage, content addressing, and inter-system exchange where readability matters.
- **Minified form** uses single-letter prefix names. Minified is for transport-efficiency contexts (high-frequency machine-to-machine traffic, embedded systems, size-constrained storage) where every byte matters.

Both forms are equivalent at the data-model level. Conversion is deterministic; round-tripping a document through canonical and minified forms preserves the data exactly.

**Recognition rules:**

- Implementations MUST recognize both canonical and minified forms on input.
- Prefix tokens are case-insensitive. `STR`, `Str`, `str`, and `sTR` are all recognized; the canonical case for output is uppercase.
- Mixed forms within a document are accepted on input. The serializer normalizes to the configured output style.
- Identifiers (Id, Key, Rel) remain case-sensitive (§4.4). The case-insensitive rule applies only to prefix tokens at prefix position.

**Form is style, not data.** Round-trip preserves data and configured output style, not authorial form choices. Tools serializing in canonical form produce uppercase three-letter prefixes regardless of input case or form.

**Strict canonical for content addressing.** The strict canonical form (Appendix A.5), used for content addressing and signing, uses canonical prefix names. Minified output is valid GDX but does not produce strict canonical bytes.

**Prefixes without minified forms.** Some prefixes are canonical-only — they are already three or more letters and represent infrequent declarative items where minification would save little: `REL`, `REM`, `PKG`, `BLK`, `GRP`, and all `DEF…` Template prefixes. These are identity-mapped between canonical and minified output: a minified document still contains `<REL gdx:knows>` and `<DEFREC Customer>`.

### 5.2 Base Vocabulary

The base prefixes are the core role vocabulary. They group into four categories:

**Value primitives** — typed scalar values:

| Canonical | Minified | Role     | Resource value-kind | Detail   |
|-----------|----------|----------|---------------------|----------|
| `STR`     | `S`      | Property | `string`            | §6.3   |
| `INT`     | `I`      | Property | `int`               | §6.3   |
| `FLOAT`   | `F`      | Property | `float`             | §6.3   |
| `BOOL`    | `L`      | Property | `bool`              | §6.3   |
| `BYTE`    | `B`      | Property | `bytes`             | §6.3   |

**Structural roles** — composite entities and organization:

| Canonical | Minified | Role   | Resource value-kind | Detail   |
|-----------|----------|--------|---------------------|----------|
| `OBJ`     | `O`      | Object | —                   | §6.1   |
| `REC`     | `R`      | Record | —                   | §6.2   |
| `BLK`     | —        | Block  | —                   | §6.5   |
| `VAL`     | `V`      | VAL (element-as-value slot) | — (occupant is a child) | §6.11 |

**Graph-statement roles** — first-class directed relationships (the `@` sigil):

| Canonical | Minified | Role            | Resource value-kind | Detail |
|-----------|----------|-----------------|---------------------|--------|
| `@OBJ`    | —        | @-Edge (Object-targeted)  | `reference` (Object) | §6.4 |
| `@REL`    | —        | @-Edge (REL-targeted)     | `reference` (REL)    | §6.4 |

The @-Edge family has exactly two members (§6.4 axiom E5). The `EDGE` and `E` prefixes used through v0.11.0 were removed in v0.12.0; the `E` minified slot is currently unassigned. The `@OBJ*` and `@REL*` Set forms are malformed — each @-edge is a single graph statement; multiple edges from one source are expressed as siblings.

**Slot-filling reference role** — typed handles to independently-existing elements (the `%` sigil):

| Canonical | Minified | Role | Resource value-kind | Detail |
|-----------|----------|------|---------------------|--------|
| `%KIND`   | —        | %-Reference (REF) | `reference` (to KIND) | §6.16 |

The REF family is generated by a single composition rule: `%` wraps any layer-1 role-prefix to produce a REF-to-that-role. The closed family covers every role that can be a reference target — `%OBJ`, `%REC`, `%BLK`, value-Property forms (`%STR`, `%INT`, etc.), refinement-modifier forms (`%.STR`, `%_INT`, `%NSTR`), Templates (`%DEFOBJ`, `%DEFSTR`, etc.), `%VAL`, the reification forms (`%@OBJ`, `%@REL`), the transport wrappers (`%PKG`, `%GRP`), and Set forms (`%KIND*`, only for bases that admit Sets — see §5.3.3). The compositions `%%KIND`, `@%KIND`, and `@@KIND` are malformed. See §5.3.3 and §6.16.

**Declaration roles** — vocabulary and schema declarations:

| Canonical | Minified | Role                 | Detail   |
|-----------|----------|----------------------|----------|
| `REL`     | —        | Relation Declaration | §6.15  |
| `DEF…`    | —        | Template (see §6.14) | §6.14 |

**Transport roles** — exchange-layer wrappers:

| Canonical | Minified | Role    | Detail   |
|-----------|----------|---------|----------|
| `REM`     | —        | Annotation | §6.6 |
| `PKG`     | —        | Package | §6.7   |
| `GRP`     | —        | Group   | §6.8   |

### 5.3 Special Operators

Two operators compose with the base vocabulary to produce new roles. They are structurally distinct from the refinement modifiers (§5.4) because they change what value-kind the resulting element carries.

#### 5.3.1 The Set Suffix `*`

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

The `*` suffix MAY combine with refinement modifiers: `.STR*` is a Set of string Qualifiers, `_INT*` is a Set of integer Extensions, and so on. See §5.4.

Detailed treatment of Sets, including empty-vs-absent semantics, is in §6.13.

#### 5.3.2 The Sigil `@` — @-Edge Family

The `@` prefix marks an element as a member of the **@-Edge family** (§6.4) — a first-class graph-statement element whose Resource is a reference to an independent graph-entity and whose `#rel` predicate asserts a relationship the parent has to that target. The family has exactly two members:

- `@OBJ` — @-edge to an Object (data-Objects, including concept-Objects per §6.15)
- `@REL` — @-edge to a REL declaration (predicate-to-predicate or REL-to-concept-Object)

@-edges carry an associative `#rel` specifying the relationship's nature — the field MAY be empty, leaving the relationship undefined pending refinement (§6.4 axiom E4, §6.4.2) — and MAY carry attribute children, identity (`Id`/`Key`), Qualifiers, Extensions, and Annotations.

```
<@OBJ #knows : @bob>                  ← @-edge: alice knows bob
<@OBJ edge-1 #knows : @bob            ← @-edge with identity and attributes
    <STR since : "2024-01-15">
>
<@REL #gdx:inverse : @gdx:knownBy>    ← @-edge: predicate-to-predicate
```

**Decision: the `@` sigil marks the @-Edge family.** When the v0.12–v0.13 revisions split the link constructs, the `@` sigil — previously the reference operator — was reallocated to edges, and the leaf-like reference family moved to `%`. *Alternative considered:* keeping `@` on references and minting a new sigil for edges. *Rejected because:* three independent grounds point the same way. **Origin** — GDX began with the edge as its only link construct; the reference family was a later accretion, added to handle REL-to-REL linking before role-explicit edges existed, and the prime symbol belongs to the original primitive. **Frequency** — the primary GDX use cases are edge-shaped: REL-to-REL ontology axioms (inverse, subproperty, equivalence) and Object-to-Object labeled property graphs; value-resolution is the minority specialist. **Centrality** — edges are central to what GDX is (a property graph with a light ontology layer); references are a supporting tool. The reallocation restores `@` to the primary construct after a period in which the reference family had been stretched to do edge-shaped work; recording the grounds here is what keeps the reallocation from reading as arbitrary churn.

Three compositions are malformed:

- **`@@KIND`** — no @-edge-to-@-edge directly. To reify a specific @-edge from outside, use a `%@OBJ` or `%@REL` REF with the edge's key (see §5.3.3 and §6.16).
- **`@OBJ*`** and **`@REL*`** — no Set-of-@-edges. Each @-edge is a single graph statement (§6.4 axiom E2′); multiple edges from one source are expressed as siblings, not as Set members.
- **`@%KIND`** — no @-edge-to-REF. @-edges target only independent graph-entities (§6.4 axiom E5); REFs are slot-contents, not graph-entities.

Detailed treatment of the @-Edge family is in §6.4.

#### 5.3.3 The Sigil `%` — REF Family

The `%` prefix marks an element as a member of the **%-Reference family** (§6.16) — a slot-filler whose Resource is a typed handle to an independently-existing element. The `%` sigil wraps any layer-1 role-prefix to produce a REF-to-that-role:

```
% [layer-1 role-prefix]
```

where the layer-1 role-prefix follows the established pattern `[@?][refinement-modifier?][base][*?]`. The closed REF family is generated by this single composition rule:

- `%OBJ`, `%REC`, `%BLK` — REF to an Object, Record, Block
- `%STR`, `%INT`, `%FLOAT`, `%BOOL`, `%BYTE` — REF to a value Property
- `%.STR`, `%_INT`, `%NSTR` — REF to a Qualifier, Extension, Node-zone element
- `%DEFOBJ`, `%DEFSTR`, etc. — REF to a Template
- `%VAL` — REF to a VAL slot
- `%@OBJ`, `%@REL` — REF to an @-edge (reification)
- `%PKG`, `%GRP` — REF to a transport wrapper (meaningful within the transport context that carries it; see §6.16)
- `%STR*`, `%REC*`, `%.STR*`, and the other Set forms — REF to a Set element. Valid only where the wrapped Set prefix itself exists (value primitives, REC, and modifier Sets per §5.3.1/rule 13); `%DEF…*`, `%REL*`, `%OBJ*`, and `%@OBJ*` are malformed — their target Sets cannot exist (rules 13, 14a; Templates and RELs are root-level declarations, a Set of them does not make sense)

REFs MAY carry a `#rel` (compositional — aligns the slot to a vocabulary predicate; see §6.16 axiom R4), Qualifiers, Extensions, an `Id` (slot name), and Annotations. They MUST NOT carry structural children — that would make a REF a substance rather than a handle.

```
<%OBJ database : @primary-db>                ← REF: handle to an Object
<%DEFOBJ #conformsTo : @ServerConfigV2>      ← REF: handle to a Template, slot aligned
<%REC mailing : @alice$/addresses/[1]>       ← REF: handle to an interior Record
<%@OBJ source : @alice&knows-bob-2024>       ← REF: handle to an @-edge (reification)
```

Two compositions are malformed:

- **`%%KIND`** — no REF-to-REF. A REF is not a reference target (§6.16 axiom R6); to point at what a REF points at, write the address of the target directly.
- **`@%KIND`** — no @-edge-to-REF, as noted in §5.3.2.

The `@` and `%` sigils mark distinct genera and are never synonyms: `@` asserts a relationship (associative `#rel`); `%` fills a slot (compositional `#rel`). See §6.16 axiom R4 for the genus split and the proof-by-collapse.

Detailed treatment of the REF family is in §6.16.

### 5.4 Refinement Modifiers

Three refinement modifiers compose with value primitives (and in some cases Records) to produce elements that carry the base's value-kind but function in a refined contextual role. They do not change what value an element carries; they change what the element *does* with that value.

| Modifier | Name      | Role                                                  | Composes with               |
|----------|-----------|-------------------------------------------------------|-----------------------------|
| `.`      | Qualifier | Refines the meaning of the parent element             | value primitives            |
| `_`      | Extension | Carries a vocabulary-defined adjunct assertion        | value primitives            |
| `N`      | Node-zone | Marks a semi-structured subtree                       | value primitives, REC       |

Examples:

```
<FLOAT temperature : 21.5
    <.STR unit : "celsius">             ← Qualifier refining what 21.5 means
>

<DEFSTR Email
    <_STR pattern : "^[^@]+@[^@]+$">    ← Extension carrying a constraint
>

<NREC audit                             ← Node-zone container for ad-hoc data
    <NSTR source : "import">
>
```

**Embedded values use the VAL role**, not a refinement modifier. To place a fully first-class element in value position, wrap the occupant in a VAL slot (§6.11). This is a base role rather than a modifier because the embedded value is itself an element with its own role; VAL provides the slot, the occupant carries its own prefix.

**Refinement modifiers do not stack with each other.** `*.STR`, `.NSTR`, `N.STR`, `_NSTR` and similar combinations are all malformed. Each element carries at most one refinement modifier.

**Refinement modifiers MAY combine with `*` and `%`.** `.STR*` is a Set of Qualifiers; `_STR*` is a Set of Extensions. The REF sigil `%` may wrap any modifier-base composition: `%.STR`, `%_BOOL`, `%NSTR` (§5.3.3). The `@` sigil does NOT combine with refinement modifiers — the @-Edge family is closed at `@OBJ` and `@REL` (§5.3.2, rule 14). The pattern `[%?][@?][refinement-modifier?][base][*?]` governs which combinations are well-formed, subject to the family restrictions of §5.3.

**The `DEF` declaration construct** (§6.14) shares the prefix-position slot with the refinement modifiers but is structurally different — it declares a Template rather than refining an element's contextual role. See §6.14 for details.

---

## 6. Element Roles

This section is the authoritative reference for GDX's closed role vocabulary. Each subsection covers one role, its prefix(es), composition rules, identifier requirements, structural and conceptual identity, and worked examples. Roles are introduced syntactically in §5 (Prefix Vocabulary); this section defines what each role *means* and how it behaves.

The closed enumeration of roles is: Object, Record, Property, Block, Annotation, Package, Group, Qualifier, Extension, VAL, Node, Set, Template, Relation Declaration, the @-Edge family (`@OBJ`, `@REL`), and the REF family (every `%KIND` form). Composition rules between roles — what each may contain and what may contain it — are defined in §7 (Composition Axioms).

### 6.1 Object — `OBJ`

| Canonical | Minified | Resource? | Identifier |
|-----------|----------|-----------|------------|
| `OBJ`     | `O`      | —         | required (resource identifier) |

An Object is a **thing with identity** — an independently-identified entity that stands on its own and may be referenced from other elements. Conceptually, an Object is the kind of element that *other* elements refer to: a Person, an Order, a Server, a Document. It is the unit of persistence and exchange, and its identifier (a path, CURIE, GUID, IRI, or other globally meaningful identifier) is what makes that referenceability possible. The specification places no constraint on the identifier scheme.

Structurally, an Object is a container of named children. The root of a GDX document MUST be one or more Objects, Templates, REL declarations, or Packages. Objects may not contain other Objects except as the occupant of a VAL slot (§6.11) or as transport-layer payload (§6.7, §9).

```
<OBJ people/alice
    <STR firstName : "Alice">
    <INT age : 24>
>
```

### 6.2 Record — `REC`

| Canonical | Minified | Resource? | Identifier |
|-----------|----------|-----------|------------|
| `REC`     | `R`      | —         | required (local) when in named-attribute position |

A Record is a **compound property** — a named composite that belongs to its parent's structure rather than standing on its own. Conceptually, a Record is to its parent what a struct or nested object is to a programming-language container: it represents "the parent has an X of shape Y," where Y is a collection of named properties. A Record contributes to its parent's identity rather than carrying its own.

Structurally, a Record is a container of named children, admitting Properties, Sets, Edges, nested Records, Qualifiers, Nodes, and Annotations. A Record's identifier is a local identifier, meaningful within its containing scope — it names the slot the Record fills in its parent, not a globally addressable resource. A Record *contains* its children, in contrast to a Block which merely *labels* a region of children (§6.5). Records may not contain Objects (except as the occupant of a VAL slot, §6.11) or Groups.

**Records and Objects share structural shape** (both are composites of named children) but differ conceptually: an Object stands as an identified thing; a Record belongs to its parent as a compound property. The distinction matters for addressability (Objects are globally referenceable by Id; Records require path or key composition to reach — see §4.2.6), for persistence (Objects are the unit of storage; Records are stored as part of their containing Object), and for what may root a document (Objects may root; Records may not).

```
<REC address
    <STR street : "Main St">
    <INT number : 100>
>
```

### 6.3 Property — `STR`, `INT`, `FLOAT`, `BOOL`, `BYTE`

A Property is an **attribute of its parent** — a single typed value that participates in the parent's structure. Conceptually, the Property's Id names the attribute; the Resource carries its value; the parent owns the Property as part of its content. Properties do not stand on their own — like Records, they belong to their containing element rather than being addressable as independent entities. Properties are GDX's leaves: the smallest carriers of typed information.

Structurally, value primitives carry typed scalar values as their Resource. The Resource value-kind MUST match the prefix family.

| Canonical | Minified | Resource value-kind |
|-----------|----------|---------------------|
| `STR`     | `S`      | `string`            |
| `INT`     | `I`      | `int`               |
| `FLOAT`   | `F`      | `float`             |
| `BOOL`    | `L`      | `bool`              |
| `BYTE`    | `B`      | `bytes`             |

Properties require an identifier when in named-attribute position and may carry Annotations, Qualifiers, Extensions, and Nodes — but not other structural children. The three-state semantics (Absent, Null, Present-with-value) are defined in §3.3.3.

```
<STR name : "Alice">
<INT age : 24>
<FLOAT weight : 5.7>
<BOOL active : true>
<BYTE contentHash : 0xe3b0c44298fc1c14>
```

### 6.4 The @-Edge Family — `@OBJ`, `@REL`

| Canonical | Minified | Resource value-kind | Identifier |
|-----------|----------|---------------------|------------|
| `@OBJ`    | —        | `reference` (target: Object) | `#rel` OPTIONAL (empty = undefined nature, §6.4.2); `Id`/`Key` OPTIONAL |
| `@REL`    | —        | `reference` (target: REL)    | `#rel` OPTIONAL (empty = undefined nature, §6.4.2); `Id`/`Key` OPTIONAL |

An **@-edge** is a **first-class but dependent relationship** — a persistent entity owned by and serialized within its source element, asserting a directed link to an independent graph-entity that exists elsewhere. The prefix asserts the *target's role*; the Resource asserts the *target's identity*; the `#rel` predicate, when specified, asserts what the relationship currently means (an empty rel leaves the relationship undefined as to nature — §6.4.2); an optional `Id` or `Key` makes the relationship itself a trackable, reference-able substance; optional attribute children describe that substance.

@-edges are GDX's labeled-property-graph mechanism. They are how a document carries the assertion that one graph-entity stands in some named relation to another, optionally with attributes refining the relation.

**Family enumeration.** The @-Edge family has exactly two members: `@OBJ` (target is a data-Object, including concept-Objects) and `@REL` (target is a predicate, i.e., a REL declaration). The membership is restricted to *independent graph-entities*. Targets that are not independent graph-entities — Templates (vocabulary-layer artifacts), value Properties (interior to their parent), Qualifiers/Extensions/Annotations (refinement-only), VAL slots (a container, not an entity), Records/Blocks (composite parts of an Object), or any element addressed by path or key into another element's interior — are not @-edges; pointing at such targets is the job of the Reference family (§6.16).

**Both endpoints are role-explicit.** The *source* endpoint of an @-edge is its containing element, whose role is self-declared by its own prefix; the *target* endpoint's role is declared by the @-edge prefix and verified at resolution (E1). No endpoint's role is ever discovered by resolution — the declared-not-discovered discipline holds at both ends, and any source/target role combination admitted by the composition matrix (§7.1) is expressible (Object-to-Object data graphs, REL-to-REL predicate axioms, REL-to-concept-Object domain/range statements, and so on). This is also why @-edges connect whole, self-declaring elements only: a `$path`- or `&key`-addressed interior has no role-declaring definition of its own for the endpoint claim to verify against, so interior addressing is value-resolution territory (REF, §6.16), never edge territory.

```
<OBJ alice
    <@OBJ #knows : @bob>                              ← simple edge: alice — knows → bob
    <@OBJ employmentEdge #worksFor : @com.acme        ← edge with identity and attributes
        <STR startDate : "2024-01-15">
        <FLOAT confidence : 0.95>
    >
>

<REL gdx:knows
    <@REL #gdx:inverse  : @gdx:knownBy>               ← predicate-to-predicate edge
    <@OBJ #gdx:domain   : @gdx:Person>                ← predicate-to-concept-Object edge
    <@OBJ #gdx:range    : @gdx:Person>
>
```

#### Axioms (E1–E6)

**E1 — Prefix asserts target role; Resource asserts target identity.** The `@KIND` prefix declares the role the resolved target MUST occupy: `@OBJ` declares an Object target; `@REL` declares a REL target. The Resource is a reference literal (`@identifier` form per §4.2.6) naming the target. Resolution checks both — the named target MUST exist AND MUST be of the declared role. The prefix asserts the target-role even when the Resource is empty: `<@OBJ #knows : >` in Template context declares "a slot for an Object-edge predicated `knows` belongs here"; in instance context, the empty Resource represents a null edge (per §3.3.3). The prefix is the type assertion; it stands whether or not a Resource is present.

**E2′ — First-class but dependent.** An @-edge is its own entity in the data model — it can carry identity, predicate, and attribute children — but it does not exist independently of its source: it is owned by and serialized within the source element. The edge *relates* its source to its target; it does NOT contain either. An edge dies with its source (delete the source and the edge is gone; relational projections SHOULD model this with `ON DELETE CASCADE` on the source side). An edge dangles if its target is removed (the edge persists as a structurally well-formed element with an unresolved reference; the consumer decides how to handle dangling). This distinguishes @-edges from VAL: VAL *contains* its value as an occupant; an @-edge *points at* its target.

**E3 — Attribute children describe the relationship-entity.** Children of an @-edge — Properties (`STR`, `INT`, …), Qualifiers, and other adjunct elements — describe the edge itself: when it started, its confidence, who recorded it, and so on. When the edge carries identity (per E6), those attributes belong to the persistent edge-entity, not to any single predication; an edge whose `#rel` is rewritten retains its attribute children unless those attributes are also rewritten. Attributes are properties of the edge-as-substance, not of the current predication.

**E4 — The role asserts existence; the `#rel` field specifies nature and MAY be empty.** An @-edge is, by its role, the assertion that a relationship *exists* between its source and its target. The `#rel` field specifies *what* that relationship is; when present, it is associative (it asserts the parent-to-target meaning), and it MAY be empty. An edge with no `#rel` parses to a null Rel, is well-formed, and denotes a relationship **undefined as to nature** — to be refined later by filling the field on the same edge (§6.4.2). An empty rel is therefore never ambiguous about whether a relationship exists; the role settled that. Since the v0.13 sigil partition, the construct discrimination this field once carried lives in the sigil itself: `@`-prefixed elements are @-edges and `%`-prefixed elements are REFs, with or without a `#rel`. The empty rel is not an idiom for *permanent* type-free adjacency — empty means "not yet specified," and a relationship whose intended meaning is bare adjacency is expressed with a designated predicate value (e.g., `<@OBJ #gdx:adjacent : @target>`).

**E5 — Family is enumerated by target role and restricted to independent graph-entities.** The family consists of exactly `@OBJ` and `@REL`. The restriction is principled: only Objects (including concept-Objects, since GDX uses concept-as-Object for ontology layering — see §6.15) and REL declarations are independent graph-entities. Source role is orthogonal to membership: any role admitted as a parent of an @-edge by the composition matrix (§7.1) may carry @-edges; the family enumeration is over targets only. The Set forms `@OBJ*` and `@REL*` are **malformed** — each @-edge is a single graph statement (per E2′); multiple edges from one source are expressed as siblings, not as Set members. To address multiple edges as a collection (e.g., for iteration or query), the parent Object's children are filtered by prefix — the structure is implicit in the document, not collected into a Set.

**E6 — Identity is optional and orthogonal to predicate.** An @-edge MAY carry an `Id` and/or a `Key`. Identity is the edge's *substance* — the persistent entity (per E2′) that endures across changes to predicate, attributes, or target, and that disambiguates multiple edges between the same endpoints. The substance/predication distinction is the spine of the model: the `Id` is what the edge *is* (the enduring thing); the `#rel` is what it currently *asserts*; attribute children ride on the substance. Identity with an empty rel is well-formed and is the canonical capture-first pattern (§6.4.2 — the substance exists, the predication is pending); predicate-without-identity is the common case (most edges are content with anonymous substance). Two @-edges that share the same source, predicate, and target ARE distinct edges if they carry different identities; uniqueness is by identity, not by the (source, predicate, target) triple.

**Resolution.** An @-edge's Resource is name-addressed only — `@identifier` per §4.2.6. Bare `$path` and `&key` reference literals address interior elements of other Objects and are handled by the Reference family (§6.16), not by @-Edges. The composed `@Anchor$PathExpression` and `@Anchor&Key` forms similarly address interior locations and are Reference territory. An @-edge resolves to a single named entity; interior-reaching is by definition not an @-edge concern.

**Predicate-to-predicate and concept-Object edges.** @-edges from a REL to another REL (vocabulary metadata: inverse, subPropertyOf, equivalentProperty) and from a REL to a concept-Object (domain, range) are first-class edges. They are how a vocabulary document expresses the predicate-axiom layer (§6.15) without leaving the same element form used for everyday data. The substance/predication split applies uniformly: a `<@REL eid #gdx:inverse : @gdx:reportsTo>` edge with identity `eid` is a trackable assertion about predicate relationships, addressable from other documents and editable independently of the predication it currently carries.

**Composition.** Per §7.1, @-edges are admitted as children of Object, Record, Block, Edge (recursion: an edge's attribute children may themselves be edges), VAL (when the slot's value is an edge-shaped element), Template (declaring an edge slot), and REL (carrying vocabulary metadata about the predicate). Sources of @-edges are determined by where they are admitted; targets are determined by the prefix.

**The substance/predication split as the spine.** Identity is the substance; `#rel` is the predication; attributes ride on the substance. This pattern keeps GDX out of RDF-style reification machinery (no `rdf:Statement` indirection, no quadruple-form rewriting) while delivering everything reification provides: named edges, predicate mutation over stable identity, attribute attachment to the edge rather than to the predication, and multi-edge disambiguation between the same endpoints. All of this follows from E2′ (first-class but dependent) with no new structural machinery.

#### 6.4.1 @-Edge Reference Resolution

An @-edge's Resource is a reference to a target entity (Object for `@OBJ`, REL for `@REL`). Reference resolution determines whether the named target exists and what entity it identifies.

The default resolution order is normative:

1. **Local document.** Identifiers declared within the same document as the @-edge are checked first. An `@bob` reference from an `@OBJ` edge is satisfied by an `<OBJ bob>` declaration in the same document if one exists.
2. **Loaded document context.** When the @-edge appears in a document being processed alongside other documents (e.g., in a batch import or registry context), entities from those documents are checked in the order they were loaded.
3. **Globally-resolvable IRIs.** References expressed as full IRIs (e.g., `@https://acme.com/people/alice`) MAY be resolved by a network-aware resolver if the implementation supports it. Implementations are not required to perform network resolution.
4. **Resolution failure.** If no scope yields a target, the reference is unresolved.

The resolver MUST verify that the resolved entity's role matches the @-edge prefix: an `@OBJ` reference resolving to a REL is a role mismatch and yields an unresolved reference; an `@REL` reference resolving to an Object is similarly malformed.

When an @-edge reference cannot be resolved:

- The @-edge is reported as schema-non-conforming (Layer 2). The @-edge itself is structurally well-formed (Layer 1); it is the schema-level claim that the reference points to an existing entity of the matching role that fails.
- The error MUST identify the unresolved reference and the @-edge that carries it.
- Validation of the rest of the document continues; unresolved references do not halt validation.

A resolution failure means the named target does not exist in any resolvable scope. The unresolved reference does not become a placeholder, virtual entity, or tombstone — the data is exactly what was authored, and the validator's report indicates that the reference does not resolve. Applications that want placeholder behavior must construct it at the application layer.

Implementations MAY support modes where unresolved references are treated as warnings rather than errors, useful for ingesting partial data sets or for documents that intentionally reference targets in other systems. When such modes are supported, the strict mode (errors on unresolved references) SHOULD remain the default.

#### 6.4.2 Undefined Relationships — the Empty Rel

An @-edge's `#rel` field MAY be empty. An edge with no `#rel` parses to a null Rel and round-trips as empty — no special syntax, no distinct construct. The interpretation follows from the **existence/specification split**:

- **Role** (`@OBJ`, `@REL`) — asserts that a relationship *exists*. An @-edge is, by its role, a relationship (E1, E2′); its presence in the document is the existence assertion.
- **Rel field** — specifies *what* the relationship is. It MAY be empty, in which case the relationship is **undefined as to nature**.

An empty rel is therefore never ambiguous about whether a relationship exists — the role settled that. Emptiness speaks only to the relationship's nature, which is not yet specified.

**Motivation.** Sometimes an edge must be created before the form of the relationship is known — real-time capture, pending classification, a connection observed before its nature is determined. Forcing a fully-specified rel at creation time would mean either inventing a placeholder predicate or failing to capture the connection; both are worse than recording "connected, nature to be determined."

**Position in the model.** The empty rel is the natural limit of the identity-present / meaning-deferred principle that runs through GDX — not merely "meaning needs the schema" but "meaning not yet assigned at all." The edge is a declared-but-unbound relationship: well-formed and *undefined-pending*, never malformed. This is consistent with the well-formed-but-undefined posture GDX takes elsewhere (predicates need no REL declaration to be used; constraint identifiers are recorded without interpretation).

**Refinement.** Later refinement fills the `#rel` field on the same edge. Edges are first-class identifiable entities (E6): the edge's identity persists across refinement — nothing is created or replaced; an open field is filled. This is the open-slot refinement pattern of the universal claim rule (§8.2.3): narrowing what was left open, never mutating a prior claim. Because refinement addresses the edge after the fact, an edge with an empty rel SHOULD carry an `Id` or `Key` so it remains addressable; an unidentified one can be refined only by structural position, which is fragile under restructuring.

**Required disciplines.** Three disciplines keep undefined relationships from becoming invisible debt:

- **Queryable as undefined.** "Show all undefined relationships" MUST be answerable by any consumer that can enumerate edges — an edge with a null Rel is a distinct, selectable state. Undefined relationships are a visible backlog, not hidden debt.
- **Schema can require specification.** A Template or vocabulary MAY declare empty rels unacceptable for a given slot or document class (a vocabulary-defined constraint, §8.4), so the permissiveness is opt-out-able where strictness matters — a real-time capture context allows them; a published ontology forbids them. See §8.2.3 for how empty rels reconcile with Template-declared predicates.
- **Staleness is a tooling concern.** Long-lived undefined relationships are a data-quality signal for tooling to surface; the core takes no position on edge age.

The empty rel is a legitimate *transitional* state, not a permanent resting place: easy to create when needed, easy to find, and easy to require resolution of where appropriate.

```
<OBJ sensor-hub
    <@OBJ &link-0192 : @device-7>
>
```

The edge asserts "sensor-hub stands in some relationship to device-7, nature pending," keyed for later address. Once classified, the same edge is refined in place:

```
<OBJ sensor-hub
    <@OBJ &link-0192 #gdx:controls : @device-7>
>
```

Same edge, same key, same target — the field is filled.

### 6.5 Block — `BLK`

| Canonical | Minified | Resource? | Identifier |
|-----------|----------|-----------|------------|
| `BLK`     | —        | —         | required (local) |

A Block is a **labeled section** — an organizational grouping that names a region of children sharing a common purpose or context. Conceptually, where a Record asks "what is this composite value?", a Block asks "what context do these children belong to?". A Block is not a data type — it does not represent a structured composite that the parent *has*; it labels a region of the parent's children, asserting that they share a common purpose. Structurally, a Block is a named container that holds children but does not contribute a composite value to its parent. BLK has no minified form.

Blocks accept the same children as Objects and may only appear under an Object, another Block, or a REL declaration (§6.15; a Block on a REL labels a region of the predicate's metadata). The children of a Block are addressed through the Block identifier (e.g., `/article-42/head/title`).

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

### 6.6 Annotation — `REM`

| Canonical | Minified | Resource value-kind |
|-----------|----------|---------------------|
| `REM`     | —        | `string`            |

An Annotation is a **note attached to its parent** — out-of-band commentary intended for human or tooling consumption rather than for the data model's content. Conceptually, an Annotation is text added alongside the data: provenance, audit notes, explanatory remarks, processing hints. Structurally, a `REM` element is a leaf-only string-valued element that may appear under any role and never has children. REM has no minified form.

```
<OBJ Order123
    <REM : "Imported from legacy system 2026-04-27">
    <STR customer : "ACME Corp">
    <FLOAT total : 1250.00d>
>
```

Annotations survive round-trip serialization and are part of the document, but carry no semantic weight in the data model. Tooling MAY surface them; data interpretation SHOULD NOT depend on them.

### 6.7 Package — `PKG`

| Canonical | Minified | Resource? | Identifier |
|-----------|----------|-----------|------------|
| `PKG`     | —        | —         | optional   |

A Package is a **transport wrapper** — an envelope around a payload of Objects, Templates, and REL declarations along with metadata describing the transmission itself. Conceptually, a Package is what travels between systems: it groups the things being exchanged with the information needed to interpret that exchange (source, timestamp, version, correlation identifiers). A Package is a first-class document root (§7.2) and is the idiomatic root for material *in transit*. "Not persisted as such" (below, and §10.4) describes a *receiver's* behavior — a receiver unpacks the wrapper and stores the contained data items individually, discarding the envelope — and is not a discouragement from authoring Packages; authoring a Package to carry a payload between stages or systems is exactly its purpose. A useful rule of thumb: root in Object(s) for data *at rest* (data that lives somewhere and is referenced), and root in a Package for data *in motion* (a payload being handed from one party or stage to another, with provenance about the handoff). Structurally, a Package is a transport-layer construct (§9) admitting Objects, Templates, REL declarations, Properties, Records, Sets, Annotations, and Groups as children. PKG has no minified form.

```
<PKG color-export-2026-04-29
    <STR source : "color-system-v3">
    <OBJ Pantone-485-C ...>
    <OBJ Pantone-186-C ...>
>
```

Packages MUST NOT contain other Packages. See §7.2 rule 10 and §9 for transport semantics.

### 6.8 Group — `GRP`

| Canonical | Minified | Resource? | Identifier |
|-----------|----------|-----------|------------|
| `GRP`     | —        | —         | required (local) |

A Group is a **transport partition** — a named subset within a Package, used to organize the payload into addressable chunks with their own metadata. Conceptually, a Group answers "which slice of the payload does this material belong to?" — vendor partitions, semantic categories, batch identifiers, or any other organizational axis the sender wishes to surface to the receiver. Structurally, a Group is a transport-layer construct (§9) that partitions a Package's contents into named subsets with optional metadata; Groups may only appear as children of Packages or other Groups. Like Packages, Groups are not designed to be persisted. GRP has no minified form.

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

### 6.9 Qualifier — `.`

A Qualifier is a **refinement on its parent's value** — metadata that adjusts how the parent's content is to be interpreted. Conceptually, a Qualifier says "the parent's value, but read it with this additional context": a unit on a measurement, a confidence on an observation, a mode on a configuration value, a constraint context on a typed value. Qualifiers contribute to data semantics and travel with the data; removing a Qualifier changes what the data means.

Structurally, the `.` modifier marks an element as a Qualifier. Qualifiers may contain Annotations and Extensions only (per the §7.1 matrix); an Extension on a Qualifier is an adjunct statement about the refinement itself.

Qualifier is best understood by contrast with Extension (§6.10): a **Qualifier** modifies *instance data* (it travels with the value, contributes to interpretation, and is part of the data model); an **Extension** defines *schema rules* (it travels with the Template, contributes to validation, and is NOT part of the data model). Removing a Qualifier changes the meaning of the data; removing an Extension may relax validation but does not change instance data semantics. The `.` sigil signals "this element refines what its parent *means*"; the `_` sigil signals "this element constrains what instances *must satisfy*."

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

### 6.10 Extension — `_`

An Extension is an **adjunct statement** — a vocabulary-defined assertion attached to an element that a consumer may interpret and act upon, but on which the element does not depend for its identity, value, or validity. Unlike `--type` and `Rel`, which are *constitutive* (claims the element makes about what it is and how it relates), an Extension is *adjunct*: offered alongside the element rather than part of it. The element is complete without the Extension; a consumer that understands the Extension's vocabulary may act on the statement, while one that does not ignores it harmlessly. This adjunct property is what allows multiple vocabularies to coexist on the same element — each consumer reads its own slice and ignores the rest.

Extensions range broadly. Schema-validation rules on Templates (numeric bounds, string patterns — §8.4) are one application. Predicate semantics on RELs (domain, range, inverse, cardinality — §6.15) are another. Provenance metadata, presentation hints, migration directives, ontology axioms, and any other vocabulary-defined assertion attachable to an element all share the same adjunct shape — offered alongside the element rather than constituting it. The Extension is uniformly opaque to GDX-core; the world around it (validator, renderer, migration tool, ontology reasoner) decides how much weight to give it.

Even when a Template incorporates an Extension into a conformance claim — e.g., a `_INT xsd:minInclusive` that the validator enforces against instance values — the Extension itself remains adjunct in nature. Its bindingness is conferred by the validator or Template that chooses to honor its vocabulary, not by the Extension itself.

Structurally, the `_` modifier marks an element as an Extension. Extensions MAY appear as children of any element that admits children (the composition matrix §7.1 lists Extension as an admitted child role on all data-bearing and declaration-bearing parents). The Extension carries a vocabulary-defined identifier in its `Id`-position field (the assertion's name) and a typed Resource (the assertion's value, of value-kind matching the Extension's prefix family).

**Extensions carry primitive value-kinds only.** The closed Extension prefix vocabulary admits string, int, float, bool, and bytes (the table below). To express an adjunct statement whose value is a *reference* to another element, the construct depends on whether the assertion is a graph statement or a slot binding:

- For **graph statements between graph-entities** (Objects, RELs) — e.g., "this predicate has `gdx:Person` as domain," "this REL is the inverse of `gdx:knownBy`" — use an @-edge (`@OBJ` or `@REL`, §6.4) with an associative `#rel`: `<@OBJ #gdx:domain : @gdx:Person>`, `<@REL #gdx:inverse : @gdx:knownBy>`.
- For **handle slots pointing at non-graph-entities** (Templates, Records, value-Properties, Qualifiers, etc.) — e.g., "this slot holds a handle to the schema that types the range" — use a REF (`%KIND`, §6.16) with optional compositional `#rel`: `<%DEFOBJ #gdx:rangeSchema : @PersonTemplate>`, `<%REC defaultMailing : @global-defaults$/mailing>`.

Extensions, @-edges, and REFs together cover the adjunct-statement surface: Extensions for primitive-valued assertions, @-edges for graph-statement assertions, REFs for handle-valued assertions. All three attach to elements as vocabulary-keyed children; the choice follows the value-kind and genus (slot-filling vs relationship-asserting) of the assertion.

See §6.9 for the contrast with Qualifier — both attach to elements, but Qualifier is *constitutive* (refining what the parent's value means; removing changes meaning), while Extension is *adjunct* (an assertion alongside; removing does not change what the parent is or means). See §6.6 for the contrast with Annotation (REM) — Annotation is free-text without a vocabulary identifier; Extension is vocabulary-keyed and typed.

| Canonical | Minified | Resource value-kind |
|-----------|----------|---------------------|
| `_STR`    | `_S`     | `string`            |
| `_INT`    | `_I`     | `int`               |
| `_FLOAT`  | `_F`     | `float`             |
| `_BOOL`   | `_L`     | `bool`              |
| `_BYTE`   | `_B`     | `bytes`             |

A schema constraint on a Template (Extensions' most common use):

```
<DEFSTR Email
    <_BOOL required : true>
    <_STR pattern : "^[^@]+@[^@]+$">
>
```

A predicate-semantic Extension on a REL:

```
<REL gdx:knows
    <@OBJ #gdx:domain : @gdx:Person>
    <@OBJ #gdx:range  : @gdx:Person>
    <_BOOL gdx:symmetric : true>
>
```

Provenance attached to an Object instance:

```
<OBJ orders/2026-04-29
    <_STR acme:source-system : "legacy-orders-v3">
    <_STR acme:imported-at : "2026-04-29T14:22:10Z">
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
>
```

In each case, GDX records the Extension as a well-formed element; what (if anything) happens with it is determined by the consumer that understands the vocabulary. The same value-kind may appear as a Qualifier or as an Extension depending on what is being said:

```
<FLOAT temperature : 21.5
    <.STR unit : "celsius">          ← Qualifier (constitutive): part of the value's meaning
    <_STR acme:precision : "high">   ← Extension (adjunct): an assertion about the measurement
>

<DEFFLOAT TemperatureReading
    <_STR unit : "celsius">          ← Extension on Template (incorporated into validation by the validator)
>
```

The first is data ("the temperature value, expressed in celsius"); the second is schema ("any instance of TemperatureReading is required to be in celsius"). The grammar makes the distinction explicit at the prefix level.

### 6.11 VAL — `VAL` (element-as-value)

A VAL is **an element in value position** — a slot that holds, as its value, a fully first-class element complete with its own role, identity, resource, and children. Where a value-bearing prefix says "my value is a {string, int, …}," VAL says "my value is an element." The alternative — serializing a structure into a string literal — forfeits parsing, validation, addressability, and round-trip integrity; VAL is how GDX keeps a value structured when that value is itself an element.

The mechanism distinguishes two levels of identity:

```
<REC person
    <VAL #gdx:legalName nameSlot
        <STR #gdx:givenName name : "bob">
    >
>
```

Here the VAL `nameSlot` carries `#gdx:legalName` — the slot's alignment in `person`'s structure. The occupant STR carries `#gdx:givenName` — the value's own semantic alignment. These are two distinct alignments at two different levels; a single-element form would collapse one of them. VAL preserves both.

| Canonical | Minified | Resource? | Identifier |
|-----------|----------|-----------|------------|
| `VAL`     | `V`      | —         | required (local) when in named-attribute position |

#### Axioms

1. **No Resource of its own.** VAL carries no Resource and no `:` separator; the value is the occupant child, not a literal Resource value.

2. **Exactly one occupant.** VAL holds exactly one *occupant* — the embedded value, which is a structural element of any data-bearing role (see Axiom 3). Annotations, Qualifiers, and Extensions MAY additionally appear as children of the VAL itself, refining the slot rather than the value. Zero occupants or multiple occupants are malformed.

3. **Occupant role.** The occupant may be a Property (any value-kind), Record, Object, Block, Set, Edge (as a graph-statement value), or Reference (as a typed-pointer value). The occupant MUST NOT be a VAL (no double-wrapping), a Template or REL (vocabulary-layer roles are not embeddable as values), a Qualifier, Extension, or Annotation (refinement and adjunct roles are not values), a Node-zone element (Node isolation per §7.2 rule 5), or a transport-layer role (Package/Group).

4. **Two-level identity.** The occupant's identity fields (`Id`, `Key`, `Rel`) are part of the embedded value and describe the value itself. The VAL's own identity fields name the slot the value fills and describe the slot's role in the parent. The occupant is in *positional* context within the VAL (the slot name comes from the VAL, not the occupant); the §3.2 identifier requirements that apply to Properties and Records in named-attribute position are accordingly relaxed for VAL occupants. The occupant MAY be anonymous (slot-named only) or identified (when its own identity carries information — for example, a `Rel` alignment for the value's semantic content).

5. **Object licensing.** VAL is the data-layer licensing point for an Object in non-root position. The §7.2 rule-2 Object-exclusion exception that historically permitted `%O` is provided by VAL: a VAL whose occupant is an Object is the only data-layer form (besides transport-layer Package and Group payload) that may contain an Object as a non-root element.

6. **No `VAL*` form.** Sets of VAL are malformed. Set members are positional and need no slot-wrapping; a Set of `X`-typed values is written `X*` directly.

#### Examples

A simple primitive value with an anonymous occupant:

```
<OBJ alice
    <VAL nickname
        <STR : "Allie">
    >
>
```

A structured value (Record) as the occupant:

```
<OBJ order123
    <VAL deliveryAddress
        <REC <STR street : "123 Main"> <STR city : "Oakland">>
    >
>
```

An Object embedded as the value of a slot:

```
<OBJ orderRecord
    <VAL primaryContact
        <OBJ contacts/alice
            <STR firstName : "Alice">
            <STR email : "alice@example.com">
        >
    >
>
```

Two-level alignment — slot purpose and occupant content both surfaced:

```
<OBJ employee
    <VAL #gdx:emergencyContact contactSlot
        <OBJ contacts/bob --gdx:Person
            <STR #gdx:givenName name : "Bob">
            <STR #gdx:phoneNumber phone : "555-0100">
        >
    >
>
```

The slot says "this is the employee's emergency-contact slot." The occupant says "the value is a Person Object with these attributes."

### 6.12 Node — `N`

A Node-zone element is a **semi-structured pocket** within an otherwise structured document — a region where ad-hoc or schema-less material is permitted to live without disturbing the structured discipline around it. Conceptually, Node-zone material says "this subtree's shape isn't fixed; treat it as provisional or external": imported audit data of unknown shape, blobs of metadata from external systems, ad-hoc payloads where a Template would be premature.

Structurally, the `N` letter-modifier marks an element as establishing or participating in a Node-zone subtree. The Node modifier composes with primitive bases or with `REC`. The `N` is a letter rather than a sigil because Node-zone elements have different composition rules than the rest of GDX (isolated subtrees — see §7.2 rule 5).

| Canonical | Minified | Carries Resource? | Resource value-kind |
|-----------|----------|-------------------|---------------------|
| `NREC`    | `NR`     | No                | —                   |
| `NSTR`    | `NS`     | Yes               | `string`            |
| `NINT`    | `NI`     | Yes               | `int`               |
| `NFLOAT`  | `NF`     | Yes               | `float`             |
| `NBOOL`   | `NL`     | Yes               | `bool`              |
| `NBYTE`   | `NB`     | Yes               | `bytes`             |

Node-zone elements are permissive in what may *contain* them — any structural role may have a Node-zone child — but restrictive in what they may *contain*: a Node-zone element's children MUST themselves be Node-zone elements (or Annotations). This isolation rule keeps validation and tooling tractable: above the Node, structured rules apply; within the Node, structure is informal.

The Node modifier always composes with a base — there is no bare Node prefix. `NREC` is the no-Resource form (Node-zone container analogous to a Record); it replaces the bare `<N>` form used in earlier versions. Earlier-version documents using bare `<N>` MUST be migrated to `<NREC>`.

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

### 6.13 Set — `*`

A Set is a **multiplicity** — "many of these" as a named element. Conceptually, a Set says "the parent has not one of these but a collection of them": tags on a record, contacts on a person, allowed values in a configuration. The Set itself is the element that participates in its parent's structure; its members are positional rather than named (their order is the document order in which they appear).

Structurally, the `*` suffix promotes a value-bearing primitive or Record into a multi-valued collection. A Set is a role in its own right, not a transparent overlay on its base role: Set composition rules differ from Property composition rules — a Set may contain its members; a Property cannot.

The `*` is a suffix because Set construction is structurally distinct from the refinement modifiers. A Set's value-kind is "collection of base-instances" rather than the base's single-value-kind. Reading `STR*` as "STR, plural" makes the collection semantics legible at the prefix level. See §5.3.1 for the suffix's role in the overall prefix vocabulary.

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

#### 6.13.1 Empty and Absent States for Sets

Sets have two distinguishable states (in contrast to Properties, which have three per §3.3.3). The simpler taxonomy reflects the fact that a Set's content is determined by its child members rather than by a Resource value:

- **Absent.** The Set element does not appear in the parent at all. The collection is undeclared; consumers cannot determine whether members exist or not.
- **Present.** The Set element appears in the parent, with zero or more member children. The collection is declared to exist; the members are exactly what is present.

A Present Set with zero members — `<STR* tags>` — is an **empty Set**. It is semantically distinct from an Absent Set: the empty form asserts "this collection exists and contains no members"; absence asserts nothing about the collection.

This distinction is significant for database mapping, query semantics, and API design. A `Customer` Object with `<STR* tags>` (no children) and a `Customer` Object with no `tags` element at all are not equivalent. The first declares an empty tag list (which can be serialized, stored, queried as "tag count = 0"); the second declares nothing about tags (which may indicate the field doesn't apply, hasn't been computed, or is unknown).

There is no third "null Set" state analogous to a null Property. Sets do not carry a Resource value of their own — their content is fully expressed through their children — so the present-but-null state has no meaningful encoding for Sets.

Templates that declare a Set property MAY use the `gdx:required` and `gdx:optional` extensions (Common Vocabulary §4.1) to constrain whether the Set element must be present:

- **Required Set.** The Set element MUST appear; it MAY be empty.
- **Optional Set.** The Set element MAY be absent; if present, it MAY be empty.

When emptiness itself is meaningful for the application, the Template MAY use `gdx:minItems` to require at least one member:

- **Required non-empty Set.** The Set element MUST appear and MUST contain at least one member.

### 6.14 Template — `DEF`

A Template is a **type contract** — a declaration of the shape and constraints an instance must satisfy to claim conformance to a named type (§8). Conceptually, a Template is written in instance vocabulary: it looks like the elements it describes, with the same prefixes and the same composition rules, but with `DEF` declaring "this is the contract, not the data." A Template is what an instance points to through its `--type` field; the instance asserts identity, the Template defines what that identity means structurally.

Structurally, the `DEF` prefix-fragment marks an element as a Template definition. A Template MUST carry an identifier (the type name) and follows the composition rules of its base role. Like most roles, Templates admit Extension children (§6.10); on a Template, Extensions carry the constraints a validator enforces against instances (§8.4). Template prefixes are canonical-only; they have no minified form.

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

### 6.15 Relation Declaration — `REL`

| Canonical | Minified | Resource? | Identifier |
|-----------|----------|-----------|------------|
| `REL`     | —        | —         | required (predicate identifier) |

A Relation Declaration is a **predicate as a first-class entity** — the published counterpart to the `#rel` predicates that @-edges and other elements carry. Conceptually, a REL says "this predicate has its own identity, and the metadata describing it lives here": its domain and range, its inverse, whether it is symmetric or transitive, what vocabulary it participates in, what concepts it connects, what humans should read about it. Without RELs, predicates would be opaque identifiers used inline; with RELs, the predicate vocabulary itself becomes part of the data model.

Structurally, a `REL` element declares a predicate as a first-class entity. The `REL` element's identifier is the predicate identifier itself (typically a CURIE such as `gdx:knows` or `schema:givenName`). REL has no minified form.

**Defined by purpose, not by structure.** A REL's function — naming a first-class predicate — is fully discharged by its identifier. It needs no particular children to be well-formed; the predicate-as-named-entity exists once the REL is declared. Children, when present, supply metadata *about* the predicate: data carriers (labels, descriptions, version info), graph-statement axioms about how the predicate relates to concepts and other predicates, primitive-valued semantic claims (symmetry, cardinality), and handles to vocabulary-layer artifacts. None is required; each contributes only what its consumer interprets. A REL is held to no stricter standard than any other element — Properties stand on a REL exactly as they stand on any Object, with meaning conferred by interpreting consumers and meaninglessness never disqualifying.

**Admitted children.** A REL admits:

- **Properties** (`STR`, `INT`, `FLOAT`, `BOOL`, `BYTE` and their variants) — data about the predicate: human-facing labels and descriptions, version markers, authorship, deprecation flags, alias strings. Properties on a REL stand as Properties stand everywhere; GDX-core records them as well-formed typed data, and consumers (documentation tools, vocabulary browsers, lookup indices) decide what to do with them.
- **Records, Sets, Blocks, VAL** — structured data, collections of data, and organizational containers, by the same principle. A REL with a `<REC documentation>` carrying a structured description block, or a `<STR* aliases>` carrying alias strings, is well-formed; the vocabulary that interprets these structures decides their meaning.
- **@-edges** (§6.4) — first-class predicate-axiom statements. An `@OBJ` edge to a concept-Object expresses domain or range (`<@OBJ #gdx:domain : @gdx:Person>`); an `@REL` edge to another REL expresses inverse, subPropertyOf, or equivalent-property relationships (`<@REL #gdx:inverse : @gdx:reportsTo>`). These are first-class assertions about the predicate — they may carry their own identity (per §6.4 axiom E6) and attribute children describing the axiom.
- **Extensions** (§6.10) — primitive-valued vocabulary-defined predicate axioms: symmetry (`<_BOOL gdx:symmetric : true>`), cardinality bounds, transitivity flags, and any other primitive-valued semantic claim a vocabulary defines.
- **REFs** (§6.16) — typed handles to non-graph-entities: a Template that types the range (`<%DEFOBJ #gdx:rangeSchema : @PersonTemplate>`), a documentation document, a related vocabulary file.
- **Annotations** (§6.6) — free-text notes for human readers.

**Excluded children.** A REL does NOT admit:

- **Object** — vocabulary entities don't contain data-instance Objects; an Object inside a REL would make a contextually false claim about what the REL is.
- **Node** — RELs are structured vocabulary entities, not semi-structured pockets.
- **Qualifier** — RELs carry no Resource, so there is nothing to refine.
- **REL** and **Template (`DEF…`) forms** — vocabulary entities do not nest within RELs. This is an explicit carve-out from the modifier-transparency rule (§7.2 rule 9): even though REL admits Record and Object base roles would otherwise be admitted via transparency, the `DEF…` Template forms and the `REL` form itself are blocked because they declare vocabulary entities, and nesting one vocabulary entity inside another is structurally false.
- **Transport-layer roles (Package, Group)** — transport wrappers do not appear inside vocabulary declarations.

**Property vs Extension on a REL.** Both Properties and Extensions are admitted; authors choose based on what the metadata *is*:

- Use a **Property** for data about the predicate that a downstream consumer reads as-is — `rdfs:label`, `rdfs:comment`, version markers, authorship, alias strings, deprecation flags. The Property's meaning, if any, is conferred by the consuming vocabulary, never by GDX-core.
- Use an **Extension** for a vocabulary-defined semantic axiom about the predicate that a validator or reasoner acts on — `gdx:symmetric`, `gdx:transitive`, cardinality bounds, value-set constraints. The Extension's bindingness is conferred by the validator that recognizes the vocabulary identifier.

The mechanical version: would a validator or reasoner act on it? → Extension. Is it for humans or downstream lookup? → Property. Both are well-formed in either case; the choice is about communicating intent to consumers.

**@-edge target convention.** The @-Edge family axioms (§6.4 E5) restrict @-edge targets to Objects or RELs structurally. The vocabulary convention for @-edges on a REL is to target *concept-Objects* (Objects with conceptual identifiers like `@gdx:Person`) or other RELs — to make predicate-axiom statements like domain/range/inverse. This is convention, not enforcement; the matrix admits any @OBJ or @REL target, and the concept-vs-data-instance distinction lives in vocabulary practice, not in GDX-core.

```
<REL gdx:knows
    <STR rdfs:label : "knows">
    <STR rdfs:comment : "Asserts an acquaintance relationship between two Persons.">
    <STR* aliases <STR : "is_acquainted_with"> <STR : "is_connected_to">>
    <@OBJ #gdx:domain : @gdx:Person>
    <@OBJ #gdx:range  : @gdx:Person>
    <_BOOL gdx:symmetric : true>
>

<REL gdx:manages
    <@REL #gdx:inverse : @gdx:reportsTo>
>
```

The interplay among the three vocabulary-related constructs is:

- The `#rel` field on any element names a predicate by identifier (§6.15). The reference is lightweight — it asserts "this element represents this predicate" without requiring the predicate to be declared anywhere.
- A `REL` element declares the predicate with its metadata, making the predicate's semantics part of the data model.
- An `@REL` @-edge from one REL declaration to another expresses a first-class predicate-to-predicate axiom (inverse, subPropertyOf, etc.).

The three are coordinated by name: when a `#rel` identifier matches a `REL` identifier in scope, the REL's metadata applies to the predicate. When an `@REL` @-edge targets a REL identifier, the edge resolves to that declaration as a graph-statement-level link between two predicates.

**Concept-as-Object.** GDX deliberately has no separate Class or Concept role. When a vocabulary needs to name a concept (a type of entity, an enumeration value, a category), an Object with a conceptual identifier serves that role. RELs reference concepts as Objects (`@gdx:Person`, `@gdx:Organization`), and concept Objects may carry their own properties, labels, and relationships like any other Object. This keeps the data model uniform — vocabulary, schema, and data all use the same element form.

**A small vocabulary document:**

```
<OBJ gdx:Person
    <STR rdfs:label : "Person">
>

<OBJ gdx:Organization
    <STR rdfs:label : "Organization">
>

<REL gdx:knows
    <STR rdfs:label : "knows">
    <STR rdfs:comment : "Asserts an acquaintance relationship.">
    <@OBJ #gdx:domain : @gdx:Person>
    <@OBJ #gdx:range  : @gdx:Person>
    <_BOOL gdx:symmetric : true>
>

<REL gdx:employs
    <STR rdfs:label : "employs">
    <@OBJ #gdx:domain : @gdx:Organization>
    <@OBJ #gdx:range  : @gdx:Person>
    <@REL #gdx:inverse : @gdx:employedBy>
>

<REL gdx:employedBy
    <STR rdfs:label : "employed by">
    <@OBJ #gdx:domain : @gdx:Person>
    <@OBJ #gdx:range  : @gdx:Organization>
>

<DEFOBJ gdx:Person
    <STR firstName>
    <STR familyName>
    <REC* knows>
>
```

This document defines a small vocabulary: two concept Objects (`gdx:Person`, `gdx:Organization`), three predicates with declared domain/range, inverse relationships, and human-facing labels, and one Template binding instance shape to the Person concept. The entire vocabulary is GDX; consumers parse it with the same parser they use for instances. The labels on each REL are Properties (data carried for downstream consumers like documentation tools); the domain/range/inverse are @-edges (graph statements); the symmetry flag is an Extension (a vocabulary-defined axiom a reasoner may act on).

A REL declaration MAY appear at the document root (alongside or instead of primary Objects) or within a vocabulary document. Resolution of `#rel` references follows the same mechanism as Type reference resolution (§8.5).

### 6.16 The %-Reference Family — `%KIND`

A %-Reference is GDX's **value-resolution construct**: a slot whose effective content is the resolved target — "my content comes from there." Structurally, it is a **slot-filler whose Resource is a typed handle to an independently-existing element**: the slot carries the address; the target lives elsewhere; dereferencing yields the target. A Reference asserts no relationship about its parent (that is Edge's job, §6.4), does not contain its target (that is VAL's job, §6.11), and is inert toward the target while elaborated toward its own slot, exactly as any Property is.

REF is *defined* by value-resolution, not by what it may target. Resolving to a non-identifiable interior field (a Record's street, a Qualifier's unit) is its most common use, but resolving to a whole identifiable element used as a value is equally a REF. The capability boundary between the two link constructs falls exactly on their conceptual boundary: REF-into-a-field is coherent — value-resolution naturally reaches interior resources — while EDGE-into-a-field is incoherent — a labeled relationship connects whole elements; one does not draw a graph edge to half of an object. That what each construct *can address* coincides with what each construct *means* is the structural evidence that REF and @-Edge are genuinely different constructs, not variants of one mechanism.

Choosing between REF and @-Edge in borderline cases is authoring guidance, not spec ambiguity: value-sourcing ("this slot's value is that element, or that field") takes a REF; a labeled relationship ("these two stand in this relation") takes an @-edge. The non-normative guidance lives in the GDX Primer and the Cookbook.

#### Family Generation: `%` Wraps Any Role-Prefix

The closed REF family is generated by a single composition rule: the `%` sigil wraps any *layer-1 role-prefix* to produce a REF-to-that-role. Layer-1 role-prefixes follow the established pattern `[@?][refinement-modifier?][base][*?]`. The two-layer composition is:

```
%KIND   where KIND is any layer-1 role-prefix
```

Examples spanning the closed family:

- `%OBJ`, `%REC`, `%BLK` — REF to an Object, a Record, a Block
- `%STR`, `%INT`, `%FLOAT`, `%BOOL`, `%BYTE` — REF to a value Property
- `%.STR`, `%_INT`, `%NSTR` — REF to a Qualifier, an Extension, a Node-zone element
- `%DEFOBJ`, `%DEFSTR`, `%DEFREC`, etc. — REF to a Template
- `%VAL` — REF to a VAL slot
- `%@OBJ`, `%@REL` — REF to an @-edge (reification; see below)
- `%PKG`, `%GRP` — REF to a transport wrapper (Package, Group)
- `%STR*`, `%REC*`, `%.STR*`, and the other Set forms — REF to a Set element. Valid only where the wrapped Set prefix itself exists (value primitives, REC, and modifier Sets per §5.3.1/rule 13); `%DEF…*`, `%REL*`, `%OBJ*`, and `%@OBJ*` are malformed — their target Sets cannot exist (rules 13, 14a; Templates and RELs are root-level declarations, a Set of them does not make sense)

The transport-wrapper forms (`%PKG`, `%GRP`) follow from the generation rule like every other member; their practical scope follows from their targets' lifecycle. Packages and Groups are transient (§9, §7.2 rule 11), so a REF targeting one is meaningful within the transport context that carries it — a manifest entry pointing at a sibling Group, a correlation handle pointing at the enclosing Package — and becomes an ordinary unresolved reference once the wrapper is unpacked and discarded. That is acceptable and consumer-handled like any other unresolved REF (§6.4.1 failure semantics apply by extension); authors SHOULD NOT persist `%PKG`/`%GRP` REFs beyond the transport exchange they describe.

Three compositions are malformed:

- **`%%KIND`** — no REF-to-REF. A REF is not a reference target (axiom R6); to point at what a REF points at, write the address of the target directly.
- **`@%KIND`** — no @-edge-to-REF. @-edges target only independent graph-entities (§6.4 axiom E5); REFs are slot-contents.
- **`@@KIND`** — no @-edge-to-@-edge directly. To reify a specific @-edge, use a `%@OBJ` or `%@REL` REF with the edge's key.

#### Axioms

**R1 — Prefix asserts target role; Resource asserts target identity.** The `%KIND` prefix declares the role the resolved target MUST occupy. `%OBJ` declares an Object target; `%STR` declares a string-Property target; `%@OBJ` declares an Object-edge target; and so on. The Resource is a reference literal (`@id`, `$path`, `&key`, `@anchor$path`, or `@anchor&key` per §4.2.6) naming the target's address. Resolution dereferences the address and verifies the resolved element's role matches the prefix; mismatch yields an unresolved reference (the REF element remains structurally well-formed; the resolution fails).

The prefix is a *role-to-dereference* (`%OBJ` → resolves to an Object element): it declares the role of the element the address resolves to, and resolution verifies that role. What the slot *stores* is the address — that is what serializes, signs, and round-trips (Appendix A.2); the target's content is never copied into the document. What the slot *means* is the resolved target: dereferencing yields the target element and, for value-Property targets, its value. This is the value-resolution reading (see the lead of this section) of the same structural fact. The two views are layered, not competing — address at the representation layer, resolved value at the consumption layer.

**R1a — The role-assertion is a resolution constraint, particularly pronounced for path/key resolution.** Because path/key targets may be locatable only at resolution time, the role-check piggybacks on the dereference step rather than being verifiable at parse time. The same constraint applies in principle to identifier-mode resolution but can often be verified earlier (when the target is locally declared). The constraint is structural — it stands whether enforcement happens at parse time or resolution time.

**R2 — A REF fills a slot; it asserts nothing about its parent.** The REF's only assertion is "this slot holds a handle to that element." No relationship (vs. @-Edge), no containment (vs. VAL). Removing a REF removes a slot's content — not a relationship, not a contained thing, and never the target (which exists independently and is unaffected by the REF).

**R3 — A REF is Property-shaped.** It carries Property elaboration and nothing more. It MAY carry:
- a `#rel` field (compositional alignment — see R4),
- Qualifiers (refining the slot's binding, exactly as on any Property),
- Extensions (adjunct vocabulary-keyed assertions about the slot or its binding — e.g., provenance about how the handle was selected, audit info about the binding),
- an `Id` (the slot name within its parent's scope),
- Annotations.

It MUST NOT carry structural children (Properties, @-edges, Sets, Records, Objects, Blocks, VAL, REL, Template). Structural children would make the REF a substance rather than a handle, collapsing the genus distinction with VAL and Edge.

**R4 — A REF MAY carry a `#rel`; on a REF the `#rel` is always *compositional alignment*.** The `#rel` aligns the slot to a vocabulary predicate (§6.15); it does NOT assert a relationship the parent has. An associative predicate (a relationship a parent has) is the exclusive province of @-Edge. The same `#rel` token aligns-a-slot on `%` and asserts-a-relationship on `@`; this role-parameterization of `#rel` (§6.4, §6.15) is what keeps the two sigils from collapsing into synonyms.

The structural test (proof by collapse): if `<%OBJ #X x : @T>` and `<@OBJ #X : @T>` could both assert "parent X T," the sigils would be interchangeable spellings and the partition would dissolve. They do not assert the same thing, by R4 — the `%` form is "slot named `x` holds a handle to `T`, slot semantically aligned with `#X`"; the `@` form is "parent stands in relation `#X` to `T`."

**R5 — A REF may target any addressable element, by any resolution mode.** Entities by name (`@id`), interiors by path (`$path`), keyed elements (including @-edges via E6 identity) by key (`&key`), and composed cross-document forms (`@anchor$path`, `@anchor&key`). The full addressable space — including the interior, cross-document, and reification reach that the @-Edge family (entities-by-name-only, §6.4 resolution rule) cannot provide — is Reference territory.

Anchors in composed references remain restricted to globally-addressable entities (Objects, RELs, Templates, transport wrappers — see §4.2.6). The "any addressable element" of R5 refers to the *target*, not the anchor; reaching interior targets is accomplished by anchor-plus-path/key.

**R6 — A REF has no identity of its own beyond the slot it fills.** A handle is not a substance; the only name a REF bears is its slot name (a Property's `Id`), which is the parent's concern. A REF is **never itself a reference target** — to point at what a REF points at, point at the target directly; do not chain through the intermediate REF.

To attach adjunct metadata about a specific binding (provenance, audit, confidence in the resolution), use Extensions *on* the REF (per R3). The asymmetry with @-Edge is intentional: @-edges have identity (E6) and ARE addressable substances; REFs are slot-contents and are NOT. Reification — making a statement *about* a relationship — is the @-Edge's identity-bearing job, accessed from outside via a `%@OBJ` or `%@REL` REF.

#### Element Shape and Composition

A REF element is admitted wherever the §7.1 matrix shows REF as a child: under Object, Record, Block, @-Edge (as an attribute child), Set (as a member, when the Set's element type is a `%KIND*` form), REL, Templates (via their base role's row), VAL (as the occupant), and the transport wrappers Package and Group (as metadata handles). It admits Qualifiers, Extensions, and Annotations as children; it admits no structural children. Its Resource is a reference literal per §4.2.6.

#### Earning Its Keep

REF earns its keep on three jobs no other role can do:

- **Configuration and wiring** — typed slot-content pointing at independently-existing entities: `<%OBJ database : @primary-db>`, `<%OBJ fallback : @secondary-db>`.
- **Reification** — pointing at an @-edge by its identity to make a statement about the relationship: `<%@OBJ source : @alice&knows-bob-2024>`.
- **Interior and cross-document addressing** — pointing at Records, Blocks, Qualifiers, Extensions, value-Properties, or any other interior element by path or key: `<%REC home : @com.acme$/locations/[1]>`, `<%.STR unit : @units$/celsius>`.

All three are one shape: a typed handle held in a slot.

#### Examples

Configuration / wiring:

```
<OBJ ServerConfig
    <STR name : "web-01">
    <%OBJ database : @primary-db>
    <%OBJ fallback : @secondary-db>
    <%DEFOBJ #conformsTo : @ServerConfigV2>
>
```

Interior and cross-document addressing:

```
<%REC mailing : @alice$/addresses/[1]>
<%BLK headSection : @article-42$/head>
<%.STR currencyUnit : @global-defaults$/currency>
```

Reification (REF to an @-edge):

```
<OBJ alice
    <@OBJ knows-bob-2024 #knows : @bob>     ← @-edge with identity
>

<OBJ audit-record
    <%@OBJ source : @alice&knows-bob-2024>  ← REF to the edge by key
    <STR confidence : "verified">
    <STR auditedBy : "SecurityTeam">
>
```

The audit-record references the specific @-edge by its key; the edge itself remains owned by `alice` per E2′. The REF asserts only "this slot holds a handle to that @OBJ-edge" — no relationship, no containment, no claim over the edge.

Adjunct metadata about a binding (R6 workaround):

```
<%OBJ employer : @com.acme
    <_STR audit:bindBy : "import-2024">
    <_STR audit:source : "hr-system">
>
```

The Extensions describe how this *binding* was made (who selected the handle, when), not the target. The REF itself remains a slot-content; the Extensions ride alongside as adjunct statements about the binding-decision.

REF to a Set:

```
<%STR* allowedTags : @global-tag-set>
```

A `%KIND*` REF is a single REF whose target is a `KIND*` Set element — `%STR*` resolves to a `<STR* …>` Set. There is no "Set of REFs" form: a `%KIND*` element carries a Resource (the Set's address) and no members; multiple handles from one parent are expressed as sibling REFs, exactly as multiple edges are expressed as sibling @-edges. Because the target must be a Set that can exist, `%KIND*` is valid only for bases that admit Sets (the value primitives, REC, and the modifier Sets); `%DEF…*`, `%REL*`, `%OBJ*`, and `%@OBJ*` are malformed.

#### Resolution

A REF's Resource is a reference literal per §4.2.6. Resolution behavior parallels @-Edge reference resolution (§6.4.1) with the role-check generalized to the prefix-declared role:

1. **Local document.** Identifiers and paths within the same document are checked first.
2. **Loaded document context.** When the REF appears alongside other documents (batch import, registry context), those documents are checked in load order.
3. **Globally-resolvable IRIs.** References expressed as full IRIs MAY be resolved by a network-aware resolver if the implementation supports it.
4. **Role verification.** The resolver MUST verify the resolved element's role matches the REF prefix. Mismatch yields an unresolved reference.
5. **Resolution failure.** Unresolved REFs are reported as schema-non-conforming (Layer 2); the REF element itself is structurally well-formed (Layer 1).

A REF whose Resource is any non-`IdRef` literal MUST resolve to exactly one element of the matching role (see `GDX-Path.md` §7).

#### Summary

A %-reference is GDX's value-resolution construct: structurally, a slot-filler whose Resource is a typed handle — the slot stores the resolved address of an independently-existing target whose role the prefix asserts, and dereferencing yields that target as the slot's effective content. The prefix declares the target's role; the Resource gives its address by name, path, or key; resolution dereferences and verifies the role. A REF asserts no relationship about its parent (its `#rel`, if present, aligns the slot — it never associates), does not contain its target (vs. VAL), and is inert toward the target while elaborated toward its own slot, exactly as any Property is.

---

## 7. Composition Axioms

GDX nesting is governed by a single matrix indexed by **role**. The table below is the normative source: any prefix is admitted as a child if its role appears in the parent role's row.

Modifier-bearing and operator-bearing prefixes are validated as their base role with two refinements:

- **Refinement modifiers** (`.`, `_`, `N`) do not change which roles may contain or be contained by the element, with one exception: Node-zone elements (`NX`) have isolated-subtree semantics (rule 5). The **`DEF` declaration construct** is transparent for an element's *children* — a Template's contents are validated by its base role's row (`DEFOBJ` composes as Object, `DEFREC` as Record, `DEFSTR`/`DEFINT`/… as Property) — but NOT for its *placement*: `DEF…` elements are declarations, admitted as children only where the Template column shows ✓ (Package and Group payload) and at the document root (rule 9(c)). VAL is a role rather than a modifier (§6.11), governed by its own row in §7.1.

- **The `*` suffix** (Set), **the `@` sigil**, and **the `%` sigil** produce roles whose composition rules are distinct from their bases. A Set follows the Set row; `@OBJ` and `@REL` follow the @-Edge row (§6.4); all `%KIND` forms follow the REF row (§6.16).

### 7.1 Allowed Children Matrix

| Parent ↓ \ Child →  | Object | Record | Block | Property | Set | Node | @-Edge | REF | Qualifier | Extension | Annotation | VAL | Template | REL | Package | Group |
|---------------------|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| **Object**          |   | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |   |   |   |   |
| **Record**          |   | ✓ |   | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |   |   |   |   |
| **Block**           |   | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |   |   |   |   |
| **Property**        |   |   |   |   |   | ✓ |   |   | ✓ | ✓ | ✓ |   |   |   |   |   |
| **Set**             |   | ✓ |   | ✓ |   | ✓ |   |   | ✓ | ✓ | ✓ |   |   |   |   |   |
| **Node**            |   |   |   |   |   | ✓ |   |   |   |   | ✓ |   |   |   |   |   |
| **@-Edge**          |   | ✓ |   | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |   |   |   |   |
| **REF**             |   |   |   |   |   |   |   |   | ✓ | ✓ | ✓ |   |   |   |   |   |
| **Qualifier**       |   |   |   |   |   |   |   |   |   | ✓ | ✓ |   |   |   |   |   |
| **Extension**       |   |   |   |   |   |   |   |   |   | ✓ | ✓ |   |   |   |   |   |
| **Annotation**      |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
| **VAL**             | ✓ | ✓ | ✓ | ✓ | ✓ |   | ✓ | ✓ | ✓ | ✓ | ✓ |   |   |   |   |   |
| **REL**             |   | ✓ | ✓ | ✓ | ✓ |   | ✓ | ✓ |   | ✓ | ✓ | ✓ |   |   |   |   |
| **Package**         | ✓ | ✓ |   | ✓ | ✓ |   |   | ✓ |   | ✓ | ✓ |   | ✓ | ✓ |   | ✓ |
| **Group**           | ✓ | ✓ |   | ✓ | ✓ |   |   | ✓ |   | ✓ | ✓ |   | ✓ | ✓ |   | ✓ |
| **Document root**   | ✓ |   |   |   |   |   |   |   |   |   |   |   | ✓ | ✓ | ✓ |   |

There is no Template *row*: per rule 9, a `DEF…` element's children are validated by its base role's row (`DEFOBJ` admits what Object admits, `DEFREC` what Record admits, Property Templates what Property admits). The Template *column* governs where `DEF…` elements may be placed — Package and Group payload, and the document root (rule 9(c)).

**Extension admittance is broad.** Extensions are adjunct statements (§6.10) — vocabulary-defined assertions attached to an element that consumers may interpret and act on. Because Extensions don't constitute the element they attach to, they may appear under almost any parent role: data-bearing elements (Object, Record, Property, etc.) for provenance, presentation hints, and other instance-level adjuncts; vocabulary and schema elements (REL, Template) for predicate semantics and schema constraints; transport-layer wrappers (Package, Group) for transport metadata such as signing or routing directives. The Extension column in the matrix shows ✓ for all parent roles that admit children. The exceptions are Annotation (leaf-only) and Node (isolated-subtree per rule 5 — Extensions cannot leak into Node-zone material).

Template (`DEF…`) prefixes compose by their base role's row (rule 9): a `DEFOBJ` admits what an Object admits (including Blocks), a `DEFREC` what a Record admits, and the Property Templates what a Property admits. In every case this includes Extensions (carrying constraints that the validator enforces against instances — §8.4), REFs (for cross-references to Templates, configuration handles, interior pointers), and — on structure-base Templates — @-edges (declaring edge slots on instances). The Template column fixes their placement: Package and Group payload, and the document root (rule 9(c)).

The @-Edge row covers `@OBJ` and `@REL` (§6.4). Per axiom E3, attribute children describe the edge-as-substance. @-edges admit Records, Properties, Sets, Nodes, other @-edges (recursive: an edge's attribute may itself be an edge), REFs (handles in the attribute structure), VAL, and the adjunct triplet (Qualifier/Extension/Annotation).

The REF row covers every `%KIND` form (§6.16) — the closed family generated by `%` wrapping any layer-1 role-prefix. REFs are slot-fillers (Property-shaped): they admit Qualifiers (refining the slot binding), Extensions (adjunct metadata about the binding), and Annotations. They MUST NOT carry structural children (Properties, @-edges, Sets, Records, Objects, Blocks, VAL, REL, Template) — that would make a REF a substance rather than a handle.

The REL row admits the full data-carrier set (Properties, Records, Sets, Blocks, VAL) along with @-edges, REFs, Extensions, and Annotations. The widening reflects the principle that a REL is defined by its *purpose* (naming a first-class predicate) rather than by a canonical structure; data carriers on a REL stand exactly as they stand on any Object, with meaning conferred by interpreting consumers (§6.15). The structurally-justified exclusions are Object (vocabulary entities don't contain data-instance Objects), Node (RELs are structured), Qualifier (RELs carry no Resource), REL (vocabulary entities don't nest), and the Template (`DEF…`) forms (vocabulary entities don't nest — explicit carve-out from rule 9 below). The matrix admits @OBJ and @REL @-edges structurally; the vocabulary convention for predicate-axiom metadata is to target concept-Objects and other RELs (§6.15).

The VAL row admits exactly one occupant of any data-bearing role (Object, Record, Property, Block, Set, @-Edge, REF) plus optional Annotations, Qualifiers, and Extensions as slot-level metadata (§6.11). VAL does NOT admit other VALs (no double-wrapping), Node-zone elements (isolated subtree per §7.2 rule 5), Templates, RELs, transport-layer roles, or refinement/adjunct-only roles (Qualifier/Extension/Annotation alone without an occupant). VAL is admitted by data-container parents (Object, Record, Block, @-Edge) and by Templates as a slot type; it is NOT admitted by Properties (which are scalar leaves), Sets (members are positional and need no slot wrapping), REFs (leaf-like, no structural children), RELs (no data slots), or transport-layer wrappers (which contain Objects directly as payload). A VAL containing a REF as its occupant is structurally valid but degenerate — `<VAL slot <%OBJ : @t>>` is data-model-equivalent to `<%OBJ slot : @t>`; prefer the direct form.

The Package row admits Templates (any `DEF…` prefix) and REL declarations as payload along with Objects. For clarity: a Package may directly contain Object instances (`OBJ`/`O`), Template definitions (`DEF…`), REL declarations (`REL`), Properties (metadata), Records (metadata), Sets (metadata), REFs (metadata handles — e.g., a manifest entry pointing at a payload item or at another Group), Extensions, Annotations, and Groups for transport-internal organization. A Group admits the same children as a Package except for Package itself.

The Document root row covers what may appear as the root of a GDX document: an Object, a Template, a REL, or a Package. A document MAY contain multiple top-level elements when each is a recognized root-eligible type.

### 7.2 Derived Rules

The following rules are consequences of the matrix and are repeated here for clarity:

1. **Root rule.** A document root MUST be an Object, a Template, a REL declaration, or a Package. A document MAY contain multiple top-level elements; each top-level element MUST independently be a root-eligible type.

2. **Object exclusion.** No role other than VAL (when its occupant is an Object, §6.11) and the transport-layer roles (Package, Group) may contain another Object. VAL expresses Object-as-element-value within data-layer composition; Package and Group contain Objects as transport-layer payload, not as ownership relationships.

3. **Block containment.** A Block may only appear under an Object, another Block, or a REL declaration (§6.15) — or under a `DEFOBJ`, which composes as Object per rule 9. Blocks do not appear in transport-layer roles.

4. **Annotations are leaves.** Annotations never have children and are permitted under every role except other Annotations.

5. **Nodes are isolated subtrees.** A Node-zone element's children MUST themselves be Node-zone elements (or Annotations). Once a Node-zone element appears in a document, its subtree is uniformly Node-typed. Structured roles (Properties, Records, Sets, Edges, etc.) cannot appear within a Node subtree. Use `NREC` for the no-Resource Node-zone container that establishes such a subtree.

6. **Sets contain members of their declared element type.** A Set's children MUST be elements whose prefix matches the Set's element type. A `STR*` Set contains `STR` members; a `REC*` Set contains `REC` members. Qualifiers, Extensions (where permitted), and Annotations may also appear as Set children to refine or annotate the Set itself.

7. **Extensions are adjunct.** An Extension is a vocabulary-defined assertion attached to an element (§6.10). It may appear under any parent that admits children, with two exceptions: it cannot appear under an Annotation (leaf-only) or inside a Node-zone subtree (rule 5). The Extension's bindingness is determined by the consumer that interprets its vocabulary — validators acting on Constraint Extensions on Templates (§8.4), renderers acting on presentation Extensions, ontology reasoners acting on axiom Extensions on RELs, and so on. A consumer that does not recognize an Extension's vocabulary identifier ignores it; this is the property that allows multiple vocabularies to coexist on the same element.

8. **REL declares predicates.** A REL element may carry Extension children that declare predicate semantics (domain, range, inverse, symmetry, cardinality, etc.). The vocabulary of those Extensions is defined by the GDX Common Vocabulary or by application vocabularies. This is one specific application of rule 7's adjunct-Extension mechanism.

9. **Modifier transparency.** Template (`DEF…`), Qualifier (`.X`), Extension (`_X`), and Node (`NX`) modifier-bearing prefixes are validated as their base role for composition purposes. The modifier never changes what is allowed inside or what may contain the element, with two exceptions:

    (a) **Node-zone isolation.** Node-zone elements have isolated-subtree semantics (rule 5) and admit only Node-zone elements (or Annotations) as children, regardless of what their base role would admit.

    (b) **REL does not admit Templates.** Although REL admits Record, Property, Set, Block, and VAL (§6.15), the Template (`DEF…`) forms of those roles are NOT admitted under REL. This is an explicit carve-out: vocabulary entities (REL declarations, Template declarations) do not nest within REL declarations, even when modifier transparency would otherwise permit it. The same principle excludes nested `REL` children (already absent from the matrix REL row). The carve-out is principled rather than mechanical: a nested vocabulary declaration inside a REL would make a contextually false claim about what the REL is (§6.15).

    (c) **Template placement is not transparent.** The `DEF` construct's transparency covers a Template's *children* (validated by the base role's row), not its *placement*. `DEF…` elements are vocabulary declarations: they are admitted as children only of Package and Group (payload) and at the document root (the Template column in §7.1). They are NOT admitted in data-layer positions — a `DEFREC` may not sit inside an Object or Record the way a `REC` may — nor under REL (clause b), nor as a VAL occupant (§6.11). Declarations live at declaration positions; instances reference them by name (`--type`, §8.5.2).

    VAL is not a modifier of a base role — it is its own role (§6.11) and is governed by its own row in the composition matrix.

10. **Transport non-nesting.** Packages MUST NOT contain other Packages. Groups MUST NOT contain Packages. Transport-layer roles form a controlled structure: a Package may contain Groups and data-layer items; a Group may contain other Groups and data-layer items.

11. **Transport non-persistence.** Package and Group elements are not persisted as such by storage layers. Their contents (Objects, Templates, REL declarations) are extracted and stored individually; transport-layer metadata MAY be retained as provenance information but the transport wrapping is discarded. See §9 and §10.4.

12. **Refinement modifiers do not stack.** An element carries at most one refinement modifier (`.`, `_`, `N`) or the `DEF` construct. The following are NOT valid prefixes:
    - `.NSTR` — Qualifier of a Node-zone element
    - `_.STR` — Extension of a Qualifier
    - `DEFN.STR` — DEF-on-modifier-on-modifier

    The reading pattern for any prefix is `[@?][refinement-modifier?][base][*?]`. At most one of `.`, `_`, `N`, `DEF` may appear in the prefix-position-element slot; the slot is filled by at most one of these.

    Where multi-valued semantics on a template-defined value are needed, the structure is expressed through normal nesting:

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

13. **Set is a suffix.** The `*` Set marker is a suffix on the base prefix: `STR*`, `INT*`, `REC*`, etc. The `*` suffix MAY combine with refinement modifiers (`.STR*`, `_INT*`, `NSTR*`) and with the `%` REF sigil only where the wrapped Set prefix exists (`%STR*`, `%REC*`, `%.STR*`; `%DEF…*`, `%REL*`, `%OBJ*`, and `%@OBJ*` are malformed — their target Sets cannot exist). The `*` suffix does not compose with `DEF` (Templates are not collected as a Set in normal usage), with VAL (`VAL*` is malformed per §6.11 axiom 6), or with the @-Edge family (`@OBJ*` and `@REL*` are malformed per rule 14a — each @-edge is a single graph statement).

14. **@-Edge family is closed at two members.** The `@` sigil produces the @-Edge family (§6.4), restricted to two members by axiom E5: `@OBJ` (target is an Object) and `@REL` (target is a REL declaration). The sigil does not compose with other bases or modifier-bases to produce additional edge prefixes; `@REC`, `@BLK`, `@.STR`, `@DEFOBJ`, `@VAL`, and so on are NOT @-edges. To point at any of those roles, use a REF (`%REC`, `%BLK`, `%.STR`, `%DEFOBJ`, `%VAL`, §6.16).

14a. **No Sets of @-edges.** The compositions `@OBJ*` and `@REL*` are malformed. Each @-edge is a single graph statement (§6.4 axiom E2′); multiple edges from one source with the same predicate are expressed as siblings (`<@OBJ #knows : @bob> <@OBJ #knows : @carol>`), not as Set members.

15. **Sigil composition prohibitions.** The following compositions are malformed:
    - `@@KIND` — no @-edge whose target is itself an @-edge. To reify a specific @-edge, use a `%@OBJ` or `%@REL` REF with the edge's key (§6.16 R5).
    - `%%KIND` — no REF whose target is itself a REF (§6.16 axiom R6). A REF is not a reference target; to point at what a REF points at, address the target directly.
    - `@%KIND` — no @-edge whose target is a REF. @-edges target independent graph-entities (§6.4 axiom E5); REFs are slot-contents, not graph-entities.

16. **REF is Property-shaped; @-Edge is statement-shaped.** A REF element (any `%KIND`, §6.16) MAY carry a `#rel` (compositional alignment per R4), Qualifiers, Extensions, and Annotations as children, but MUST NOT carry structural children (Properties, @-edges, Sets, Records, Objects, Blocks, VAL, REL, Template). Its Resource is a reference literal; the target lives elsewhere. An @-Edge element (`@OBJ`, `@REL`, §6.4) carries an associative `#rel` specifying the relationship's nature — the field MAY be empty, leaving the relationship undefined pending refinement (E4, §6.4.2) — and MAY carry attribute children describing the edge-as-substance (per E3). The REF and @-Edge composition rows in §7.1 reflect this structural split. See §6.16 axiom R4 for the genus distinction and the proof-by-collapse that establishes the sigil partition.

### 7.3 Identifier Rules

Per §3.2, the following roles MUST carry at least one identifier (`Id`, `Key`, or `Rel`) when in named-attribute position:

- Object (`OBJ`/`O`, `DEFOBJ`)
- Record (`REC`/`R`, `DEFREC`, `NREC`/`NR`) when not in Set member position
- Property (`STR`/`S`, `INT`/`I`, `FLOAT`/`F`, `BOOL`/`L`, `BYTE`/`B` and their variants) when not in Set member position
- Template (any `DEF…`)
- REL (`REL`)
- VAL (`VAL`/`V`) when not in Set member position and when used as a named slot
- Block (`BLK`) — Blocks always require an identifier; the identifier names the partition
- Group (`GRP`) — Groups always require an identifier; the identifier names the partition within a Package

Set members are exempt from the identifier requirement but MUST carry their prefix.

Package (`PKG`) carries an optional identifier. When present, the identifier is used for transport correlation (request/response matching, deduplication, audit) and is local to the transport context.

Other roles MAY carry identifiers but do not require them.

### 7.4 Resource Rules

Per §3.3, when a Resource is present, its value-kind MUST be consistent with the element's prefix family:

- String-family prefixes (`STR`/`S`, `.STR`/`.S`, `_STR`/`_S`, `NSTR`/`NS`) carry `string` resources.
- Int-family prefixes (`INT`/`I`, `.INT`/`.I`, `_INT`/`_I`, `NINT`/`NI`) carry `int` resources.
- Float-family prefixes (`FLOAT`/`F`, `.FLOAT`/`.F`, `_FLOAT`/`_F`, `NFLOAT`/`NF`) carry `float` resources.
- Bool-family prefixes (`BOOL`/`L`, `.BOOL`/`.L`, `_BOOL`/`_L`, `NBOOL`/`NL`) carry `bool` resources.
- Bytes-family prefixes (`BYTE`/`B`, `.BYTE`/`.B`, `_BYTE`/`_B`, `NBYTE`/`NB`) carry `bytes` resources.
- Set prefixes (`STR*`, `INT*`, `REC*`, and the other `*` forms) carry NO Resource of their own — a Set's content is its member children (§6.13). The members carry resources of the base family.
- @-Edge prefixes (`@OBJ`, `@REL`) carry `reference` resources whose target-kind matches the prefix: `@OBJ` resolves to an Object; `@REL` resolves to a REL declaration (see §6.4).
- REF prefixes (every `%KIND` form, §6.16) carry `reference` resources. The Resource is a reference literal (any of the five forms per §4.2.6) whose resolved-element role is declared by the wrapped layer-1 prefix: `%OBJ` → Object; `%REC` → Record; `%DEFOBJ` → Template; `%@OBJ` → Object-edge; `%STR*` → Set of strings; and so on.
- Annotation prefix (`REM`) carries `string` resources.

The Resource itself is optional on Resource-bearing roles; an element with no Resource represents a null value (instance context) or a slot declaration (Template context). See §3.3.3.

Object, Block, Record, Node-Record (`NREC`/`NR`), Set, Template (all `DEF…`), Relation Declaration (`REL`), Package, and Group elements do not carry Resources at the element level. Their content is conveyed through their children. A Template (`DEFSTR`, `DEFREC`, etc.) declares a contract for instances; it does not carry an instance value itself.

### 7.5 Validation

A conforming parser MUST reject any document containing:

- An element whose prefix is not in the closed vocabulary (§5.2).
- A child element whose role is not admitted by the parent role's row in §7.1.
- A required identifier that is missing (§7.3).
- A Resource whose value-kind does not match the prefix family (§7.4).
- A Node containing a non-Node, non-Annotation child.
- A Set containing a member whose prefix does not match the Set's element type.
- An Object appearing as a non-root child in a position other than the occupant of a VAL slot (§6.11), a Package payload, or a Group payload.
- A Package containing another Package.
- A Group containing a Package.
- A Block at the document root or in any position other than as a child of an Object or another Block.
- A Block missing its required identifier.
- A Group missing its required identifier.

A conforming parser MUST accept any element with an identifier but no Resource section. Such elements are well-formed and represent null values (instance context) or slot declarations (Template context). A conforming parser MUST likewise accept an @-Edge element with no `#rel` field: the empty rel parses to null and denotes a relationship that exists but is undefined as to nature (§6.4.2).

The above are the *primitive axioms* of GDX validity — Layer 1 (§2.2). They are owned by GDX, evaluated at parse time, and absolute. Constraint validity beyond these axioms (numeric bounds, string patterns, regex matches, semantic types) is vocabulary-defined and evaluated externally; GDX hosts those constraints as Extension elements but does not interpret them. See §8 for the schema-layer validation model (Layer 2a — Template structural conformance) and Constraint Extensions (Layer 2b — vocabulary-defined constraint conformance).

---

## 8. Schemas and Validation (Templates)

> **Status:** §8.1 through §8.5 are stable. §8.6 describes mechanisms that are committed in principle but require companion specifications for full normative treatment.

Schemas in GDX are themselves GDX documents. This section defines the schema mechanism in three layers, mirroring the conformance model of §2.2.

- **Template structure (§8.1, §8.3).** Templates are GDX elements with `DEF…` prefixes. They declare the structural shape an instance is expected to satisfy: which child slots exist, what role each carries, what Rel each aligns to, what Type each claims. Templates are GDX-native: this specification defines their grammar, their composition rules, and how they are resolved.

- **Template-instance alignment (§8.2).** An instance carrying a `--type` reference asserts that its structure aligns to the referenced Template. Slot alignment is by `Id` with `Rel` as a cross-vocabulary bridge; the universal claim rule (§8.2.3) governs how Template-declared and instance-declared values reconcile on aligned slots; materialization (§8.2.4) is the recommended persistence form. This is Layer 2a (§2.2).

- **Constraint Extensions (§8.4).** Constraint vocabularies — XSD, JSON Schema, OWL profiles, bespoke domain vocabularies — are expressed as Extension elements on Template slots. The Extension carriers (`_STR`, `_INT`, etc.) are GDX-native. The constraint identifiers they carry (`xsd:minInclusive`, `json:pattern`, etc.) are vocabulary-defined and opaque to GDX. This is Layer 2b (§2.2). Multiple constraint vocabularies MAY coexist on the same Template; each is interpreted independently by consumers that understand its identifiers.

This layered model is what makes GDX a structural schema with pluggable constraint vocabularies (§1.2). The structural surface — what Templates declare, how slots align, how Extensions attach — is fixed by this specification. The constraint surface is open: any vocabulary may define its own constraint identifiers and ride atop the GDX carrier mechanism without negotiating with GDX or with other vocabularies.

### 8.1 Templates

A Template is an element with a `DEF…` prefix. It declares an *interface* — a set of properties, records, edges, and constraints — that an instance element claims to satisfy via its `--type` field.

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

A Template MUST carry an identifier — the Template's name. The identifier follows the same lexical rules as any other identifier (§4.4) and typically takes a CURIE form (`gdx:Address`, `com.acme.Person`) for cross-vocabulary clarity.

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

An instance child satisfies a Template slot by matching the slot's `Id` field, using case-sensitive ordinal comparison (§3.2). When a Template slot declares a `Rel`, an instance child carrying the same `Rel` MAY align to the slot even if its local `Id` differs; in that case the `Rel` is the alignment key, supporting cross-vocabulary instances whose local names differ from the Template's slot names.

The `Key` field plays no role in Template-instance alignment. Keys are an independent named-anchor mechanism (§4.4.5) used by `KeyRef` and `IdKeyRef` reference literals; they are not consulted during schema validation and are excluded from the universal claim rule (§8.2.3).

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

A Template MAY declare values for fields that the instance can also carry: the `Rel` field, the `--type` field (in nested slots), and a default `Resource` value. When both the Template and the instance carry a value for the same field on the same aligned slot, the values are reconciled by a uniform rule that applies independently to each claim-bearing field.

| Template state | Instance state | Effective value | Disposition |
|---|---|---|---|
| silent | silent | none | No claim on this field for this slot. |
| silent | claimed | instance | The instance freely supplies a value; the Template imposes no constraint. |
| claimed | silent | Template | The Template's value is the effective value, implicitly carried by the instance. |
| claimed | claimed (matching) | confirmed | Both sides agree. The claim is affirmed. |
| claimed | claimed (differing) | — | **Conflict.** The instance is Layer 2a non-conforming. |

The rule applies independently to each field. An instance MAY satisfy the rule for `Rel` (case 3 — Template-supplied) while freely supplying its own `Resource` value where the Template declares no default (case 2 — free instance claim), with both outcomes being valid.

**The `Key` field is excluded from the claim rule.** A Key is an instance-level addressing decision, never a Template claim. The exclusion is structural, not stylistic: Keys are scope-unique (§4.4.5) while a Template is a reusable shape, so a Template-declared Key would stamp a scope-unique value onto every instance of the shape — two instances of the same Template within one Object scope would materialize (§8.2.4) colliding Keys, producing a malformed document from conforming inputs. Templates MUST NOT declare a `Key` on a slot; instances freely carry Keys, which are invisible to schema validation (§8.2.1).

The differing-claims case (the conflict row) MUST be reported as a Layer 2a non-conformance. Implementations MUST NOT silently prefer one side over the other.

**Empty rels under the claim rule.** An @-edge with an empty rel (§6.4.2) is *silent* on the `Rel` field: the relationship's existence is asserted by the role, but the field carries no claim about its nature. The standard matrix applies — when the aligned Template slot declares a predicate, the Template's name is the effective value (case 3), and materialization (§8.2.4) fills the field on the same edge: refinement of what was left open, never mutation of a prior claim. A Template MAY instead declare, through a vocabulary-defined constraint (§8.4), that undefined relationships are unacceptable for a slot or document class; an instance edge with an empty rel is then Layer 2b non-conforming with respect to that vocabulary.

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

# Authored instance (Template values are implicit; locale is present but null):
<REC alice --com.acme.Customer
    <STR firstName : "Alice">
    <STR locale>
>

# Materialized instance (Template values are explicit):
<REC alice --com.acme.Customer
    <STR firstName #schema:givenName : "Alice">    # Rel materialized from Template
    <STR locale : "en-US">                          # default Resource materialized
>
```

Materialization applies to the claim-bearing fields named in §8.2.3: `Rel`, `Resource` defaults, and nested `--type` claims. It does NOT extrapolate beyond what the Template explicitly declares; missing slots are not added — including slots for which the Template declares a default Resource — and Constraint Extensions (§8.4) are not copied from Template to instance (constraints are vocabulary-defined and remain Template-scoped). The absent-slot rule is deliberate: the instance is the reality, and a slot's absence may itself be meaningful (the slot may have been deliberately removed); materialization refines what is present, it never resurrects what is not. A default Resource therefore materializes only into a *present-but-null* slot. Imparting Template defaults into a brand-new instance — filling in every defaulted slot at creation time — is an *instantiation* operation performed by authoring tooling, not part of materialization; once instantiated, the materialized representation is authoritative.

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

Constraint Extensions are one specific application of the broader Extension mechanism (§6.10): Extensions attached to Template slots that vocabulary-aware validators interpret as constraints on instances. Other applications of the Extension mechanism — provenance on Objects, presentation hints on Properties, predicate semantics on RELs (§6.15), and so on — share the same underlying shape (vocabulary-defined adjunct statements) but are interpreted by other consumers. This section covers the constraint use specifically; the general adjunct-Extension framing is in §6.10.

Extensions on Template properties carry vocabulary-defined constraints that instances may be evaluated against. The structural carriers — the closed set of Extension prefixes (`_STR`/`_S`, `_INT`/`_I`, `_FLOAT`/`_F`, `_BOOL`/`_L`, `_BYTE`/`_B`) — are defined by this specification (§6.10). The *vocabulary* of constraint identifiers (such as `xsd:minInclusive`, `json:pattern`, `gdx:maxLength`, or any application-defined equivalents) is defined by external vocabularies, not by this core specification.

**GDX records but does not interpret.** Constraint identifiers are extension-layer content (§2.1): the core records them as well-formed elements but assigns no meaning. The parser validates that each Constraint Extension is structurally well-formed (correct carrier prefix, valid value-kind, attached at a position where Extensions are admitted per §7.1). It does not interpret the identifier itself. Implementations MUST NOT interpret constraint semantics at the core layer; meaning is supplied by the vocabulary that defines the identifier, evaluated by a consumer that understands that vocabulary.

**Decision: the core records constraints but never interprets them.** Constraint semantics live entirely in the extension layer; the core assigns no meaning even to constraints that look primitive. *Alternative considered:* having the core interpret a base set of common constraints itself — `min`, `max`, `pattern`, length bounds — so that simple validation needs no vocabulary. *Rejected because:* the moment the core holds a constraint opinion, it competes with the constraint vocabularies layered on top. A core `min` and an `xsd:minInclusive` could disagree, raising the unanswerable question of which wins, and that competition would break the parallel-vocabulary coexistence that §8.4.1 depends on. By keeping core validation at only the primitive axioms (Layer 1) and treating *all* constraints as extension-layer content (Layer 2b), the core never competes with any vocabulary, and multiple vocabularies — XSD, JSON Schema, bespoke — coexist on the same Template, each interpreted by its own consumer. The discipline is what enables the pluralism; a convenience built-in constraint set would forfeit it.

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

Package (§6.7) and Group (§6.8) are the two roles reserved for transport: they wrap and partition data items during exchange between systems. This section defines how transport-layer wrappers behave operationally — how to populate them with metadata and payload, how receivers unpack them, and what survives serialization. Definitions of the prefix tokens, composition rules, and required identifiers live in §6.7 and §6.8.

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

This section addresses how GDX data is stored and how stored data round-trips through serialization. It is informative: it originates no normative requirement of its own. The data model (§3), the canonical text form (§4), and the round-trip contract (Appendix A) are the normative sources; this section shows how those layer onto persistence and interchange, and defers to them for every conformance question.

### 10.1 Wire Formats

The Standard Form defined in §4 is the normative reference. Two additional wire formats are defined in companion specifications:

- **Compact Form** — a line-oriented form using indentation rather than brackets to convey structure, designed for human authoring and for environments where bracket noise is unwanted. Each element occupies one line; child elements are indented under their parents. The Compact Form preserves all fields of the Standard Form; only the structural delimiters differ.
- **Binary Form** — a binary encoding optimized for fast wire transfer, compact size, and efficient parsing. The Binary Form preserves the data model of §3 with no information loss; round-trip through the Binary Form produces a document equivalent to the original Standard Form input.

A wire format preserves a GDX document when it carries the data model of §3 intact and admits round-trip equivalence under Appendix A; the Compact and Binary Forms are defined against that bar in their companion specifications, which are the authoritative source for their grammars and encoding rules.

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

Store-native columns beyond those above (a persistence identifier, a row version, delta-handling state) are the at-rest form of an element's metadata channel (§4.8): they annotate how the element is stored, not what it means, and they are the natural source and sink for the `{...}` block when the element is serialized out or parsed in. Because metadata is out-of-model, these columns are outside the round-trip contract (§A.2) — a backend MAY carry, add, or drop them without affecting model-conforming round-trip.

@-Edge and Reference targets stored as `ResourceValue` strings can be resolved at query time by joining against the `Id` column. For systems where the reference target is known at write time, the reference can be denormalized to a `TargetRowId` column for direct lookup. The resolution strategy is implementation choice; both approaches are conforming. Per §6.4 axiom E2′, @-edges are owned by their source element and SHOULD be modeled with `ON DELETE CASCADE` on the source foreign key so that deleting a source removes its edges.

Other persistence strategies (document store, graph database, key-value store) follow analogous principles: preserve the data model, preserve element ordering, and preserve the round-trip contract (§10.4).

### 10.3 Data Interchange

GDX maps to and from common data interchange formats (JSON, YAML, XML, RDF). Each mapping has trade-offs: GDX features without direct equivalents in the target format (predicate alignment, native bytes, schema-as-data) require convention-based encoding or are lost in conversion. Implementations performing format conversion SHOULD document their conventions.

Side-by-side comparisons between GDX and other formats, along with discussion of mapping considerations, are in the Format Comparison companion specification.

### 10.4 Persistence Round-Trip

A persistence layer is *round-trip conforming* if a document parsed, stored, and serialized back produces a document equivalent to the original. The normative round-trip contract — what MUST be preserved, what MAY be normalized, and what MUST NOT be altered — is the serialization-conformance contract of Appendix A.2; a round-trip-conforming persistence layer is one that satisfies A.2 across the parse-store-serialize cycle. This section does not restate those requirements; Appendix A.2 is authoritative.

What persistence adds over plain serialization is only that the round-trip passes *through storage* rather than directly between two serializations. A relational backend (§10.2) satisfies A.2 by giving each preserved field (prefix, the identity fields, resource value-kind and value with its representation tag, type reference, child order, and REM annotations) a faithful column or row representation, and by reconstructing the document in original child order on read. A document store satisfies it by storing and returning the serialized form unaltered. The mechanism differs by backend; the contract does not.

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

**Section:** §3.3.3

The introduction of distinct absent/null/present states implies three independent constraint dimensions for Templates: required-vs-optional (whether the Property may be absent), nullable-vs-non-nullable (whether a present Property may have no Resource), and value constraints (what values are permitted when a Resource is present). The Common Vocabulary v0.2 begins addressing this through `gdx:required`, `gdx:optional`, and value-constraint extensions; the full taxonomy and the relationship between optionality and nullability may benefit from further refinement based on implementer experience.

### 11.4 Path-Mode Reference Resolution — Resolved

**Section:** §4.2.6, §6.16, §7.1

**Status:** Closed as of v0.11.0.

Path-mode reference resolution is normatively specified by the companion GDX Path Expressions Specification (`GDX-Path.md`). All sub-items previously open under this issue are addressed:

- **The path expression grammar.** Specified in `GDX-Path.md` §3 and §3. Three segment kinds — `IdSegment`, `RelSegment`, `OrdinalSegment` — separated by `/`. Key-based addressing is provided through the `KeyRef` and `IdKeyRef` reference-literal forms rather than as a path segment.
- **The resolution algorithm.** Specified in `GDX-Path.md` §5. Left-to-right segment traversal from a defined starting element, with per-segment match semantics and explicit handling of zero, one, and multi-match outcomes.
- **The composition matrix expansion.** Specified in `GDX-Path.md` §8.5 (changes required of this specification). `%REC`, `%BLK`, `%.STR`, `%_INT`, `%VAL`, value-Property targets, and other interior elements become resolvable when the resource is a `PathRef` or `IdPathRef`. Parent-context restrictions are unchanged; only target-addressability expands. (Under v0.13.0, these targets are addressed via the REF family `%KIND`; prior to v0.13.0 they were addressed via leaf-like `@KIND` references, now migrated.)
- **Modifier-stacking interaction.** The `%`-with-modifier-bases pattern (`%.STR`, `%_INT`) resolves through path-mode references.

The composed reference form `@Identifier$PathExpression` is the canonical mechanism for cross-document addressing: the identifier portion resolves a starting element by scope walk (per §6.16), and the path portion traverses structurally from there. This form replaces the speculative path-style identifier convention (e.g., `@/people/alice`) used in earlier revisions; with the v0.11.0 narrowing of `IdStartChar` (§4.4.1), identifiers no longer begin with `/`.

---

## Appendix A — Conformance

This appendix defines what it means for a serialization, a parser, and a document to be conforming to this specification. Conformance is layered: a document may be representation-conforming without being schema-conforming, and an implementation may support one or both layers.

### A.1 Conformance Layers

This subsection is the authoritative normative statement of GDX's layered conformance model (introduced conceptually in §2.2). Conformance is layered into three rungs with three owners:

**Representation Conformance (Layer 1)** addresses how data is encoded. It covers the structural rules of §3 through §7: prefix vocabulary, identifier grammar, literal forms, composition rules, and resource value-kind matching. Owned by GDX. Decidable at parse time without external context.

**Schema Conformance (Layer 2)** addresses how data aligns to declared shapes and constraints, and decomposes into two sub-layers:

- **Layer 2a — Template structural conformance.** Covers the Template-based alignment of §8.1, §8.2, §8.3: instance children align to Template slots by `Id` (with `Rel` as cross-vocabulary bridge), and `--type` references resolve. Owned by GDX. Decidable by any consumer that can resolve Templates.

- **Layer 2b — Constraint-vocabulary conformance.** Covers the Constraint Extensions of §8.4: whether instance values satisfy the constraints expressed by a particular constraint vocabulary (XSD, JSON Schema, the GDX Common Vocabulary, or any other). Owned by the constraint vocabulary, not by GDX. Decidable only by a consumer that understands the vocabulary's constraint identifiers. A document MAY be Layer 2b-conforming with respect to one vocabulary and Layer 2b-non-conforming with respect to another; the verdicts are independent and vocabulary-relative.

A document is **fully conforming** at Layer 1 + Layer 2a if it is representation-conforming and Template-structurally-conforming with respect to all Templates referenced via `--type`. A document with no Template references is fully conforming at Layer 1 + Layer 2a if it is representation-conforming. **Layer 2b conformance is always vocabulary-qualified**: a document is "Layer 2b-conforming with respect to vocabulary V" when consumed by a V-aware consumer that finds no constraint violations. Claims of Layer 2b conformance MUST name the vocabulary against which the claim is made.

The layers are independent: a document may be Layer 1-conforming but Layer 2a-non-conforming (well-formed but not satisfying its Template's structure), and Layer 2a-conforming but Layer 2b-non-conforming for one vocabulary while Layer 2b-conforming for another (same data; one vocabulary's constraints pass, another's fail). A conforming implementation MUST report and handle these verdicts distinctly; it MUST NOT collapse them into a single valid/invalid answer.

### A.2 Serialization Conformance

A serialization is **representation-conforming** if and only if a round-trip through it preserves, for every element in the document:

1. The Prefix.
2. The presence and value of `Key`, `Rel`, `Id`, and `Type` (string-equal, ordinal comparison). An @-edge's empty Rel (§6.4.2) round-trips as empty — the field is not invented, defaulted, or dropped.
3. The Resource — including:
   - The null state (an element with no Resource value MUST round-trip as a null-state element).
   - The value-kind of the Resource (per §3.3).
   - The value of the Resource (per §3.3 and §4.2).
   - Any representation tag carried by the Resource (numeric width suffixes, bytes encoding prefixes).
4. The ordered list of children (recursively conforming).
5. Annotation (REM) elements in their original positions.

A serialization MAY normalize:

- Whitespace within and between elements (per §4.1.2).
- Hex digit case in bytes literals (canonical form is lowercase).
- Boolean literal forms (canonical form is `true`/`false`; serializers normalize `t`/`f` and uppercase variants to lowercase spelled-out form).
- Null Property forms (canonical form is the bare `<STR name>`; serializers normalize the empty-value form `<STR name : >` to the bare form).
- Indentation in Standard or Compact forms.
- The choice of Standard, Compact, or Binary form (so long as the chosen form is itself conforming).
- The metadata channel (§4.8). Metadata is out-of-model: a serialization MAY add, alter, or drop metadata blocks without affecting representation-conformance. A serialization that preserves metadata is not thereby *more* conforming, and one that drops it is not *less* conforming — the round-trip contract above ranges over model content only. (Preserving metadata is nonetheless RECOMMENDED where downstream carriage depends on it; §4.8.4.)

A serialization MUST NOT alter:

- Numeric representation suffixes. A value stored as `25i8` is retrieved as `25i8`, not as `25` or `25i64`.
- String content within quotes. Escape sequences are part of the literal and round-trip exactly.
- Element identity, order, or structure.
- The semantic distinction between null and present-with-value. The bare and empty-value forms of null are interchangeable; null and present-with-value are not.

### A.3 Parser Conformance

A parser is **representation-conforming** if it:

1. Accepts every document well-formed under §3 through §8.
2. Rejects every document that violates a MUST clause in §3 through §8 and §4.4.
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
12. Prefixes appear in **canonical form** — three-letter (or longer where natural) names in uppercase: `STR`, `INT`, `BOOL`, `FLOAT`, `BYTE`, `REC`, `OBJ`, `GRP`, `STR*`, `INT*`, `.STR`, `_STR`, `VAL`, `NSTR`, `NINT`, `NBOOL`, `NFLOAT`, `NBYTE`, `NREC`, `DEFSTR`, `DEFINT`, `DEFBOOL`, `DEFFLOAT`, `DEFBYTE`, `DEFREC`, `DEFOBJ`, `REL`, `REM`, `PKG`, `BLK`, the @-Edge family (`@OBJ`, `@REL`), and the REF family (every `%KIND` form generated by `%` wrapping a layer-1 role-prefix). The minified single-letter forms (`S`, `I`, `L`, `F`, `B`, `R`, `O`, `G`, `S*`, `.S`, `_S`, `V`, `NS`, etc.) are valid GDX but do not produce strict canonical bytes; conversion to canonical form is required before content addressing. The `E` minified slot is unassigned (formerly the minified form of `EDGE`, now removed in v0.12.0; see changelog entry 41).

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

- Valid documents at every prefix in §5 (one positive test per prefix).
- Identifier forms: local names, single letters, paths, CURIEs, GUIDs, IRIs (one positive test per form per applicable position).
- Literal forms: each value-kind including representation suffixes (`i8` through `u64`, `bigint`, `f32`/`f64`, the `d`/`d32`/`d64`/`d128` decimal family), boolean variants (`true`/`false`/`t`/`f` and case variants), bytes (`0x` and `0z` forms), null forms (bare and empty-value).
- Sigil tightness: positive tests for valid `&`, `#`, `--`, `@` usage; negative tests for whitespace violations.
- Empty rels on edges: an @-Edge element with no `#rel` accepted (positive); bare `#` rejected everywhere (negative); round-trip preserves the empty Rel without inventing or defaulting a predicate (§6.4.2); Template-declared predicate fills the empty field under the claim rule and materialization (§8.2.3, §8.2.4).
- Whitespace handling: documents with various legal whitespace patterns; round-trip equivalence to canonical form.
- Comments: file-level `#` comments in Compact Form (positive); rejection of comment-position `#` in Standard Form (negative).

**Composition tests (Layer 1):**

- Each row of the §7.1 composition matrix represented as a positive test (parent role contains valid child role) and at least one negative test (parent role contains invalid child role).
- Top-level element restriction: positive tests for Object/Template/REL/Package as document root; negative tests for other roles at document root.
- Set member rules: positive tests for homogeneous Set members; negative tests for heterogeneous members where applicable.
- Node subtree isolation: positive tests for Node-only subtrees; negative tests for structured roles appearing within Node subtrees.

**Resource and value tests (Layer 1):**

- Each value-kind with edge values: zero, negative numbers, max/min values per width, infinity, NaN, very long strings, empty strings, empty bytes.
- Reference resolution within a single document: positive tests where targets exist; tests for unresolved references (which produce Layer 2 errors per §6.4.1, not Layer 1).
- Round-trip preservation: a document parsed and serialized to canonical form, then re-parsed, produces an identical data model.

**Schema validation tests (Layer 2):**

- Template conformance: instances claiming `--type` references that resolve and satisfy / fail the Template's constraints.
- Resolution order: tests confirming the §8.5.2 default order is followed (local document first, then loaded vocabularies, then global IRI).
- Resolution failure: tests confirming unresolved type references produce Layer 2 errors per §8.5.3.
- Validator modes: strict and lenient mode behavior on the same input documents.
- Error accumulation: validator returns multiple errors from a single pass (per §8.5.4).
- @-Edge reference resolution: tests confirming §6.4.1 behavior for resolved and unresolved `@OBJ`/`@REL` references, including role-mismatch detection (an `@OBJ` resolving to a REL is unresolved).

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
- Reference cycles in @-Edge graphs: cycles are valid data; tests confirm parsers accept them and traversal-based operations terminate.

**Compact Form tests (when supported):**

- Equivalence: every Standard Form test document, converted to Compact Form, should parse to the same data model.
- Indentation: documents with various consistent indent units; rejection of mixed indents.
- Multi-line strings (`|` and `|<` forms).
- Round-trip: Compact → Standard → Compact (data-model preservation; comment loss is expected per Compact Form spec).

The test suite is exhaustive in concept and bounded in practice. Each category above expands to dozens of specific tests when fully populated. A conformance test suite covering these categories is a substantial deliverable; implementations may bootstrap from a subset and expand coverage over time. The intent is that an implementation passing the full suite has demonstrably correct behavior on the cases that matter for interoperability.

---

## Appendix B — Detailed Revision History

This appendix records the per-revision change descriptions accumulated during the pre-1.0 working-draft phase. The Revision History table at the top of this document gives the version + date summary; this appendix carries the prose detail. Once the specification reaches v1.0, this appendix will be archived and subsequent revisions will use a leaner change-log format.

### v0.23.0 — Realization metadata channel (`{...}`)

(53) **A per-element metadata channel is added: a brace-delimited block of flat key/value pairs, written last, carrying out-of-model carriage annotation** (persistence identifiers, stream/instance identifiers, delta-handling state). The channel fills a real gap — previously the only homes for such information were Extensions (model content, wrong: they travel with equality, signing, and round-trip) or a reserved-name hack in the data namespace (the CouchDB `_id`/`_rev`, Mongo `_id`, OData `@odata.etag` pattern, with its documented leakage and collision costs). The design mirrors USD's metadata dicts and the HTTP header/body split: a lexically distinct, structurally ignorable channel.

- **New §4.8 (Metadata).** Defines the channel: out-of-model status as a Layer 1 structural fact (§4.8.1), grammar (§4.8.2), the representational-not-semantic typing boundary (§4.8.3), and positions/persistence/signing consequences (§4.8.4). Syntax: `{ key = literal, key = literal }` — keys are identifiers (CURIEs admitted for namespacing), values are the five Resource-literal value-kinds (no references), `=` separates key from value and `,` separates pairs. The channel is deliberately impoverished (flat, no nesting, no elements, no references, no semantic typing); the poverty is what makes the not-data claim credible and routes richer needs to Extensions or Records.

- **Element tuple and field model updated (§3, §3.1).** The element tuple gains a trailing `Metadata?`; the §3.1 field table gains a `Metadata` row explicitly marked out-of-model. The six model fields are unchanged; metadata is set apart as a channel, not a seventh model field.

- **Field order and lexis (§4.1, §4.1.1).** Write order becomes `Prefix · Key · Rel · Id · Resource · Type · Metadata`; the general form and the sigil/separator table are updated (`{ }` delimit the block). Trailing position lets a reader or stripping tool consume all model content before the annotation.

- **Persistence and conformance (§10.2, §A.2).** §10.2 notes that store-native columns (persistence id, row version, delta state) are the at-rest form of the metadata channel and sit outside the round-trip contract. §A.2 adds metadata to the MAY-normalize set with an explicit out-of-model note: adding, altering, or dropping metadata does not affect representation-conformance, which ranges over model content only. The canonical form (§A.5) excludes metadata, so signatures over canonical content cover model content only.

Migration: purely additive. No existing document changes; documents with no `{...}` block are unaffected. Implementations that round-trip model content remain conforming; those that wish to carry metadata gain a defined channel. The Compact and Binary Form companions SHOULD define a flat key/value metadata block aligned to §4.8; a realization MAY omit the channel and remain model-faithful.

### v0.22.0 — Key field: claim-rule exclusion, unified token grammar, embedded-Object scope

(52) **Three corrections to the Key mechanism, closing a materialization defect and two underspecifications.** Findings from a red-team review of the Key field against prior art (JSON Schema `$id`/`$anchor`, YAML anchors, `xml:id`, protobuf field numbering).

- **`Key` removed from the universal claim rule (§8.2.3, §8.2.4, §8.2.1).** Templates could previously declare a `Key` value on a slot, reconciled by the claim rule and written into instances by materialization. This was incoherent: Keys are scope-unique (§4.4.5) while Templates are reusable shapes, so a Template-declared Key materialized into two same-typed instances within one Object scope produced a Key collision — a malformed document generated from conforming inputs by a RECOMMENDED operation. The claim-bearing field list is now `Rel`, nested `--type`, and default `Resource`; a normative exclusion paragraph in §8.2.3 records the rationale (Templates MUST NOT declare a `Key` on a slot). This removes the last residue of the pre-v0.11 Template-slot-alignment role of Keys (entry 7).

- **Key token grammar unified; the “numeric form” framing removed (§4.4.3, §4.2.6, §4.4).** The Key field previously admitted two forms — standard identifiers and all-digit “numeric Keys” — which left digit-leading mixed tokens (GUIDs such as `7f3a2b9c-…`, already used as examples in §4.2.6 and §4.4.3) formally admitted by neither form. The Key token is now a single grammar: an identifier token whose start set additionally admits digits (`KeyToken ::= (ALPHA | DIGIT | "_") IdContChar*`). The relaxation is lexically safe because Key tokens are always `&`-introduced, so the digit-start/numeric-literal ambiguity behind the `IdStartChar` restriction cannot arise. Key tokens carry no numeric interpretation; comparison remains ordinal (`&01` ≠ `&1`). Authority for `KeyToken` moves from `GDX-Path.md` into §4.4.3 (the field's home); `GDX-Path.md` retains `PathExpression` and the resolution algorithms and MUST be aligned to the §4.4.3 grammar in its next revision.

- **Key scope pinned for embedded Objects; single-scope lookup made explicit (§4.4.5).** An Object occupying a VAL slot opens a fresh Key scope: its interior Keys are scoped to it, while its own Key belongs to the host's scope. `KeyRef` resolution consults exactly one scope — no outward walk, no shadowing, no fallback — and cross-boundary addressing uses `IdKeyRef`. Previously the nearest-enclosing-container rule left the VAL-occupant case implicit and the single-scope discipline unstated, inviting divergent lexical-scoping implementations.

Migration: Templates that declare `Key` on slots MUST remove the declaration (instances carry their own Keys). Documents using digit-leading GUID Keys — previously present in the specification's own examples but admitted by neither lexical form — become formally valid. No other document changes.

### v0.21.0 — Matrix/prose consistency pass; Template placement pinned; `%PKG`/`%GRP` confirmed

(51) **Hygiene revision resolving the internal contradictions accumulated across the v0.11–v0.13 reworks, in preparation for implementation conformance work.** No new constructs; several rules are clarified or pinned where prose and the §7.1 matrix disagreed. Sub-items:

- **Stale Extension restriction removed (§7.5, §6.14).** §7.5 still rejected "an Extension appearing outside a Template parent" and §6.14 still claimed Templates were "the only role" with Extension children — both leftovers from before the adjunct-Extension broadening (v0.11 entry 38), contradicting §6.10 and the matrix. The §7.5 bullet is deleted (the general matrix bullet covers Extension placement); §6.14 is corrected.
- **Qualifier children (§6.9).** Prose said "Annotations only"; the matrix admits Extensions and Annotations. Resolved in favor of the matrix, consistent with the adjunct-Extension principle: Qualifiers admit Annotations and Extensions.
- **Set row admits Record (§7.1).** The matrix Set row omitted Record, contradicting `REC*` (§6.13) and rule 6. Record added.
- **Block containment widened to REL (§6.5, §7.2 rule 3).** Rule 3 said Object-or-Block only, contradicting the REL row and §6.15's admitted-children list (v0.13 widening). Rule 3 and §6.5 now name Object, Block, and REL (and `DEFOBJ` via rule 9).
- **Template and Package child columns added; Template row removed (§7.1, rule 9(c)).** The matrix previously could not express "Package admits Templates" or the root rule's Template/Package eligibility (no columns), while carrying a single Template *row* that contradicted rule 9 (one row cannot serve `DEFOBJ`, `DEFREC`, and the Property Templates, whose base rows differ). Resolution: a `DEF…` element's *children* are validated by its base role's row (rule 9 transparency — so `DEFOBJ` admits Blocks exactly as Object does); its *placement* is fixed by the new Template column — Package and Group payload and the document root only. New rule 9(c) states the placement rule explicitly: declarations live at declaration positions; data-layer nesting of `DEF…` (e.g., a `DEFREC` inside an Object) is malformed.
- **REF admitted under Package and Group (§7.1, §6.16).** The matrix omitted REF from the transport rows while §6.16 said REFs follow the Property row (and Property is admitted there) — REFs are useful as transport metadata handles (manifest entries, correlation pointers). The rows gain REF; the §6.16 admitting-parent list is corrected to the full set (it had also omitted Set and REL).
- **`%PKG` and `%GRP` confirmed as REF-family members (§5.2, §5.3.3, §6.16).** The generation rule ("% wraps any layer-1 role-prefix") produces them, and the §4.2.6 anchor-role set already included PKG and GRP, but the family enumerations omitted them. They are now listed, with a lifecycle note: transport wrappers are transient, so such REFs are meaningful within the transport context that carries them and SHOULD NOT be persisted beyond it.
- **Example-annotation marker normalized (` -- ` → ` ← `).** Example blocks used ` -- ` as an inline annotation marker — visually colliding with the `--` Type sigil and producing non-parseable example lines. All example annotations now use ` ← `, the marker §6.10's examples already used. GDX itself remains comment-free (§4.6).
- **Mechanical staleness:** §8 status note renumbered (pre-v0.17 "9.1–9.5" → "8.1–8.5"); Appendix A.7's matrix reference corrected (§8.1 → §7.1) and its suffix enumeration completed (`bigint`, `d`/`d32`/`d64`/`d128`); §6.12's "for v0.9 conformance" version pin dropped.
- **`%KIND*` pinned to REF-to-Set; the Set-of-REFs reading removed (addendum).** §6.16's example had given `%KIND*` two readings (children = Set of REFs; Resource = REF to a Set). Resolved: only the REF-to-Set reading exists — `%STR*` is a REF resolving to a `<STR* …>` element; multiple handles are sibling REFs. Consequently `%KIND*` is valid only where the wrapped Set prefix exists (value primitives, REC, modifier Sets): `%DEF…*`, `%REL*` (Templates and RELs are root-level declarations — a Set of them does not make sense), `%OBJ*`, and `%@OBJ*` are malformed. §5.2, §5.3.3, §6.16, rule 13 updated; the Set row's REF admittance (a v0.13 artifact of the removed reading) is withdrawn from the matrix.
- **§8.2.4 example aligned with the absent-slot rule (addendum).** The materialization example showed a Template default materializing an *absent* slot into existence, contradicting the normative "missing slots are not added." The example now shows the `locale` slot present-but-null in the authored instance; new prose states the rationale (the instance is the reality; absence may be deliberate; materialization refines what is present and never resurrects what is not) and distinguishes creation-time *instantiation* (tooling imparts defaults into a new instance) from materialization.
- **Rule 16 aligned with v0.20 (addendum).** §7.2 rule 16 still said an @-Edge "MUST carry a `#rel` predicate" — missed in the v0.20 empty-rel pass. Corrected: the field may be empty per E4/§6.4.2.
- **§7.4 Set-resource contradiction resolved (addendum).** The five family bullets listed Set forms (`STR*`, `INT*`, …) as resource-carrying while the closing paragraph correctly listed Set among the no-Resource roles (and §6.13 defines a Set's content as its members). Set forms removed from the family bullets; an explicit no-Resource bullet added for Sets.
- **§5.4 stale `@`-composition sentence corrected (addendum, found during implementation conformance).** §5.4 still said "the reference operator `@` may prefix any modifier-base composition: `@.STR`, `@_BOOL`, `@NSTR`" — a pre-v0.13 leftover contradicting rule 14 and axiom E5 (the @-Edge family is closed at two members). Corrected to the `%` sigil, which is the operator that wraps modifier-base compositions.

Migration: none for documents. One previously-underspecified case is now explicitly malformed (a `DEF…` element nested in a data-layer position, per rule 9(c)) and one previously-contradicted case is now explicitly valid (Extensions outside Templates, Record members in `REC*` Sets, Blocks under REL, REFs under transport wrappers) — implementations should re-derive their composition tables from the §7.1 matrix as revised.

### v0.20.0 — Empty rels on @-edges (the existence/specification split)

(50) **An @-edge's `#rel` field MAY be empty — the relationship exists by role, undefined as to nature.** Previously E4 made the predicate mandatory on every @-edge. This revision splits what that rule had fused: the *role* (`@OBJ`, `@REL`) asserts that a relationship **exists** — an @-edge is, by its role, a relationship — while the `#rel` field **specifies** what the relationship is, and may be empty. An edge with no `#rel` parses to a null Rel, is well-formed, and round-trips as empty; no special syntax and no distinct construct are involved. The empty rel is never ambiguous about whether a relationship exists (the role settled that); it is silent only as to the relationship's nature. Later refinement fills the field on the same edge — edges are identity-bearing entities (E6), so the edge's identity and any references to it survive refinement; nothing is created or replaced.

The motivating need: an edge must sometimes be created before the form of the relationship is known (real-time capture, pending classification, a connection observed before its nature is determined). Forcing a fully-specified rel at creation time means inventing a placeholder or failing to capture the connection; both are worse than recording "connected, nature to be determined."

Normative changes:
- **§3.1** — the existence/specification split stated at the model layer: role asserts existence; Rel specifies nature and may be empty (null).
- **§4.1.1** — sigil tightness restated: a bare sigil is malformed everywhere; an empty field is expressed by omitting the sigil.
- **§6.4 E4** — rewritten from "predicate is mandatory" to "role asserts existence; `#rel` specifies nature and MAY be empty." The construct-discrimination function the mandatory field once served is obsolete since the v0.13 sigil partition (`@` is edges, `%` is REFs). The empty rel is not an idiom for permanent type-free adjacency — that remains a designated predicate value.
- **§6.4 E6** — identity with an empty rel is well-formed and is the canonical capture-first pattern.
- **§6.4.2 (new), "Undefined Relationships — the Empty Rel"** — the split, the motivation, the refinement model (same edge, field filled; `Id`/`Key` RECOMMENDED for addressability), and the three disciplines: queryable-as-undefined (MUST), schema-may-require-specification (vocabulary constraint, §8.4), staleness-is-tooling. Empty rels are a transitional state — easy to create, easy to find, easy to require resolution of.
- **§6.4 identifier table, §5.3.2** — `#rel` is OPTIONAL (empty = undefined nature).
- **§7.5** — a conforming parser MUST accept an @-Edge element with no `#rel` field.
- **§8.2.3** — an empty instance rel is the instance-silent case of the universal claim rule: a Template-declared predicate supplies the name (case 3) and materialization (§8.2.4) fills the field on the same edge; a vocabulary constraint may instead forbid undefined relationships (Layer 2b).
- **Appendix A.2 / A.7** — an empty Rel round-trips as empty (not invented, defaulted, or dropped); test-suite outline updated.

Migration: none. The change is additive permissiveness — documents previously rejected for a missing edge predicate become well-formed; no existing well-formed document changes meaning.

### v0.19.0 — REF as value-resolution; link-construct rationale recorded

This revision reconciles the Link-Types design memo (REF/EDGE, 2026-05-23) against the v0.12–v0.13 revisions that implemented its sigil reallocation. Most of the memo had already landed: `@` is the edge sigil, `%` the reference sigil, embedding is a role, edge targets are role-explicit and name-addressed only, REL-to-REL axioms are edges. The memo's "EMBED role" is the VAL role (§6.11) under its adopted name; the memo's open verification item — that the role form preserves everything the `%`-embedding sigil expressed — was discharged by the v0.12 VAL design (two-level identity, closed occupant enumeration; changelog entry 40). Three items from the memo are integrated here:

(47) **§6.16 reframed: REF is GDX's value-resolution construct.** The section lead now defines a %-Reference by what it asserts — "the value here is the value of the resolved target" — with the slot-filler/typed-handle description retained as its structural realization. Axiom R1 is reworded to layer the two views instead of opposing them: what the slot *stores* is the address (the representation-layer content that serializes, signs, and round-trips per A.2); what the slot *means* is the resolved target (dereferencing yields the target element and, for value-Property targets, its value). The prior "never a value-kind-to-fetch" sentence, which denied the value-resolution reading outright, is replaced. Two clarifications from the memo are adopted: REF is defined by value-resolution, *not* by "non-identifiable targets only" (interior fields are the most common use, not the definition); and the capability boundary is the conceptual boundary — REF-into-a-field is coherent, EDGE-into-a-field is incoherent, which is the structural evidence the two link constructs are genuinely different. A non-normative pointer on choosing REF vs @-Edge in borderline cases routes to the Primer and Cookbook. The §6.16 summary paragraph is aligned. No grammar, vocabulary, composition, or conformance requirement changes.

(48) **§5.3.2 ADR: the `@` sigil marks the @-Edge family.** Records the three grounds for the v0.12–v0.13 sigil reallocation (origin: the edge is GDX's original link construct; frequency: the primary use cases are edge-shaped; centrality: GDX is a property graph with a light ontology layer) against the rejected alternative of keeping `@` on references, so the reallocation is not mistaken for arbitrary churn.

(49) **§6.4: both endpoints are role-explicit.** A new paragraph makes explicit what the model implied: the source endpoint's role is self-declared by the containing element's prefix, the target endpoint's role is declared by the @-edge prefix and verified at resolution (E1), and any source/target combination admitted by the composition matrix is expressible. The same paragraph states why interior (`$path`/`&key`) addressing is barred at edge endpoints: an interior has no role-declaring definition for the endpoint claim to verify against.

Migration: none. Documents and implementations conforming to v0.18.0 remain conforming.

### v0.18.0 — §1.1 reframed around model primacy and realizations

§1.1 (What GDX Is) is rewritten to lead with the model/realization relationship and the document's role relative to it. The opening now states explicitly that the model is the primary, authoritative thing and that the text form, binary form, and in-memory form are *realizations* of it — concrete forms the same abstract content takes — rather than co-equal "representations." A second paragraph makes the consequence explicit: the specification defines a model together with the rules a realization must satisfy to carry it without loss, and conformance (deferred to Appendix A) is layered. The orienting framing — "GDX is not, at bottom, a file format; it is a model, and a realization conforms by carrying that model faithfully" — is introduced here and developed fully in the GDX Primer. This sharpens the model-primacy that §3's intro (the data model is the single authoritative content every realization faithfully renders) already stated normatively; §1.1 now introduces it at the top rather than leaving it to §3. No normative semantics change; the conformance definition in Appendix A is unchanged and authoritative.

### v0.17.0 — §1.4 removed; §3 stub removed and sections renumbered

Two cleanups completed the spine/primer separation begun in v0.16.0. No normative semantics change; documents and implementations conforming to v0.16.0 remain conforming.

- **§1.4 (Where GDX Fits) removed.** The applicability discussion — a "well-suited to / not the right tool for" use-case enumeration — was editorial positioning, not a normative requirement, and was the only such content in the document. It is not replaced; a specification adjudicates conformance and does not prescribe where the format should be applied. The former §1.5 (Scope of This Document) is renumbered §1.2, closing the §1.2/§1.3 gap the v0.16.0 split had left in §1.

- **§3 relocation stub removed and sections renumbered.** v0.16.0 retained an empty §3 slot (a "moved to the Primer" notice) to preserve section numbering. That stub is now removed and the following sections renumbered to close the gap: the former §4–§12 become §3–§11. The mapping is a uniform decrement by one for all sections at or above the old §4:
  - §4 Abstract Data Model → §3
  - §5 Canonical Text Form → §4
  - §6 Prefix Vocabulary → §5
  - §7 Element Roles → §6 (the REF genus split and proof-by-collapse, formerly §7.16, is now §6.16; the @-Edge family, formerly §7.4, is now §6.4)
  - §8 Composition Axioms → §7
  - §9 Schemas and Validation → §8
  - §10 Transport Semantics → §9
  - §11 Data Persistence → §10
  - §12 Open Issues → §11
  All internal cross-references and the table of contents were updated accordingly. **Companion documents that cite specification sections by number (the Path specification, the Cookbook, and any others) must re-point their references using the mapping above.** Appendix B entries below this one are frozen historical records and retain the section numbers that were correct at the time each was written.

### v0.9.7

§1 (Introduction) restructured into §1.1 What GDX Is, §1.2 Existing Approaches, §1.3 GDX Design Goals, §1.4 Where GDX Fits, §1.5 Scope. Conventions and Terminology promoted back to top-level §2, scoped to term definitions only (Core Terms, Conformance Layers, Vocabulary Terms, Identifier Schemes). The GDX Element and Core Data Model renumbered to §3 and §4. §6 renamed to "Element Kinds." Field Sigils and Separators (previously in §2.7 of Conventions) absorbed into §3.4. No normative semantics change.

### v0.9.8

The technical term "Kind" (naming the closed enumeration of Object, Record, Edge, etc.) renamed to "Role" throughout. "Kind" was ambiguous against generic English usage and competed with "value-kind." §6 renamed from "Element Kinds" to "Element Roles"; §6.3 from "Kinds in Detail" to "Roles in Detail." §4.1 renamed from "Field Roles" to "Element Fields" to avoid collision with the new technical term; its column header changed from "Role" to "Purpose." The §3.4 sigil table column header changed from "Role" to "Purpose" for the same reason. Sub-subsection numbering in §4.3 corrected from the stale §3.3.x to §4.3.x. The term "value-kind" (the value-category of a Resource) is preserved unchanged; only the structural-position concept renamed. No normative semantics change.

### v0.9.9

Sharpened the conceptual / data-model / text-form boundary across §3, §4, §5. Each section now has a distinct focus, signaled by its title: §3 "GDX Concepts" defines what GDX expresses; §4 "The GDX Abstract Data Model" defines what an implementation represents at runtime; §5 "Canonical Text Form" defines how elements are written. Substantial content moved between sections to eliminate the three-way overlap that had accumulated. The Sigils and Separators subsection moved from §3.4 to §5.1.1 (sigils are textual mechanics, not conceptual). The Real-Number Representation subsection from §4.3.2 merged into §5.2.3 Float Literals (suffix mechanics are textual). The two textual forms of null values (bare and empty-value) moved from §4.3.4 to a new §5.1.3, while the three-state Property semantics (Absent/Null/Present-with-value) remain in §4. A new §3.4 "The Implicit Parent-Child Relationship" promotes a conceptual point previously buried in §3.2's closing paragraph. Cross-references throughout the spec updated accordingly. No normative semantics change.

### v0.9.10

Corrections: (1) §8.4 Resource Rules erroneously listed DEF prefixes (`DEFSTR`, `DEFINT`, etc.) as Resource-bearing; Templates declare a contract for instances and do not carry an instance value themselves. The DEF prefixes have been removed from the resource-family bullets and added to the list of roles that do not carry Resources. (2) Appendix C.2 was reframed from "Reference Cycle Detection" (a generic implementation-correctness concern) to "Reference Cycle Exploitation," emphasizing the adversarial framing consistent with the rest of Appendix C. (3) Several Quick Start examples in Appendix D (§D.1, §D.2, §D.3, §D.4, §D.6, §D.7, §D.8) were updated to wrap their content in an Object or Template root; the previous examples placed bare Properties, Records, or Sets at the document root, which violates the conformance requirement that a document's root MUST be an Object, Template, REL declaration, or Package (§8). No normative semantics change.

### v0.10.0

Substantial changes:

(1) **Reference operator `@` added.** The `@` prefix produces typed references, joining the existing modifier vocabulary. The `@KIND` family composes universally with role bases and modifier-bearing bases (subject to a `@@` prohibition: references cannot target other references). The composition reads as "a reference to a KIND-element." Closed-vocabulary expansion: the §6 specification describes the composition pattern rather than enumerating every cell; valid `@KIND` prefixes are derived from the pattern. This adds the reflexivity primitive — any addressable element can be referenced by any other element through a typed pointer.

(2) **Set notation moved to suffix.** The Set marker `*` is now a suffix: `STR*`, `REC*`, `*INT` notation is replaced by `INT*`. This is a breaking change. The suffix position reflects Set's structural distinction from the refinement modifiers — `*` is value-kind-redirecting (creates a collection) rather than value-kind-preserving. The suffix also unlocks compositions previously forbidden by the no-modifier-stacking rule: `.STR*` (Set of Qualifiers), `_INT*` (Set of Extensions), `@STR*` (reference to a Set) are all valid.

(3) **§3 trio framing added.** New §3.6 "Embedding, Referencing, and Graph Relationships" articulates the three distinct mechanisms for involving an addressable element from within another: `%X` for inline embedding, `@X` for typed references, EDGE for graph statements. Each has a distinct structural shape and use case.

(4) **§6 restructured around the composition pattern.** The §6 intro introduces the prefix pattern `[@?][refinement-modifier?][base][*?]`. §6.1 (Canonical/Minified Forms) and §6.2 (Base Vocabulary) cover the core role vocabulary. §6.3 (Special Operators) introduces the `*` suffix and `@` operator. §6.4 (Refinement Modifiers) covers `.`, `_`, `%`, `N`. §7 (Roles in Detail) is the detailed per-role treatment, renumbered from old §6.3.x with `@KIND` added as §7.16. The base vocabulary is grouped by category: value primitives, structural roles, declaration roles, transport roles.

(5) **Reference literal grammar reserves `$` for path expressions.** The reference literal grammar accepts `@Identifier` (normative in this revision) and reserves `$PathExpression` for a future revision. Path-mode resolution is an open issue (§12).

(6) **§7.4 EDGE clarified.** A note distinguishes EDGE (structured graph-statement element with its own children) from `@OBJ` (leaf-like typed pointer). Both carry references to Objects but have different structural shapes.

(7) **§8.1 Composition Matrix expanded.** New Reference column and row reflect the leaf-like nature of `@KIND` elements (Qualifiers and Annotations as children only). REL and Template rows admit Reference children for vocabulary metadata.

(8) **§8.2 rules updated.** Rule 12 reframed to describe the refinement-modifier-non-stacking constraint precisely. New rules 13 (Set as suffix), 14 (universal `@` composition), 15 (`@@` prohibition), 16 (References are leaf-like).

(9) **Migration impact.** All `*PREFIX` notation in existing documents must migrate to `PREFIX*`. The `@KIND` family is additive and does not affect existing documents. EDGE is unchanged. Templates and RELs are unchanged.

(10) **Appendix split.** The previously-included Appendices B (Format Comparison), C (Security Considerations), and D (Quick Start) have been extracted into separate companion specifications, joining the existing Grammar, Compact Form, and Cookbook companions. The previously-included Appendix A (Conformance) remains in the core specification as normative content. Cross-references throughout the spec updated to point at the new companion documents. The split reduces the core specification by approximately 700 lines and clarifies the boundary between normative content (this specification, Appendix A, Grammar) and non-normative content (Format Comparison, Security Considerations, Quick Start, Cookbook).

### v0.11.0

(1) **Path-mode and key-mode reference resolution are now normative.** The `$PathExpression` form reserved in v0.10.0 §12.4 is fully specified in a new companion document, the GDX Path Expressions Specification (`GDX-Path.md`). The path grammar supports three segment kinds — `IdSegment` (bareword, matches `Id`), `RelSegment` (`#R`, matches `Rel`), and `OrdinalSegment` (`[N]`, matches by 1-based document order). The `&Key` reference literal provides key-based addressing through the location-stable `Key` anchor. Five reference-literal forms are recognized: `IdRef` (`@Anchor`), `PathRef` (`$PathExpression`), `KeyRef` (`&Key`), `IdPathRef` (`@Anchor$PathExpression`), and `IdKeyRef` (`@Anchor&Key`). The composed forms express cross-document addressing. The `@Anchor` production carries an optional role marker (`@OBJ:id`, `@REL:id`, `@DEFOBJ:id`, etc.) drawn from the closed prefix vocabulary; when omitted, composed references default to an Object anchor. The marker enables composed references to non-Object anchors (RELs, Templates, transport-layer containers) without requiring resolver inference.

(2) **§12.4 closes.** Path-mode reference resolution is no longer an open issue. References to §12.4 in earlier revisions are satisfied by the companion specification.

(3) **`/` removed from `IdStartChar` (§5.4.1).** Breaking change. Identifiers may no longer begin with `/`. The `IdContChar` set is unchanged, so identifiers may continue to contain `/` (e.g., IRIs like `https://acme.com/people/alice` remain valid). The narrowing makes the `$/...` path-mode form unambiguous and the `@Identifier$PathExpression` boundary lexically clean. Path-style local identifiers such as `<OBJ /people/alice>` MUST be migrated; the recommended migrations are documented in `GDX-Path.md` Appendix C.

(4) **§5.4.2 identifier-forms table updated.** The "Path" row is removed; path-style local naming is replaced by path-mode reference resolution. The remaining identifier forms (Local name, Single letter, CURIE, Dotted CURIE local, GUID/opaque, IRI) are unchanged.

(5) **§5.2.6 reference literal grammar updated.** The five-form `ReferenceLiteral` production replaces the placeholder. Address-mode singularity is incorporated by reference to `GDX-Path.md` §8.

(6) **§8.1 composition matrix expands the resolvable target set.** `@KIND` references composed with a `PathRef`, `KeyRef`, `IdPathRef`, or `IdKeyRef` resource may now resolve against Records, Blocks, value Properties, Qualifiers, Extensions, and Embedded elements. Parent-context restrictions on where `@KIND` reference elements may appear are unchanged; only the addressability of the target expands.

(7) **`Key` field reframed from Template-slot identifier to location-stable anchor.** The §3.2 description of `Key` is rewritten: Keys are an opt-in named-anchor mechanism providing stable reference targets for elements that lack global identifiers. The Template-alignment justification is removed. A new §5.4.5 defines Key uniqueness rules (scoped to the enclosing Object, or to the document at transport-layer position). The numeric Key form is retained as a lexical convenience but no longer carries Template-slot semantics.

(8) **§9.2 Template-instance alignment pinned.** A new normative paragraph specifies that alignment is by `Id` matching (case-sensitive ordinal), with `Rel` matching as a cross-vocabulary fallback. The `Key` field plays no role in alignment.

(9) **§5.1.1 sigils table updated.** The `$` row is updated from "reserved, grammar deferred" to "introduces a path-mode reference literal (§5.2.6, `GDX-Path.md`)."

(10) **Migration impact.** Existing documents using leading-slash identifiers MUST be migrated (drop the leading slash, retaining any interior `/`). Existing documents using only `@Identifier` references are unaffected. Existing documents that used `&K` numeric Keys for Template-slot alignment MUST be migrated to named-Id Template slots; the `Key` field continues to be lexically valid but no longer carries slot-alignment semantics. Templates that previously used `&1`, `&2` positional slots are migrated to named slots (`firstName`, `lastName`, etc.). Existing documents that used a non-normative `$`-prefixed literal under v0.10.0 §12.4's implementation-defined window MUST conform to the v0.11.0 grammar.

(11) **§1.3 adds the substrate-with-pluggable-constraints positioning.** A new design-goal paragraph names GDX as a structural schema (the prefix vocabulary, element model, Template structure, composition axioms) with no constraint implementation. Constraint vocabularies layer on top as Constraint Extensions whose identifiers GDX records but never interprets.

(12) **§2.2 decomposes Layer 2 conformance.** The single "Schema Conformance" layer splits into Layer 2a (Template structural conformance, owned by GDX) and Layer 2b (constraint-vocabulary conformance, owned by the constraint vocabulary). The decomposition makes explicit that an instance may be Layer 2a-conforming while Layer 2b-non-conforming for one vocabulary and Layer 2b-conforming for another; verdicts are vocabulary-relative above the primitive floor. Appendix A is updated to mirror the split.

(13) **§8.5 closes with a primitive-axioms statement.** A new closing paragraph names the §8.5 validation requirements as the GDX-native floor (Layer 1) and points to §9 for the schema layer.

(14) **§9 gains a layered-intro paragraph.** A new paragraph at the top of §9 frames the section in three layers — Template structure, Template-instance alignment, and Constraint Extensions — and ties each to the §2.2 conformance layers.

(15) **§9.4 renamed and rewritten.** Previous title "Validation Extensions" becomes "Constraint Extensions" to reflect that GDX hosts, vocabularies validate. New normative MUST: GDX implementations MUST NOT interpret constraint semantics; constraint identifiers are opaque to GDX. A new §9.4.1 "Parallel Constraint Vocabularies" formalizes the multi-vocabulary coexistence property and gives a worked example showing XSD, JSON Schema, and a vendor vocabulary on the same Template. §9.4.2 covers validator behavior on recognized vs unrecognized identifiers.

(16) **§9.6 updates Common Vocabulary framing.** The Common Vocabulary is positioned as one vocabulary among many — it ships with GDX as a convenience and reference but has no privileged status at the GDX layer. `gdx:` constraint identifiers are opaque to GDX in the same way as `xsd:`, `json:`, or any other.

(17) **§9.2 expands into subsections covering the universal claim rule and materialization.** §9.2.1 (Slot Alignment) carries forward the Id-and-Rel alignment rule. §9.2.2 (Cross-Role and Transport Restrictions) carries forward the prefix-matching rules. §9.2.3 (The Universal Claim Rule) is new: a four-case matrix governs how Template-declared and instance-declared values reconcile on any claim-bearing field (Rel, Key, default Resource, nested `--type`). Differing claims are conflicts; matching claims are confirmed; silent-on-one-side defers to the other. §9.2.4 (Materialization) is new: defines the build-time operation of writing Template-supplied values into the instance as explicit fields, motivated by the searchability and self-description requirements of persisted documents. Materialization is RECOMMENDED for persistence and OPTIONAL for in-memory processing.

(18) **§9.2 distinguishes named type identity from observable conformance.** Two new paragraphs in §9.2: (a) `--type` is an identity claim anchored to the Template name; two Templates with identical shapes are different types, and an instance of one is not silently an instance of the other; (b) cross-type conformance — an instance of type A structurally satisfying Template B — is a separately observable relation, useful for validation against multiple schemas, version-compatibility checks, and migration tooling. Conformance does not redefine identity. Consumers MUST NOT silently substitute one type for another even when their instances mutually conform; any cross-conformance is surfaced as a derived observation, not a rewrite of the original `--type` claim.

(19) **§1.1 reframes GDX as an abstract data model with concrete representations.** A new opening paragraph establishes that GDX is the data model — the abstract content — and that Standard Form, Compact Form, Binary Form, and programmatic in-memory form are alternative concrete representations of that model. This clarifies the boundary "what is GDX?" and positions the various forms as faithful realizations rather than separate things.

(20) **§4 (Abstract Data Model) intro reinforces the model/representation distinction.** New paragraph ties the data model defined in §4 to its concrete realizations: each preserves the same elements, prefixes, field semantics, and composition relationships; conversions transform surface syntax without changing content.

(21) **§2.1 adds Core / Extension definitional pair.** New terminology entries formally distinguish the **core** (what GDX itself defines and interprets — prefix vocabulary, element model, grammars, composition, Template-instance alignment, claim rule, primitive value-kind axioms) from **extension** (vocabulary-defined material that rides on top of the core — Rel identifiers, Type identifiers, Constraint Extension content). The pair names the **interpretation** boundary between what GDX understands and what GDX hosts. The terms *core layer* and *extension layer* are introduced as a convenient shorthand for the two sides of the boundary.

(22) **§3.7 articulates the optional-elaboration principle.** New subsection: elements are valid at their minimum form (Prefix alone); elaboration is opt-in and proceeds along two structurally distinct patterns. Identity elaboration is orthogonal across three independent axes (`Id`, `Key`, `Rel`). Value elaboration is layered across five layers (category, value, structural identity, constraint attachment, constraint semantics), where each layer is optional given the layer above and the layers live in different places (instance, Template, vocabulary). The principle organizes how to read the rest of §3 and explains why simple GDX stays simple while richer cases grow without restructuring.

(23) **§1.3 adds integration-surface, not-LCD framing.** A clause in the substrate-paragraph clarifies that GDX is the integration surface where multiple vocabularies coexist, not a lowest-common-denominator subset of them. The structural schema is opinionated and complete; what's neutral is the constraint layer.

(24) **Targeted wording sweep: core/extension terminology adopted across §2.2 and §9.4.** Replaced phrases like "the GDX layer" and "hosted on top" with "core layer" and "extension layer" where the new terminology disambiguates the interpretation boundary from the mechanism boundary. Other places — where wording was already clear — were left as-is to avoid churn.

(25) **§1 restructured.** §1.1 (What GDX Is) trimmed for redundancy: removed the duplicate prose paragraph (with `strutural`/`resouce` typos), pulled the "single repeating element shape" paragraph forward from §1.3's tail, and framed the small example explicitly as Standard Form (one realization of the abstract model). §1.2 and §1.3 swapped positions: design goals now lead, format comparison follows. The rhetorical flow shifts from "survey landscape, explain choices" to "introduce model, state goals, validate against alternatives."

(26) **§1.2 (Design Goals) rewritten with nine model-focused goals.** Replaced the previous mixed list (which combined model goals with serialization-format virtues) with nine goals that each articulate a capability GDX commits to and a corresponding gap in established formats or data models. The new goals: Typed elements baked into the form; Inline semantic alignment; Schema is data; Structural schema with pluggable constraint vocabularies; Unified document and graph model; Self-describing instances without the schema; Reflexivity through typed references; Layered validity with clear ownership; Minimum-is-enough elaboration. Serialization-format virtues (round-trip stability, encoding efficiency, canonical content addressing) are explicitly relegated to §5 as properties of Standard Form rather than goals of the data model.

(27) **§1.3 (Comparison with Existing Formats) absorbed the side-by-side example.** The composite-record side-by-side comparison block (showing the same record in GDX Standard, GDX Compact, JSON, YAML, and XML) moved from the tail of the old §1.3 to the tail of the new §1.3 (the comparison section). It now functions as a synthesizing example after each format's individual discussion rather than as an interruption of the design-goals list. The new §1.3 also gets typo cleanup (`unweildly`, `loosing`, `Write and Read`) and a sharpened opening that frames the section as validation of the §1.2 claims against the landscape.

(28) **Three cross-references updated for the §1.2/§1.3 swap.** §3.1 ("responds to the problems §1.2 named" → "§1.3 names"); §9 layered-model paragraph ("(§1.3)" → "(§1.2)"); the new §1.3 closing references "(§1.2)" for the layering/pluralism design goals. Historical changelog entries that reference §1.3 at the time of their item are left as-is; they correctly describe the change at the point it was made.

(29) **Role definition reframed as both structural and conceptual.** §2.1 and the §6 intro paragraph now describe Role as carrying both the structural shape (what fields and children are admitted) and the conceptual interpretation (how the element is to be read in relation to its parent). This makes explicit a property the spec previously relied on implicitly — that two roles may share a structural shape and still differ as roles because their conceptual interpretation differs.

(30) **§7 role descriptions sharpened with explicit conceptual leads.** Each per-role subsection now leads with a one-sentence conceptual identity for the role before describing structural mechanics. Object is "a thing with identity"; Record is "a compound property"; Property is "an attribute of its parent"; Edge is "a graph statement"; Block is "a labeled section"; Annotation is "a note attached to its parent"; Package is "a transport wrapper"; Group is "a transport partition"; Qualifier is "a refinement on its parent's value"; Extension is "a schema rule attached to a Template"; Embedded is "a structured value in a scalar's position"; Node is "a semi-structured pocket"; Set is "a multiplicity"; Template is "a type contract"; REL is "a predicate as a first-class entity"; Reference is "a typed pointer to another element". The conceptual lead resolves why structurally-similar roles (Object/Record being the load-bearing example) exist as distinct roles — the distinction lives in interpretation, not in structure.

(31) **§7.2 adds explicit Object/Record comparison.** A new paragraph at the end of §7.2 names that Records and Objects share structural shape but differ conceptually, with the consequences for addressability, persistence, and document root spelled out.

(32) **RELs admit Extension children — internal inconsistency resolved.** The §8.1 composition matrix and rule 8 already allowed Extensions on RELs (for predicate semantics like domain, range, inverse, cardinality), but §7.10, §8 modifier transparency, §8.1 prose, and §8.2 rule 7 contradicted this by saying "Extensions only under Templates." All five passages now consistently state that Extensions are valid under Templates or RELs. §7.10's conceptual lead is broadened from "a schema rule attached to a Template" to "a schema rule attached to a Template or REL." The composition matrix is unchanged (REL row already shows Extension as admitted child).

(33) **§1.2 adds "First-class predicates and vocabulary documents" design goal.** Elevates Rel to a design-level concern, framed as a single primitive — a vocabulary-defined named relationship between two roles — whose expressed form (compositional alignment, associative predicate, predicate-axiom) is parameterized by the role-combination it sits in. The structural side — the `Rel` field, the `REL` element, References — is the implementation of that primitive. Concept-as-Object is named as the decision that lets GDX have RDF/OWL's ontology-building primitive without a separate Class role.

(34) **§3.7 (new) Predicates and Vocabularies.** New subsection between §3.6 and (former) §3.7 articulating the foundational principle — a Rel is a single, constant primitive that names a relationship between two roles; the form of statement it expresses is parameterized by role-context (one rule, three readings: compositional alignment, associative predicate, predicate-axiom). The three coordinated structural mechanisms (Rel field, REL element, @REL reference) are presented as the implementation of that principle. Names the concept-as-Object decision. Defines what a vocabulary document is. Optional Elaboration moves to §3.8.

(35) **§7.15 (Relation Declaration) sharpened.** Expanded to show: the three-component vocabulary system in detail; what Extensions and References on RELs do; the concept-as-Object pattern; a small worked vocabulary document combining concept Objects, REL declarations with domain/range Extensions, predicate-to-predicate References, and Templates.

(36) **Cookbook Pattern 25 (Complete Vocabulary Document) added.** Shows a full vocabulary document with concept Objects, RELs carrying predicate-semantic Extensions, inverse References between RELs, and Templates bound to concepts. Patterns 25-34 renumbered to 26-35; TOC ranges updated.

(37) **§6.5 promoted to standalone §7 (Element Roles); subsequent sections renumbered.** The per-role catalog (formerly §6.5.1 through §6.5.16) is now §7 as a top-level section, with each role at `###` depth instead of `####`. §6 is renamed from "Element Roles" to "Prefix Vocabulary" to reflect its narrowed scope (the prefix grammar, base vocabulary, special operators, and refinement modifiers). The new §7 is the authoritative role reference. Subsequent sections shift up by one: Composition Axioms (was §7) → §8, Schemas and Validation (was §8) → §9, Transport (was §9) → §10, Data Persistence (was §10) → §11, Open Issues (was §11) → §12. All body cross-references to §6.5.x → §7.x, §7.x → §8.x, §8.x → §9.x, §9 → §10, §10.x → §11.x, §11.x → §12.x. The Table of Contents and the v0.11.0 changelog entries are updated to current section numbers; older changelog entries (v0.9.x, v0.10.0) are also updated since the sections being described still exist at the new numbers.

(38) **Extensions reframed as adjunct statements; structural restriction lifted.** §7.10 (Extension) is rewritten to define Extensions as *adjunct statements* — vocabulary-defined assertions attached to an element that consumers may interpret and act upon, but on which the element does not depend for its identity, value, or validity. This reframing replaces the prior "schema rule attached to a Template or REL" framing, which described one application of Extensions rather than what they are. The "harmlessly ignored" property (consumers that don't recognize an Extension's vocabulary ignore it) follows directly from the adjunct nature and is what allows multiple vocabularies to coexist on the same element.

Structurally, Extensions are no longer restricted to Templates and RELs. §8.1 composition matrix is updated: the Extension column shows ✓ for all parent roles that admit children, with the exceptions of Annotation (leaf-only) and Node (isolated-subtree per rule 5). §8.2 rule 7 is rewritten to state the adjunct property; rule 9 (modifier transparency) drops the Extension-restriction exception. This is an additive change for existing documents — Constraint Extensions on Templates still work exactly as before; predicate-semantic Extensions on RELs still work; what is now also permitted is provenance Extensions on Objects, presentation Extensions on Properties, audit Extensions on Edges, signing Extensions on transport wrappers, and any other vocabulary-defined adjunct statement attached to any element.

§9.4 (Constraint Extensions) retains its title and content as a normative description of one specific Extension application (schema validation on Templates). A new opening paragraph in §9.4 frames constraint validation as one use of the broader Extension mechanism.

Cookbook Pattern 20 (Adjunct Extension for Non-Schema Metadata) added, showing provenance on an Object, presentation hint on a Property, migration directive on a Template, and confidence Extension on an Edge. Pattern 19 (Annotation for Provenance) updated to clarify the Annotation-vs-Extension distinction (free-text vs vocabulary-keyed) with a pointer to Pattern 20. Patterns 20-35 renumbered to 21-36; TOC ranges updated.

(39) **Extension value-kind scope clarified; broken `_OBJ` examples fixed.** §7.10 (Extension) adds a normative paragraph stating that Extensions carry primitive value-kinds only (the closed table: `_STR`, `_INT`, `_FLOAT`, `_BOOL`, `_BYTE`). Reference-valued adjunct statements use a Reference (`@KIND`, §7.16) child with a `#rel` predicate naming the assertion. Extensions and References together cover the adjunct-statement surface — Extensions for primitive values, References for reference values — and both attach as vocabulary-keyed children of the parent.

The prior commit (38) introduced examples in §7.10, §7.15, and Cookbook Pattern 26 that incorrectly used `_OBJ` (`<_OBJ gdx:domain : @gdx:Person>`) to express reference-valued REL metadata. `_OBJ` is not in the closed Extension prefix vocabulary and would semantically mean "Extension whose Resource is an Object" — either meaningless (Objects are not a value-kind) or illegal (embedding an Object inside an Extension violates §8.2 rule 2 Object exclusion). All occurrences corrected to use References with `#rel` predicates: `<@OBJ #gdx:domain : @gdx:Person>` for Object references, `<@DEFOBJ #gdx:supersedes : @CustomerV1>` for Template references. No structural change required; the composition matrix already admits References as REL and Template children.

### v0.12.0

(40) **Embedded modifier `%` replaced by VAL role.** The Embedded refinement modifier (`%STR`, `%REC`, `%OBJ`, etc.) is removed from the prefix vocabulary and replaced by a new first-class role, **VAL** (minified **V**), defined in §7.11. Where `%X` previously expressed "an X element in property-value position," `<VAL name <X : value>>` now expresses "a slot named `name` whose value is an X element." The change is motivated by two structural needs the modifier could not express: (a) two-level identity — VAL distinguishes the slot's identity (Id/Key/Rel on the VAL itself) from the occupant's identity (Id/Key/Rel on the contained element), allowing a single position to be both vocabulary-aligned at the slot level and addressable at the occupant level; (b) closed occupant enumeration — VAL admits Properties, Records, Objects, Blocks, Sets, Edges, and References as occupants under a single uniform role, rather than requiring a separate `%`-form for each primitive.

§6.2 adds the VAL/V row. §6.4 drops the `%` modifier row and gains a paragraph pointing readers to VAL. §8.1 adds a VAL column and VAL row to the composition matrix. §8.2 rules 2 (Object exclusion), 9 (modifier transparency), 12 (modifier stacking), and 13 (Set as suffix) are updated to reference VAL rather than `%`. §8.4 drops `%X` from the value-kind family lists. §8.5 validation lists VAL rather than `%O` as the data-layer Object licensing form. §4.2 gains an "occupant of a VAL slot" carve-out paragraph mirroring the Set-member positional exemption, so that an occupant inherits the slot's named-attribute role without needing its own identifier.

Migration: every `%X` element becomes `<VAL #rel name <X : value>>`, with the slot's `#rel` predicate (and any Id/Key) moved to the VAL and the value moved into the occupant. The `VAL*` form is malformed (§7.11 axiom 6); Set-of-VAL semantics are expressed as a Set of the underlying occupant role. Examples throughout the spec, Cookbook, Path, Grammar, Compact Form, and Quick Start companions are migrated. Cookbook Pattern 10 (Embedded Structured Value) and Pattern 11 (Embedded Alternative Representations) are rewritten to demonstrate slot-level and occupant-level alignment respectively.

(41) **EDGE prefix replaced by the @-Edge family (`@OBJ`, `@REL`); §7.4 rewritten as the @-Edge Family with axioms E1–E6.** The `EDGE`/`E` prefix is removed from the closed vocabulary. Where `<EDGE #knows : @bob>` previously expressed a graph-statement element, `<@OBJ #knows : @bob>` now does. `@REL` is promoted from a leaf-like reference to a first-class @-edge whose target is a predicate (REL declaration). The two members of the @-Edge family, `@OBJ` and `@REL`, are enumerated by target role and restricted to *independent graph-entities* — Objects (including concept-Objects per §3.7's concept-as-Object pattern) and REL declarations. All other `@KIND` forms remain in the Reference family (§7.16, pending a separate refinement pass), which retains leaf-like-pointer semantics.

§7.4 is rewritten as "The @-Edge Family" and adds six normative axioms:
- **E1.** Prefix asserts target role; Resource asserts target identity; the assertion stands even when the Resource is empty (`<@OBJ #knows : >` in Template context declares an Object-edge slot).
- **E2′.** First-class but dependent: an @-edge is its own entity (carries identity, predicate, attributes) but is owned by and serialized within its source. Dies with source (relational projections SHOULD use `ON DELETE CASCADE`); dangles on target loss.
- **E3.** Attribute children describe the relationship-entity. When the edge carries identity (E6), attributes belong to the persistent edge-entity, not to any single predication.
- **E4.** Predicate (`#rel`) is MANDATORY. No unlabeled adjacency — adjacency-if-wanted is a designated `#rel` value, not its absence.
- **E5.** Family is enumerated by target role and restricted to independent graph-entities. Exactly two members (`@OBJ`, `@REL`); source role is orthogonal.
- **E6.** Identity (`Id`/`Key`) is optional and orthogonal to predicate. Identity is the edge's *substance* — the persistent entity that endures across predicate/attribute changes and disambiguates multiple edges between the same endpoints. Two @-edges with the same (source, predicate, target) ARE distinct if their identities differ; uniqueness is by identity, not by the triple. Identity-without-predicate is malformed; predicate-without-identity is the common case.

The substance/predication split is the spine: Id is what the edge *is*, `#rel` is what it currently *asserts*, attributes ride on the substance. This delivers named edges, predicate mutation over stable identity, and multi-edge disambiguation — all from E2′ with no RDF-style reification machinery.

Cascade changes:
- §3.5 (graph statements) updated to use `@OBJ` instead of `EDGE`.
- §3.6 (trio framing) re-anchored: VAL = embedding, @-Edge = graph relationships, Reference = leaf-like pointing.
- §3.7 (predicates and vocabularies) updated to describe @-edge children on RELs alongside Extensions.
- §4.3.2 (reference value-kind) and §4.3 (Resource definition) updated to use `@OBJ` examples.
- §6.2 (base vocabulary) drops the `EDGE`/`E` row; adds a graph-statement-roles subsection listing `@OBJ`/`@REL`.
- §6.3.2 (Reference Operator) rewritten to describe the @-Edge / Reference structural split.
- §7.15 (REL) updated to describe three kinds of vocabulary-metadata children (@-edges, Extensions, References) with the new framing.
- §7.16 (References) rewritten: family scope explicitly excludes `@OBJ` and `@REL`; transitional-status note added.
- §8.1 composition matrix: `Edge` column/row renamed to `@-Edge`; REL row gains `@-Edge` admittance (`✗` → `✓`) for vocabulary metadata.
- §8.2 rule 16 reframed around the @-Edge-vs-Reference split.
- §8.4 (resource rules) splits the former `EDGE` resource line into separate `@-Edge` and `Reference` lines.
- Appendix A.5 (canonical form) removes `EDGE` from the prefix list and notes the `E` slot is unassigned.
- Appendix A.7 (test suite) updated.
- §11 (Data Persistence) gains an `ON DELETE CASCADE` recommendation per E2′.

The `E` minified slot is currently unassigned; reassignment is deferred to the Reference-family refinement pass where the full minified-letter map can be reconsidered together.

Companion-document updates:
- **Cookbook Pattern 7 (Edge Between Entities)** rewritten with `@OBJ` syntax and a new example demonstrating edge-as-substance via E6 identity.
- **Cookbook Pattern 22** retitled "Typed Reference to a Non-Graph-Entity"; examples replaced with `@DEFOBJ`/`@REC` targets to match the new Reference-family scope (graph-entity references are @-edges, covered by Pattern 7).
- **Path spec** Appendix B.4 and B.6 examples migrated from Edge targets to Property targets to remain neutral with respect to the pending Reference-family refinement (cross-references to specific @-edges are a Reference-side concern).
- **Grammar / Compact Form** drop `EdgePrefix` production.
- **Quick Start** §5 retitled "Edges Between Objects" with the new syntax.
- **Format Comparison** §5 (graph data) updated; @-Edge model now positioned with named-edge / substance-without-reification as a distinguishing feature.
- **Security Considerations** §2 (reference cycles) updated to use @-edge framing.

Migration: every `<EDGE …>` element becomes `<@OBJ …>`. The `#rel` requirement is unchanged (it was already required for EDGE). Existing `@OBJ`/`@REL` references that *do* carry `#rel` continue to work; those that *don't* carry `#rel` are no longer well-formed @-edges and must either add a `#rel` (becoming proper @-edges) or migrate to the Reference family when its surface form is refined. The `E` minified prefix is unassigned and MUST NOT appear in new documents.

### v0.13.0

(42) **REF family completed; `%` sigil added; the slot-filler trichotomy made structural.** The Reference family is given its own sigil `%`, distinct from the `@` sigil that names the @-Edge family. Every `@KIND` prefix that previously served as a leaf-like reference (`@DEFOBJ`, `@REC`, `@BLK`, `@.STR`, `@_INT`, `@VAL`, value-Property forms, and their Set forms) is renamed to `%KIND`. The `@` sigil is now exclusively the @-Edge sigil, restricted to its two members (`@OBJ`, `@REL`) by §7.4 axiom E5.

The closure rule: the `%` sigil wraps any layer-1 role-prefix to produce a REF-to-that-role. The closed REF family is generated by this single composition:

- `%OBJ`, `%REC`, `%BLK` — REF to an Object, Record, Block
- `%STR`, `%INT`, `%FLOAT`, `%BOOL`, `%BYTE` — REF to a value Property
- `%.STR`, `%_INT`, `%NSTR` — REF to a Qualifier, Extension, Node-zone element
- `%DEFOBJ`, `%DEFSTR`, etc. — REF to a Template
- `%VAL` — REF to a VAL slot
- `%@OBJ`, `%@REL` — REF to an @-edge (reification)
- `%KIND*` — REF to a Set (or, in Set-member position, a Set of REFs)

The genus split is the load-bearing structural commitment: `%` always fills a slot (compositional `#rel`); `@` always asserts a relationship (associative `#rel`); the two sigils are never synonyms. The proof-by-collapse (§7.16 axiom R4): if `<%OBJ #X x : @T>` and `<@OBJ #X : @T>` could both assert "parent X T," the sigils would be interchangeable spellings and the partition would dissolve. They do not — the `%` form is "slot holds a handle, slot semantically aligned with `#X`"; the `@` form is "parent stands in relation `#X` to `T`."

§7.16 is rewritten as "The %-Reference Family" with six normative axioms:

- **R1.** Prefix asserts target role; Resource asserts target identity; resolution dereferences and verifies the role.
- **R1a.** The role-assertion is a resolution constraint, particularly pronounced for path/key resolution.
- **R2.** A REF fills a slot; it asserts nothing about its parent.
- **R3.** A REF is Property-shaped — admits `#rel` (compositional), Qualifiers, Extensions, an `Id` (slot name), and Annotations; admits no structural children.
- **R4.** `#rel` on a REF is always compositional alignment (slot-level), never associative (parent-relationship). Associative is the exclusive province of @-Edge.
- **R5.** A REF may target any addressable element by any resolution mode (name, path, key, composed cross-document) — including @-edges by key (reification via `%@OBJ`/`%@REL`).
- **R6.** A REF has no identity of its own beyond its slot; a REF is never itself a reference target.

The trichotomy and edge framing in §3.6 is rewritten around the genus split: Property fills a slot with a literal; VAL fills a slot with a dependent element; REF fills a slot with a handle to an independent element; @-Edge asserts a relationship its source participates in. The three slot-fillers share Property-shaped elaboration (compositional `#rel`, Qualifiers, Extensions); the @-Edge stands apart with associative `#rel`, attribute children, and identity.

Edge-side fall-out:

- **§7.4 axiom E5 strengthened:** `@OBJ*` and `@REL*` are declared malformed. Each @-edge is a single graph statement (per E2′); multiple edges from one source are expressed as siblings, not as Set members. Multiple-edges-as-a-collection is implicit in the document structure, not collected into a Set.
- **§8.2 derived rules 14, 14a, 15, 16 rewritten.** New rule 14 narrows the @-Edge family to two members. New rule 14a closes off `@OBJ*`/`@REL*`. Rule 15 enumerates the three malformed sigil compositions: `@@KIND`, `%%KIND`, `@%KIND`. Rule 16 distinguishes Property-shaped (REF) from statement-shaped (@-Edge) composition.

Cascade changes in Core Spec:
- §3.6 rewritten as "The Slot-Filler Trichotomy and Graph Statements"
- §4.3.2 (reference value-kind) updated for the two-family split
- §6.2 base vocabulary: REF row added; reading note updated
- §6.3 split into §6.3.2 (sigil `@`) and §6.3.3 (sigil `%`); §6 prefix-pattern intro updated to the two-layer grammar
- §7.16 rewritten as "The %-Reference Family"
- §8.1 matrix: Reference column/row renamed to REF; Set row admits REF (for Sets of REFs)
- §8.4 resource rules: REF prefix family clause added
- §A.5 (canonical form) prefix list updated
- Closed-enumeration listings in §7 intro and §1.2 reflexivity goal updated

Companion-doc cascade (to be migrated):
- Cookbook: Pattern 22 rewritten in `%` syntax; reification pattern added (`%@OBJ`); Template-version-chain examples migrated with the conceptual-shift note
- Path spec: §8.6 expanded; Appendix B examples migrated to `%KIND`
- Grammar / Compact Form: add `%` sigil production; document layer-2 wrapping
- Quick Start, Security, Format Comparison: incidental cleanup
- All companions bumped to v0.13.0 (or next minor)

Migration: every leaf-like `@KIND` reference (no `#rel`, no attribute children) becomes `%KIND` mechanically. `@DEFOBJ`, `@REC`, `@BLK`, `@.STR`, `@_INT`, `@VAL`, `@KIND*` for non-graph-entity targets all migrate. The `@OBJ` and `@REL` forms WITH `#rel` and attribute children continue to work as @-edges with no syntactic change; the conceptual reading shifts (under v0.12, leaf-like vocabulary-metadata links; under v0.13, first-class associative graph statements per E4). Documents using `@OBJ #conformsTo : @Template` patterns where `#conformsTo` is *compositional* (slot-aligning, not relating) migrate to `%DEFOBJ #conformsTo : @Template`; the change is semantically meaningful and documents will need review on a case-by-case basis. The `@OBJ*` and `@REL*` Set forms (rare in practice) are no longer well-formed; multiple edges become siblings.

(43) **§7.15 REL composition widened; "defined by purpose, not by structure" principle made explicit.** A REL is defined by its purpose — naming a first-class predicate — and that purpose is fully discharged by its identifier. The prior matrix held RELs to a stricter standard than any other role ("every child must be semantically load-bearing"); the new matrix admits the full data-carrier set, with structurally-justified exclusions only.

The REL composition row widens from `@-Edge, REF, Extension, Annotation` to add `Property, Record, Block, Set, VAL`. Authors may now attach labels (`<STR rdfs:label : "knows">`), descriptions (`<STR rdfs:comment : "...">`), version markers, authorship, alias Sets, documentation Records, or any other data carrier directly to a REL — the same kinds of children that stand on any Object. Properties on a REL are data carriers whose meaning is consumer-conferred, never by GDX-core; meaninglessness is never disqualifying. RELs are held to no stricter standard than any other element.

The structurally-justified exclusions remain:

- **Object** — vocabulary entities don't contain data-instance Objects; a contextually false claim about what the REL is.
- **Node** — RELs are structured vocabulary entities, not semi-structured pockets.
- **Qualifier** — RELs carry no Resource, so there is nothing to refine.
- **REL (self)** and **Template (`DEF…`) forms** — vocabulary entities don't nest within RELs. The Template exclusion is an explicit carve-out from modifier transparency (§8.2 rule 9(b)): even though `DEFREC` would otherwise be admitted as a Record-modifier (and REL now admits Record), the `DEF…` forms are blocked because they declare another vocabulary entity, and one vocabulary entity does not nest inside another.
- **Transport-layer roles (Package, Group)** — transport wrappers do not appear inside vocabulary declarations.

**Property-vs-Extension guidance on RELs.** Both are admitted; the choice communicates intent to consumers:

- Use **Property** for data about the predicate (labels, descriptions, version, authorship) — meaning conferred by the reading consumer (documentation tool, vocabulary browser, lookup index).
- Use **Extension** for a vocabulary-defined semantic axiom (symmetry, transitivity, cardinality bounds) — meaning conferred by the validator or reasoner that recognizes the vocabulary identifier.

Mechanical rule: would a validator/reasoner act on it? → Extension. Is it for humans or downstream lookup? → Property.

**@-edge target convention on RELs.** The @-Edge family axioms (§7.4 E5) restrict @-edge targets to Objects or RELs structurally. The vocabulary convention for @-edges on a REL is to target concept-Objects or other RELs (predicate-axiom statements like domain/range/inverse). This is convention, not structural enforcement; the matrix admits any @OBJ or @REL target, and the concept-vs-data-instance distinction lives in vocabulary practice.

Cascade in Core Spec:
- §7.15 rewritten with the "purpose, not structure" framing and the full admitted/excluded enumeration
- §8.1 matrix REL row: Property, Record, Block, Set, VAL added (✓)
- §8.2 rule 9 updated with the REL-doesn't-admit-Templates carve-out as new clause 9(b); existing Node-zone isolation becomes clause 9(a)
- Small vocabulary example in §7.15 updated to show Property usage alongside @-edges and Extensions

Migration: existing REL declarations continue to work; the change is additive. New patterns become available — labels, descriptions, alias Sets, documentation Records can sit on a REL directly without Extension wrapping. Existing uses of `<_STR rdfs:label : "...">` (Extension form) on RELs remain syntactically valid, but `rdfs:label` is data about the predicate rather than a semantic axiom — the cleaner expression is `<STR rdfs:label : "...">` (Property form). Authors should review their `_STR`/`_INT` uses on RELs and reclassify by the Property-vs-Extension guidance above when convenient.

---

### v0.14.0

This revision is an editorial / hygiene pass — no change to the data model, prefix vocabulary, composition rules, or any conformance requirement. It resolves register leaks (informative content carrying normative weight, and normative content mislabeled as informative), de-duplicates content that was stated authoritatively in two places, and corrects a stale token. It is the first in a planned sequence of passes preparing the specification for an eventual split into a lean normative spine and a separate non-normative Primer; the changes here pre-cut extraction seams without moving content out of this document.

(44) **Register-hygiene and de-duplication pass.** Ten in-place corrections, each a correctness fix rather than a packaging change:

- **§11 (Data Persistence) made honestly informative.** §11 declares itself informative, but §11.4 (Persistence Round-Trip) carried its own MUST / MAY / MUST-NOT preservation lists — normative content inside a non-normative section — which also duplicated the serialization-conformance contract of Appendix A.2. §11.4 is rewritten to defer to A.2 as authoritative and to keep only persistence-specific informative framing (how the round-trip passes *through storage*). §11.1's normative-reading "a wire format is *conforming* if…" clause is softened to defer to §4 and Appendix A. The §11 introduction now states explicitly that the section originates no normative requirement of its own. No requirement is lost: the round-trip law lives in A.2, where it already lived.

- **Conformance-layer model: single authoritative home.** The Layer 1 / 2a / 2b model was stated in full in both §2.2 (under "Conventions and Terminology") and Appendix A.1, with A.1 even cross-referencing §2.2 as its source while restating the whole model — a duplication with a circular deferral. A.1 is now the single authoritative normative statement; §2.2 is reduced to a conceptual introduction that defers to A.1 and retains the *decision rationale* (the core/extension ownership split, and the rejected monolithic-verdict alternative). The normative reporting requirement formerly in §2.2 ("all four verdicts MUST be reported and handled distinctly") is moved into A.1 and strengthened ("MUST NOT collapse them into a single valid/invalid answer"). This also removes normative content from the terminology section, where it did not belong.

- **Root rule de-duplicated.** The "Document" term entry in §2.1 stated the root rule as a MUST, duplicating §8.2 rule 1. It now defers to §8.2 as authoritative and states the root set only informally. (True normative duplication — two copies of one rule that could drift into a conformance ambiguity — is the case that warrants hard de-duplication.)

- **Slot-filler proof-by-collapse de-duplicated.** §3.6 restated the full proof-by-collapse argument that §7.16 axiom R4 owns authoritatively. §3.6 now gives the intuition and the contrasting readings of `<%OBJ #X x : @T>` versus `<@OBJ #X : @T>`, then points to §7.16 R4 for the argument itself, which is no longer reproduced. The unique conceptual framing in §3.6 (the spine, the genus split, the construct/question table, the question-matching guidance) is retained.

- **§5.7 (Use-Case Adaptability) trimmed.** The "look how it scales" worked simple-to-rich example was informative teaching at the tail of the normative §5. The point (the form scales without changing parser; features are opt-in) is kept in one sentence; the worked progression is noted as belonging to the introductory material and the Cookbook rather than reproduced in the normative text.

- **§7.7 (Package) clarity.** The phrase "Packages are not persisted as such; receivers unpack them" reads, to a careful modeler, as a discouragement from authoring Packages at all. A clarifying passage now states that "not persisted as such" describes a *receiver's* unpack behavior, not a reason to avoid the construct, and gives a rule of thumb: root in Object(s) for data *at rest*, root in a Package for data *in motion*. (This correction was prompted by a real modeling error made while authoring GDX-formatted tooling against the spec.)

- **§9.1 stale token corrected.** The prose described a Template as "an element with a `TD…` prefix"; the prefix is `DEF…` (per §7.14). Corrected.

No migration is required. Every change either removes a redundant statement in favor of an existing authoritative one, relocates a normative clause's authority without changing its content, or rewords informative prose. Documents and implementations conforming to v0.13.0 remain conforming to v0.14.0.

### v0.15.0

Additive editorial pass: four design-decision records (ADRs) written inline at the section each decision governs. No change to the data model, vocabulary, composition rules, or conformance. ADRs are kept inline (rather than collected into a consolidated section) so each decision sits beside the role or rule it justifies; the uniform inline shape is **Decision / Alternative considered / Rejected because**, and an ADR is written only where a real alternative was rejected for a concrete reason.

(45) **Four inline ADRs added for decisions whose rationale was previously undocumented or only present in teaching prose:**

- **§5.4.1 — ASCII identifier start.** Records the human-factors rationale (identifiers stay greppable and keyboard-reachable) behind `IdStartChar = ALPHA | "_"`, which the spec previously justified only on numeric-ambiguity and path-syntax grounds.
- **§6 — closed prefix vocabulary.** Records why the prefix set is closed (structural validity must not depend on loading a vocabulary; an open set would collapse the Layer 1 / Layer 2b boundary), against the rejected open/extensible-prefix alternative.
- **§4.1 — the identity/value split.** Records the data-model basis for the split (self-describing elements; the typed-elements and inline-alignment goals) against the rejected JSON/XML/RDF alternatives. Placed in §4.1 (the authoritative field model) rather than §3.3 (which teaches the concept).
- **§9.4 — core records but does not interpret constraints.** Records why the core holds no constraint opinion even for primitives (a built-in `min` would compete with vocabulary constraints and break parallel-vocabulary coexistence), against the rejected core-interprets-common-constraints alternative.

The five ADRs already present in the role catalog (§7.4 substance/predication over RDF reification; §7.11 VAL over the embedded modifier; §7.15 concept-as-Object over a Class role; §7.16 the genus split / proof-by-collapse; §9.6 flat Templates over inheritance) are unchanged; this entry only adds the four that lacked a home. No migration required.

### v0.16.0

**The specification splits into a normative spine and a non-normative Primer.** This revision extracts the teaching and motivation content into a separate companion document, the **GDX Primer**, leaving this document as the lean normative specification. The split is structural, not semantic: no data-model, vocabulary, composition, or conformance requirement changes, and documents/implementations conforming to v0.15.0 remain conforming. This is the long-planned spine/primer separation prepared by the v0.14.0 register-hygiene pass and the v0.15.0 ADR pass.

(46) **Extraction of the Primer.**

- **Moved to the GDX Primer (companion, non-normative):** the former §1.1 (What GDX Is, full version), §1.2 (Design Goals), §1.3 (Comparison with Existing Formats), and all of §3 (GDX Concepts) in its entirety. The Primer is the document a reader new to GDX should start with; it teaches the model by example and states the rationale, citing the specification sections that define each concept normatively.

- **§1 (Introduction) made spine-appropriate.** §1.1 is rewritten as a compact "what GDX is" orientation that retains the small worked example, states that this document is the normative definition, and routes the learner/evaluator to the Primer. §1.4 (Where GDX Fits) and §1.5 (Scope) are retained. The design-goals enumeration and the five-format comparison now live in the Primer.

- **§3 retained as a relocation stub.** The §3 slot is kept (numbering is unchanged to preserve cross-references in this document and its companions) and now contains a short notice that the conceptual development is in the Primer, with pointers to the normative homes of those concepts (§4 data model, §5 text form, §6 vocabulary, §7 roles/axioms, §8 composition).

- **Spine cross-references re-pointed to spine-internal homes.** Every reference that previously cited §3.x for a concept the spine depends on now cites the authoritative spine section instead: genus-split / proof-by-collapse citations point to §7.16 axiom R4; predicate-model citations point to §7.4 and §7.15; the Key-anchor citations point to §5.4.5; the identity/value-split mentions in the §4.1 ADR point to the Primer for the concept and to §4.1 itself for the authoritative basis. The specification body now contains no cross-reference *into* the Primer for any normative dependency — the spine stands alone for an implementer.

- **Section numbering preserved.** Rather than renumber (which would invalidate section citations in this document and in the Path, Cookbook, Format Comparison, and other companions), the original numbers are kept, with §3 occupied by the relocation stub. The TOC marks §3 as moved.

Migration: none for data or implementations. Readers who used §1.1–1.3 or §3 for learning should now use the GDX Primer; readers using the normative sections are unaffected. Companion documents that cite §3.x of the specification for *conceptual* material should re-point to the Primer; companions citing the normative sections are unaffected.

---
