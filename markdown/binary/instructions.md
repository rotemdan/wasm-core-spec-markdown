## Instructions

Instructions are encoded by *opcodes*.
Each opcode is represented by a single byte,
and is followed by the instruction's immediate arguments, where present.
The only exception are structured control instructions, which consist of several opcodes bracketing their nested instruction sequences.

> **Note:** The byte codes chosen to encode instructions are historical and do not follow a consistent pattern. In this section, instructions are hence not presented in opcode order, but instead grouped consistently with other sections in this document. An instruction index ordered by opcode can be found in the Appendix.
>
> Gaps in the byte code ranges are reserved for future extensions.

### Parametric Instructions

Parametric instructions are represented by single byte codes, possibly followed by a type annotation.

```text
instr ::=
  | 0x00                             => unreachable
  | 0x01                             => nop
  | 0x1A                             => drop
  | 0x1B                             => select
  | 0x1C t* : list(valtype)          => select t*
```

### Control Instructions

Control instructions have varying encodings. For structured instructions, the instruction sequences forming nested blocks are delimited with explicit opcodes for `end` and `else`.

Block types are encoded in special compressed form, by either the byte `0x40` indicating the empty type, as a single value type, or as a type index encoded as a positive signed integer.

```text
blocktype ::=
  | 0x40                  => ε
  | t : valtype           => t
  | i : s33               => i (if i >= 0)

instr ::=
  | ...
  | 0x02 bt : blocktype (in : instr)* 0x0B    => block bt in*
  | 0x03 bt : blocktype (in : instr)* 0x0B    => loop bt in*
  | 0x04 bt : blocktype (in : instr)* 0x0B    => if bt in* else ε
  | 0x04 bt : blocktype in1* : list(instr) 0x05 in2* : list(instr) 0x0B    => if bt in1* else in2*
  | 0x08 x : tagidx       => throw x
  | 0x0A                  => throw_ref
  | 0x0C l : labelidx     => br l
  | 0x0D l : labelidx     => br_if l
  | 0x0E l* : list(labelidx) l_n : labelidx    => br_table l* l_n
  | 0x0F                  => return
  | 0x10 x : funcidx      => call x
  | 0x11 y : typeidx x : tableidx    => call_indirect x y
  | 0x12 x : funcidx      => return_call x
  | 0x13 y : typeidx x : tableidx    => return_call_indirect x y
  | 0x14 x : typeidx      => call_ref x
  | 0x15 x : typeidx      => return_call_ref x
  | 0x1F bt : blocktype c* : list(catch) (in : instr)* 0x0B    => try_table bt c* in*
  | 0xD5 l : labelidx     => br_on_null l
  | 0xD6 l : labelidx     => br_on_non_null l
  | 0xFB 24 : u32 (null1?, null2?) : castop l : labelidx ht1 : heaptype ht2 : heaptype    => br_on_cast l (ref null1? ht1) (ref null2? ht2)
  | 0xFB 25 : u32 (null1?, null2?) : castop l : labelidx ht1 : heaptype ht2 : heaptype    => br_on_cast_fail l (ref null1? ht1) (ref null2? ht2)

catch ::=
  | 0x00 x : tagidx l : labelidx    => catch x l
  | 0x01 x : tagidx l : labelidx    => catch_ref x l
  | 0x02 l : labelidx               => catch_all l
  | 0x03 l : labelidx               => catch_all_ref l

castop ::=
  | 0x00    => (ε, ε)
  | 0x01    => (null, ε)
  | 0x02    => (ε, null)
  | 0x03    => (null, null)
```

> **Note:** The `else` opcode `0x05` in the encoding of an `if` instruction can be omitted if the following instruction sequence is empty.
>
> Unlike any other occurrence, the type index in a block type is encoded as a positive signed integer, so that its SignedLEB128 bit pattern cannot collide with the encoding of value types or the special code `0x40`, which correspond to the LEB128 encoding of negative integers. To avoid any loss in the range of allowed indices, it is treated as a 33 bit signed integer.

### Variable Instructions

Variable instructions are represented by byte codes followed by the encoding of the respective index.

```text
instr ::=
  | ...
  | 0x20 x : localidx     => local.get x
  | 0x21 x : localidx     => local.set x
  | 0x22 x : localidx     => local.tee x
  | 0x23 x : globalidx    => global.get x
  | 0x24 x : globalidx    => global.set x
```

### Table Instructions

Table instructions are represented either by a single byte or a one byte prefix followed by a variable-length unsigned integer.

