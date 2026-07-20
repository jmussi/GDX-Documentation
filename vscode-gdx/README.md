# GDX for Visual Studio Code

Syntax highlighting for **GDX (Graph Data eXchange)** Standard Form documents (`.gdx`), based on the GDX Standard Form Grammar Specification v0.11 (companion to Core Specification v0.20.0).

## Features

- **Prefix vocabulary** — the full closed vocabulary with the composition pattern `%? @? [._N|DEF]? base *?`: bases (`OBJ`, `REC`, `STR`, `INT`, `FLOAT`, `BOOL`, `BYTE`, `VAL`), minified single-letter forms, templates (`DEFOBJ` … `DEFBYTE`), declarations (`REL`, `REM`), organization and transport (`BLK`, `PKG`, `GRP`), refinement modifiers (`.` qualifier, `_` extension, `N` node), the `@` edge sigil, the `%` REF sigil, and the `*` Set suffix. Prefix matching is case-insensitive and positional (first token after `<`), so `<I I : 42>` highlights the prefix and the identifier differently.
- **Unknown prefixes flagged** — a token in prefix position outside the closed vocabulary (e.g. `<STRangle …>`) is scoped `invalid.illegal`, which most themes render with an error color.
- **Identity fields** — `&key` (numeric and named), `#predicate` (CURIEs and full URIs), and `--type` sigils each get distinct scopes.
- **Resource literals** — strings with the closed escape set (`\"`, `\\`, `\n`, `\t`, `\r`, `\u{…}`; anything else scoped as illegal), multi-line strings, decimal/binary/octal integers with the closed suffix set, floats with exponents/suffixes/`nan`/`inf`, booleans, and `0x`/`0z` bytes literals.
- **Reference literals** — `@Anchor`, `@Role:Anchor`, `$/path`, `&key`, and the composed `@Anchor$/path` / `@Anchor&key` forms.
- **REM as comments** — `REM` annotation elements (GDX's comment mechanism, which lives in the data model) are scoped as `comment.block`, including nested children.

## Installing

From this directory:

```sh
npm install -g @vscode/vsce   # once
vsce package                  # produces gdx-syntax-0.1.0.vsix
code --install-extension gdx-syntax-0.1.0.vsix
```

Or for development: open this folder in VS Code and press `F5` to launch an Extension Development Host, then open `samples/example.gdx`. Use the **Developer: Inspect Editor Tokens and Scopes** command to see the scope assigned to any character.

## Testing

Grammar snapshot tests use [`vscode-tmgrammar-test`](https://github.com/PanAeon/vscode-tmgrammar-test):

```sh
npm install
npm test              # verify tokens against tests/*.gdx.snap
npm run test:update   # regenerate snapshots after a grammar change
```

## Known limitations

TextMate grammars are regex-based approximations; this one is deliberately lenient where full parsing would be required:

- **Int vs. float discrimination is contextual in GDX** — a bare `5` under a `FLOAT` prefix is a float, but the grammar scopes all bare integers as `constant.numeric.integer`.
- **Single-letter booleans over-match** — `t`/`f` surrounded by whitespace are scoped as booleans even in identity position (e.g. an identifier actually named `t`).
- **The composition matrix is not enforced** — semantically malformed compositions such as `@REC`, `@OBJ*`, or `VAL*` still highlight (the closed vocabulary itself *is* enforced; unknown tokens are flagged).
- **No comment toggling** — GDX has no lexical comment syntax (comments are `REM` elements in the data model), so `Ctrl+/` is intentionally unbound.
- **No validation, navigation, or completion** — sigil-whitespace rules (§8), reference resolution, template conformance, etc. require a language server, which is out of scope for this extension.

## Layout

```
package.json                  extension manifest (language + grammar contributions)
language-configuration.json   brackets, auto-closing pairs, word pattern
syntaxes/gdx.tmLanguage.json  TextMate grammar
samples/example.gdx           kitchen-sink sample document
tests/                        grammar snapshot tests
```
