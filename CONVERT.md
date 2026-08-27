# Instructions for conversion from WebAssembly Spec to GitHub Markdown

### Purpose

Converting the WebAssembly Core Specification (generated `_spectec/*.rst` bundle), in mostly reStructuredText format with embedded LaTeX, section-by-section, into clean, consistent, copy-paste-friendly GitHub-Flavored Markdown (GFM), suitable both as an LLM reference and for human reading.

### Sources used

`WebAssembly/spec/document/core/_spectec/<part>/*.rst` (authored prose + SpecTec-expanded formal content, one file per section).

### Folder structure

* `reference` contains the sources in `.rst` format (read-only). Copied from original `WebAssembly/spec/document/core/_spectec/*`
* `markdown` contain the corresponding target outputs in `.md` format. Its directory tree and file names directly mirror those in `reference`

## 1. RST → what is rendered (the LLM must NOT guess)

| RST construct | Rendered? | Action |
|---|---|---|
| `Title` + `=====` | yes (heading) | → `# Title` |
| `Heading` + `-----` | yes | → `## Heading` |
| `Sub` + `~~~~~` | yes | → `### Sub` |
| `Para` + `^^^^^` | yes | → `#### Para` |
| `Name` + `.....` (5th level, **dotted** underline) | yes | → `#### Name` |
| `:ref:`X <t>`` *used as a heading* (dotted underline) | yes | → `#### X` (keep the link text only. Drop the anchor — see §5.1) |
| prose paragraphs | yes | keep as a single block of text; **never** split a paragraph across lines (one-sentence-per-line) — see §5.6 |
| `* item` | yes (bullet list) | → `* item` (nest by indentation. Never use `-`) |
| `1. item` / `#. item` | yes (ordered list) | → `1. item` (always explicit `1.`/`2.`/`3`. Never `#.`) |
| `a. item` | yes (lettered sublist) | → nested ordered list |
| `.. note::` | **yes** (admonition) | → `> **Note:** …` blockquote |
| `:ref:`Text <t>`` (inline) | yes (hyperlink) | → `Text` (or `[Text](#t)`) |
| `\|Name\|` / `\|Name\|_` | yes (substitution reference) | → `Name` (the `\|…\|` is a *substitution*. The trailing `_` only adds a hyperlink defined outside this tree — render as plain text, see §5.5) |
| `:math:`x`` | yes (inline math) | → inline code / Unicode (§3) |
| `:code:`x`` | yes (inline code) | → `` `x` `` |
| `.. math::` block | yes (display math) | → fenced `text` block (§2, §6) |
| `.. code-block:: lang` / `::` literal | yes (code) | → fenced `lang`/`text` block (§5.3) |
| `[#name]_` | yes (footnote ref) | → `[^name]` (GitHub-Flavored Markdown footnote, §5.4) |
| `.. [#name]` | yes (footnote def) | → `[^name]: text` (§5.4) |
| `.. index::` | **no** (index only) | **drop** |
| `pair:` / `single:` | **no** (index only) | **drop** |
| `.. _name:` | **no** (anchor target) | **drop** |
| `.. _name: <url>` | rarely used here | drop unless the URL literally appears in the file (link targets are usually substitutions defined elsewhere) |
| `.. only::` / `.. toctree::` | **no** (Sphinx/build) | **drop** |

Rules of thumb for the LLM:

* Drop every `.. ` directive unless it is `.. note::`, `.. math::`, `.. code-block::`, or a footnote definition `.. [#name]`
* Drop `.. _…:` anchor targets (keep `.. _name: <url>` only if the URL is present in the file)
* Drop `.. index::`, `pair:`, `single:`, `.. only::`, `.. toctree::`. Substitution references `|Name|` / `|Name|_` are *not* directives — render them as the name text (§5.5).

## 2. Display math (`.. math::`) → fenced code block

Never keep LaTeX. Convert each `.. math::` into a `text` block.

