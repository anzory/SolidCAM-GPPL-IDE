# SolidCAM GPPL IDE

Language support for SolidCAM GPPL postprocessor files (`.gpp`) in Visual Studio Code.

> **Early Preview** — the extension is in active development. Some features may be incomplete. Your [feedback and bug reports](https://github.com/anzory/SolidCAM-GPPL-IDE/issues) are very welcome!

![Hero](https://github.com/anzory/SolidCAM-GPPL-IDE/raw/master/images/hero.png)

---

## Features

### Semantic Highlighting & Hover

All syntax highlighting is provided by the language server — variables, procedures, parameters, types, keywords, operators, built-in functions and system variables are color-coded by meaning, not just pattern matching.

Hover over any symbol to see its kind, type, scope, and reference count. System variables show a description from the SolidCAM reference.

### Auto-Completion

Context-aware suggestions for keywords, built-in functions, your own variables and procedures, plus **950+ SolidCAM system variables** and **94 system procedures** with descriptions.

![Completion](https://github.com/anzory/SolidCAM-GPPL-IDE/raw/master/images/completion.gif)

### Go to Definition & Find All References

**F12** / **Ctrl+Click** — jump to where a variable or procedure is defined.
**Shift+F12** — see all usages across the file.

![Go to Definition](https://github.com/anzory/SolidCAM-GPPL-IDE/raw/master/images/goto-definition.gif)

### Rename Symbol

**F2** — rename a variable or procedure across the entire file. Scope-aware: local renames stay local.

![Rename](https://github.com/anzory/SolidCAM-GPPL-IDE/raw/master/images/rename.gif)

### Diagnostics

Parse errors with clear, context-aware messages. Semantic checks: undeclared variables, redeclarations, system variable conflicts.

![Diagnostics](https://github.com/anzory/SolidCAM-GPPL-IDE/raw/master/images/diagnostics.png)

### Document Formatting

**Shift+Alt+F** — automatic indentation and operator spacing based on the parse tree.

![Formatting](https://github.com/anzory/SolidCAM-GPPL-IDE/raw/master/images/formatting.gif)

### Signature Help

Parameter hints when calling procedures — see expected argument types.

![Signature Help](https://github.com/anzory/SolidCAM-GPPL-IDE/raw/master/images/signature-help.png)

### And More

- **Document Outline** — see the structure of your postprocessor (Ctrl+Shift+O)
- **Breadcrumbs** — navigate by procedure name
- **Folding** — collapse `proc/endp`, `if/endif`, `while/endw`, comment blocks
- **System Catalog** — built-in reference for SolidCAM 2022 system variables and procedures

---

## Formatting Rules

The formatter uses the GPPL parse tree (CST) for precise formatting. Only runs on files with no parse errors.

| Rule           | Example                            |
| -------------- | ---------------------------------- |
| Indentation    | 2 spaces per nesting level         |
| Assignment     | `x = 1`                            |
| Comparison     | `x == 1`, `x != 2`, `x <= 10`      |
| Arithmetic     | `x + 1`, `y * 2`                   |
| Word operators | `x eq 1`, `a and b`                |
| Power `^`      | `x^2` (no spaces)                  |
| Unary sign     | `z = -x + 2` (no space after sign) |
| Colon `:`      | `result:'+3.3'` (no spaces)        |
| Comma `,`      | `f(a, b)` (space after)            |

Empty lines, comments, and string literals are preserved as-is.

---

## Installation

### From VSIX (Offline)

1. Download `solidcam-gppl-ide-x.y.z.vsix` from [GitHub Releases](https://github.com/anzory/SolidCAM-GPPL-IDE/releases)
2. In VSCode: Extensions panel → `...` → **Install from VSIX...**

Or via command line:

```
code --install-extension solidcam-gppl-ide-x.y.z.vsix
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
