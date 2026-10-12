# GDX Binary Form — Specification

- **Status:** Draft v0.1 (companion to GDX Core Specification v0.29.0, working draft). Sections 1 and 2 only.
- **Date:** 2026-10-12
- **Companion to:** GDX Core Specification v0.29.0
- **Authors:** Julian Mussi


## Abstract

This document defines the GDX Binary Form, a compact encoding of the GDX data model for machine exchange. A Binary
Form **frame** carries one element tree. The form preserves the data model defined in the Core Specification (§3)
with no information loss (Core §10.1), and a reader can decode it without knowing the prefix vocabulary.

This draft states the **preservation contract**: what a frame carries and what a round trip through it keeps. The
byte-level encoding, the code registries, framing and limits follow in later sections, once the encoding has been
chosen between two candidates by measurement.


## Status of This Document

Working draft v0.1. Section 1 (Introduction) and section 2 (Preservation Contract) are written, and section 2 is
normative. Sections 3 to 6 are placeholders.
The encoding replaces the unspecified binary encodings of the reference implementation, none of which has been
released. When this specification is released, the form is marked **v1**, and its registry codes and field
layout never change after that.


## Table of Contents

1. [Introduction](#1-introduction)
2. [Preservation Contract](#2-preservation-contract)
3. [Encoding](#3-encoding) *(to be written)*
4. [Registries](#4-registries) *(to be written)*
5. [Framing and Features](#5-framing-and-features) *(to be written)*
6. [Decoding Limits and Refusal](#6-decoding-limits-and-refusal) *(to be written)*


## 1. Introduction

### 1.1 Purpose and Position

The Binary Form is a realization of the GDX data model (Core §1.1), as Standard Form and Compact Form are. Standard
Form remains the canonical form for reading, writing by hand, and canonical comparison (Core Appendix A.5). The
Binary Form is for machines: actor messages between hosts, replication, and any exchange where parsing text is
wasted work.

Its first obligation is to carry the model faithfully across implementations. Compactness and speed come second.

### 1.2 Design Principles

- **The model, not a runtime.** The form encodes the value-kinds of Core §3.3 and the representation tags of Core
  §4.2. It never encodes how one implementation happens to store a value (a 32-bit or 64-bit integer type, a
  string held as a character buffer, a platform's decimal layout).
- **Values describe themselves.** A reader identifies every value's kind from the value's own header. The prefix
  constrains a value when it is validated, never when it is decoded, so adding a prefix to the Core Specification
  never changes how bytes are read.
- **Refuse, don't drop.** A reader that meets something it does not understand refuses the frame. It never skips
  something that changes meaning, and never substitutes a default.
- **Frames decode on their own.** Nothing a reader needs is held over from an earlier frame.

### 1.3 What Binary Form Is Not

- It is not a content digest. Identity is computed over typed values (see §2.9). Two frames with different bytes
  may carry the same content.
- It is not a storage layout. A store may keep elements in its own form (Core §10.2) and use the Binary Form only
  at its edges.
- It is not a container for large binary media. Large byte content belongs in a blob store, referenced from the
  tree.


## 2. Preservation Contract

The serialization-conformance contract of Core Appendix A.2 applies to the Binary Form in full. This section
states how it applies, and what the Binary Form preserves beyond it.

### 2.1 What a Frame Carries

A frame carries **exactly one element tree**. The tree need not be a document in the sense of Core §7.2: its
root may have any prefix, so an actor message rooted in a Record travels as readily as a stored Object. A frame
reader checks no composition rule of Core §7, root eligibility (§7.2, rule 1) included. The receiver does: a store
checks that a root may be persisted, and a validator checks the rest.

Several Objects that travel together are carried as the children of one Package root (Core §6.7). A receiver that
persists them unpacks the Package. Two Packages are two frames.

Frames delimit themselves, so a stream of frames is the frames written one after another. A reader can read one
frame and report how many bytes it consumed, or read exactly one frame and refuse anything after it.

### 2.2 What Is Preserved

For every element, a round trip through a frame preserves:

1. **The Prefix.**
2. **Id, Predicate, Name and Type**: whether each is present, and its exact characters (ordinal comparison).
   See §2.3 and §2.4.
3. **The Resource:**
   - whether the element has a value (the null state of Core §3.3.3 round-trips as the null state);
   - the value-kind (Core §3.3);
   - the value itself, exactly (Core §3.3, §4.2);
   - the representation tag, if any (Core §4.2: integer widths, float and decimal representations, bytes
     encodings).
4. **The ordered children**, recursively. This includes the order of Set members: Sets are ordered collections in
   the data model.
5. **Annotation (REM) elements** in their original positions.
6. **The metadata block** (Core §4.8): see §2.7.

Beyond the minimum of Core Appendix A.2, the Binary Form also preserves:

- **Negative zero.** A float `-0.0` reads back as `-0.0`.
- **Decimal scale.** A float tagged as decimal, such as `1.50d`, reads back as `1.50d`, not `1.5d`.

The content digest folds both of these (§2.9); the frame does not.

### 2.3 Empty Identity Fields

An identity field or Type is absent or a non-empty identifier (Core §3.1). A writer given `""` for one of these
fields encodes it as absent, and a reader never produces `""` for them.

A string Resource is different. The empty string is a value, distinct from the null state (Core §3.3.3), and is
preserved.

### 2.4 Identifiers

Id, Predicate, Name and Type hold identifiers (Core §4.4). A writer refuses a tree whose identity field or Type does
not match the identifier grammar, and a reader refuses a frame that holds one. Without this check a frame could
carry a Name that Standard Form cannot write, and a round trip through Standard Form would lose it.

### 2.5 Value-Kinds

A frame encodes the six value-kinds of Core §3.3. A representation tag (Core §4.2) chooses how a number is
represented; it does not make a different kind.

| Kind | What is encoded |
|---|---|
| `int` | the mathematical integer, at any magnitude. A width tag (`i8` … `u64`, `bigint`) is carried as the tag. |
| `float` | the real number in the representation its tag names: IEEE 754 binary64 for `f64`, binary32 for `f32`, and for a decimal tag (`d`, `d32`, `d64`, `d128`) a coefficient and a base-10 exponent, which together keep the scale. An untagged float takes the tag its canonical text would carry, as the content digest defines it (`d` for a decimal value, `f64` otherwise); a value that representation cannot hold exactly is refused, never rounded (Core §3.3.1). |
| `bool` | true or false |
| `string` | the sequence of Unicode scalar values, encoded as UTF-8 |
| `bytes` | the octet sequence. The text form's bytes encoding (`0x`, `0z`) is carried as the tag. |
| `reference` | the reference literal (Core §4.2.6), as its text |

A string is a sequence of Unicode scalar values (Core §3.3). A writer refuses a string holding an unpaired
surrogate, which UTF-8 cannot carry, and a reader refuses invalid UTF-8. Text is not Unicode-normalized.

A reader that cannot hold a value refuses the frame rather than approximating it (Core §3.3.1). For example, a
runtime whose decimal type has fewer digits than a `d128` value refuses that value; it does not round it.

### 2.6 What Is Not Carried

- **Change-tracking state** (whether an element is new, modified or deleted). A delta is a separate structure.
- **NaN payloads.** Standard Form has one `nan`, so a frame carries one NaN.

### 2.7 Metadata

A frame carries the metadata block of Core §4.8 for any element that has one: every pair it is given, in order,
with no key treated specially. Each value is one of the five literal kinds the block admits: string, integer,
float, boolean, bytes. There are no references, no nesting and no elements.

Metadata is outside the model (Core §4.8.1). It does not take part in model equality or in the content digest, and
a frame that drops it is still model-faithful. A writer SHOULD carry it and a reader SHOULD keep it, because
pipelines such as delta streaming rely on it.

### 2.8 Unknown Content

The form has no optional features in v1. A reader refuses a frame that holds anything it does not recognize: an
unknown prefix code, value-kind, representation tag or feature.

One header bit is **reserved** for optional features that a reader may skip. No such feature is defined. A v1
reader that finds the bit set, with features it does not know, refuses the frame.

### 2.9 Conformance and the Content Digest

An implementation conforms when, for every tree, the tree it reads from a frame equals the tree it wrote, element
for element, under §2.2.

The content digest is an additional check. It is **gdx-digest-v1**, defined in the reference implementation's
repository (`GDX/docs/GDX_Spec_Digest.md` in probable-eureka) and computed over typed values, not over any
realization's bytes. A round trip through a frame preserves the digest. The digest is coarser than §2.2: it folds
`-0` into `+0`, normalizes decimal scale and ignores metadata, so an equal digest does not prove a faithful round
trip; the equality above does.

The reverse does not hold either. Frames carrying the same content may differ in their bytes, for example in
metadata, so a frame's bytes are not an identity.


## 3. Encoding

*To be written.* The element and value encoding.

## 4. Registries

*To be written.* The prefix codes (a base form plus modifier bits that mirror the text grammar), value-kind codes
and representation-tag codes. None of them is tied to the numbering of any implementation's types.

## 5. Framing and Features

*To be written.* The frame header (magic, version, feature bits), and how a frame ends.

## 6. Decoding Limits and Refusal

*To be written.* The limits every reader enforces on untrusted input (depth, element count, value length, frame
length), and the refusals it reports.