```text
instr ::=
  | ...
  | 0x25 x : tableidx                 => table.get x
  | 0x26 x : tableidx                 => table.set x
  | 0xFC 12 : u32 y : elemidx x : tableidx    => table.init x y
  | 0xFC 13 : u32 x : elemidx         => elem.drop x
  | 0xFC 14 : u32 x1 : tableidx x2 : tableidx    => table.copy x1 x2
  | 0xFC 15 : u32 x : tableidx        => table.grow x
  | 0xFC 16 : u32 x : tableidx        => table.size x
  | 0xFC 17 : u32 x : tableidx        => table.fill x
```

### Memory Instructions

Each variant of memory instruction is encoded with a different byte code. Loads and stores are followed by the encoding of their memarg immediate, which includes the memory index if bit 6 of the flags field containing alignment is set; the memory index defaults to 0 otherwise.

```text
memarg ::=
  | n : u32 m : u64    => (0, { align n, offset m }) (if n < 2^6)
  | n : u32 x : memidx m : u64    => (x, { align (n - 2^6), offset m }) (if 2^6 <= n < 2^7)

instr ::=
  | ...
  | 0x28 (x, ao) : memarg    => i32.load x ao
  | 0x29 (x, ao) : memarg    => i64.load x ao
  | 0x2A (x, ao) : memarg    => f32.load x ao
  | 0x2B (x, ao) : memarg    => f64.load x ao
  | 0x2C (x, ao) : memarg    => i32.load8_s x ao
  | 0x2D (x, ao) : memarg    => i32.load8_u x ao
  | 0x2E (x, ao) : memarg    => i32.load16_s x ao
  | 0x2F (x, ao) : memarg    => i32.load16_u x ao
  | 0x30 (x, ao) : memarg    => i64.load8_s x ao
  | 0x31 (x, ao) : memarg    => i64.load8_u x ao
  | 0x32 (x, ao) : memarg    => i64.load16_s x ao
  | 0x33 (x, ao) : memarg    => i64.load16_u x ao
  | 0x34 (x, ao) : memarg    => i64.load32_s x ao
  | 0x35 (x, ao) : memarg    => i64.load32_u x ao
  | 0x36 (x, ao) : memarg    => i32.store x ao
  | 0x37 (x, ao) : memarg    => i64.store x ao
  | 0x38 (x, ao) : memarg    => f32.store x ao
  | 0x39 (x, ao) : memarg    => f64.store x ao
  | 0x3A (x, ao) : memarg    => i32.store8 x ao
  | 0x3B (x, ao) : memarg    => i32.store16 x ao
  | 0x3C (x, ao) : memarg    => i64.store8 x ao
  | 0x3D (x, ao) : memarg    => i64.store16 x ao
  | 0x3E (x, ao) : memarg    => i64.store32 x ao
  | 0x3F x : memidx         => memory.size x
  | 0x40 x : memidx         => memory.grow x
  | 0xFC 8 : u32 y : dataidx x : memidx    => memory.init x y
  | 0xFC 9 : u32 x : dataidx    => data.drop x
  | 0xFC 10 : u32 x1 : memidx x2 : memidx    => memory.copy x1 x2
  | 0xFC 11 : u32 x : memidx    => memory.fill x
```

### Reference Instructions

Generic reference instructions are represented by single byte codes, others use prefixes and type operands.

```text
instr ::=
  | ...
  | 0xD0 ht : heaptype    => ref.null ht
  | 0xD1                  => ref.is_null
  | 0xD2 x : funcidx      => ref.func x
  | 0xD3                  => ref.eq
  | 0xD4                  => ref.as_non_null
  | 0xFB 20 : u32 ht : heaptype    => ref.test (ref ht)
  | 0xFB 21 : u32 ht : heaptype    => ref.test (ref null ht)
  | 0xFB 22 : u32 ht : heaptype    => ref.cast (ref ht)
  | 0xFB 23 : u32 ht : heaptype    => ref.cast (ref null ht)
```

### Aggregate Instructions

Aggregate instructions all use a prefix.

