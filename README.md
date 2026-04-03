# SolidCAM GPPL IDE

Language support for SolidCAM GPPL postprocessor files (`.gpp`) in Visual Studio Code.

## Features

- **Semantic highlighting** — variables, procedures, parameters, types, scope keywords
- **Go to Definition** (F12 / Ctrl+Click)
- **Find All References** (Shift+F12)
- **Rename Symbol** (F2) — scope-aware, renames definition + all references
- **Document Symbols** — Outline panel, Breadcrumbs, Ctrl+Shift+O
- **Signature Help** — parameter hints on procedure calls
- **Hover** — symbol kind, type, scope, reference count
- **Auto-completion** — keywords, built-in functions, scope-aware symbols, procedure snippets after `call`
- **Folding Ranges** — proc/endp, if/endif, while/endw, comment blocks
- **Document Formatting** (Shift+Alt+F) — automatic indentation and operator spacing
- **Diagnostics** — parse errors + semantic warnings (redeclaration)
- **System Catalog** — 950+ system variables and 94 system procedures with descriptions

## Formatting Rules

The formatter uses the GPPL parse tree (CST) for precise formatting. Trigger with **Shift+Alt+F** or via the command palette.

**Indentation** (2 spaces per level):
- Procedure body (`@proc` ... `endp`) — indent level 1
- `if` / `elseif` / `else` / `endif` — keywords at current level, body +1
- `while` / `endw` — keyword at current level, body +1
- Nested blocks increase indent cumulatively

**Operator spacing** (one space on each side):
- Assignment: `x = 1`
- Comparison: `==`, `!=`, `<>`, `<=`, `>=`, `<`, `>`
- Arithmetic: `+`, `-`, `*`, `/`
- Word operators: `and`, `or`, `not`, `eq`, `ne`, `lt`, `gt`, `le`, `ge`

**Exceptions:**
- Power `^` — no spaces: `x^2`
- Unary `-` / `+` — no space after sign: `z = -x + 2`
- Colon `:` — no spaces (format specifier): `result:'+3.3'`
- Comma `,` — space after, no space before: `f(a, b)`

**Preserved as-is:**
- Empty lines
- Comments
- String literals

> The formatter only runs on files with no parse errors.

## Installation

### From VS Marketplace

Search for **SolidCAM GPPL IDE** in the Extensions panel.

### Offline (VSIX)

Download the `.vsix` file from [GitHub Releases](https://github.com/anzory/solidcam-gppl-ide/releases) and install:

```
code --install-extension anzory.solidcam-gppl-ide-X.Y.Z.vsix
```

## License

The VSCode extension client is licensed under MIT.
The language server binary is proprietary. See [LICENSE](LICENSE) for details.