**(a) Grammar** (contains `::=`): write compact productions.
* Drop: `\begin{array}…`, column specs, `&`, `\\`, `\quad\Rightarrow\quad`, `\mathtt{}`.
* `::=` stays.
* `|` stays as alternative.
* `\mathtt{0x..}` → `` `0x..` `` (or plain `0x..`).
* `\quad\Rightarrow\quad` → `=>`.
* Metavariables `{\mathit{x}}` → `x`
* Bindings `x{:}{\Bfoo}` → `x : Bfoo`
* Sequences `{x^\ast}` → `x*`
* Optionals `{X^?}` → `X?`
* `\epsilon` → `ε`
* Apply the Notation table (§4) to all `\MACRO` tokens.
* The `=>` arrows within a production block: Ensure exactly four spaces before `=>`, so the arrow does not visually appear as if it is attached to a member
* **Blank line between productions.** When several productions are emitted into the *same* `text` code block, separate each production from the next with a blank line: before every production header line `NAME ::=` that follows a non-blank line (a body line, or another production), insert one blank line. This matches the official rendered HTML, which shows visible spacing between successive productions. Do **not** add a blank line before the first production (immediately after the ```` ```text ```` fence) or anywhere one already exists. (Automated fix: insert a blank line before any column-0 `::=` line that is preceded by a non-blank, non-fence line, applied only inside fenced code blocks — see the global-correction script.)

**(b) Inference rule** (a `.. math::` containing `\frac`, `\vdash`, `~>`, or a fractional premise/conclusion — see §6 for the full rendering algorithm): write the rule name (if present, from the nearest `:math:`/`:ref:` heading), then premises (one per line), a `────` divider, then the conclusion, preserving every `if …` side condition.

Every alternative, byte, constructor, and condition MUST be preserved.

## 3. Inline math (`:math:`…`)

* Identifier/symbol only (e.g. `\BOT`, `\mathtt{0x4E}`) → inline code: `` `bot` ``, `` `0x4E` ``.
* Connectors render in ASCII: `->`, `>=`, `*`.
* Only if a fragment is a genuine subscript/relation formula AND you want
  MathJax: wrap in `$…$`. Otherwise prefer inline code (most portable).

## 4. Notation cheat-sheet (macro → Markdown)

### 4.1 General macro rule
Every TeX macro `\NAME` denotes a single token. The converted markdown shows the **plain** nonterminal name with **no organizational category prefix** — strip the leading `B`, `T`, `S`, `H`, `C`, `M`, `XI`, `XA`, or `X`/`XX` from every macro and keep the remainder. (The human-visible specification renders these without the prefixes.)
* `B` (binary) and `T` (text syntax) → strip the prefix; keep the rest. E.g. `\Bnumtype`→`numtype`, `\Tsource`→`source`.
* `S` / `H` / `C` / `M` / `XI` / `XA` (store / heap / context / module-instance / export / address **field labels**) → strip the prefix; keep the remainder. E.g. `\STAGS`→`TAGS`, `\CTYPES`→`TYPES`, `\HITYPE`→`TYPE`, `\XINAME`→`NAME`.
* `X` or `XX` followed by a kind → the kind name lowercased (`func`, `table`, `mem`, `global`, `tag`). E.g. `\XXFUNC`→`func`, `\XTFUNC`→`func`.
Everything else (value/heap constructors, opcodes, instance nonterminals, judgments) → lowercase word, e.g. `\F64`→`f64`, `\NOP`→`nop`, `\CONST`→`const`, `\moduleinst`→`moduleinst`, `\EXPORT`→`export`.

