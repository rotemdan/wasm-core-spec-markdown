## Numerics

Numeric primitives are defined in a generic manner, by operators indexed over a bit width `N`.

Some operators are *non-deterministic*, because they can return one of several possible results (such as different [NaN](syntax-nan) values). Technically, each operator thus returns a *set* of allowed values. For convenience, deterministic results are expressed as plain values, which are assumed to be identified with a respective singleton set.

Some operators are *partial*, because they are not defined on certain inputs. Technically, an empty set of results is returned for these inputs.

In formal notation, each operator is defined by equational clauses that apply in decreasing order of precedence. That is, the first clause that is applicable to the given arguments defines the result. In some cases, similar clauses are combined into one by using the notation `±` or `∓`. When several of these placeholders occur in a single clause, then they must be resolved consistently: either the upper sign is chosen for all of them or the lower sign.

> **Note:** For example, the fcopysign operator is defined as follows:
>
> ```text
> fcopysign_N(± p1, ± p2) = ± p1
> fcopysign_N(± p1, ∓ p2) = ∓ p1
> ```
>
> This definition is to be read as a shorthand for the following expansion of each clause into two separate ones:
>
> ```text
> fcopysign_N(+ p1, + p2) = + p1
> fcopysign_N(- p1, - p2) = - p1
> fcopysign_N(+ p1, - p2) = - p1
> fcopysign_N(- p1, + p2) = + p1
> ```

Numeric operators are lifted to input sequences by applying the operator element-wise, returning a sequence of results. When there are multiple inputs, they must be of equal length.

```text
op(c1^n, ..., c_k^n) = op(c1^n[0], ..., c_k^n[0]) ... op(c1^n[n-1], ..., c_k^n[n-1])
```

> **Note:** For example, the unary operator fabs, when given a sequence of floating-point values, return a sequence of floating-point results:
>
> ```text
> fabs_N(z^n) = fabs_N(z[0]) ... fabs_N(z[n])
> ```
>
> The binary operator iadd, when given two sequences of integers of the same length, `n`, return a sequence of integer results:
>
> ```text
> iadd_N(i1^n, i2^n) = iadd_N(i1[0], i2[0]) ... iadd_N(i1[n], i2[n])
> ```

### Conventions

* The meta variable `d` is used to range over single bits.
* The meta variable `p` is used to range over (signless) magnitudes of floating-point values, including NAN and `∞`.
* The meta variable `q` is used to range over (signless) rational magnitudes, excluding NAN or `∞`.
* The notation `f^{-1}` denotes the inverse of a bijective function `f`.
* Truncation of rational values is written `truncz(± q)`, with the usual mathematical definition:

  ```text
  truncz(± q) = ± i  (iff i ∈ ℕ ∧ +q - 1 < i ≤ +q)
  ```

* Saturation of integers is written `satu_N(i)` and `sats_N(i)`. The arguments to these two functions range over arbitrary signed integers.
  * Unsigned saturation, `satu_N(i)` clamps `i` to between `0` and `2^N-1`:

    ```text
    satu_N(i) = 0      (iff i < 0)
    satu_N(i) = 2^N-1  (iff i > 2^N-1)
    satu_N(i) = i      (otherwise)
    ```

  * Signed saturation, `sats_N(i)` clamps `i` to between `-2^{N-1}` and `2^{N-1}-1`:

    ```text
    sats_N(i) = -2^{N-1}  (iff i < -2^{N-1})
    sats_N(i) = 2^{N-1}-1  (iff i > 2^{N-1}-1)
    sats_N(i) = i          (otherwise)
    ```

### Representations

Numbers and numeric vectors have an underlying binary representation as a sequence of bits:

```text
bits_iN(i) = ibits_N(i)
bits_fN(z) = fbits_N(z)
bits_vN(i) = ibits_N(i)
```

The first case of these applies to representations of both integer [value types](syntax-valtype) and [packed types](syntax-packtype).

Each of these functions is a bijection, hence they are invertible.

#### Integers

[Integers](syntax-int) are represented as base two unsigned numbers:

```text
ibits_N(i) = d_{N-1} ... d_0   (i = 2^{N-1}·d_{N-1} + ... + 2^0·d_0)
```

Boolean operators like `and`, `or`, or `xor` are lifted to bit sequences of equal length by applying them pointwise.

#### Floating-Point

[Floating-point values](syntax-float) are represented in the respective binary format defined by IEEE754 (Section 3.4):

```text
fbits_N(± (1+m·2^{-M})·2^e) = fsign(±) ibits_E(e+fbias_N) ibits_M(m)
fbits_N(± (0+m·2^{-M})·2^e) = fsign(±) (0)^E ibits_M(m)
fbits_N(± ∞)                = fsign(±) (1)^E (0)^M
fbits_N(± NAN(n))           = fsign(±) (1)^E ibits_M(n)

fbias_N = 2^{E-1}-1
fsign(+) = 0
fsign(-) = 1
```

where `M = signif(N)` and `E = expon(N)`.

#### Vectors

Numeric vectors of type VN have the same underlying representation as an iN. They can also be interpreted as a sequence of numeric values packed into a VN with a particular shape `t x M`, provided that `N = |t|·M`.

```text
lanes_{t x M}(c) = c_0 ... c_{M-1}
  (where w   = |t| / 8
   ∧ b*      = bytes_iN(c)
   ∧ c_i     = bytes_t^{-1}(b*[i·w slice w]))
```

This function is a bijection on iN, hence it is invertible.

Numeric values can be *packed* into lanes of a specific [lane type](syntax-lanetype) and vice versa:

```text
packnum_numtype(c)   = c
packnum_packtype(c)  = wrap_{|unpack(packtype)|, |packtype|}(c)

unpacknum_numtype(c) = c
unpacknum_packtype(c) = extend_{|packtype|, |unpack(packtype)|}^U(c)
```

#### Storage

When a number is stored into [memory](syntax-mem), it is converted into a sequence of [bytes](syntax-byte) in LittleEndian byte order:

```text
bytes_t(i) = littleendian(bits_t(i))

littleendian(ε) = ε
littleendian(d^8 d'*) = littleendian(d'*) ibits_8^{-1}(d^8)
```

Again these functions are invertible bijections.

### Integer Operations

#### Sign Interpretation

Integer operators are defined on iN values. Operators that use a signed interpretation convert the value using the following definition, which takes the two's complement when the value lies in the upper half of the value range (i.e., its most significant bit is `1`):

```text
signed_N(i) = i        (0 ≤ i < 2^{N-1})
signed_N(i) = i - 2^N  (2^{N-1} ≤ i < 2^N)
```

This function is bijective, and hence invertible.

#### Boolean Interpretation

The integer result of predicates -- i.e., [tests](syntax-testop) and [relational](syntax-relop) operators -- is defined with the help of the following auxiliary function producing the value `1` or `0` depending on a condition.

```text
tobool(C) = 1  (iff C)
tobool(C) = 0  (otherwise)
```

#### iadd_N(i1, i2)

* Return the result of adding `i1` and `i2` modulo `2^N`.

```text
iadd_N(i1, i2) = (i1 + i2) mod 2^N
```

#### isub_N(i1, i2)

* Return the result of subtracting `i2` from `i1` modulo `2^N`.

```text
isub_N(i1, i2) = (i1 - i2 + 2^N) mod 2^N
```

#### imul_N(i1, i2)

* Return the result of multiplying `i1` and `i2` modulo `2^N`.

```text
imul_N(i1, i2) = (i1 · i2) mod 2^N
```

#### idivu_N(i1, i2)

* If `i2` is `0`, then the result is undefined.
* Else, return the result of dividing `i1` by `i2`, truncated toward zero.

```text
idivu_N(i1, 0) = {}
idivu_N(i1, i2) = truncz(i1 / i2)
```

> **Note:** This operator is [partial](exec-op-partial).

#### idivs_N(i1, i2)

* Let `j1` be the [signed interpretation](aux-signed) of `i1`.
* Let `j2` be the [signed interpretation](aux-signed) of `i2`.
* If `j2` is `0`, then the result is undefined.
* Else if `j1` divided by `j2` is `2^{N-1}`, then the result is undefined.
* Else, return the result of dividing `j1` by `j2`, truncated toward zero.

```text
idivs_N(i1, 0) = {}
idivs_N(i1, i2) = {}        (iff signed_N(i1) / signed_N(i2) = 2^{N-1})
idivs_N(i1, i2) = signed_N^{-1}(truncz(signed_N(i1) / signed_N(i2)))
```

> **Note:** This operator is [partial](exec-op-partial). Besides division by `0`, the result of `(-2^{N-1})/(-1) = +2^{N-1}` is not representable as an `N`-bit signed integer.

#### iremu_N(i1, i2)

* If `i2` is `0`, then the result is undefined.
* Else, return the remainder of dividing `i1` by `i2`.

```text
iremu_N(i1, 0) = {}
iremu_N(i1, i2) = i1 - i2·truncz(i1 / i2)
```

> **Note:** This operator is [partial](exec-op-partial). As long as both operators are defined, it holds that `i1 = i2·idivu(i1, i2) + iremu(i1, i2)`.

