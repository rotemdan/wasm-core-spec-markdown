## Instructions

Instructions are syntactically distinguished into *plain* and *structured* instructions.

```text
instr_I ::=
  | in : plaininstr_I  => in
  | in : blockinstr_I  => in

instrs_I ::=
  | in* : instr_I*  => in*
```

In addition, as a syntactic abbreviation, instructions can be written as S-expressions in folded form, to group them visually.

### Labels

Structured control instructions can be annotated with a symbolic label identifier.

They are the only symbolic identifiers that can be bound locally in an instruction sequence.

The following grammar handles the corresponding update to the identifier context by composing the context with an additional label entry.

```text
label_I ::=
  | ε  => (ε, { LABELS ε } ⊕ I)
  | id : id  => (id, { LABELS id } ⊕ I)    (if id ∉ I.LABELS)
  | id : id  => (id, { LABELS id } ⊕ I[.LABELS[x] = ε])    (if id = I.LABELS[x])
```

> **Note:** The new label entry is inserted at the *beginning* of the label list in the identifier context. This effectively shifts all existing labels up by one, mirroring the fact that control instructions are indexed relatively not absolutely.
>
> If a label with the same name already exists, then it is shadowed and the earlier label becomes inaccessible.

### Parametric Instructions

```text
plaininstr_I ::=
  | 'unreachable'  => unreachable
  | 'nop'  => nop
  | 'drop'  => drop
  | 'select' (t* : result_I*)^?  => select (t*)^?
```

### Control Instructions

Structured control instructions can bind an optional symbolic label identifier.

The same label identifier may optionally be repeated after the corresponding `end` or `else` keywords, to indicate the matching delimiters.

Their block type is given as a type use, analogous to the type of functions.

However, the special case of a type use that is syntactically empty or consists of only a single result is not regarded as an abbreviation for an inline function type, but is parsed directly into an optional value type.

```text
blocktype_I ::=
  | t? : result_I?  => t?
  | (x, I') : typeuse_I  => x    (if I' = { LOCALS (ε)* })

blockinstr_I ::=
  | 'block' (id?, I') : label_I bt : blocktype_I in* : instrs_I' 'end' id'? : id?
        => block bt in*    (if id'? = ε || id'? = id?)
  | 'loop' (id?, I') : label_I bt : blocktype_I in* : instrs_I' 'end' id'? : id?
        => loop bt in*    (if id'? = ε || id'? = id?)
  | 'if' (id?, I') : label_I bt : blocktype_I in1* : instrs_I' 'else' id1? : id? in2* : instrs_I' 'end' id2? : id?
        => if bt in1* else in2*    (if (id1? = ε || id1? = id?) && (id2? = ε || id2? = id?))
  | 'try_table' (id?, I') : label_I bt : blocktype_I c* : catch_I* in* : instrs_I' 'end' id'? : id?
        => try_table bt c* in*    (if id'? = ε || id'? = id?)
  | 'catch' x : tagidx_I l : labelidx_I  => catch x l
  | 'catch_ref' x : tagidx_I l : labelidx_I  => catch_ref x l
  | 'catch_all' l : labelidx_I  => catch_all l
  | 'catch_all_ref' l : labelidx_I  => catch_all_ref l
```

> **Note:** The side condition stating that the identifier context `I'` must only contain unnamed entries in the rule for `typeuse` block types enforces that no identifier can be bound in any `param` declaration for a block type.

All other control instruction are represented verbatim.

> **Note:** The side condition stating that the identifier context `I'` must only contain unnamed entries in the rule for CALLINDIRECT enforces that no identifier can be bound in any `param` declaration appearing in the type annotation.

#### Abbreviations

The `else` keyword of an `if` instruction can be omitted if the following instruction sequence is empty.

```text
blockinstr_I ::=
  | ...
  | 'if' label_I blocktype_I instrs_I 'end' id?  ≡  'if' label_I blocktype_I instrs_I 'else' 'end' id?
```

Also, for backwards compatibility, the table index to `call_indirect` and `return_call_indirect` can be omitted, defaulting to `0`.

```text
plaininstr_I ::=
  | ...
  | 'call_indirect' typeuse_I  ≡  'call_indirect' '0' typeuse_I
  | 'return_call_indirect' typeuse_I  ≡  'return_call_indirect' '0' typeuse_I
```

### Variable Instructions

```text
plaininstr_I ::=
  | ...
  | 'local.get' x : localidx_I    => local.get x
  | 'local.set' x : localidx_I    => local.set x
  | 'local.tee' x : localidx_I    => local.tee x
  | 'global.get' x : globalidx_I  => global.get x
  | 'global.set' x : globalidx_I  => global.set x
```

### Table Instructions

```text
plaininstr_I ::=
  | ...
  | 'table.get' x : tableidx_I   => table.get x
  | 'table.set' x : tableidx_I   => table.set x
  | 'table.size' x : tableidx_I  => table.size x
  | 'table.grow' x : tableidx_I  => table.grow x
  | 'table.fill' x : tableidx_I  => table.fill x
  | 'table.copy' x1 : tableidx_I x2 : tableidx_I  => table.copy x1 x2
  | 'table.init' x : tableidx_I y : elemidx_I     => table.init x y
  | 'elem.drop' x : elemidx_I    => elem.drop x
```

#### Abbreviations

For backwards compatibility, all table indices may be omitted from table instructions, defaulting to `0`.

```text
plaininstr_I ::=
  | ...
  | 'table.get'  ≡  'table.get' '0'
  | 'table.set'  ≡  'table.set' '0'
  | 'table.size'  ≡  'table.size' '0'
  | 'table.grow'  ≡  'table.grow' '0'
  | 'table.fill'  ≡  'table.fill' '0'
  | 'table.copy'  ≡  'table.copy' '0' '0'
  | 'table.init' elemidx_I  ≡  'table.init' '0' elemidx_I
```

### Memory Instructions