### 4.2 Category-prefixed nonterminals (strip the prefix)
* **Binary nonterminals** (strip `B`): `\Babsheaptype`→`absheaptype`, `\Bheaptype`→`heaptype`, `\Bnumtype`→`numtype`, `\Bvectype`→`vectype`, `\Breftype`→`reftype`, `\Bvaltype`→`valtype`, `\Bresulttype`→`resulttype`, `\Bcomptype`→`comptype`, `\Bfieldtype`→`fieldtype`, `\Bstoragetype`→`storagetype`, `\Bpacktype`→`packtype`, `\Brectype`→`rectype`, `\Bsubtype`→`subtype`, `\Blimits`→`limits`, `\Btagtype`→`tagtype`, `\Bglobaltype`→`globaltype`, `\Bmemtype`→`memtype`, `\Btabletype`→`tabletype`, `\Bexterntype`→`externtype`, `\Bsection`→`section`, `\Bbyte`→`byte`, `\Bblocktype`→`blocktype`, `\Bcatch`→`catch`, `\Bcustom`→`custom`, `\Bname`→`name`, `\Btype`→`type`, `\Bimport`→`import`, `\Bfuncsec`→`funcsec`. (The bottom type `\BOT`renders as `bot` per §4.6.)
* **Index / integer / list *sorts*** (also strip `B`): `\Btypeidx`→`typeidx`, `\Bfuncidx`→`funcidx`, `\Btableidx`→`tableidx`, `\Bmemidx`→`memidx`, `\Bglobalidx`→`globalidx`, `\Btagidx`→`tagidx`, `\Belemidx`→`elemidx`, `\Bdataidx`→`dataidx`, `\Bfieldidx`→`fieldidx`, `\Bexternidx`→`externidx`, `\Blocalidx`→`localidx`, `\Blabelidx`→`labelidx`, `\BuN`/`\BsN`→`uN`/`sN` (e.g. `u32`, `s33`), `\Blist(X)`→`list(X)` (or `X*`).
* **Text syntax nonterminals** (strip `T`):
  * lexical: `\Tsource`→`source`, `\Tchar`→`char`, `\Ttoken`→`token`, `\Tkeyword`→`keyword`, `\Tstring`→`string`, `\Tid`→`id`, `\Tidchar`→`idchar`, `\Treserved`→`reserved`
  * numeric: `\TuNX`→`uN`, `\TsNX`→`sN`, `\TfNX`→`fN`
  * type forms: `\Tstruct`→`struct`, `\Tarray`→`array`, `\Tfunc`→`func`, `\Tarrow`→`->`.
* **Record / context / store / instance field labels** (strip `S`/`H`/`C`/`M`/`XI`/`XA`): `\STAGS`→`TAGS`, `\SGLOBALS`→`GLOBALS`, `\SMEMS`→`MEMS`, `\STABLES`→`TABLES`, `\SFUNCS`→`FUNCS`, `\SDATAS`→`DATAS`, `\SELEMS`→`ELEMS`, `\CTYPES`→`TYPES`, `\HITYPE`→`TYPE`, `\MITAGS`→`TAGS`, `\MIGLOBALS`→`GLOBALS`, `\MIMEMS`→`MEMS`, `\XINAME`→`NAME`, `\XIADDR`→`ADDR`, `\XATAG`→`TAG`, `\XAGLOBAL`→`GLOBAL`, `\XAMEM`→`MEM`.
* **External-kind macros** (`\X…` / `\XX…`) → the kind name lowercased
* `\PAGE`→`page` (full list with the value/heap constructors in §4.3)

### 4.3 Value / heap / reference constructors (lowercase WAT)
Value/heap constructors use WAT-style lowercase. Keep them consistent:
* **Numeric / vector value types**: `\F64`→`f64`, `\F32`→`f32`, `\I64`→`i64`, `\I32`→`i32`, `\V128`→`v128`
* **Heap type constructors**: `\EXN`→`exn`, `\ARRAY`→`array`, `\STRUCT`→`struct`, `\I31`→`i31`, `\EQT`→`eq`, `\ANY`→`any`, `\EXTERN`→`extern`, `\FUNCT`→`func`
* **Bottom / none / no- forms**: `\NONE`→`none`, `\NOEXTERN`→`noextern`, `\NOFUNC`→`nofunc`, `\NOEXN`→`noexn`
* **Reference / mutability / type forms**: `\REF`→`ref`, `\NULL`→`null`, `\TMUT`→`mut`, `\TARRAY`→`array`, `\TSTRUCT`→`struct`, `\TFUNC`→`func`, `\TREC`→`rec`, `\TSUB`→`sub`, `\TFINAL`→`final`, `\Tarrow`→`->`
* **External kind & page**: `\PAGE`→`page`, `\XTFUNC`/`\XTTABLE`/`\XTMEM`/`\XTGLOBAL`/`\XTTAG`→`func`/`table`/`mem`/`global`/`tag`

