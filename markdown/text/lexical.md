## Lexical Format

### Characters

The text format assigns meaning to *source text*, which consists of a sequence of *characters*.

Characters are assumed to be represented as valid Unicode (Section 2.4) *scalar values*.

```text
source ::=
  | char*

char ::=
  | U+00 | ... | U+D7FF | U+E000 | ... | U+10FFFF
```

> **Note:** While source text may contain any Unicode character in comments or string literals, the rest of the grammar is formed exclusively from the characters supported by the 7-bit ASCII subset of Unicode.

### Tokens

The character stream in the source text is divided, from left to right, into a sequence of *tokens*, as defined by the following grammar.

```text
token ::=
  | keyword | uN | sN | fN | string | id | '(' | ')' | reserved

keyword ::=
  | ('a' | ... | 'z') idchar*

reserved ::=
  | (idchar | string | ',' | ';' | '[' | ']' | '{' | '}')^+
```

Tokens are formed from the input character stream according to the *longest match* rule.

That is, the next token always consists of the longest possible sequence of characters that is recognized by the above lexical grammar.

Tokens can be separated by white space, but except for strings, they cannot themselves contain whitespace.

*Keyword* tokens always start with a lower-case letter.

The set of keywords is defined implicitly: only those tokens are defined to be keywords that occur as a terminal symbol in literal form, such as `'keyword'`, in a syntactic production of this chapter.

Any token that does not fall into any of the other categories is considered *reserved*, and cannot occur in source text.

> **Note:** The effect of defining the set of reserved tokens is that all tokens must be separated by either parentheses, white space, or comments.
>
> For example, `'0$x'` is a single reserved token, as is `'a''b'`.
>
> Consequently, they are not recognized as two separate tokens `0` and `$x`, or `'a'` and `'b'`, respectively, but instead disallowed.
>
> This property of tokenization is not affected by the fact that the definition of reserved tokens overlaps with other token classes.

### White Space

*White space* is any sequence of literal space characters, formatting characters, comments, or annotations.

The allowed formatting characters correspond to a subset of the ASCII *format effectors*, namely, *horizontal tabulation* (`U+09`), *line feed* (`U+0A`), and *carriage return* (`U+0D`).

```text
space ::=
  | ( ' ' | format | comment | annot )*

format ::=
  | newline | U+09

newline ::=
  | U+0A | U+0D | U+0D U+0A
```

The only relevance of white space is to separate tokens. It is otherwise ignored.

### Comments

A *comment* can either be a *line comment*, started with a double semicolon `;;` and extending to the end of the line, or a *block comment*, enclosed in delimiters `(;` ... `;)`.

Block comments can be nested.

```text
comment ::=
  | linecomment | blockcomment

linecomment ::=
  | ';;' linechar* (newline | eof)

linechar ::=
  | c : char    (if c != U+0A && c != U+0D)

blockcomment ::=
  | '(;' blockchar* ';)'

blockchar ::=
  | c : char    (if c != ';' && c != '(')
  | ';'+ c : char    (if c != ';' && c != ')')
  | '('+ c : char    (if c != ';' && c != '(')
  | blockcomment
```

Here, the pseudo token `eof` indicates the end of the input. The *look-ahead* restrictions on the productions for `blockchar` disambiguate the grammar such that only well-bracketed uses of block comment delimiters are allowed.

> **Note:** Any formatting and control characters are allowed inside comments.

### Annotations

An *annotation* is a bracketed token sequence headed by an *annotation id* of the form `@id` or `@'...'`.

No space is allowed between the opening parenthesis and this id.

Annotations are intended to be used for third-party extensions; they can appear anywhere in a program but are ignored by the WebAssembly semantics itself, which treats them as white space.

Annotations can contain other parenthesized token sequences (including nested annotations), as long as they are well-nested. String literals and comments occurring in an annotation must also be properly nested and closed.

```text
annot ::=
  | '(@' annotid (space | token)* ')'

annotid ::=
  | idchar+ | name
```

> **Note:** The annotation id is meant to be an identifier categorizing the extension, and plays a role similar to the name of a custom section.
>
> By convention, annotations corresponding to a custom section should use the custom section's name as an id.
>
> Implementations are expected to ignore annotations with ids that they do not recognize.
>
> On the other hand, they may impose restrictions on annotations that they do recognize, e.g., requiring a specific structure by superimposing a more concrete grammar.
>
> It is up to an implementation how it deals with errors in such annotations.