#### irems_N(i1, i2)

* Let `j1` be the [signed interpretation](aux-signed) of `i1`.
* Let `j2` be the [signed interpretation](aux-signed) of `i2`.
* If `i2` is `0`, then the result is undefined.
* Else, return the remainder of dividing `j1` by `j2`, with the sign of the dividend `j1`.

```text
irems_N(i1, 0) = {}
irems_N(i1, i2) = signed_N^{-1}(j1 - j2·truncz(j1 / j2))
  (where j1 = signed_N(i1) ∧ j2 = signed_N(i2))
```

> **Note:** This operator is [partial](exec-op-partial). As long as both operators are defined, it holds that `i1 = i2·idivs(i1, i2) + irems(i1, i2)`.

#### inot_N(i)

* Return the bitwise negation of `i`.

```text
inot_N(i) = ibits_N^{-1}(ibits_N(i) xor ibits_N(2^N-1))
```

#### irev_N(i)

* Return the bitwise reversal of `i`.

```text
irev_N(i) = ibits_N^{-1}((d^N[N-i])^{i ≤ N})   (iff d^N = ibits_N(i))
```

#### iand_N(i1, i2)

* Return the bitwise conjunction of `i1` and `i2`.

```text
iand_N(i1, i2) = ibits_N^{-1}(ibits_N(i1) and ibits_N(i2))
```

#### iandnot_N(i1, i2)

* Return the bitwise conjunction of `i1` and the bitwise negation of `i2`.

```text
iandnot_N(i1, i2) = iand_N(i1, inot_N(i2))
```

#### ior_N(i1, i2)

* Return the bitwise disjunction of `i1` and `i2`.

```text
ior_N(i1, i2) = ibits_N^{-1}(ibits_N(i1) or ibits_N(i2))
```

#### ixor_N(i1, i2)

* Return the bitwise exclusive disjunction of `i1` and `i2`.

```text
ixor_N(i1, i2) = ibits_N^{-1}(ibits_N(i1) xor ibits_N(i2))
```

#### ishl_N(i1, i2)

* Let `k` be `i2` modulo `N`.
* Return the result of shifting `i1` left by `k` bits, modulo `2^N`.

```text
ishl_N(i1, i2) = ibits_N^{-1}(d2^{N-k} 0^k)
  (iff ibits_N(i1) = d1^k d2^{N-k} ∧ k = i2 mod N)
```

#### ishru_N(i1, i2)

* Let `k` be `i2` modulo `N`.
* Return the result of shifting `i1` right by `k` bits, extended with `0` bits.

```text
ishru_N(i1, i2) = ibits_N^{-1}(0^k d1^{N-k})
  (iff ibits_N(i1) = d1^{N-k} d2^k ∧ k = i2 mod N)
```

#### ishrs_N(i1, i2)

* Let `k` be `i2` modulo `N`.
* Return the result of shifting `i1` right by `k` bits, extended with the most significant bit of the original value.

```text
ishrs_N(i1, i2) = ibits_N^{-1}(d0^{k+1} d1^{N-k-1})
  (iff ibits_N(i1) = d0 d1^{N-k-1} d2^k ∧ k = i2 mod N)
```

#### irotl_N(i1, i2)

* Let `k` be `i2` modulo `N`.
* Return the result of rotating `i1` left by `k` bits.

```text
irotl_N(i1, i2) = ibits_N^{-1}(d2^{N-k} d1^k)
  (iff ibits_N(i1) = d1^k d2^{N-k} ∧ k = i2 mod N)
```

#### irotr_N(i1, i2)

* Let `k` be `i2` modulo `N`.
* Return the result of rotating `i1` right by `k` bits.

```text
irotr_N(i1, i2) = ibits_N^{-1}(d2^k d1^{N-k})
  (iff ibits_N(i1) = d1^{N-k} d2^k ∧ k = i2 mod N)
```

#### iclz_N(i)

* Return the count of leading zero bits in `i`; all bits are considered leading zeros if `i` is `0`.

```text
iclz_N(i) = k   (iff ibits_N(i) = 0^k (1 d*)^?)
```

#### ictz_N(i)

* Return the count of trailing zero bits in `i`; all bits are considered trailing zeros if `i` is `0`.

```text
ictz_N(i) = k   (iff ibits_N(i) = (d* 1)^? 0^k)
```

#### ipopcnt_N(i)

* Return the count of non-zero bits in `i`.

```text
ipopcnt_N(i) = k   (iff ibits_N(i) = (0* 1)^k 0*)
```

#### ieqz_N(i)

* Return `1` if `i` is zero, `0` otherwise.

```text
ieqz_N(i) = tobool(i = 0)
```

#### inez_N(i)

* Return `0` if `i` is zero, `1` otherwise.

```text
inez_N(i) = tobool(i ≠ 0)
```

#### ieq_N(i1, i2)

* Return `1` if `i1` equals `i2`, `0` otherwise.

```text
ieq_N(i1, i2) = tobool(i1 = i2)
```

#### ine_N(i1, i2)

* Return `1` if `i1` does not equal `i2`, `0` otherwise.

```text
ine_N(i1, i2) = tobool(i1 ≠ i2)
```

#### iltu_N(i1, i2)

* Return `1` if `i1` is less than `i2`, `0` otherwise.

```text
iltu_N(i1, i2) = tobool(i1 < i2)
```

#### ilts_N(i1, i2)

* Let `j1` be the [signed interpretation](aux-signed) of `i1`.
* Let `j2` be the [signed interpretation](aux-signed) of `i2`.
* Return `1` if `j1` is less than `j2`, `0` otherwise.

```text
ilts_N(i1, i2) = tobool(signed_N(i1) < signed_N(i2))
```

#### igtu_N(i1, i2)

* Return `1` if `i1` is greater than `i2`, `0` otherwise.

```text
igtu_N(i1, i2) = tobool(i1 > i2)
```

#### igts_N(i1, i2)

* Let `j1` be the [signed interpretation](aux-signed) of `i1`.
* Let `j2` be the [signed interpretation](aux-signed) of `i2`.
* Return `1` if `j1` is greater than `j2`, `0` otherwise.

```text
igts_N(i1, i2) = tobool(signed_N(i1) > signed_N(i2))
```

#### ileu_N(i1, i2)

* Return `1` if `i1` is less than or equal to `i2`, `0` otherwise.

```text
ileu_N(i1, i2) = tobool(i1 ≤ i2)
```

#### iles_N(i1, i2)

* Let `j1` be the [signed interpretation](aux-signed) of `i1`.
* Let `j2` be the [signed interpretation](aux-signed) of `i2`.
* Return `1` if `j1` is less than or equal to `j2`, `0` otherwise.

```text
iles_N(i1, i2) = tobool(signed_N(i1) ≤ signed_N(i2))
```

#### igeu_N(i1, i2)

* Return `1` if `i1` is greater than or equal to `i2`, `0` otherwise.

```text
igeu_N(i1, i2) = tobool(i1 ≥ i2)
```

#### iges_N(i1, i2)

* Let `j1` be the [signed interpretation](aux-signed) of `i1`.
* Let `j2` be the [signed interpretation](aux-signed) of `i2`.
* Return `1` if `j1` is greater than or equal to `j2`, `0` otherwise.

```text
iges_N(i1, i2) = tobool(signed_N(i1) ≥ signed_N(i2))
```

#### iextendMs_N(i)

* Let `j` be the result of computing `wrap_{N,M}(i)`.
* Return `extends_{M,N}(j)`.

```text
iextendMs_N(i) = extends_{M,N}(wrap_{N,M}(i))
```

#### ibitselect_N(i1, i2, i3)

* Let `j1` be the bitwise conjunction of `i1` and `i3`.
* Let `j3'` be the bitwise negation of `i3`.
* Let `j2` be the bitwise conjunction of `i2` and `j3'`.
* Return the bitwise disjunction of `j1` and `j2`.

```text
ibitselect_N(i1, i2, i3) = ior_N(iand_N(i1, i3), iand_N(i2, inot_N(i3)))
```

#### iabs_N(i)

* Let `j` be the [signed interpretation](aux-signed) of `i`.
* If `j` is greater than or equal to `0`, then return `i`.
* Else return the negation of `j`, modulo `2^N`.

```text
iabs_N(i) = i            (iff signed_N(i) ≥ 0)
iabs_N(i) = -signed_N(i) mod 2^N   (otherwise)
```

#### ineg_N(i)

* Return the result of negating `i`, modulo `2^N`.

```text
ineg_N(i) = (2^N - i) mod 2^N
```

#### iminu_N(i1, i2)

* Return `i1` if `iltu_N(i1, i2)` is `1`, return `i2` otherwise.

```text
iminu_N(i1, i2) = i1   (iff iltu_N(i1, i2) = 1)
iminu_N(i1, i2) = i2   (otherwise)
```

#### imins_N(i1, i2)

* Return `i1` if `ilts_N(i1, i2)` is `1`, return `i2` otherwise.

