# GDX — Quick Start

- **Status:** Draft v0.20.0 (companion to GDX Core Specification v0.20.0, working draft, pre-1.0)
- **Date:** 2026-06-09
- **Editors:** 
- **Authors:** Julian Mussi


## Abstract

This document is a non-normative companion to the GDX Core Specification. It introduces GDX through a progression of small examples for readers who want a fast on-ramp before reading the specification proper. The examples are deliberately minimal; the Core Specification is the authority on what's valid.

A reader who finishes this document should understand enough GDX to recognize the basic shapes, choose between primitive types, and know where to look in the specification for deeper detail.


## Status of This Document

This is the v0.20.0 working draft of the Quick Start, companion to the GDX Core Specification v0.20.0. It was previously published as Appendix D of the Core Specification through v0.10.0; it is now a separate companion document and is recommended as the entry-point for newcomers to GDX.

---


## 1. The Simplest GDX Document

An Object with one Property:

```
<OBJ Person
    <STR name : "Alice">
>
```

Read this as: an Object named `Person` containing a string-valued Property named `name` with the value `"Alice"`. The `<` and `>` delimit each element; the `STR` prefix declares the inner element is a string; the `:` separates the Property's identifier from its value; the outer `OBJ` declares the document's root entity.

A GDX document's root must be an Object, Template, Relation Declaration, or Package (Core Spec §7) — the Object here plays that role. Inside it, the `STR` Property carries the typed value.

If you've used JSON, the equivalent is `{"Person": {"name": "Alice"}}` — but with one important difference: GDX states explicitly that `name` is a string. There's no ambiguity about whether `42` means the integer 42 or the string "42"; in GDX you write `<INT age : 42>` for the integer or `<STR age : "42">` for the string.

## 2. Several Typed Values

An Object holding a short configuration:

```
<OBJ DatabaseConfig
    <STR databaseHost : "db.internal">
    <INT databasePort : 5432>
    <BOOL tlsEnabled : true>
    <FLOAT timeoutSeconds : 30.0>
>
```

Each inner line is a typed Property. The prefix says what type the value is: `STR` for string, `INT` for integer, `BOOL` for boolean, `FLOAT` for float. No type inference, no surprises — `<INT port : 5432>` is unambiguously an integer; `<STR port : "5432">` is unambiguously a string.

## 3. Composing Values into Records

A Record groups properties into a structured composite. Records appear as children of an Object (or another structural element); a Record cannot be a document root on its own.

```
<OBJ Customer
    <REC address
        <STR street : "Main St">
        <INT number : 100>
        <STR city : "Oakland">
    >
>
```

The Record `address` has three child Properties. The whole Record is one logical thing — an address — with parts. Records are like JSON objects, but the composite itself is named (`address`) rather than just being a value.

Records can nest within Objects alongside other Properties:

```
<OBJ Customer
    <STR firstName : "Alice">
    <REC address
        <STR street : "Main St">
        <INT number : 100>
    >
>
```

The Object `Customer` contains a Property and a nested Record.

## 4. Multiple Values with Sets

A Set holds multiple values of the same role:

```
<OBJ CorsConfig
    <STR* allowedOrigins
        <STR : "https://app.example.com">
        <STR : "https://admin.example.com">
    >
>
```

The `*` suffix on the prefix turns it into a Set. `STR*` is "Set of strings." The members carry the same prefix (`STR`) but no identifier — they identify themselves by position and value.

## 5. Edges Between Objects

Objects can carry @-edges to other Objects, forming graph relationships:

```
<OBJ &alice
    <STR firstName : "Alice">
    <@OBJ #knows : @bob>
>

<OBJ &bob
    <STR firstName : "Bob">
>
```

The @-edge `<@OBJ #knows : @bob>` says: from `alice`, there is a `knows` relationship to `bob`. The `@OBJ` prefix declares the target is an Object; the `#` introduces the predicate (`knows`); the `@bob` Resource names the target root by its Id (`&bob`). Links always bind to an Id, never to a Name, so renaming `bob` does not break the edge.

The predicate can be left out when the relationship's nature isn't known yet: `<@OBJ &link1 : @bob>` says "alice stands in *some* relationship to bob, to be classified later." The edge exists either way — being an edge is what asserts the relationship; the `#` predicate says what it is, and an edge without one is simply undefined as to nature until the field is filled in (spec §6.4.2).

This is the graph dimension of GDX: tree-structured documents with explicit relationships between elements that aren't parent-child.

