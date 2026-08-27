## Values

### Bytes

Bytes encode themselves.

```text
byte ::= 0x00 | ... | 0xFF
```

### Integers

All integers are encoded using the LEB128 variable-length integer encoding, in either unsigned or signed variant.

Unsigned integers are encoded in UnsignedLEB128 format.

As an additional constraint, the total number of bytes encoding a `uN N` value must not exceed `ceil(N / 7)` bytes.

```text
uN N ::=
  | n : byte                  => n  (if n < 2^7 ∧ n < 2^N)
  | n : byte m : uN(N - 7)    => 2^7 · m + (n - 2^7) (if n >= 2^7 ∧ N > 7)
```

Signed integers are encoded in SignedLEB128 format, which uses a two's complement representation.

As an additional constraint, the total number of bytes encoding an `sN N` value must not exceed `ceil(N / 7)` bytes.

```text
sN N ::=
  | n : byte                  => n  (if n < 2^6 ∧ n < 2^(N - 1))
  | n : byte                  => n - 2^7  (if 2^6 <= n < 2^7 ∧ n >= 2^7 - 2^(N - 1))
  | n : byte i : sN(N - 7)    => 2^7 · i + (n - 2^7) (if n >= 2^7 ∧ N > 7)
```

Uninterpreted integers are encoded as signed integers.

```text
iN N ::=
  | i : sN N    => signed_N^{-1}(i)
```

> **Note:** The side conditions `N > 7` in the productions for non-terminal bytes of the `uN N` and `sN N` encodings restrict the encoding's length. However, "trailing zeros" are still allowed within these bounds. For example, `0x03` and `0x83 0x00` are both well-formed encodings for the value `3` as a `u8`. Similarly, either of `0x7E` and `0xFE 0xFF 0x7F` and `0xFE 0xFF 0x7F` are well-formed encodings of the value `-2` as an `s16`.
>
> The side conditions on the value `n` of terminal bytes further enforce that any unused bits in these bytes must be `0` for positive values and `1` for negative ones. For example, `0x83 0x10` is malformed as a `u8` encoding. Similarly, both `0x83 0x3E` and `0xFF 0x7B` are malformed as `s8` encodings.

### Floating-Point

Floating-point values are encoded directly by their IEEE754 (Section 3.4) bit pattern in LittleEndian byte order:

```text
fN N ::=
  | b* : byte^(N / 8)    => bytes_{fN N}^{-1}(b*)
```

### Names

Names are encoded as a list of bytes containing the Unicode (Section 3.9) UTF-8 encoding of the name's character sequence.

```text
name ::=
  | b* : list(byte)    => name    (if utf8(name) = b*)
```

The auxiliary `utf8` function expressing this encoding is defined as follows:

```text
utf8(ch*)    = ⋆ utf8(ch)*
utf8(ch)     = b
  (if ch < U+80 ∧ ch = b)
utf8(ch)     = b1 b2
  (if U+80 <= ch < U+0800 ∧ ch = 2^6 · (b1 - 0xC0) + cont(b2))
utf8(ch)     = b1 b2 b3
  (if U+0800 <= ch < U+D800 ∨ U+E000 <= ch < U+10000 ∧ ch = 2^12 · (b1 - 0xE0) + 2^6 · cont(b2) + cont(b3))
utf8(ch)     = b1 b2 b3 b4
  (if U+10000 <= ch < U+11000 ∧ ch = 2^18 · (b1 - 0xF0) + 2^12 · cont(b2) + 2^6 · cont(b3) + cont(b4))
```

where `cont(b) = b - 0x80` (if `0x80 < b < 0xC0`).

> **Note:** Unlike in some other formats, name strings are not 0-terminated.