```text
imins_N(i1, i2) = i1   (iff ilts_N(i1, i2) = 1)
imins_N(i1, i2) = i2   (otherwise)
```

#### imaxu_N(i1, i2)

* Return `i1` if `igtu_N(i1, i2)` is `1`, return `i2` otherwise.

```text
imaxu_N(i1, i2) = i1   (iff igtu_N(i1, i2) = 1)
imaxu_N(i1, i2) = i2   (otherwise)
```

#### imaxs_N(i1, i2)

* Return `i1` if `igts_N(i1, i2)` is `1`, return `i2` otherwise.

```text
imaxs_N(i1, i2) = i1   (iff igts_N(i1, i2) = 1)
imaxs_N(i1, i2) = i2   (otherwise)
```

#### iaddsatu_N(i1, i2)

* Let `i` be the result of adding `i1` and `i2`.
* Return `satu_N(i)`.

```text
iaddsatu_N(i1, i2) = satu_N(i1 + i2)
```

#### iaddsats_N(i1, i2)

* Let `j1` be the signed interpretation of `i1`.
* Let `j2` be the signed interpretation of `i2`.
* Let `j` be the result of adding `j1` and `j2`.
* Return the value whose signed interpretation is `sats_N(j)`.

```text
iaddsats_N(i1, i2) = signed_N^{-1}(sats_N(signed_N(i1) + signed_N(i2)))
```

#### isubsatu_N(i1, i2)

* Let `i` be the result of subtracting `i2` from `i1`.
* Return `satu_N(i)`.

```text
isubsatu_N(i1, i2) = satu_N(i1 - i2)
```

#### isubsats_N(i1, i2)

* Let `j1` be the signed interpretation of `i1`.
* Let `j2` be the signed interpretation of `i2`.
* Let `j` be the result of subtracting `j2` from `j1`.
* Return the value whose signed interpretation is `sats_N(j)`.

```text
isubsats_N(i1, i2) = signed_N^{-1}(sats_N(signed_N(i1) - signed_N(i2)))
```

#### iavgru_N(i1, i2)

* Let `j` be the result of adding `i1`, `i2`, and `1`.
* Return the result of dividing `j` by `2`, truncated toward zero.

```text
iavgru_N(i1, i2) = truncz((i1 + i2 + 1) / 2)
```

#### iq15mulrsats_N(i1, i2)

* Return the whose signed interpretation is the result of `sats_N(ishrs_N(i1·i2 + 2^{14}, 15))`.

```text
iq15mulrsats_N(i1, i2) = signed_N^{-1}(sats_N(ishrs_N(i1·i2 + 2^{14}, 15)))
```

### Floating-Point Operations

Floating-point arithmetic follows the IEEE754 standard, with the following qualifications:

* All operators use round-to-nearest ties-to-even, except where otherwise specified. Non-default directed rounding attributes are not supported.
* Following the recommendation that operators propagate [NaN](syntax-nan) payloads from their operands is permitted but not required.
* All operators use "non-stop" mode, and floating-point exceptions are not otherwise observable. In particular, neither alternate floating-point exception handling attributes nor operators on status flags are supported. There is no observable difference between quiet and signalling NaNs.

> **Note:** Some of these limitations may be lifted in future versions of WebAssembly.

#### Rounding

Rounding always is round-to-nearest ties-to-even, in correspondence with IEEE754 (Section 4.3.1).

An *exact* floating-point number is a rational number that is exactly representable as a [floating-point number](syntax-float) of given bit width `N`.

A *limit* number for a given floating-point bit width `N` is a positive or negative number whose magnitude is the smallest power of `2` that is not exactly representable as a floating-point number of width `N` (that magnitude is `2^{128}` for `N = 32` and `2^{1024}` for `N = 64`).

A *candidate* number is either an exact floating-point number or a positive or negative limit number for the given bit width `N`.

A *candidate pair* is a pair `z1,z2` of candidate numbers, such that no candidate number exists that lies between the two.

A real number `r` is converted to a floating-point value of bit width `N` as follows:

* If `r` is `0`, then return `+0`.
* Else if `r` is an exact floating-point number, then return `r`.
* Else if `r` greater than or equal to the positive limit, then return `+∞`.
* Else if `r` is less than or equal to the negative limit, then return `-∞`.
* Else if `z1` and `z2` are a candidate pair such that `z1 < r < z2`, then:
  * If `|r - z1| < |r - z2|`, then let `z` be `z1`.
  * Else if `|r - z1| > |r - z2|`, then let `z` be `z2`.
  * Else if `|r - z1| = |r - z2|` and the [significand](syntax-float) of `z1` is even, then let `z` be `z1`.
  * Else, let `z` be `z2`.
* If `z` is `0`, then:
  * If `r < 0`, then return `-0`.
  * Else, return `+0`.
* Else if `z` is a limit number, then:
  * If `r < 0`, then return `-∞`.
  * Else, return `+∞`.
* Else, return `z`.

```text
ieee_N(0) = +0
ieee_N(r) = r          (iff r ∈ F_exact_N)
ieee_N(r) = +∞         (iff r ≥ +F_limit_N)
ieee_N(r) = -∞         (iff r ≤ -F_limit_N)
ieee_N(r) = F_closest_N(r, z1, z2)   (iff z1 < r < z2 ∧ (z1,z2) ∈ F_candidatepair_N)

F_closest_N(r, z1, z2) = F_rectify_N(r, z1)   (iff |r-z1|<|r-z2|)
F_closest_N(r, z1, z2) = F_rectify_N(r, z2)   (iff |r-z1|>|r-z2|)
F_closest_N(r, z1, z2) = F_rectify_N(r, z1)   (iff |r-z1|=|r-z2| ∧ F_even_N(z1))
F_closest_N(r, z1, z2) = F_rectify_N(r, z2)   (iff |r-z1|=|r-z2| ∧ F_even_N(z2))

F_rectify_N(r, ± F_limit_N) = ± ∞
F_rectify_N(r, 0) = +0   (r ≥ 0)
F_rectify_N(r, 0) = -0   (r < 0)
F_rectify_N(r, z) = z
```

where:

```text
F_exact_N                     = fN ∩ ℚ
F_limit_N                     = 2^(2^(expon(N)-1))
F_candidate_N                 = F_exact_N ∪ {+F_limit_N, -F_limit_N}
F_candidatepair_N             = { (z1, z2) ∈ F_candidate_N^2 | z1 < z2 ∧ ∀ z ∈ F_candidate_N, z ≤ z1 ∨ z ≥ z2 }

F_even_N((d + m·2^{-M})·2^e) ⇔ m mod 2 = 0
F_even_N(± F_limit_N)        ⇔ true
```

#### NaN Propagation

When the result of a floating-point operator other than fneg, fabs, or fcopysign is a [NaN](syntax-nan), then its sign is non-deterministic and the [payload](syntax-payload) is computed as follows:

* If the payload of all NaN inputs to the operator is [canonical](canonical-nan) (including the case that there are no NaN inputs), then the payload of the output is canonical as well.
* Otherwise the payload is picked non-deterministically among all [arithmetic NaNs](arithmetic-nan); that is, its most significant bit is `1` and all others are unspecified.
* In the [deterministic profile](profile-deterministic), however, a positive canonical NaNs is reliably produced in the latter case.

The non-deterministic result is expressed by the following auxiliary function producing a set of allowed outputs from a set of inputs:

```text
nans_N{z*}            = { + NAN(canon_N) }
exprofiles(PROFDET) nans_N{z*} = { + NAN(n), - NAN(n) | n = canon_N }   (iff {z*} ⊆ { + NAN(canon_N), - NAN(canon_N) }
exprofiles(PROFDET) nans_N{z*} = { + NAN(n), - NAN(n) | n ≥ canon_N }   (iff {z*} ⊄ { + NAN(canon_N), - NAN(canon_N) }
```

#### fadd_N(z1, z2)

* If either `z1` or `z2` is a NaN, then return an element of `nans_N{z1, z2}`.
* Else if both `z1` and `z2` are infinities of opposite signs, then return an element of `nans_N{}`.
* Else if both `z1` and `z2` are infinities of equal sign, then return that infinity.
* Else if either `z1` or `z2` is an infinity, then return that infinity.
* Else if both `z1` and `z2` are zeroes of opposite sign, then return positive zero.
* Else if both `z1` and `z2` are zeroes of equal sign, then return that zero.
* Else if either `z1` or `z2` is a zero, then return the other operand.
* Else if both `z1` and `z2` are values with the same magnitude but opposite signs, then return positive zero.
* Else return the result of adding `z1` and `z2`, [rounded](aux-ieee) to the nearest representable value.

```text
fadd_N(± NAN(n), z2)        = nans_N{± NAN(n), z2}
fadd_N(z1, ± NAN(n))        = nans_N{± NAN(n), z1}
fadd_N(± ∞, ∓ ∞)            = nans_N{}
fadd_N(± ∞, ± ∞)            = ± ∞
fadd_N(z1, ± ∞)             = ± ∞
fadd_N(± ∞, z2)             = ± ∞
fadd_N(± 0, ∓ 0)            = +0
fadd_N(± 0, ± 0)            = ± 0
fadd_N(z1, ± 0)             = z1
fadd_N(± 0, z2)             = z2
fadd_N(± q, ∓ q)            = +0
fadd_N(z1, z2)              = ieee_N(z1 + z2)
```

