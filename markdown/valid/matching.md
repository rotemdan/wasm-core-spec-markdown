## Matching

On most types, a notion of *subtyping* is defined that is applicable in validation rules, during module instantiation when checking the types of imports, or during execution, when performing casts.

### Number Types

The number type `numtype` matches only itself.

```text
C |- numtype <: numtype
```

### Vector Types

The vector type `vectype` matches only itself.

```text
C |- vectype <: vectype
```

### Heap Types

The heap type `heaptype1` matches the heap type `heaptype2` if:

* Either:
   * The heap type `heaptype2` is of the form `heaptype1`.
* Or:
   * The heap type `heaptype'` is valid.
   * The heap type `heaptype1` matches the heap type `heaptype'`.
   * The heap type `heaptype'` matches the heap type `heaptype2`.
* Or:
   * The heap type `heaptype1` is of the form `eq`.
   * The heap type `heaptype2` is of the form `any`.
* Or:
   * The heap type `heaptype1` is of the form `i31`.
   * The heap type `heaptype2` is of the form `eq`.
* Or:
   * The heap type `heaptype1` is of the form `struct`.
   * The heap type `heaptype2` is of the form `eq`.
* Or:
   * The heap type `heaptype1` is of the form `array`.
   * The heap type `heaptype2` is of the form `eq`.
* Or:
   * The heap type `heaptype1` is of the form `deftype`.
   * The heap type `heaptype2` is of the form `struct`.
   * The expansion of `deftype` is `(struct fieldtype*)`.
* Or:
   * The heap type `heaptype1` is of the form `deftype`.
   * The heap type `heaptype2` is of the form `array`.
   * The expansion of `deftype` is `(array fieldtype)`.
* Or:
   * The heap type `heaptype1` is of the form `deftype`.
   * The heap type `heaptype2` is of the form `func`.
   * The expansion of `deftype` is `(func t1* -> t2*)`.
* Or:
   * The heap type `heaptype1` is of the form `deftype1`.
   * The heap type `heaptype2` is of the form `deftype2`.
   * The defined type `deftype1` matches the defined type `deftype2`.
* Or:
   * The heap type `heaptype1` is of the form `typeidx`.
   * The type `C.TYPES[typeidx]` exists.
   * The type `C.TYPES[typeidx]` matches the heap type `heaptype2`.
* Or:
   * The heap type `heaptype2` is of the form `typeidx`.
   * The type `C.TYPES[typeidx]` exists.
   * The heap type `heaptype1` matches the type `C.TYPES[typeidx]`.
* Or:
   * The heap type `heaptype1` is of the form `(rec . i)`.
   * The heap type `heaptype2` is of the form `struct`.
   * The recursive type `C.RECS[i]` exists.
   * The recursive type `C.RECS[i]` is of the form `(sub final? (struct fieldtype*))`.
* Or:
   * The heap type `heaptype1` is of the form `(rec . i)`.
   * The heap type `heaptype2` is of the form `array`.
   * The recursive type `C.RECS[i]` exists.
   * The recursive type `C.RECS[i]` is of the form `(sub final? (array fieldtype))`.
* Or:
   * The heap type `heaptype1` is of the form `(rec . i)`.
   * The heap type `heaptype2` is of the form `func`.
   * The recursive type `C.RECS[i]` exists.
   * The recursive type `C.RECS[i]` is of the form `(sub final? (func t1* -> t2*))`.
* Or:
   * The heap type `heaptype1` is of the form `(rec . i)`.
   * The length of `typeuse*` is greater than `j`.
   * The heap type `heaptype2` is of the form `typeuse*[j]`.
   * The recursive type `C.RECS[i]` exists.
   * The recursive type `C.RECS[i]` is of the form `(sub final? typeuse* ct)`.
* Or:
   * The heap type `heaptype1` is of the form `none`.
   * The heap type `heaptype2` matches the heap type `any`.
   * The heap type `heaptype2` is not of the form `bot`.
* Or:
   * The heap type `heaptype1` is of the form `nofunc`.
   * The heap type `heaptype2` matches the heap type `func`.
   * The heap type `heaptype2` is not of the form `bot`.