The offset and alignment immediates to memory instructions are optional.
The offset defaults to `0`, the alignment to the storage size of the respective memory access, which is its *natural alignment*.
Lexically, an `offset` or `align` phrase is considered a single keyword token, so no white space is allowed around the `=`.

```text
memarg_N ::=
  | m : offset n : align_N  => { align n, offset m }

offset ::=
  | 'offset=' m : u64  => m
  | ε  => 0

align_N ::=
  | 'align=' m : u64  => n    (if m = 2^n)
  | ε  => N

laneidx ::=
  | i : u8  => i
```

```text
plaininstr_I ::=
  | ...
  | 'i32.load' x : memidx_I ao : memarg_4  => i32.load x ao
  | 'i64.load' x : memidx_I ao : memarg_8  => i64.load x ao
  | 'f32.load' x : memidx_I ao : memarg_4  => f32.load x ao
  | 'f64.load' x : memidx_I ao : memarg_8  => f64.load x ao
  | 'i32.load8_s' x : memidx_I ao : memarg_1  => i32.load8_s x ao
  | 'i32.load8_u' x : memidx_I ao : memarg_1  => i32.load8_u x ao
  | 'i32.load16_s' x : memidx_I ao : memarg_2  => i32.load16_s x ao
  | 'i32.load16_u' x : memidx_I ao : memarg_2  => i32.load16_u x ao
  | 'i64.load8_s' x : memidx_I ao : memarg_1  => i64.load8_s x ao
  | 'i64.load8_u' x : memidx_I ao : memarg_1  => i64.load8_u x ao
  | 'i64.load16_s' x : memidx_I ao : memarg_2  => i64.load16_s x ao
  | 'i64.load16_u' x : memidx_I ao : memarg_2  => i64.load16_u x ao
  | 'i64.load32_s' x : memidx_I ao : memarg_4  => i64.load32_s x ao
  | 'i64.load32_u' x : memidx_I ao : memarg_4  => i64.load32_u x ao
  | 'v128.load' x : memidx_I ao : memarg_16  => v128.load x ao
  | 'v128.load8x8_s' x : memidx_I ao : memarg_8  => v128.load8x8_s x ao
  | 'v128.load8x8_u' x : memidx_I ao : memarg_8  => v128.load8x8_u x ao
  | 'v128.load16x4_s' x : memidx_I ao : memarg_8  => v128.load16x4_s x ao
  | 'v128.load16x4_u' x : memidx_I ao : memarg_8  => v128.load16x4_u x ao
  | 'v128.load32x2_s' x : memidx_I ao : memarg_8  => v128.load32x2_s x ao
  | 'v128.load32x2_u' x : memidx_I ao : memarg_8  => v128.load32x2_u x ao
  | 'v128.load8_splat' x : memidx_I ao : memarg_1  => v128.load8_splat x ao
  | 'v128.load16_splat' x : memidx_I ao : memarg_2  => v128.load16_splat x ao
  | 'v128.load32_splat' x : memidx_I ao : memarg_4  => v128.load32_splat x ao
  | 'v128.load64_splat' x : memidx_I ao : memarg_8  => v128.load64_splat x ao
  | 'v128.load32_zero' x : memidx_I ao : memarg_4  => v128.load32_zero x ao
  | 'v128.load64_zero' x : memidx_I ao : memarg_8  => v128.load64_zero x ao
  | 'v128.load8_lane' x : memidx_I ao : memarg_1 i : laneidx  => v128.load8_lane x ao i
  | 'v128.load16_lane' x : memidx_I ao : memarg_2 i : laneidx  => v128.load16_lane x ao i
  | 'v128.load32_lane' x : memidx_I ao : memarg_4 i : laneidx  => v128.load32_lane x ao i
  | 'v128.load64_lane' x : memidx_I ao : memarg_8 i : laneidx  => v128.load64_lane x ao i
  | 'i32.store' x : memidx_I ao : memarg_4  => i32.store x ao
  | 'i64.store' x : memidx_I ao : memarg_8  => i64.store x ao
  | 'f32.store' x : memidx_I ao : memarg_4  => f32.store x ao
  | 'f64.store' x : memidx_I ao : memarg_8  => f64.store x ao
  | 'i32.store8' x : memidx_I ao : memarg_1  => i32.store8 x ao
  | 'i32.store16' x : memidx_I ao : memarg_2  => i32.store16 x ao
  | 'i64.store8' x : memidx_I ao : memarg_1  => i64.store8 x ao
  | 'i64.store16' x : memidx_I ao : memarg_2  => i64.store16 x ao
  | 'i64.store32' x : memidx_I ao : memarg_4  => i64.store32 x ao
  | 'v128.store' x : memidx_I ao : memarg_16  => v128.store x ao
  | 'v128.store8_lane' x : memidx_I ao : memarg_1 i : laneidx  => v128.store8_lane x ao i
  | 'v128.store16_lane' x : memidx_I ao : memarg_2 i : laneidx  => v128.store16_lane x ao i
  | 'v128.store32_lane' x : memidx_I ao : memarg_4 i : laneidx  => v128.store32_lane x ao i
  | 'v128.store64_lane' x : memidx_I ao : memarg_8 i : laneidx  => v128.store64_lane x ao i
  | 'memory.size' x : memidx_I  => memory.size x
  | 'memory.grow' x : memidx_I  => memory.grow x
  | 'memory.fill' x : memidx_I  => memory.fill x
  | 'memory.copy' x1 : memidx_I x2 : memidx_I  => memory.copy x1 x2
  | 'memory.init' x : memidx_I y : dataidx_I  => memory.init x y
  | 'data.drop' x : dataidx_I  => data.drop x
```

#### Abbreviations

As an abbreviation, the memory index can be omitted in all memory instructions, defaulting to `0`.

