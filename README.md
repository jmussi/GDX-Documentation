# GDX-Documentation
Graph data eXchange Format

DRAFT SPEC for FEEDBACK

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