### 4.4 Instruction / opcode macros (lowercase WAT, dots where standard)
Opcode macros render as their WAT mnemonic in lowercase, inserting `.` where the standard mnemonic is dotted:
* **Control / parametric**: `\NOP`→`nop`, `\UNREACHABLE`→`unreachable`, `\SELECT`→`select`, `\DROP`→`drop`, `\BLOCK`→`block`, `\LOOP`→`loop`
* **Branches**: `\BR`→`br`, `\BRIF`→`br_if`, `\BRTABLE`→`br_table`, `\BRONNULL`→`br_on_null`, `\BRONNONNULL`→`br_on_non_null`, `\BRONCAST`→`br_on_cast`, `\BRONCASTFAIL`→`br_on_cast_fail`
* **Calls / exceptions**: `\CALL`→`call`, `\CALLINDIRECT`→`call_indirect`, `\RETURNCALL`→`return_call`, `\RETURNCALLINDIRECT`→`return_call_indirect`, `\THROW`→`throw`, `\CATCH`→`catch`, `\Rethrow`→`rethrow`
* **Constants / arithmetic**: `\CONST`→`const`, `\VCONST`→`vconst`, `\ADD`→`add`
* **Reference / value address forms**: `\REF`→`ref`, `\NULL`→`null`, `\REFNULLADDR`→`ref.null`, `\REFI31NUM`→`ref.i31`, `\REFSTRUCTADDR`→`ref.struct`, `\REFARRAYADDR`→`ref.array`, `\REFFUNCADDR`→`ref.func`, `\REFEXNADDR`→`ref.exn`, `\REFHOSTADDR`→`ref.host`, `\REFEXTERN`→`ref.extern`

### 4.5 Instance & abstract-syntax nonterminals (mixed-case → lowercase word)
Mixed-case / lowercase single-word macros are nonterminals or meta-variables. Render as the word itself:
* **Instance nonterminals**: `\moduleinst`→`moduleinst`, `\funcinst`→`funcinst`, `\meminst`→`meminst`, `\tableinst`→`tableinst`, `\globalinst`→`globalinst`, `\taginst`→`taginst`, `\datainst`→`datainst`, `\eleminst`→`eleminst`, `\structinst`→`structinst`, `\arrayinst`→`arrayinst`, `\exninst`→`exninst`
* **Runtime / abstract syntax**: `\store`→`store`, `\frame`→`frame`, `\config`→`config`, `\admininstr`→`admininstr`
* **Misc. identifiers**: `\name`→`name`, `\externidx`→`externidx`, `\EXPORT`→`export`, `\NAME`→`name`

### 4.6 Judgment & relation symbols
* `\vdash` and all `\vdash⟨X⟩` (e.g. `\vdashinstr`, `\vdashinstrtype`, `\vdashcomptype`, `\vdashheaptype`, `\vdashtype`) → `|-` (drop the ⟨X⟩ qualifier word. It is recoverable from the section heading).
* Matching / subtyping infix `\sub⟨X⟩match` (e.g. `\subheaptymatch`, `\subnumtypematch`) → `<:`
* `|-` (already in source as text in some rules) → `|-`
* `~>` → reduction (keep as `~>`)
* `\rightarrow` / `\to` → `->` (the only arrow form. Never `→`)
* `\quad\Rightarrow\quad` → `=>`
* `\approx` → `≈` (e.g. `\approxexpanddt` → `≈`)
* `\oplus` → `⊕`
* `\Ldotdot` → `..` (range, e.g. `[n .. m]`)
* `\epsilon` → `ε`
* `\BOT` → `bot`
* `\geq` → `>=` (never `≥`)
* `{\mathit{x}}` → `x`, `x^\ast` → `x*`, `X^?` → `X?`
* `\sNX` / `\sN` → signed-N representation (e.g. `s33`). Inline form reads as "N-bit signed"