#### fsub_N(z1, z2)

* If either `z1` or `z2` is a NaN, then return an element of `nans_N{z1, z2}`.
* Else if both `z1` and `z2` are infinities of equal signs, then return an element of `nans_N{}`.
* Else if both `z1` and `z2` are infinities of opposite sign, then return `z1`.
* Else if `z1` is an infinity, then return that infinity.
* Else if `z2` is an infinity, then return that infinity negated.
* Else if both `z1` and `z2` are zeroes of equal sign, then return positive zero.
* Else if both `z1` and `z2` are zeroes of opposite sign, then return `z1`.
* Else if `z2` is a zero, then return `z1`.
* Else if `z1` is a zero, then return `z2` negated.
* Else if both `z1` and `z2` are the same value, then return positive zero.
* Else return the result of subtracting `z2` from `z1`, [rounded](aux-ieee) to the nearest representable value.

```text
fsub_N(± NAN(n), z2)        = nans_N{± NAN(n), z2}
fsub_N(z1, ± NAN(n))        = nans_N{± NAN(n), z1}
fsub_N(± ∞, ± ∞)            = nans_N{}
fsub_N(± ∞, ∓ ∞)            = ± ∞
fsub_N(z1, ± ∞)             = ∓ ∞
fsub_N(± ∞, z2)             = ± ∞
fsub_N(± 0, ± 0)            = +0
fsub_N(± 0, ∓ 0)            = ± 0
fsub_N(z1, ± 0)             = z1
fsub_N(± 0, ± q2)           = ∓ q2
fsub_N(± q, ± q)            = +0
fsub_N(z1, z2)              = ieee_N(z1 - z2)
```

> **Note:** Up to the non-determinism regarding NaNs, it always holds that `fsub_N(z1, z2) = fadd_N(z1, fneg_N(z2))`.

#### fmul_N(z1, z2)

* If either `z1` or `z2` is a NaN, then return an element of `nans_N{z1, z2}`.
* Else if one of `z1` and `z2` is a zero and the other an infinity, then return an element of `nans_N{}`.
* Else if both `z1` and `z2` are infinities of equal sign, then return positive infinity.
* Else if both `z1` and `z2` are infinities of opposite sign, then return negative infinity.
* Else if either `z1` or `z2` is an infinity and the other a value with equal sign, then return positive infinity.
* Else if either `z1` or `z2` is an infinity and the other a value with opposite sign, then return negative infinity.
* Else if both `z1` and `z2` are zeroes of equal sign, then return positive zero.
* Else if both `z1` and `z2` are zeroes of opposite sign, then return negative zero.
* Else return the result of multiplying `z1` and `z2`, [rounded](aux-ieee) to the nearest representable value.

```text
fmul_N(± NAN(n), z2)        = nans_N{± NAN(n), z2}
fmul_N(z1, ± NAN(n))        = nans_N{± NAN(n), z1}
fmul_N(± ∞, ± 0)            = nans_N{}
fmul_N(± ∞, ∓ 0)            = nans_N{}
fmul_N(± 0, ± ∞)            = nans_N{}
fmul_N(± 0, ∓ ∞)            = nans_N{}
fmul_N(± ∞, ± ∞)            = +∞
fmul_N(± ∞, ∓ ∞)            = -∞
fmul_N(± q1, ± ∞)           = +∞
fmul_N(± q1, ∓ ∞)           = -∞
fmul_N(± ∞, ± q2)           = +∞
fmul_N(± ∞, ∓ q2)           = -∞
fmul_N(± 0, ± 0)            = + 0
fmul_N(± 0, ∓ 0)            = - 0
fmul_N(z1, z2)              = ieee_N(z1 · z2)
```

#### fdiv_N(z1, z2)

* If either `z1` or `z2` is a NaN, then return an element of `nans_N{z1, z2}`.
* Else if both `z1` and `z2` are infinities, then return an element of `nans_N{}`.
* Else if both `z1` and `z2` are zeroes, then return an element of `nans_N{z1, z2}`.
* Else if `z1` is an infinity and `z2` a value with equal sign, then return positive infinity.
* Else if `z1` is an infinity and `z2` a value with opposite sign, then return negative infinity.
* Else if `z2` is an infinity and `z1` a value with equal sign, then return positive zero.
* Else if `z2` is an infinity and `z1` a value with opposite sign, then return negative zero.
* Else if `z1` is a zero and `z2` a value with equal sign, then return positive zero.
* Else if `z1` is a zero and `z2` a value with opposite sign, then return negative zero.
* Else if `z2` is a zero and `z1` a value with equal sign, then return positive infinity.
* Else if `z2` is a zero and `z1` a value with opposite sign, then return negative infinity.
* Else return the result of dividing `z1` by `z2`, [rounded](aux-ieee) to the nearest representable value.

```text
fdiv_N(± NAN(n), z2)        = nans_N{± NAN(n), z2}
fdiv_N(z1, ± NAN(n))        = nans_N{± NAN(n), z1}
fdiv_N(± ∞, ± ∞)            = nans_N{}
fdiv_N(± ∞, ∓ ∞)            = nans_N{}
fdiv_N(± 0, ± 0)            = nans_N{}
fdiv_N(± 0, ∓ 0)            = nans_N{}
fdiv_N(± ∞, ± q2)           = +∞
fdiv_N(± ∞, ∓ q2)           = -∞
fdiv_N(± q1, ± ∞)           = +0
fdiv_N(± q1, ∓ ∞)           = -0
fdiv_N(± 0, ± q2)           = +0
fdiv_N(± 0, ∓ q2)           = -0
fdiv_N(± q1, ± 0)           = +∞
fdiv_N(± q1, ∓ 0)           = -∞
fdiv_N(z1, z2)              = ieee_N(z1 / z2)
```

#### fma_N(z1, z2, z3)

The function `fma` is the same as *fusedMultiplyAdd* defined by IEEE754 (Section 5.4.1). It computes `(z1·z2) + z3` as if with unbounded range and precision, rounding only once for the final result.

* If either `z1` or `z2` or `z3` is a NaN, return an element of `nans_N{z1, z2, z3}`.
* Else if either `z1` or `z2` is a zero and the other is an infinity, then return an element of `nans_N{}`.
* Else if both `z1` or `z2` are infinities of equal sign, and `z3` is a negative infinity, then return an element of `nans_N{}`.
* Else if both `z1` or `z2` are infinities of opposite sign, and `z3` is a positive infinity, then return an element of `nans_N{}`.
* Else if either `z1` or `z2` is an infinity and the other is a value of the same sign, and `z3` is a negative infinity, then return an element of `nans_N{}`.
* Else if either `z1` or `z2` is an infinity and the other is a value of the opposite sign, and `z3` is a positive infinity, then return an element of `nans_N{}`.
* Else if both `z1` and `z2` are zeroes of the same sign and `z3` is a zero, then return positive zero.
* Else if both `z1` and `z2` are zeroes of the opposite sign and `z3` is a positive zero, then return positive zero.
* Else if both `z1` and `z2` are zeroes of the opposite sign and `z3` is a negative zero, then return negative zero.
* Else return the result of multiplying `z1` and `z2`, adding `z3` to the intermediate, and the final result [rounded](aux-ieee) to the nearest representable value.

```text
fma_N(± NAN(n), z2, z3)                = nans_N{± NAN(n), z2, z3}
fma_N(z1, ± NAN(n), z3)                = nans_N{± NAN(n), z1, z3}
fma_N(z1, z2, ± NAN(n))                = nans_N{± NAN(n), z1, z2}
fma_N(± ∞, ± 0, z3)                    = nans_N{}
fma_N(± ∞, ∓ 0, z3)                    = nans_N{}
fma_N(± ∞, ± ∞, - ∞)                   = nans_N{}
fma_N(± ∞, ∓ ∞, + ∞)                   = nans_N{}
fma_N(± q1, ± ∞, - ∞)                  = nans_N{}
fma_N(± q1, ∓ ∞, + ∞)                  = nans_N{}
fma_N(± ∞, ± q1, - ∞)                  = nans_N{}
fma_N(∓ ∞, ± q1, + ∞)                  = nans_N{}
fma_N(± 0, ± 0, ∓ 0)                   = + 0
fma_N(± 0, ± 0, ± 0)                   = + 0
fma_N(± 0, ∓ 0, + 0)                   = + 0
fma_N(± 0, ∓ 0, - 0)                   = - 0
fma_N(z1, z2, z3)                      = ieee_N(z1 · z2 + z3)
```

#### fmin_N(z1, z2)

* If either `z1` or `z2` is a NaN, then return an element of `nans_N{z1, z2}`.
* Else if either `z1` or `z2` is a negative infinity, then return negative infinity.
* Else if either `z1` or `z2` is a positive infinity, then return the other value.
* Else if both `z1` and `z2` are zeroes of opposite signs, then return negative zero.
* Else return the smaller value of `z1` and `z2`.

