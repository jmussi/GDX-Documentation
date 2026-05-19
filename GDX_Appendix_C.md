
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

## Appendix B — Format Comparison

This appendix is informative. It provides side-by-side comparisons between GDX and other widely-used data formats to illustrate GDX's design properties and to serve as conversion guidance for readers familiar with those formats.

### B.1 Purpose and Scope

The comparisons in this appendix are intended to illustrate, not to argue superiority. Each format compared has strengths GDX does not match (JSON's universality, YAML's hand-edit friendliness, XML's mixed-content support, RDF's inference ecosystem). The point of comparison is to make GDX's distinguishing properties visible by contrast and to help readers familiar with other formats see how their data would translate.

The comparisons are at a moment in time. Format ecosystems evolve; new variants emerge. The specific syntactic forms shown here reflect widely-used conventions at the time of this draft. Where a format has multiple variants (JSON Schema versions, RDF serializations), one representative variant is chosen and noted.

Each comparison follows the same structure: a brief problem statement, the data shown in each relevant format, and a brief discussion of what each form captures and what it leaves implicit.

### B.2 Typed Composite Record

**Problem:** Express a customer with a name, age, contact email, active status, and a content hash for record integrity. The name field aligns with a shared semantic vocabulary (`schema:givenName`).

**GDX:**

```
<REC customer
    <STR #schema:givenName firstName : "Alice">
    <INT age : 24>
    <STR email : "alice@example.com">
    <BOOL active : true>
    <BYTE contentHash : 0xe3b0c44298fc1c149afbf4c8996fb92427ae41e4>
>
```

**JSON:**

```json
{
  "customer": {
    "firstName": "Alice",
    "age": 24,
    "email": "alice@example.com",
    "active": true,
    "contentHash": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4"
  }
}
```

**YAML:**

```yaml
customer:
  firstName: Alice
  age: 24
  email: alice@example.com
  active: true
  contentHash: e3b0c44298fc1c149afbf4c8996fb92427ae41e4
```

**XML:**

```xml
<customer>
  <firstName>Alice</firstName>
  <age>24</age>
  <email>alice@example.com</email>
  <active>true</active>
  <contentHash>e3b0c44298fc1c149afbf4c8996fb92427ae41e4</contentHash>
</customer>
```

**Discussion:**

JSON, YAML, and XML each express the same nested shape but leave several things implicit:

