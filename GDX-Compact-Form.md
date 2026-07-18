# GDX Compact Form — Grammar Specification

- **Status:** Draft v0.11 (companion to GDX Core Specification v0.26.0, working draft)
- **Date:** 2026-07-17
- **Companion to:** GDX Core Specification v0.26.0, GDX Standard Grammar v0.14, GDX Path Expressions v0.8
- **Authors:** Julian Mussi


## Abstract

This document defines the grammar for GDX Compact Form, a line-oriented serialization of GDX intended for human authoring and reading. The Compact Form preserves the data model defined in the Core Specification and admits a bidirectional mapping to the Standard Form. It uses indentation to convey structural nesting in place of the bracket delimiters used by Standard Form.

The Compact Form is positioned as an input/authoring format. Standard Form remains the canonical form for storage, transport, content addressing, and machine processing. Implementations consuming Compact Form typically parse it, build the data model, and either operate on the model directly or serialize to Standard Form for downstream use.

This document does not address the Standard Form, the Binary Form, semantic validation, or vocabulary content. Those concerns are addressed in their respective specifications.


## Status of This Document

This is the v0.3 working draft. The grammar, indentation model, and Standard Form mapping are stable. The conversion algorithm in §6 is sketched at the structural level; a future revision may include a more detailed algorithmic treatment if implementer experience reveals ambiguities.

Changes from v0.10 to v0.11: conformance with Core Specification v0.26.0. **Terminology rename; no grammar-shape change.** The `Anchor` field is renamed `Id` (Core Spec v0.26.0, entry 60), completing the identity/naming split begun in v0.25.0 (in which the former descriptive `Id` field became `Name`). The `&` sigil is unchanged and continues to introduce the field and the id-mode reference literal. The Appendix A grammar production is updated (`Anchor`→`Id`, and its inherited content token to `IdToken`); the metadata block's keys keep the word *key* (a metadata key is not an id). Companion-version line updated to Core v0.26.0, Standard Grammar v0.14, and Path v0.8.

Changes from v0.1 to v0.2: conformance with Core Specification v0.9.10. Cross-references updated to current section numbering: §8.2 rule 1 (root rule) → §7.2 rule 1; §5.10 (REM Annotation) → §6.3.6; §4.2 (Resource Literals) → §5.2; §3.3.4 (Null Forms) → §5.1.3; §3.1, §4.1 (field order/sigils) → §5.1 and §5.1.1; "§4" in the intro (Standard Form definition) → §5. Example fixes: the §1.1 opening example, the §6.2 standard-to-compact example, and the §7.1 Simple Configuration example were updated to use an Object root (`O config`) rather than a bare Record (`R config`), which is non-conformant per §7.2 rule 1. The §7.4 example was updated from the legacy `TDR Customer` Template form to the current `DEFOBJ Customer`. The §7.6 Transport Wrappers example was updated from `PACKAGE`/`BLOCK` (legacy names) to `PKG`/`GRP` (current names); Block is a data-layer construct, while Group is the transport-layer partition. Companion-version line updated to GDX Core Specification v0.9.10 and GDX Standard Grammar v0.5.

Changes from v0.2 to v0.3: conformance with Core Specification v0.10.0 and Standard Grammar v0.6. (1) The Set marker `*` migrated from prefix to suffix throughout examples: `*S tags` becomes `S* tags`; `*R contacts` becomes `R* contacts`. (2) The `@` reference operator and `@KIND` reference elements are introduced; §4.1 adds an example showing `@OBJ employer : @com.acme` in Compact Form. (3) The `$` reference-literal sigil is reserved for future path-mode references; identifier-mode `@id` is normatively specified. (4) The Appendix A grammar reframes the Prefix production around the composition pattern `[@?][refinement-modifier?][base][*?]`, inheriting the sub-prefix vocabulary from the Standard Grammar. Cross-reference to REM Annotation updated to §6.5.6 (its new location in Core v0.10.0). Companion-version line updated to GDX Core Specification v0.10.0 and GDX Standard Grammar v0.6.

Changes from v0.8 to v0.9: conformance with Core Specification v0.23.0. Core v0.23.0 adds a per-element metadata channel (Core Spec §4.8): a brace-delimited block of flat `key = literal` pairs, written last on an element line (after the Type, before any children), carrying out-of-model carriage annotation. The Compact Form line grammar carries the block verbatim — the `{...}` sits at the tail of the element line exactly as in Standard Form; §3.3 gains a field-order bullet and §3.5 defines the block. Because metadata is out-of-model, it does not participate in the data-model round-trip contract (§6.3); both forms carry it, and a conversion that drops it remains model-faithful. There is no collision with the Compact-Form `#` line-comment feature: metadata is inline and brace-delimited, comments are whole-line. Companion-version lines updated (Core v0.23.0, Standard Grammar v0.12, Path v0.6).

