# Changelog

## [0.5.1] — 2026-04-07

### Fixed
- README image links now use absolute GitHub URLs (marketplace doesn't support relative paths)

## [0.5.0] — 2026-04-06

### Added
- **Full Semantic Highlighting** — all syntax highlighting now provided by LSP instead of TextMate
  - Keywords, operators, numbers, strings, comments, built-in functions
  - TextMate grammar reduced to minimal (comments + strings for pre-LSP fallback)
  - No more conflicts between TextMate and LSP highlighting
- **Humanized Diagnostics** — clear, context-aware error messages instead of raw ANTLR output
  - Rule invocation stack for precise context (e.g., "Missing 'endif' - 'if' block is not closed before 'endp'")
  - Missing closing parenthesis detected from grammar context
  - Unterminated string literal detection
  - Type keyword set recognition (e.g., "Expected a type keyword (integer, numeric, logical, string)")
- **Cascading Error Suppression** — lexer errors no longer produce misleading parser error cascades

### Changed
- System variable redeclaration severity changed from Warning to Error (matches SolidCAM runtime behavior)
- Parser service now returns token stream for full semantic token coverage

## [0.4.0] — 2026-04-03

### Added
- **System Catalog** — 950+ system variables and 94 system procedures from SolidCAM reference
  - Completion with type and description
  - Hover with summary from catalog
  - Semantic tokens: `defaultLibrary` + `readonly` modifiers for system symbols
  - Lazy registration: system symbols resolved on first reference in user code
- **Document Formatting** (Shift+Alt+F) — CST-based formatter
  - Indentation: 2 spaces per level (proc body, if/while blocks)
  - Operator spacing: `=`, `==`, `+`, `-`, `*`, `/`, comparisons
  - Power `^` without spaces: `x^2`
  - Unary sign without trailing space: `-x`
  - Preserves empty lines, comments, string literals
  - Only formats files with no parse errors

### Changed
- System variable descriptions cleaned: 207 duplicate entries merged into single descriptions
- Embedded JSON resources (variables + procedures) loaded at server startup

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