### 4.7 LaTeX text & formatting (strip)
* `\mathrm{…}` → upright text as-is
* `\mathrm{U{+ }XX}` → `U+XX` (Unicode code point — drop the `{}`/`+` braces)
* `\mathsf{…}` → the word in lowercase (e.g. `\mathsf{select}`→`select`, `\mathsf{i{\scriptstyle 32}}`→`i32`)
* `\mbox{…}` / `\texttt{…}` → the inner text
* `\mbox{‘\texttt{x}’}` → `'x'` (Unicode quotes `‘’` become straight `'`)
* `\dots` / `\ldots` → `...`
* Spacing/formatting commands are dropped: `\quad`, `\qquad`, `\kern`, `\allowbreak`, `\displaystyle`, `\begin{array}`/`\end{array}`, `@{…}`, `[0.8ex]`, `[3ex]`, `\multicolumn{…}{…}{…}` (render its content as a side-condition line, see §2 / §6)

### 4.8 Sub-/superscripts, primes & record access
* `^\ast` → `*`, `^?` → `?` (sequence / optional)
* Other superscripts `^{…}` → keep as a `(…)` qualifier or `^{…}`
* `x^{i<n}` → `x^(i<n)`
* Single-token subscripts merge into the base: `t_1`→`t1`, `n_0`→`n0`
* Typed subscripts keep the underscore: `num_{numtype}`→`num_numtype`
* A LaTeX prime stays attached: `{\heaptype}'`→`ht'`
* Record/context field access `X{.}Y` → `X.Y`
* Lookup/application `X{[…]}` → `X[…]`
* Function application `f(a,b)` → `f(a, b)`
* Cardinality/length `|…|` (e.g. `{|s{.}\STAGS|}` ) → `|s.TAGS|`

## 5. Additional RST constructs

Beyond §1–§4 the tree uses the following. Inference rules get their own section (§6).

### 5.1 Headings — 5th level and reference headings
The spec uses a 5th section level underlined with dots, and headings that *are themselves* `:ref:` / `:math:` wrappers:

```rst
:ref:`Tags <syntax-taginst>`
............................

:math:`{\alloctag}(s, {\tagtype})`
.................................
```