* Or:
   * The heap type `heaptype1` is of the form `noexn`.
   * The heap type `heaptype2` matches the heap type `exn`.
   * The heap type `heaptype2` is not of the form `bot`.
* Or:
   * The heap type `heaptype1` is of the form `noextern`.
   * The heap type `heaptype2` matches the heap type `extern`.
   * The heap type `heaptype2` is not of the form `bot`.
* Or:
   * The heap type `heaptype1` is of the form `bot`.

```text
C |- heaptype <: heaptype
```

```text
C |- heaptype' : OK
C |- heaptype1 <: heaptype'
C |- heaptype' <: heaptype2
────────────────────────────
C |- heaptype1 <: heaptype2
```

```text
C |- eq <: any
```

```text
C |- i31 <: eq
```

```text
C |- struct <: eq
```

```text
C |- array <: eq
```

```text
deftype ≈ struct fieldtype*
────────────────────────────
C |- deftype <: struct
```

```text
deftype ≈ array fieldtype
──────────────────────────
C |- deftype <: array
```

```text
deftype ≈ func t1* -> t2*
──────────────────────────
C |- deftype <: func
```

```text
C |- C.TYPES[typeidx] <: heaptype
───────────────────────────────────
C |- typeidx <: heaptype
```

```text
C |- heaptype <: C.TYPES[typeidx]
───────────────────────────────────
C |- heaptype <: typeidx
```

```text
C |- heaptype <: any
heaptype ≠ bot
────────────────────
C |- none <: heaptype
```

```text
C |- heaptype <: func
heaptype ≠ bot
────────────────────
C |- nofunc <: heaptype
```

```text
C |- heaptype <: exn
heaptype ≠ bot
────────────────────
C |- noexn <: heaptype
```

```text
C |- heaptype <: extern
heaptype ≠ bot
──────────────────────
C |- noextern <: heaptype
```

```text
C |- bot <: heaptype
```

### Reference Types

The reference type `(ref null? ht1)` matches the reference type `(ref null? ht2)` if:

* The heap type `ht1` matches the heap type `ht2`.
* Either:
   * The optional nullability of the first reference type is absent.
   * The optional nullability of the second reference type is absent.
* Or:
   * The optional nullability of the first reference type is of the form `null?`.
   * The optional nullability of the second reference type is of the form `null`.

```text
C |- ht1 <: ht2
──────────────────
C |- ref ht1 <: ref ht2
```

```text
C |- ht1 <: ht2
──────────────────────
C |- ref null? ht1 <: ref null ht2
```

### Value Types

The value type `valtype1` matches the value type `valtype2` if:

* Either:
   * The value type `valtype1` is of the form `numtype1`.
   * The value type `valtype2` is of the form `numtype2`.
   * The number type `numtype1` matches the number type `numtype2`.
* Or:
   * The value type `valtype1` is of the form `vectype1`.
   * The value type `valtype2` is of the form `vectype2`.
   * The vector type `vectype1` matches the vector type `vectype2`.
* Or:
   * The value type `valtype1` is of the form `reftype1`.
   * The value type `valtype2` is of the form `reftype2`.
   * The reference type `reftype1` matches the reference type `reftype2`.
* Or:
   * The value type `valtype1` is of the form `bot`.

```text
C |- bot <: valtype
```

### Result Types

Subtyping is lifted to result types in a pointwise manner.

The result type `t1*` matches the result type `t2*` if:

* For all `t1` in `t1*`, and corresponding `t2` in `t2*`:
   * The value type `t1` matches the value type `t2`.

```text
(C |- t1 <: t2)*
──────────────────────────────────
C |- t1* <: t2*
```

### Instruction Types

Subtyping is further lifted to instruction types.

The instruction type `t11* ->_{x1*} t12*` matches the instruction type `t21* ->_{x2*} t22*` if:

* The result type `t21*` matches the result type `t11*`.
* The result type `t12*` matches the result type `t22*`.
* The local index sequence `x*` is of the form `x2* ∖ x1*`.
* For all `x` in `x*`:
   * The local `C.LOCALS[x]` exists.
   * The local `C.LOCALS[x]` is of the form `(set t)`.