- **Numeric typing.** JSON's `24` is a number — it could be intended as an integer or a float; the format does not distinguish. YAML similarly: `24` is a number; `24.0` would be a float, but the format infers from the literal's appearance. XML's `<age>24</age>` is a string by default; consumers must know to interpret it as an integer.
- **Boolean typing.** JSON has native `true`/`false`. YAML has them too but with edge cases (`yes`, `no`, `on`, `off`, `Y`, `N` may also parse as booleans depending on YAML version, which causes the well-known "Norway problem" where the country code `NO` parses as the boolean `false`). XML's `<active>true</active>` is a string; the consumer interprets.
- **Bytes typing.** None of these formats has a native bytes type. The hex hash is encoded as a string; the consumer must know to decode it.
- **Predicate alignment.** The fact that `firstName` corresponds to `schema:givenName` is not expressible in JSON, YAML, or XML without adding a separate convention (JSON-LD's `@context`, XML namespaces, or external documentation).

GDX makes each of these explicit on the element itself. The receiver does not need an external schema to know that `age` is an integer, `active` is a boolean, `contentHash` is bytes, or that `firstName` maps to `schema:givenName`.

The size cost is moderate: GDX is slightly more verbose per element (the prefix and predicate add characters) but carries more information per element. For applications where types and semantic alignment matter, the per-element overhead is repaid by not requiring external schemas or documentation.

### B.3 Predicate-Aligned Data

**Problem:** Two systems exchange customer data. System A uses `firstName`/`lastName`; System B uses `givenName`/`familyName`. Both want to recognize the data as the same concept.

**GDX:**

```
<REC customer
    <STR #schema:givenName firstName : "Alice">
    <STR #schema:familyName lastName : "Smith">
>
```

**JSON-LD:**

```json
{
  "@context": {
    "firstName": "http://schema.org/givenName",
    "lastName": "http://schema.org/familyName"
  },
  "firstName": "Alice",
  "lastName": "Smith"
}
```

**RDF/Turtle:**

```turtle
@prefix schema: <http://schema.org/> .

_:customer schema:givenName "Alice" ;
           schema:familyName "Smith" .
```

**Plain JSON (with informal convention):**

```json
{
  "firstName": "Alice",
  "lastName": "Smith"
}
```

(Plain JSON has no built-in mechanism for predicate alignment; the receiver must know by convention that `firstName` corresponds to schema.org's `givenName`.)

**Discussion:**

The three formats with built-in alignment mechanisms (GDX, JSON-LD, RDF/Turtle) all express the same semantic content but in different shapes:

- **GDX** carries the predicate inline on each property: `#schema:givenName` is a field of the property element. The local name (`firstName`) and the shared predicate are both visible at the point of the property, and the data can also be read without consulting the predicate.
- **JSON-LD** separates the alignment into a `@context` mapping. The properties themselves use only local names; the `@context` provides the translation. This is compact when many properties share alignment but requires reading the context to know each property's predicate.
- **RDF/Turtle** uses the predicate directly as the property identifier (`schema:givenName`). There is no separate "local name"; the predicate IS the name. This is the most compact for the alignment case but loses the local naming convenience that systems often want internally.

For systems that have established local naming conventions and want to add semantic alignment without renaming everything, GDX's inline predicate field is a natural fit — the local name stays local, and the predicate provides the shared meaning. JSON-LD achieves the same goal with a different shape; RDF/Turtle replaces local names with predicates entirely.

### B.4 Schema-Validated Data

**Problem:** A customer record must satisfy a defined schema: certain fields are required, the email field must match a pattern, and the age must be a non-negative integer.

**GDX (schema and instance in one document):**

```
<DEFREC Customer
    <STR firstName
        <_BOOL required : true>
    >
    <STR email
        <_BOOL required : true>
        <_STR pattern : "^[^@]+@[^@]+$">
    >
    <INT age
        <_BOOL required : true>
        <_INT min : 0>
    >
>

<REC alice --Customer
    <STR firstName : "Alice">
    <STR email : "alice@example.com">
    <INT age : 24>
>
```

**JSON Schema (separate schema document):**

`customer-schema.json`:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "required": ["firstName", "email", "age"],
  "properties": {
    "firstName": {"type": "string"},
    "email": {
      "type": "string",
      "pattern": "^[^@]+@[^@]+$"
    },
    "age": {
      "type": "integer",
      "minimum": 0
    }
  }
}
```

`alice.json`:

```json
{
  "firstName": "Alice",
  "email": "alice@example.com",
  "age": 24
}
```

**XML Schema (separate XSD document):**

`customer.xsd`:

```xml
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema">
  <xs:element name="customer">
    <xs:complexType>
      <xs:sequence>
        <xs:element name="firstName" type="xs:string"/>
        <xs:element name="email">
          <xs:simpleType>
            <xs:restriction base="xs:string">
              <xs:pattern value="^[^@]+@[^@]+$"/>
            </xs:restriction>
          </xs:simpleType>
        </xs:element>
        <xs:element name="age">
          <xs:simpleType>
            <xs:restriction base="xs:integer">
              <xs:minInclusive value="0"/>
            </xs:restriction>
          </xs:simpleType>
        </xs:element>
      </xs:sequence>
    </xs:complexType>
  </xs:element>
</xs:schema>
```

`alice.xml`:

```xml
<customer>
  <firstName>Alice</firstName>
  <email>alice@example.com</email>
  <age>24</age>
</customer>
```

**Discussion:**

The schema-and-instance pattern is where GDX's "schema is data" property becomes most visible. JSON and XML both require a separate schema language (JSON Schema, XSD) with its own grammar, vocabulary, and tooling. The schema lives in a separate document; tools that consume the data may or may not consume the schema; validation requires resolving the schema reference and applying a schema-aware validator.

GDX uses the same syntax for the Template (the schema) as for the instance. The Template is itself a GDX element with the same composition rules. Tools that read GDX read Templates the same way they read instances. There is no separate schema language to learn, no separate format for schema documents, and the schema can travel in the same document as the instances if the application chooses.

The trade-off is real: dedicated schema languages (JSON Schema, XSD) have more expressive validation vocabularies than GDX's core extension set. The GDX Common Vocabulary provides common validators (pattern, min, max, required), but specialized validation needs may require either application-defined extensions or external validation logic. JSON Schema has had years of community refinement and supports complex constraints (oneOf, allOf, conditional schemas) that GDX does not natively express.

For applications where schema and data should share a format and travel together, GDX is a natural fit. For applications already invested in JSON Schema or XSD tooling, those remain appropriate.

### B.5 Graph-Structured Data

**Problem:** Express a small graph of three people with directed `knows` relationships, where the relationships have attributes (the date the connection was made).

**GDX:**

```
<OBJ alice
    <STR name : "Alice">
    <EDGE #knows : @bob
        <STR since : "2024-01-15">
    >
    <EDGE #knows : @carol
        <STR since : "2024-03-22">
    >
>

<OBJ bob
    <STR name : "Bob">
    <EDGE #knows : @carol
        <STR since : "2024-02-08">
    >
>

<OBJ carol
    <STR name : "Carol">
>
```

**RDF/Turtle:**

```turtle
@prefix : <http://example.org/> .

:alice :knows :bob, :carol ;
       :name "Alice" .

:bob :knows :carol ;
     :name "Bob" .

:carol :name "Carol" .
```

(RDF in its basic form doesn't express edge attributes naturally. The `since` dates would require either reification, RDF-star, or a separate "knows event" entity for each relationship.)

**RDF-star (RDF 1.2):**

```turtle
@prefix : <http://example.org/> .

:alice :name "Alice" .
:bob :name "Bob" .
:carol :name "Carol" .

<< :alice :knows :bob >> :since "2024-01-15" .
<< :alice :knows :carol >> :since "2024-03-22" .
<< :bob :knows :carol >> :since "2024-02-08" .
```

**Property Graph (Cypher-style, illustrative):**

```cypher
CREATE (alice:Person {name: 'Alice'})
CREATE (bob:Person {name: 'Bob'})
CREATE (carol:Person {name: 'Carol'})
CREATE (alice)-[:KNOWS {since: '2024-01-15'}]->(bob)
CREATE (alice)-[:KNOWS {since: '2024-03-22'}]->(carol)
CREATE (bob)-[:KNOWS {since: '2024-02-08'}]->(carol)
```

**JSON (with embedded references by convention):**

```json
{
  "people": [
    {
      "id": "alice",
      "name": "Alice",
      "knows": [
        {"target": "bob", "since": "2024-01-15"},
        {"target": "carol", "since": "2024-03-22"}
      ]
    },
    {
      "id": "bob",
      "name": "Bob",
      "knows": [
        {"target": "carol", "since": "2024-02-08"}
      ]
    },
    {
      "id": "carol",
      "name": "Carol"
    }
  ]
}
```

**Discussion:**

Graph data exposes a real format-design tension: how do you express edges, particularly edges with attributes? The four formats above each handle it differently:

- **GDX** uses Edge elements (`EDGE`/`E`) as children of the source Object. The Edge's `#rel` field names the predicate (`knows`), the Resource is a reference to the target (`@bob`), and the Edge's children carry the edge attributes (`since`). The pattern naturally extends to multiple edges per source and edges with rich attribute structure. GDX is essentially a property-graph format in this dimension.
- **RDF basic** expresses simple subject-predicate-object triples cleanly but lacks a native way to attach attributes to a triple. The historical workarounds (reification, n-ary relations) are verbose and rarely used in practice.
- **RDF-star** adds the `<< s p o >>` syntax to address this, allowing properties on triples directly. This is now standardized as RDF 1.2 and mirrors the property-graph model.
- **Property graph databases** (Neo4j, Neptune, etc.) treat edge attributes as first-class. GDX's Edge model is essentially the same shape; GDX adds the document-friendly serialization that property graph databases typically lack.
- **JSON** has no native graph notion. Embedded references (`{"target": "bob"}`) are convention; the consumer must know to traverse them. References across documents require additional convention (JSON Pointer, URI fragments, etc.). For small graphs this works; for large graphs it becomes unwieldy.

GDX occupies a useful position in this space: it expresses property-graph semantics natively (edges with attributes), supports graph traversal through reference resolution, and serializes as a document-friendly text format. For systems that want graph semantics without committing to a graph database stack, GDX provides a natural representation.

### B.6 Summary

The following table summarizes what each format expresses natively and what it leaves to convention or external schema.

| Property                    | GDX | JSON | YAML | XML | JSON-LD | RDF/Turtle |
|-----------------------------|:---:|:----:|:----:|:---:|:-------:|:----------:|
| Explicit primitive types    |  ✓  |  ◐   |  ◐   |  ✗  |    ◐    |     ✓      |
| Native bytes type           |  ✓  |  ✗   |  ✗   |  ✗  |    ✗    |     ◐      |
| Predicate alignment         |  ✓  |  ✗   |  ✗   |  ◐  |    ✓    |     ✓      |
| Schema in same format       |  ✓  |  ✗   |  ✗   |  ✗  |    ✗    |     ◐      |
| Graph relationships         |  ✓  |  ◐   |  ◐   |  ◐  |    ✓    |     ✓      |
| Edge attributes             |  ✓  |  ◐   |  ◐   |  ◐  |    ✗    |     ◐      |
| Round-trip stability        |  ✓  |  ◐   |  ✗   |  ◐  |    ◐    |     ◐      |
| Single document for instance + schema |  ✓  |  ✗   |  ✗   |  ✗  |    ✗    |     ◐      |
| Universally supported       |  ✗  |  ✓   |  ✓   |  ✓  |    ◐    |     ◐      |
| Friendly for hand-editing   |  ◐  |  ◐   |  ✓   |  ✗  |    ◐    |     ◐      |
| Compact wire form           |  ◐  |  ✓   |  ✓   |  ✗  |    ◐    |     ◐      |

Legend: ✓ native support, ◐ partial or convention-based, ✗ not supported.

The table is at a moment in time and reflects the most widely-used variants of each format. Some entries (JSON-LD predicate alignment, RDF-star edge attributes) reflect format extensions that may not be supported by all tooling.

GDX is not universally better; it is differently positioned. The "✓" column for GDX reflects a deliberate combination of features designed for a specific class of applications. The "✗" entries (universally supported, hand-edit friendliness in some contexts) are real costs that may or may not matter depending on use case. The "Where GDX Fits" discussion (§1.5) covers the application classes where the trade-offs favor GDX.

---

## Appendix C — Security Considerations

This appendix discusses security risks associated with implementing GDX parsers, validators, and resolvers, along with recommended mitigations. It is informative; specific limits and policies are implementation choices.

### C.1 Parser Resource Exhaustion

GDX is a tree-structured format with potentially deep nesting. A maliciously-constructed document with extreme nesting depth can exhaust a parser's stack (in recursive-descent parsers) or memory (in tree-building parsers). The same risk applies to documents with very large numbers of sibling elements at a single level.

Mitigations:

- Implementations SHOULD impose a maximum nesting depth limit. A reasonable default is 100 levels, which accommodates legitimate deeply-nested documents while preventing pathological inputs. The limit SHOULD be configurable.
- Implementations SHOULD impose a maximum total element count per document. A reasonable default is 1 million elements.
- Streaming parsers (those that emit events without building a tree) avoid the memory exhaustion vector but should still impose depth limits to avoid stack exhaustion.
- Implementations SHOULD report a clear error when limits are exceeded, identifying the location where the limit was reached.

Numeric literal bounds present a related vector. A bigint literal with an extremely large digit count (e.g., 1 million decimal digits) requires proportional memory to parse and store; high-precision decimal literals (e.g., a `decimal128` declared with hundreds of significant digits) require comparable resources. Even single such literals can exhaust memory on constrained systems.

Mitigations for numeric literals:

- Implementations SHOULD impose a maximum length on numeric literal text. A reasonable default is 1024 characters per literal, which accommodates any practical fixed-width or decimal value while rejecting pathological inputs.
- Implementations SHOULD validate that integer literals carrying width suffixes (`i8`, `u32`, etc.) fall within the declared width's range at parse time, before allocating storage for the value.
- Implementations MAY further restrict bigint and arbitrary-precision decimal literals to application-appropriate maximum lengths, since these types are most often abused as a vector for resource exhaustion.

These limits exist to prevent denial-of-service attacks against parsers processing untrusted input. Documents from trusted sources may not need limits enforced.

### C.2 Reference Cycle Exploitation

GDX permits Edge references between Objects, allowing arbitrary graph structures including cycles. Cycles are valid data and MUST be supported by parsers and storage layers. The security risk arises when adversarial documents use cycles to attack consumers that traverse references without cycle detection — for example, a service that loads referenced Objects to display them, or that walks Edge networks to compute derived data.

Attack vectors:

- **Resource exhaustion via traversal.** A document with a cyclic reference graph causes naive traversal to loop indefinitely, consuming CPU and memory until limits are hit or the process is killed.
- **Amplification via deep chains.** Long reference chains that eventually cycle can multiply the cost of traversal-based operations, making moderate-size documents trigger expensive processing.

Mitigations:

- Reference traversal implementations MUST track visited Objects and avoid re-traversing them within a single traversal operation.
- Implementations SHOULD impose a maximum reference traversal depth as a defense-in-depth measure even when cycle detection is implemented.
- Resolvers SHOULD detect cycles in Template chains during effective-shape computation (though Template inheritance is not part of GDX, future Common Vocabulary or application Templates may reference each other; cycles in such reference chains should be detected).

### C.3 Embedded Reference Expansion

Embedded values (`%STR`/`%S`, `%REC`/`%R`, etc., per §6.3.11 of this specification) inline content from referenced sources at parse time. A document that contains many Embedded references, each resolving to content with further Embedded references, can cause exponential expansion analogous to the "billion laughs" attack against XML.

Mitigations:

- Implementations supporting Embedded references SHOULD impose a maximum expansion depth (suggested default: 20 levels).
- Implementations SHOULD impose a maximum total expanded size (suggested default: 100x the original document size, or an absolute byte limit).
- Implementations SHOULD detect cycles in Embedded reference chains and reject documents that would expand infinitely.
- For untrusted input, implementations MAY disable Embedded reference expansion entirely, treating Embedded references as opaque data.

### C.4 Network Resolution

The default Type and Edge reference resolution orders (§8.5.2 and §6.3.4.1) include "globally-resolvable IRIs" as a fallback resolution scope. Implementations that resolve such references over the network expose themselves to several risks:

- **Information disclosure.** Network requests to URLs in untrusted documents can leak information about the processing system to attacker-controlled servers (timing, request patterns, internal IP addresses).
- **Server-side request forgery (SSRF).** Documents containing IRIs targeting internal network addresses can use the parser as a proxy to reach systems otherwise inaccessible to the attacker.
- **Resource exhaustion.** Slow-responding URLs can stall parsing; large responses can exhaust memory.
- **Trust escalation.** Content fetched from a URL is not necessarily trustworthy; using it as a Template or Object source may grant unwarranted authority.

Mitigations:

- Implementations SHOULD disable network resolution by default for untrusted input; require explicit opt-in.
- When network resolution is enabled, implementations SHOULD restrict requests to allowlisted domains or networks.
- Implementations SHOULD impose timeouts and response size limits on fetches.
- Implementations SHOULD validate fetched content against the same conformance requirements as locally-parsed content; fetched content carries no special trust.
- Implementations MAY cache fetched content to reduce repeated requests; cache policies should consider both performance and freshness needs.

### C.5 Template Validation Behavior

Validators that resolve and apply Templates from untrusted vocabularies face risks:

- **Malicious constraints.** A Template defining a regex pattern can include patterns vulnerable to ReDoS (regular expression denial of service) — exponential-time backtracking on certain inputs. Validators applying such patterns expose themselves to DoS.
- **Resource exhaustion via constraint evaluation.** Some constraint extensions may permit computationally expensive evaluation; an adversarial Template could combine many such extensions.

Mitigations:

- Regex-evaluating validators SHOULD use regex engines with linear-time guarantees (e.g., RE2, Hyperscan) when processing Templates from untrusted sources.
- Implementations SHOULD impose per-element validation time limits for untrusted input.
- Implementations MAY restrict which constraint extensions are honored when processing untrusted Templates.

### C.6 Content Addressing and Signing

Content addressing of GDX documents requires the strict canonical form (Appendix A.5). Signatures over content addresses derive their security from both the canonical-form algorithm and the underlying hash function.

Considerations:

- Content addresses are valid only over the strict canonical form. Any non-canonical input must be normalized before hashing; non-canonical forms produce different hashes for equivalent data.
- The choice of hash function is application-defined. SHA-256 is conventional and currently considered secure; applications using GDX for long-lived signed content should be prepared to migrate to stronger hash functions as cryptographic guidance evolves.
- Signatures cover the canonical bytes, not the data model. Two implementations that disagree about the canonical form will produce different signatures even for equivalent data. Conformance to the canonical-form rules (Appendix A.5) is therefore security-critical, not just stylistic.
- Templates and Common Vocabulary references in signed documents create a dependency on the referenced definitions: if those definitions change, the signature still validates (the bytes are unchanged) but validation against the (now-different) Templates may produce different results. Long-lived signed documents should pin vocabulary versions to avoid this.

### C.7 Transport Wrapper Trust

Transport-layer roles (Package, Group) carry metadata about the conveyance. This metadata is data — it can be authored by any party and should not be trusted as an authority signal.

Considerations:

- Package metadata fields like `source`, `exportedBy`, or timestamps are application-defined assertions, not enforced provenance. Receivers MUST NOT treat such fields as proof of origin without independent verification (signatures, transport-layer authentication, etc.).
- A Package carrying claims about its contents (counts, content type) does not bind those claims; a receiver should validate the actual contents independently rather than relying on the wrapper's metadata.
- Discarding transport wrappers after unpacking (§9.3) means Package-level metadata is lost unless explicitly preserved as provenance. Applications that care about the chain of custody should establish provenance preservation conventions explicitly.

### C.8 Document-Level Considerations

A few practices apply across the categories above:

- **Unknown is untrusted.** Documents from unknown sources should be processed with all relevant limits enabled, regardless of how benign they appear. Trust comes from authenticated sources or signatures, not from document content.
- **Validation is not authorization.** A Layer 2 schema-conforming document is not necessarily trustworthy; it just satisfies a particular schema. Authentication, authorization, and integrity verification are application-layer concerns above GDX.
- **Errors should not leak.** Error messages from parsers and validators may inadvertently disclose internal paths, file contents, or system configuration. Error messages exposed to untrusted parties should be sanitized.
- **Resource limits are configuration.** The recommended defaults in this appendix are starting points. Production deployments should tune limits based on observed legitimate document characteristics and threat model.

The security posture of a GDX-consuming system is determined by the implementation, the deployment environment, and the trust relationships with document sources. This appendix identifies attack classes specific to GDX's structural and reference features; standard application security practices apply on top of these.

---