```text
plaininstr_I ::=
  | ...
  | 'i32.load' memarg_4  ≡  'i32.load' '0' memarg_4
  | 'i64.load' memarg_8  ≡  'i64.load' '0' memarg_8
  | 'f32.load' memarg_4  ≡  'f32.load' '0' memarg_4
  | 'f64.load' memarg_8  ≡  'f64.load' '0' memarg_8
  | 'i32.load8_s' memarg_1  ≡  'i32.load8_s' '0' memarg_1
  | 'i32.load8_u' memarg_1  ≡  'i32.load8_u' '0' memarg_1
  | 'i32.load16_s' memarg_2  ≡  'i32.load16_s' '0' memarg_2
  | 'i32.load16_u' memarg_2  ≡  'i32.load16_u' '0' memarg_2
  | 'i64.load8_s' memarg_1  ≡  'i64.load8_s' '0' memarg_1
  | 'i64.load8_u' memarg_1  ≡  'i64.load8_u' '0' memarg_1
  | 'i64.load16_s' memarg_2  ≡  'i64.load16_s' '0' memarg_2
  | 'i64.load16_u' memarg_2  ≡  'i64.load16_u' '0' memarg_2
  | 'i64.load32_s' memarg_4  ≡  'i64.load32_s' '0' memarg_4
  | 'i64.load32_u' memarg_4  ≡  'i64.load32_u' '0' memarg_4
  | 'v128.load' memarg_16  ≡  'v128.load' '0' memarg_16
  | 'v128.load8x8_s' memarg_8  ≡  'v128.load8x8_s' '0' memarg_8
  | 'v128.load8x8_u' memarg_8  ≡  'v128.load8x8_u' '0' memarg_8
  | 'v128.load16x4_s' memarg_8  ≡  'v128.load16x4_s' '0' memarg_8
  | 'v128.load16x4_u' memarg_8  ≡  'v128.load16x4_u' '0' memarg_8
  | 'v128.load32x2_s' memarg_8  ≡  'v128.load32x2_s' '0' memarg_8
  | 'v128.load32x2_u' memarg_8  ≡  'v128.load32x2_u' '0' memarg_8
  | 'v128.load8_splat' memarg_1  ≡  'v128.load8_splat' '0' memarg_1
  | 'v128.load16_splat' memarg_2  ≡  'v128.load16_splat' '0' memarg_2
  | 'v128.load32_splat' memarg_4  ≡  'v128.load32_splat' '0' memarg_4
  | 'v128.load64_splat' memarg_8  ≡  'v128.load64_splat' '0' memarg_8
  | 'v128.load32_zero' memarg_4  ≡  'v128.load32_zero' '0' memarg_4
  | 'v128.load64_zero' memarg_8  ≡  'v128.load64_zero' '0' memarg_8
  | 'v128.load8_lane' memarg_1 laneidx  ≡  'v128.load8_lane' '0' memarg_1 laneidx
  | 'v128.load16_lane' memarg_2 laneidx  ≡  'v128.load16_lane' '0' memarg_2 laneidx
  | 'v128.load32_lane' memarg_4 laneidx  ≡  'v128.load32_lane' '0' memarg_4 laneidx
  | 'v128.load64_lane' memarg_8 laneidx  ≡  'v128.load64_lane' '0' memarg_8 laneidx
  | 'i32.store' memarg_4  ≡  'i32.store' '0' memarg_4
  | 'i64.store' memarg_8  ≡  'i64.store' '0' memarg_8
  | 'f32.store' memarg_4  ≡  'f32.store' '0' memarg_4
  | 'f64.store' memarg_8  ≡  'f64.store' '0' memarg_8
  | 'i32.store8' memarg_1  ≡  'i32.store8' '0' memarg_1
  | 'i32.store16' memarg_2  ≡  'i32.store16' '0' memarg_2
  | 'i64.store8' memarg_1  ≡  'i64.store8' '0' memarg_1
  | 'i64.store16' memarg_2  ≡  'i64.store16' '0' memarg_2
  | 'i64.store32' memarg_4  ≡  'i64.store32' '0' memarg_4
  | 'v128.store' memarg_16  ≡  'v128.store' '0' memarg_16
  | 'v128.store8_lane' memarg_1 laneidx  ≡  'v128.store8_lane' '0' memarg_1 laneidx
  | 'v128.store16_lane' memarg_2 laneidx  ≡  'v128.store16_lane' '0' memarg_2 laneidx
  | 'v128.store32_lane' memarg_4 laneidx  ≡  'v128.store32_lane' '0' memarg_4 laneidx
  | 'v128.store64_lane' memarg_8 laneidx  ≡  'v128.store64_lane' '0' memarg_8 laneidx
  | 'memory.size'  ≡  'memory.size' '0'
  | 'memory.grow'  ≡  'memory.grow' '0'
  | 'memory.fill'  ≡  'memory.fill' '0'
  | 'memory.copy'  ≡  'memory.copy' '0' '0'
  | 'memory.init' dataidx_I  ≡  'memory.init' '0' dataidx_I
```

### Reference Instructions

```text
plaininstr_I ::=
  | ...
  | 'ref.null' ht : heaptype_I  => ref.null ht
  | 'ref.func' x : funcidx_I  => ref.func x
  | 'ref.is_null'  => ref.is_null
  | 'ref.as_non_null'  => ref.as_non_null
  | 'ref.eq'  => ref.eq
  | 'ref.test' rt : reftype_I  => ref.test rt
  | 'ref.cast' rt : reftype_I  => ref.cast rt
```

### Aggregate Instructions

