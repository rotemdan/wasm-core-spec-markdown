## Types

Simple types, such as number types are universally valid.

However, restrictions apply to most other types, such as reference types, function types, as well as the limits of table types and memory types, which must be checked during validation.

Moreover, block types are converted to instruction types for ease of processing.

### Number Types

The number type `numtype` is always valid.

```text
C |- numtype : OK
```

### Vector Types

The vector type `vectype` is always valid.

```text
C |- vectype : OK
```

### Type Uses

The type use `typeidx` is valid if:

* The type `C.TYPES[typeidx]` exists.

```text
C.TYPES[typeidx] = dt
──────────────────────
C |- typeidx : OK
```

### Heap Types

The heap type `absheaptype` is always valid.

```text
C |- absheaptype : OK
```

### Reference Types

The reference type `(ref null? heaptype)` is valid if:

* The heap type `heaptype` is valid.

```text
C |- heaptype : OK
──────────────────────
C |- ref null? heaptype : OK
```

### Value Types

The value type `valtype` is valid if:

* Either:
   * The value type `valtype` is of the form `numtype`.
   * The number type `numtype` is valid.
* Or:
   * The value type `valtype` is of the form `vectype`.
   * The vector type `vectype` is valid.
* Or:
   * The value type `valtype` is of the form `reftype`.
   * The reference type `reftype` is valid.
* Or:
   * The value type `valtype` is of the form `bot`.

```text
C |- bot : OK
```

### Result Types

The result type `t*` is valid if:

* For all `t` in `t*`:
   * The value type `t` is valid.

```text
(C |- t : OK)*
────────────────────────
C |- t* : OK
```

### Block Types

Block types may be expressed in one of two forms, both of which are converted to instruction types by the following rules.

The block type `typeidx` is valid as the instruction type `t1* -> t2*` if:

* The type `C.TYPES[typeidx]` exists.
* The expansion of `C.TYPES[typeidx]` is `(func t1* -> t2*)`.

```text
C.TYPES[typeidx] ≈ func t1* -> t2*
──────────────────────────────────
C |- typeidx : t1* -> t2*
```

The block type `valtype?` is valid as the instruction type `ε -> valtype?` if:

* If `valtype` is defined, then:
   * The value type `valtype` is valid.

```text
(C |- valtype : OK)?
──────────────────────────────
C |- valtype? : ε -> valtype?
```

### Instruction Types

The instruction type `t1* ->_{x*} t2*` is valid if:

* The result type `t1*` is valid.
* The result type `t2*` is valid.
* For all `x` in `x*`:
   * The local `C.LOCALS[x]` exists.

```text
C |- t1* : OK
C |- t2* : OK
(C.LOCALS[x] = lt)*
──────────────────────
C |- t1* ->_{x*} t2* : OK
```

### Composite Types

The composite type `(struct fieldtype*)` is valid if:

* For all `fieldtype` in `fieldtype*`:
   * The field type `fieldtype` is valid.

```text
(C |- fieldtype : OK)*
──────────────────────────────────
C |- struct fieldtype* : OK
```

The composite type `(array fieldtype)` is valid if:

* The field type `fieldtype` is valid.

```text
C |- fieldtype : OK
──────────────────────────────
C |- array fieldtype : OK
```

The composite type `(func t1* -> t2*)` is valid if:

* The result type `t1*` is valid.
* The result type `t2*` is valid.

```text
C |- t1* : OK
C |- t2* : OK
──────────────────
C |- func t1* -> t2* : OK
```

The field type `(mut? storagetype)` is valid if:

* The storage type `storagetype` is valid.

```text
C |- storagetype : OK
────────────────────────────────────
C |- mut? storagetype : OK
```

The packed type `packtype` is always valid.

```text
C |- packtype : OK
```

### Recursive Types

Recursive types are validated with respect to the first type index defined by the recursive group.

The recursive type `(rec subtype*)` is valid for the type index `x` if:

* Either:
   * The sub type sequence `subtype*` is empty.