* Dotted underline (`.....`) is the 5th level → `#### Heading`. **Never emit `#####`.** A 5th-level `#####` heading renders *smaller* than the surrounding body text in several common Markdown renderers (notably GitHub's), which looks broken. Collapse every dotted-level heading to `####` so it stays at least as large as normal text.
* When the heading text is a `:ref:`…`` or `:math:`…`` wrapper, keep only the inner text (resolve the ref to its link text, or typeset the math via §3/§4) and drop the wrapper/anchor.
  * `:ref:`Tags <syntax-taginst>`` → `#### Tags`
  * `:math:`\NOP`` → `#### NOP`

### 5.2 Lists
* Bullet lists: `*` at line start → `* item`. Nest by indentation (2 spaces per level). Never use `-` or `+` for bullets — always `*`.
* **No blank lines between list items.** Keep lists *tight*: consecutive items (and their nested children) must be on adjacent lines with no blank line between them. Blank lines between items make each item render as a separate paragraph and break visual nesting; a blank line after a parent item, before its nested child list, is likewise removed. (Blank lines that separate a list from surrounding prose, or that separate two genuinely separate lists, are preserved.)
* Ordered lists: always write explicit `1.`, `2.`, `3.` (`1. item`). Never use `#.` — the output is a static reference consumed by AI, so numbering must be literal and stable (GitHub auto-renumbering of `#.` does not apply to a plain `.md` reference)
* Lettered sublists: `a.` / `A.` → a nested ordered list
* The exec chapter's numbered steps (source `1.`, then `#.`, then `a.` sub-steps) always render as explicit `1.`, `2.`, `3.` nested ordered lists

### 5.3 Code blocks
* `.. code-block:: <lang>` (e.g. `pseudo`) → a fenced code block tagged with that language.
* A paragraph followed by `::` and an indented block → also a fenced code block (use `text` if no language is implied).
* Keep the code content verbatim — do **not** “fix” pseudo-code.

For example, `.. code-block:: pseudo` becomes the fence:

```pseudo
...
```

### 5.4 Footnotes
Footnotes appear as `[#name]_` inline and `.. [#name]` definition blocks (usually at the end of a file).
* Inline: `[#cite-pldi2017]_` → `[^cite-pldi2017]` (GFM footnote reference)
* Group all footnote definitions at the bottom of the converted file

**Definition example** — RST source:

```rst
.. [#cite-pldi2017]
   Andreas Rossberg. "…". PLDI 2017.
```

becomes

```md
[^cite-pldi2017]: Andreas Rossberg. "…". PLDI 2017.
```

### 5.5 Substitution references
`|Unicode|_` and `|ASCII|_` (and `|FUNCREF|` in the changes appendix) are *substitution references*. Their definitions live outside this tree (in the Sphinx project), so the LLM cannot resolve the target URL
* Render `|Name|` and `|Name|_` simply as the name text: `Unicode`, `ASCII`, `FUNCREF`
* Do **not** invent URLs. (If a definition is present in the same file, link it.)

### 5.6 Paragraphs and line breaks (no one-sentence-per-line)
* **Never break a paragraph with a single newline.** In Markdown, a single newline inside a paragraph is a *soft break* and renders as an ordinary space — it has no semantic meaning and does **not** match what is actually rendered. A paragraph must be a single block of text.
* When the RST source splits one prose paragraph across several physical lines (the common “one sentence per line” style), either rejoin those lines into a single paragraph, or split them to multiple paragraphs during conversion. Use personal judgment to decide which approach to take.
* If the text is genuinely two paragraphs, separate them with a **blank line** (a hard break / new paragraph). Do not use a bare newline to imply a paragraph break.
* The same rule applies *inside blockquotes*: a `> **Note:** …` whose body spreads over several `>` lines is one note paragraph — join it into a single `>` line. Keep the blank `>` line that separates two genuinely separate notes.
* (Rationale: source that relies on single newlines for “visual” line breaks looks broken in any Markdown editor/preview and misleads readers about where paragraphs actually start and end.)

### 5.7 Blank lines — never double
* **Collapse any run of two or more consecutive blank lines into a single blank line.** More than one blank line has no effect on rendering — Markdown only uses a blank line as a block separator, and any number ≥ 1 behaves identically — so multiple blank lines add nothing but inconsistent gaps in the source.
* This applies everywhere: between headings, paragraphs, lists, code fences, footnote blocks, and at the start/end of the file. The only invariant to preserve is that a *single* blank line separates two distinct blocks.
* Do not *introduce* double blank lines either. Converted output should have at most one blank line between any two blocks.
* (For lists specifically, §5.2 goes further and removes the single blank line between items entirely, to keep them tight.)

## 6. Inference rules (`\frac`)

The `valid/` and `exec/` chapters express typing/reduction as fractional rules:

```rst
.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashheaptype {\heaptype'} : \OKheaptype
    \qquad
   C \vdashheaptypematch {\heaptype}_1 \subheaptypematch {\heaptype'}
    \qquad
   C \vdashheaptypematch {\heaptype'} \subheaptypematch {\heaptype}_2
   }{
   C \vdashheaptypematch {\heaptype}_1 \subheaptypematch {\heaptype}_2
   }
   \qquad
   \end{array}
```

Rules are wrapped in `\frac{numerator}{denominator}` (premises over conclusion). Rendering rules:

1. **One block per `\frac`.** Two `\frac` blocks separated by `\qquad` at the same level are *separate* rules → render each in its own fenced `text` block
2. **Premises** are the numerator, split on either `\\` (explicit line break) or `\qquad` (when several premises share one numerator). Render one premise per line
3. **Conclusion** is the denominator → place it after a `────` divider
4. **Rule name**: take it from the nearest preceding heading (often a `:math:`-heading, e.g. `:math:`\NOP``). Typeset via §3/§4 and use as a title above the block
5. **Judgment macros**:
   * `\vdash⟨X⟩` → `|-`
   * the matching/relation infix `\sub⟨X⟩match` → `<:`
   * reduction `\to`/`\rightarrow` → `->`
   * validity result `\OK⟨X⟩` → `OK`
   (The ⟨X⟩ qualifier word — `heaptype`, `instr`, … — is dropped. It is recoverable from the section heading.)
6. **Side conditions**:
   * written as `\quad \mbox{if}…` after a premise/conclusion → append `(if …)` to that line
   * a `\multicolumn` side condition → its own `(if …)` line

Target:

```text
Heap Types

C |- ht' : OK
C |- ht1 <: ht'
C |- ht' <: ht2
─────────────────────────────
C |- ht1 <: ht2
```

Reduction rules use the same layout with `->` (or `~>` for small-step, kept verbatim per §4.6).

## 7. Full worked example (input → output)

### Input (excerpt from `binary/types.rst`)

```rst
Heap Types
~~~~~~~~~~

:ref:`Heap types <syntax-reftype>` are encoded as either a single byte, or as a :ref:`type index <binary-typeidx>` encoded as a positive :ref:`signed integer <binary-sint>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Babsheaptype} & ::= & \mathtt{0x69} & \quad\Rightarrow\quad{} & \EXN \\
   & & | & \mathtt{0x6A} & \quad\Rightarrow\quad{} & \ARRAY \\
   & & | & \mathtt{0x6B} & \quad\Rightarrow\quad{} & \STRUCT \\
   & & | & \mathtt{0x6C} & \quad\Rightarrow\quad{} & \I31 \\
   & & | & \mathtt{0x6D} & \quad\Rightarrow\quad{} & \EQT \\
   & & | & \mathtt{0x6E} & \quad\Rightarrow\quad{} & \ANY \\
   & & | & \mathtt{0x6F} & \quad\Rightarrow\quad{} & \EXTERN \\
   & & | & \mathtt{0x70} & \quad\Rightarrow\quad{} & \FUNCT \\
   & & | & \mathtt{0x71} & \quad\Rightarrow\quad{} & \NONE \\
   & & | & \mathtt{0x72} & \quad\Rightarrow\quad{} & \NOEXTERN \\
   & & | & \mathtt{0x73} & \quad\Rightarrow\quad{} & \NOFUNC \\
   & & | & \mathtt{0x74} & \quad\Rightarrow\quad{} & \NOEXN \\[0.8ex]
   & {\Bheaptype} & ::= & {\mathit{ht}}{:}{\Babsheaptype} & \quad\Rightarrow\quad{} & {\mathit{ht}} \\
   & & | & x{:}{\Bs33} & \quad\Rightarrow\quad{} & x & \quad \mbox{if}~ x \geq 0 \\
   \end{array}

