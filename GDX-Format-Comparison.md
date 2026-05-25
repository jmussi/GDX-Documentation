# GDX — Format Comparison

- **Status:** Draft v0.13.0 (companion to GDX Core Specification v0.13.0, working draft, pre-1.0)
- **Date:** 2026-05-20
- **Editors:** 
- **Authors:** Julian Mussi


## Abstract

This document is a non-normative companion to the GDX Core Specification. It provides side-by-side comparisons between GDX and other widely-used data formats to illustrate GDX's design properties and to serve as conversion guidance for readers familiar with those formats.

The comparisons are illustrative, not exhaustive. Where GDX's design overlaps with another format, the comparison shows what is preserved and what differs. Where GDX adds capabilities not present in the comparison format, those additions are noted as design extensions rather than as criticisms of the other format.


## Status of This Document

This is the v0.11.0 working draft of the Format Comparison, companion to the GDX Core Specification v0.11.0. It was previously published as Appendix B of the Core Specification through v0.10.0; it is now a separate companion document for clearer separation of normative and non-normative content.

---


## 1. Purpose and Scope

The comparisons in this document are intended to illustrate, not to argue superiority. Each format compared has strengths GDX does not match (JSON's universality, YAML's hand-edit friendliness, XML's mixed-content support, RDF's inference ecosystem). The point of comparison is to make GDX's distinguishing properties visible by contrast and to help readers familiar with other formats see how their data would translate.

The comparisons are at a moment in time. Format ecosystems evolve; new variants emerge. The specific syntactic forms shown here reflect widely-used conventions at the time of this draft. Where a format has multiple variants (JSON Schema versions, RDF serializations), one representative variant is chosen and noted.

Each comparison follows the same structure: a brief problem statement, the data shown in each relevant format, and a brief discussion of what each form captures and what it leaves implicit.

## 2. Typed Composite Record

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

## 3. Predicate-Aligned Data

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

## 4. Schema-Validated Data

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

## 5. Graph-Structured Data

**Problem:** Express a small graph of three people with directed `knows` relationships, where the relationships have attributes (the date the connection was made).

**GDX:**

```
<OBJ alice
    <STR name : "Alice">
    <@OBJ #knows : @bob
        <STR since : "2024-01-15">
    >
    <@OBJ #knows : @carol
        <STR since : "2024-03-22">
    >
>

<OBJ bob
    <STR name : "Bob">
    <@OBJ #knows : @carol
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

- **GDX** uses the @-Edge family (`@OBJ`, `@REL`) as children of the source element. The edge's `#rel` field names the predicate (`knows`), the Resource is a reference to the target (`@bob`), and the edge's children carry the edge attributes (`since`). The pattern naturally extends to multiple edges per source and edges with rich attribute structure, and an optional identity on the edge (per Core Spec §7.4 axiom E6) gives the edge itself a persistent substance that survives changes to its predicate and attributes — named edges without RDF-style reification. GDX is essentially a property-graph format in this dimension.
- **RDF basic** expresses simple subject-predicate-object triples cleanly but lacks a native way to attach attributes to a triple. The historical workarounds (reification, n-ary relations) are verbose and rarely used in practice.
- **RDF-star** adds the `<< s p o >>` syntax to address this, allowing properties on triples directly. This is now standardized as RDF 1.2 and mirrors the property-graph model.
- **Property graph databases** (Neo4j, Neptune, etc.) treat edge attributes as first-class. GDX's @-Edge model is essentially the same shape; GDX adds the document-friendly serialization that property graph databases typically lack and a name-by-identity substance/predication split that property graph databases handle case-by-case.
- **JSON** has no native graph notion. Embedded references (`{"target": "bob"}`) are convention; the consumer must know to traverse them. References across documents require additional convention (JSON Pointer, URI fragments, etc.). For small graphs this works; for large graphs it becomes unwieldy.

GDX occupies a useful position in this space: it expresses property-graph semantics natively (edges with attributes), supports graph traversal through reference resolution, and serializes as a document-friendly text format. For systems that want graph semantics without committing to a graph database stack, GDX provides a natural representation.

## 6. Summary

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