```text
instr ::=
  | ...
  | 0xFB 0 : u32 x : typeidx           => struct.new x
  | 0xFB 1 : u32 x : typeidx           => struct.new_default x
  | 0xFB 2 : u32 x : typeidx i : fieldidx     => struct.get x i
  | 0xFB 3 : u32 x : typeidx i : fieldidx     => struct.get_s x i
  | 0xFB 4 : u32 x : typeidx i : fieldidx     => struct.get_u x i
  | 0xFB 5 : u32 x : typeidx i : fieldidx     => struct.set x i
  | 0xFB 6 : u32 x : typeidx           => array.new x
  | 0xFB 7 : u32 x : typeidx           => array.new_default x
  | 0xFB 8 : u32 x : typeidx n : u32   => array.new_fixed x n
  | 0xFB 9 : u32 x : typeidx y : dataidx       => array.new_data x y
  | 0xFB 10 : u32 x : typeidx y : elemidx      => array.new_elem x y
  | 0xFB 11 : u32 x : typeidx          => array.get x
  | 0xFB 12 : u32 x : typeidx          => array.get_s x
  | 0xFB 13 : u32 x : typeidx          => array.get_u x
  | 0xFB 14 : u32 x : typeidx          => array.set x
  | 0xFB 15 : u32                     => array.len
  | 0xFB 16 : u32 x : typeidx         => array.fill x
  | 0xFB 17 : u32 x1 : typeidx x2 : typeidx    => array.copy x1 x2
  | 0xFB 18 : u32 x : typeidx y : dataidx      => array.init_data x y
  | 0xFB 19 : u32 x : typeidx y : elemidx      => array.init_elem x y
  | 0xFB 26 : u32                     => any.convert_extern
  | 0xFB 27 : u32                     => extern.convert_any
  | 0xFB 28 : u32                     => ref.i31
  | 0xFB 29 : u32                     => i31.get_s
  | 0xFB 30 : u32                     => i31.get_u
```

### Numeric Instructions

All variants of numeric instructions are represented by separate byte codes.
The `const` instructions are followed by the respective literal.

```text
instr ::=
  | ...
  | 0x41 i : i32    => i32.const i
  | 0x42 i : i64    => i64.const i
  | 0x43 p : f32    => f32.const p
  | 0x44 p : f64    => f64.const p
```

All other numeric instructions are plain opcodes without any immediates.

```text
instr ::=
  | ...
  | 0x45    => i32.eqz
  | 0x46    => i32.eq
  | 0x47    => i32.ne
  | 0x48    => i32.lt_s
  | 0x49    => i32.lt_u
  | 0x4A    => i32.gt_s
  | 0x4B    => i32.gt_u
  | 0x4C    => i32.le_s
  | 0x4D    => i32.le_u
  | 0x4E    => i32.ge_s
  | 0x4F    => i32.ge_u
```

```text
instr ::=
  | ...
  | 0x50    => i64.eqz
  | 0x51    => i64.eq
  | 0x52    => i64.ne
  | 0x53    => i64.lt_s
  | 0x54    => i64.lt_u
  | 0x55    => i64.gt_s
  | 0x56    => i64.gt_u
  | 0x57    => i64.le_s
  | 0x58    => i64.le_u
  | 0x59    => i64.ge_s
  | 0x5A    => i64.ge_u
```

```text
instr ::=
  | ...
  | 0x5B    => f32.eq
  | 0x5C    => f32.ne
  | 0x5D    => f32.lt
  | 0x5E    => f32.gt
  | 0x5F    => f32.le
  | 0x60    => f32.ge
```

```text
instr ::=
  | ...
  | 0x61    => f64.eq
  | 0x62    => f64.ne
  | 0x63    => f64.lt
  | 0x64    => f64.gt
  | 0x65    => f64.le
  | 0x66    => f64.ge
```

```text
instr ::=
  | ...
  | 0x67    => i32.clz
  | 0x68    => i32.ctz
  | 0x69    => i32.popcnt
  | 0x6A    => i32.add
  | 0x6B    => i32.sub
  | 0x6C    => i32.mul
  | 0x6D    => i32.div_s
  | 0x6E    => i32.div_u
  | 0x6F    => i32.rem_s
  | 0x70    => i32.rem_u
  | 0x71    => i32.and
  | 0x72    => i32.or
  | 0x73    => i32.xor
  | 0x74    => i32.shl
  | 0x75    => i32.shr_s
  | 0x76    => i32.shr_u
  | 0x77    => i32.rotl
  | 0x78    => i32.rotr
```

```text
instr ::=
  | ...
  | 0x79    => i64.clz
  | 0x7A    => i64.ctz
  | 0x7B    => i64.popcnt
  | 0x7C    => i64.add
  | 0x7D    => i64.sub
  | 0x7E    => i64.mul
  | 0x7F    => i64.div_s
  | 0x80    => i64.div_u
  | 0x81    => i64.rem_s
  | 0x82    => i64.rem_u
  | 0x83    => i64.and
  | 0x84    => i64.or
  | 0x85    => i64.xor
  | 0x86    => i64.shl
  | 0x87    => i64.shr_s
  | 0x88    => i64.shr_u
  | 0x89    => i64.rotl
  | 0x8A    => i64.rotr
```

