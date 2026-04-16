# SolidCAM Postprocessor IDE

[![Marketplace Version](https://img.shields.io/visual-studio-marketplace/v/anzory.vscode-gppl-ide?label=marketplace)](https://marketplace.visualstudio.com/items?itemName=anzory.vscode-gppl-ide)
[![Installs](https://img.shields.io/visual-studio-marketplace/i/anzory.vscode-gppl-ide)](https://marketplace.visualstudio.com/items?itemName=anzory.vscode-gppl-ide)
[![Downloads](https://img.shields.io/visual-studio-marketplace/d/anzory.vscode-gppl-ide)](https://marketplace.visualstudio.com/items?itemName=anzory.vscode-gppl-ide)
[![Rating](https://img.shields.io/visual-studio-marketplace/r/anzory.vscode-gppl-ide)](https://marketplace.visualstudio.com/items?itemName=anzory.vscode-gppl-ide&ssr=false#review-details)

Language support for SolidCAM GPPL postprocessor files (`.gpp`) in Visual Studio Code.

> **Early Preview** — the extension is in active development. Some features may be incomplete. Your [feedback and bug reports](https://github.com/anzory/SolidCAM-GPPL-IDE/issues) are very welcome!

![Hero](https://github.com/anzory/SolidCAM-GPPL-IDE/raw/master/images/hero.png)

---

## Features

### Semantic Highlighting & Hover

All syntax highlighting is provided by the language server — variables, procedures, parameters, types, keywords, operators, built-in functions and system variables are color-coded by meaning, not just pattern matching.

Hover over any symbol to see its kind, type, scope, and reference count. System variables show a description from the SolidCAM reference. **48 built-in functions** (`abs`, `substr`, `atan2`, `date`, …) show their full signature, return type, localized description, and usage examples.

### Auto-Completion

Context-aware suggestions for keywords, built-in functions, your own variables and procedures, plus **950+ SolidCAM system variables** and **94 system procedures** with descriptions. Built-in functions expand as snippets with tab-stops over parameter names and show localized documentation in the completion popup. Descriptions follow VS Code's display language — English, Russian, and German are shipped.

![Completion](https://github.com/anzory/SolidCAM-GPPL-IDE/raw/master/images/completion.gif)

### Go to Definition & Find All References

**F12** / **Ctrl+Click** — jump to where a variable or procedure is defined.
**Shift+F12** — see all usages across the file.

![Go to Definition](https://github.com/anzory/SolidCAM-GPPL-IDE/raw/master/images/goto-definition.gif)

### Rename Symbol

**F2** — rename a variable or procedure across the entire file. Scope-aware: local renames stay local.

![Rename](https://github.com/anzory/SolidCAM-GPPL-IDE/raw/master/images/rename.gif)

### Diagnostics

Parse errors with clear, context-aware messages. Semantic checks: undeclared variables, redeclarations, system variable conflicts, **local-shadows-global** warnings, **missing `@init`** detection, and **globals-outside-`@init`** hints. File encoding check (`GPPL3001`) warns about characters not representable in the target ANSI encoding. Every diagnostic carries a stable code (`GPPL1xxx` syntax, `GPPL2xxx` semantic, `GPPL3xxx` encoding).

![Diagnostics](https://github.com/anzory/SolidCAM-GPPL-IDE/raw/master/images/diagnostics.png)

### Quick Fixes

**Ctrl+.** on a diagnostic to apply a one-click fix:

- Insert missing `endp` / `endif` / `endw`
- Close an unterminated string literal
- Insert a missing `}` for a code generation block
- Rename a variable that conflicts with a system name
- Remove a duplicate variable declaration
- Rename a local variable that shadows a global (`x` → `x_local`)
- Generate a missing `@init` procedure stub
- Move a global declaration into `@init`

### Snippets

Type a prefix and press **Tab** to expand common constructs with tab-stops at the places you care about:

| Prefix                   | Expands to                       |
| ------------------------ | -------------------------------- |
| `procedure` / `proc`     | `@name ... endp`                 |
| `global` / `local`       | `global integer name` (type choice via tab) |
| `if` / `ife`             | `if cond then ... endif`         |
| `ifelse` / `ifel`        | `if ... then ... else ... endif` |
| `while` / `wh`           | `while cond ... endw`            |
| `call`                   | `call @procedure_name`           |
| `region` / `#region`     | `;#region ... ;#endregion`       |
| `cg` / `codegen`         | `{ nl, 'text' }`                 |

### Document Formatting

**Shift+Alt+F** — automatic indentation and operator spacing based on the parse tree.

![Formatting](https://github.com/anzory/SolidCAM-GPPL-IDE/raw/master/images/formatting.gif)

### Signature Help

Parameter hints when calling procedures and built-in functions — see expected argument types, optional parameters, and localized descriptions. Supports all **48 built-in functions** and user-defined `@`-procedures.

![Signature Help](https://github.com/anzory/SolidCAM-GPPL-IDE/raw/master/images/signature-help.png)

### Clickable `inc`

**Ctrl+Click** on the filename in `inc "other.gpp"` opens the referenced file. Paths are resolved relative to the directory of the current `.gpp` file (absolute paths are honoured as-is). Both quote styles and forward-slash subdirectories work.

### And More

- **Document Outline** — see the structure of your postprocessor (Ctrl+Shift+O)
- **Breadcrumbs** — navigate by procedure name
- **Folding** — collapse `proc/endp`, `if/endif`, `while/endw`, comment blocks, and custom `;#region NAME` / `;#endregion` markers (C#-style; works across or inside procedures, no semantic meaning)
- **Smart call completion** — after `call `, completion of your procedures expands into a snippet with tab-stops over the declared parameter names
- **System Catalog** — built-in reference for SolidCAM 2022 system variables and procedures

---

## Formatting Rules

The formatter uses the GPPL parse tree (CST) for precise formatting. Parse errors no longer block it — valid lines are reformatted even when the file contains syntax errors. Lines where the parser lost or reordered tokens are preserved verbatim.

| Rule            | Example                              |
| --------------- | ------------------------------------ |
| Indentation     | 2 spaces per nesting level           |
| Assignment      | `x = 1`                              |
| Comparison      | `x == 1`, `x != 2`, `x <= 10`        |
| Arithmetic      | `x + 1`, `y * 2`                     |
| Word operators  | `x eq 1`, `a and b`                  |
| Power `^`       | `x^2` (no spaces)                    |
| Unary sign      | `z = -x + 2` (no space after sign)   |
| Colon `:`       | `result:'+3.3'` (no spaces)          |
| Comma `,`       | `f(a, b)` (space after)              |
| Code-gen braces | `{ nl, 'G1' }` (spaces inside `{ }`) |

Empty lines, comments, and string literals are preserved as-is.

---

## File Encoding

SolidCAM reads `.gpp` files in the system's ANSI codepage. The extension defaults `.gpp` to Windows-1252 and warns (`GPPL3001`) about characters that cannot be saved in the selected encoding.

### For Cyrillic postprocessors (Windows-1251)

Add to your User or Workspace `settings.json`:

```json
"[gppl]": {
  "files.encoding": "windows1251"
},
"gppl.encoding": "windows1251"
```

`files.encoding` tells VS Code how to read/write the file; `gppl.encoding` tells the language server which characters to accept. Both should match.

### Switching encoding of an open file

Use VS Code's built-in commands:

- **`Change File Encoding → Reopen with Encoding`** — re-read the file bytes with a different codepage (useful when a file opens with mojibake).
- **`Change File Encoding → Save with Encoding`** — re-save the file in a different codepage.

The extension intentionally does not offer per-character Quick Fixes — re-encoding the whole file is almost always the right action.

---

## Language / Localization

The extension is fully localized in **English** (default), **Russian**, and **German**.
The active language follows VS Code's display language (`vscode.env.language`) — install the matching Language Pack and the extension follows automatically. A reload (`Developer: Reload Window`) is needed for the language server to pick up a newly changed locale.

What gets translated:

- **System catalog** — all **948 system variable** descriptions and **94 system procedure** descriptions (hover tooltips and completion item details).
- **Diagnostics** — parser error messages, semantic diagnostics, encoding warnings.
- **UI strings** — extension settings, command titles, hover labels, completion details.

No configuration is required.

---

## Installation

### From VS Marketplace (Recommended)

Search for **SolidCAM Postprocessor IDE** in the Extensions panel (Ctrl+Shift+X), or install directly:

```
code --install-extension anzory.vscode-gppl-ide
```

Or visit the [marketplace page](https://marketplace.visualstudio.com/items?itemName=anzory.vscode-gppl-ide).

### From VSIX (Offline)

1. Download `vscode-gppl-ide-x.y.z.vsix` from [GitHub Releases](https://github.com/anzory/SolidCAM-GPPL-IDE/releases)
2. In VSCode: Extensions panel → `...` → **Install from VSIX...**

Or via command line:

```
code --install-extension vscode-gppl-ide-x.y.z.vsix
```

### Requirements

- Windows (SolidCAM is Windows-only)
- VSCode 1.75+

---

## Feedback

This extension is built for postprocessor developers. If you work with GPPL, your input is invaluable:

- **Bug reports** — [open an issue](https://github.com/anzory/SolidCAM-GPPL-IDE/issues)
- **Feature requests** — what would save you the most time?
- **General impressions** — is this heading in the right direction?

---

## License

The VSCode extension client is licensed under MIT.
The language server binary is proprietary. See [LICENSE](https://github.com/anzory/SolidCAM-GPPL-IDE/blob/master/LICENSE) for details.