.. note::
   The heap type :math:`\BOT` cannot occur in a module.
```

### Output (target Markdown)

---

### Heap Types

Heap types are encoded as either a single byte, or as a type index encoded as a positive signed integer.

```text
absheaptype ::=
  | 0x69    => exn
  | 0x6A    => array
  | 0x6B    => struct
  | 0x6C    => i31
  | 0x6D    => eq
  | 0x6E    => any
  | 0x6F    => extern
  | 0x70    => func
  | 0x71    => none
  | 0x72    => noextern
  | 0x73    => nofunc
  | 0x74    => noexn

heaptype ::=
  | ht : absheaptype    => ht
  | x : s33             => x (if x >= 0)
```

> **Note:** The heap type `bot` (bottom) cannot occur in a module.

---

## 8. Math conversion worked examples

### 8.1 Text grammar (text syntax nonterminals, Unicode points)

Input (excerpt `text/lexical.rst`):

```rst
.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Tsource} & ::= & {{\Tchar}^\ast} \\[0.8ex]
   & {\Tchar} & ::= & \mathrm{U{+}00} ~~|~~ \ldots ~~|~~ \mathrm{U{+}D7FF} ~~|~~ \mathrm{U{+}E000} ~~|~~ \ldots ~~|~~ \mathrm{U{+}10FFFF} \\
   \end{array}