Changes from v0.7 to v0.8: conformance with Core Specification v0.20.0. No grammar change. Core v0.20.0 makes the `#rel` field semantically optional on @-Edge elements: an edge line with no `#rel` is well-formed and denotes a relationship that exists but is undefined as to nature (Core Spec §6.4.2); the Rel field was already optional in this document's line grammar ("if present", §4). A bare `#` remains malformed; the empty Rel is expressed by omitting the field. An empty-rel example is added to §4.1. Core v0.19.0 (REF reframed as value-resolution) changes no syntax. Companion-version line updated to Core v0.20.0 and Standard Grammar v0.11.

Changes from v0.3 to v0.4: conformance with Core Specification v0.11.0, Standard Grammar v0.7, and the new GDX Path Expressions Specification v0.1. (1) The `$`-introduced path-mode reference literal and the `&`-introduced key-mode reference literal are now normatively specified by `GDX-Path.md`; the §4 prose updates to recognize five reference-literal forms (`@anchor`, `$path`, `&key`, `@anchor$path`, `@anchor&key`). The composed forms express cross-document addressing by path or by key. The `@anchor` production carries an optional role marker (`@OBJ:id`, `@REL:id`, `@DEFOBJ:id`, etc.) drawn from the closed prefix vocabulary; when omitted, composed references default to an Object anchor. (2) The identifier grammar narrows in line with Core Spec §4.4.1: identifiers may not begin with `/`. Examples in this document that used leading-slash identifiers are migrated accordingly. (3) Companion-version line updated to GDX Core Specification v0.11.0, GDX Standard Grammar v0.7, and GDX Path Expressions v0.1.


## Table of Contents