```text
instr ::=
  | ...
  | 0x8B    => f32.abs
  | 0x8C    => f32.neg
  | 0x8D    => f32.ceil
  | 0x8E    => f32.floor
  | 0x8F    => f32.trunc
  | 0x90    => f32.nearest
  | 0x91    => f32.sqrt
  | 0x92    => f32.add
  | 0x93    => f32.sub
  | 0x94    => f32.mul
  | 0x95    => f32.div
  | 0x96    => f32.fmin
  | 0x97    => f32.fmax
  | 0x98    => f32.copysign
```

```text
instr ::=
  | ...
  | 0x99    => f64.abs
  | 0x9A    => f64.neg
  | 0x9B    => f64.ceil
  | 0x9C    => f64.floor
  | 0x9D    => f64.trunc
  | 0x9E    => f64.nearest
  | 0x9F    => f64.sqrt
  | 0xA0    => f64.add
  | 0xA1    => f64.sub
  | 0xA2    => f64.mul
  | 0xA3    => f64.div
  | 0xA4    => f64.fmin
  | 0xA5    => f64.fmax
  | 0xA6    => f64.copysign
```

```text
instr ::=
  | ...
  | 0xA7    => i32.wrap_i64
  | 0xA8    => i32.trunc_s_f32
  | 0xA9    => i32.trunc_u_f32
  | 0xAA    => i32.trunc_s_f64
  | 0xAB    => i32.trunc_u_f64
  | 0xAC    => i64.extend_s_i32
  | 0xAD    => i64.extend_u_i32
  | 0xAE    => i64.trunc_s_f32
  | 0xAF    => i64.trunc_u_f32
  | 0xB0    => i64.trunc_s_f64
  | 0xB1    => i64.trunc_u_f64
  | 0xB2    => f32.convert_s_i32
  | 0xB3    => f32.convert_u_i32
  | 0xB4    => f32.convert_s_i64
  | 0xB5    => f32.convert_u_i64
  | 0xB6    => f32.demote_f64
  | 0xB7    => f64.convert_s_i32
  | 0xB8    => f64.convert_u_i32
  | 0xB9    => f64.convert_s_i64
  | 0xBA    => f64.convert_u_i64
  | 0xBB    => f64.promote_f32
  | 0xBC    => i32.reinterpret_f32
  | 0xBD    => i64.reinterpret_f64
  | 0xBE    => f32.reinterpret_i32
  | 0xBF    => f64.reinterpret_i64
```

```text
instr ::=
  | ...
  | 0xC0    => i32.extend8_s
  | 0xC1    => i32.extend16_s
  | 0xC2    => i64.extend8_s
  | 0xC3    => i64.extend16_s
  | 0xC4    => i64.extend32_s
```

The saturating truncation instructions all have a one byte prefix, whereas the actual opcode is encoded by a variable-length unsigned integer.

```text
instr ::=
  | ...
  | 0xFC 0 : u32    => i32.trunc_sat_s_f32
  | 0xFC 1 : u32    => i32.trunc_sat_u_f32
  | 0xFC 2 : u32    => i32.trunc_sat_s_f64
  | 0xFC 3 : u32    => i32.trunc_sat_u_f64
  | 0xFC 4 : u32    => i64.trunc_sat_s_f32
  | 0xFC 5 : u32    => i64.trunc_sat_u_f32
  | 0xFC 6 : u32    => i64.trunc_sat_s_f64
  | 0xFC 7 : u32    => i64.trunc_sat_u_f64
```

### Vector Instructions

All variants of vector instructions are represented by separate byte codes. They all have a one byte prefix, whereas the actual opcode is encoded by a variable-length unsigned integer.

Vector loads and stores are followed by the encoding of their memarg immediate.