```text
fmin_N(± NAN(n), z2)    = nans_N{± NAN(n), z2}
fmin_N(z1, ± NAN(n))    = nans_N{± NAN(n), z1}
fmin_N(+ ∞, z2)         = z2
fmin_N(- ∞, z2)         = - ∞
fmin_N(z1, + ∞)         = z1
fmin_N(z1, - ∞)         = - ∞
fmin_N(± 0, ∓ 0)        = -0
fmin_N(z1, z2)          = z1   (iff z1 ≤ z2)
fmin_N(z1, z2)          = z2   (iff z2 ≤ z1)
```

#### fmax_N(z1, z2)

* If either `z1` or `z2` is a NaN, then return an element of `nans_N{z1, z2}`.
* Else if either `z1` or `z2` is a positive infinity, then return positive infinity.
* Else if either `z1` or `z2` is a negative infinity, then return the other value.
* Else if both `z1` and `z2` are zeroes of opposite signs, then return positive zero.
* Else return the larger value of `z1` and `z2`.

```text
fmax_N(± NAN(n), z2)    = nans_N{± NAN(n), z2}
fmax_N(z1, ± NAN(n))    = nans_N{± NAN(n), z1}
fmax_N(+ ∞, z2)         = + ∞
fmax_N(- ∞, z2)         = z2
fmax_N(z1, + ∞)         = + ∞
fmax_N(z1, - ∞)         = z1
fmax_N(± 0, ∓ 0)        = +0
fmax_N(z1, z2)          = z1   (iff z1 ≥ z2)
fmax_N(z1, z2)          = z2   (iff z2 ≥ z1)
```

#### fcopysign_N(z1, z2)

* If `z1` and `z2` have the same sign, then return `z1`.
* Else return `z1` with negated sign.

```text
fcopysign_N(± p1, ± p2) = ± p1
fcopysign_N(± p1, ∓ p2) = ∓ p1
```

#### fabs_N(z)

* If `z` is a NaN, then return `z` with positive sign.
* Else if `z` is an infinity, then return positive infinity.
* Else if `z` is a zero, then return positive zero.
* Else if `z` is a positive value, then `z`.
* Else return `z` negated.

```text
fabs_N(± NAN(n)) = +NAN(n)
fabs_N(± ∞)      = +∞
fabs_N(± 0)      = +0
fabs_N(± q)      = +q
```

#### fneg_N(z)

* If `z` is a NaN, then return `z` with negated sign.
* Else if `z` is an infinity, then return that infinity negated.
* Else if `z` is a zero, then return that zero negated.
* Else return `z` negated.

```text
fneg_N(± NAN(n)) = ∓ NAN(n)
fneg_N(± ∞)      = ∓ ∞
fneg_N(± 0)      = ∓ 0
fneg_N(± q)      = ∓ q
```

#### fsqrt_N(z)

* If `z` is a NaN, then return an element of `nans_N{z}`.
* Else if `z` is negative infinity, then return an element of `nans_N{}`.
* Else if `z` is positive infinity, then return positive infinity.
* Else if `z` is a zero, then return that zero.
* Else if `z` has a negative sign, then return an element of `nans_N{}`.
* Else return the square root of `z`.

```text
fsqrt_N(± NAN(n)) = nans_N{± NAN(n)}
fsqrt_N(- ∞)      = nans_N{}
fsqrt_N(+ ∞)      = + ∞
fsqrt_N(± 0)      = ± 0
fsqrt_N(- q)      = nans_N{}
fsqrt_N(+ q)      = ieee_N(√(q))
```

#### fceil_N(z)

* If `z` is a NaN, then return an element of `nans_N{z}`.
* Else if `z` is an infinity, then return `z`.
* Else if `z` is a zero, then return `z`.
* Else if `z` is smaller than `0` but greater than `-1`, then return negative zero.
* Else return the smallest integral value that is not smaller than `z`.

```text
fceil_N(± NAN(n)) = nans_N{± NAN(n)}
fceil_N(± ∞)      = ± ∞
fceil_N(± 0)      = ± 0
fceil_N(- q)      = -0   (iff -1 < -q < 0)
fceil_N(± q)      = ieee_N(i)   (iff ± q ≤ i < ± q + 1)
```

#### ffloor_N(z)

* If `z` is a NaN, then return an element of `nans_N{z}`.
* Else if `z` is an infinity, then return `z`.
* Else if `z` is a zero, then return `z`.
* Else if `z` is greater than `0` but smaller than `1`, then return positive zero.
* Else return the largest integral value that is not larger than `z`.

```text
ffloor_N(± NAN(n)) = nans_N{± NAN(n)}
ffloor_N(± ∞)      = ± ∞
ffloor_N(± 0)      = ± 0
ffloor_N(+ q)      = +0   (iff 0 < +q < 1)
ffloor_N(± q)      = ieee_N(i)   (iff ± q - 1 < i ≤ ± q)
```

#### ftrunc_N(z)

* If `z` is a NaN, then return an element of `nans_N{z}`.
* Else if `z` is an infinity, then return `z`.
* Else if `z` is a zero, then return `z`.
* Else if `z` is greater than `0` but smaller than `1`, then return positive zero.
* Else if `z` is smaller than `0` but greater than `-1`, then return negative zero.
* Else return the integral value with the same sign as `z` and the largest magnitude that is not larger than the magnitude of `z`.

```text
ftrunc_N(± NAN(n)) = nans_N{± NAN(n)}
ftrunc_N(± ∞)      = ± ∞
ftrunc_N(± 0)      = ± 0
ftrunc_N(+ q)      = +0   (iff 0 < +q < 1)
ftrunc_N(- q)      = -0   (iff -1 < -q < 0)
ftrunc_N(± q)      = ieee_N(± i)   (iff +q - 1 < i ≤ +q)
```

#### fnearest_N(z)

* If `z` is a NaN, then return an element of `nans_N{z}`.
* Else if `z` is an infinity, then return `z`.
* Else if `z` is a zero, then return `z`.
* Else if `z` is greater than `0` but smaller than or equal to `0.5`, then return positive zero.
* Else if `z` is smaller than `0` but greater than or equal to `-0.5`, then return negative zero.
* Else return the integral value that is nearest to `z`; if two values are equally near, return the even one.

```text
fnearest_N(± NAN(n)) = nans_N{± NAN(n)}
fnearest_N(± ∞)      = ± ∞
fnearest_N(± 0)      = ± 0
fnearest_N(+ q)      = +0   (iff 0 < +q ≤ 0.5)
fnearest_N(- q)      = -0   (iff -0.5 ≤ -q < 0)
fnearest_N(± q)      = ieee_N(± i)   (iff |i - q| < 0.5)
fnearest_N(± q)      = ieee_N(± i)   (iff |i - q| = 0.5 ∧ i even)
```

#### feq_N(z1, z2)

* If either `z1` or `z2` is a NaN, then return `0`.
* Else if both `z1` and `z2` are zeroes, then return `1`.
* Else if both `z1` and `z2` are the same value, then return `1`.
* Else return `0`.

```text
feq_N(± NAN(n), z2) = 0
feq_N(z1, ± NAN(n)) = 0
feq_N(± 0, ∓ 0)     = 1
feq_N(z1, z2)       = tobool(z1 = z2)
```

#### fne_N(z1, z2)

* If either `z1` or `z2` is a NaN, then return `1`.
* Else if both `z1` and `z2` are zeroes, then return `0`.
* Else if both `z1` and `z2` are the same value, then return `0`.
* Else return `1`.

```text
fne_N(± NAN(n), z2) = 1
fne_N(z1, ± NAN(n)) = 1
fne_N(± 0, ∓ 0)     = 0
fne_N(z1, z2)       = tobool(z1 ≠ z2)
```

#### flt_N(z1, z2)

* If either `z1` or `z2` is a NaN, then return `0`.
* Else if `z1` and `z2` are the same value, then return `0`.
* Else if `z1` is positive infinity, then return `0`.
* Else if `z1` is negative infinity, then return `1`.
* Else if `z2` is positive infinity, then return `1`.
* Else if `z2` is negative infinity, then return `0`.
* Else if both `z1` and `z2` are zeroes, then return `0`.
* Else if `z1` is smaller than `z2`, then return `1`.
* Else return `0`.

```text
flt_N(± NAN(n), z2) = 0
flt_N(z1, ± NAN(n)) = 0
flt_N(z, z)         = 0
flt_N(+ ∞, z2)      = 0
flt_N(- ∞, z2)      = 1
flt_N(z1, + ∞)      = 1
flt_N(z1, - ∞)      = 0
flt_N(± 0, ∓ 0)     = 0
flt_N(z1, z2)       = tobool(z1 < z2)
```

#### fgt_N(z1, z2)

