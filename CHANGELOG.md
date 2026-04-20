# Changelog

## [1.0.2] — 2026-04-20

### Security
- **XXE / XML bomb hardening in VMID parser.** `GpplVmidParser.TryParse`
  previously used `XDocument.Load(path)` without explicit `XmlReaderSettings`,
  relying on the framework's default DTD policy. Malicious `.vmid` files
  (which are shipped with third-party postprocessors and therefore untrusted)
  could theoretically:
  - Read local files via external entities (`<!ENTITY x SYSTEM "file:///…">`)
  - Exhaust memory via billion-laughs recursive entity expansion
  - Cause DoS via oversized or deeply nested XML
  `XmlReader` is now configured with `DtdProcessing = Prohibit`,
  `XmlResolver = null`, `MaxCharactersFromEntities = 0`, and a 10 MB document
  size cap. A pre-read `FileInfo.Length` check rejects oversized files before
  opening the stream.
- **Path traversal hardening in `inc` document links.** The `inc "filename"`
  directive previously accepted arbitrary paths — absolute, relative with
  parent-directory segments (`..\..\..`), UNC (`\\server\share`), and
  subfolders — and rendered each existing file as a clickable link. This
  enabled two attack surfaces:
  - **Information disclosure via File.Exists probing** — a malicious `.gpp`
    with many `inc` directives pointing at system files could expose which
    files exist on the victim's machine through visible/invisible links.
  - **NTLM hash leak via UNC File.Exists** — clicking (or even parsing) a
    UNC path triggers an SMB authentication exchange that may leak the user's
    NTLM hash to an attacker-controlled server.
  Per the GPPL specification, `inc "abc"` accepts only a basename in the
  same directory as the host file. The v0.7.0 extension to subfolders and
  absolute paths was incorrect and has been removed. Rejected patterns:
  path separators (`/`, `\`), parent-directory segments (`..`), drive
  letters (`c:foo`), UNC prefixes (`//`, `\\`), dot-only names, and any
  character in `Path.GetInvalidFileNameChars()`. A canonical post-resolution
  check ensures the final path remains inside the host's directory.

### Tests
- **9 new security tests**: XXE external-entity rejection, billion-laughs
  DoS with 2-second timeout budget, 11 MB oversized file rejection, normal
  file still parses (`GpplVmidParserTests`); path traversal, UNC, drive
  letter, invalid chars, dot-only name rejection (`GpplDocumentLinkHandlerTests`).
  Total: **226 tests, all green**.

## [1.0.1] — 2026-04-20

### Fixed
- **Marketplace badges in README.** The four badges at the top of the README
  (Version, Installs, Downloads, Rating) showed a "retired badge" placeholder
  because shields.io discontinued its `visual-studio-marketplace/*` endpoints
  (upstream Microsoft API changes). Switched to the community-maintained
  `vsmarketplacebadges.dev` service, which renders all four correctly.

## [1.0.0] — 2026-04-16

### Added
- **VMID variable support.** Variables from the machine-specific `.vmid` file
  (resolved by convention: `foo.gpp` → `foo.vmid` in the same directory) are
  now recognized by the language server. VMID variables appear in completion
  and hover with their GUI name and description from the XML, and participate
  in type checking. Registration is lazy — a VMID variable is added to the
  symbol table on first occurrence in code.
- **GPPL2007 — Undeclared identifier.** Warning when a variable or procedure
  name is used without being declared. If the file contains `inc` directives,
  severity is downgraded to Hint and the message notes the identifier may be
  defined in an included file.
- **GPPL2008 — Type mismatch on assignment.** Warning when the right-hand side
  of an assignment is incompatible with the declared type of the left-hand side.
  String is isolated from the numeric hierarchy; `integer`, `numeric`, and
  `logical` are all mutually compatible.
- **Expression type resolver.** New ANTLR4 Visitor (`GpplExpressionTypeResolver`)
  infers the type of the right-hand side of assignments — distinguishing
  `integer` vs `numeric` for arithmetic, recognizing string concatenation,
  logical expressions, built-in function return types, and variable types from
  the symbol table.
- **VMID completion & hover.** Unused VMID variables appear in the completion
  list with their type and GUI name. Hover shows "VMID variable" label with
  description from the `.vmid` file.
- **Localized messages** for all new diagnostics (EN/RU/DE).

### Changed
- Removed "Early Preview" label from README — the extension is now stable.
- VMID cache no longer stores null results, so a `.vmid` file added after
  the `.gpp` file is opened is picked up on the next edit.

## [0.9.0] — 2026-04-16

### Added
- **GPPL2004 — Local shadows global.** Warning when a local variable hides
  a global variable with the same name (case-insensitive). Quick Fix:
  rename to `name_local`.
- **GPPL2005 — Missing `@init_post` procedure.** Warning when the file contains
  procedures but none of them is `@init_post`. Quick Fix: generate an `@init_post`
  stub before the first procedure.
- **GPPL2006 — Global declared outside `@init_post`.** Info-level hint when a
  `global` declaration appears in a procedure other than `@init_post` (fires
  only when `@init_post` exists — otherwise GPPL2005 covers it). Quick Fix:
  move the declaration line into `@init_post` (single-variable declarations
  only).
- **Localized diagnostic messages** for all three new codes (EN/RU/DE).

## [0.8.0] — 2026-04-16

### Added
- **Built-in function catalog.** All **48 built-in functions** (28 numeric,
  16 string, 4 logical) are now described in a structured JSON catalog with
  base type information, parameter signatures, and per-locale text overlays
  (English, Russian, German). The catalog is the single source of truth for
  completion, hover, and signature help — replacing the hardcoded arrays that
  existed since v0.7.3.
- **Hover for built-in functions.** Hovering over `abs`, `atan2`, `substr`,
  `date` and any other built-in now shows a rich tooltip with the full
  signature (`name(type param, ...) -> returns`), category label, localized
  description, remarks, and usage examples in a fenced `gppl` block.
  Parenless functions like `date` render as `date -> string` (no parentheses).
  Optional colon parameters render as `[:type name]`.
- **Signature Help for built-in functions.** Typing `abs(`, `atan2(5,` etc.
  now shows parameter hints with the active parameter highlighted — the same
  experience that was previously available only for user-defined `@`-procedures.
  Parenless functions (`date`, `time`) correctly return no signature.
- **Localized completion documentation.** Built-in function completion items
  now include a localized summary in the documentation popup (previously only
  the category label was shown).

### Changed
- **Completion handler migrated to catalog.** The six hardcoded collections
  (`NumericFunctions`, `StringFunctions`, `LogicalFunctions`,
  `BuiltinSignatures`, `ParenlessBuiltins` and the two helper methods) are
  replaced by a single loop over `GpplBuiltinFunctionCatalog.Functions`.
  Adding a new built-in function now requires only a JSON entry — no C# code
  changes.
- **SignatureHelp handler refactored.** `FindCallContext` no longer requires
  an `@`-prefix. The handler branches: names starting with `@` look up
  user-defined procedures in the symbol table; plain names look up the
  built-in catalog.

## [0.7.3] — 2026-04-16

### Added
- **Snippet completion for built-in functions.** Picking a built-in
  function from the completion list now inserts a parameterised call
  skeleton with Tab-stops — e.g. `substr` expands to
  `substr(${1:string}, ${2:from}, ${3:len})`, `atan2` to
  `atan2(${1:y}, ${2:x})`, `abs` to `abs(${1:number})`. Parameter names
  are sourced from the GPPL specification. Optional parameters (e.g. the
  `[val]` of `instr`, `[, k]` of `replace`, `:format` of `tostr`) are
  intentionally omitted from the snippet — experienced users can add
  them by hand, and beginners aren't forced to delete a placeholder they
  don't need.
- **Logical functions in completion.** `active`, `change`, `even`, and
  `odd` were missing from the completion list entirely. They are now
  offered (also with snippet skeletons) alongside numeric and string
  functions.

### Fixed
- **Parenless built-ins.** `date` and `time` are parameterless and, per
  the GPPL grammar, are called **without** parentheses (`date`, not
  `date()`). The completion now inserts just the name in that case.

## [0.7.2] — 2026-04-16

### Fixed
- **Procedure completion regressions introduced by v0.7.1.** Two connected
  bugs on procedure auto-complete after the v0.7.1 `wordPattern` change:
  - Typing `call @a`, then accepting `@arc`, produced `call @@arc` (double
    `@`) — because VSCode's new word-at-cursor is just `a`, but `insertText`
    was the full `@arc`.
  - Typing `call @` (just the trigger character, no letters) showed an
    empty "No suggestions" popup.
  The fix: every procedure completion item now ships an explicit `TextEdit`
  whose range covers only the letters after `@` (the `@` itself stays in
  the buffer). `NewText` is set to the name **without** `@` when `@` is
  already in the buffer, or **with** `@` for the Ctrl+Space case on a bare
  `call ` line. This gives `call @arc` in every scenario —
  `call @a|` + accept, `call @|` + accept, and `call |` + Ctrl+Space +
  accept. The empty-list bug was caused by the earlier attempt to include
  `@` inside the `TextEdit` range: VSCode derives its filter-query from
  `document[range.start..cursor]`, so including `@` made the query `@a`,
  which no longer matched the stripped `filterText` (just `arc`).
- **Server crash on `call @` with no identifier.** The symbol-table builder
  dereferenced `ProcedureName()` without a null-check when the user was
  mid-typing `call @|`. ANTLR error-recovery produces a
  `procedureNameDeclaration` node whose `ProcedureName()` returns `null` in
  that state, and the resulting `NullReferenceException` propagated out of
  the document-update path — causing the completion response to come back
  empty. Added the missing null guard.

## [0.7.1] — 2026-04-16

### Fixed
- **Procedure completion after `@` filter prefix.** Typing `call @a` used to
  hide procedures whose name started with `a` right after `@`
  (`@abort_posting`, `@arc`, `@axes_sync`, …) while keeping those whose name
  had `a` after an underscore (`@wc_abs_rel`, `@get_arc_center`, …). Root
  cause: VSCode's fuzzy scorer treats `_`, `-`, `.`, space and a few others
  as word separators but **not `@`** — so the `a` right after `@` was
  classified as an inner character rather than a word start, and VSCode's
  default `matchOnWordStartOnly` mode dropped the item. The fix is two
  synchronized edits:
  - `language-configuration.json#wordPattern` no longer includes `@` as an
    optional prefix. This aligns with VSCode's default `wordSeparators`,
    which already counted `@` as a separator — the previous `@?…` pattern
    was actually fighting the rest of the editor.
  - Completion items for procedures (both user-defined and system-catalog)
    now set `filterText` equal to the name without the `@` prefix. Display
    (`label`) and insertion (`insertText`) remain `@name(...)` — only the
    fuzzy-match input changes.
  Typing `call @a` now shows all procedures whose name starts with `a`,
  ranked by match quality, as expected.

## [0.7.0] — 2026-04-15

### Added
- **Clickable `inc` directives.** `Ctrl+Click` (or `Cmd+Click`) on the string
  literal in `inc "filename.gpp"` now opens the referenced file. Paths are
  resolved relative to the directory of the current `.gpp` file; absolute
  paths are honoured as-is. Both quote styles (`"..."` and `'...'`) and
  forward-slash subdirectories (`inc "lib/util.gpp"`) are supported. Missing
  files silently produce no link — this milestone is navigation only; parsing
  of included files, cross-file symbol resolution, and `@init_inc` diagnostics
  remain planned for the multi-file workspace milestone.
- **Smart procedure-call completion.** After typing `call `, completion items
  for your own procedures expand into a full snippet with tab-stops over the
  declared parameter names — e.g. selecting `@worker` for
  `@worker(integer x, numeric y)` inserts `@worker(${1:x}, ${2:y})`.
  Procedures without parameters insert just the name. Outside the `call`
  context procedures are still listed as plain identifiers so the snippet
  expansion never fires in positions where it would be syntactically invalid.

## [0.6.8] — 2026-04-15

### Added
- **Base snippets** for common GPPL constructs. Type the prefix and press
  `Tab` to expand with tab-stops at the interesting positions.

  | Prefix                        | Expands to                    |
  | ----------------------------- | ----------------------------- |
  | `procedure` / `proc`          | `@name ... endp`              |
  | `region` / `#region`          | `;#region ... ;#endregion`    |
  | `if` / `ife`                  | `if cond then ... endif`      |
  | `ifelse` / `ifel`             | `if ... then ... else ... endif` |
  | `while` / `wh`                | `while cond ... endw`         |
  | `call`                        | `call @procedure_name`        |
  | `cg` / `codegen`              | `{ nl, 'text' }`              |

  Snippets for `global` / `local` with a type-choice placeholder were
  already available since v0.6.4.

  Context-aware "smart" snippets (e.g. auto-filling procedure arguments
  from the symbol table on call-site completion) are planned for v0.7.0.

## [0.6.7] — 2026-04-15

### Fixed
- **Russian and German hover descriptions for system variables.** The RU/DE
  JSON catalogs were physically present on disk but silently excluded from
  the compiled server assembly — MSBuild was interpreting the `.ru.` / `.de.`
  filename segments as culture designators and routing those files into
  satellite-assembly generation instead of embedding them. As a result,
  `FindVariable(...).Summary` always returned the English text regardless of
  the requested locale, even though UI strings, parser messages, and
  procedure descriptions were translated correctly.
  - Resources are now embedded explicitly with `<WithCulture>false</WithCulture>`,
    guaranteeing all three locales end up in the main DLL.
  - Catalog loader rewritten to an overlay model: the English file is the
    single source of truth for the full variable list and types; locale
    files only override the `Summary` field. This matches the on-disk
    schema (EN is a list of `{name, type, summary}`, RU/DE are `name → text`
    dictionaries) and prevents the type information from being lost when a
    locale is applied.
- **7 regression tests** (`GpplSystemCatalogLocaleTests`) covering: EN baseline,
  RU overlay (Cyrillic present), DE overlay (umlauts/German stopwords), type
  preservation after overlay, RU procedure descriptions, unknown-locale
  fallback, and total variable count sanity check.

## [0.6.6] — 2026-04-15

### Added
- **Mandatory space inside code-generation braces.** Formatter now requires a
  space after `{` and before `}` in `{ ... }` blocks. Empty blocks are written
  as `{ }`.
  ```gppl
  { nl, ';TOOL: ' + tool_name }    ; was: {nl, ';TOOL: ' + tool_name}
  ```

### Changed
- **Formatter no longer bails out on parse errors.** Previously a single
  syntax error anywhere in the file silently disabled the formatter. Now
  the formatter runs on the best-effort parse tree and, for each line,
  keeps the formatted version when the lexer preserved all of the line's
  tokens; lines where tokens were lost or reordered by ANTLR error-recovery
  are preserved verbatim. Net effect: valid procedures around a broken one
  are still reformatted.

### Fixed
- **Line-structure preservation in error-recovered trees.** The formatter
  now syncs its output with source token line numbers, so collapsed
  statements (e.g. `if a==` merged with the next line during recovery)
  no longer shift subsequent lines in the formatted output.

## [0.6.5] — 2026-04-14

### Added
- **Full internationalization (i18n).** The extension now supports English
  (default), Russian, and German. The display language is detected
  automatically from VS Code's `locale` setting.
  - **System catalog translations:** all 948 system variable descriptions
    and 94 system procedure descriptions are now available in Russian and
    German — visible in hover tooltips and completion item details.
  - **Server-side UI strings:** parser error messages, semantic diagnostics,
    encoding warnings, hover labels, and completion details are localized.
  - **Client-side strings:** extension settings, command titles, and error
    messages are localized via `package.nls.{locale}.json` and
    `vscode.l10n.t()`.
- **Locale forwarding.** The VS Code display language is passed to the
  language server via `initializationOptions.gppl.locale`, enabling
  locale-aware responses from the first request.

### Changed
- Removed `.2022` suffix from system catalog JSON files (the data is not
  version-year-specific).

## [0.6.4] — 2026-04-09

### Added
- **Doc-comments from trailing comments.** Any trailing comment on a
  declaration line is now exposed as hover documentation and shown in
  completion items. Works for global/local variables, procedures, and
  applies to all variables in a multi-declaration:
  ```gppl
  global integer chuck_bypass  ; bypass chuck lock (lathe ops)
  global numeric x, y, z       ; coordinates  (applies to x, y, z)

  @set_units                   ; switches between mm and inches
  endp
  ```
  System symbols keep their documentation from the built-in catalog and
  are not overridden by user comments.
- **`;#region` / `;#endregion` folding markers** (C#-style). These are
  plain GPPL comments and can wrap any range — several procedures,
  a part of a procedure body, or a block of global code. Purely visual,
  no semantic meaning. Permissive parsing: `; #region` (with spaces) also
  works. Implemented both client-side (`language-configuration.json`) and
  server-side (`textDocument/foldingRange`) — the latter is required
  because VS Code does not merge folding sources and the server response
  replaces client-side markers once the LSP session is ready.
- **Snippet expansion for `global` / `local`.** Picking these keywords
  from autocomplete now inserts a snippet with a type-choice placeholder
  (`integer | numeric | logical | string`) followed by a name placeholder,
  ensuring the declaration is always syntactically complete.

### Changed
- **Syntax highlighting: word-form logical operators** (`and`, `or`, `not`)
  are now colored as keywords, matching how they read visually. The
  symbolic `!` stays as an operator.
- **Formatter preserves trailing-comment alignment.** Previously the
  formatter collapsed all whitespace before a trailing comment to a single
  space, breaking hand-aligned comment columns. Now the original column of
  each trailing comment is preserved; if the reformatted code overflows
  that column, the formatter falls back to one space to avoid overlap.
  Block alignment (aligning comments to a shared column across adjacent
  lines) is intentionally not implemented.

### Fixed
- **Extension no longer writes `.vscode/settings.json` into user folders.**
  Earlier versions forced `solidcam-gppl.trace.server` via
  `ConfigurationTarget.Workspace` on every activation, which physically
  created or modified `.vscode/settings.json` in whatever folder the user
  had opened. On folders under git this produced a permanently dirty
  working tree. Trace is now driven via `client.setTrace()` after
  `client.start()` — an in-memory `$/setTrace` LSP message with no disk
  side-effects. In production, the trace setting is read directly from
  user settings (default `off`) and is never overwritten. Existing
  `.vscode/settings.json` files created by older versions are left alone;
  users can delete them safely.

## [0.6.3] — 2026-04-09

### Added
- **Encoding diagnostic `GPPL3001`.** Warns when the file contains characters
  that cannot be saved in the target ANSI encoding. The diagnostic points at
  the first offending character and reports the total count.
- **Setting `gppl.encoding`** (`windows1252` | `windows1251`, default
  `windows1252`). Selects the target ANSI encoding used by `GPPL3001`.
  Changes take effect immediately — all open documents are re-diagnosed.
- **Default editor encoding for `.gpp`** set to Windows-1252 via
  `configurationDefaults` (`[gppl]` → `files.encoding: windows1252`,
  `files.autoGuessEncoding: false`). Users working with Cyrillic
  postprocessors should override both settings:
  ```json
  "[gppl]": { "files.encoding": "windows1251" },
  "gppl.encoding": "windows1251"
  ```
  The diagnostic intentionally does not offer a Quick Fix — use VS Code's
  built-in `Save with Encoding` / `Reopen with Encoding` to switch.

## [0.6.2] — 2026-04-08

### Fixed
- **Grammar: false-positive `GPPL1099 "Unexpected token"` in code generation
  blocks.** Two real-world SolidCAM postprocessor patterns were rejected by
  the parser:
  - **Multiple formatted outputs in one `{ ... }` block separated by
    juxtaposition** — e.g. `{ nl, ';'tool_number:'<T>z2.0(p)!=' ': 'tool_name_comment'' }`.
    Root cause: `unconditionalCodeGenerationMember: unformattedOutput | formattedOutput`
    was not left-factored, and SLL prediction mode (introduced in v0.6.1) cannot
    resolve such ambiguities. Rewritten as
    `unconditionalCodeGenerationMember: unformattedOutput formatOfOutput?`.
  - **Explicit `+` concatenation inside `{ ... }` blocks** — e.g.
    `{ nl, ';HEADER ' + upper(version) + ' ' + date }`. The grammar previously
    only allowed juxtaposition (`'a' 'b' c`); the `Plus` operator was missing.
    `codeGenerationStringExpression` now accepts `(Plus? stringExpressionMember)*`
    so both juxtaposition and explicit `+` work.
- **Test harness: parser tests now use `PredictionMode.SLL`** to match
  production. Previously they ran in implicit LL mode and silently masked
  SLL-specific grammar bugs (the v0.6.1 release was published assuming
  35/35 tests guaranteed SLL correctness — that assumption was wrong).

### Added
- **5 regression tests** in `GpplParserTests` covering both grammar bugs,
  including the exact strings from `test/fixtures/large_file.gpp:515 / 528 / 1108`,
  plus an anti-regression test ensuring the old juxtaposition style still parses.

### Changed
- **Decision 029** documents the left-factoring rule and the codegen `+`
  fix; **decision 028** corrected with a note that the original "tests pass
  identically" claim was based on a faulty test harness.

## [0.6.1] — 2026-04-08

### Changed
- **Parser: SLL-only mode (no LL fallback).** Performance investigation
  (2026-04-08) showed that the previous SLL → LL fallback strategy degraded
  4× on large files with syntax errors (median parse 285 ms → 65 ms on a
  4768-line file with 20 errors). LL re-parse with full-context lookahead
  was the sole bottleneck; SLL with `DefaultErrorStrategy` is sufficient for
  GPPL — all 35 diagnostic and code-action tests pass identically. See
  decision 028.

### Fixed
- **Production server log file is now actually created.** Previously the Serilog
  minimum level was set to `Error`, silently dropping all `[perf]` and
  diagnostic `Information` entries — the log file never appeared on disk.
- **Development server logs are now visible in the VS Code Output channel.**
  Switched the dev logger to a stderr-based console provider so the
  `LanguageClient` automatically routes them into the channel.
- **"Open Server Logs" command** now finds the correct log file — the client
  filename pattern (`yyyyMMdd`) is aligned with Serilog's rolling default.

### Added
- **Performance instrumentation** on the LSP hot path: `[perf] UpdateDocument`
  (parse + symbol table), `[perf] PublishDiagnostics`, and
  `[perf] SemanticTokens` log entries with timings, line counts, and token
  counts. Always-on observability for diagnosing future regressions.

## [0.6.0] — 2026-04-08

### Added
- **Code Actions / Quick Fixes** (Ctrl+.) for parser and semantic diagnostics:
  - `GPPL1001` Close unterminated string literal
  - `GPPL1011` Insert missing closing `}` for code generation block
  - `GPPL1020` / `GPPL1021` / `GPPL1022` Insert missing `endp` / `endif` / `endw`
  - `GPPL2001` Rename variable conflicting with system name (`*_custom` suggestion)
  - `GPPL2002` / `GPPL2003` Remove duplicate variable declaration (single-ident or full-line)
- **Stable diagnostic codes** — every parser and semantic diagnostic now carries a
  `GPPL1xxx` (syntax) or `GPPL2xxx` (semantic) code that Quick Fixes target.
- **Code generation block awareness** in parser error messages — unclosed `{ ... }`
  blocks now report `Missing closing brace '}'` instead of a generic ANTLR error.
- **Semantic token modifiers** — `boolean` (true/false) and `interrupt`
  (abort/exit/break/return) for richer themable highlighting.

### Changed
- **Parser performance: SLL → LL fallback strategy.** Valid documents are now
  parsed in a single fast SLL pass; only files with syntax errors fall back to
  the full LL prediction with error recovery. Noticeable speedup on large files.
- **Debounced `didChange` (150 ms)** — rapid typing no longer triggers a parse
  on every keystroke; handlers briefly serve the previous snapshot, diagnostics
  settle shortly after the user stops typing.
- **System catalog lookups** no longer allocate via `ToLowerInvariant()` —
  dictionaries already use `OrdinalIgnoreCase`, removing per-lookup allocations.
- **Symbol table** precomputes the procedure list at build time (`GetProcedures`
  is now O(1) instead of linear scan).
- **TextMate grammar** — string literals now terminate at end-of-line, preventing
  unclosed strings from bleeding colorization into following lines (the LSP
  diagnostic + Quick Fix take over from there).
- **Boolean constants** are now highlighted as read-only default-library
  variables (consistent with system constants) instead of numbers.
- README — added VS Marketplace badges and marketplace install instructions.

### Fixed
- `NoViable alternative` errors inside code generation blocks and procedure
  argument lists now produce accurate hints (missing `}` / `)`) based on the
  parser rule invocation stack instead of echoing the raw token.

## [0.5.4] — 2026-04-07

### Changed
- Renamed `displayName` from "SolidCAM GPPL IDE" to "SolidCAM Postprocessor IDE" (marketplace display name was already taken)

## [0.5.3] — 2026-04-07

### Changed
- Renamed extension `name` from `solidcam-gppl-ide` to `vscode-gppl-ide` (marketplace name was already taken)

## [0.5.2] — 2026-04-07

### Fixed
- Corrected `repository.url` in package.json to point to the actual GitHub repo

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