* Or:
   * The sub type sequence `subtype*` is of the form `subtype1 subtype'*`.
   * The sub type `subtype1` is valid for the type index `x`.
   * The recursive type `(rec subtype'*)` is valid for the type index `x + 1`.

```text
────────────────────
C |- rec ε : OK(x)

C |- subtype1 : OK(x)
C |- rec subtype* : OK(x + 1)
──────────────────────────────────────
C |- rec (subtype1 subtype*) : OK(x)
```

The sub type `(sub final? x* comptype)` is valid for the type index `x0` if:

* The length of `x*` is less than or equal to `1`.
* For all `x` in `x*`:
   * The index `x` is less than `x0`.
   * The type `C.TYPES[x]` exists.
   * The sub type `unrolldt(C.TYPES[x])` is of the form `(sub y* comptype')`.
* `comptype'*` is the concatenation of all such `comptype'`.
* The composite type `comptype` is valid.
* For all `comptype'` in `comptype'*`:
   * The composite type `comptype` matches the composite type `comptype'`.

```text
|x*| <= 1
(x < x0)*
(unrolldt(C.TYPES[x]) = sub y* comptype')*
C |- comptype : OK
(C |- comptype <: comptype')*
──────────────────────────────────
C |- sub final? x* comptype : OK(x0)
```

> **Note:** The side condition on the index ensures that a declared supertype is a previously defined type, preventing cyclic subtype hierarchies.
>
> Future versions of WebAssembly may allow more than one supertype.

### Limits

Limits must have meaningful bounds that are within a given range.

The limits range `[ n .. m? ]` is valid within `k` if:

* `n` is less than or equal to `k`.
* If `m` is defined, then:
   * `n` is less than or equal to `m`.
   * `m` is less than or equal to `k`.

```text
n <= k
(n <= m <= k)?
──────────────────
C |- limits [ n .. m? ] : k
```

### Tag Types

The tag type `typeuse` is valid if:

* The type use `typeuse` is valid.
* The expansion of `C` is `(func t1* ->)`.

```text
C |- typeuse : OK
typeuse ≈_C func t1* -> []
──────────────────────────────
C |- typeuse : OK
```

### Global Types

The global type `(mut? t)` is valid if:

* The value type `t` is valid.

```text
C |- t : OK
──────────────────────
C |- mut? t : OK
```

### Memory Types

The memory type `(addrtype limits page)` is valid if:

* The limits range `limits` is valid within `2^(|addrtype| - 16)`.

```text
C |- limits : 2^(|addrtype| - 16)
────────────────────────────────────────────
C |- addrtype limits page : OK
```

### Table Types

The table type `(addrtype limits reftype)` is valid if:

* The limits range `limits` is valid within `2^(|addrtype|) - 1`.
* The reference type `reftype` is valid.

```text
C |- limits : 2^(|addrtype|) - 1
C |- reftype : OK
──────────────────────────────
C |- addrtype limits reftype : OK
```

### External Types

The external type `(tag tagtype)` is valid if:

* The tag type `tagtype` is valid.

```text
C |- tagtype : OK
──────────────────────────
C |- tag tagtype : OK
```

The external type `(global globaltype)` is valid if:

* The global type `globaltype` is valid.

```text
C |- globaltype : OK
─────────────────────────────────
C |- global globaltype : OK
```

The external type `(mem memtype)` is valid if:

* The memory type `memtype` is valid.

```text
C |- memtype : OK
──────────────────────────
C |- mem memtype : OK
```

The external type `(table tabletype)` is valid if:

* The table type `tabletype` is valid.

```text
C |- tabletype : OK
───────────────────────────────
C |- table tabletype : OK
```

The external type `(func typeuse)` is valid if:

* The type use `typeuse` is valid.
* The expansion of `C` is `(func t1* -> t2*)`.

```text
C |- typeuse : OK
typeuse ≈_C func t1* -> t2*
──────────────────────────────
C |- func typeuse : OK
```