* If either `z1` or `z2` is a NaN, then return `0`.
* Else if `z1` and `z2` are the same value, then return `0`.
* Else if `z1` is positive infinity, then return `1`.
* Else if `z1` is negative infinity, then return `0`.
* Else if `z2` is positive infinity, then return `0`.
* Else if `z2` is negative infinity, then return `1`.
* Else if both `z1` and `z2` are zeroes, then return `0`.
* Else if `z1` is larger than `z2`, then return `1`.
* Else return `0`.

```text
fgt_N(± NAN(n), z2) = 0
fgt_N(z1, ± NAN(n)) = 0
fgt_N(z, z)         = 0
fgt_N(+ ∞, z2)      = 1
fgt_N(- ∞, z2)      = 0
fgt_N(z1, + ∞)      = 0
fgt_N(z1, - ∞)      = 1
fgt_N(± 0, ∓ 0)     = 0
fgt_N(z1, z2)       = tobool(z1 > z2)
```

#### fle_N(z1, z2)

* If either `z1` or `z2` is a NaN, then return `0`.
* Else if `z1` and `z2` are the same value, then return `1`.
* Else if `z1` is positive infinity, then return `0`.
* Else if `z1` is negative infinity, then return `1`.
* Else if `z2` is positive infinity, then return `1`.
* Else if `z2` is negative infinity, then return `0`.
* Else if both `z1` and `z2` are zeroes, then return `1`.
* Else if `z1` is smaller than or equal to `z2`, then return `1`.
* Else return `0`.

```text
fle_N(± NAN(n), z2) = 0
fle_N(z1, ± NAN(n)) = 0
fle_N(z, z)         = 1
fle_N(+ ∞, z2)      = 0
fle_N(- ∞, z2)      = 1
fle_N(z1, + ∞)      = 1
fle_N(z1, - ∞)      = 0
fle_N(± 0, ∓ 0)     = 1
fle_N(z1, z2)       = tobool(z1 ≤ z2)
```

#### fge_N(z1, z2)

* If either `z1` or `z2` is a NaN, then return `0`.
* Else if `z1` and `z2` are the same value, then return `1`.
* Else if `z1` is positive infinity, then return `1`.
* Else if `z1` is negative infinity, then return `0`.
* Else if `z2` is positive infinity, then return `0`.
* Else if `z2` is negative infinity, then return `1`.
* Else if both `z1` and `z2` are zeroes, then return `1`.
* Else if `z1` is larger than or equal to `z2`, then return `1`.
* Else return `0`.

```text
fge_N(± NAN(n), z2) = 0
fge_N(z1, ± NAN(n)) = 0
fge_N(z, z)         = 1
fge_N(+ ∞, z2)      = 1
fge_N(- ∞, z2)      = 0
fge_N(z1, + ∞)      = 0
fge_N(z1, - ∞)      = 1
fge_N(± 0, ∓ 0)     = 1
fge_N(z1, z2)       = tobool(z1 ≥ z2)
```

#### fpmin_N(z1, z2)

* If `z2` is less than `z1` then return `z2`.
* Else return `z1`.

```text
fpmin_N(z1, z2) = z2   (iff flt_N(z2, z1) = 1)
fpmin_N(z1, z2) = z1   (otherwise)
```

#### fpmax_N(z1, z2)

* If `z1` is less than `z2` then return `z2`.
* Else return `z1`.

```text
fpmax_N(z1, z2) = z2   (iff flt_N(z1, z2) = 1)
fpmax_N(z1, z2) = z1   (otherwise)
```

### Conversions

#### extendu_{M,N}(i)

* Return `i`.

```text
extendu_{M,N}(i) = i
```

> **Note:** In the abstract syntax, unsigned extension just reinterprets the same value.

#### extends_{M,N}(i)

* Let `j` be the [signed interpretation](aux-signed) of `i` of size `M`.
* Return the two's complement of `j` relative to size `N`.

```text
extends_{M,N}(i) = signed_N^{-1}(signed_M(i))
```

#### wrap_{M,N}(i)

* Return `i` modulo `2^N`.

```text
wrap_{M,N}(i) = i mod 2^N
```

#### truncu_{M,N}(z)

* If `z` is a NaN, then the result is undefined.
* Else if `z` is an infinity, then the result is undefined.
* Else if `z` is a number and `truncz(z)` is a value within range of the target type, then return that value.
* Else the result is undefined.

```text
truncu_{M,N}(± NAN(n)) = {}
truncu_{M,N}(± ∞)      = {}
truncu_{M,N}(± q)      = truncz(± q)   (iff -1 < truncz(± q) < 2^N)
truncu_{M,N}(± q)      = {}            (otherwise)
```

> **Note:** This operator is [partial](exec-op-partial). It is not defined for NaNs, infinities, or values for which the result is out of range.

#### truncs_{M,N}(z)

* If `z` is a NaN, then the result is undefined.
* Else if `z` is an infinity, then the result is undefined.
* If `z` is a number and `truncz(z)` is a value within range of the target type, then return that value.
* Else the result is undefined.

```text
truncs_{M,N}(± NAN(n)) = {}
truncs_{M,N}(± ∞)      = {}
truncs_{M,N}(± q)      = truncz(± q)   (iff -2^{N-1} - 1 < truncz(± q) < 2^{N-1})
truncs_{M,N}(± q)      = {}            (otherwise)
```

> **Note:** This operator is [partial](exec-op-partial). It is not defined for NaNs, infinities, or values for which the result is out of range.

#### truncsatu_{M,N}(z)

* If `z` is a NaN, then return `0`.
* Else if `z` is negative infinity, then return `0`.
* Else if `z` is positive infinity, then return `2^N - 1`.
* Else, return `satu_N(truncz(z))`.

```text
truncsatu_{M,N}(± NAN(n)) = 0
truncsatu_{M,N}(- ∞)      = 0
truncsatu_{M,N}(+ ∞)      = 2^N - 1
truncsatu_{M,N}(z)        = satu_N(truncz(z))
```

#### truncsats_{M,N}(z)

* If `z` is a NaN, then return `0`.
* Else if `z` is negative infinity, then return `-2^{N-1}`.
* Else if `z` is positive infinity, then return `2^{N-1} - 1`.
* Else, return the value whose signed interpretation is `sats_N(trunc(z))`.

```text
truncsats_{M,N}(± NAN(n)) = 0
truncsats_{M,N}(- ∞)      = -2^{N-1}
truncsats_{M,N}(+ ∞)      = 2^{N-1}-1
truncsats_{M,N}(z)        = signed_N^{-1}(sats_N(trunc(z)))
```

#### promote_{M,N}(z)

* If `z` is a [canonical NaN](canonical-nan), then return an element of `nans_N{}` (i.e., a canonical NaN of size `N`).
* Else if `z` is a NaN, then return an element of `nans_N{± NAN(1)}` (i.e., any [arithmetic NaN](arithmetic-nan) of size `N`).
* Else, return `z`.

```text
promote_{M,N}(± NAN(n)) = nans_N{}            (iff n = canon_N)
promote_{M,N}(± NAN(n)) = nans_N{+ NAN(1)}    (otherwise)
promote_{M,N}(z)        = z
```

#### demote_{M,N}(z)

* If `z` is a [canonical NaN](canonical-nan), then return an element of `nans_N{}` (i.e., a canonical NaN of size `N`).
* Else if `z` is a NaN, then return an element of `nans_N{± NAN(1)}` (i.e., any NaN of size `N`).
* Else if `z` is an infinity, then return that infinity.
* Else if `z` is a zero, then return that zero.
* Else, return `ieee_N(z)`.

```text
demote_{M,N}(± NAN(n)) = nans_N{}            (iff n = canon_N)
demote_{M,N}(± NAN(n)) = nans_N{+ NAN(1)}    (otherwise)
demote_{M,N}(± ∞)      = ± ∞
demote_{M,N}(± 0)      = ± 0
demote_{M,N}(± q)      = ieee_N(± q)
```

#### convertu_{M,N}(i)

* Return `ieee_N(i)`.

```text
convertu_{M,N}(i) = ieee_N(i)
```

#### converts_{M,N}(i)

* Let `j` be the [signed interpretation](aux-signed) of `i`.
* Return `ieee_N(j)`.

```text
converts_{M,N}(i) = ieee_N(signed_M(i))
```

#### reinterpret_{t1,t2}(c)

* Let `d*` be the bit sequence `bits_{t1}(c)`.
* Return the constant `c'` for which `bits_{t2}(c') = d*`.

```text
reinterpret_{t1,t2}(c) = bits_{t2}^{-1}(bits_{t1}(c))
```

#### narrows_{M,N}(i)

* Let `j` be the [signed interpretation](aux-signed) of `i` of size `M`.
* Return the value whose signed interpretation is `sats_N(j)`.

```text
narrows_{M,N}(i) = signed_N^{-1}(sats_N(signed_M(i)))
```

#### narrowu_{M,N}(i)

* Let `j` be the [signed interpretation](aux-signed) of `i` of size `M`.
* Return `satu_N(j)`.

```text
narrowu_{M,N}(i) = satu_N(signed_M(i))
```

### Vector Operations

Most vector operations are performed by applying numeric operations lanewise. However, some operators consider multiple lanes at once.