```text
plaininstr_I ::=
  | ...
  | 'ref.i31'  => ref.i31
  | 'i31.get_s'  => i31.get_s
  | 'i31.get_u'  => i31.get_u
  | 'struct.new' x : typeidx_I  => struct.new x
  | 'struct.new_default' x : typeidx_I  => struct.new_default x
  | 'struct.get' x : typeidx_I i : fieldidx_I,x  => struct.get x i
  | 'struct.get_s' x : typeidx_I i : fieldidx_I,x  => struct.get_s x i
  | 'struct.get_u' x : typeidx_I i : fieldidx_I,x  => struct.get_u x i
  | 'struct.set' x : typeidx_I i : fieldidx_I,x  => struct.set x i
  | 'array.new' x : typeidx_I  => array.new x
  | 'array.new_default' x : typeidx_I  => array.new_default x
  | 'array.new_fixed' x : typeidx_I n : u32  => array.new_fixed x n
  | 'array.new_data' x : typeidx_I y : dataidx_I  => array.new_data x y
  | 'array.new_elem' x : typeidx_I y : elemidx_I  => array.new_elem x y
  | 'array.get' x : typeidx_I  => array.get x
  | 'array.get_s' x : typeidx_I  => array.get_s x
  | 'array.get_u' x : typeidx_I  => array.get_u x
  | 'array.set' x : typeidx_I  => array.set x
  | 'array.len'  => array.len
  | 'array.fill' x : typeidx_I  => array.fill x
  | 'array.copy' x1 : typeidx_I x2 : typeidx_I  => array.copy x1 x2
  | 'array.init_data' x : typeidx_I y : dataidx_I  => array.init_data x y
  | 'array.init_elem' x : typeidx_I y : elemidx_I  => array.init_elem x y
  | 'any.convert_extern'  => any.convert_extern
  | 'extern.convert_any'  => extern.convert_any
```

### Numeric Instructions

```text
plaininstr_I ::=
  | ...
  | 'i32.const' c : i32  => i32.const c
  | 'i64.const' c : i64  => i64.const c
  | 'f32.const' c : f32  => f32.const c
  | 'f64.const' c : f64  => f64.const c
```

```text
plaininstr_I ::=
  | ...
  | 'i32.eqz'  => i32.eqz
  | 'i32.eq'  => i32.eq
  | 'i32.ne'  => i32.ne
  | 'i32.lt_s'  => i32.lt_s
  | 'i32.lt_u'  => i32.lt_u
  | 'i32.gt_s'  => i32.gt_s
  | 'i32.gt_u'  => i32.gt_u
  | 'i32.le_s'  => i32.le_s
  | 'i32.le_u'  => i32.le_u
  | 'i32.ge_s'  => i32.ge_s
  | 'i32.ge_u'  => i32.ge_u
  | 'i32.clz'  => i32.clz
  | 'i32.ctz'  => i32.ctz
  | 'i32.popcnt'  => i32.popcnt
  | 'i32.extend8_s'  => i32.extend8_s
  | 'i32.extend16_s'  => i32.extend16_s
  | 'i32.add'  => i32.add
  | 'i32.sub'  => i32.sub
  | 'i32.mul'  => i32.mul
  | 'i32.div_s'  => i32.div_s
  | 'i32.div_u'  => i32.div_u
  | 'i32.rem_s'  => i32.rem_s
  | 'i32.rem_u'  => i32.rem_u
  | 'i32.and'  => i32.and
  | 'i32.or'  => i32.or
  | 'i32.xor'  => i32.xor
  | 'i32.shl'  => i32.shl
  | 'i32.shr_s'  => i32.shr_s
  | 'i32.shr_u'  => i32.shr_u
  | 'i32.rotl'  => i32.rotl
  | 'i32.rotr'  => i32.rotr
```

```text
plaininstr_I ::=
  | ...
  | 'i64.eqz'  => i64.eqz
  | 'i64.eq'  => i64.eq
  | 'i64.ne'  => i64.ne
  | 'i64.lt_s'  => i64.lt_s
  | 'i64.lt_u'  => i64.lt_u
  | 'i64.gt_s'  => i64.gt_s
  | 'i64.gt_u'  => i64.gt_u
  | 'i64.le_s'  => i64.le_s
  | 'i64.le_u'  => i64.le_u
  | 'i64.ge_s'  => i64.ge_s
  | 'i64.ge_u'  => i64.ge_u
  | 'i64.clz'  => i64.clz
  | 'i64.ctz'  => i64.ctz
  | 'i64.popcnt'  => i64.popcnt
  | 'i64.extend8_s'  => i64.extend8_s
  | 'i64.extend16_s'  => i64.extend16_s
  | 'i64.extend32_s'  => i64.extend32_s
  | 'i64.add'  => i64.add
  | 'i64.sub'  => i64.sub
  | 'i64.mul'  => i64.mul
  | 'i64.div_s'  => i64.div_s
  | 'i64.div_u'  => i64.div_u
  | 'i64.rem_s'  => i64.rem_s
  | 'i64.rem_u'  => i64.rem_u
  | 'i64.and'  => i64.and
  | 'i64.or'  => i64.or
  | 'i64.xor'  => i64.xor
  | 'i64.shl'  => i64.shl
  | 'i64.shr_s'  => i64.shr_s
  | 'i64.shr_u'  => i64.shr_u
  | 'i64.rotl'  => i64.rotl
  | 'i64.rotr'  => i64.rotr
```

```text
plaininstr_I ::=
  | ...
  | 'f32.eq'  => f32.eq
  | 'f32.ne'  => f32.ne
  | 'f32.lt'  => f32.lt
  | 'f32.gt'  => f32.gt
  | 'f32.le'  => f32.le
  | 'f32.ge'  => f32.ge
  | 'f32.abs'  => f32.abs
  | 'f32.neg'  => f32.neg
  | 'f32.sqrt'  => f32.sqrt
  | 'f32.ceil'  => f32.ceil
  | 'f32.floor'  => f32.floor
  | 'f32.trunc'  => f32.trunc
  | 'f32.nearest'  => f32.nearest
  | 'f32.add'  => f32.add
  | 'f32.sub'  => f32.sub
  | 'f32.mul'  => f32.mul
  | 'f32.div'  => f32.div
  | 'f32.min'  => f32.min
  | 'f32.max'  => f32.max
  | 'f32.copysign'  => f32.copysign
```

