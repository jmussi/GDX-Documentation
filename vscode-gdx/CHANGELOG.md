# Changelog

## 0.2.0

- **Breaking (scope rename):** the Rel field's TextMate scopes are renamed, following the Core Specification v0.27.0 rename of the `Rel` field to `Predicate` (entry 66):
  - `keyword.operator.sigil.rel.gdx` → `keyword.operator.sigil.predicate.gdx`
  - `entity.other.attribute-name.rel.gdx` → `entity.other.attribute-name.predicate.gdx`

  Themes or extensions targeting the old scope names must retarget. The grammar rule `rel-field` is renamed `predicate-field`; its match pattern is unchanged, so highlighting behavior for `#curie` and `#scheme://uri` field content is identical. No GDX document content changes — the `#` sigil is unchanged.

## 0.1.0

- Initial release: TextMate grammar for GDX Standard Form (Grammar Specification v0.11, Core Specification v0.20.0), language configuration, and sample document.