For references to elements that don't have a global identifier — a Record inside an Object, a specific item in a list — GDX has a path syntax introduced by `$`:

```
<OBJ alice
    <REC mailing
        <STR street : "Main St">
        <STR city : "Oakland">
    >
>

<%REC residence : $/alice/mailing>
```

The `$/alice/mailing` traverses the document tree: into `alice`, then into `mailing`. A root reference and a path compose for targets inside another root — `@bob$/contacts/[1]` resolves the root Object whose Id is `bob` and then takes its first contact. Path expressions are defined in the companion GDX Path Expressions Specification.

## 6. Cross-System Semantic Alignment with Predicates

The `#predicate` field also works on Properties to identify what concept the property represents:

```
<OBJ Person
    <STR #schema:givenName firstName : "Alice">
>
```

The Property is locally called `firstName` but semantically aligned with `schema:givenName` (a well-known predicate from schema.org). Two systems using different local names (`firstName` vs `givenName` vs `prenom`) can recognize the same concept by matching the predicate.

This is why GDX is described as a "graph format with semantic alignment" — predicates carry shared meaning across documents and systems.

## 7. Validation with Templates

A Template declares the expected shape of conforming instances:

```
<DEFOBJ Customer
    <STR firstName>
    <STR lastName>
    <INT age>
>

<OBJ alice --Customer
    <STR firstName : "Alice">
    <STR lastName : "Smith">
    <INT age : 24>
>
```

The Template `DEFOBJ Customer` declares that a Customer Object has `firstName`, `lastName`, and `age` fields. The instance Object `alice` claims `--Customer` (it conforms to the Customer Template). A validator checks that the instance has all the declared fields.

Templates are themselves GDX. They use the same syntax as the data they describe. There's no separate schema language.

## 8. The Compact Form

For human authoring, GDX has a Compact Form using indentation instead of brackets:

```
OBJ config
  STR databaseHost : "db.internal"
  INT databasePort : 5432
  BOOL tlsEnabled : true
  STR* allowedOrigins
    STR : "https://app.example.com"
    STR : "https://admin.example.com"
```

Each element is one line. Children are indented under their parents. The same data, the same prefixes, the same semantics — just less visual weight. The Compact Form is defined in a companion specification.

The same content in the minified single-letter form (for comparison):

```
O config
  S databaseHost : "db.internal"
  I databasePort : 5432
  L tlsEnabled : true
  S* allowedOrigins
    S : "https://app.example.com"
    S : "https://admin.example.com"
```

Both forms are equivalent; canonical (above) is the default for authoring, and minified is available for transport-efficiency contexts.

## 9. Why Use GDX

Three things GDX gives you that JSON or YAML don't:

**1. Explicit types.** No more "is this a string or a number?" The prefix tells you. `<INT age : 42>` is unambiguously an integer; `<STR version : "1.10">` doesn't accidentally become the float `1.1`.

**2. Semantic alignment.** The `#predicate` field carries shared meaning across systems. Your `firstName` can be aligned to `schema:givenName` without renaming your fields.

**3. Schema is data.** Templates are GDX elements. The same parser reads them. There's no second format to learn for validation; there's no separate schema file to keep in sync with the data.

What it costs: a few characters per element (the prefix and brackets). For data that crosses system boundaries or persists over time, the cost pays for itself in fewer surprises.

## 10. Where to Go Next

The Quick Start above shows about 80% of what you'll use day to day. For the rest:

- Core Spec §1 (Introduction) explains the problem space and design philosophy
- Core Spec §3 (GDX Concepts) is the conceptual centerpiece: how the element form works and why it's shaped that way
- Core Spec §3 (Abstract Data Model) and §5 (Canonical Text Form) are the foundational definitions
- Core Spec §5 (Prefix Vocabulary) and §7 (Element Roles) together are the complete reference for every role of element and its prefix
- Core Spec §9 (Transport) covers how Packages and Groups behave during exchange
- The GDX Cookbook (companion specification) shows worked patterns for common needs
- The Companion Vocabulary specification defines standardized identifiers (Templates, predicates, validation extensions) you can use across applications
- The Compact Form Grammar specification defines the human-friendly serialization shown in §8
- The GDX Path Expressions specification defines the `$path` syntax shown in §5, including the address-mode singularity rules for path-based references
- The Implementer's Guide provides practical guidance for building parsers, validators, and tools

The full specification is more rigorous than a tutorial can be — it defines what every implementation must agree on. The Quick Start above is a friendly door; the rest of the spec is the building.
