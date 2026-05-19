
## Appendix A — Conformance

This appendix defines what it means for a serialization, a parser, and a document to be conforming to this specification. Conformance is layered: a document may be representation-conforming without being schema-conforming, and an implementation may support one or both layers.

### A.1 Conformance Layers

Two distinct conformance layers are defined, corresponding to the layers introduced in §2.2:

**Representation Conformance (Layer 1)** addresses how data is encoded. It covers the structural rules of §4 through §8: prefix vocabulary, identifier grammar, literal forms, composition rules, and resource value-kind matching. Representation conformance is decidable at parse time without external context.

**Schema Conformance (Layer 2)** addresses what data is allowed to mean. It covers the Template-based validation of §8: Template satisfaction, Extension-defined constraints, and vocabulary semantics. Schema conformance is decidable after parsing and requires Template resolution.

A document is **fully conforming** if it is both representation-conforming and schema-conforming with respect to all Templates referenced via `--type`. A document with no Template references is fully conforming if it is representation-conforming.

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
12. Prefixes appear in **canonical form** — three-letter (or longer where natural) names in uppercase: `STR`, `INT`, `BOOL`, `FLOAT`, `BYTE`, `REC`, `OBJ`, `EDGE`, `GRP`, `*STR`, `*INT`, `.STR`, `_STR`, `%STR`, `%REC`, `%OBJ`, `NSTR`, `NINT`, `NBOOL`, `NFLOAT`, `NBYTE`, `NREC`, `DEFSTR`, `DEFINT`, `DEFBOOL`, `DEFFLOAT`, `DEFBYTE`, `DEFREC`, `DEFOBJ`, `REL`, `REM`, `PKG`, `BLK`. The minified single-letter forms (`S`, `I`, `L`, `F`, `B`, `R`, `O`, `E`, `G`, `*S`, `.S`, `_S`, `%S`, `NS`, etc.) are valid GDX but do not produce strict canonical bytes; conversion to canonical form is required before content addressing.

Two strict-canonical-form documents that represent the same data model produce byte-identical serializations. This property allows the strict canonical form to serve as the input to content-addressing functions.

The strict canonical form is OPTIONAL. Implementations not requiring byte-stability may produce any representation-conforming serialization, including the minified form for transport efficiency.

**Minified form output.** Implementations MAY produce minified form output for transport-efficiency contexts. The minified form replaces canonical prefix names with their single-letter equivalents where they exist (`STR` → `S`, `*STR` → `*S`, `NSTR` → `NS`, etc.). Prefixes without minified equivalents (`REL`, `REM`, `PKG`, `BLK`, `DEFSTR`, `DEFREC`, etc.) are emitted unchanged. Minified output produces a different byte sequence than canonical output for the same data; content addresses computed against minified output are NOT comparable to addresses computed against canonical output.

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
- Reference resolution within a single document: positive tests where targets exist; tests for unresolved references (which produce Layer 2 errors per §6.3.4.1, not Layer 1).
- Round-trip preservation: a document parsed and serialized to canonical form, then re-parsed, produces an identical data model.

**Schema validation tests (Layer 2):**

- Template conformance: instances claiming `--type` references that resolve and satisfy / fail the Template's constraints.
- Resolution order: tests confirming the §8.5.2 default order is followed (local document first, then loaded vocabularies, then global IRI).
- Resolution failure: tests confirming unresolved type references produce Layer 2 errors per §8.5.3.
- Validator modes: strict and lenient mode behavior on the same input documents.
- Error accumulation: validator returns multiple errors from a single pass (per §8.5.4).
- Edge reference resolution: tests confirming §6.3.4.1 behavior for resolved and unresolved Edge `@` references.

**Canonical form tests:**

- Documents in non-canonical form serialize to byte-identical canonical form per Appendix A.5.
- Two structurally-equivalent documents produce the same canonical form.
- Boolean normalization (`t`/`f` and uppercase forms normalize to lowercase spelled-out).
- Hex-bytes case normalization (uppercase normalizes to lowercase).
- Null form normalization (empty-value form normalizes to bare form).

**Resilience and security tests:**

- Documents at the recommended depth limit (positive); documents exceeding it (rejected per §C.1).
- Documents at the recommended element-count limit (positive); documents exceeding it (rejected).
- Numeric literals at maximum length (positive); numeric literals exceeding it (rejected per §C.1).
- Reference cycles in Edge graphs: cycles are valid data; tests confirm parsers accept them and traversal-based operations terminate.

**Compact Form tests (when supported):**

- Equivalence: every Standard Form test document, converted to Compact Form, should parse to the same data model.
- Indentation: documents with various consistent indent units; rejection of mixed indents.
- Multi-line strings (`|` and `|<` forms).
- Round-trip: Compact → Standard → Compact (data-model preservation; comment loss is expected per Compact Form spec).

The test suite is exhaustive in concept and bounded in practice. Each category above expands to dozens of specific tests when fully populated. A conformance test suite covering these categories is a substantial deliverable; implementations may bootstrap from a subset and expand coverage over time. The intent is that an implementation passing the full suite has demonstrably correct behavior on the cases that matter for interoperability.

---