```text
plaininstr_I ::=
  | ...
  | 'f64.eq'  => f64.eq
  | 'f64.ne'  => f64.ne
  | 'f64.lt'  => f64.lt
  | 'f64.gt'  => f64.gt
  | 'f64.le'  => f64.le
  | 'f64.ge'  => f64.ge
  | 'f64.abs'  => f64.abs
  | 'f64.neg'  => f64.neg
  | 'f64.sqrt'  => f64.sqrt
  | 'f64.ceil'  => f64.ceil
  | 'f64.floor'  => f64.floor
  | 'f64.trunc'  => f64.trunc
  | 'f64.nearest'  => f64.nearest
  | 'f64.add'  => f64.add
  | 'f64.sub'  => f64.sub
  | 'f64.mul'  => f64.mul
  | 'f64.div'  => f64.div
  | 'f64.min'  => f64.min
  | 'f64.max'  => f64.max
  | 'f64.copysign'  => f64.copysign
```

```text
plaininstr_I ::=
  | ...
  | 'i32.wrap_i64'  => i32.wrap_i64
  | 'i32.trunc_f32_s'  => i32.trunc_f32_s
  | 'i32.trunc_f32_u'  => i32.trunc_f32_u
  | 'i32.trunc_f64_s'  => i32.trunc_f64_s
  | 'i32.trunc_f64_u'  => i32.trunc_f64_u
  | 'i32.trunc_sat_f32_s'  => i32.trunc_sat_f32_s
  | 'i32.trunc_sat_f32_u'  => i32.trunc_sat_f32_u
  | 'i32.trunc_sat_f64_s'  => i32.trunc_sat_f64_s
  | 'i32.trunc_sat_f64_u'  => i32.trunc_sat_f64_u
  | 'i64.extend_i32_s'  => i64.extend_i32_s
  | 'i64.extend_i32_u'  => i64.extend_i32_u
  | 'i64.trunc_f32_s'  => i64.trunc_f32_s
  | 'i64.trunc_f32_u'  => i64.trunc_f32_u
  | 'i64.trunc_f64_s'  => i64.trunc_f64_s
  | 'i64.trunc_f64_u'  => i64.trunc_f64_u
  | 'i64.trunc_sat_f32_s'  => i64.trunc_sat_f32_s
  | 'i64.trunc_sat_f32_u'  => i64.trunc_sat_f32_u
  | 'i64.trunc_sat_f64_s'  => i64.trunc_sat_f64_s
  | 'i64.trunc_sat_f64_u'  => i64.trunc_sat_f64_u
  | 'f32.demote_f64'  => f32.demote_f64
  | 'f32.convert_i32_s'  => f32.convert_i32_s
  | 'f32.convert_i32_u'  => f32.convert_i32_u
  | 'f32.convert_i64_s'  => f32.convert_i64_s
  | 'f32.convert_i64_u'  => f32.convert_i64_u
  | 'f64.promote_f32'  => f64.promote_f32
  | 'f64.convert_i32_s'  => f64.convert_i32_s
  | 'f64.convert_i32_u'  => f64.convert_i32_u
  | 'f64.convert_i64_s'  => f64.convert_i64_s
  | 'f64.convert_i64_u'  => f64.convert_i64_u
  | 'i32.reinterpret_f32'  => i32.reinterpret_f32
  | 'i64.reinterpret_f64'  => i64.reinterpret_f64
  | 'f32.reinterpret_i32'  => f32.reinterpret_i32
  | 'f64.reinterpret_i64'  => f64.reinterpret_i64
```

### Vector Instructions

Vector constant instructions have a mandatory shape descriptor, which determines how the following values are parsed.

```text
plaininstr_I ::=
  | ...
  | 'v128.const' 'i8x16' c* : i8^16  => v128.const bytes128^(-1)( ++ (bytes8(c)*) )
  | 'v128.const' 'i16x8' c* : i16^8  => v128.const bytes128^(-1)( ++ (bytes16(c)*) )
  | 'v128.const' 'i32x4' c* : i32^4  => v128.const bytes128^(-1)( ++ (bytes32(c)*) )
  | 'v128.const' 'i64x2' c* : i64^2  => v128.const bytes128^(-1)( ++ (bytes64(c)*) )
  | 'v128.const' 'f32x4' c* : f32^4  => v128.const bytes128^(-1)( ++ (bytes32(c)*) )
  | 'v128.const' 'f64x2' c* : f64^2  => v128.const bytes128^(-1)( ++ (bytes64(c)*) )
```

```text
plaininstr_I ::=
  | ...
  | 'i8x16.shuffle' i* : laneidx^16  => i8x16.shuffle i*
  | 'i8x16.swizzle'  => i8x16.swizzle
  | 'i8x16.relaxed_swizzle'  => i8x16.relaxed_swizzle
  | 'i8x16.splat'  => i8x16.splat
  | 'i16x8.splat'  => i16x8.splat
  | 'i32x4.splat'  => i32x4.splat
  | 'i64x2.splat'  => i64x2.splat
  | 'f32x4.splat'  => f32x4.splat
  | 'f64x2.splat'  => f64x2.splat
  | 'i8x16.extract_lane_s' i : laneidx  => i8x16.extract_lane_s i
  | 'i8x16.extract_lane_u' i : laneidx  => i8x16.extract_lane_u i
  | 'i16x8.extract_lane_s' i : laneidx  => i16x8.extract_lane_s i
  | 'i16x8.extract_lane_u' i : laneidx  => i16x8.extract_lane_u i
  | 'i32x4.extract_lane' i : laneidx  => i32x4.extract_lane i
  | 'i64x2.extract_lane' i : laneidx  => i64x2.extract_lane i
  | 'f32x4.extract_lane' i : laneidx  => f32x4.extract_lane i
  | 'f64x2.extract_lane' i : laneidx  => f64x2.extract_lane i
  | 'i8x16.replace_lane' i : laneidx  => i8x16.replace_lane i
  | 'i16x8.replace_lane' i : laneidx  => i16x8.replace_lane i
  | 'i32x4.replace_lane' i : laneidx  => i32x4.replace_lane i
  | 'i64x2.replace_lane' i : laneidx  => i64x2.replace_lane i
  | 'f32x4.replace_lane' i : laneidx  => f32x4.replace_lane i
  | 'f64x2.replace_lane' i : laneidx  => f64x2.replace_lane i
```

