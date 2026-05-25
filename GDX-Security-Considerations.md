# GDX — Security Considerations

- **Status:** Draft v0.12.0 (companion to GDX Core Specification v0.12.0, working draft, pre-1.0)
- **Date:** 2026-05-20
- **Editors:** 
- **Authors:** Julian Mussi


## Abstract

This document is a non-normative companion to the GDX Core Specification. It discusses security risks associated with implementing GDX parsers, validators, and resolvers, along with recommended mitigations. The risks and mitigations described here are advisory; specific limits, policies, and implementation choices are left to consumers based on their threat models.


## Status of This Document

This is the v0.11.0 working draft of the Security Considerations, companion to the GDX Core Specification v0.11.0. It was previously published as Appendix C of the Core Specification through v0.10.0; it is now a separate companion document for clearer separation of normative and non-normative content.

v0.11.0 introduces path-mode reference resolution (`$path` and `@id$path`). The threat surface for path-mode references is anticipated to be similar to that of identifier-mode references — cycles, unresolved targets, scope confusion — with one additional consideration: path expressions traverse the document tree by direct child match, and a maliciously-constructed document may exploit ordinal segments (`[N]`) to address arbitrary sub-elements. The mitigations enumerated in §2 (Reference Resolution) extend to path-mode references. A future revision SHOULD examine path-specific threats once implementations accumulate experience.

---


## 1. Parser Resource Exhaustion

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

## 2. Reference Cycle Exploitation

GDX permits @-edges (`@OBJ`, `@REL`) between graph-entities, allowing arbitrary graph structures including cycles. Cycles are valid data and MUST be supported by parsers and storage layers. The security risk arises when adversarial documents use cycles to attack consumers that traverse references without cycle detection — for example, a service that loads referenced Objects to display them, or that walks @-edge networks to compute derived data.

Attack vectors:

- **Resource exhaustion via traversal.** A document with a cyclic reference graph causes naive traversal to loop indefinitely, consuming CPU and memory until limits are hit or the process is killed.
- **Amplification via deep chains.** Long reference chains that eventually cycle can multiply the cost of traversal-based operations, making moderate-size documents trigger expensive processing.

Mitigations:

- Reference traversal implementations MUST track visited Objects and avoid re-traversing them within a single traversal operation.
- Implementations SHOULD impose a maximum reference traversal depth as a defense-in-depth measure even when cycle detection is implemented.
- Resolvers SHOULD detect cycles in Template chains during effective-shape computation (though Template inheritance is not part of GDX, future Common Vocabulary or application Templates may reference each other; cycles in such reference chains should be detected).

## 3. VAL Occupant Nesting

VAL slots (`VAL`/`V`, per §7.11 of the Core Specification) carry exactly one occupant element each, and the occupant may itself contain further VAL slots. A maliciously-constructed document may use deeply-nested VAL chains to amplify traversal cost on consumers that recursively descend into slot occupants — analogous to deep-tree resource-exhaustion (§1) but with the additional consideration that each VAL contributes two element layers (the slot and its occupant) rather than one.

GDX's VAL role inlines its occupant directly in the document text; it does NOT expand external references at parse time. Consequently, the classical "billion laughs"-style amplification attack (where each expansion resolves a reference that yields further expansions) does not apply to VAL as specified. Implementations that layer reference-expansion semantics on top of VAL (e.g., a vocabulary-defined "include this referenced fragment as a slot occupant" extension) MUST treat such expansions as their own attack surface and impose limits comparable to those recommended in §1.

Mitigations:

- The §1 nesting-depth and element-count limits apply directly to VAL chains; no separate VAL-specific limit is normatively required.
- Consumers walking VAL trees SHOULD count each VAL and each occupant against any element-traversal budget.
- Reference resolvers MUST NOT silently substitute an occupant fetched from an external source for a VAL occupant declared in the document; any vocabulary-defined inclusion mechanism is out of scope for this specification and is the responsibility of the consuming application.