```text
C |- t21* <: t11*
C |- t12* <: t22*
x* = x2* ∖ x1*
(C.LOCALS[x] = set t)*
──────────────────────────────────────────────────────
C |- t11* ->_{x1*} t12* <: t21* ->_{x2*} t22*
```

> **Note:** Instruction types are contravariant in their input and covariant in their output. Moreover, the supertype may ignore variables from the init set `x1*`. It may also *add* variables to the init set, provided these are already set in the context, i.e., are vacuously initialized.

### Composite Types

The composite type `comptype1` matches the composite type `comptype2` if:

* Either:
   * The composite type `comptype1` is of the form `(struct ft1* ft'1*)`.
   * The composite type `comptype2` is of the form `(struct ft2*)`.
   * For all `ft1` in `ft1*`, and corresponding `ft2` in `ft2*`:
      * The field type `ft1` matches the field type `ft2`.
* Or:
   * The composite type `comptype1` is of the form `(array ft1)`.
   * The composite type `comptype2` is of the form `(array ft2)`.
   * The field type `ft1` matches the field type `ft2`.
* Or:
   * The composite type `comptype1` is of the form `(func t11* -> t12*)`.
   * The composite type `comptype2` is of the form `(func t21* -> t22*)`.
   * The result type `t21*` matches the result type `t11*`.
   * The result type `t12*` matches the result type `t22*`.

```text
(C |- ft1 <: ft2)*
────────────────────────────────────────────
C |- struct (ft1* ft'1*) <: struct ft2*
```

```text
C |- ft1 <: ft2
──────────────────────────────────────
C |- array ft1 <: array ft2
```

```text
C |- t21* <: t11*
C |- t12* <: t22*
──────────────────────────────────
C |- func t11* -> t12* <: func t21* -> t22*
```

### Field Types

The field type `(mut? zt1)` matches the field type `(mut? zt2)` if:

* The storage type `zt1` matches the storage type `zt2`.
* Either:
   * The optional mutability of the first field type is absent.
   * The optional mutability of the second field type is absent.
* Or:
   * The optional mutability of the first field type is of the form `mut`.
   * The optional mutability of the second field type is of the form `mut`.
   * The storage type `zt2` matches the storage type `zt1`.

```text
C |- zt1 <: zt2
──────────────────
C |- zt1 <: zt2

C |- zt1 <: zt2
C |- zt2 <: zt1
──────────────────────
C |- mut zt1 <: mut zt2
```

The storage type `storagetype1` matches the storage type `storagetype2` if:

* Either:
   * The storage type `storagetype1` is of the form `valtype1`.
   * The storage type `storagetype2` is of the form `valtype2`.
   * The value type `valtype1` matches the value type `valtype2`.
* Or:
   * The storage type `storagetype1` is of the form `packtype1`.
   * The storage type `storagetype2` is of the form `packtype2`.
   * The packed type `packtype1` matches the packed type `packtype2`.

The packed type `packtype` matches only itself.

```text
C |- packtype <: packtype
```

### Defined Types

The defined type `deftype1` matches the defined type `deftype2` if:

* Either:
   * The defined type `clostype_C(deftype1)` is of the form `clostype_C(deftype2)`.
* Or:
   * The sub type `unrolldt(deftype1)` is of the form `(sub final? typeuse* ct)`.
   * The length of `typeuse*` is greater than `i`.
   * The type use `typeuse*[i]` matches the heap type `deftype2`.

```text
clostype_C(deftype1) = clostype_C(deftype2)
────────────────────────────────────────────
C |- deftype1 <: deftype2
```

```text
unrolldt(deftype1) = sub final? typeuse* ct
C |- typeuse*[i] <: deftype2
──────────────────────────────────────────
C |- deftype1 <: deftype2
```

> **Note:** Note that there is no explicit definition of type *equivalence*, since it coincides with syntactic equality, as used in the premise of the former rule above.

### Limits