```text
plaininstr_I ::=
  | ...
  | 'v128.any_true'  => v128.any_true
  | 'v128.not'  => v128.not
  | 'v128.and'  => v128.and
  | 'v128.andnot'  => v128.andnot
  | 'v128.or'  => v128.or
  | 'v128.xor'  => v128.xor
  | 'v128.bitselect'  => v128.bitselect
```

```text
plaininstr_I ::=
  | ...
  | 'i8x16.all_true'  => i8x16.all_true
  | 'i8x16.eq'  => i8x16.eq
  | 'i8x16.ne'  => i8x16.ne
  | 'i8x16.lt_s'  => i8x16.lt_s
  | 'i8x16.lt_u'  => i8x16.lt_u
  | 'i8x16.gt_s'  => i8x16.gt_s
  | 'i8x16.gt_u'  => i8x16.gt_u
  | 'i8x16.le_s'  => i8x16.le_s
  | 'i8x16.le_u'  => i8x16.le_u
  | 'i8x16.ge_s'  => i8x16.ge_s
  | 'i8x16.ge_u'  => i8x16.ge_u
  | 'i8x16.abs'  => i8x16.abs
  | 'i8x16.neg'  => i8x16.neg
  | 'i8x16.popcnt'  => i8x16.popcnt
  | 'i8x16.add'  => i8x16.add
  | 'i8x16.add_sat_s'  => i8x16.add_sat_s
  | 'i8x16.add_sat_u'  => i8x16.add_sat_u
  | 'i8x16.sub'  => i8x16.sub
  | 'i8x16.sub_sat_s'  => i8x16.sub_sat_s
  | 'i8x16.sub_sat_u'  => i8x16.sub_sat_u
  | 'i8x16.min_s'  => i8x16.min_s
  | 'i8x16.min_u'  => i8x16.min_u
  | 'i8x16.max_s'  => i8x16.max_s
  | 'i8x16.max_u'  => i8x16.max_u
  | 'i8x16.avgr_u'  => i8x16.avgr_u
  | 'i8x16.relaxed_laneselect'  => i8x16.relaxed_laneselect
  | 'i8x16.shl'  => i8x16.shl
  | 'i8x16.shr_s'  => i8x16.shr_s
  | 'i8x16.shr_u'  => i8x16.shr_u
  | 'i8x16.bitmask'  => i8x16.bitmask
  | 'i8x16.narrow_i16x8_s'  => i8x16.narrow_i16x8_s
  | 'i8x16.narrow_i16x8_u'  => i8x16.narrow_i16x8_u
```

```text
plaininstr_I ::=
  | ...
  | 'i16x8.all_true'  => i16x8.all_true
  | 'i16x8.eq'  => i16x8.eq
  | 'i16x8.ne'  => i16x8.ne
  | 'i16x8.lt_s'  => i16x8.lt_s
  | 'i16x8.lt_u'  => i16x8.lt_u
  | 'i16x8.gt_s'  => i16x8.gt_s
  | 'i16x8.gt_u'  => i16x8.gt_u
  | 'i16x8.le_s'  => i16x8.le_s
  | 'i16x8.le_u'  => i16x8.le_u
  | 'i16x8.ge_s'  => i16x8.ge_s
  | 'i16x8.ge_u'  => i16x8.ge_u
  | 'i16x8.abs'  => i16x8.abs
  | 'i16x8.neg'  => i16x8.neg
  | 'i16x8.add'  => i16x8.add
  | 'i16x8.add_sat_s'  => i16x8.add_sat_s
  | 'i16x8.add_sat_u'  => i16x8.add_sat_u
  | 'i16x8.sub'  => i16x8.sub
  | 'i16x8.sub_sat_s'  => i16x8.sub_sat_s
  | 'i16x8.sub_sat_u'  => i16x8.sub_sat_u
  | 'i16x8.mul'  => i16x8.mul
  | 'i16x8.min_s'  => i16x8.min_s
  | 'i16x8.min_u'  => i16x8.min_u
  | 'i16x8.max_s'  => i16x8.max_s
  | 'i16x8.max_u'  => i16x8.max_u
  | 'i16x8.avgr_u'  => i16x8.avgr_u
  | 'i16x8.q15mulr_sat_s'  => i16x8.q15mulr_sat_s
  | 'i16x8.relaxed_q15mulr_s'  => i16x8.relaxed_q15mulr_s
  | 'i16x8.relaxed_laneselect'  => i16x8.relaxed_laneselect
  | 'i16x8.shl'  => i16x8.shl
  | 'i16x8.shr_s'  => i16x8.shr_s
  | 'i16x8.shr_u'  => i16x8.shr_u
  | 'i16x8.bitmask'  => i16x8.bitmask
  | 'i16x8.narrow_i32x4_s'  => i16x8.narrow_i32x4_s
  | 'i16x8.narrow_i32x4_u'  => i16x8.narrow_i32x4_u
```

```text
plaininstr_I ::=
  | ...
  | 'i32x4.all_true'  => i32x4.all_true
  | 'i32x4.eq'  => i32x4.eq
  | 'i32x4.ne'  => i32x4.ne
  | 'i32x4.lt_s'  => i32x4.lt_s
  | 'i32x4.lt_u'  => i32x4.lt_u
  | 'i32x4.gt_s'  => i32x4.gt_s
  | 'i32x4.gt_u'  => i32x4.gt_u
  | 'i32x4.le_s'  => i32x4.le_s
  | 'i32x4.le_u'  => i32x4.le_u
  | 'i32x4.ge_s'  => i32x4.ge_s
  | 'i32x4.ge_u'  => i32x4.ge_u
  | 'i32x4.abs'  => i32x4.abs
  | 'i32x4.neg'  => i32x4.neg
  | 'i32x4.add'  => i32x4.add
  | 'i32x4.sub'  => i32x4.sub
  | 'i32x4.mul'  => i32x4.mul
  | 'i32x4.min_s'  => i32x4.min_s
  | 'i32x4.min_u'  => i32x4.min_u
  | 'i32x4.max_s'  => i32x4.max_s
  | 'i32x4.max_u'  => i32x4.max_u
  | 'i32x4.relaxed_laneselect'  => i32x4.relaxed_laneselect
  | 'i32x4.shl'  => i32x4.shl
  | 'i32x4.shr_s'  => i32x4.shr_s
  | 'i32x4.shr_u'  => i32x4.shr_u
  | 'i32x4.bitmask'  => i32x4.bitmask
```