## 4. Network Resolution

The default Type and @-Edge reference resolution orders (Core Spec §9.5.2 and §7.4.1) include "globally-resolvable IRIs" as a fallback resolution scope. Implementations that resolve such references over the network expose themselves to several risks:

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

## 5. Template Validation Behavior

Validators that resolve and apply Templates from untrusted vocabularies face risks:

- **Malicious constraints.** A Template defining a regex pattern can include patterns vulnerable to ReDoS (regular expression denial of service) — exponential-time backtracking on certain inputs. Validators applying such patterns expose themselves to DoS.
- **Resource exhaustion via constraint evaluation.** Some constraint extensions may permit computationally expensive evaluation; an adversarial Template could combine many such extensions.

Mitigations:

- Regex-evaluating validators SHOULD use regex engines with linear-time guarantees (e.g., RE2, Hyperscan) when processing Templates from untrusted sources.
- Implementations SHOULD impose per-element validation time limits for untrusted input.
- Implementations MAY restrict which constraint extensions are honored when processing untrusted Templates.

## 6. Content Addressing and Signing

Content addressing of GDX documents requires the strict canonical form (Appendix A.5). Signatures over content addresses derive their security from both the canonical-form algorithm and the underlying hash function.

Considerations:

- Content addresses are valid only over the strict canonical form. Any non-canonical input must be normalized before hashing; non-canonical forms produce different hashes for equivalent data.
- The choice of hash function is application-defined. SHA-256 is conventional and currently considered secure; applications using GDX for long-lived signed content should be prepared to migrate to stronger hash functions as cryptographic guidance evolves.
- Signatures cover the canonical bytes, not the data model. Two implementations that disagree about the canonical form will produce different signatures even for equivalent data. Conformance to the canonical-form rules (Appendix A.5) is therefore security-critical, not just stylistic.
- Templates and Common Vocabulary references in signed documents create a dependency on the referenced definitions: if those definitions change, the signature still validates (the bytes are unchanged) but validation against the (now-different) Templates may produce different results. Long-lived signed documents should pin vocabulary versions to avoid this.

## 7. Transport Wrapper Trust

Transport-layer roles (Package, Group) carry metadata about the conveyance. This metadata is data — it can be authored by any party and should not be trusted as an authority signal.

Considerations:

- Package metadata fields like `source`, `exportedBy`, or timestamps are application-defined assertions, not enforced provenance. Receivers MUST NOT treat such fields as proof of origin without independent verification (signatures, transport-layer authentication, etc.).
- A Package carrying claims about its contents (counts, content type) does not bind those claims; a receiver should validate the actual contents independently rather than relying on the wrapper's metadata.
- Discarding transport wrappers after unpacking (Core Spec §10.3) means Package-level metadata is lost unless explicitly preserved as provenance. Applications that care about the chain of custody should establish provenance preservation conventions explicitly.

## 8. Document-Level Considerations

A few practices apply across the categories above:

- **Unknown is untrusted.** Documents from unknown sources should be processed with all relevant limits enabled, regardless of how benign they appear. Trust comes from authenticated sources or signatures, not from document content.
- **Validation is not authorization.** A Layer 2 schema-conforming document is not necessarily trustworthy; it just satisfies a particular schema. Authentication, authorization, and integrity verification are application-layer concerns above GDX.
- **Errors should not leak.** Error messages from parsers and validators may inadvertently disclose internal paths, file contents, or system configuration. Error messages exposed to untrusted parties should be sanitized.
- **Resource limits are configuration.** The recommended defaults in this document are starting points. Production deployments should tune limits based on observed legitimate document characteristics and threat model.

The security posture of a GDX-consuming system is determined by the implementation, the deployment environment, and the trust relationships with document sources. This appendix identifies attack classes specific to GDX's structural and reference features; standard application security practices apply on top of these.

---