1. [Introduction](#1-introduction)
2. [Document Structure](#2-document-structure)
3. [Element Structure](#3-element-structure)
4. [Resource Literals](#4-resource-literals)
5. [Whitespace and Comments](#5-whitespace-and-comments)
6. [Mapping to Standard Form](#6-mapping-to-standard-form)
7. [Examples](#7-examples)
8. [Open Items](#8-open-items)
- [Appendix A — Complete Grammar](#appendix-a--complete-grammar)


---

## 1. Introduction

### 1.1 Purpose and Position

The GDX Standard Form (defined by the Core Specification §4 and the Standard Grammar) is bracket-delimited and explicit about every structural boundary. It is unambiguous and easy for machines to parse, but reading it as a human, especially in deeply nested documents, can be visually noisy.

The Compact Form trades bracket delimiters for indentation. The same data, expressed in Standard Form:

```
<O config
    <S databaseHost : "db.internal">
    <I databasePort : 5432>
    <L tlsEnabled : true>
>
```

becomes in Compact Form:

```
O config
  S databaseHost : "db.internal"
  I databasePort : 5432
  L tlsEnabled : true
```

The element kinds, identity fields, separators, and resource literals are unchanged. Only the structural delimiters differ: the opening `<` becomes the start of a line at the appropriate indent depth; the closing `>` becomes deindentation back to the parent's level.

The Compact Form exists for cases where humans author or read GDX directly: configuration files, hand-written test fixtures, vocabulary documents, and similar.

### 1.2 Relationship to Standard Form

The Compact Form preserves the Core Spec data model with no information loss. Every Compact Form document maps to a Standard Form document and vice versa. The two forms are equivalent at the data-model level.

What the Compact Form does NOT preserve:

- **Bracket-style whitespace flexibility.** Standard Form admits whitespace between fields; Compact Form requires fields on a single line.
- **Source-level comments.** File-level comments (§5.4) appear in Compact Form but are NOT preserved when converting to Standard Form. Comments that should travel with the data are expressed using REM Annotations (Core Spec §6.6), which work identically in both forms.

The Standard Form is the canonical wire and storage format. The Compact Form is for authoring. A typical pipeline:

1. A developer writes a configuration file in Compact Form.
2. The file is parsed into the data model.
3. The data model is serialized to Standard Form for storage, signing, or transport.
4. Other tools consume the Standard Form.
5. If a developer needs to inspect the data again, it can be re-serialized to Compact Form for readability.

Round-trip through this pipeline preserves the data but not source-level comments. This is a deliberate choice: comments that should survive are REMs.

### 1.3 What Compact Form Is and Is Not

The Compact Form is:

- A line-oriented serialization of GDX.
- A bidirectional mapping to/from Standard Form at the data-model level.
- Suitable for human authoring of GDX documents.
- Friendly for editing in text editors, version control diffs, and tooling that treats files as line sequences.

The Compact Form is NOT:

- A different data model. The Core Spec defines the model; Compact Form is one of several serializations of that model.
- A canonical form for storage or transport. Use Standard Form (or Binary Form, when defined) for those purposes.
- A subject of content addressing. Strict canonical form (Core Spec §A.5) is defined for Standard Form only; documents to be content-addressed are converted to Standard Form first.
- A different feature set. Every GDX feature works in Compact Form; the differences are syntactic only.

---

## 2. Document Structure

A Compact Form document consists of one or more top-level elements at zero indentation, separated by blank lines. The structural rules of the Core Spec apply: each top-level element MUST be one of Object, Template, REL, or Package (Core Spec §7.2 rule 1).

### 2.1 Top-Level Elements

A top-level element starts at column zero (no leading indentation). Its children, if any, appear on subsequent lines at greater indentation. The element ends when subsequent content returns to zero indentation (signaling the start of the next top-level element) or when end of file is reached.

```
O Alice
  S firstName : "Alice"
  S lastName : "Smith"

O Bob
  S firstName : "Bob"
  S lastName : "Jones"
```

The two Objects above are independent top-level elements separated by a blank line. The blank line is informative; conforming parsers SHOULD treat it as a visual separator without semantic meaning.

### 2.2 Indentation Model

Compact Form uses spaces only for indentation. Tab characters are NOT permitted as indentation; a tab character in indentation position is malformed.

The canonical indent unit is **two spaces per nesting level**. Conforming serializers producing Compact Form output use two-space indents.

Conforming parsers MAY accept any consistent indent unit (two, three, four spaces) so long as the document uses it consistently throughout. If a document mixes indent amounts (e.g., two-space at one level, four-space at another), the parser MUST reject the document as malformed.

The first nested element in a document determines the document's indent unit. Subsequent nested elements MUST use indent amounts that are multiples of that unit. If the first nested content uses three spaces, the parser expects three-space indents throughout; if it uses four spaces, the parser expects four-space indents.

Indentation depth corresponds to nesting depth. An element at indent depth 1 is a child of the most recently appearing element at indent depth 0. An element at indent depth 2 is a child of the most recently appearing element at indent depth 1. And so on.

### 2.3 Multiple Top-Level Elements

A Compact Form document MAY contain multiple top-level elements, each at zero indentation. The boundary between top-level elements is the return to zero indentation:

```
TDR Customer
  S firstName
  S lastName
  I age

REL gdx:knows
  _L gdx:symmetric : true

O alice --Customer
  S firstName : "Alice"
  S lastName : "Smith"
  I age : 24
  @OBJ #gdx:knows : @bob
```

This document contains a Template, a REL declaration, and an Object — three top-level elements, each independent. Blank lines between them are convention and improve readability but are not required.

### 2.4 Empty Lines

Empty lines (lines containing only whitespace, or no content) are ignored by the parser. They serve as visual separators and have no semantic effect. Empty lines may appear:

- Between top-level elements (recommended for readability)
- Between groups of children within an element (allowed but not required)
- At the start or end of the document (allowed)

Empty lines do NOT terminate elements. Element termination is determined by indentation depth, not by blank lines.

---

## 3. Element Structure

Each non-empty, non-comment line in a Compact Form document represents one element. The line carries the element's prefix, identity fields, separators, and resource literal — all on one line.

### 3.1 Single-Line Elements

A leaf element (one with no children) is a single line:

```
S name : "Alice"
I age : 24
L active : true
```

The line content uses the same lexical rules as the corresponding Standard Form element body, with the `<` and `>` brackets implicit at the line boundaries.

Field write order on a line is the same as in Standard Form: **Prefix · Id · Rel · Name · Resource · Type**.

```
S &1 #schema:givenName firstName : "Alice" --gdx:string
```

This is a string Property with key `1`, predicate `schema:givenName`, identifier `firstName`, value `"Alice"`, and Type reference `gdx:string`. The single-line form mirrors the Standard Form `<S &1 #schema:givenName firstName : "Alice" --gdx:string>`.

### 3.2 Elements with Children

A non-leaf element is one line for the parent followed by indented lines for the children:

```
R address
  S street : "Main St"
  I number : 100
  S city : "Oakland"
```

The parent line carries the prefix, identity fields, and (optionally) resource and type as usual. Children appear on subsequent lines at greater indentation depth.

The parent line MAY include a resource value that the children augment:

```
R address : --com.acme.Address
  S street : "Main St"
  I number : 100
```

Or the parent line MAY have no resource section (children carry the structure):

```
R address
  S street : "Main St"
  I number : 100
```

Both forms are valid; the choice depends on whether the parent itself has a typed value or not.

Nesting can go arbitrarily deep:

```
O Server
  S name : "web-01"
  R network
    S host : "10.0.1.42"
    I port : 8080
    R tls
      L enabled : true
      S certPath : "/etc/ssl/cert.pem"
```

Each indent level represents one step deeper in the tree. An element's children end when the next line is at the same indent as the element itself (a sibling) or shallower (a parent's continuation).

### 3.3 Field Order and Sigils

The field order, sigils, and lexical rules are identical to Standard Form (Core Spec §4.1 and §5.1.1):

- The prefix is first.
- The Id field, if present, follows the prefix and uses the `&` sigil.
- The Rel field, if present, follows the Id (or prefix) and uses the `#` sigil.
- The Name field, if present, follows the Rel (or prior fields) with no sigil.
- The Resource section, if present, follows the Name and uses `:` as separator.
- The Type field, if present, follows the Resource (or Name) and uses the `--` sigil.
- The metadata block, if present, is written last on the line — after the Type — as a brace-delimited `{...}` sequence (Core Spec §4.8; §3.5 below). It is out-of-model carriage annotation, not a model field.

All sigil tightness rules from the Standard Grammar (§8) apply: `&`, `#`, `--`, `@` MUST be immediately followed by their content with no intervening whitespace.

```
S &1 : "value"        # valid: tight sigils
S & 1 : "value"       # malformed: whitespace after &
S name --gdx:string   # valid: whitespace before --, none after
S name -- gdx:string  # malformed: whitespace after --
```

### 3.4 Closing Behavior

In Standard Form, an element is closed by the `>` bracket. In Compact Form, an element is closed when subsequent content deindents to the element's depth or shallower.

```
R outer
  S a : "one"
  R inner
    S x : "ex"
    S y : "wai"
  S b : "two"
```

In this example:
- `R outer` opens at depth 0.
- `S a` is a child of `R outer` at depth 1.
- `R inner` opens at depth 1, also a child of `R outer`.
- `S x` and `S y` are children of `R inner` at depth 2.
- `S b` returns to depth 1, which closes `R inner` and continues `R outer`'s child list.

The corresponding Standard Form:

```
<R outer
    <S a : "one">
    <R inner
        <S x : "ex">
        <S y : "wai">
    >
    <S b : "two">
>
```

Deindentation can close multiple nested elements at once:

```
R outer
  R inner
    R innermost
      S x : "deep"
S sibling : "value"
```

Here, `S sibling` at depth 0 closes `R innermost`, `R inner`, and `R outer` simultaneously. This is the Compact Form equivalent of three consecutive `>` brackets in Standard Form.

End of file closes all remaining open elements.

### 3.5 Metadata Block

An element line MAY end with a metadata block: a brace-delimited sequence of flat key/value pairs carrying out-of-model carriage annotation (Core Spec §4.8). It is written after the Type and before the newline; on an element with children, it stays on the parent's line (children follow on subsequent indented lines, as usual).

```
S name : "Bob" {sid = 455, pid = 6}
R address {sid = 9021}
  S street : "Main St"
  I number : 100 {pid = 6}
```

Keys are identifiers (CURIEs admitted for namespacing); values are Resource literals of the five value-kinds (no references); `=` separates a key from its value and `,` separates pairs. The grammar is identical to Standard Form (Core Spec §4.8.2); the Compact Form carries the block verbatim. An empty block `{}` is permitted and equivalent to no block.

Because the block is brace-delimited and inline, it does not collide with the whole-line `#` comment feature (§5.4). Metadata is out-of-model: a converter MAY carry, add, or drop it without affecting the data model (§6.3).


---

## 4. Resource Literals

### 4.1 Inline Literals

Inline resource literals follow the same lexical rules as Standard Form (Core Spec §4.2). String, integer, float, boolean, bytes, and reference literals all work identically:

```
S name : "Alice"
S empty : ""
I count : 42
I sized : 25i8
I hex : 0xFF
F price : 19.99d
F precise : 3.14159f64
L active : true
L compact : t
B hash : 0x4a3f8c2b
B encoded : 0zSGVsbG8sIHdvcmxkIQ==
@OBJ #knows : @bob
```

The lexical productions for each literal type are inherited from the Standard Grammar. Compact Form does not modify them.

**Reference literals.** Five forms are recognized in Resource position, all carried over from Standard Form:

- Identifier-mode: `@identifier` (Core Spec §4.2.6).
- Path-mode: `$/path/segments` (specified in `GDX-Path.md`).
- Id-mode: `&target-id` (specified in `GDX-Path.md`; Id uniqueness rules in Core Spec §4.4.5).
- Composed name + path: `@Base$/path/segments` for cross-document path addressing.
- Composed name + id: `@Base&target-id` for cross-document id addressing.

The `@Base` production may carry an optional role marker (`@OBJ:id`, `@REL:id`, `@DEFOBJ:id`, etc.) drawn from the closed prefix vocabulary; when omitted, composed references default to an Object base.

```
@OBJ #knows : @bob                       // @-edge, identifier-mode target
%REC mailing : $/people/alice/address    // REF, path-mode
%REC home : @people/bob$/contacts/[1]    // REF, composed name + path (OBJ default)
%REC shipping : &shippingAddress         // REF, id-mode (current document)
%REC origEvent : @system/log&entry-12345 // REF, composed name + id (OBJ default)
%STR desc : @REL:my-rel&description      // REF, composed name + id, REL base
%STR slotDoc : @DEFOBJ:com.acme.Customer&firstName-doc  // REF, Template base
```

@-edges (Core Spec §6.4) and REFs (Core Spec §6.16) appear in Compact Form with the prefix in the usual position:

```
O alice
  S name : "Alice"
  %DEFOBJ #conformsTo : &Customer
  @OBJ #knows : @bob
```

An @-edge MAY omit `#rel` entirely: the relationship exists by virtue of the role, but is undefined as to nature, pending later refinement (Core Spec §6.4.2). The empty Rel is expressed by omitting the field — there is no bare-`#` form:

```
O sensor-hub
  @OBJ &link-0192 : @device-7
```

### 4.2 Multi-Line String Literals

For long string content that does not fit comfortably on a single line, Compact Form supports multi-line string literals using a continuation marker.

A multi-line string is introduced by a vertical bar `|` after the `:` separator, in place of the inline string literal:

```
S description : |
  This is a multi-line string.
  It can contain any characters,
  including blank lines and special chars like " or \.
```

The string content begins on the line following the `|` marker. Each subsequent line at greater indent than the parent line is part of the string content. The first line of content establishes the string's base indentation; that base indent is stripped from all content lines.

The string content above resolves to:

```
This is a multi-line string.
It can contain any characters,
including blank lines and special chars like " or \.
```

The multi-line form preserves line breaks within the string. Trailing whitespace on each line is preserved as written; leading whitespace beyond the base indent is preserved.

Multi-line strings end when a line returns to the parent element's indent depth or shallower.

```
R document
  S title : "Annual Report"
  S body : |
    This is the document body.
    It spans multiple lines.

    Including blank lines like the one above.
  S footer : "End of report"
```

The `S footer` line, at the same indent as `S body`, ends the multi-line content for `S body`.

For strings with no trailing newline, the closing marker `|<` ends the string immediately at end of last content line:

```
S oneLine : |<
  No trailing newline on this one
S nextField : "value"
```

The default `|` form preserves a trailing newline (the natural behavior when reading lines until deindent). The `|<` form strips the trailing newline.

### 4.3 Multi-Line Bytes Literals

Bytes literals (hex or base64) on a single line work the same as in Standard Form:

```
B hash : 0xe3b0c44298fc1c149afbf4c8996fb92427ae41e4
```

For long byte sequences, base64 encoding can use a continuation marker analogous to multi-line strings, with `0z|` as the introducer:

```
B largeBlob : 0z|
  SGVsbG8sIHdvcmxkIQ==
  SGVsbG8sIHdvcmxkIQ==
  SGVsbG8sIHdvcmxkIQ==
```

Continuation lines contain only base64 characters. Whitespace between or within continuation lines is not part of the encoded data; it is removed before decoding.

The bytes literal ends when content returns to the parent's indent depth.

Hex literals (`0x` prefix) do not have a multi-line form in this revision; long hex sequences should use base64 (`0z`) instead, which is more compact.

### 4.4 Null Forms

The two null forms from Standard Form (Core Spec §4.1.3) work in Compact Form:

```
S middleName              # bare form (canonical)
S middleName :            # empty-value form (alternative)
```

Both forms denote a null Property. The bare form (no `:`) is canonical for serialization. Conforming serializers normalize the empty-value form to the bare form on output.

---

## 5. Whitespace and Comments

### 5.1 Indent Unit Determination

The indent unit is determined by the first nested content in the document. Conforming serializers producing Compact Form output use two spaces per indent level (the canonical unit).

```
O Alice
  S firstName : "Alice"   # indent unit established as 2 spaces
    S inner : "value"     # this would need to be 4 spaces (2 * 2 levels deep)
```

Within a document, the indent unit is consistent. Mixing two-space and four-space indents within the same document is malformed.

### 5.2 Tab vs Space

Tab characters MUST NOT appear in indentation positions. A tab character at the start of a non-empty line, before any non-whitespace content, is malformed.

Tab characters within a line (after non-whitespace content has appeared) are permitted as part of string literal content (where they are part of the string value) but discouraged elsewhere.

This restriction exists because tab interpretation varies across editors and tooling, leading to documents that look correct in one environment and broken in another. Spaces have unambiguous width.

### 5.3 Within-Line Whitespace

Within an element's line, whitespace rules follow Standard Form (§8 of the Standard Grammar):

- Sigils (`&`, `#`, `--`, `@`) MUST be immediately followed by their content.
- Sigils MUST be preceded by whitespace separating them from the previous token.
- The `:` separator MAY have whitespace on either side or none.
- Whitespace between adjacent identifier tokens is required.

```
S name : "Alice"          # canonical: spaces around :
S name:"Alice"            # valid: no spaces around :
S name :"Alice"           # valid: space before, none after
```

Conforming serializers producing canonical Compact Form output use single spaces around `:` and between adjacent fields.

### 5.4 Comments

Compact Form supports file-level line comments using the `#` character. A line whose first non-whitespace character is `#` is a comment line and is ignored by the parser.

```
# This file configures the production database connection.
# Last updated: 2026-04-29

S databaseHost : "db.internal"
S databasePort : 5432

# TLS settings — required for production
L tlsEnabled : true
S certPath : "/etc/ssl/cert.pem"
```

Comment lines may appear anywhere in a document — at zero indentation, between elements, or indented within nested element blocks. The indentation of a comment is informative but does not affect parsing; comments are not treated as elements and do not participate in the indent-depth tracking that determines element nesting.

Trailing comments on element lines are NOT supported. Comments may only appear on lines that contain nothing but the comment.

Comments are NOT part of the data model. They exist in the source file and are NOT preserved when:

- Converting Compact Form to Standard Form (comments are discarded)
- Converting Standard Form to Compact Form (no comments are introduced)
- Persisting the data through any storage layer

For documentation that should travel with the data, use REM Annotations (Core Spec §6.6), which work identically in both forms:

```
# This comment is editorial — for the developer's eyes only.
O Customer
  REM : "This comment travels with the data through round-trip."
  S firstName : "Alice"
```

The `#` character appears elsewhere in the grammar — it is the Rel sigil. Disambiguation: the Rel sigil `#` is followed immediately by an identifier (no whitespace) and appears within an element line, while a comment `#` appears as the first non-whitespace character on a line. The distinction is unambiguous because comment-position `#` and Rel-position `#` cannot occur in the same lexical context.

---

## 6. Mapping to Standard Form

The Compact Form and Standard Form are bidirectionally equivalent at the data-model level. This section describes the mapping in both directions.

### 6.1 Compact-to-Standard Conversion

Conversion from Compact Form to Standard Form follows this structural transformation:

1. Parse the Compact Form document into the data model. Indentation determines parent-child relationships; element prefixes, identity fields, separators, and resource literals follow the same rules as Standard Form.
2. Serialize the data model to Standard Form. Each element becomes a `<...>` bracketed element; nesting becomes child elements within parent brackets; comments are discarded; multi-line strings become single-line strings with `\n` escapes for line breaks.

The conversion preserves all data-model content: prefixes, identifiers, predicates, types, resources, child order, and Annotations (REMs). The conversion does NOT preserve comments or original whitespace formatting.

A simple example:

Compact:
```
O Alice
  S firstName : "Alice"
  R address
    S street : "Main St"
    I number : 100
```

Standard:
```
<O Alice
    <S firstName : "Alice">
    <R address
        <S street : "Main St">
        <I number : 100>
    >
>
```

A multi-line string example:

Compact:
```
S description : |
  Line one.
  Line two.
```

Standard:
```
<S description : "Line one.\nLine two.\n">
```

### 6.2 Standard-to-Compact Conversion

Conversion from Standard Form to Compact Form follows the inverse transformation:

1. Parse the Standard Form document into the data model.
2. Serialize the data model to Compact Form. Each element becomes a single line (or a parent line with indented children); nesting becomes indentation; multi-line string content may use the `|` continuation marker if the string contains newlines.

The conversion preserves all data-model content: prefixes, identifiers, predicates, types, resources, child order, and Annotations (REMs).

A simple example:

Standard:
```
<O config
    <S databaseHost : "db.internal">
    <I databasePort : 5432>
>
```

Compact:
```
O config
  S databaseHost : "db.internal"
  I databasePort : 5432
```

The conversion is deterministic: a given data-model state produces a single canonical Compact Form output. The canonical form uses two-space indentation, single spaces around `:` separators, lowercase boolean keywords, and the bare null form (`<S name>` not `<S name : >`).

### 6.3 Round-Trip Preservation

A document that round-trips Compact → Standard → Compact (or Standard → Compact → Standard) preserves the data model exactly. The properties preserved are the same as the Core Spec's round-trip contract (§A.2):

- Element prefixes
- Identity fields (Name, Id, Rel, Type)
- Resource value-kinds and values, including representation tags
- Child element order
- Annotations (REM elements)
- Null state (bare or empty-value form normalizes to bare)

The metadata channel (Core Spec §4.8) is carried verbatim by both forms, but it is out-of-model: it is not part of the data-model round-trip contract above, and a converter MAY add, alter, or drop it without affecting model-faithfulness. Carrying it is RECOMMENDED where downstream carriage depends on it.

What is NOT preserved through round-trip:

- File-level comments (`#` lines) — Compact-only feature, lost on conversion to Standard
- Original whitespace formatting — both forms produce canonical whitespace on output
- The choice of multi-line vs. single-line string representation — both forms produce canonical representation (single-line preferred unless content contains newlines)

### 6.4 Information NOT Preserved

The following items exist in Compact Form source files but are NOT part of the data model and are NOT preserved through any conversion:

- File-level line comments (`#` lines)
- Blank lines between elements
- Original indentation amount (canonical output uses two spaces)
- Choice of single-line vs. multi-line literal forms (canonical output picks based on content)

Items that are part of the data model and ARE preserved:

- All element content (prefixes, identifiers, resources, types)
- REM Annotations (Core Spec §6.6) — carry textual notes as data
- Child order
- Null state of properties

The distinction matters for editorial workflows: notes that should survive conversion belong in REMs; notes that are scaffolding for the source file belong in `#` comments.

---

## 7. Examples

This section presents Compact Form examples covering common patterns. Each example also shows the equivalent Standard Form for comparison.

### 7.1 Simple Configuration

Compact:
```
O config
  S databaseHost : "db.internal"
  I databasePort : 5432
  L tlsEnabled : true
  S logLevel : "info"
  F timeoutSeconds : 30.0
```

Standard:
```
<O config
    <S databaseHost : "db.internal">
    <I databasePort : 5432>
    <L tlsEnabled : true>
    <S logLevel : "info">
    <F timeoutSeconds : 30.0>
>
```

### 7.2 Nested Records

Compact:
```
O Server
  S name : "web-01"
  R network
    S host : "10.0.1.42"
    I port : 8080
    L tlsEnabled : true
  R limits
    I maxConnections : 1000
    F timeoutSeconds : 30.0
```

Standard:
```
<O Server
    <S name : "web-01">
    <R network
        <S host : "10.0.1.42">
        <I port : 8080>
        <L tlsEnabled : true>
    >
    <R limits
        <I maxConnections : 1000>
        <F timeoutSeconds : 30.0>
    >
>
```

### 7.3 Sets and Sequences

Compact:
```
O Post42
  S title : "Introducing GDX"
  S* tags
    S : "data-formats"
    S : "structured-data"
    S : "graph"
  R* contacts
    R alice
      S email : "alice@example.com"
    R bob
      S email : "bob@example.com"
```

Standard:
```
<O Post42
    <S title : "Introducing GDX">
    <S* tags
        <S : "data-formats">
        <S : "structured-data">
        <S : "graph">
    >
    <R* contacts
        <R alice
            <S email : "alice@example.com">
        >
        <R bob
            <S email : "bob@example.com">
        >
    >
>
```

### 7.4 Templates and RELs

Compact:
```
DEFOBJ Customer
  S firstName
    _L gdx:required : true
  S email
    _L gdx:required : true
    _S gdx:pattern : "^[^@]+@[^@]+$"
  I age
    _L gdx:optional : true

REL gdx:knows
  _L gdx:symmetric : true

O alice --Customer
  S firstName : "Alice"
  S email : "alice@example.com"
  I age : 24
  @OBJ #gdx:knows : @bob
```

Standard:
```
<DEFOBJ Customer
    <S firstName
        <_L gdx:required : true>
    >
    <S email
        <_L gdx:required : true>
        <_S gdx:pattern : "^[^@]+@[^@]+$">
    >
    <I age
        <_L gdx:optional : true>
    >
>

<REL gdx:knows
    <_L gdx:symmetric : true>
>

<O alice --Customer
    <S firstName : "Alice">
    <S email : "alice@example.com">
    <I age : 24>
    <@OBJ #gdx:knows : @bob>
>
```

### 7.5 Multi-Line Content

Compact:
```
O Article
  S title : "How GDX Works"
  S body : |
    GDX is a typed graph data format.
    It combines several properties typically
    split across separate formats.

    Each line of GDX carries explicit type information.
  S footer : "End of article"
  B headerImage : 0z|
    iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAQAAAC1HAwCAAAA
    C0lEQVR42mNkYAAAAAYAAjCB0C8AAAAASUVORK5CYII=
```

Standard:
```
<O Article
    <S title : "How GDX Works">
    <S body : "GDX is a typed graph data format.\nIt combines several properties typically\nsplit across separate formats.\n\nEach line of GDX carries explicit type information.\n">
    <S footer : "End of article">
    <B headerImage : 0ziVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAQAAAC1HAwCAAAAC0lEQVR42mNkYAAAAAYAAjCB0C8AAAAASUVORK5CYII=>
>
```

### 7.6 Transport Wrappers (Package and Group)

Compact:
```
PKG customer-export-2026-04-29
  S source : "crm-v3"
  S exportedBy : "jmussi"
  I count : 3

  GRP pantone
    S libraryName : "Pantone Solid Coated"
    S version : "2023"
    O Pantone-485-C
      F l : 47.2
      F a : 67.8
      F b : 54.1
    O Pantone-186-C
      F l : 41.5
      F a : 65.2
      F b : 32.0

  GRP ral
    S libraryName : "RAL Classic"
    O RAL-3020
      F l : 50.1
      F a : 65.0
      F b : 35.0
```

Standard:
```
<PKG customer-export-2026-04-29
    <S source : "crm-v3">
    <S exportedBy : "jmussi">
    <I count : 3>
    <GRP pantone
        <S libraryName : "Pantone Solid Coated">
        <S version : "2023">
        <O Pantone-485-C
            <F l : 47.2>
            <F a : 67.8>
            <F b : 54.1>
        >
        <O Pantone-186-C
            <F l : 41.5>
            <F a : 65.2>
            <F b : 32.0>
        >
    >
    <GRP ral
        <S libraryName : "RAL Classic">
        <O RAL-3020
            <F l : 50.1>
            <F a : 65.0>
            <F b : 35.0>
        >
    >
>
```

The Compact Form's readability advantage compounds in deeply nested transport wrappers: the bracket noise of nested Packages and Groups is replaced by indentation that conveys structure visually.

---

## 8. Open Items

This grammar specification has the following open items that should be resolved before its v1.0 publication.

### 8.1 Trailing Comments

The current grammar disallows trailing comments on element lines (comments must occupy their own line). This is simpler to specify and parse but loses a convenience YAML and TOML provide:

```
S databaseHost : "db.internal"  # production server
```

Resolution: trailing comments are deferred. If implementer feedback indicates strong demand and the disambiguation rules can be made unambiguous, a future revision may add them.

### 8.2 Folded Multi-Line Strings

The current grammar supports literal multi-line strings (preserving line breaks via `|`) but not folded multi-line strings (joining lines with spaces, like YAML's `>`). Folded form is useful for long prose that should display as one paragraph in consumers.

Resolution: deferred to v0.2 or later. Adding the `>` continuation marker is straightforward but requires defining the folding rules precisely.

### 8.3 String Indentation Within Multi-Line Content

The current grammar describes base indentation stripping for multi-line strings but does not formalize edge cases (blank lines within content, content lines at unexpected depths). A future revision should formalize the algorithm.

### 8.4 Document-Level Pragmas

Some implementations may want document-level metadata (target indent unit, vocabulary version, character encoding) declared at the document head. The current grammar has no provision for this; declarations of this kind would need to be added as a header construct.

Resolution: deferred. If experience reveals genuine need, a future revision may add a header convention. The principle to preserve: any header content must NOT affect the data model — it's purely for parser configuration.

---

## Appendix A — Complete Grammar

This appendix consolidates the productions defined above into a single grammar. Productions are listed in dependency order; later productions may reference earlier ones.

```ebnf
// ===== Lexical Primitives =====

ALPHA           ::= [A-Z] | [a-z]
DIGIT           ::= [0-9]
SPACE           ::= " "
NEWLINE         ::= "\n" | "\r\n" | "\r"

// Tab characters are NOT permitted in indentation positions.

// ===== Document Structure =====

Document        ::= (BlankLine | CommentLine | TopLevelElement)*

TopLevelElement ::= Element              // at zero indentation

BlankLine       ::= SPACE* NEWLINE

CommentLine     ::= SPACE* "#" CommentContent NEWLINE
CommentContent  ::= ~NEWLINE*            // any characters except newline


// ===== Element Structure =====

// An Element occupies one or more consecutive lines:
// - The element line (prefix, identity, resource, type)
// - Zero or more child lines at greater indentation
// - Multi-line literal continuations, if applicable

Element         ::= ElementLine ChildBlock?

ElementLine     ::= IndentSpaces ElementBody NEWLINE

ChildBlock      ::= (Element | MultiLineContinuation)+
                  // children at indent depth greater than parent

ElementBody     ::= Prefix
                    (SPACE Id)?
                    (SPACE Rel)?
                    (SPACE Name)?
                    (ResourceSection)?

ResourceSection ::= SPACE? ":" SPACE? Resource? (SPACE "--" Type)?
                  | SPACE "--" Type
                  // Resource? makes value side optional after ':'.
                  // The forms `S name :` and `S name` are both null.


// ===== Indentation =====

IndentSpaces    ::= SPACE*               // depth determines parent-child

// The indent unit is determined by the first nested element in the document.
// All subsequent indentation MUST be a multiple of the established unit.
// Canonical form uses two spaces per level.


// ===== Fields =====

// Field productions are inherited from the Standard Grammar (§4).
// All sigil-tightness rules apply.

Prefix          ::= ReferencePrefix? ModifierBase SetSuffix?

ReferencePrefix ::= "@"
SetSuffix       ::= "*"

// ModifierBase and the sub-prefix vocabulary (ObjectPrefix, RecordPrefix,
// PropertyPrefix, BlockPrefix, RefinementBase including
// QualifierComposition/ExtensionComposition/NodeComposition, ValPrefix,
// TemplatePrefix, RelPrefix, AnnotationPrefix, PackagePrefix, GroupPrefix)
// are inherited from the Standard Grammar (Appendix A, §7.1).
// The EDGE/E prefix is removed in Core v0.12.0; @-Edge elements are
// produced by the @ operator composing with ObjectPrefix or RelPrefix.
//
// The composition pattern is the same in Compact Form as in Standard Form;
// only the surrounding delimiters and indentation rules differ.

Id             ::= "&" IdToken
Rel             ::= "#" RelContent
Name              ::= NameStartChar NameContChar*
Type            ::= "--" TypeContent

// IdToken, RelContent, TypeContent, NameStartChar, NameContChar
// are inherited from the Standard Grammar.


// ===== Resource Literals =====

Resource        ::= StringLiteral
                  | IntLiteral
                  | FloatLiteral
                  | BoolLiteral
                  | BytesLiteral
                  | ReferenceLiteral
                  | MultiLineString
                  | MultiLineBytes

// Inline literal productions inherited from Standard Grammar (§5).

MultiLineString ::= "|" Trailing? NEWLINE
                    StringContentBlock

StringContentBlock ::= StringContentLine+
                      // each line indented deeper than parent
                      // base indent stripped from content

StringContentLine ::= IndentSpaces ~NEWLINE* NEWLINE

Trailing        ::= "<"                  // optional: strip trailing newline

MultiLineBytes  ::= "0z|" NEWLINE
                    BytesContentBlock

BytesContentBlock ::= BytesContentLine+

BytesContentLine ::= IndentSpaces Base64Char+ NEWLINE


// ===== Top-Level Element Restriction =====

// Per Core Spec §7.2 rule 1, the prefix of a TopLevelElement
// MUST be one of: ObjectPrefix, TemplatePrefix, RelPrefix,
// PackagePrefix.
```

End of grammar.
