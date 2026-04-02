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
- **Diagnostics** — parse errors + semantic warnings (redeclaration)

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
