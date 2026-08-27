# WebAssembly Core Specification Markdown Conversion Project

This is a project to maintain an AI-assisted conversion of the latest official core WebAssembly specification (currently at version 3.0), originally written in reStructuredText (`.rst`) and LaTeX markup, to human-readable, GitHub flavored Markdown.

## Goals

* Provide a simple, easy-to-read, human and AI friendly markdown reference that can be viewed by any existing GitHub flavored Markdown renderer
* Allow the markdown reference to be readily used for AI-assisted verification of existing WebAssembly-related projects, as well of creation of new projects, with or without AI involvement
* Allow easier AI-assisted knowledge extraction and consolidation of WebAssembly-related topics
* Allow easier copy-and-paste from fragments of specification documents
* Improve searchability of the overall specification with regular expressions and tools like `grep`
* Improve ease of automated editing and proofreading via specially written scripts
* Improve compatibility with assistive technologies like text-to-speech and screen readers

## Directory structure

* `reference/**/*` contains the reference `.rst` files generated via the [official repository](https://github.com/WebAssembly/spec/) using tools like `sphinx` and `SpecTec`. The files are extracted from `WebAssembly/spec/document/core/_spectec/**/*.rst`, which contains intermediate outputs, after the compiled LaTeX is embedded into the reStructuredText templates
* `markdown/**/*` contains the corresponding AI converted markdown files, with a matching directory tree, and file names ending with `.md` extensions

## Example of reference source and output markdown

### Source reStructuredText + embedded LaTeX math blocks (excerpt from `binary/types.rst`)

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

### Output Markdown

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

## Conversion guide

The [CONVERT.md](CONVERT.md) file contains detailed instructions, primarily for AI agents, on how to correctly perform the `.rst` -> `.md` conversion, and how to handle various special input patterns.

Note the guide was itself mostly developed by AI agents, so it may contain some unintended inaccuracies that were not fully human-audited.

## License

The converted markdown documents are dedicated to the public domain under [Creative Commons CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/).

The reference documentation is licensed under the [W3C Software and Document License](https://www.w3.org/copyright/software-license/).