#### ivbitmask_N(i^m)

1. For each `i_k` in `i^m`, let `b_k` be the result of computing `ilts_N(i, 0)`.
2. Let `b^m` be the concatenation of all `b_k`.
3. Return the result of computing `ibits_{32}^{-1}((0)^{32-m}~b^m)`.

```text
ivbitmask_N(i^m) = ibits_{32}^{-1}((0)^{32-m} ilts_N(i, 0)^m)
```

#### ivswizzle(i^n, j^n)

1. For each `j_k` in `j^n`, let `r_k` be the value `ivswizzlelane(i^n, j_k)`.
2. Let `r^n` be the concatenation of all `r_k`.
3. Return `r^n`.

```text
ivswizzle(i^n, j^n) = ivswizzlelane(i^n, j)^n
```

where:

```text
ivswizzlelane(i^n, j) = i^n[j]   (iff j < n)
ivswizzlelane(i^n, j) = 0         (otherwise)
```

#### ivshuffle(j^n, i1^n, i2^n)

1. Let `i*` be the concatenation of `i1^n` and `i2^n`.
2. For each `j_k` in `j^n`, let `r_k` be `i*[j_k]`.
3. Let `r^n` be the concatenation of all `r_k`.
4. Return `r^n`.

```text
ivshuffle(j^n, i1^n, i2^n) = ((i1^n i2^n)[j])^n   (iff (j < 2·n)^n)
```

#### ivaddpairwise_N(i^{2m})

1. Let `(i1 i2)^m` be `i^{2m}`, decomposed into pairwise elements.
2. For each `i1_k` in `i1^m` and corresponding `i2_k` in `i2^m`, let `r_k` be `iadd_N(i1_k, i2_k)`.
3. Let `r^m` be the concatenation of all `r_k`.
4. Return `r^m`.

```text
ivaddpairwise_N(i^{2m}) = (iadd_N(i1, i2))^m   (iff i^{2m} = (i1 i2)^m)
```

#### ivmul_N(i1^m, i2^m)

1. For each `i1_k` in `i1^m` and corresponding `i2_k` in `i2^m`, let `r_k` be `imul_N(i1_k, i2_k)`.
2. Let `r^m` be the concatenation of all `r_k`.
3. Return `r^m`.

```text
ivmul_N(i1^m, i2^m) = (imul_N(i1, i2))^m
```

#### ivdot_N(i1^{2m}, i2^{2m})

1. For each `i1_k` in `i1^{2m}` and corresponding `i2_k` in `i2^{2m}`, let `j_k` be `imul_N(i1_k, i2_k)`.
2. Let `j^{2m}` be the concatenation of all `j_k`.
3. Let `(j1 j2)^m` be `j^{2m}`, decomposed into pairwise elements.
4. For each `i1_k` in `i1^m` and corresponding `i2_k` in `i2^m`, let `r_k` be `iadd_N(i1_k, i2_k)`.
5. Let `r^m` be the concatenation of all `r_k`.
6. Return `r^m`.

```text
ivdot_N(i1^{2m}, i2^{2m}) = (iadd_N(j1, j2))^m   (iff (imul_N(i1, i2))^{2m} = (j1 j2)^m)
```

#### ivdotsat_N(i1^{2m}, i2^{2m})

1. For each `i1_k` in `i1^{2m}` and corresponding `i2_k` in `i2^{2m}`, let `j_k` be `imul_N(i1_k, i2_k)`.
2. Let `j^{2m}` be the concatenation of all `j_k`.
3. Let `(j1 j2)^m` be `j^{2m}`, decomposed into pairwise elements.
4. For each `i1_k` in `i1^m` and corresponding `i2_k` in `i2^m`, let `r_k` be `iaddsat_N(i1_k, i2_k)`.
5. Let `r^m` be the concatenation of all `r_k`.
6. Return `r^m`.

```text
ivdotsat_N(i1^{2m}, i2^{2m}) = (iaddsat_N(j1, j2))^m   (iff (imul_N(i1, i2))^{2m} = (j1 j2)^m)
```

The previous operators are lifted to operators on arguments of vector type by wrapping them in corresponding lane projections and injections and intermediate extension operations:

#### vextunop_{sh1, sh2}(c)

```text
vextunop_{iN1 x M1, iN2 x M2}(c) = lanes_{iN2 x M2}^{-1}(j*)
  (iff i*    = lanes_{iN1 x M1}(c)
   ∧ i'*     = extend^{sx}_{N1, N2}(i)*
   ∧ j*      = ivaddpairwise_{N2}(i'*))
```

#### vextbinop_{sh1, sh2}(c1, c2)

```text
vextbinop_{iN1 x M1, iN2 x M2}(c1, c2) = lanes_{iN2 x M2}^{-1}(j*)
  (iff i1*    = lanes_{iN1 x M1}(c1)[h slice k]
   ∧ i2*      = lanes_{iN1 x M1}(c2)[h slice k]
   ∧ i1'*     = extend^{sx}_{N1, N2}(i1)*
   ∧ i2'*     = extend^{sx}_{N1, N2}(i2)*
   ∧ j*       = f_{N2}(i1'*, i2'*))
```

where `f`, `sx1`, `sx2`, `h`, and `k` are instantiated as follows, depending on the operator:

```text
vextbinop        f        sx1   sx2   h    k
────────────────────────────────────────────
vextmul_low_sx   ivmul    sx    sx    0    M2
vextmul_high_sx  ivmul    sx    sx    M2   M2
vdot_s           ivdot    S     S     0    M1
vrelaxeddot_s    ivdotsat S     relaxed(R_idot)[ S, U ]  0   M1
```

> **Note:** Relaxed operations and the paramater `R_idot` are introduced [below](relaxed-ops).

#### vextternop_{sh1, sh2}(c1, c2, c3)

```text
vrelaxeddotadd_S_{iN1 x M1, iN2 x M2}(c1, c2, c3) = c
  (iff N   = 2·N1
   ∧ M     = 2·M2
   ∧ c'    = vrelaxeddot_S_{iN1 x M1, iN x M}(c1, c2)
   ∧ c''   = vextaddpairwise_S_{iN x M, iN2 x M2}(c')
   ∧ c ∈ vadd_{iN2 x M2}(c'', c3))
```

#### vnarrow_{sx, sh1, sh2}(c1, c2)

```text
vnarrow_sx_{iN1 x M1, iN2 x M2}(c1, c2) = lanes_{iN2 x M2}^{-1}(j*)
  (iff i1*    = lanes_{iN1 x M1}(c1)
   ∧ i2*      = lanes_{iN1 x M1}(c2)
   ∧ i1'*     = narrow^{sx}_{N1, N2}(i1)*
   ∧ i2'*     = narrow^{sx}_{N1, N2}(i2)*
   ∧ j*       = i1'* ⊕ i1'*)
```

#### vcvtop_half?_zero?_{sh1, sh2}(i)

```text
vcvtop_half?_zero?_{t1 x M1, t2 x M2}(i) = j
  (iff condition
   ∧ c*        = lanes_{t1 x M1}(i)[h slice k]
   ∧ c'* *     = × (vcvtop_{|t1|, |t2|}(c)* ⊕ (0)^n)
   ∧ j ∈ lanes_{t2 x M2}^{-1}(c'*)*)
```

where `h`, `k`, `n`, and `condition` are instantiated as follows, depending on the operator:

```text
half?    zero?    h      k      n      condition
─────────────────────────────────────────────────
ε        ε        0      M1     0      (M1 = M2)
LOW      ε        0      M2     0      (M1 = 2·M2)
HIGH     ε        M2     M2     0      (M1 = 2·M2)
ε        ZERO     0      M1     M1     (2·M1 = M2)
```

while `×(x*)^N` transforms a sequence of `N` sets of non-deterministic values into a set of non-deterministic sequences of `N` values by computing the set product:

```text
×(S1 ... SN) = { x1 ... xN | x1 ∈ S1 ∧ ... ∧ xN ∈ SN }
```

### Relaxed Operations

The result of *relaxed* operators are *implementation-dependent*, because the set of possible results may depend on properties of the host environment, such as its hardware. Technically, their behaviour is controlled by a set of *global parameters* to the semantics that an implementation can instantiate in different ways. These choices are fixed, that is, parameters are constant during the execution of any given program.

Every such parameter is an index into a sequence of possible sets of results and must be instantiated to a defined index. In the [deterministic profile](profile-deterministic), every parameter is prescribed to be 0. This behaviour is expressed by the following auxiliary function, where `R` is a global parameter selecting one of the allowed outcomes:

```text
exprofiles(PROFDET) relaxed(R)[ A0, ..., An ] = A_R
                     relaxed(R)[ A0, ..., An ] = A_0
```

> **Note:** Each parameter can be thought of as inducing a family of operations that is fixed to one particular choice by an implementation. The fixed operation itself can still be non-deterministic or partial.
>
> Implementations are expexted to either choose the behaviour that is the most efficient on the underlying hardware, or the behaviour of the deterministic profile.

#### frelaxedmadd_N(z1, z2, z3)

