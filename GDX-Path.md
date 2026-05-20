# GDX Path Expressions — Specification

- **Status:** Draft v0.1 (working draft)
- **Date:** 2026-05-20
- **Companion to:** GDX Core Specification v0.11.0
- **Authors:** Julian Mussi


## Abstract

This document defines the GDX Path Expression — a syntax and resolution model for addressing individual elements within a GDX document by their structural location. Path expressions complement the identifier-mode reference syntax (`@Identifier`) and the key-mode reference syntax (`&Key`) defined in the GDX Core Specification by providing access to elements that lack a globally addressable identifier, including Records, Blocks, value Properties, Qualifiers, and Extensions.

The path grammar is uniform across all consuming contexts. This specification defines the syntax and per-segment selection semantics; the cardinality and failure semantics for any given consumer (references, delta operations, queries) are specified by the consuming layer. This document normatively defines address-mode behavior — the cardinality contract used by `@KIND` reference elements (Core Specification §6.5.16). Query-mode and delta-mode behaviors are reserved for future companion specifications.

This specification resolves the `$PathExpression` reservation in Core Specification §11.4 (Path-Mode Reference Resolution). It defines the five reference-literal forms recognized in `@KIND` reference resources: `IdRef` (`@Identifier`), `PathRef` (`$PathExpression`), `KeyRef` (`&Key`), `IdPathRef` (`@Identifier$PathExpression`), and `IdKeyRef` (`@Identifier&Key`). The composed forms `IdPathRef` and `IdKeyRef` express cross-document addressing by structural path or by Key-based anchor respectively.

This document does not address query-mode semantics (wildcards, filters, descendant axis, projections), delta/patch operations, or filter expression languages. The syntactic territory those features require is enumerated as reserved (§9) so that v1 path expressions remain forward-compatible.


## Table of Contents