```text
laneidx ::=
  | l : byte    => l

instr ::=
  | ...
  | 0xFD 0 : u32 (x, ao) : memarg                 => v128.load x ao
  | 0xFD 1 : u32 (x, ao) : memarg                 => v128.load8x8_s x ao
  | 0xFD 2 : u32 (x, ao) : memarg                 => v128.load8x8_u x ao
  | 0xFD 3 : u32 (x, ao) : memarg                 => v128.load16x4_s x ao
  | 0xFD 4 : u32 (x, ao) : memarg                 => v128.load16x4_u x ao
  | 0xFD 5 : u32 (x, ao) : memarg                 => v128.load32x2_s x ao
  | 0xFD 6 : u32 (x, ao) : memarg                 => v128.load32x2_u x ao
  | 0xFD 7 : u32 (x, ao) : memarg                 => v128.load8_splat x ao
  | 0xFD 8 : u32 (x, ao) : memarg                 => v128.load16_splat x ao
  | 0xFD 9 : u32 (x, ao) : memarg                 => v128.load32_splat x ao
  | 0xFD 10 : u32 (x, ao) : memarg                => v128.load64_splat x ao
  | 0xFD 11 : u32 (x, ao) : memarg                => v128.store x ao
  | 0xFD 84 : u32 (x, ao) : memarg i : laneidx    => v128.load8_lane x ao i
  | 0xFD 85 : u32 (x, ao) : memarg i : laneidx    => v128.load16_lane x ao i
  | 0xFD 86 : u32 (x, ao) : memarg i : laneidx    => v128.load32_lane x ao i
  | 0xFD 87 : u32 (x, ao) : memarg i : laneidx    => v128.load64_lane x ao i
  | 0xFD 88 : u32 (x, ao) : memarg i : laneidx    => v128.store8_lane x ao i
  | 0xFD 89 : u32 (x, ao) : memarg i : laneidx    => v128.store16_lane x ao i
  | 0xFD 90 : u32 (x, ao) : memarg i : laneidx    => v128.store32_lane x ao i
  | 0xFD 91 : u32 (x, ao) : memarg i : laneidx    => v128.store64_lane x ao i
  | 0xFD 92 : u32 (x, ao) : memarg                => v128.load32_zero x ao
  | 0xFD 93 : u32 (x, ao) : memarg                => v128.load64_zero x ao
```

The `const` instruction for vectors is followed by 16 immediate bytes, which are converted into an `u128` in little endian byte order:

```text
instr ::=
  | ...
  | 0xFD 12 : u32 (b : byte)^16    => v128.const bytes_{u128}^{-1}(b^16)
```

The `shuffle` instruction is also followed by the encoding of 16 `laneidx` immediates.

```text
instr ::=
  | ...
  | 0xFD 13 : u32 (l : laneidx)^16    => i8x16.shuffle l^16
  | 0xFD 14 : u32                    => i8x16.swizzle
  | 0xFD 256 : u32                   => i8x16.relaxed_swizzle
```

Lane instructions are followed by the encoding of a `laneidx` immediate.

```text
instr ::=
  | ...
  | 0xFD 21 : u32 l : laneidx    => i8x16.extract_lane_s l
  | 0xFD 22 : u32 l : laneidx    => i8x16.extract_lane_u l
  | 0xFD 23 : u32 l : laneidx    => i8x16.replace_lane l
  | 0xFD 24 : u32 l : laneidx    => i16x8.extract_lane_s l
  | 0xFD 25 : u32 l : laneidx    => i16x8.extract_lane_u l
  | 0xFD 26 : u32 l : laneidx    => i16x8.replace_lane l
  | 0xFD 27 : u32 l : laneidx    => i32x4.extract_lane l
  | 0xFD 28 : u32 l : laneidx    => i32x4.replace_lane l
  | 0xFD 29 : u32 l : laneidx    => i64x2.extract_lane l
  | 0xFD 30 : u32 l : laneidx    => i64x2.replace_lane l
  | 0xFD 31 : u32 l : laneidx    => f32x4.extract_lane l
  | 0xFD 32 : u32 l : laneidx    => f32x4.replace_lane l
  | 0xFD 33 : u32 l : laneidx    => f64x2.extract_lane l
  | 0xFD 34 : u32 l : laneidx    => f64x2.replace_lane l
```

All other vector instructions are plain opcodes without any immediates.

```text
instr ::=
  | ...
  | 0xFD 15 : u32    => i8x16.splat
  | 0xFD 16 : u32    => i16x8.splat
  | 0xFD 17 : u32    => i32x4.splat
  | 0xFD 18 : u32    => i64x2.splat
  | 0xFD 19 : u32    => f32x4.splat
  | 0xFD 20 : u32    => f64x2.splat
```

