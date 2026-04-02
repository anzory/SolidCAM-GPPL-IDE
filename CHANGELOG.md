# Changelog

## [0.3.0] — 2026-04-03

### Added
- Document Symbols (Outline panel, Breadcrumbs, Ctrl+Shift+O)
  - Hierarchical: procedures → parameters + variables (global & local)
- Signature Help on procedure calls (trigger: `(` and `,`)
- Completion after `call` — only procedures with snippet insertion
- Document Highlight (definition + references highlighting)
- Folding Ranges (proc/endp, if/endif, while/endw, comment blocks)
- FullRange for procedures (accurate folding and Outline)

### Changed
- Completion: procedures show signature in detail
- FindProcedureAtPosition uses FullRange instead of heuristic
- Prod mode resets trace to `off` if left from dev session

## [0.2.0] — 2026-04-02

### Added
- Semantic highlighting (variables, procedures, parameters, types, scope keywords)
- Go to Definition (F12)
- Find All References (Shift+F12)
- Rename Symbol (F2) with scope awareness
- Symbol Table infrastructure (single-parse architecture)
- Scope-aware completion (user-defined symbols visible in current scope)
- Semantic diagnostics (variable redeclaration)
- Hover shows symbol kind, type, scope, reference count
- `endp` highlighted as function, `global`/`local` as namespace, types as class

### Changed
- Hover and completion now use cached symbol table instead of re-parsing
- Dev trace via workspace configuration (persistent across reloads)

## [0.1.0] — 2026-04-02

### Added
- Syntax highlighting for `.gpp` files
- Diagnostics (parse errors)
- Hover information
- Auto-completion
- Command "SolidCAM GPPL: Open Logs"
