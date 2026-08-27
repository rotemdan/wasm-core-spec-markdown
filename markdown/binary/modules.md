## Modules

The binary encoding of modules is organized into *sections*.

Most sections correspond to one component of a module record, except that function definitions are split into two sections, separating their type declarations in the function section from their bodies in the code section.

> **Note:** This separation enables *parallel* and *streaming* compilation of the functions in a module.

### Indices

All basic indices are encoded with their respective value.

```text
typeidx   ::= x : u32    => x

funcidx   ::= x : u32    => x

tableidx  ::= x : u32    => x

memidx    ::= x : u32    => x

globalidx ::= x : u32    => x

tagidx    ::= x : u32    => x

elemidx   ::= x : u32    => x

dataidx   ::= x : u32    => x

localidx  ::= x : u32    => x

fieldidx  ::= x : u32    => x

labelidx  ::= l : u32    => l
```

External indices are encoded by a distinguishing byte followed by an encoding of their respective value.

```text
externidx ::=
  | 0x00 x : funcidx      => func x
  | 0x01 x : tableidx     => table x
  | 0x02 x : memidx       => mem x
  | 0x03 x : globalidx    => global x
  | 0x04 x : tagidx       => tag x
```

### Sections

Each section consists of

* a one-byte section *id*,
* the `u32` *length* of the contents, in bytes,
* the actual *contents*, whose structure is dependent on the section id.

Every section is optional; an omitted section is equivalent to the section being present with empty contents.

The following parameterized grammar rule defines the generic structure of a section with id `N` and contents described by the grammar `X`.

```text
section_N(X) ::=
  | N : byte  len : u32  en* : X    => en*    (if len = ||X||)
  | ε                               => ε
```

For most sections, the contents `X` encodes a list.
In these cases, the empty result `ε` is interpreted as the empty list.

> **Note:** Other than for unknown custom sections, the `size` is not required for decoding, but can be used to skip sections when navigating through a binary. The module is malformed if the size does not match the length of the binary contents `X`.

The following section ids are used:

| Id | Section |
|----|---------|
| 0  | custom section |
| 1  | type section |
| 2  | import section |
| 3  | function section |
| 4  | table section |
| 5  | memory section |
| 6  | global section |
| 7  | export section |
| 8  | start section |
| 9  | element section |
| 10 | code section |
| 11 | data section |
| 12 | data count section |
| 13 | tag section |

> **Note:** Section ids do not always correspond to the order of sections in the encoding of a module.

### Custom Section

*Custom sections* have the id 0.

They are intended to be used for debugging information or third-party extensions, and are ignored by the WebAssembly semantics.

Their contents consist of a name further identifying the custom section, followed by an uninterpreted sequence of bytes for custom use.

```text
customsec ::=
  | section_0(custom)

custom ::=
  | name byte*
```

> **Note:** If an implementation interprets the data of a custom section, then errors in that data, or the placement of the section, must not invalidate the module.

### Type Section

The *type section* has the id 1.

It decodes into the list of recursive types of a module.

```text
typesec ::=
  | ty* : section_1(list(type))    => ty*

type ::=
  | qt : rectype    => type qt
```

### Import Section

The *import section* has the id 2.

It decodes into the list of imports of a module.

```text
importsec ::=
  | im* : section_2(list(import))    => im*

import ::=
  | nm1 : name  nm2 : name  xt : externtype    => import nm1 nm2 xt
```

### Function Section

The *function section* has the id 3.

It decodes into a list of type indices that classify the functions defined by a module. The bodies of the respective functions are encoded separately in the code section.

```text
funcsec ::=
  | x* : section_3(list(typeidx))    => x*
```

### Table Section

The *table section* has the id 4.

It decodes into the list of tables defined by a module.

```text
tablesec ::=
  | tab* : section_4(list(table))    => tab*

table ::=
  | tt : tabletype                       => table tt (ref.null ht)    (if tt = at lim (ref null? ht))
  | 0x40 0x00 tt : tabletype e : expr    => table tt e
```

> **Note:** The encoding of a table type cannot start with byte `0x40`, hence decoding is unambiguous.The zero byte following it is reserved for future extensions.

### Memory Section

The *memory section* has the id 5.

It decodes into the list of memories defined by a module.

```text
memsec ::=
  | mem* : section_5(list(mem))    => mem*

mem ::=
  | mt : memtype    => memory mt
```

### Global Section

The *global section* has the id 6.

It decodes into the list of globals defined by a module.

```text
globalsec ::=
  | glob* : section_6(list(global))    => glob*

global ::=
  | gt : globaltype e : expr    => global gt e
```

### Export Section

The *export section* has the id 7.

It decodes into the list of exports of a module.

```text
exportsec ::=
  | ex* : section_7(list(export))    => ex*

export ::=
  | nm : name xx : externidx    => export nm xx
```

### Start Section

The *start section* has the id 8.

It decodes into the optional start function of a module.

```text
startsec ::=
  | start? : section_8(start)    => start?

start ::=
  | x : funcidx    => (start x)
```

### Element Section

The *element section* has the id 9.

It decodes into the list of element segments defined by a module.

