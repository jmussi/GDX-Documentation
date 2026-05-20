# GDX Standard Form — Grammar Specification

- **Status:** Draft v0.7 (working draft)
- **Date:** 2026-05-20
- **Companion to:** GDX Core Specification v0.11.0
- **Authors:** Julian Mussi


## Abstract

This document defines the formal grammar for GDX Standard Form, the canonical bracket-delimited text representation defined in §5 of the GDX Core Specification. The grammar is presented in Extended Backus-Naur Form (EBNF) and is intended as the normative reference for parser implementations.

The grammar is partitioned into three layers: document and element structure (high-level syntax), field and literal content (mid-level), and lexical primitives (character classes and token rules). Cross-references to the Core Specification appear throughout.

This document does not address the Compact Form, the Binary Form, semantic validation, or any vocabulary content. Those concerns are addressed in separate companion specifications.

Changes from v0.3: prefix vocabulary restructured for the canonical/minified system introduced in Core Spec v0.9. Each prefix now has a canonical form (three-letter or longer where natural) and, for most prefixes, a minified form (single-letter). Prefix recognition is case-insensitive on input. The Template modifier changed from `TD` to `DEF`. Bare `N` prefix removed; Node modifier always composes with a base. PACKAGE renamed to PKG; BLOCK to BLK; GROUP-canonical is GRP. See §7.1.

Changes from v0.4 (this revision, v0.5): conformance with Core Specification v0.9.10. Cross-references updated to current section numbering: §8.1 (composition matrix) → §7.1; §8.2 rule 1 (root rule) → §7.2 rule 1; §11 (transport-layer) → §9; §3.3.1 (Numeric Value-Kinds) → §4.3.1; §6.10 (REM Annotation) → §6.3.6; "§4" in the abstract (Standard Form definition) → §5. Corrected `GroupPrefix` (§7.1 and Appendix A): GRP has no minified form; the previous `"G"` minified alternative was removed. Group prefix relocated in the §7.1 grammar from "bare structural primitives" to "transport-layer prefixes" to match the Core Spec's typological grouping.

Changes from v0.5 (revision v0.6): conformance with Core Specification v0.10.0. The prefix vocabulary is reframed around the composition pattern `[@?][refinement-modifier?][base][*?]`. (1) The Set marker `*` is now a suffix rather than a prefix; the EBNF `SetPrefix` production is replaced by `SetSuffix`, which composes after a base or refinement-modifier-base. (2) A new `ReferencePrefix` (`@`) is added as the reference operator; the `@KIND` family composes with any `ModifierBase`, optionally followed by `SetSuffix`. The composition pattern is described rather than enumerated; valid `@KIND` prefixes are derived from the pattern. (3) The Reference operator MUST NOT compose with itself (`@@`-prefixed compositions are malformed). (4) The reference literal grammar (§5.6) is extended: identifier-mode `@Id` is normatively specified; path-mode `$PathExpression` is syntactically reserved with grammar deferred to a future revision (Core Specification §11.4). (5) The §3.1 element-variations table is reorganized by role category (instance, refinement, declaration, transport) and includes `ReferenceElement` and `SetElement` as derived forms. Cross-references to the Core Specification updated for §6.5.x renumbering (REM Annotation §6.3.6 → §6.5.6). No changes to literal grammars (strings, numerics, booleans, bytes), identifier grammar, field grammars, or whitespace rules.

Changes from v0.6 (this revision, v0.7): conformance with Core Specification v0.11.0. (1) **§5.6 reference literal grammar normatively specifies path-mode and key-mode.** The placeholder `PathExpression ::= (* reserved; grammar deferred *)` is replaced by a forward reference to the companion GDX Path Expressions Specification (`GDX-Path.md`). Five `ReferenceLiteral` productions are recognized: `IdentifierReference` (`@Id`), `PathReference` (`$PathExpression`), `KeyReference` (`&KeyToken`), `IdPathReference` (`@Id$PathExpression`), and `IdKeyReference` (`@Id&KeyToken`). The `KeyReference` and `IdKeyReference` forms address an element by its location-stable `Key` anchor, scoped per Core Specification §5.4.5. (2) **§6 identifier grammar narrows.** `IdStartChar` no longer admits `/`. Identifiers may continue to contain `/` (`IdContChar` unchanged); only the leading-`/` form is removed. This change is a breaking change for documents conforming to v0.6 or earlier; the migration is mechanical (drop the leading `/` or move to a path-mode reference). (3) Appendix A updated to reflect the new reference-literal productions and the narrowed identifier grammar. (4) Cross-references to Core Specification updated to v0.11.0.


## Table of Contents

