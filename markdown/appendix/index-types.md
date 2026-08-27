## Index of Types

| Category | Constructor | Binary Opcode |
|----------|-------------|---------------|
| [Type index](syntax-typeidx) | `x` | (positive number as Bs32 or Bu32) |
| [Number type](syntax-numtype) | `I32` | `0x7F` (-1 as Bs7) |
| [Number type](syntax-numtype) | `I64` | `0x7E` (-2 as Bs7) |
| [Number type](syntax-numtype) | `F32` | `0x7D` (-3 as Bs7) |
| [Number type](syntax-numtype) | `F64` | `0x7C` (-4 as Bs7) |
| [Vector type](syntax-vectype) | `V128` | `0x7B` (-5 as Bs7) |
| (reserved) | | `0x7A` .. `0x79` |
| [Packed type](syntax-packtype) | `I8` | `0x78` (-8 as Bs7) |
| [Packed type](syntax-packtype) | `I16` | `0x77` (-9 as Bs7) |
| (reserved) | | `0x78` .. `0x75` |
| [Heap type](syntax-heaptype) | `NOEXN` | `0x74` (-12 as Bs7) |
| [Heap type](syntax-heaptype) | `NOFUNC` | `0x73` (-13 as Bs7) |
| [Heap type](syntax-heaptype) | `NOEXTERN` | `0x72` (-14 as Bs7) |
| [Heap type](syntax-heaptype) | `NONE` | `0x71` (-15 as Bs7) |
| [Heap type](syntax-heaptype) | `FUNC` | `0x70` (-16 as Bs7) |
| [Heap type](syntax-heaptype) | `EXTERN` | `0x6F` (-17 as Bs7) |
| [Heap type](syntax-heaptype) | `ANY` | `0x6E` (-18 as Bs7) |
| [Heap type](syntax-heaptype) | `EQT` | `0x6D` (-19 as Bs7) |
| [Heap type](syntax-heaptype) | `I31` | `0x6C` (-20 as Bs7) |
| [Heap type](syntax-heaptype) | `STRUCT` | `0x6B` (-21 as Bs7) |
| [Heap type](syntax-heaptype) | `ARRAY` | `0x6A` (-22 as Bs7) |
| [Heap type](syntax-heaptype) | `EXN` | `0x69` (-23 as Bs7) |
| (reserved) | | `0x68` .. `0x65` |
| [Reference type](syntax-reftype) | `REF` | `0x64` (-28 as Bs7) |
| [Reference type](syntax-reftype) | `REF NULL` | `0x63` (-29 as Bs7) |
| (reserved) | | `0x62` .. `0x61` |
| [Composite type](syntax-comptype) | `func [valtype*] -> [valtype*]` | `0x60` (-32 as Bs7) |
| [Composite type](syntax-comptype) | `struct fieldtype*` | `0x5F` (-33 as Bs7) |
| [Composite type](syntax-comptype) | `array fieldtype` | `0x5E` (-34 as Bs7) |
| (reserved) | | `0x5D` .. `0x51` |
| [Sub type](syntax-subtype) | `sub typeidx* comptype` | `0x50` (-48 as Bs7) |
| [Sub type](syntax-subtype) | `sub final typeidx* comptype` | `0x4F` (-49 as Bs7) |
| [Recursive type](syntax-rectype) | `rec subtype*` | `0x4E` (-50 as Bs7) |
| (reserved) | | `0x4D` .. `0x41` |
| [Result type](syntax-resulttype) | `[ε]` | `0x40` (-64 as Bs7) |
| [Tag type](syntax-tagtype) | `typeuse` | (none) |
| [Global type](syntax-globaltype) | `mut valtype` | (none) |
| [Memory type](syntax-memtype) | `addrtype limits` | (none) |
| [Table type](syntax-tabletype) | `addrtype limits reftype` | (none) |
