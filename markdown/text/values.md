## Values

The grammar productions in this section define *lexical syntax*, hence no white space is allowed.

### Integers

All integers can be written in either decimal or hexadecimal notation. In both cases, digits can optionally be separated by underscores.

```text
sign ::=
  | ε                  => +1
  | '+'                => +1
  | '-'                => -1

digit ::=
  | '0'                => 0
  | ...
  | '9'                => 9

hexdigit ::=
  | d : digit          => d
  | 'A'                => 10
  | ...
  | 'F'                => 15
  | 'a'                => 10
  | ...
  | 'f'                => 15

num ::=
  | d : digit                          => d
  | n : num  '_'?  d : digit           => 10*n + d

hexnum ::=
  | h : hexdigit                       => h
  | n : hexnum  '_'?  h : hexdigit      => 16*n + h
```

The allowed syntax for integer literals depends on size and signedness. Moreover, their value must lie within the range of the respective type.

```text
uN ::=
  | n : num            => n    (if n < 2^N)
  | '0x' n : hexnum    => n    (if n < 2^N)

sN ::=
  | s : sign n : uN    => s * n    (if -2^(N-1) <= s * n < 2^(N-1))
```

Uninterpreted integers can be written as either signed or unsigned, and are normalized to unsigned in the abstract syntax.

```text
iN ::=
  | n : uN    => n
  | i : sN    => signed_N^(-1)(i)
```

### Floating-Point

Floating-point values can be represented in either decimal or hexadecimal notation.

```text
frac ::=
  | d : digit                     => d / 10
  | d : digit  '_'?  p : frac     => (d + p/10) / 10

hexfrac ::=
  | h : hexdigit                  => h / 16
  | h : hexdigit  '_'?  p : hexfrac  => (h + p/16) / 16

mant ::=
  | p : num  '.'?                 => p
  | p : num  '.'  q : frac        => p + q

hexmant ::=
  | p : hexnum  '.'?              => p
  | p : hexnum  '.'  q : hexfrac  => p + q

float ::=
  | p : mant  ('E' | 'e') s : sign e : num  => p * 10^(s*e)

hexfloat ::=
  | '0x' p : hexmant  ('P' | 'p') s : sign e : num  => p * 2^(s*e)
```

The value of a literal must not lie outside the representable range of the corresponding IEEE754 type (that is, a numeric value must not overflow to `±∞`), but it may be rounded to the nearest representable value.

> **Note:** Rounding can be prevented by using hexadecimal notation with no more significant bits than supported by the required type.

Floating-point values may also be written as constants for *infinity* or *canonical NaN* (*not a number*).

Furthermore, arbitrary NaN values may be expressed by providing an explicit payload value.

```text
fN ::=
  | (+1) : sign  q : fNmag  => +q
  | (-1) : sign  q : fNmag  => -q

fNmag ::=
  | q : float     => ieee_N(q)    (if ieee_N(q) != ∞)
  | q : hexfloat  => ieee_N(q)    (if ieee_N(q) != ∞)
  | 'inf'         => ∞
  | 'nan'         => nan(canon_N)
  | 'nan:0x' n : hexnum  => nan(n)    (if 1 <= n < 2^signif(N))
```

### Strings

*Strings* denote sequences of bytes that can represent both textual and binary data.

They are enclosed in quotation marks and may contain any character other than ASCII control characters, quotation marks (`'`), or backslash (`\`), except when expressed with an *escape sequence*.

```text
string ::=
  | ''  (b* : stringelem)*  ''  => ++ (b*)*    (if | ++ (b*)* | < 2^32)

stringelem ::=
  | c : stringchar  => utf8(c)
  | '\' h1 : hexdigit h2 : hexdigit  => 16*h1 + h2
```

Each character in a string literal represents the byte sequence corresponding to its UTF-8 Unicode (Section 2.5) encoding, except for hexadecimal escape sequences `'\hh'`, which represent raw bytes of the respective value.

```text
stringchar ::=
  | c : char  => c    (if c >= U+20 && c != U+7F && c != '' && c != '\')
  | '\t'  => U+09
  | '\n'  => U+0A
  | '\r'  => U+0D
  | '\"'  => U+22
  | '\''  => U+27
  | '\\'  => U+5C
  | '\u{' n : hexnum '}'  => n    (if n < 0xD800 || 0xE800 <= n < 0x110000)
```

### Names

Names are strings denoting a literal character sequence.

A name string must form a valid UTF-8 encoding as defined by Unicode (Section 2.5) and is interpreted as a string of Unicode scalar values.

```text
name ::=
  | b* : string  => c*    (if b* = utf8(c*))
```

> **Note:** Presuming the source text is itself encoded correctly, strings that do not contain any uses of hexadecimal byte escapes are always valid names.

### Identifiers

Indices can be given in both numeric and symbolic form.

Symbolic *identifiers* that stand in lieu of indices start with `$`, followed by either a sequence of printable ASCII characters that does not contain a space, quotation mark, comma, semicolon, or bracket, or by a quoted name.

```text
id ::=
  | '$' c* : idchar^+  => c*
  | '$' c* : name      => c*    (if |c*| > 0)

idchar ::=
  | '0' | ... | '9'
  | 'A' | ... | 'Z'
  | 'a' | ... | 'z'
  | '!' | '#' | '$' | '%' | '&' | ''' | '*' | '+' | '-' | '.' | '/'
  | ':' | '<' | '=' | '>' | '?' | '@' | '\' | '^' | '_' | '`' | '|' | '~'
```

> **Note:** The value of an identifier character is the Unicode codepoint denoting it.

#### Conventions

The expansion rules of some abbreviations require insertion of a *fresh* identifier.
That may be any syntactically valid identifier that does not already occur in the given source text.