1. [Notation](#1-notation)
2. [Document Structure](#2-document-structure)
3. [Element Structure](#3-element-structure)
4. [Fields](#4-fields)
5. [Resource Literals](#5-resource-literals)
6. [Identifiers](#6-identifiers)
7. [Lexical Primitives](#7-lexical-primitives)
8. [Whitespace Rules](#8-whitespace-rules)
9. [Open Items](#9-open-items)
10. [Appendix A — Complete Grammar](#appendix-a--complete-grammar)


---

## 1. Notation

This document uses Extended Backus-Naur Form with the following conventions:

| Symbol | Meaning                                        |
|--------|------------------------------------------------|
| `::=`  | Production definition                          |
| `\|`   | Alternation                                    |
| `*`    | Zero or more repetitions                       |
| `+`    | One or more repetitions                        |
| `?`    | Optional (zero or one occurrence)              |
| `(...)`| Grouping                                       |
| `"..."` | Literal terminal (exact characters)           |
| `[a-z]` | Character class (single character in range)  |
| `~`    | Negation (any character except those listed)   |
| `// ...` | Comment (informative; not part of grammar)   |

Productions are case-sensitive. Whitespace between tokens is governed by §8 and is generally not shown explicitly within productions; the rules in §8 are normative.

Grammar names use `PascalCase`. Lexical terminals and character classes use lowercase or descriptive names.


---

## 2. Document Structure

A GDX document consists of one or more top-level elements separated by optional whitespace.

```ebnf
Document        ::= WS? TopLevelElement (WS+ TopLevelElement)* WS?

TopLevelElement ::= Object
                  | Template
                  | RelDeclaration
                  | Package
```

Per Core Specification §7.2 rule 1, the root of a GDX document MUST be one of: an Object, a Template, a REL declaration, or a Package. A document MAY contain multiple top-level elements; each top-level element MUST independently be a root-eligible type. This supports vocabulary documents that declare multiple Templates or REL elements at the root, files containing several independent Objects, and similar multi-root use cases.

An empty document (no top-level elements) is malformed.

```ebnf
Object          ::= ObjectElement
Template        ::= TemplateElement
RelDeclaration  ::= RelElement
Package         ::= PackageElement
```

The detailed structure of these elements is defined in §3. Package and Group elements (transport-layer constructs introduced in Core Specification §9) follow the same general element grammar as data-layer elements; they are distinguished by their prefix tokens.


---

## 3. Element Structure

Every GDX element has the same general structure: an opening bracket, a prefix declaring the kind, optional identity and value fields, optional children, and a closing bracket.

```ebnf
Element         ::= "<" WS? Prefix ElementBody WS? ">"

ElementBody     ::= IdentitySection? ResourceSection? Children?

IdentitySection ::= (WS Key)? (WS Rel)? (WS Id)?

ResourceSection ::= WS? ":" WS? Resource? (WS "--" Type)?
                  | WS "--" Type
                  // Resource? makes the value side optional after ':'.
                  // The forms `<STR name : >`, `<STR name : --gdx:type>`,
                  // and `<STR name>` are all valid; the bare form is canonical.
                  // The bare "WS '--' Type" form (no ':') applies when the
                  // element carries a Type but no Resource and the writer
                  // prefers to omit the separator.

Children        ::= (WS Element)+
```

### 3.1 Element Variations by Kind

The general Element production above accepts any well-formed element. Additional structural rules are imposed by the role (declared via the Prefix) and the composition matrix of Core Specification §7.1. Those rules are semantic and are validated separately from grammar parsing.

The element forms recognized at the grammar level, organized by role category:

```ebnf
(* Instance roles *)
ObjectElement       ::= "<" WS? ObjectPrefix ElementBody WS? ">"
RecordElement       ::= "<" WS? RecordPrefix ElementBody WS? ">"
PropertyElement     ::= "<" WS? PropertyPrefix ElementBody WS? ">"
EdgeElement         ::= "<" WS? EdgePrefix ElementBody WS? ">"
BlockElement        ::= "<" WS? BlockPrefix ElementBody WS? ">"
AnnotationElement   ::= "<" WS? AnnotationPrefix ElementBody WS? ">"

(* Refinement-modifier compositions *)
QualifierElement    ::= "<" WS? QualifierComposition ElementBody WS? ">"
ExtensionElement    ::= "<" WS? ExtensionComposition ElementBody WS? ">"
EmbeddedElement     ::= "<" WS? EmbeddedComposition ElementBody WS? ">"
NodeElement         ::= "<" WS? NodeComposition ElementBody WS? ">"

(* Declaration roles *)
TemplateElement     ::= "<" WS? TemplatePrefix ElementBody WS? ">"
RelElement          ::= "<" WS? RelPrefix ElementBody WS? ">"

(* Reference operator — composes with any of the above except itself *)
ReferenceElement    ::= "<" WS? "@" ModifierBase SetSuffix? ElementBody WS? ">"

(* Set suffix — composes with most bases and refinement modifiers *)
SetElement          ::= "<" WS? ModifierBase "*" ElementBody WS? ">"

(* Transport roles *)
PackageElement      ::= "<" WS? PackagePrefix ElementBody WS? ">"
GroupElement        ::= "<" WS? GroupPrefix ElementBody WS? ">"
```

These are aliases of the general Element production for documentation purposes. A parser may match the general Element form and then determine the role from the prefix; the role-specific aliases are not required at the lexical level.

`ReferenceElement` and `SetElement` are not mutually exclusive: a prefix may have both a leading `@` and a trailing `*` (a reference to a Set, e.g., `@STR*`). The composition pattern in §7.1 governs which prefix forms are valid.


---

## 4. Fields

Each identity field has its own production that includes its sigil and content.

### 4.1 Key Field

```ebnf
Key             ::= "&" KeyContent

KeyContent      ::= NumericKey
                  | Id

NumericKey      ::= DIGIT+
```

The `&` sigil MUST be immediately followed by the key content with no intervening whitespace. A Key may be a sequence of decimal digits (numeric slot reference) or an identifier (named slot reference).

### 4.2 Rel Field

```ebnf
Rel             ::= "#" RelContent

RelContent      ::= UriReference
                  | Id
```

The `#` sigil MUST be immediately followed by the rel content with no intervening whitespace. A Rel content is either a full URI reference (recognized by the `://` sequence) or an identifier (typically a CURIE).

```ebnf
UriReference    ::= UriScheme "://" UriChars

UriScheme       ::= ALPHA (ALPHA | DIGIT | "+" | "-" | ".")*

UriChars        ::= UriChar+

UriChar         ::= ALPHA | DIGIT | "_" | "-" | "." | "/" | ":"
                  | "?" | "#" | "[" | "]" | "@" | "!" | "$"
                  | "&" | "'" | "(" | ")" | "*" | "+" | ","
                  | ";" | "=" | "%" | "~"
```

The UriChar set follows RFC 3986. URI references are permitted in Rel position. URI references are also permitted in any other identifier position; they fall under the general Id grammar (§6) when they begin with an alphabetic character followed by `://`.

### 4.3 Type Field

```ebnf
Type            ::= "--" TypeContent

TypeContent     ::= Id
```

The `--` sigil MUST be immediately followed by the type content with no intervening whitespace. Whitespace MUST appear before the `--` sigil to separate it from the previous token. The type content uses the standard identifier grammar (§6).

### 4.4 Resource

The Resource production matches one of six value-kinds:

```ebnf
Resource        ::= StringLiteral
                  | IntLiteral
                  | FloatLiteral
                  | BoolLiteral
                  | BytesLiteral
                  | ReferenceLiteral
```

Each literal form is defined in §5.


---

## 5. Resource Literals

### 5.1 String Literals

```ebnf
StringLiteral   ::= '"' StringChar* '"'

StringChar      ::= EscapeSequence
                  | ~('"' | '\')

EscapeSequence  ::= "\\" '"'           // literal "
                  | "\\" "\\"          // literal \
                  | "\\" "n"           // newline
                  | "\\" "t"           // tab
                  | "\\" "r"           // carriage return
                  | "\\" "u" "{" HexDigit+ "}"  // Unicode codepoint
```

Any backslash sequence not in the escape set is malformed.

A StringChar matches any character (including whitespace and newlines) except an unescaped `"` or `\`. String content is preserved literally; internal whitespace including line breaks is part of the value.

### 5.2 Integer Literals

```ebnf
IntLiteral      ::= IntNumeric IntSuffix?

IntNumeric      ::= DecimalInt
                  | BinaryInt
                  | OctalInt

DecimalInt      ::= Sign? DIGIT+

BinaryInt       ::= Sign? "0b" BinaryDigit+

OctalInt        ::= Sign? "0o" OctalDigit+

Sign            ::= "+" | "-"

IntSuffix       ::= "i8" | "i16" | "i32" | "i64"
                  | "u8" | "u16" | "u32" | "u64"
                  | "bigint"
```

The integer suffix vocabulary is closed. A parser encountering an alphabetic suffix not in this set MUST reject the literal as malformed.

A signed unsigned-suffix combination (e.g., `-25u8`) is grammatically valid but is a representation-conformance failure (§4.3.1 of the Core Specification) and MUST be rejected at parse time.

### 5.3 Float Literals

```ebnf
FloatLiteral    ::= FloatStandard FloatSuffix?
                  | IntNumeric FloatSuffix      // integer-form, suffix required
                  | IntNumeric                  // integer-form, no suffix
                                                //   (only valid in float-prefix
                                                //    positions; lexically
                                                //    indistinguishable from
                                                //    IntLiteral — context
                                                //    discriminates)
                  | FloatKeyword

FloatStandard   ::= Sign? DIGIT+ "." DIGIT+ Exponent?
                  | Sign? "." DIGIT+ Exponent?
                  | Sign? DIGIT+ Exponent

Exponent        ::= ("e" | "E") Sign? DIGIT+

FloatSuffix     ::= "f32" | "f64"
                  | "d" | "d32" | "d64" | "d128"

FloatKeyword    ::= "nan" | "inf" | "-inf"
```

The float suffix vocabulary is closed. A parser encountering an alphabetic suffix not in this set MUST reject the literal as malformed.

**Lexical-context discrimination.** The lexical form `5` matches both IntNumeric (as an IntLiteral) and IntNumeric (as the integer-form alternative of FloatLiteral). The disambiguation is by *prefix context*: in an int-prefix position (`<INT count : 5>`), the literal is an IntLiteral; in a float-prefix position (`<FLOAT price : 5>`), the literal is a FloatLiteral. The parser uses the prefix to choose which production applies.

The lexical form `5f32` is unambiguously a FloatLiteral (the `f32` suffix is a FloatSuffix, not an IntSuffix). The lexical form `5i8` is unambiguously an IntLiteral.

**Asymmetry with integer literals.** Float literals containing a decimal point, an exponent, a float suffix, or a float keyword are NOT valid in integer-prefix positions. A literal like `5.0` is malformed under an `<INT count : ...>` prefix even though the value is integer-valued. The asymmetry is intentional: integers are a subset of reals, but reals are not a subset of integers. The literal form signals the intended representation domain.

| Position prefix | `5` | `5.0` | `5f32` | `5i8` | `5.0f32` |
|:---:|:---:|:---:|:---:|:---:|:---:|
| `I` (int)       | accept (IntLiteral) | reject | reject | accept (IntLiteral with suffix) | reject |
| `F` (float)     | accept (FloatLiteral, integer-form) | accept (FloatStandard) | accept (integer-form with suffix) | reject (IntSuffix in float position) | accept (FloatStandard with suffix) |

### 5.4 Boolean Literals

```ebnf
BoolLiteral     ::= BoolTrue | BoolFalse

BoolTrue        ::= "true" | "TRUE" | "True"
                  | "t" | "T"

BoolFalse       ::= "false" | "FALSE" | "False"
                  | "f" | "F"
```

Boolean literals are case-insensitive on read. Both spelled-out (`true`, `false`) and single-letter (`t`, `f`) forms are accepted. The single-letter forms exist to support input from systems that use abbreviated boolean conventions (PostgreSQL `t`/`f`, certain command-line tools) and to accommodate compact human entry.

The single-letter forms are unambiguous in Resource position because the prefix declares the value-kind: `<BOOL flag : t>` parses `t` as the boolean true. The same input outside a bool-prefix position parses as the identifier `t`.

Conforming serializers MUST write boolean values in the lowercase spelled-out form (`true`, `false`). The single-letter and uppercase forms are accepted on input but normalized on output.

### 5.5 Bytes Literals

```ebnf
BytesLiteral    ::= HexBytes
                  | Base64Bytes

HexBytes        ::= "0x" HexDigit+

Base64Bytes     ::= "0z" Base64Char+ "="*
```

A bytes literal MUST contain at least one digit/character following the prefix; bare `0x` or `0z` is malformed.

A HexBytes literal MUST contain an even number of hex digits. An odd count is malformed.

A Base64Bytes literal MUST conform to standard Base64 encoding (RFC 4648), including correct padding (`=` characters) to align the encoded length to a multiple of four characters.

```ebnf
Base64Char      ::= ALPHA | DIGIT | "+" | "/"
```

### 5.6 Reference Literals

Reference literals designate addressable elements. Five forms are recognized, combining the three operations of name resolution (`@`), structural traversal (`$`), and key lookup (`&`):

```ebnf
ReferenceLiteral    ::= IdentifierReference
                      | PathReference
                      | KeyReference
                      | IdPathReference
                      | IdKeyReference

IdentifierReference ::= "@" Id
PathReference       ::= "$" PathExpression
KeyReference        ::= "&" KeyToken
IdPathReference     ::= "@" Id "$" PathExpression
IdKeyReference      ::= "@" Id "&" KeyToken

PathExpression      ::= (* specified in GDX-Path.md §3-§4 *)
KeyToken            ::= (* specified in GDX-Path.md §5 *)
```

**Identifier-mode** (`@Id`) is normatively defined. The `@` sigil MUST be immediately followed by an identifier with no intervening whitespace. The identifier may be in any of the forms permitted by §6 (CURIEs, GUIDs, IRIs, local names, or hierarchical local names).

**Path-mode** (`$PathExpression`) is normatively defined by the companion GDX Path Expressions Specification (`GDX-Path.md`). The `$` sigil introduces a path expression addressing an element by its structural location in the document tree. The `$` sigil MUST be immediately followed by the `/` of the path with no intervening whitespace.

**Key-mode** (`&KeyToken`) is normatively defined by `GDX-Path.md`. The `&` sigil introduces a key lookup; the Key is resolved within the scope of the reference (the enclosing Object, or the document at transport-layer position; see Core Specification §5.4.5). The `&` sigil MUST be immediately followed by the Key token with no intervening whitespace.

**Composed name + path** (`@Id$PathExpression`) expresses cross-document addressing by structural traversal. The `@Id` portion resolves to a starting element by scope walk (per Core Specification §6.5.16), and the `$PathExpression` portion is then evaluated rooted at that starting element. No whitespace is permitted between `@`, `Id`, `$`, and the path. The boundary is unambiguous because `$` is not a member of `IdContChar` (§6).

**Composed name + key** (`@Id&KeyToken`) expresses cross-document addressing by key anchor. The `@Id` portion resolves to a starting Object, and the `&KeyToken` portion is then looked up within that Object's scope. No whitespace is permitted; the boundary is unambiguous because `&` is not a member of `IdContChar`.

The `$` and `&` operators do not combine within a single reference literal. The form `@Id$PathExpression&KeyToken` is malformed; an author chooses one of `IdPathReference` or `IdKeyReference` for the composed reference.

When a reference literal appears as the Resource of a `@KIND` reference element (Core Specification §6.5.16), it MUST resolve to exactly one element of the matching role. See `GDX-Path.md` §7 for the normative address-mode singularity rule.


---

## 6. Identifiers

A single identifier grammar applies to all identifier-bearing positions: the `Id` field, the contents of named keys (§4.1), the contents of Rel field when not a URI (§4.2), the contents of Type field (§4.3), and the target of reference literals (§5.6).

```ebnf
Id              ::= IdStartChar IdContChar*

IdStartChar     ::= ALPHA | "_"

IdContChar      ::= ALPHA | DIGIT | "_" | "-" | "." | "/" | ":"
```

An identifier MUST begin with a letter or an underscore. It MAY continue with any combination of letters, digits, underscores, hyphens, periods, forward slashes, and colons.

The forward slash (`/`) is admitted as a continuation character so that IRIs (e.g., `https://acme.com/people/alice`) remain valid identifiers. It is NOT admitted as a start character; identifiers may not begin with `/`. The leading-`/` syntactic slot is reserved for path-mode reference literals (§5.6 and `GDX-Path.md`). This restriction (introduced in v0.7) is a breaking change from v0.6 and earlier; see `GDX-Path.md` Appendix C for migration patterns.

Identifiers MUST NOT begin with a digit (which would be lexically ambiguous with numeric literals). Identifiers MUST NOT contain whitespace, brackets, sigil characters (other than as listed above), or other punctuation.

Identifiers are tokens that extend as far as possible from their start position. An identifier ends when the parser encounters a character outside IdContChar (most commonly, whitespace). A `:` within an identifier is an IdContChar (e.g., the CURIE separator in `gdx:Person`); a `:` separated by whitespace is the Resource separator (§4.4).

### 6.1 Disambiguation from Numeric Literals

An identifier that starts with a letter or underscore is unambiguously an identifier. The `IdStartChar` set excludes digits to prevent collision with numeric literals.

Identifiers beginning with `/` (path-style identifiers) are also unambiguous because `/` does not appear at the start of any literal form.

### 6.2 Disambiguation from URI References (in Rel position)

In Rel position only (§4.2), a content beginning with an alphabetic sequence followed by `://` is parsed as a URI reference rather than an identifier. The lexer SHOULD recognize this prefix pattern and switch to URI parsing. In all other identifier positions, the same input is parsed as a maximal identifier (which incidentally captures the URI-like form because all its characters are valid IdContChars).

### 6.3 Single-Character Identifiers

Single-letter identifiers are valid. Position parsing disambiguates them from prefix tokens: the first non-whitespace token after `<` is always the prefix; subsequent tokens in identifier position are identifiers regardless of length.

```
<INT I : 42>          // valid: prefix INT, identifier I, integer 42
<REC r>               // valid: prefix REC, identifier r, no Resource (null)
<I I : 42>            // valid: prefix I (minified), identifier I, integer 42
<R r>                 // valid: prefix R (minified), identifier r, no Resource
```

Note that `<I I : 42>` parses unambiguously: the first token after `<` is the prefix `I` (minified form of INT); the second token is the identifier `I`. Position determines role.


---

## 7. Lexical Primitives

### 7.1 Prefix Vocabulary

The Prefix production matches one of the closed vocabulary tokens defined in Core Specification §6. GDX prefix tokens have two equivalent textual forms — **canonical** (three-letter or longer where natural) and **minified** (single-letter where applicable). Both forms are recognized on input; serializers produce one or the other based on configuration.

A GDX prefix follows the composition pattern:

```
[@?] [refinement-modifier?] [base] [*?]
```

Where:
- `@` is the optional Reference operator (prefix position)
- `refinement-modifier` is one of `.`, `_`, `%`, `N`, `DEF` (at most one)
- `base` is a core role name
- `*` is the optional Set suffix

Prefix recognition is case-insensitive. The grammar productions below show prefixes in their uppercase canonical case for clarity; conforming parsers MUST accept the same tokens in any combination of upper and lower case (`STR`, `Str`, `str`, `sTR` are all valid). Identifiers (Id, Key, Rel) remain case-sensitive (§6).

```ebnf
Prefix              ::= ReferencePrefix? ModifierBase SetSuffix?

ReferencePrefix     ::= "@"

ModifierBase        ::= RefinementBase
                      | TemplatePrefix
                      | RelPrefix
                      | AnnotationPrefix
                      | TransportPrefix
                      | BlockPrefix
                      | EdgePrefix
                      | BasePrefix

SetSuffix           ::= "*"

(* Refinement-modifier composition with bases. At most one refinement
   modifier may appear; refinement modifiers do not stack with each other. *)

RefinementBase      ::= QualifierComposition
                      | ExtensionComposition
                      | EmbeddedComposition
                      | NodeComposition

QualifierComposition ::= "." PropertyPrefix

ExtensionComposition ::= "_" PropertyPrefix

EmbeddedComposition  ::= "%" PropertyPrefix
                       | "%" RecordPrefix
                       | "%" ObjectPrefix

NodeComposition      ::= "N" PropertyBase
                       | "N" RecordBase

(* Bare base prefixes — canonical | minified *)

BasePrefix          ::= ObjectPrefix
                      | RecordPrefix
                      | PropertyPrefix

ObjectPrefix        ::= "OBJ"  | "O"
RecordPrefix        ::= "REC"  | "R"

PropertyPrefix      ::= "STR"   | "S"
                      | "INT"   | "I"
                      | "FLOAT" | "F"
                      | "BOOL"  | "L"
                      | "BYTE"  | "B"

PropertyBase        ::= "STR"   | "S"
                      | "INT"   | "I"
                      | "FLOAT" | "F"
                      | "BOOL"  | "L"
                      | "BYTE"  | "B"

RecordBase          ::= "REC"   | "R"

(* Data-layer specialty prefixes *)

EdgePrefix          ::= "EDGE" | "E"

(* Template prefixes — canonical only, no minified form *)

TemplatePrefix      ::= "DEFOBJ"
                      | "DEFREC"
                      | "DEFSTR"
                      | "DEFINT"
                      | "DEFFLOAT"
                      | "DEFBOOL"
                      | "DEFBYTE"

(* Standalone declaration prefixes — canonical only, no minified form *)

RelPrefix           ::= "REL"

AnnotationPrefix    ::= "REM"

(* Organizational prefix — canonical only, no minified form *)

BlockPrefix         ::= "BLK"

(* Transport-layer prefixes — canonical only, no minified form *)

TransportPrefix     ::= PackagePrefix
                      | GroupPrefix

PackagePrefix       ::= "PKG"

GroupPrefix         ::= "GRP"
```

**The Reference operator** `@` composes with any `ModifierBase`, optionally followed by `SetSuffix`. The composition produces a `@KIND` reference prefix where the KIND is the modifier-base (and optional set-suffix) being referenced. Examples:

- `@OBJ` — reference to an Object
- `@REL` — reference to a REL declaration
- `@DEFSTR` — reference to a string-Template
- `@.STR` — reference to a string-Qualifier
- `@_INT` — reference to an int-Extension
- `@%REC` — reference to an Embedded Record
- `@STR*` — reference to a Set of strings
- `@.STR*` — reference to a Set of string-Qualifiers

The Reference operator MUST NOT compose with another Reference operator. `@@`-prefixed compositions are malformed.

**Recognition note.** Implementations MUST match the longest valid prefix at prefix position. When both `STR` and `S` could match (because `S` is a valid prefix and `STR` extends it), the longer match wins: `<STRangle>` is invalid (no element separator after `STRangle`), but `<STR angle>` parses with prefix `STR` and identifier `angle`. Similarly, when both `STR*` and `STR` could match at prefix position, the longer match (`STR*`) wins.

A token in prefix position not matching the above is malformed.

**On the closed vocabulary.** The prefix vocabulary is closed and committed against future change. The composition pattern (`[@?][refinement-modifier?][base][*?]`) defines which compositions are valid; the spec describes the pattern rather than enumerating every prefix. New primitive types, structural roles, or domain concerns are expressed by composing existing prefixes with Templates, Extensions, and vocabulary identifiers — never by adding new modifiers, operators, or bases. The closed vocabulary enables uniform parser behavior across all implementations. See Core Specification §6.

### 7.2 Character Classes

```ebnf
ALPHA           ::= [A-Z] | [a-z]

DIGIT           ::= [0-9]

HexDigit        ::= DIGIT | [a-f] | [A-F]

BinaryDigit     ::= "0" | "1"

OctalDigit      ::= [0-7]
```

### 7.3 Whitespace

```ebnf
WS              ::= WhitespaceChar+

WhitespaceChar  ::= " " | "\t" | "\n" | "\r"
```

Whitespace is a sequence of one or more space, tab, newline, or carriage-return characters. Other Unicode whitespace characters (non-breaking space, em space, etc.) are NOT recognized as whitespace by this grammar.

A line break does not terminate an element; element boundaries are exclusively bracket-delimited (§3).


---

## 8. Whitespace Rules

The whitespace rules of the GDX Standard Form are explicit and must be honored by parsers and serializers alike. Two whitespace categories are distinguished:

### 8.1 Sigil-Tight Boundaries

The sigils `&`, `#`, `--`, and `@` MUST be immediately followed by their content with no intervening whitespace. The following are malformed:

```
<STR &  1 : "x">       // whitespace after & is malformed
<STR #  schema:foo>    // whitespace after # is malformed
<REC address --  Foo>  // whitespace after -- is malformed
<EDGE #knows : @  Bob>  // whitespace after @ is malformed
```

### 8.2 Sigil-Preceding Whitespace

The sigils `&`, `#`, `--` (when introducing a Type field) MUST be preceded by whitespace separating them from the previous token. A parser encountering these sigils with no whitespace before them MUST treat the input as malformed.

```
<STR name&1>           // malformed: no whitespace before &
<STR name#foo>         // malformed: no whitespace before #
<STR : "x"--Foo>       // malformed: no whitespace before --
```

The `@` sigil, appearing only in Resource position after a Resource separator, is preceded by either the `:` separator (with optional whitespace) or whitespace within the Resource section. There is no separate rule for `@`-preceding whitespace beyond the Resource section's own whitespace handling.

### 8.3 Separator-Tolerant Whitespace

The `:` separator (introducing the Resource section) MAY have whitespace on either side. The following are all valid and equivalent:

```
<STR name:"Alice">
<STR name :"Alice">
<STR name: "Alice">
<STR name : "Alice">
```

### 8.4 Bracket-Adjacent Whitespace

Whitespace MAY appear immediately after `<` and immediately before `>`:

```
<  S name : "Alice"  >
```

This form is conforming but uncommon. Implementations writing canonical form do not include such whitespace.

### 8.5 Token-Separating Whitespace

Whitespace separates adjacent tokens that would otherwise concatenate. Specifically:

- Whitespace MUST separate the Prefix from a following Id when no sigil-introduced field intervenes (e.g., `<REC address>` not `<RECaddress>`).
- Whitespace MUST separate adjacent identifier tokens.
- Whitespace MUST appear between sibling child elements within a parent.

### 8.6 Suffix-Tight Whitespace

Numeric and bytes literal suffixes MUST be immediately adjacent to their numeric portion with no intervening whitespace:

```
<INT count : 25i8>     // valid
<I count : 25 i8>    // malformed: whitespace before i8
<F price : 3.14f32>  // valid
<B blob : 0x4a3f8c>  // valid: 0x is a prefix to the literal, no space
```


---

## 9. Open Items

This grammar specification has the following open items that should be resolved before its v1.0 publication.

### 9.1 Unicode Identifiers

The grammar restricts identifiers to ASCII characters. Internationalized identifiers (Cyrillic, CJK, etc.) are not supported in v1.

Resolution: Unicode identifier support is a v2 design discussion. The grammar may be extended in a future version.

### 9.2 Resolved Items

Items resolved since v0.1 and v0.2:

- **Multi-element documents.** Confirmed in Core Specification §7.2 rule 1: a document may contain multiple top-level elements, each independently a root-eligible kind (Object, Template, REL, or Package). The grammar reflects this in the Document production.

- **Comment syntax via REM.** Comments in GDX are expressed via `REM` Annotation elements (Core Specification §6.5.6). REM elements participate in the data model and survive round-trip; they are not stripped at the lexical layer. The grammar has no separate comment production because none is needed.

- **Transport-layer prefixes.** Package and Group transport constructs (Core Specification §9) added to the prefix vocabulary. Document grammar updated to include Package as a root-eligible type.

- **Float suffix on integer-form numerics.** Resolved in v0.3: integer-form literals (`5`, `100`, `5f32`, `100d`) are accepted in float-prefix positions. The grammar (§5.3) admits IntNumeric with optional FloatSuffix as a FloatLiteral alternative. The asymmetry (float-form literals are NOT accepted in integer positions) is preserved because integers are a subset of reals while reals are not a subset of integers.

- **Empty Resource after separator.** Resolved in v0.3: the form `<STR name : >` is accepted as an alternative null representation, equivalent to the bare form `<STR name>`. The grammar (§3) makes Resource optional within ResourceSection. The bare form is canonical for serialization; the empty-value form normalizes to the bare form on output.

- **Single-letter bool literals.** Resolved in v0.3: `t` and `f` (in any case) are accepted as boolean literals in addition to `true` and `false`. Disambiguation from identifiers is by Resource position. Conforming serializers normalize all forms to the lowercase spelled-out form on output.


---

## Appendix A — Complete Grammar

This appendix consolidates all productions defined above into a single normative grammar. The productions are listed in dependency order; a production references only productions defined above it in this listing.

```ebnf
// ===== Lexical Primitives =====

ALPHA           ::= [A-Z] | [a-z]
DIGIT           ::= [0-9]
HexDigit        ::= DIGIT | [a-f] | [A-F]
BinaryDigit     ::= "0" | "1"
OctalDigit      ::= [0-7]

WhitespaceChar  ::= " " | "\t" | "\n" | "\r"
WS              ::= WhitespaceChar+

Sign            ::= "+" | "-"

Base64Char      ::= ALPHA | DIGIT | "+" | "/"


// ===== Identifiers =====

IdStartChar     ::= ALPHA | "_"
IdContChar      ::= ALPHA | DIGIT | "_" | "-" | "." | "/" | ":"
Id              ::= IdStartChar IdContChar*


// ===== URI References =====

UriScheme       ::= ALPHA (ALPHA | DIGIT | "+" | "-" | ".")*
UriChar         ::= ALPHA | DIGIT | "_" | "-" | "." | "/" | ":"
                  | "?" | "#" | "[" | "]" | "@" | "!" | "$"
                  | "&" | "'" | "(" | ")" | "*" | "+" | ","
                  | ";" | "=" | "%" | "~"
UriChars        ::= UriChar+
UriReference    ::= UriScheme "://" UriChars


// ===== Prefix Vocabulary =====
// Each prefix follows the composition pattern:
//   [@?] [refinement-modifier?] [base] [*?]
// Prefix recognition is case-insensitive; parsers MUST accept any case.
// The grammar shows uppercase canonical case for clarity.

// --- Base prefixes ---

ObjectPrefix     ::= "OBJ"  | "O"
RecordPrefix     ::= "REC"  | "R"
EdgePrefix       ::= "EDGE" | "E"

PropertyPrefix   ::= "STR"   | "S"
                   | "INT"   | "I"
                   | "FLOAT" | "F"
                   | "BOOL"  | "L"
                   | "BYTE"  | "B"

PropertyBase     ::= "STR"   | "S"
                   | "INT"   | "I"
                   | "FLOAT" | "F"
                   | "BOOL"  | "L"
                   | "BYTE"  | "B"

RecordBase       ::= "REC"   | "R"

BasePrefix       ::= ObjectPrefix
                   | RecordPrefix
                   | PropertyPrefix

// --- Refinement-modifier compositions ---

QualifierComposition ::= "." PropertyPrefix
ExtensionComposition ::= "_" PropertyPrefix
EmbeddedComposition  ::= "%" PropertyPrefix
                       | "%" RecordPrefix
                       | "%" ObjectPrefix
NodeComposition      ::= "N" PropertyBase
                       | "N" RecordBase

RefinementBase   ::= QualifierComposition
                   | ExtensionComposition
                   | EmbeddedComposition
                   | NodeComposition

// --- Template prefixes (canonical only) ---

TemplatePrefix   ::= "DEFOBJ"
                   | "DEFREC"
                   | "DEFSTR"
                   | "DEFINT"
                   | "DEFFLOAT"
                   | "DEFBOOL"
                   | "DEFBYTE"

// --- Standalone declaration and organizational prefixes (canonical only) ---

RelPrefix        ::= "REL"
AnnotationPrefix ::= "REM"
BlockPrefix      ::= "BLK"

// --- Transport-layer prefixes (canonical only) ---

PackagePrefix    ::= "PKG"
GroupPrefix      ::= "GRP"

TransportPrefix  ::= PackagePrefix | GroupPrefix

// --- ModifierBase: anything that can carry an @ prefix or * suffix ---

ModifierBase     ::= RefinementBase
                   | TemplatePrefix
                   | RelPrefix
                   | AnnotationPrefix
                   | TransportPrefix
                   | BlockPrefix
                   | EdgePrefix
                   | BasePrefix

// --- Special operators ---

SetSuffix        ::= "*"
ReferencePrefix  ::= "@"

// --- Composed Prefix ---
// Reference operator cannot apply to another Reference (no @@).
// Set suffix may compose with most ModifierBase forms.

Prefix           ::= ReferencePrefix? ModifierBase SetSuffix?


// ===== Suffix Vocabularies =====

IntSuffix       ::= "i8" | "i16" | "i32" | "i64"
                  | "u8" | "u16" | "u32" | "u64"
                  | "bigint"

FloatSuffix     ::= "f32" | "f64"
                  | "d" | "d32" | "d64" | "d128"


// ===== Resource Literals =====

EscapeSequence  ::= "\\" '"'
                  | "\\" "\\"
                  | "\\" "n"
                  | "\\" "t"
                  | "\\" "r"
                  | "\\" "u" "{" HexDigit+ "}"

StringChar      ::= EscapeSequence
                  | ~('"' | '\')

StringLiteral   ::= '"' StringChar* '"'

DecimalInt      ::= Sign? DIGIT+
BinaryInt       ::= Sign? "0b" BinaryDigit+
OctalInt        ::= Sign? "0o" OctalDigit+
IntNumeric      ::= DecimalInt | BinaryInt | OctalInt
IntLiteral      ::= IntNumeric IntSuffix?

Exponent        ::= ("e" | "E") Sign? DIGIT+
FloatStandard   ::= Sign? DIGIT+ "." DIGIT+ Exponent?
                  | Sign? "." DIGIT+ Exponent?
                  | Sign? DIGIT+ Exponent
FloatKeyword    ::= "nan" | "inf" | "-inf"
FloatLiteral    ::= FloatStandard FloatSuffix?
                  | IntNumeric FloatSuffix
                  | IntNumeric              // valid only in float-prefix
                                            //   contexts (lexically same as
                                            //   IntLiteral; prefix discriminates)
                  | FloatKeyword

BoolLiteral     ::= BoolTrue | BoolFalse
BoolTrue        ::= "true" | "TRUE" | "True" | "t" | "T"
BoolFalse       ::= "false" | "FALSE" | "False" | "f" | "F"

HexBytes        ::= "0x" HexDigit+
Base64Bytes     ::= "0z" Base64Char+ "="*
BytesLiteral    ::= HexBytes | Base64Bytes

ReferenceLiteral    ::= IdentifierReference
                      | PathReference
                      | KeyReference
                      | IdPathReference
                      | IdKeyReference
IdentifierReference ::= "@" Id
PathReference       ::= "$" PathExpression
KeyReference        ::= "&" KeyToken
IdPathReference     ::= "@" Id "$" PathExpression
IdKeyReference      ::= "@" Id "&" KeyToken
PathExpression      ::= /* specified in GDX-Path.md §3-§4 */
KeyToken            ::= /* specified in GDX-Path.md §5 */

Resource        ::= StringLiteral
                  | IntLiteral
                  | FloatLiteral
                  | BoolLiteral
                  | BytesLiteral
                  | ReferenceLiteral


// ===== Fields =====

NumericKey      ::= DIGIT+
KeyContent      ::= NumericKey | Id
Key             ::= "&" KeyContent

RelContent      ::= UriReference | Id
Rel             ::= "#" RelContent

TypeContent     ::= Id
Type            ::= "--" TypeContent


// ===== Element Structure =====

IdentitySection ::= (WS Key)? (WS Rel)? (WS Id)?

ResourceSection ::= WS? ":" WS? Resource? (WS "--" TypeContent)?
                  | WS "--" TypeContent

Children        ::= (WS Element)+

ElementBody     ::= IdentitySection? ResourceSection? Children?

Element         ::= "<" WS? Prefix ElementBody WS? ">"


// ===== Document =====

TopLevelElement ::= Element       // Restricted by Core Spec §7.2 rule 1 to:
                                  //   ObjectElement | TemplateElement
                                  //   | RelElement | PackageElement

Document        ::= WS? TopLevelElement (WS+ TopLevelElement)* WS?
```

End of grammar.
