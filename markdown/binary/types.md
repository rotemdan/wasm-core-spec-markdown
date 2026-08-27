## Types

> **Note:** In some places, possible types include both type constructors or types denoted by type indices. Thus, the binary format for type constructors corresponds to the encodings of small negative `sN` values, such that they can unambiguously occur in the same place as (positive) type indices.

### Number Types

Number types are encoded by a single byte.

```text
numtype ::=
  | 0x7C    => f64
  | 0x7D    => f32
  | 0x7E    => i64
  | 0x7F    => i32
```

### Vector Types

Vector types are also encoded by a single byte.

```text
vectype ::=
  | 0x7B   => v128
```

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

### Reference Types

Reference types are either encoded by a single byte followed by a heap type, or, as a short form, directly as an abstract heap type.

```text
reftype ::=
  | 0x63 ht : heaptype    => ref null ht
  | 0x64 ht : heaptype    => ref ht
  | ht : absheaptype      => ref null ht
```

### Value Types

Value types are encoded with their respective encoding as a number type, vector type, or reference type.

```text
valtype ::=
  | nt : numtype    => nt
  | vt : vectype    => vt
  | rt : reftype    => rt
```

> **Note:** The value type `bot` (bottom) cannot occur in a module.
>
> Value types can occur in contexts where type indices are also allowed, such as in the case of block types. Thus, the binary format for types corresponds to the SignedLEB128 encoding of small negative `sN` values, so that they can coexist with (positive) type indices in the future.

### Result Types

Result types are encoded by the respective lists of value types.

```text
resulttype ::=
  | t* : list(valtype) => t*
```

### Composite Types

Composite types are encoded by a distinct byte followed by a type encoding of the respective form.

```text
mut ::=
  | 0x00 => ε
  | 0x01 => mut

comptype ::=
  | 0x5E ft : fieldtype                       => array ft
  | 0x5F ft* : list(fieldtype)                => struct ft*
  | 0x60 t1* : resulttype t2* : resulttype    => func t1* -> t2*

fieldtype ::=
  | zt : storagetype mut? : mut    => mut? zt

storagetype ::=
  | t : valtype      => t
  | pt : packtype    => pt

packtype ::=
  | 0x77    => i16
  | 0x78    => i8
```

### Recursive Types

Recursive types are encoded by the byte `0x4E` followed by a list of sub types.
Additional shorthands are recognized for unary recursions and sub types without super types.

```text
rectype ::=
  | 0x4E st* : list(subtype)    => rec st*
  | st : subtype                => rec st

subtype ::=
  | 0x4F x* : list(typeidx) ct : comptype    => sub final x* ct
  | 0x50 x* : list(typeidx) ct : comptype    => sub x* ct
  | ct : comptype                            => sub final ε ct
```

### Limits

Limits are encoded with a preceding flag indicating whether a maximum is present, and a flag for the address type.

```text
limits ::=
  | 0x00 n : u64            => (i32, [ n .. ε ])
  | 0x01 n : u64 m : u64    => (i32, [ n .. m ])
  | 0x04 n : u64            => (i64, [ n .. ε ])
  | 0x05 n : u64 m : u64    => (i64, [ n .. m ])
```

### Tag Types

Tag types are encoded by a type index denoting a function type.

```text
tagtype ::=
  | 0x00 x : typeidx    => x
```

> **Note:** In future versions of WebAssembly, the preceding zero byte may encode additional attributes.

### Global Types

Global types are encoded by their value type and a flag for their mutability.

```text
globaltype ::=
  | t : valtype mut? : mut    => mut? t
```

### Memory Types

Memory types are encoded with their limits.

```text
memtype ::=
  | (at, lim) : limits    => at lim page
```

### Table Types

Table types are encoded with their limits and the encoding of their element reference type.

```text
tabletype ::=
  | rt : reftype (at, lim) : limits    => at lim rt
```

### External Types

External types are encoded by a distinguishing byte followed by an encoding of the respective form of type.

```text
externtype ::=
  | 0x00 x : typeidx        => func x
  | 0x01 tt : tabletype     => table tt
  | 0x02 mt : memtype       => mem mt
  | 0x03 gt : globaltype    => global gt
  | 0x04 jt : tagtype       => tag jt
```
