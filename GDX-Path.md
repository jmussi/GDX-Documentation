# GDX Path Expressions — Specification

- **Status:** Draft v0.9 (companion to GDX Core Specification v0.27.0, working draft)
- **Date:** 2026-07-19
- **Companion to:** GDX Core Specification v0.27.0
- **Authors:** Julian Mussi


## Abstract

This document defines the GDX Path Expression — a syntax and resolution model for addressing individual elements within a GDX document by their structural location. Path expressions complement the identifier-mode reference syntax (`@Identifier`) and the id-mode reference syntax (`&Id`) defined in the GDX Core Specification by providing access to elements that lack a globally addressable identifier, including Records, Blocks, value Properties, Qualifiers, and Extensions.

The path grammar is uniform across all consuming contexts. This specification defines the syntax and per-segment selection semantics; the cardinality and failure semantics for any given consumer (references, delta operations, queries) are specified by the consuming layer. This document normatively defines address-mode behavior — the cardinality contract used by `@KIND` reference elements (Core Specification §6.16). Query-mode and delta-mode behaviors are reserved for future companion specifications.

This specification resolves the `$PathExpression` reservation in Core Specification §11.4 (Path-Mode Reference Resolution). It defines the five reference-literal forms recognized in `%KIND` REF resources and `@OBJ`/`@REL`/`@HEAD` @-Edge resources: `NameRef` (`@Base`), `PathRef` (`$PathExpression`), `IdRef` (`&Id`), `NamePathRef` (`@Base$PathExpression`), and `NameIdRef` (`@Base&Id`). The composed forms `NamePathRef` and `NameIdRef` express cross-document addressing by structural path or by durable id respectively. The `Base` production carries an optional role marker (`@OBJ:id`, `@REL:id`, `@DEFOBJ:id`, etc.) drawn from the closed prefix vocabulary; when omitted, composed references default to an Object base.

This document does not address query-mode semantics (wildcards, filters, descendant axis, projections), delta/patch operations, or filter expression languages. The syntactic territory those features require is enumerated as reserved (§9) so that v1 path expressions remain forward-compatible.


## Table of Contents