```text
plaininstr_I ::=
  | ...
  | 'i64x2.all_true'  => i64x2.all_true
  | 'i64x2.eq'  => i64x2.eq
  | 'i64x2.ne'  => i64x2.ne
  | 'i64x2.lt_s'  => i64x2.lt_s
  | 'i64x2.gt_s'  => i64x2.gt_s
  | 'i64x2.le_s'  => i64x2.le_s
  | 'i64x2.ge_s'  => i64x2.ge_s
  | 'i64x2.abs'  => i64x2.abs
  | 'i64x2.neg'  => i64x2.neg
  | 'i64x2.add'  => i64x2.add
  | 'i64x2.sub'  => i64x2.sub
  | 'i64x2.mul'  => i64x2.mul
  | 'i64x2.relaxed_laneselect'  => i64x2.relaxed_laneselect
  | 'i64x2.shl'  => i64x2.shl
  | 'i64x2.shr_s'  => i64x2.shr_s
  | 'i64x2.shr_u'  => i64x2.shr_u
  | 'i64x2.bitmask'  => i64x2.bitmask
```

```text
plaininstr_I ::=
  | ...
  | 'f32x4.eq'  => f32x4.eq
  | 'f32x4.ne'  => f32x4.ne
  | 'f32x4.lt'  => f32x4.lt
  | 'f32x4.gt'  => f32x4.gt
  | 'f32x4.le'  => f32x4.le
  | 'f32x4.ge'  => f32x4.ge
  | 'f32x4.abs'  => f32x4.abs
  | 'f32x4.neg'  => f32x4.neg
  | 'f32x4.sqrt'  => f32x4.sqrt
  | 'f32x4.ceil'  => f32x4.ceil
  | 'f32x4.floor'  => f32x4.floor
  | 'f32x4.trunc'  => f32x4.trunc
  | 'f32x4.nearest'  => f32x4.nearest
  | 'f32x4.add'  => f32x4.add
  | 'f32x4.sub'  => f32x4.sub
  | 'f32x4.mul'  => f32x4.mul
  | 'f32x4.div'  => f32x4.div
  | 'f32x4.min'  => f32x4.min
  | 'f32x4.max'  => f32x4.max
  | 'f32x4.copysign'  => f32x4.copysign
  | 'f32x4.pmin'  => f32x4.pmin
  | 'f32x4.pmax'  => f32x4.pmax
  | 'f32x4.relaxed_min'  => f32x4.relaxed_min
  | 'f32x4.relaxed_max'  => f32x4.relaxed_max
  | 'f32x4.relaxed_madd'  => f32x4.relaxed_madd
  | 'f32x4.relaxed_nmadd'  => f32x4.relaxed_nmadd
```

```text
plaininstr_I ::=
  | ...
  | 'f64x2.eq'  => f64x2.eq
  | 'f64x2.ne'  => f64x2.ne
  | 'f64x2.lt'  => f64x2.lt
  | 'f64x2.gt'  => f64x2.gt
  | 'f64x2.le'  => f64x2.le
  | 'f64x2.ge'  => f64x2.ge
  | 'f64x2.abs'  => f64x2.abs
  | 'f64x2.neg'  => f64x2.neg
  | 'f64x2.sqrt'  => f64x2.sqrt
  | 'f64x2.ceil'  => f64x2.ceil
  | 'f64x2.floor'  => f64x2.floor
  | 'f64x2.trunc'  => f64x2.trunc
  | 'f64x2.nearest'  => f64x2.nearest
  | 'f64x2.add'  => f64x2.add
  | 'f64x2.sub'  => f64x2.sub
  | 'f64x2.mul'  => f64x2.mul
  | 'f64x2.div'  => f64x2.div
  | 'f64x2.min'  => f64x2.min
  | 'f64x2.max'  => f64x2.max
  | 'f64x2.pmin'  => f64x2.pmin
  | 'f64x2.pmax'  => f64x2.pmax
  | 'f64x2.relaxed_min'  => f64x2.relaxed_min
  | 'f64x2.relaxed_max'  => f64x2.relaxed_max
  | 'f64x2.relaxed_madd'  => f64x2.relaxed_madd
  | 'f64x2.relaxed_nmadd'  => f64x2.relaxed_nmadd
```