```text
elemkind ::=
  | 0x00    => ref func

elem ::=
  | 0 : u32 e_o : expr y* : list(funcidx)               => elem (ref func) (ref.func y)* (active 0 e_o)
  | 1 : u32 rt : elemkind y* : list(funcidx)            => elem rt (ref.func y)* passive
  | 2 : u32 x : tableidx e : expr rt : elemkind y* : list(funcidx)  => elem rt (ref.func y)* (active x e)
  | 3 : u32 rt : elemkind y* : list(funcidx)            => elem rt (ref.func y)* declare
  | 4 : u32 e_o : expr e* : list(expr)                  => elem (ref null func) e* (active 0 e_o)
  | 5 : u32 rt : reftype e* : list(expr)                => elem rt e* passive
  | 6 : u32 x : tableidx e_o : expr rt : reftype e* : list(expr)  => elem rt e* (active x e_o)
  | 7 : u32 rt : reftype e* : list(expr)                => elem rt e* declare
```

> **Note:** The initial integer can be interpreted as a bitfield. Bit 0 distinguishes a passive or declarative segment from an active segment, bit 1 indicates the presence of an explicit table index for an active segment and otherwise distinguishes passive from declarative segments, bit 2 indicates the use of element type and element expressions instead of element kind and element indices.
>
> Additional element kinds may be added in future versions of WebAssembly.

### Code Section

The *code section* has the id 10.

It decodes into the list of *code* entries that are pairs of lists of locals and expressions. They represent the body of the functions defined by a module.

The types of the respective functions are encoded separately in the function section.

The encoding of each code entry consists of

* the `u32` *length* of the function code in bytes,
* the actual *function code*, which in turn consists of
  * the declaration of *locals*,
  * the function *body* as an expression.

Local declarations are compressed into a list whose entries consist of

* a `u32` *count*,
* a value type,

denoting *count* locals of the same value type.

```text
codesec ::=
  | code* : section_10(list(code))    => code*

code ::=
  | len : u32 code : func    => code    (if len = ||func||)

func ::=
  | loc** : list(locals) e : expr    => (⋆ loc**, e)    (if |⋆ loc**| < 2^32)

locals ::=
  | n : u32 t : valtype    => (local t)^n
```

Here, `code` ranges over pairs `(local*, expr)`.

Any code for which the length of the resulting sequence is out of bounds of the maximum size of a list is malformed.

> **Note:** Like with sections, the `size` is not needed for decoding, but can be used to skip functions when navigating through a binary. The module is malformed if a size does not match the length of the respective function code.

### Data Section

The *data section* has the id 11.

It decodes into the list of data segments defined by a module.

```text
datasec ::=
  | data* : section_11(list(data))    => data*

data ::=
  | 0 : u32 e : expr b* : list(byte)            => data b* (active 0 e)
  | 1 : u32 b* : list(byte)                     => data b* passive
  | 2 : u32 x : memidx e : expr b* : list(byte)    => data b* (active x e)
```

> **Note:** The initial integer can be interpreted as a bitfield. Bit 0 indicates a passive segment, bit 1 indicates the presence of an explicit memory index for an active segment.

### Data Count Section

The *data count section* has the id 12.

It decodes into an optional `u32` count that represents the number of data segments in the data section.

If this count does not match the length of the data segment list, the module is malformed.

```text
datacntsec ::=
  | n? : section_12(datacnt)    => n?

datacnt ::=
  | n : u32    => n
```

> **Note:** The data count section is used to simplify single-pass validation. Since the data section occurs after the code section, the `memory.init` and `data.drop` instructions would not be able to check whether the data segment index is valid until the data section is read. The data count section occurs before the code section, so a single-pass validator can use this count instead of deferring validation.

### Tag Section

The *tag section* has the id 13.

It decodes into the list of tags defined by a module.

```text
tagsec ::=
  | tag* : section_13(list(tag))    => tag*

tag ::=
  | jt : tagtype    => tag jt
```

### Modules

The encoding of a module starts with a preamble containing a 4-byte magic number (the string `\0asm`) and a version field.

The current version of the WebAssembly binary format is 1.

The preamble is followed by a sequence of sections.

Custom sections may be inserted at any place in this sequence, while other sections must occur at most once and in the prescribed order. All sections can be empty.

The lengths of lists produced by the (possibly empty) function and code section must match up.

Similarly, the optional data count must match the length of the data segment list. Furthermore, it must be present if any data index occurs in the code section.

```text
magic ::=
  | 0x00 0x61 0x73 0x6D

version ::=
  | 0x01 0x00 0x00 0x00

module ::=
  magic version
  customsec* type* : typesec
  customsec* import* : importsec
  customsec* typeidx* : funcsec
  customsec* table* : tablesec
  customsec* mem* : memsec
  customsec* tag* : tagsec
  customsec* global* : globalsec
  customsec* export* : exportsec
  customsec* start? : startsec
  customsec* elem* : elemsec
  customsec* n? : datacntsec
  customsec* (local*, expr)* : codesec
  customsec* data* : datasec
  customsec*
  => module type* import* tag* global* mem* table* func* data* elem* start? export*
       (if (n = |data*|)?
            ∧ (n? ≠ ε ∨ freedataidx(func*) = ε)
            ∧ (func = func typeidx local* expr)*)
```

> **Note:** The version of the WebAssembly binary format may increase in the future if backward-incompatible changes have to be made to the format. However, such changes are expected to occur very infrequently, if ever. The binary format is intended to be extensible, such that future features can be added without incrementing its version.