```text
instr ::=
  | ...
  | 0xFD 35 : u32    => i8x16.eq
  | 0xFD 36 : u32    => i8x16.ne
  | 0xFD 37 : u32    => i8x16.lt_s
  | 0xFD 38 : u32    => i8x16.lt_u
  | 0xFD 39 : u32    => i8x16.gt_s
  | 0xFD 40 : u32    => i8x16.gt_u
  | 0xFD 41 : u32    => i8x16.le_s
  | 0xFD 42 : u32    => i8x16.le_u
  | 0xFD 43 : u32    => i8x16.ge_s
  | 0xFD 44 : u32    => i8x16.ge_u
  | 0xFD 45 : u32    => i16x8.eq
  | 0xFD 46 : u32    => i16x8.ne
  | 0xFD 47 : u32    => i16x8.lt_s
  | 0xFD 48 : u32    => i16x8.lt_u
  | 0xFD 49 : u32    => i16x8.gt_s
  | 0xFD 50 : u32    => i16x8.gt_u
  | 0xFD 51 : u32    => i16x8.le_s
  | 0xFD 52 : u32    => i16x8.le_u
  | 0xFD 53 : u32    => i16x8.ge_s
  | 0xFD 54 : u32    => i16x8.ge_u
  | 0xFD 55 : u32    => i32x4.eq
  | 0xFD 56 : u32    => i32x4.ne
  | 0xFD 57 : u32    => i32x4.lt_s
  | 0xFD 58 : u32    => i32x4.lt_u
  | 0xFD 59 : u32    => i32x4.gt_s
  | 0xFD 60 : u32    => i32x4.gt_u
  | 0xFD 61 : u32    => i32x4.le_s
  | 0xFD 62 : u32    => i32x4.le_u
  | 0xFD 63 : u32    => i32x4.ge_s
  | 0xFD 64 : u32    => i32x4.ge_u
  | 0xFD 214 : u32   => i64x2.eq
  | 0xFD 215 : u32   => i64x2.ne
  | 0xFD 216 : u32   => i64x2.lt_s
  | 0xFD 217 : u32   => i64x2.gt_s
  | 0xFD 218 : u32   => i64x2.le_s
  | 0xFD 219 : u32   => i64x2.ge_s
```

```text
instr ::=
  | ...
  | 0xFD 65 : u32    => f32x4.eq
  | 0xFD 66 : u32    => f32x4.ne
  | 0xFD 67 : u32    => f32x4.lt
  | 0xFD 68 : u32    => f32x4.gt
  | 0xFD 69 : u32    => f32x4.le
  | 0xFD 70 : u32    => f32x4.ge
  | 0xFD 71 : u32    => f64x2.eq
  | 0xFD 72 : u32    => f64x2.ne
  | 0xFD 73 : u32    => f64x2.lt
  | 0xFD 74 : u32    => f64x2.gt
  | 0xFD 75 : u32    => f64x2.le
  | 0xFD 76 : u32    => f64x2.ge
```

```text
instr ::=
  | ...
  | 0xFD 77 : u32    => v128.not
  | 0xFD 78 : u32    => v128.and
  | 0xFD 79 : u32    => v128.andnot
  | 0xFD 80 : u32    => v128.or
  | 0xFD 81 : u32    => v128.xor
  | 0xFD 82 : u32    => v128.bitselect
  | 0xFD 83 : u32    => v128.any_true
```

```text
instr ::=
  | ...
  | 0xFD 96 : u32     => i8x16.abs
  | 0xFD 97 : u32     => i8x16.neg
  | 0xFD 98 : u32     => i8x16.popcnt
  | 0xFD 99 : u32     => i8x16.all_true
  | 0xFD 100 : u32    => i8x16.bitmask
  | 0xFD 101 : u32    => i8x16.narrow_i16x8_s
  | 0xFD 102 : u32    => i8x16.narrow_i16x8_u
  | 0xFD 107 : u32    => i8x16.shl
  | 0xFD 108 : u32    => i8x16.shr_s
  | 0xFD 109 : u32    => i8x16.shr_u
  | 0xFD 110 : u32    => i8x16.add
  | 0xFD 111 : u32    => i8x16.add_sat_s
  | 0xFD 112 : u32    => i8x16.add_sat_u
  | 0xFD 113 : u32    => i8x16.sub
  | 0xFD 114 : u32    => i8x16.sub_sat_s
  | 0xFD 115 : u32    => i8x16.sub_sat_u
  | 0xFD 118 : u32    => i8x16.min_s
  | 0xFD 119 : u32    => i8x16.min_u
  | 0xFD 120 : u32    => i8x16.max_s
  | 0xFD 121 : u32    => i8x16.max_u
  | 0xFD 123 : u32    => i8x16.avgr_u
```