The implementation-specific behaviour of this operation is determined by the global parameter `R_fmadd ∈ {0, 1}`.

* Return `relaxed(R_fmadd)[ fadd_N(fmul_N(z1, z2), z3), fma_N(z1, z2, z3) ]`.

```text
frelaxedmadd_N(z1, z2, z3) = relaxed(R_fmadd)[ fadd_N(fmul_N(z1, z2), z3), fma_N(z1, z2, z3) ]
```

> **Note:** Relaxed multiply-add allows for fused or unfused results, which leads to implementation-dependent rounding behaviour. In the [deterministic profile](profile-deterministic), the unfused behaviour is used.

#### frelaxednmadd_N(z1, z2, z3)

* Return `frelaxedmadd(-z1, z2, z3)`.

```text
frelaxednmadd_N(z1, z2, z3) = frelaxedmadd_N(-z1, z2, z3)
```

> **Note:** This operation is implementation-dependent because `frelaxedmadd` is implementation-dependent.

#### frelaxedmin_N(z1, z2)

The implementation-specific behaviour of this operation is determined by the global parameter `R_fmin ∈ {0, 1, 2, 3}`.

* If `z1` is a NaN, then return `relaxed(R_fmin)[ fmin_N(z1, z2), NAN(n), z2, z2 ]`.
* If `z2` is a NaN, then return `relaxed(R_fmin)[ fmin_N(z1, z2), z1, NAN(n), z1 ]`.
* If both `z1` and `z2` are zeroes of opposite sign, then return `relaxed(R_fmin)[ fmin_N(z1, z2), ± 0, ∓ 0, -0 ]`.
* Return `fmin_N(z1, z2)`.

```text
frelaxedmin_N(± NAN(n), z2) = relaxed(R_fmin)[ fmin_N(± NAN(n), z2), NAN(n), z2, z2 ]
frelaxedmin_N(z1, ± NAN(n)) = relaxed(R_fmin)[ fmin_N(z1, ± NAN(n)), z1, NAN(n), z1 ]
frelaxedmin_N(± 0, ∓ 0)     = relaxed(R_fmin)[ fmin_N(± 0, ∓ 0), ± 0, ∓ 0, -0 ]
frelaxedmin_N(z1, z2)       = fmin_N(z1, z2)   (otherwise)
```

> **Note:** Relaxed minimum is implementation-dependent for NaNs and for zeroes with different signs. In the [deterministic profile](profile-deterministic), it behaves like regular `fmin`.

#### frelaxedmax_N(z1, z2)

The implementation-specific behaviour of this operation is determined by the global parameter `R_fmax ∈ {0, 1, 2, 3}`.

* If `z1` is a NaN, then return `relaxed(R_fmax)[ fmax_N(z1, z2), NAN(n), z2, z2 ]`.
* If `z2` is a NaN, then return `relaxed(R_fmax)[ fmax_N(z1, z2), z1, NAN(n), z1 ]`.
* If both `z1` and `z2` are zeroes of opposite sign, then return `relaxed(R_fmax)[ fmax_N(z1, z2), ± 0, ∓ 0, +0 ]`.
* Return `fmax_N(z1, z2)`.

```text
frelaxedmax_N(± NAN(n), z2) = relaxed(R_fmax)[ fmax_N(± NAN(n), z2), NAN(n), z2, z2 ]
frelaxedmax_N(z1, ± NAN(n)) = relaxed(R_fmax)[ fmax_N(z1, ± NAN(n)), z1, NAN(n), z1 ]
frelaxedmax_N(± 0, ∓ 0)     = relaxed(R_fmax)[ fmax_N(± 0, ∓ 0), ± 0, ∓ 0, +0 ]
frelaxedmax_N(z1, z2)       = fmax_N(z1, z2)   (otherwise)
```

> **Note:** Relaxed maximum is implementation-dependent for NaNs and for zeroes with different signs. In the [deterministic profile](profile-deterministic), it behaves like regular `fmax`.

#### irelaxedq15mulrs_N(i1, i2)

The implementation-specific behaviour of this operation is determined by the global parameter `R_iq15mulr ∈ {0, 1}`.

* If both `i1` and `i2` equal `signed_N^{-1}(-2^{N-1})`, then return `relaxed(R_iq15mulr)[ 2^{N-1}-1, signed_N^{-1}(-2^{N-1}) ]`.
* Return `iq15mulrsats(i1, i2)`.

```text
irelaxedq15mulrs_N(signed_N^{-1}(-2^{N-1}), signed_N^{-1}(-2^{N-1})) = relaxed(R_iq15mulr)[ 2^{N-1}-1, signed_N^{-1}(-2^{N-1}) ]
irelaxedq15mulrs_N(i1, i2)                                            = iq15mulrsats(i1, i2)
```

> **Note:** Relaxed Q15 multiplication is implementation-dependent when the result overflows. In the [deterministic profile](profile-deterministic), it behaves like regular `iq15mulrsats`.

#### relaxedtruncu_{M,N}(z)

The implementation-specific behaviour of this operation is determined by the global parameter `R_trunc_u ∈ {0, 1}`.

* If `z` is normal or subnormal and `trunc(z)` is non-negative and less than `2^N`, then return `truncu_{M,N}(z)`.
* Else, return `relaxed(R_trunc_u)[ truncsatu_{M,N}(z), R ]`.

```text
relaxedtruncu_{M,N}(± q) = truncu_{M,N}(± q)                  (iff 0 ≤ trunc(± q) < 2^N)
relaxedtruncu_{M,N}(z)   = relaxed(R_trunc_u)[ truncsatu_{M,N}(z), R ]   (otherwise)
```

> **Note:** Relaxed unsigned truncation is non-deterministic for NaNs and out-of-range values. In the [deterministic profile](profile-deterministic), it behaves like regular `truncsatu`.

#### relaxedtruncs_{M,N}(z)

The implementation-specific behaviour of this operation is determined by the global parameter `R_trunc_s ∈ {0, 1}`.

* If `z` is normal or subnormal and `trunc(z)` is greater than or equal to `-2^{N-1}` and less than `2^{N-1}`, then return `truncs_{M,N}(z)`.
* Else, return `relaxed(R_trunc_s)[ truncsats_{M,N}(z), R ]`.

```text
relaxedtruncs_{M,N}(± q) = truncs_{M,N}(± q)                  (iff -2^{N-1} ≤ trunc(± q) < 2^{N-1})
relaxedtruncs_{M,N}(z)   = relaxed(R_trunc_s)[ truncsats_{M,N}(z), R ]   (otherwise)
```

> **Note:** Relaxed signed truncation is non-deterministic for NaNs and out-of-range values. In the [deterministic profile](profile-deterministic), it behaves like regular `truncsats`.

#### ivrelaxedswizzle(i^n, j^n)

The implementation-specific behaviour of this operation is determined by the global parameter `R_swizzle ∈ {0, 1}`.

* For each `j_k` in `j^n`, let `r_k` be the value `ivrelaxedswizzlelane(i^n, j_k)`.
* Let `r^n` be the concatenation of all `r_k`.
* Return `r^n`.

```text
ivrelaxedswizzle(i^n, j^n) = ivrelaxedswizzlelane(i^n, j)^n
```

where:

```text
ivrelaxedswizzlelane(i^n, j) = i[j]                       (iff j < 16)
ivrelaxedswizzlelane(i^n, j) = 0                           (iff signed_8(j) < 0)
ivrelaxedswizzlelane(i^n, j) = relaxed(R_swizzle)[ 0, i^n[j mod n] ]   (otherwise)
```

> **Note:** Relaxed swizzle is implementation-dependent if the signed interpretation of any of the 8-bit indices in `j^n` is larger than or equal to 16. In the [deterministic profile](profile-deterministic), it behaves like regular `ivswizzle`.

#### vrelaxeddot(i1, i2)

The implementation-specific behaviour of this operation is determined by the global parameter `R_idot ∈ {0, 1}`. It also affects the behaviour of `vrelaxeddotadd`.

Its definition is part of the definition of `vextbinop` specified [above](op-vextbinop).

> **Note:** Relaxed dot product is implementation-dependent when the second operand is negative in a signed intepretation. In the [deterministic profile](profile-deterministic), it behaves like signed dot product.

#### irelaxedlaneselect_N(i1, i2, i3)

The implementation-specific behaviour of this operation is determined by the global parameter `R_laneselect ∈ {0, 1}`.

* If `i3` is smaller than `2^{N-1}`, then let `i3'` be the value `0`, otherwise `2^N-1`.
* Let `i3''` be `relaxed(R_laneselect)[i3, i3']`.
* Return `ibitselect_N(i1, i2, i3'')`.

```text
irelaxedlaneselect_N(i1, i2, i3) = ibitselect_N(i1, i2, relaxed(R_laneselect)[ i3, extends_{1,N}(ishru_N(i3, N-1)) ])
```

> **Note:** Relaxed lane selection is non-deterministic when the mask mixes set and cleared bits, since the value of the high bit may or may not be expanded to all bits. In the [deterministic profile](profile-deterministic), it behaves like `ibitselect`.