```text
plaininstr_I ::=
  | ...
  | 'i16x8.extend_low_i8x16_s'  => i16x8.extend_low_i8x16_s
  | 'i16x8.extend_low_i8x16_u'  => i16x8.extend_low_i8x16_u
  | 'i16x8.extend_high_i8x16_s'  => i16x8.extend_high_i8x16_s
  | 'i16x8.extend_high_i8x16_u'  => i16x8.extend_high_i8x16_u
  | 'i32x4.extend_low_i16x8_s'  => i32x4.extend_low_i16x8_s
  | 'i32x4.extend_low_i16x8_u'  => i32x4.extend_low_i16x8_u
  | 'i32x4.extend_high_i16x8_s'  => i32x4.extend_high_i16x8_s
  | 'i32x4.extend_high_i16x8_u'  => i32x4.extend_high_i16x8_u
  | 'i32x4.trunc_sat_f32x4_s'  => i32x4.trunc_sat_f32x4_s
  | 'i32x4.trunc_sat_f32x4_u'  => i32x4.trunc_sat_f32x4_u
  | 'i32x4.trunc_sat_f64x2_s_zero'  => i32x4.trunc_sat_f64x2_s_zero
  | 'i32x4.trunc_sat_f64x2_u_zero'  => i32x4.trunc_sat_f64x2_u_zero
  | 'i32x4.relaxed_trunc_f32x4_s'  => i32x4.relaxed_trunc_f32x4_s
  | 'i32x4.relaxed_trunc_f32x4_u'  => i32x4.relaxed_trunc_f32x4_u
  | 'i32x4.relaxed_trunc_f64x2_s_zero'  => i32x4.relaxed_trunc_f64x2_s_zero
  | 'i32x4.relaxed_trunc_f64x2_u_zero'  => i32x4.relaxed_trunc_f64x2_u_zero
  | 'i64x2.extend_low_i32x4_s'  => i64x2.extend_low_i32x4_s
  | 'i64x2.extend_low_i32x4_u'  => i64x2.extend_low_i32x4_u
  | 'i64x2.extend_high_i32x4_s'  => i64x2.extend_high_i32x4_s
  | 'i64x2.extend_high_i32x4_u'  => i64x2.extend_high_i32x4_u
  | 'f32x4.demote_f64x2_zero'  => f32x4.demote_f64x2_zero
  | 'f32x4.convert_i32x4_s'  => f32x4.convert_i32x4_s
  | 'f32x4.convert_i32x4_u'  => f32x4.convert_i32x4_u
  | 'f64x2.promote_low_f32x4'  => f64x2.promote_low_f32x4
  | 'f64x2.convert_low_i32x4_s'  => f64x2.convert_low_i32x4_s
  | 'f64x2.convert_low_i32x4_u'  => f64x2.convert_low_i32x4_u
```

```text
plaininstr_I ::=
  | ...
  | 'i16x8.extadd_pairwise_i8x16_s'  => i16x8.extadd_pairwise_i8x16_s
  | 'i16x8.extadd_pairwise_i8x16_u'  => i16x8.extadd_pairwise_i8x16_u
  | 'i16x8.extmul_low_i8x16_s'  => i16x8.extmul_low_i8x16_s
  | 'i16x8.extmul_low_i8x16_u'  => i16x8.extmul_low_i8x16_u
  | 'i16x8.extmul_high_i8x16_s'  => i16x8.extmul_high_i8x16_s
  | 'i16x8.extmul_high_i8x16_u'  => i16x8.extmul_high_i8x16_u
  | 'i16x8.relaxed_dot_i8x16_i7x16_s'  => i16x8.relaxed_dot_i8x16_i7x16_s
  | 'i32x4.extadd_pairwise_i16x8_s'  => i32x4.extadd_pairwise_i16x8_s
  | 'i32x4.extadd_pairwise_i16x8_u'  => i32x4.extadd_pairwise_i16x8_u
  | 'i32x4.extmul_low_i16x8_s'  => i32x4.extmul_low_i16x8_s
  | 'i32x4.extmul_low_i16x8_u'  => i32x4.extmul_low_i16x8_u
  | 'i32x4.extmul_high_i16x8_s'  => i32x4.extmul_high_i16x8_s
  | 'i32x4.extmul_high_i16x8_u'  => i32x4.extmul_high_i16x8_u
  | 'i32x4.dot_i16x8_s'  => i32x4.dot_i16x8_s
  | 'i32x4.relaxed_dot_i8x16_i7x16_add_s'  => i32x4.relaxed_dot_i8x16_i7x16_add_s
  | 'i64x2.extmul_low_i32x4_s'  => i64x2.extmul_low_i32x4_s
  | 'i64x2.extmul_low_i32x4_u'  => i64x2.extmul_low_i32x4_u
  | 'i64x2.extmul_high_i32x4_s'  => i64x2.extmul_high_i32x4_s
  | 'i64x2.extmul_high_i32x4_u'  => i64x2.extmul_high_i32x4_u
```

### Folded Instructions

Instructions can be written as S-expressions by grouping them into *folded* form. In that notation, an instruction is wrapped in parentheses and optionally includes nested folded instructions to indicate its operands.

In the case of block instructions, the folded form omits the `end` delimiter.

For `if` instructions, both branches have to be wrapped into nested S-expressions, headed by the keywords `then` and `else`.

The set of all phrases defined by the following abbreviations recursively forms the auxiliary syntactic class `foldedinstr`.

Such a folded instruction can appear anywhere a regular instruction can.

```text
foldedinstr_I ::=
  | '(' plaininstr_I instrs_I ')'  ≡  instrs_I plaininstr_I
  | '(' 'block' label_I blocktype_I instrs_I' ')'  ≡  'block' label_I blocktype_I instrs_I' 'end'
  | '(' 'loop' label_I blocktype_I instrs_I' ')'  ≡  'loop' label_I blocktype_I instrs_I' 'end'
  | '(' 'if' label_I blocktype_I foldedinstr_I* '(' 'then' in1* : instrs_I' ')' ('(' 'else' in2* : instrs_I' ')')^? ')'
        ≡  foldedinstr_I* 'if' label_I blocktype_I in1* : instrs_I' ('else' in2* : instrs_I')^? 'end'
  | '(' 'try_table' label_I blocktype_I catch_I* instrs_I' ')'  ≡  'try_table' label_I blocktype_I catch_I* instrs_I' 'end'
```

> **Note:** For example, the instruction sequence
>
> ```text
> (local.get $x) (i32.const 2) i32.add (i32.const 3) i32.mul
> ```
>
> can be folded into
>
> ```text
> (i32.mul (i32.add (local.get $x) (i32.const 2)) (i32.const 3))
> ```
>
> Folded instructions are solely syntactic sugar, no additional syntactic or type-based checking is implied.

### Expressions

Expressions are written as instruction sequences.

```text
expr_I ::=
  | in* : instrs_I  => in*
```