```text
instr ::=
  | ...
  | 0xFD 124 : u32    => i16x8.extadd_pairwise_s_i8x16
  | 0xFD 125 : u32    => i16x8.extadd_pairwise_u_i8x16
  | 0xFD 128 : u32    => i16x8.abs
  | 0xFD 129 : u32    => i16x8.neg
  | 0xFD 131 : u32    => i16x8.all_true
  | 0xFD 132 : u32    => i16x8.bitmask
  | 0xFD 133 : u32    => i16x8.narrow_i32x4_s
  | 0xFD 134 : u32    => i16x8.narrow_i32x4_u
  | 0xFD 135 : u32    => i16x8.extend_low_s_i8x16
  | 0xFD 136 : u32    => i16x8.extend_high_s_i8x16
  | 0xFD 137 : u32    => i16x8.extend_low_u_i8x16
  | 0xFD 138 : u32    => i16x8.extend_high_u_i8x16
  | 0xFD 139 : u32    => i16x8.shl
  | 0xFD 140 : u32    => i16x8.shr_s
  | 0xFD 141 : u32    => i16x8.shr_u
  | 0xFD 130 : u32    => i16x8.q15mulr_sat_s
  | 0xFD 142 : u32    => i16x8.add
  | 0xFD 143 : u32    => i16x8.add_sat_s
  | 0xFD 144 : u32    => i16x8.add_sat_u
  | 0xFD 145 : u32    => i16x8.sub
  | 0xFD 146 : u32    => i16x8.sub_sat_s
  | 0xFD 147 : u32    => i16x8.sub_sat_u
  | 0xFD 149 : u32    => i16x8.mul
  | 0xFD 150 : u32    => i16x8.min_s
  | 0xFD 151 : u32    => i16x8.min_u
  | 0xFD 152 : u32    => i16x8.max_s
  | 0xFD 153 : u32    => i16x8.max_u
  | 0xFD 155 : u32    => i16x8.avgr_u
  | 0xFD 156 : u32    => i16x8.extmul_low_s_i8x16
  | 0xFD 157 : u32    => i16x8.extmul_high_s_i8x16
  | 0xFD 158 : u32    => i16x8.extmul_low_u_i8x16
  | 0xFD 159 : u32    => i16x8.extmul_high_u_i8x16
  | 0xFD 273 : u32    => i16x8.relaxed_q15mulr_s
  | 0xFD 274 : u32    => i16x8.relaxed_dot_s_i8x16
```

```text
instr ::=
  | ...
  | 0xFD 126 : u32    => i32x4.extadd_pairwise_s_i16x8
  | 0xFD 127 : u32    => i32x4.extadd_pairwise_u_i16x8
  | 0xFD 160 : u32    => i32x4.abs
  | 0xFD 161 : u32    => i32x4.neg
  | 0xFD 163 : u32    => i32x4.all_true
  | 0xFD 164 : u32    => i32x4.bitmask
  | 0xFD 167 : u32    => i32x4.extend_low_s_i16x8
  | 0xFD 168 : u32    => i32x4.extend_high_s_i16x8
  | 0xFD 169 : u32    => i32x4.extend_low_u_i16x8
  | 0xFD 170 : u32    => i32x4.extend_high_u_i16x8
  | 0xFD 171 : u32    => i32x4.shl
  | 0xFD 172 : u32    => i32x4.shr_s
  | 0xFD 173 : u32    => i32x4.shr_u
  | 0xFD 174 : u32    => i32x4.add
  | 0xFD 177 : u32    => i32x4.sub
  | 0xFD 181 : u32    => i32x4.mul
  | 0xFD 182 : u32    => i32x4.min_s
  | 0xFD 183 : u32    => i32x4.min_u
  | 0xFD 184 : u32    => i32x4.max_s
  | 0xFD 185 : u32    => i32x4.max_u
  | 0xFD 186 : u32    => i32x4.dot_s_i16x8
  | 0xFD 188 : u32    => i32x4.extmul_low_s_i16x8
  | 0xFD 189 : u32    => i32x4.extmul_high_s_i16x8
  | 0xFD 190 : u32    => i32x4.extmul_low_u_i16x8
  | 0xFD 191 : u32    => i32x4.extmul_high_u_i16x8
  | 0xFD 275 : u32    => i32x4.relaxed_dot_add_s_i16x8
```

```text
instr ::=
  | ...
  | 0xFD 192 : u32    => i64x2.abs
  | 0xFD 193 : u32    => i64x2.neg
  | 0xFD 195 : u32    => i64x2.all_true
  | 0xFD 196 : u32    => i64x2.bitmask
  | 0xFD 199 : u32    => i64x2.extend_low_s_i32x4
  | 0xFD 200 : u32    => i64x2.extend_high_s_i32x4
  | 0xFD 201 : u32    => i64x2.extend_low_u_i32x4
  | 0xFD 202 : u32    => i64x2.extend_high_u_i32x4
  | 0xFD 203 : u32    => i64x2.shl
  | 0xFD 204 : u32    => i64x2.shr_s
  | 0xFD 205 : u32    => i64x2.shr_u
  | 0xFD 206 : u32    => i64x2.add
  | 0xFD 209 : u32    => i64x2.sub
  | 0xFD 213 : u32    => i64x2.mul
  | 0xFD 220 : u32    => i64x2.extmul_low_s_i32x4
  | 0xFD 221 : u32    => i64x2.extmul_high_s_i32x4
  | 0xFD 222 : u32    => i64x2.extmul_low_u_i32x4
  | 0xFD 223 : u32    => i64x2.extmul_high_u_i32x4
```