```

Output:

```text
source ::= char*
char   ::= U+00 | ... | U+D7FF | U+E000 | ... | U+10FFFF
```

### 8.2 Store record (S/H/C/M field prefixes, sequences)

Input (excerpt `exec/runtime.rst`):

```rst
.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\store} & ::= & \{ \STAGS~{{\taginst}^\ast} \\
   & & & \SGLOBALS~{{\globalinst}^\ast} \\
   & & & \SMEMS~{{\meminst}^\ast} \\
   & & & \STABLES~{{\tableinst}^\ast} \\
   & & & \SFUNCS~{{\funcinst}^\ast} \\
   & & & \SDATAS~{{\datainst}^\ast} \\
   \end{array}
```

Output:

```text
store ::=
  { TAGS taginst*
    GLOBALS globalinst*
    MEMS meminst*
    TABLES tableinst*
    FUNCS funcinst*
    DATAS datainst* }
```

### 8.3 Auxiliary function equation (operators, cardinality, side condition)

Input (excerpt `exec/modules.rst`):

```rst
.. math::
   \begin{array}[t]{@{}lcl@{}l@{}}
   {\alloctag}(s, {\tagtype}) & = & (s \oplus \{ \STAGS~{\taginst} \}, {|s{.}\STAGS|}) &  \\
    \multicolumn{4}{@{}l@{}}{\quad
   \quad \mbox{if}~ {\taginst} = \{ \HITYPE~{\tagtype} \}
   } \\
   \end{array}
```

Output:

```text
alloctag(s, tagtype) = (s ⊕ { TAGS taginst }, |s.TAGS|)
  (if taginst = { TYPE tagtype })
```

### 8.4 Inference rule (premises, divider, transitivity)

Input (excerpt `valid/matching.rst` — the `\frac` from §6):

```rst
.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashheaptype {\heaptype'} : \OKheaptype
    \qquad
   C \vdashheaptypematch {\heaptype}_1 \subheaptypematch {\heaptype'}
    \qquad
   C \vdashheaptypematch {\heaptype'} \subheaptypematch {\heaptype}_2
   }{
   C \vdashheaptypematch {\heaptype}_1 \subheaptypematch {\heaptype}_2
   }
   \end{array}
```

Output (rule name taken from the enclosing *Heap Types* heading):

```text
Heap Types

C |- ht' : OK
C |- ht1 <: ht'
C |- ht' <: ht2
─────────────────────────────
C |- ht1 <: ht2
```

## 9. Per-file conversion checklist

* [ ] Every `.. math::` became a `text` fence (grammar/equation) or a rule block (`\frac`, §6).
* [ ] Every `\MACRO` resolved per §4 (no raw backslash commands left in output).
* [ ] All `.. index::` / `pair:` / `single:` / `.. _name:` / `.. only::` / `.. toctree::` dropped.
* [ ] Headings use the correct `#` depth. 5th-level dotted headings are `####` (never `#####` — see §5.1).
* [ ] Lists, notes, code blocks, and footnotes preserved.
* [ ] Prose paragraphs are single blocks — no one-sentence-per-line breaks, and notes joined into one `>` paragraph (§5.6).
* [ ] No double (or more) blank lines anywhere — collapse runs of blank lines to a single one (§5.7).
* [ ] Substitution refs rendered as plain name text. No invented URLs.
* [ ] No information dropped: every alternative, byte, constructor, premise, and side condition preserved.