1. [Notation](#1-notation)
2. [Scope and Modes](#2-scope-and-modes)
3. [Path Expression Syntax](#3-path-expression-syntax)
4. [Segment Kinds](#4-segment-kinds)
5. [Composition with Identifier and Key References](#5-composition-with-identifier-and-key-references)
6. [Resolution Algorithm](#6-resolution-algorithm)
7. [Address-Mode Singularity](#7-address-mode-singularity)
8. [Required Changes to GDX Core Specification](#8-required-changes-to-gdx-core-specification)
9. [Reserved Syntax for Future Revisions](#9-reserved-syntax-for-future-revisions)
10. [Open Items](#10-open-items)
11. [Appendix A — Complete Grammar](#appendix-a--complete-grammar)
12. [Appendix B — Worked Examples](#appendix-b--worked-examples)
13. [Appendix C — Migration Notes](#appendix-c--migration-notes)


---

## 1. Notation

This document uses Extended Backus-Naur Form with the conventions established in the GDX Standard Form — Grammar Specification §1:

| Symbol  | Meaning                                       |
|---------|-----------------------------------------------|
| `::=`   | Production definition                         |
| `\|`    | Alternation                                   |
| `*`     | Zero or more repetitions                      |
| `+`     | One or more repetitions                       |
| `?`     | Optional (zero or one occurrence)             |
| `(...)` | Grouping                                      |
| `"..."` | Literal terminal (exact characters)           |
| `[a-z]` | Character class (single character in range)   |

Productions are case-sensitive. Whitespace within a `PathExpression` is restricted normatively (§3.4); productions in this document do not show whitespace explicitly.

Cross-references of the form "Core Spec §X.Y" refer to the GDX Core Specification. Cross-references without prefix refer to sections of this document.

The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted as described in RFC 2119.


---

## 2. Scope and Modes

A *path expression* is a textual selector that, when evaluated against a defined starting element, yields a (possibly empty) set of elements drawn from the GDX document tree. The same grammar serves multiple consuming contexts, each of which imposes its own cardinality requirement on the result.

| Mode     | Consumer                                              | Cardinality required | Failure semantics                |
|----------|-------------------------------------------------------|----------------------|----------------------------------|
| Address  | `@KIND` reference elements; future delta operations   | Exactly one          | Unresolved reference / op fails  |
| Query    | Future query / projection contexts                    | Any (0..many)        | Empty result is valid            |
| Test     | Future existence-test operations                      | Any (0..1 as boolean)| Path absence is a defined outcome|

The grammar defined in this specification is identical across all modes. Cardinality enforcement is a property of the consuming specification, not of the path expression itself. Address mode is normatively defined here (§7); query and test modes are reserved for future revisions.

A v1 path expression — using only the segment kinds and separators defined in §3 and §4 — is forward-compatible with any future query-mode or delta-mode extension. The syntactic slots reserved for those future modes (§9) do not appear in v1 paths.


---

## 3. Path Expression Syntax

### 3.1 Overall Form

A `PathExpression` is introduced by the `$` sigil in textual contexts that accept a reference literal (Core Spec §5.2.6). The path begins with a `/` segment separator and consists of one or more path segments, each separated by `/`:

```ebnf
PathExpression  ::= "/" Segment ("/" Segment)*
```

The leading `/` is REQUIRED; bare `$` followed by a segment without an intervening `/` is malformed. This requirement reserves the `$./` and `$../` forms for a future relative-path extension (§9).

A `PathExpression` MUST contain at least one segment. The following are malformed:

- `$` alone (no path body).
- `$/` (leading separator with no segment).
- `$/foo/` (trailing separator).
- `$/foo//bar` (empty segment between separators).

The leading slash of the `PathExpression` is the path-mode marker, not a segment. Subsequent slashes are segment separators.

### 3.2 Segments

Each segment is exactly one of three selector forms:

```ebnf
Segment         ::= IdSegment | RelSegment | OrdinalSegment
```

Segment kind is determined by the segment's first character:

| First character    | Segment kind     |
|--------------------|------------------|
| `#`                | `RelSegment`     |
| `[`                | `OrdinalSegment` |
| otherwise (letter or `_`) | `IdSegment` |

A v1 segment carries exactly one selector. Compound segments — those combining more than one selector form within a single step (for example, an Id combined with a Rel constraint) — are reserved for a future revision (§9).

Key-based addressing does not appear as a path segment. The `Key` field is a document-anchor mechanism (Core Spec §3.2, §5.4.3) and is addressed via the `&Key` reference literal forms (§5.3, §5.4) rather than by path traversal. A path expression that walks structurally to a Keyed element is permissible (matching the element by Id, Rel, or position) but inefficient compared to the direct Key reference; the path syntax intentionally does not provide a Key-matching segment kind.

### 3.3 Lexical Alphabet

The character set within an `IdSegment` and a `RelSegment` is governed by the `SegmentName` production (§4.1). The `SegContChar` alphabet is a narrowing of the Core Spec §5.4.1 `IdContChar` alphabet: within a path segment, the `/` character is **not** a valid continuation character. This narrowing applies only inside a path expression; in other identifier-bearing positions, identifiers continue to admit `/` as a continuation character so that IRIs remain valid identifiers.

A `PathExpression` terminates when the lexer encounters a character outside the segment-internal alphabet. In practice, this is typically a whitespace character, a `>` closing-bracket, or another sigil character.

### 3.4 Whitespace Rules

No whitespace is permitted anywhere within a `PathExpression`. The path is a single contiguous token bounded by:

- The `$` introducer (or the `$` following the `@Identifier` portion of a composed reference, §5) on the left.
- The first character outside the segment-internal alphabet on the right.

The whitespace tightness rule for `$` and `@` (no whitespace between sigil and content) is established by Core Spec §5.1.1.


---

## 4. Segment Kinds

### 4.1 IdSegment

```ebnf
IdSegment       ::= SegmentName

SegmentName     ::= SegStartChar SegContChar*
SegStartChar    ::= ALPHA | "_"
SegContChar     ::= ALPHA | DIGIT | "_" | "-" | "." | ":"

ALPHA           ::= "A".."Z" | "a".."z"
DIGIT           ::= "0".."9"
```

An `IdSegment` is a bareword. It matches a child element whose `Id` field is string-equal to the segment text, using the case-sensitive ordinal comparison defined in Core Spec §4.2.

Examples:

- `addr` matches a child with `Id` `addr`.
- `gdx:Person` matches a child with `Id` `gdx:Person` (a CURIE; `:` is in `SegContChar`).
- `com.acme.Address` matches a child with that dotted Id.

The `SegContChar` alphabet excludes `/` (cf. Core Spec §5.4.1 `IdContChar`). To address an element whose `Id` itself contains `/`, give the element a Key (Core Spec §3.2, §5.4.3) and reference it via the `&Key` reference literal (§5.3), or address the element directly via an identifier-mode reference (`@Identifier`).

### 4.2 RelSegment

```ebnf
RelSegment      ::= "#" SegmentName
```

A `RelSegment` matches a child element whose `Rel` field is string-equal to `SegmentName`. CURIEs and other identifier forms admitted by `SegmentName` are valid as the Rel value.

Examples:

- `#gdx:knows` matches a child with `Rel` `gdx:knows`.
- `#schema:givenName` matches a child with that Rel.

The Rel field is not constrained to be unique among the children of an element. Multiple Properties, Edges, or other elements MAY share the same Rel value (Core Spec §6.5.4 explicitly allows multiple Edges of the same Rel between the same source/target pair). A `RelSegment` therefore may match zero, one, or many child elements at the data layer.

Address-mode consumers (§7) require exactly one match. Authors writing paths in address-mode contexts SHOULD select segments such that the terminal step matches a single child; this typically means using an `IdSegment` or `OrdinalSegment` at any step where multiple siblings could share a Rel. Alternatively, when the target element carries a `Key`, the `&Key` reference literal (§5.3) addresses it directly without requiring path traversal at all.

### 4.3 OrdinalSegment

```ebnf
OrdinalSegment  ::= "[" Ordinal "]"
Ordinal         ::= NONZERO_DIGIT DIGIT*
NONZERO_DIGIT   ::= "1".."9"
```

An `OrdinalSegment` matches the *N*-th child of the parent in document order, where *N* is the bracketed numeral. Ordinals are **1-based**: `[1]` is the first child by document order.

The following forms are malformed:

- `[0]` — no zero ordinal.
- `[01]`, `[007]` — no leading-zero ordinals.
- `[-1]` — no negative ordinals (the negative form is reserved for a future "last-N" extension).
- `[]` — empty brackets.
- `[ 3 ]`, `[3 ]` — no whitespace inside brackets.

The ordinal counts all child elements of the parent in document order, regardless of role or modifier. It is a positional selector based purely on document order; it does not consult the `Key` field or Set-membership ordering.

The contents of `[ ]` are restricted to a base-10 numeral in v1. Wildcards (`[*]`), ranges (`[1:5]`, `[1..3]`), filter expressions (`[?(…)]`), unions (`[1,3]`), and named subscripts (`[name]`) are reserved (§9).

### 4.4 Selector Semantics Summary

| Selector              | Matches against         | Uniqueness guarantee at data layer                                  |
|-----------------------|-------------------------|---------------------------------------------------------------------|
| `IdSegment` (bareword)| `Id` field              | Unique within parent scope (Core Spec §4.2: collisions are malformed) |
| `RelSegment` (`#R`)   | `Rel` field             | Not unique — multiple siblings may share a Rel                       |
| `OrdinalSegment` (`[N]`) | Position in document order | Unique by construction                                          |

The uniqueness column indicates the data-layer guarantee per Core Specification. The grammar does not require an `IdSegment` to match exactly one child if the data is malformed; resolution behavior in the presence of duplicate Ids is implementation-defined. Conforming documents do not produce such collisions (Core Spec §4.2).

Key-based addressing — matching an element by its `Key` field — is not a path segment kind in v1. Keys are addressed directly via the `&Key` and `@Identifier&Key` reference literal forms (§5.3, §5.4). The rationale is that Keys provide location-stable anchors (Core Spec §3.2); folding Key matching into path segments would couple Keys to the very location they are designed to be independent of. A path expression that matches by Id, Rel, or ordinal at each step composes naturally with a Key reference at the start; a path that uses Key matching mid-way is redundant with the direct reference form.


---

## 5. Composition with Identifier and Key References

The `$PathExpression` syntax composes with the identifier-mode reference (`@Identifier`, Core Spec §5.2.6) and the key-mode reference (`&Key`, Core Spec §5.2.6) to express references that combine name resolution, structural traversal, and key-based anchoring. Five reference-literal forms are recognized:

```ebnf
ReferenceLiteral  ::= IdRef | PathRef | KeyRef | IdPathRef | IdKeyRef

IdRef             ::= "@" Identifier
PathRef           ::= "$" PathExpression
KeyRef            ::= "&" KeyToken
IdPathRef         ::= "@" Identifier "$" PathExpression
IdKeyRef          ::= "@" Identifier "&" KeyToken

KeyToken          ::= SegmentName | DIGIT+
```

### 5.1 Form Semantics

- **`IdRef`** — name-mode resolution alone. The `@Identifier` is resolved per the scope walk of Core Spec §6.5.16 (local → namespace → IRI). The reference designates the resolved element directly.
- **`PathRef`** — structural traversal alone, rooted at the root of the document containing the reference literal. The reference designates the element reached by following the `PathExpression` from the document root.
- **`KeyRef`** — key-mode lookup alone. The `&Key` form looks up an element by its `Key` field within the scope of the reference (the enclosing Object, or the document if at transport-layer position). See Core Spec §5.4.5 for Key scoping rules. The reference designates the element with the matching Key.
- **`IdPathRef`** — composed name + path. The `@Identifier` portion resolves by name-mode to a starting element; the `$PathExpression` portion is then evaluated rooted at that starting element. The reference designates the element reached at the end of the traversal.
- **`IdKeyRef`** — composed name + key. The `@Identifier` portion resolves by name-mode to a starting Object; the `&Key` portion is then looked up within that Object's scope. The reference designates the Keyed element within the resolved Object's subtree. This is the canonical form for cross-document Key-based addressing.

Each sigil retains its independent meaning under composition. The `@` operator performs name resolution; the `$` operator performs structural traversal; the `&` operator performs key lookup. Composition reads left-to-right.

### 5.2 Whitespace and Boundaries

No whitespace is permitted between `@`, `Identifier`, `$`, `&`, the path segments, and the Key token. The entire `ReferenceLiteral` is one contiguous token.

In an `IdPathRef`, the `Identifier` following `@` is consumed greedily per Core Spec §5.4.1 until a character outside `IdContChar` is encountered. Because `$` is not a member of `IdContChar`, the boundary between the identifier portion and the path portion is unambiguous; no escaping is required.

In an `IdKeyRef`, the `Identifier` following `@` is consumed up to the first `&` for the same reason; `&` is not a member of `IdContChar`, so the boundary is lexically clean.

The `$` and `&` operators do not combine within a single reference literal. The form `@Identifier$PathExpression&Key` is malformed; pick one of `IdPathRef` or `IdKeyRef`.

### 5.3 Cross-Document Addressing

`IdPathRef` and `IdKeyRef` are the canonical forms for cross-document addressing. The `@Identifier` portion may resolve to an element in any scope:

- Local-document (the identifier matches an Object Id within the same document).
- Namespace-relative (the identifier matches a name registered in a vocabulary or package scope).
- Remote IRI (the identifier is a fully qualified IRI; implementations MAY perform network resolution per Core Spec §6.5.16 rule 3).

The path or key portion is then evaluated within the resolved Object's scope:

```
@gdx:bob$/contact/address              // path within a named Object
@gdx:bob&shippingAddress               // Key lookup within a named Object
@https://acme.com/people/alice$/photo  // IRI-resolved + intra-target path
@https://example.com/audit-log&entry-12345  // IRI-resolved + Key lookup
```

A `PathRef` or `KeyRef` with no `@Identifier` portion is rooted at (or scoped within) the current document. These forms cannot address elements in other documents directly.

### 5.4 Empty Composition

The forms `@Identifier$` (trailing `$` with no path segments), `@Identifier&` (trailing `&` with no key token), `$` alone, and `&` alone are all malformed. If the resolved element itself is the desired target, omit the trailing operator and use the plain `@Identifier` form.

### 5.5 Reference-Element Composition

A `@KIND` reference element (Core Spec §6.5.16) MAY use any of the five reference literal forms as its Resource. The choice of form is governed by what the reference is addressing and how the target is most stably identified:

| Use                                                                                            | Form          |
|------------------------------------------------------------------------------------------------|---------------|
| Reference to an identifiable Object                                                            | `IdRef`       |
| Reference to a non-Id-bearing element in this document by structural location                  | `PathRef`     |
| Reference to a Keyed element anywhere in this document                                         | `KeyRef`      |
| Reference to a non-Id-bearing element in another document by structural location                | `IdPathRef`   |
| Reference to a Keyed element in another document, or to a Keyed element scoped within a named Object | `IdKeyRef` |

When both a Key and a structural path could address the target, prefer the Key form. Keys are location-stable; paths break when intervening structure changes.


---

## 6. Resolution Algorithm

Path resolution is a left-to-right traversal of the segments of a `PathExpression`, starting from a defined starting element. The starting element is determined by the consuming context:

- For `PathRef` (Core Spec §5.2.6, as revised): the root of the document containing the reference literal. For documents with multiple top-level elements (Core Spec §7.2 rule 1), the document root is a notional container of the top-level elements; the first segment matches against the top-level elements as children of that container.
- For `IdPathRef`: the element produced by name-mode resolution of the `@Identifier` portion. If that resolution fails (no element matches the identifier in any scope), the path is not evaluated and the entire reference is unresolved.
- For other consumers (delta operations, queries — future): the starting element is defined by the consuming specification.

### 6.1 Per-Segment Matching

For each segment in order, the resolver computes the **match set** — the set of child elements of the current element that satisfy the segment selector:

| Segment kind             | Matches                                                                                       |
|--------------------------|-----------------------------------------------------------------------------------------------|
| `IdSegment(name)`        | Each child whose `Id` field is string-equal to `name` (case-sensitive ordinal comparison).   |
| `RelSegment(rel)`        | Each child whose `Rel` field is string-equal to `rel`.                                       |
| `OrdinalSegment(n)`      | The *n*-th child by document order (1-based). Match set is empty if the parent has fewer than *n* children. |

A segment match operates only on the **direct children** of the current element. Path expressions in v1 provide no descendant-axis operator; descendant traversal is reserved (§9).

The match operates on all children regardless of their role or modifier: a segment may match a Property, a Record, a Block, an Edge, a Set, a Qualifier, an Extension, a Node-zone element, an Object (in transport-layer contexts), or any other element type. The role of the matched element is checked against the consumer's role requirement (§7 for address mode); the segment match itself is role-agnostic.

### 6.2 Match Set Cardinality

After each segment, the match set may have any cardinality:

- **Zero elements.** No child of the current element satisfies the selector. Resolution halts at this segment; the path resolves to no element.
- **One element.** The single matched element becomes the current element for the next segment, or is the resolution result if no further segments remain.
- **Multiple elements.** A `RelSegment` may produce a multi-element match set when multiple siblings share the same Rel. (In v1, only `RelSegment` can produce multi-match; in future query-mode extensions, wildcards and filters will introduce additional multi-match possibilities.)

In multi-match cases, the resolver does **not** select among the matches. The match set is passed to the consuming context; the cardinality contract of that context (§7 for address mode) determines whether the multi-match is acceptable.

### 6.3 Resolution Outcome

The path expression's outcome is one of:

- **Resolved to one element.** The path evaluation terminated successfully with a single matched element.
- **Resolved to multiple elements.** Match set is greater than one at some step.
- **Unresolved.** Match set is empty at some step.

The consuming specification interprets the outcome according to its cardinality contract. This specification defines address-mode behavior in §7; other modes are reserved.


---

## 7. Address-Mode Singularity

This section is **normative** for the use of path expressions as reference literals in `@KIND` reference elements (Core Spec §6.5.16) and is anticipated to be referenced normatively by future delta-operation specifications.

A `PathRef` or `IdPathRef` appearing as the Resource of a `@KIND` reference element is **resolved** if and only if path evaluation yields exactly one element whose role matches the enclosing `@KIND` prefix.

Path evaluation yielding any of the following MUST be treated as an unresolved reference:

1. **Zero elements.** No element matched the path at some segment.
2. **Multiple elements.** Match set cardinality is greater than one at the terminal segment.
3. **One element of non-matching role.** The terminal element exists, but its role does not match the enclosing `@KIND`. For example, `<@REC x : $/orders/123>` where the path resolves to an Object is unresolved, because the prefix declares the target as a Record.

A reference may additionally be unresolved by the `@Identifier` portion of an `IdPathRef`: if name-mode resolution fails (no scope yields a target for the identifier), the path is not evaluated and the entire reference is unresolved.

### 7.1 No Implicit First-Match Selection

In address mode, a resolver MUST NOT silently select a first match (or any other deterministic choice) when the terminal match set contains more than one element. The "MUST NOT" rule forecloses silent ambiguity: an author whose path matches more than one element receives an unresolved reference, not a non-deterministic or implementation-dependent selection.

### 7.2 Author Responsibility for Specificity

The grammar admits paths that may resolve ambiguously against some data shapes (typically paths ending in a `RelSegment` where multiple siblings share that Rel). Authors writing paths in address-mode contexts are responsible for making paths specific enough to resolve to exactly one element of the expected role.

When a Rel-bearing terminal step may have multiple matching siblings, authors SHOULD use a more specific terminal selector:

- An `IdSegment` if the target carries an Id.
- An `OrdinalSegment` if the target is identified by its position among children.

Alternatively, when the target element carries a `Key`, switch from a `PathRef`/`IdPathRef` to a `KeyRef`/`IdKeyRef` (§5.3, §5.4). Key-based addressing avoids path traversal entirely and is location-stable.

### 7.3 Resolution Determinism

For a conforming document, conforming resolvers MUST produce the same resolution outcome (resolved-to-element vs. unresolved) for the same path expression evaluated against the same starting element. Implementation-defined behavior is permitted only for the network-resolution component of identifier-mode resolution (Core Spec §6.5.16 rule 3); the path-traversal component is fully determined by the document.


---

## 8. Required Changes to GDX Core Specification

The introduction of path-mode reference resolution requires the following modifications to the GDX Core Specification. These changes are normative as of Core Specification v0.11.0, which incorporates path-mode in lockstep with the publication of this companion specification.

### 8.1 Identifier Grammar (Core Spec §5.4.1)

The `IdStartChar` production narrows by removing `/`:

```ebnf
IdStartChar     ::= ALPHA | "_"
```

`IdContChar` is **unchanged**; `/` remains a valid continuation character so that IRIs (e.g., `https://acme.com/people/alice`) remain valid identifiers in identifier-bearing positions outside path expressions.

This narrowing makes the `$/...` form of a path expression unambiguous: the `$` introduces path mode, and the immediately-following `/` is the segment separator (since no identifier may begin with `/`). It also makes the boundary between the `@Identifier` and `$PathExpression` halves of an `IdPathRef` lexically clean: an identifier following `@` ends at the first `$`.

Migration impact is documented in Appendix C.

### 8.2 Identifier Forms Table (Core Spec §5.4.2)

The "Path" row of the conventional identifier-forms table is removed. Path-style local naming (e.g., `<OBJ /people/alice>`) is replaced by path-mode reference resolution. The replacement form depends on the use case:

- For local naming, identifiers may continue to embed `/` as a continuation character; only the leading `/` is invalid.
- For references that were path-like in spirit, use a `PathRef` or `IdPathRef`.

### 8.3 Key Field (Core Spec §3.2, §5.4.3)

The `Key` field is reframed from a Template-slot alignment mechanism to a document-anchor mechanism:

- The §3.2 description of `Key` is replaced. Keys are an opt-in named-anchor mechanism providing location-stable reference targets for elements that lack global identifiers. Keys play no role in Template-instance alignment.
- A new normative subsection (Core Spec §5.4.5) defines Key uniqueness: Keys MUST be unique within the enclosing Object's scope. Elements at transport-layer position (direct children of a Package, Group, or document root) MUST have Keys unique within the document.
- The numeric Key form (`<STR &1 : ...>`) is retained as a lexical alternative; numeric Keys are valid anchor values but no longer carry Template-slot semantics. A numeric Key is just a Key whose value happens to be a numeral.

### 8.4 Reference Literals (Core Spec §5.2.6)

The reference-literal grammar in Core Spec §5.2.6 is replaced by the production from §5 of this specification:

```ebnf
ReferenceLiteral  ::= IdRef | PathRef | KeyRef | IdPathRef | IdKeyRef
IdRef             ::= "@" Identifier
PathRef           ::= "$" PathExpression
KeyRef            ::= "&" KeyToken
IdPathRef         ::= "@" Identifier "$" PathExpression
IdKeyRef          ::= "@" Identifier "&" KeyToken
```

The address-mode singularity requirement (§7 of this specification) is added as a normative paragraph following the production. The "reserved, grammar deferred" language for `$PathExpression` is removed.

### 8.5 Template-Instance Alignment (Core Spec §8.2)

Core Spec §8.2 is amended with an explicit alignment rule: an instance child satisfies a Template slot by matching the slot's `Id` field (case-sensitive ordinal comparison). When a Template slot declares a `Rel`, an instance child carrying the same `Rel` MAY align to the slot even if its local `Id` differs; in that case the `Rel` is the alignment key. The `Key` field plays no role in Template-instance alignment.

### 8.6 Composition Matrix (Core Spec §7.1)

`@KIND` reference elements become resolvable against targets that were previously not addressable by the identifier-mode grammar. The composition matrix expands as follows:

- `@REC`, `@BLK` — resolvable when the Resource is a `PathRef`, `KeyRef`, `IdPathRef`, or `IdKeyRef` whose terminal element is a Record or Block, respectively.
- `@.STR`, `@.INT`, `@.FLOAT`, `@.BOOL`, `@.BYTE` — resolvable when the path or key terminal is a Qualifier of the corresponding value-kind.
- `@_BOOL`, `@_INT`, `@_STR`, `@_FLOAT`, `@_BYTE` — resolvable when the path or key terminal is an Extension of the corresponding value-kind (Extensions are valid only under Templates; the reference must resolve within a Template subtree).
- `@%REC`, `@%OBJ`, `@%STR`, `@%INT`, etc. — resolvable when the path or key terminal is an Embedded element.
- `@STR`, `@INT`, `@FLOAT`, `@BOOL`, `@BYTE` to value Properties — resolvable when the path or key terminal is a value Property of the corresponding value-kind. (Distinct from references to Objects of those names, which is not a legal composition; `@STR` to an Object is malformed.)

Parent-context restrictions on where `@KIND` references may appear (Core Spec §7.1) are unchanged. Only the addressability of the **target** expands; the rules for where a reference element may sit in its parent's children are governed by the existing composition matrix.

### 8.7 Closure of Open Item §11.4

Core Spec §11.4 (Path-Mode Reference Resolution) closes upon publication of this specification. Its forward reference is satisfied here; the open issue is resolved.


---

## 9. Reserved Syntax for Future Revisions

The following syntactic forms are reserved by this specification. They MUST NOT appear in v1 path expressions. Their reservation ensures that v1 paths remain forward-compatible with anticipated query-mode and delta-mode extensions.

| Form                                       | Reserved for                                                              |
|--------------------------------------------|---------------------------------------------------------------------------|
| `*` as a segment                           | Single-step wildcard (query mode).                                        |
| `..` as a segment                          | Descendant axis (query mode).                                             |
| `?` anywhere in a path expression          | Filter / predicate sigil (query mode).                                    |
| `[*]`                                      | Wildcard subscript (query mode).                                          |
| `[0]`                                      | Reserved against use; 0-based ordinals are not introduced.                |
| `[1:5]`, `[1..3]`                          | Ranges or slices within brackets (query mode).                            |
| `[?(…)]`                                   | Filter expression within brackets (query mode).                           |
| `[name]`, `["name"]`                       | Named subscripts (`IdSegment` is the v1 equivalent).                     |
| Comma inside brackets — `[1,3]`, `[a,b]`   | Union of multiple selectors (query mode).                                 |
| `[-1]`, `[-N]`                             | Last-N-th selectors (query mode).                                         |
| `-` as a segment                           | Append-target marker for delta operations.                                |
| `$./PathExpression`                        | Relative path rooted at the referencing element (future revision).        |
| `$../PathExpression`                       | Relative path rooted at the parent of the referencing element (future).   |
| Compound segments — multiple selectors per step | Within-segment combinator (future revision).                         |
| `&Key` as a path segment                   | Key-based addressing is provided as a reference-literal form (`KeyRef`, `IdKeyRef`; §5), not as a path segment. The `&` sigil is reserved against use inside `PathExpression` so that a future revision MAY reintroduce Key matching as a path segment if implementer experience demonstrates a need. |

A v1 path expression that uses only the three segment kinds defined in §4 with the `/` separator is a strict syntactic subset of any future query-mode or delta-mode extension. Such a path expression will continue to parse and resolve identically when those extensions are introduced.


---

## 10. Open Items

The following items are not resolved by this specification and are expected to be addressed in a subsequent revision. They are listed here so that implementations can plan for clarifications and so that authors are aware of areas where behavior is currently underspecified.

### 10.1 Node-Zone Boundary Behavior

Node-zone elements (the `N` modifier; Core Spec §6.5.13) are described as semi-structured isolated subtrees with composition rules distinct from the rest of GDX (Core Spec §7.2 rule 5). It is not normatively specified whether a path expression may traverse into a Node-zone subtree from outside, whether path expressions written **within** a Node-zone subtree may reach **outside** the Node-zone, and whether the segment-kind selectors retain their semantics inside Node-zone material.

A future revision is expected to specify either (a) Node-zone subtrees are opaque to outside path expressions (paths halt at the Node-zone boundary), or (b) paths traverse Node-zone subtrees with the same segment semantics. Implementers SHOULD NOT rely on either behavior in v1.

### 10.2 Path Behavior Across Embedded Elements

Embedded elements (the `%` modifier; Core Spec §6.5.11) appear in property-value position while retaining their base role's children and identifier rules. Path traversal through an Embedded element behaves naturally — the Embedded element is a child of its parent, and its own children are accessed by further segments — but this should be stated explicitly in a subsequent revision, including the case where a `PathRef` terminates at an Embedded element.

### 10.3 Ordinal Counting and Annotations

`OrdinalSegment` is specified to count "all children in document order." It is not made explicit whether this count includes Annotations (Core Spec §6.5.6, REM and other annotation forms). Two interpretations are plausible:

- **Inclusive.** Annotations count as children for ordinal purposes; `[1]` may select an Annotation if it precedes other children.
- **Exclusive.** Annotations are skipped; only "structural" children are counted.

The inclusive interpretation is simpler to specify and implement; the exclusive interpretation is more useful for authors who treat Annotations as out-of-band material. A future revision SHOULD pick one normatively. In the absence of a normative ruling, conforming implementations SHOULD use the inclusive interpretation for deterministic behavior.

### 10.4 Set-Member Ordering

`OrdinalSegment` matches by document order. For Set elements (`KIND*`; Core Spec §6.3.1), the document order of Set members is the order in which they appear in the serialized form. The Set's logical semantics (unordered collection vs. ordered list) does not affect ordinal matching: `[3]` selects the third member as written, regardless of the consumer's interpretation of Set order. This SHOULD be clarified in a subsequent revision.

### 10.5 Resource Values Are Not Path Targets

A path expression's terminal element is always an element, not a Resource value. A path cannot address the value of a Property's Resource directly; the path resolves to the Property element, and the Resource is then accessed through the resolved element's value-kind machinery. This is implicit in the segment-matching rules (segments match children, which are elements) but should be stated explicitly in a subsequent revision.

### 10.6 Identifier Normalization

Per Core Spec §A.2, identifier comparison is case-sensitive ordinal. The path expression inherits this comparison rule for `IdSegment` and `RelSegment` matching; the `KeyRef` and `IdKeyRef` reference literals inherit the same comparison rule for `KeyToken` matching. Unicode normalization (NFC, NFD, etc.) is not specified; conforming documents MUST use a single normalization form throughout. A future revision MAY normatively specify a Unicode normalization profile.


---

## Appendix A — Complete Grammar

```ebnf
// Reference literals (§5; replaces Core Spec §5.2.6)
ReferenceLiteral  ::= IdRef | PathRef | KeyRef | IdPathRef | IdKeyRef

IdRef             ::= "@" Identifier
PathRef           ::= "$" PathExpression
KeyRef            ::= "&" KeyToken
IdPathRef         ::= "@" Identifier "$" PathExpression
IdKeyRef          ::= "@" Identifier "&" KeyToken


// Path expression (§3, §4)
PathExpression    ::= "/" Segment ("/" Segment)*

Segment           ::= IdSegment
                    | RelSegment
                    | OrdinalSegment

IdSegment         ::= SegmentName
RelSegment        ::= "#" SegmentName
OrdinalSegment    ::= "[" Ordinal "]"

KeyToken          ::= SegmentName | DIGIT+
Ordinal           ::= NONZERO_DIGIT DIGIT*


// Segment-internal lexical alphabet (§3.3, §4.1)
SegmentName       ::= SegStartChar SegContChar*
SegStartChar      ::= ALPHA | "_"
SegContChar       ::= ALPHA | DIGIT | "_" | "-" | "." | ":"


// Identifier grammar (Core Spec §5.4.1, as revised by §8.1)
Identifier        ::= IdStartChar IdContChar*
IdStartChar       ::= ALPHA | "_"
IdContChar        ::= ALPHA | DIGIT | "_" | "-" | "." | "/" | ":"


// Character classes
ALPHA             ::= "A".."Z" | "a".."z"
DIGIT             ::= "0".."9"
NONZERO_DIGIT     ::= "1".."9"
```


---

## Appendix B — Worked Examples

### B.1 Intra-Document Reference to a Record

```
<OBJ people/alice
    <REC mailing
        <STR street : "123 Main St">
        <STR city : "Oakland">
    >
>

<@REC x : $/people/alice/mailing>
```

The `$` introduces a `PathRef`. The path `/people/alice/mailing` traverses from the document root through three `IdSegment` selectors. The reference resolves to the `<REC mailing>` element.

### B.2 Cross-Document Reference to a Record (Name-Resolved Start)

```
<@REC home : @gdx:bob$/contacts/[1]>
```

The `@gdx:bob` portion resolves the Object whose Id is `gdx:bob` by scope walk. The `$/contacts/[1]` portion then traverses from that Object to its `contacts` child (matched by `IdSegment`), and then to the first child of `contacts` by document order (matched by `OrdinalSegment`). The reference resolves to that first child, which must be of role Record for the `@REC` prefix to be satisfied.

### B.3 Cross-Document Reference (IRI-Resolved Start)

```
<@.STR street : @https://acme.com/people/alice$/address/[2]>
```

The IRI `https://acme.com/people/alice` resolves (if the implementation supports network resolution) to a remote Object. The path `/address/[2]` traverses from that Object to its `address` child (by Id), and then to the second child of `address` by document order. The reference resolves to a string Qualifier, satisfying the `@.STR` prefix.

### B.4 Rel-Based Selection Within Current Document

```
<OBJ people/alice
    <EDGE #gdx:knows : @people/bob>
>

<@EDGE primary : $/people/alice/#gdx:knows>
```

The path resolves through two `IdSegment` selectors to `<OBJ people/alice>`, then selects its child by Rel field. The reference resolves to the Edge.

This example assumes only one Edge child of `<OBJ people/alice>` carries `Rel` `gdx:knows`. If multiple Edges shared that Rel, the path would match multiple elements and the address-mode resolver would treat the reference as unresolved (§7).

### B.5 Mixed Selectors

```
<@REC postal : @gdx:bob$/contacts/[1]/#gdx:postalAddress>
```

The `@gdx:bob` portion name-resolves the Object. The path then applies, in order:

- `/contacts` — `IdSegment`, selecting by Id.
- `/[1]` — `OrdinalSegment`, selecting the first child of `contacts`.
- `/#gdx:postalAddress` — `RelSegment`, selecting the child whose Rel field is `gdx:postalAddress`.

Each segment uses exactly one selector kind, as required by the one-selector-per-segment rule (§3.2).

### B.6 Address-Mode Resolution Failure (Multi-Match)

```
<OBJ people/alice
    <EDGE #gdx:knows : @people/bob>
    <EDGE #gdx:knows : @people/carol>
>

<@EDGE x : $/people/alice/#gdx:knows>     // unresolved: terminal match set has cardinality 2
```

The path is syntactically well-formed and parses correctly. However, the terminal `RelSegment` matches two Edges. In address mode (§7), the resolver MUST NOT pick a first match; the reference is unresolved. To resolve to a specific Edge, the author must refine the path — for example by Ordinal:

```
<@EDGE first  : $/people/alice/[1]>   // first child by order, assuming it is the desired Edge
<@EDGE second : $/people/alice/[2]>   // second child by order
```

### B.7 Reference to a Qualifier

```
<OBJ products/123
    <STR price : "29.99"
        <.STR currency : "USD">
    >
>

<@.STR currencyRef : $/products/123/price/currency>
```

The path traverses Object → Property → Qualifier. The terminal element is the `<.STR currency>` Qualifier; the enclosing `@.STR` prefix matches its role (string Qualifier). The reference resolves successfully.

### B.8 Reference to an Embedded Element

```
<OBJ orders/42
    <%REC delivery
        <STR street : "Broadway">
        <STR city : "Oakland">
    >
>

<@%REC d : $/orders/42/delivery>
```

The path resolves to the Embedded Record. The `@%REC` prefix matches the role (Embedded Record). Resolution succeeds.

### B.9 Key-Based Reference to a Non-Identified Element

```
<OBJ system/audit-log
    <REC* entries
        <REC &entry-12345 #log:event
            <STR severity : "warn">
            <STR message : "auth failure">
        >
        <REC &entry-12346 #log:event
            <STR severity : "info">
            <STR message : "session start">
        >
    >
>

<OBJ compliance/incident-7
    <@REC originating : @system/audit-log&entry-12345>
>
```

The reference `@system/audit-log&entry-12345` is an `IdKeyRef`. The `@system/audit-log` portion name-resolves the audit-log Object; the `&entry-12345` portion then looks up the Keyed element within that Object's scope. The reference resolves to the `<REC &entry-12345>` element regardless of its position within the Object's subtree.

The Key-based form is location-stable: if the log document is restructured (entries reorganized into nested categories, archived sections moved, etc.), the reference continues to resolve as long as the Key is preserved. A path-based equivalent such as `$/system/audit-log/entries/[1]` would break under any such restructuring.

### B.10 Key Lookup Within the Current Document

```
<OBJ system/audit-log
    <REC* entries
        <REC &entry-12345 #log:event
            <STR severity : "warn">
        >
    >
>

<OBJ compliance/internal-incident
    <@REC originating : &entry-12345>
>
```

When the referencing element and the Keyed target share a document, the `@Identifier` portion may be omitted. The bare `&entry-12345` form (a `KeyRef`) looks up the Key within the document, walking scopes per Core Spec §5.4.5. This form is shorter but requires that the consuming context know the document boundary; for cross-document references, use `IdKeyRef`.


---

## Appendix C — Migration Notes

### C.1 Existing Path-Style Identifiers

Documents conforming to GDX Core Specification v0.10.0 or earlier may use path-style local identifiers such as `<OBJ /people/alice>`. Under the revised `IdStartChar` production (§8.1), these are no longer valid because identifiers may no longer **begin** with `/`.

Two migration patterns are available:

1. **Drop the leading slash.** Rewrite `<OBJ /people/alice>` as `<OBJ people/alice>`. The interior `/` is preserved because `IdContChar` still admits `/`. References change correspondingly: `@/people/alice` becomes `@people/alice`.
2. **Replace identifier references with path references.** Where another element used `@/people/alice` as a reference, write `$/people/alice` (if the target is in the current document) or use a CURIE/IRI form for cross-document targets. This is the recommended pattern when the reference is structurally to an element, rather than nominally to an identified resource.

### C.2 Reference Literal Forms

Existing references using `@Identifier` continue to work without modification, subject to the `IdStartChar` narrowing.

Existing usages of `$PathExpression` (which were previously implementation-defined under Core Spec §11.4) MUST be reviewed against the v1 grammar in this specification. Implementations that previously accepted permissive or non-conforming `$`-prefixed literals MUST be updated to reject any form not produced by the grammar in §5 and Appendix A.

### C.3 Documents Unaffected by Migration

A document that meets **all** of the following conditions is unaffected by this specification and requires no migration:

- No identifier in any element field begins with `/`.
- No reference literal uses a `$`-prefixed form.

Such documents continue to be conforming under the revised Core Specification without change.

### C.4 Tooling and Parsers

Parser implementations conforming to a pre-v1 reference-literal grammar MUST be updated to:

1. Reject identifiers beginning with `/` (per the revised §5.4.1).
2. Accept the three `ReferenceLiteral` forms defined in §5 of this specification.
3. Enforce the address-mode singularity rule (§7) for `@KIND` reference elements.

Implementations that defer path-mode resolution to an external resolver MAY continue to do so; the grammar layer is independent of the resolution layer, and parsers MAY accept syntactically valid path literals without performing resolution.


---