```text
instr ::=
  | ...
  | 0xFD 103 : u32    => f32x4.ceil
  | 0xFD 104 : u32    => f32x4.floor
  | 0xFD 105 : u32    => f32x4.trunc
  | 0xFD 106 : u32    => f32x4.nearest
  | 0xFD 224 : u32    => f32x4.abs
  | 0xFD 225 : u32    => f32x4.neg
  | 0xFD 227 : u32    => f32x4.sqrt
  | 0xFD 228 : u32    => f32x4.add
  | 0xFD 229 : u32    => f32x4.sub
  | 0xFD 230 : u32    => f32x4.mul
  | 0xFD 231 : u32    => f32x4.div
  | 0xFD 232 : u32    => f32x4.min
  | 0xFD 233 : u32    => f32x4.max
  | 0xFD 234 : u32    => f32x4.pmin
  | 0xFD 235 : u32    => f32x4.pmax
  | 0xFD 269 : u32    => f32x4.relaxed_min
  | 0xFD 270 : u32    => f32x4.relaxed_max
  | 0xFD 261 : u32    => f32x4.relaxed_madd
  | 0xFD 262 : u32    => f32x4.relaxed_nmadd
```

```text
instr ::=
  | ...
  | 0xFD 116 : u32    => f64x2.ceil
  | 0xFD 117 : u32    => f64x2.floor
  | 0xFD 122 : u32    => f64x2.trunc
  | 0xFD 148 : u32    => f64x2.nearest
  | 0xFD 236 : u32    => f64x2.abs
  | 0xFD 237 : u32    => f64x2.neg
  | 0xFD 239 : u32    => f64x2.sqrt
  | 0xFD 240 : u32    => f64x2.add
  | 0xFD 241 : u32    => f64x2.sub
  | 0xFD 242 : u32    => f64x2.mul
  | 0xFD 243 : u32    => f64x2.div
  | 0xFD 244 : u32    => f64x2.min
  | 0xFD 245 : u32    => f64x2.max
  | 0xFD 246 : u32    => f64x2.pmin
  | 0xFD 247 : u32    => f64x2.pmax
  | 0xFD 271 : u32    => f64x2.relaxed_min
  | 0xFD 272 : u32    => f64x2.relaxed_max
  | 0xFD 263 : u32    => f64x2.relaxed_madd
  | 0xFD 264 : u32    => f64x2.relaxed_nmadd
  | 0xFD 265 : u32    => i8x16.relaxed_laneselect
  | 0xFD 266 : u32    => i16x8.relaxed_laneselect
  | 0xFD 267 : u32    => i32x4.relaxed_laneselect
  | 0xFD 268 : u32    => i64x2.relaxed_laneselect
```

```text
instr ::=
  | ...
  | 0xFD 94 : u32     => f32x4.demote_zero_f64x2
  | 0xFD 95 : u32     => f64x2.promote_low_f32x4
  | 0xFD 248 : u32    => i32x4.trunc_sat_s_f32x4
  | 0xFD 249 : u32    => i32x4.trunc_sat_u_f32x4
  | 0xFD 250 : u32    => f32x4.convert_s_i32x4
  | 0xFD 251 : u32    => f32x4.convert_u_i32x4
  | 0xFD 252 : u32    => i32x4.trunc_sat_s_zero_f64x2
  | 0xFD 253 : u32    => i32x4.trunc_sat_u_zero_f64x2
  | 0xFD 254 : u32    => f64x2.convert_low_s_i32x4
  | 0xFD 255 : u32    => f64x2.convert_low_u_i32x4
  | 0xFD 257 : u32    => i32x4.relaxed_trunc_s_f32x4
  | 0xFD 258 : u32    => i32x4.relaxed_trunc_u_f32x4
  | 0xFD 259 : u32    => i32x4.relaxed_trunc_s_zero_f64x2
  | 0xFD 260 : u32    => i32x4.relaxed_trunc_u_zero_f64x2
```

### Expressions

Expressions are encoded by their instruction sequence terminated with an explicit `0x0B` opcode for `end`.

```text
expr ::=
  | (in : instr)* 0x0B    => in*
```