The limits range `[ n1 .. u64_1? ]` matches the limits range `[ n2 .. u64_2? ]` if:

* `n1` is greater than or equal to `n2`.
* Either:
   * `u64_1?` is of the form `m1`.
   * If `u64_2` is defined, then:
      * `m1` is less than or equal to `u64_2`.
* Or:
   * `u64_1?` is absent.
   * `u64_2?` is absent.

```text
n1 >= n2
(m1 <= m2)?
──────────────────────────────
C |- [ n1 .. m1 ] <: [ n2 .. m2? ]
```

```text
n1 >= n2
──────────────────
C |- [ n1 .. ε ] <: [ n2 .. ε ]
```

### Tag Types

The tag type `deftype1` matches the tag type `deftype2` if:

* The defined type `deftype1` matches the defined type `deftype2`.
* The defined type `deftype2` matches the defined type `deftype1`.

```text
C |- deftype1 <: deftype2
C |- deftype2 <: deftype1
──────────────────────────────
C |- deftype1 <: deftype2
```

> **Note:** Although the conclusion of this rule looks identical to its premise, they in fact describe different relations: the premise invokes subtyping on defined types, while the conclusion defines it on tag types that happen to be expressed as defined types.

### Global Types

The global type `(mut? valtype1)` matches the global type `(mut? valtype2)` if:

* The value type `valtype1` matches the value type `valtype2`.
* Either:
   * The optional mutability of the first global type is absent.
   * The optional mutability of the second global type is absent.
* Or:
   * The optional mutability of the first global type is of the form `mut`.
   * The optional mutability of the second global type is of the form `mut`.
   * The value type `valtype2` matches the value type `valtype1`.

```text
C |- valtype1 <: valtype2
──────────────────────────
C |- valtype1 <: valtype2
```

```text
C |- valtype1 <: valtype2
C |- valtype2 <: valtype1
──────────────────────────────
C |- mut valtype1 <: mut valtype2
```

### Memory Types

The memory type `(addrtype limits1 page)` matches the memory type `(addrtype limits2 page)` if:

* The limits range `limits1` matches the limits range `limits2`.

```text
C |- limits1 <: limits2
─────────────────────────
C |- addrtype limits1 page <: addrtype limits2 page
```

### Table Types

The table type `(addrtype limits1 reftype1)` matches the table type `(addrtype limits2 reftype2)` if:

* The limits range `limits1` matches the limits range `limits2`.
* The reference type `reftype1` matches the reference type `reftype2`.
* The reference type `reftype2` matches the reference type `reftype1`.

```text
C |- limits1 <: limits2
C |- reftype1 <: reftype2
C |- reftype2 <: reftype1
──────────────────────────────
C |- addrtype limits1 reftype1 <: addrtype limits2 reftype2
```

### External Types

The external type `(tag tagtype1)` matches the external type `(tag tagtype2)` if:

* The tag type `tagtype1` matches the tag type `tagtype2`.

```text
C |- tagtype1 <: tagtype2
──────────────────────────
C |- tag tagtype1 <: tag tagtype2
```

The external type `(global globaltype1)` matches the external type `(global globaltype2)` if:

* The global type `globaltype1` matches the global type `globaltype2`.

```text
C |- globaltype1 <: globaltype2
─────────────────────────────────
C |- global globaltype1 <: global globaltype2
```

The external type `(mem memtype1)` matches the external type `(mem memtype2)` if:

* The memory type `memtype1` matches the memory type `memtype2`.

```text
C |- memtype1 <: memtype2
──────────────────────────
C |- mem memtype1 <: mem memtype2
```

The external type `(table tabletype1)` matches the external type `(table tabletype2)` if:

* The table type `tabletype1` matches the table type `tabletype2`.

```text
C |- tabletype1 <: tabletype2
───────────────────────────────
C |- table tabletype1 <: table tabletype2
```

The external type `(func deftype1)` matches the external type `(func deftype2)` if:

* The defined type `deftype1` matches the defined type `deftype2`.

```text
C |- deftype1 <: deftype2
──────────────────────────
C |- func deftype1 <: func deftype2
```
