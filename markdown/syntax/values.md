## Values

WebAssembly programs operate on primitive numeric *values*. Moreover, in the definition of programs, immutable sequences of values occur to represent more complex data, such as text strings or other vectors.

### Bytes

The simplest form of value are raw uninterpreted *bytes*. In the abstract syntax they are represented as hexadecimal literals.

```text
byte ::= 0x00 | ... | 0xFF
```

#### Conventions

* The meta variable `b` ranges over bytes.
* Bytes are sometimes interpreted as natural numbers `n < 256`.

### Integers

Different classes of *integers* with different value ranges are distinguished by their *bit width* `N` and by whether they are *unsigned* or *signed*.

```text
uN ::= 0 | ... | 2^N - 1

sN ::= -2^(N-1) | ... | -1 | 0 | +1 | ... | +2^(N-1) - 1

iN ::= uN
```

The class `iN` defines *uninterpreted* integers, whose signedness interpretation can vary depending on context. In the abstract syntax, they are represented as unsigned values. However, some operations convert them to signed based on a two's complement interpretation.

> **Note:** The main integer types occurring in this specification are `u8`, `u32`, `u64`, and `u128`. However, other sizes occur as auxiliary constructions, e.g., in the definition of floating-point numbers.

#### Conventions

* The meta variables `m`, `n`, `i`, `j` range over integers.
* Numbers may be denoted by simple arithmetics, as in the grammar above. In order to distinguish arithmetics like `2^N` from sequences like `(1)^N`, the latter is distinguished with parentheses.

### Floating-Point

*Floating-point* data represents 32 or 64 bit values that correspond to the respective binary formats of the IEEE 754 standard (Section 3.3).

Every value has a *sign* and a *magnitude*. Magnitudes can either be expressed as *normal* numbers of the form `m0 . m1 m2 ... mm * 2^e`, where `e` is the exponent and `m` is the *significand* whose most significant bit `m0` is `1`, or as a *subnormal* number where the exponent is fixed to the smallest possible value and `m0` is `0`; among the subnormals are positive and negative zero values. Since the significands are binary values, normals are represented in the form `(1 + m * 2^-M) * 2^e` in the abstract syntax, where `M` is the bit width of `m`; similarly for subnormals.

Possible magnitudes also include the special values `∞` (infinity) and `nan` (*NaN*, not a number). NaN values have a *payload* that describes the mantissa bits in the underlying binary representation. No distinction is made between signalling and quiet NaNs.

```text
fN ::= +fNmag | -fNmag

fNmag ::= (1 + m * 2^-M) * 2^e   (if m < 2^M && 2 - 2^(E-1) <= e <= 2^(E-1) - 1)
        | (0 + m * 2^-M) * 2^e   (if m < 2^M && 2 - 2^(E-1) = e)
        | ∞
        | nan(m)   (if 1 <= m < 2^M)
```

where `M = signif(N)` and `E = expon(N)` with

```text
signif(32) = 23
signif(64) = 52
expon(32) = 8
expon(64) = 11
```

A *canonical NaN* is a floating-point value `±nan(canon_N)` where `canon_N` is a payload whose most significant bit is `1` while all others are `0`:

```text
canon_N = 2^(signif(N) - 1)
```

An *arithmetic NaN* is a floating-point value `±nan(m)` with `m >= canon_N`, such that the most significant bit is `1` while all others are arbitrary.

> **Note:** In the abstract syntax, subnormals are distinguished by the leading `0` of the significand. The exponent of subnormals has the same value as the smallest possible exponent of a normal number. Only in the binary representation the exponent of a subnormal is encoded differently than the exponent of any normal number.
>
> The notion of canonical NaN defined here is unrelated to the notion of canonical NaN that the IEEE 754 standard (Section 3.5.2) defines for decimal interchange formats.

#### Conventions

* The meta variable `z` ranges over floating-point values where clear from context.
* Where clear from context, shorthands like `+1` denote floating point values like `+(1 + 0 * 2^-M) * 2^0`.

### Vectors

*Numeric vectors* are 128-bit values that are processed by vector instructions (also known as *SIMD* instructions, single instruction multiple data). They are represented in the abstract syntax using `u128`. The interpretation of lane types (integer or floating-point numbers) and lane sizes are determined by the specific instruction operating on them.

### Names

*Names* are sequences of *characters*, which are *scalar values* as defined by Unicode (Section 2.4).

```text
name ::= char*   (if |utf8(char*)| < 2^32)

char ::= U+00 | ... | U+D7FF | U+E000 | ... | U+10FFFF
```

Due to the limitations of the binary format, the length of a name is bounded by the length of its UTF-8 encoding.

#### Convention

* Characters (Unicode scalar values) are sometimes used interchangeably with natural numbers `n < 1114112`.