1. [Notation](#1-notation)
2. [Scope and Modes](#2-scope-and-modes)
3. [Path Expression Syntax](#3-path-expression-syntax)
4. [Segment Kinds](#4-segment-kinds)
5. [Composition with Identifier and Id References](#5-composition-with-identifier-and-id-references)
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

A `PathExpression` is introduced by the `$` sigil in textual contexts that accept a reference literal (Core Spec §4.2.6). The path begins with a `/` segment separator and consists of one or more path segments, each separated by `/`:

```ebnf
PathExpression ::= "/" Segment ("/" Segment)*
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
Segment ::= NameSegment | PredicateSegment | OrdinalSegment
```

Segment kind is determined by the segment's first character:

| First character    | Segment kind     |
|--------------------|------------------|
| `#`                | `PredicateSegment`     |
| `[`                | `OrdinalSegment` |
| otherwise (letter or `_`) | `NameSegment` |

A v1 segment carries exactly one selector. Compound segments — those combining more than one selector form within a single step (for example, an Name combined with a Predicate constraint) — are reserved for a future revision (§9).

Id-based addressing does not appear as a path segment. The `Id` field is a in-document id mechanism (Core Spec §3.2, §5.4.3) and is addressed via the `&Id` reference literal forms (§5) rather than by path traversal. A path expression that walks structurally to an id-bearing element is permissible (matching the element by Name, Predicate, or position) but inefficient compared to the direct Id reference; the path syntax intentionally does not provide an Id-matching segment kind.

### 3.3 Lexical Alphabet

The character set within an `NameSegment` and a `PredicateSegment` is governed by the `SegmentName` production (§4.1). The `SegContChar` alphabet is a narrowing of the Core Spec §4.4.1 `NameContChar` alphabet: within a path segment, the `/` character is **not** a valid continuation character. This narrowing applies only inside a path expression; in other identifier-bearing positions, identifiers continue to admit `/` as a continuation character so that IRIs remain valid identifiers.

A `PathExpression` terminates when the lexer encounters a character outside the segment-internal alphabet. In practice, this is typically a whitespace character, a `>` closing-bracket, or another sigil character.

### 3.4 Whitespace Rules

No whitespace is permitted anywhere within a `PathExpression`. The path is a single contiguous token bounded by:

- The `$` introducer (or the `$` following the `@Identifier` portion of a composed reference, §5) on the left.
- The first character outside the segment-internal alphabet on the right.

The whitespace tightness rule for `$` and `@` (no whitespace between sigil and content) is established by Core Spec §4.1.1.


---

## 4. Segment Kinds

### 4.1 NameSegment

```ebnf
NameSegment    ::= SegmentName

SegmentName  ::= SegStartChar SegContChar*
SegStartChar ::= ALPHA | "_"
SegContChar  ::= ALPHA | DIGIT | "_" | "-" | "." | ":"

ALPHA        ::= "A".."Z" | "a".."z"
DIGIT        ::= "0".."9"
```

An `NameSegment` is a bareword. It matches a child element whose `Name` field is string-equal to the segment text, using the case-sensitive ordinal comparison defined in Core Spec §3.2.

Examples:

- `addr` matches a child with `Name` `addr`.
- `gdx:Person` matches a child with `Name` `gdx:Person` (a CURIE; `:` is in `SegContChar`).
- `com.acme.Address` matches a child with that dotted Name.

The `SegContChar` alphabet excludes `/` (cf. Core Spec §4.4.1 `NameContChar`). To address an element whose `Name` itself contains `/`, give the element an Id (Core Spec §3.2, §5.4.3) and reference it via the `&Id` reference literal (§5.1), or address the element directly via an identifier-mode reference (`@Identifier`).

### 4.2 PredicateSegment

```ebnf
PredicateSegment ::= "#" SegmentName
```

A `PredicateSegment` matches a child element whose `Predicate` field is string-equal to `SegmentName`. CURIEs and other identifier forms admitted by `SegmentName` are valid as the Predicate value. An element whose Predicate field is empty — for example, an @-edge whose relationship is undefined as to nature (Core Spec §6.4.2, added in Core v0.20.0) — is matched by no `PredicateSegment`; only a named Predicate can be selected by `#SegmentName`. To address such an element, use an `OrdinalSegment` or, when it carries an `Id`, the `&Id` reference literal.

Examples:

- `#gdx:knows` matches a child with `Predicate` `gdx:knows`.
- `#schema:givenName` matches a child with that Predicate.

The Predicate field is not constrained to be unique among the children of an element. Multiple Properties, Edges, or other elements MAY share the same Predicate value (Core Spec §6.4 explicitly allows multiple Edges of the same Predicate between the same source/target pair). A `PredicateSegment` therefore may match zero, one, or many child elements at the data layer.

Address-mode consumers (§7) require exactly one match. Authors writing paths in address-mode contexts SHOULD select segments such that the terminal step matches a single child; this typically means using an `NameSegment` or `OrdinalSegment` at any step where multiple siblings could share a Predicate. Alternatively, when the target element carries an `Id`, the `&Id` reference literal (§5.1) addresses it directly without requiring path traversal at all.

### 4.3 OrdinalSegment

```ebnf
OrdinalSegment ::= "[" Ordinal "]"
Ordinal        ::= NONZERO_DIGIT DIGIT*
NONZERO_DIGIT  ::= "1".."9"
```

An `OrdinalSegment` matches the *N*-th child of the parent in document order, where *N* is the bracketed numeral. Ordinals are **1-based**: `[1]` is the first child by document order.

The following forms are malformed:

- `[0]` — no zero ordinal.
- `[01]`, `[007]` — no leading-zero ordinals.
- `[-1]` — no negative ordinals (the negative form is reserved for a future "last-N" extension).
- `[]` — empty brackets.
- `[ 3 ]`, `[3 ]` — no whitespace inside brackets.

The ordinal counts all child elements of the parent in document order, regardless of role or modifier. It is a positional selector based purely on document order; it does not consult the `Id` field or Set-membership ordering.

The contents of `[ ]` are restricted to a base-10 numeral in v1. Wildcards (`[*]`), ranges (`[1:5]`, `[1..3]`), filter expressions (`[?(…)]`), unions (`[1,3]`), and named subscripts (`[name]`) are reserved (§9).

### 4.4 Selector Semantics Summary

| Selector              | Matches against         | Uniqueness guarantee at data layer                                  |
|-----------------------|-------------------------|---------------------------------------------------------------------|
| `NameSegment` (bareword)| `Name` field              | Unique within parent scope (Core Spec §3.2: collisions are malformed) |
| `PredicateSegment` (`#R`)   | `Predicate` field             | Not unique — multiple siblings may share a Predicate                       |
| `OrdinalSegment` (`[N]`) | Position in document order | Unique by construction                                          |

The uniqueness column indicates the data-layer guarantee per Core Specification. The grammar does not require an `NameSegment` to match exactly one child if the data is malformed; resolution behavior in the presence of duplicate Names is implementation-defined. Conforming documents do not produce such collisions (Core Spec §3.2).

Id-based addressing — matching an element by its `Id` field — is not a path segment kind in v1. Ids are addressed directly via the `&Id` and `@Identifier&Id` reference literal forms (§5). The rationale is that Ids provide location-stable reference targets (Core Spec §3.2); folding Id matching into path segments would couple Ids to the very location they are designed to be independent of. A path expression that matches by Name, Predicate, or ordinal at each step composes naturally with an Id reference at the start; a path that uses Id matching mid-way is redundant with the direct reference form.


---

## 5. Composition with Identifier and Id References

The `$PathExpression` syntax composes with the identifier-mode reference (`@Identifier`, Core Spec §4.2.6) and the id-mode reference (`&Id`, Core Spec §4.2.6) to express references that combine name resolution, structural traversal, and id lookup. Five reference-literal forms are recognized:

```ebnf
ReferenceLiteral ::= NameRef | PathRef | IdRef | NamePathRef | NameIdRef

NameRef            ::= "@" Base
PathRef          ::= "$" PathExpression
IdRef        ::= "&" IdToken
NamePathRef        ::= "@" Base "$" PathExpression
NameIdRef      ::= "@" Base "&" IdToken

Base             ::= (BaseRole ":")? Identifier
BaseRole         ::= "OBJ"  | "REL"  | "PKG"  | "GRP"  | "HEAD"
                   | "DEFOBJ" | "DEFREC" | "DEFSTR"
                   | "DEFINT" | "DEFFLOAT" | "DEFBOOL" | "DEFBYTE" | "DEFHEAD"

IdToken      ::= IdStartChar NameContChar*   (* Core Spec §4.4.3 — authoritative *)
IdStartChar  ::= ALPHA | DIGIT | "_"
```

The `IdToken` production above is reproduced for reference only; Core Spec §4.4.3 is authoritative for it (a single grammar permitting a leading digit, replacing the earlier `SegmentName | DIGIT+` split). The `Base` production extends the identifier portion of `@`-references with an optional role marker. The marker is a closed-vocabulary keyword drawn from §6.2 of the Core Specification, followed by `:`, followed by the identifier. The marker is lexically distinguished from a CURIE prefix (`gdx:Person`, `schema:givenName`) by lookup against the closed `BaseRole` set: only the keywords listed above are recognized as role markers; anything else parses as part of the identifier.

### 5.1 Form Semantics

- **`NameRef`** — name-mode resolution. The `@Base` is resolved per the scope walk of Core Spec §6.16. When the base carries no role marker, the role is inferred from the enclosing `@KIND` reference element's prefix. When the base carries a role marker, the resolver looks for an element of the named role with the matching identifier.
- **`PathRef`** — structural traversal rooted at the document containing the reference literal.
- **`IdRef`** — id-mode lookup, searching outward from the reference to the document root (Core Spec §4.4.5).
- **`NamePathRef`** — composed name + path. The `@Base` portion resolves to a starting element; the `$PathExpression` portion is then evaluated rooted at that starting element. When the base carries no role marker, the starting element defaults to an Object (the most common base for cross-document path addressing).
- **`NameIdRef`** — composed name + id. The `@Base` portion resolves to a starting element; the `&Id` portion is then looked up within that element's scope. When the base carries no role marker, the starting element defaults to an Object.

Each sigil retains its independent meaning under composition. The `@` operator performs name resolution (with optional role disambiguation); the `$` operator performs structural traversal; the `&` operator performs id lookup. Composition reads left-to-right.

### 5.2 Base Role Markers

The optional `BaseRole` prefix on an `@`-reference makes the role of the resolved element explicit:

```
@bob                    // implicit; role inferred from enclosing prefix or defaults to OBJ
@OBJ:bob                // explicit Object
@REL:my-rel             // explicit REL
@DEFOBJ:Customer        // explicit Object Template
@PKG:vendor-pack        // explicit Package
```

The role marker is most useful in composed references, where the base's role differs from the terminal target's role:

```
<%STR description : @REL:my-rel&annotation-3>
```

Here the enclosing `%STR` says the terminal target is a string Property. The `@REL:my-rel` portion says the base is a REL named `my-rel`. The `&annotation-3` portion looks up an id-bearing element within the REL's subtree. Without the role marker, the base would default to Object, and the reference would be unresolvable.

**Parse-time validation.** The parser checks that `BaseRole` is one of the keywords in the closed set above. An unknown role keyword (e.g., `@FOO:bar`) is malformed and MUST be rejected.

**Resolution-time validation.** The resolver verifies that the resolved element actually has the claimed role. A mismatch — for example, `@REL:foo` where `foo` resolves to an Object — yields an unresolved reference. This deferred validation means the parser does not need access to the data; only the resolver does.

**Marker is optional.** When the role is unambiguous from context — the enclosing prefix on a non-composed reference, or the default-OBJ rule on a composed reference — the marker MAY be omitted. The terse form `@bob` and the explicit form `@OBJ:bob` are equivalent when the inferred role is OBJ.

**Default base role in composed references is OBJ.** This default reflects the common case: most cross-document references id at an Object boundary and then traverse or look up within. Authors targeting non-Object bases (RELs, Templates, transport-layer containers) MUST use an explicit role marker.

### 5.3 Whitespace and Boundaries

No whitespace is permitted between `@`, the `Base`, `$`, `&`, the path segments, or the Id token. The entire `ReferenceLiteral` is one contiguous token.

In an `NamePathRef`, the `Base` following `@` is consumed up to the first `$`. In an `NameIdRef`, the `Base` is consumed up to the first `&`. Because `$` and `&` are not members of `NameContChar`, the boundaries are lexically clean.

Within an `Base`, the role marker (if present) extends from `@` up to and including `:`. The identifier portion follows the `:`. If the bareword before `:` is in the `BaseRole` set, the `:` is interpreted as the role separator; if not, the bareword and `:` are part of a CURIE-style identifier (e.g., `gdx:Person`).

The `$` and `&` operators do not combine within a single reference literal. The form `@Base$PathExpression&Id` is malformed; pick one of `NamePathRef` or `NameIdRef`.

### 5.4 Cross-Document Addressing

`NamePathRef` and `NameIdRef` are the canonical forms for cross-document addressing. The `@Base` portion may resolve to an element in any scope:

- Local-document (the identifier matches an element within the same document).
- Namespace-relative (the identifier matches a name registered in a vocabulary or package scope).
- Remote IRI (the identifier is a fully qualified IRI; implementations MAY perform network resolution per Core Spec §6.16 rule 3).

The path or key portion is then evaluated within the resolved element's scope:

```
@gdx:bob$/contact/address                       // OBJ base (default), path within
@OBJ:gdx:bob&shippingAddress                    // OBJ base (explicit), id within
@REL:gdx:knows$/range                           // REL base, path within
@DEFOBJ:com.acme.Customer$/firstName            // Template base, path to slot
@https://acme.com/people/alice$/photo           // IRI-resolved (OBJ default) + path
@PKG:vendor-pack&main-product                   // Package base + id
```

A `PathRef` or `IdRef` with no `@Base` portion is rooted at (or scoped within) the current document. These forms cannot address elements in other documents directly.

### 5.5 Empty Composition

The forms `@Base$` (trailing `$` with no path segments), `@Base&` (trailing `&` with no key token), `$` alone, and `&` alone are all malformed. If the resolved element itself is the desired target, omit the trailing operator and use the plain `@Base` form.

### 5.6 Reference-Element Composition

A `@KIND` reference element (Core Spec §6.16) MAY use any of the five reference literal forms as its Resource. The choice of form is governed by what the reference is addressing and how the target is most stably identified:

| Use                                                                                            | Form          |
|------------------------------------------------------------------------------------------------|---------------|
| Reference to an identifiable Object                                                            | `NameRef`       |
| Reference to a non-Name-bearing element in this document by structural location                  | `PathRef`     |
| Reference to an id-bearing element anywhere in this document                                         | `IdRef`      |
| Reference to a non-Name-bearing element in another document by structural location                | `NamePathRef`   |
| Reference to an id-bearing element in another document, or to an id-bearing element scoped within a named Object | `NameIdRef` |
| Reference to a child of a non-Object base (REL, Template, transport container)               | `NamePathRef` or `NameIdRef` with explicit `BaseRole` |

When both an Id and a structural path could address the target, prefer the Id form. Ids are location-stable; paths break when intervening structure changes.


---

## 6. Resolution Algorithm

Path resolution is a left-to-right traversal of the segments of a `PathExpression`, starting from a defined starting element. The starting element is determined by the consuming context:

- For `PathRef` (Core Spec §4.2.6, as revised): the root of the document containing the reference literal. For documents with multiple top-level elements (Core Spec §7.2 rule 1), the document root is a notional container of the top-level elements; the first segment matches against the top-level elements as children of that container.
- For `NamePathRef`: the element produced by name-mode resolution of the `@Identifier` portion. If that resolution fails (no element matches the identifier in any scope), the path is not evaluated and the entire reference is unresolved.
- For other consumers (delta operations, queries — future): the starting element is defined by the consuming specification.

### 6.1 Per-Segment Matching

For each segment in order, the resolver computes the **match set** — the set of child elements of the current element that satisfy the segment selector:

| Segment kind             | Matches                                                                                       |
|--------------------------|-----------------------------------------------------------------------------------------------|
| `NameSegment(name)`        | Each child whose `Name` field is string-equal to `name` (case-sensitive ordinal comparison).   |
| `PredicateSegment(predicate)`        | Each child whose `Predicate` field is string-equal to `predicate`.                                       |
| `OrdinalSegment(n)`      | The *n*-th child by document order (1-based). Match set is empty if the parent has fewer than *n* children. |

A segment match operates only on the **direct children** of the current element. Path expressions in v1 provide no descendant-axis operator; descendant traversal is reserved (§9).

The match operates on all children regardless of their role or modifier: a segment may match a Property, a Record, a Block, an @-edge (`@OBJ`, `@REL`), a Set, a Qualifier, an Extension, a Node-zone element, a VAL slot, an Object (in transport-layer contexts), or any other element type. The role of the matched element is checked against the consumer's role requirement (§7 for address mode); the segment match itself is role-agnostic.

### 6.2 Match Set Cardinality

After each segment, the match set may have any cardinality:

- **Zero elements.** No child of the current element satisfies the selector. Resolution halts at this segment; the path resolves to no element.
- **One element.** The single matched element becomes the current element for the next segment, or is the resolution result if no further segments remain.
- **Multiple elements.** A `PredicateSegment` may produce a multi-element match set when multiple siblings share the same Predicate. (In v1, only `PredicateSegment` can produce multi-match; in future query-mode extensions, wildcards and filters will introduce additional multi-match possibilities.)

In multi-match cases, the resolver does **not** select among the matches. The match set is passed to the consuming context; the cardinality contract of that context (§7 for address mode) determines whether the multi-match is acceptable.

### 6.3 Resolution Outcome

The path expression's outcome is one of:

- **Resolved to one element.** The path evaluation terminated successfully with a single matched element.
- **Resolved to multiple elements.** Match set is greater than one at some step.
- **Unresolved.** Match set is empty at some step.

The consuming specification interprets the outcome according to its cardinality contract. This specification defines address-mode behavior in §7; other modes are reserved.


---

## 7. Address-Mode Singularity

This section is **normative** for the use of path expressions as reference literals in `@KIND` reference elements (Core Spec §6.16) and is anticipated to be referenced normatively by future delta-operation specifications.

A `PathRef` or `NamePathRef` appearing as the Resource of a `@KIND` reference element is **resolved** if and only if path evaluation yields exactly one element whose role matches the enclosing `@KIND` prefix.

Path evaluation yielding any of the following MUST be treated as an unresolved reference:

1. **Zero elements.** No element matched the path at some segment.
2. **Multiple elements.** Match set cardinality is greater than one at the terminal segment.
3. **One element of non-matching role.** The terminal element exists, but its role does not match the enclosing prefix. For example, `<%REC x : $/orders/123>` where the path resolves to an Object is unresolved, because the prefix declares the target as a Record.

A reference may additionally be unresolved by the `@Identifier` portion of an `NamePathRef`: if name-mode resolution fails (no scope yields a target for the identifier), the path is not evaluated and the entire reference is unresolved.

### 7.1 No Implicit First-Match Selection

In address mode, a resolver MUST NOT silently select a first match (or any other deterministic choice) when the terminal match set contains more than one element. The "MUST NOT" rule forecloses silent ambiguity: an author whose path matches more than one element receives an unresolved reference, not a non-deterministic or implementation-dependent selection.

### 7.2 Author Responsibility for Specificity

The grammar admits paths that may resolve ambiguously against some data shapes (typically paths ending in a `PredicateSegment` where multiple siblings share that Predicate). Authors writing paths in address-mode contexts are responsible for making paths specific enough to resolve to exactly one element of the expected role.

When a Predicate-bearing terminal step may have multiple matching siblings, authors SHOULD use a more specific terminal selector:

- An `NameSegment` if the target carries an Name.
- An `OrdinalSegment` if the target is identified by its position among children.

Alternatively, when the target element carries an `Id`, switch from a `PathRef`/`NamePathRef` to an `IdRef`/`NameIdRef` (§5). Id-based addressing avoids path traversal entirely and is location-stable.

### 7.3 Resolution Determinism

For a conforming document, conforming resolvers MUST produce the same resolution outcome (resolved-to-element vs. unresolved) for the same path expression evaluated against the same starting element. Implementation-defined behavior is permitted only for the network-resolution component of identifier-mode resolution (Core Spec §6.16 rule 3); the path-traversal component is fully determined by the document.


---

## 8. Required Changes to GDX Core Specification

The introduction of path-mode reference resolution requires the following modifications to the GDX Core Specification. These changes are normative as of Core Specification v0.11.0, which incorporates path-mode in lockstep with the publication of this companion specification.

### 8.1 Identifier Grammar (Core Spec §4.4.1)

The `NameStartChar` production narrows by removing `/`:

```ebnf
NameStartChar ::= ALPHA | "_"
```

`NameContChar` is **unchanged**; `/` remains a valid continuation character so that IRIs (e.g., `https://acme.com/people/alice`) remain valid identifiers in identifier-bearing positions outside path expressions.

This narrowing makes the `$/...` form of a path expression unambiguous: the `$` introduces path mode, and the immediately-following `/` is the segment separator (since no identifier may begin with `/`). It also makes the boundary between the `@Identifier` and `$PathExpression` halves of an `NamePathRef` lexically clean: an identifier following `@` ends at the first `$`.

Migration impact is documented in Appendix C.

### 8.2 Identifier Forms Table (Core Spec §4.4.2)

The "Path" row of the conventional identifier-forms table is removed. Path-style local naming (e.g., `<OBJ /people/alice>`) is replaced by path-mode reference resolution. The replacement form depends on the use case:

- For local naming, identifiers may continue to embed `/` as a continuation character; only the leading `/` is invalid.
- For references that were path-like in spirit, use a `PathRef` or `NamePathRef`.

### 8.3 Id Field (Core Spec §3.2, §4.4.3, §4.4.5)

The `Id` field is a location-stable in-document id mechanism, not a Template-slot alignment mechanism:

- Ids are an opt-in named-id mechanism providing location-stable reference targets for elements that lack global identifiers. Ids play no role in Template-instance alignment, and (per Core Spec v0.22.0) the `Id` field is excluded from the universal claim rule — a Template MUST NOT declare an Id on a slot.
- Core Spec §4.4.5 defines Id uniqueness: Ids MUST be unique within the enclosing Object's scope. Elements at transport-layer position (direct children of a Package, Group, or document root) MUST have Ids unique within the document.
- **The Id-token grammar is authored by the core, not by this document.** Core Spec §4.4.3 defines `IdToken` as a single grammar — an identifier token whose start set additionally admits a leading digit (`IdToken ::= (ALPHA | DIGIT | "_") NameContChar*`), superseding the former `SegmentName | DIGIT+` split. This admits sequential tokens (`&1`), named ids (`&shippingAddress`), and digit-leading GUID ids (`&7f3a2b9c-…`) under one rule. Id tokens carry no numeric interpretation and compare by ordinal equality (`&01` ≠ `&1`). The grammar blocks in §5 and §8.4 reference the core grammar; this specification no longer originates the `IdToken` production.

### 8.4 Reference Literals (Core Spec §4.2.6)

The reference-literal grammar in Core Spec §4.2.6 is replaced by the production from §5 of this specification:

```ebnf
ReferenceLiteral ::= NameRef | PathRef | IdRef | NamePathRef | NameIdRef
NameRef            ::= "@" Base
PathRef          ::= "$" PathExpression
IdRef        ::= "&" IdToken
NamePathRef        ::= "@" Base "$" PathExpression
NameIdRef      ::= "@" Base "&" IdToken

Base             ::= (BaseRole ":")? Identifier
BaseRole         ::= "OBJ"  | "REL"  | "PKG"  | "GRP"  | "HEAD"
                   | "DEFOBJ" | "DEFREC" | "DEFSTR"
                   | "DEFINT" | "DEFFLOAT" | "DEFBOOL" | "DEFBYTE" | "DEFHEAD"
```

The address-mode singularity requirement (§7 of this specification) is added as a normative paragraph following the production. The "reserved, grammar deferred" language for `$PathExpression` is removed.

The optional `BaseRole` marker disambiguates the role of the resolved element in composed references (`NamePathRef`, `NameIdRef`) when the base is not an Object. In the absence of a marker, the base defaults to OBJ. Parse-time validation rejects unknown role keywords; resolution-time validation ensures the named element matches the claimed role.

### 8.5 Template-Instance Alignment (Core Spec §8.2)

Core Spec §8.2 is amended with an explicit alignment rule: an instance child satisfies a Template slot by matching the slot's `Name` field (case-sensitive ordinal comparison). When a Template slot declares a `Predicate`, an instance child carrying the same `Predicate` MAY align to the slot even if its local `Name` differs; in that case the `Predicate` is the alignment key. The `Id` field plays no role in Template-instance alignment.

### 8.6 Composition Matrix (Core Spec §7.1)

`@KIND` reference elements become resolvable against targets that were previously not addressable by the identifier-mode grammar. The composition matrix expands as follows:

- `%REC`, `%BLK` — resolvable when the Resource is a `PathRef`, `IdRef`, `NamePathRef`, or `NameIdRef` whose terminal element is a Record or Block, respectively.
- `%.STR`, `%.INT`, `%.FLOAT`, `%.BOOL`, `%.BYTE` — resolvable when the path or id terminal is a Qualifier of the corresponding value-kind.
- `%_BOOL`, `%_INT`, `%_STR`, `%_FLOAT`, `%_BYTE` — resolvable when the path or id terminal is an Extension of the corresponding value-kind.
- `%VAL` — resolvable when the path or id terminal is a VAL slot (Core Spec §6.11). Resolution targets the VAL element itself, not its occupant; to address the occupant, the path must include a further segment selecting it.
- `%STR`, `%INT`, `%FLOAT`, `%BOOL`, `%BYTE` to value Properties — resolvable when the path or id terminal is a value Property of the corresponding value-kind. (Distinct from references to Objects of those names, which is not a legal composition; `%STR` to an Object resolves to "non-matching role" and yields an unresolved reference.)
- `%@OBJ`, `%@REL` — resolvable when the path or id terminal is an @-edge of the corresponding family. The @-edge MUST carry identity (Core Spec §6.4 axiom E6) to be id-addressable; path-addressable edges are reached by NameSegment/PredicateSegment/OrdinalSegment traversal from a globally-addressable element.

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
| `[name]`, `["name"]`                       | Named subscripts (`NameSegment` is the v1 equivalent).                     |
| Comma inside brackets — `[1,3]`, `[a,b]`   | Union of multiple selectors (query mode).                                 |
| `[-1]`, `[-N]`                             | Last-N-th selectors (query mode).                                         |
| `-` as a segment                           | Append-target marker for delta operations.                                |
| `$./PathExpression`                        | Relative path rooted at the referencing element (future revision).        |
| `$../PathExpression`                       | Relative path rooted at the parent of the referencing element (future).   |
| Compound segments — multiple selectors per step | Within-segment combinator (future revision).                         |
| `&Id` as a path segment                   | Id-based addressing is provided as a reference-literal form (`IdRef`, `NameIdRef`; §5), not as a path segment. The `&` sigil is reserved against use inside `PathExpression` so that a future revision MAY reintroduce Id matching as a path segment if implementer experience demonstrates a need. |

A v1 path expression that uses only the three segment kinds defined in §4 with the `/` separator is a strict syntactic subset of any future query-mode or delta-mode extension. Such a path expression will continue to parse and resolve identically when those extensions are introduced.


---

## 10. Open Items

The following items are not resolved by this specification and are expected to be addressed in a subsequent revision. They are listed here so that implementations can plan for clarifications and so that authors are aware of areas where behavior is currently underspecified.

### 10.1 Node-Zone Boundary Behavior

Node-zone elements (the `N` modifier; Core Spec §6.13) are described as semi-structured isolated subtrees with composition rules distinct from the rest of GDX (Core Spec §7.2 rule 5). It is not normatively specified whether a path expression may traverse into a Node-zone subtree from outside, whether path expressions written **within** a Node-zone subtree may reach **outside** the Node-zone, and whether the segment-kind selectors retain their semantics inside Node-zone material.

A future revision is expected to specify either (a) Node-zone subtrees are opaque to outside path expressions (paths halt at the Node-zone boundary), or (b) paths traverse Node-zone subtrees with the same segment semantics. Implementers SHOULD NOT rely on either behavior in v1.

### 10.2 Path Behavior Across VAL Slots and Their Occupants

VAL slots (Core Spec §6.11) carry exactly one occupant element as a child. Path traversal through a VAL behaves like traversal through any other parent — the VAL is a child of its containing element, and its occupant is in turn a child of the VAL accessed by further segments — but the two-level identity that VAL introduces (slot-level on the VAL, occupant-level on the contained element) is not given an explicit traversal-form abbreviation. A reference of the form `$/parent/slotName` resolves to the VAL; `$/parent/slotName/occupantId` resolves to the occupant. A future revision MAY introduce an abbreviated form that resolves directly to the occupant (e.g., a sigil that says "the value of this slot"), if implementer experience demonstrates a clear need; v1 does not provide one. Authors should pick the level they need explicitly.

### 10.3 Ordinal Counting and Annotations

`OrdinalSegment` is specified to count "all children in document order." It is not made explicit whether this count includes Annotations (Core Spec §6.6, REM and other annotation forms). Two interpretations are plausible:

- **Inclusive.** Annotations count as children for ordinal purposes; `[1]` may select an Annotation if it precedes other children.
- **Exclusive.** Annotations are skipped; only "structural" children are counted.

The inclusive interpretation is simpler to specify and implement; the exclusive interpretation is more useful for authors who treat Annotations as out-of-band material. A future revision SHOULD pick one normatively. In the absence of a normative ruling, conforming implementations SHOULD use the inclusive interpretation for deterministic behavior.

### 10.4 Set-Member Ordering

`OrdinalSegment` matches by document order. For Set elements (`KIND*`; Core Spec §5.3.1), the document order of Set members is the order in which they appear in the serialized form. The Set's logical semantics (unordered collection vs. ordered list) does not affect ordinal matching: `[3]` selects the third member as written, regardless of the consumer's interpretation of Set order. This SHOULD be clarified in a subsequent revision.

### 10.5 Resource Values Are Not Path Targets

A path expression's terminal element is always an element, not a Resource value. A path cannot address the value of a Property's Resource directly; the path resolves to the Property element, and the Resource is then accessed through the resolved element's value-kind machinery. This is implicit in the segment-matching rules (segments match children, which are elements) but should be stated explicitly in a subsequent revision.

### 10.6 Identifier Normalization

Per Core Spec §A.2, identifier comparison is case-sensitive ordinal. The path expression inherits this comparison rule for `NameSegment` and `PredicateSegment` matching; the `IdRef` and `NameIdRef` reference literals inherit the same comparison rule for `IdToken` matching. Unicode normalization (NFC, NFD, etc.) is not specified; conforming documents MUST use a single normalization form throughout. A future revision MAY normatively specify a Unicode normalization profile.


---

## Appendix A — Complete Grammar

```ebnf
// Reference literals (§5; replaces Core Spec §4.2.6)
ReferenceLiteral ::= NameRef | PathRef | IdRef | NamePathRef | NameIdRef

NameRef            ::= "@" Base
PathRef          ::= "$" PathExpression
IdRef        ::= "&" IdToken
NamePathRef        ::= "@" Base "$" PathExpression
NameIdRef      ::= "@" Base "&" IdToken

Base             ::= (BaseRole ":")? Identifier
BaseRole         ::= "OBJ"  | "REL"  | "PKG"  | "GRP"  | "HEAD"
                   | "DEFOBJ" | "DEFREC" | "DEFSTR"
                   | "DEFINT" | "DEFFLOAT" | "DEFBOOL" | "DEFBYTE" | "DEFHEAD"


// Path expression (§3, §4)
PathExpression   ::= "/" Segment ("/" Segment)*

Segment          ::= NameSegment
                   | PredicateSegment
                   | OrdinalSegment

NameSegment        ::= SegmentName
PredicateSegment       ::= "#" SegmentName
OrdinalSegment   ::= "[" Ordinal "]"

IdToken      ::= IdStartChar NameContChar*   (* Core Spec §4.4.3 — authoritative *)
IdStartChar  ::= ALPHA | DIGIT | "_"
Ordinal          ::= NONZERO_DIGIT DIGIT*


// Segment-internal lexical alphabet (§3.3, §4.1)
SegmentName      ::= SegStartChar SegContChar*
SegStartChar     ::= ALPHA | "_"
SegContChar      ::= ALPHA | DIGIT | "_" | "-" | "." | ":"


// Identifier grammar (Core Spec §4.4.1, as revised by §8.1)
Identifier       ::= NameStartChar NameContChar*
NameStartChar      ::= ALPHA | "_"
NameContChar       ::= ALPHA | DIGIT | "_" | "-" | "." | "/" | ":"


// Character classes
ALPHA            ::= "A".."Z" | "a".."z"
DIGIT            ::= "0".."9"
NONZERO_DIGIT    ::= "1".."9"
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

<%REC x : $/people/alice/mailing>
```

The `$` introduces a `PathRef`. The path `/people/alice/mailing` traverses from the document root through three `NameSegment` selectors. The reference resolves to the `<REC mailing>` element.

### B.2 Cross-Document Reference to a Record (Name-Resolved Start)

```
<%REC home : @gdx:bob$/contacts/[1]>
```

The `@gdx:bob` portion resolves the Object whose Name is `gdx:bob` by scope walk. The `$/contacts/[1]` portion then traverses from that Object to its `contacts` child (matched by `NameSegment`), and then to the first child of `contacts` by document order (matched by `OrdinalSegment`). The reference resolves to that first child, which must be of role Record for the `%REC` prefix to be satisfied.

### B.3 Cross-Document Reference (IRI-Resolved Start)

```
<%.STR street : @https://acme.com/people/alice$/address/[2]>
```

The IRI `https://acme.com/people/alice` resolves (if the implementation supports network resolution) to a remote Object. The path `/address/[2]` traverses from that Object to its `address` child (by Name), and then to the second child of `address` by document order. The reference resolves to a string Qualifier, satisfying the `%.STR` prefix.

### B.4 Predicate-Based Selection Within Current Document

```
<OBJ people/alice
    <STR #gdx:givenName name : "Alice">
>

<%STR alignedName : $/people/alice/#gdx:givenName>
```

The path resolves through two `NameSegment` selectors to `<OBJ people/alice>`, then selects its child by Predicate field. The reference resolves to the string Property.

This example assumes only one child of `<OBJ people/alice>` carries `Predicate` `gdx:givenName`. If multiple Properties shared that Predicate, the path would match multiple elements and the address-mode resolver would treat the reference as unresolved (§7).

### B.5 Mixed Selectors

```
<%REC postal : @gdx:bob$/contacts/[1]/#gdx:postalAddress>
```

The `@gdx:bob` portion name-resolves the Object. The path then applies, in order:

- `/contacts` — `NameSegment`, selecting by Name.
- `/[1]` — `OrdinalSegment`, selecting the first child of `contacts`.
- `/#gdx:postalAddress` — `PredicateSegment`, selecting the child whose Predicate field is `gdx:postalAddress`.

Each segment uses exactly one selector kind, as required by the one-selector-per-segment rule (§3.2).

### B.6 Address-Mode Resolution Failure (Multi-Match)

```
<OBJ people/alice
    <STR #gdx:nickname casual : "Ali">
    <STR #gdx:nickname formal : "Alicia">
>

<%STR x : $/people/alice/#gdx:nickname>     // unresolved: terminal match set has cardinality 2
```

The path is syntactically well-formed and parses correctly. However, the terminal `PredicateSegment` matches two Properties. In address mode (§7), the resolver MUST NOT pick a first match; the reference is unresolved. To resolve to a specific Property, the author must refine the path — for example by Ordinal:

```
<%STR first  : $/people/alice/[1]>   // first child by order
<%STR second : $/people/alice/[2]>   // second child by order
```

### B.7 Reference to a Qualifier

```
<OBJ products/123
    <STR price : "29.99"
        <.STR currency : "USD">
    >
>

<%.STR currencyRef : $/products/123/price/currency>
```

The path traverses Object → Property → Qualifier. The terminal element is the `<.STR currency>` Qualifier; the enclosing `%.STR` prefix matches its role (string Qualifier). The reference resolves successfully.

### B.8 Reference to a VAL Slot

```
<OBJ orders/42
    <VAL delivery
        <REC <STR street : "Broadway"> <STR city : "Oakland">>
    >
>

<%VAL d : $/orders/42/delivery>
```

The path resolves to the VAL slot. The `%VAL` prefix matches the role. To reach the occupant Record instead, the path would extend by one more segment selecting the Record (e.g., by ordinal: `$/orders/42/delivery/[1]`).

### B.9 Id-Based Reference to a Non-Identified Element

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
    <%REC originating : @system/audit-log&entry-12345>
>
```

The reference `@system/audit-log&entry-12345` is an `NameIdRef`. The `@system/audit-log` portion name-resolves the audit-log Object; the `&entry-12345` portion then looks up the id-bearing element within that Object's scope. The reference resolves to the `<REC &entry-12345>` element regardless of its position within the Object's subtree.

The Id-based form is location-stable: if the log document is restructured (entries reorganized into nested categories, archived sections moved, etc.), the reference continues to resolve as long as the Id is preserved. A path-based equivalent such as `$/system/audit-log/entries/[1]` would break under any such restructuring.

### B.10 Id Lookup Within the Current Document

```
<OBJ system/audit-log
    <REC* entries
        <REC &entry-12345 #log:event
            <STR severity : "warn">
        >
    >
>

<OBJ compliance/internal-incident
    <%REC originating : &entry-12345>
>
```

When the referencing element and the id-bearing target share a document, the `@Identifier` portion may be omitted. The bare `&entry-12345` form (a `IdRef`) looks up the Id within the document, walking scopes per Core Spec §4.4.5. This form is shorter but requires that the consuming context know the document boundary; for cross-document references, use `NameIdRef`.

### B.11 Reference to a Child of a Non-Object Base

When the base of a composed reference is not an Object — for example, a REL declaration with an annotation child — an explicit base role marker is required.

```
<REL my-rel
    <STR &description : "A description annotation on the REL itself.">
>

<REL other-rel
    <%STR descRef : @REL:my-rel&description>
>
```

The reference `@REL:my-rel&description` is an `NameIdRef` with an explicit `BaseRole` of `REL`. The `@REL:` marker tells the resolver that `my-rel` should be looked up as a REL, not as an Object (the default for composed references). The `&description` portion then looks up the id-bearing STR within the REL's subtree. Without the role marker, the resolver would attempt to resolve `my-rel` as an Object, fail, and the reference would be unresolved.

The same pattern applies when the base is a Template, a Package, or a Group:

```
<%STR slotDoc : @DEFOBJ:com.acme.Customer&firstName-doc>
<@OBJ entry   : @PKG:vendor-pack&primary-product>
```


---

## Appendix C — Migration Notes

### C.1 Existing Path-Style Identifiers

Documents conforming to GDX Core Specification v0.10.0 or earlier may use path-style local identifiers such as `<OBJ /people/alice>`. Under the revised `NameStartChar` production (§8.1), these are no longer valid because identifiers may no longer **begin** with `/`.

Two migration patterns are available:

1. **Drop the leading slash.** Rewrite `<OBJ /people/alice>` as `<OBJ people/alice>`. The interior `/` is preserved because `NameContChar` still admits `/`. References change correspondingly: `@/people/alice` becomes `@people/alice`.
2. **Replace identifier references with path references.** Where another element used `@/people/alice` as a reference, write `$/people/alice` (if the target is in the current document) or use a CURIE/IRI form for cross-document targets. This is the recommended pattern when the reference is structurally to an element, rather than nominally to an identified resource.

### C.2 Reference Literal Forms

Existing references using `@Identifier` continue to work without modification, subject to the `NameStartChar` narrowing.

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

