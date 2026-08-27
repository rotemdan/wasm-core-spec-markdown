## Change History

Since the original release 1.0 of the WebAssembly specification, a number of proposals for extensions have been integrated. The following sections provide an overview of what has changed.

All present and future versions of WebAssembly are intended to be *backwards-compatible* with all previous versions. Concretely:

1. All syntactically well-formed (in [binary](binary) or [text](text) format) and [valid](valid) modules remain well-formed and valid with an equivalent [module type](syntax-moduletype) (or a subtype).

   > **Note:** This allows previously malformed or invalid modules to become legal, e.g., by adding new features or by relaxing typing rules.
   >
   > It also allows reclassifying previously malformed modules as well-formed but invalid, or vice versa.
   >
   > And it allows refining the typing of [imports](syntax-import) and [exports](syntax-export), such that previously unlinkable modules become linkable.
   >
   > Historically, minor breaking changes to the *text format* have been allowed that turned previously possible valid modules invalid, as long as they were unlikely to occur in practice.

2. All non-[trapping](trap) [executions](exec) of a valid program retain their behaviour with an equivalent set of possible [results](syntax-result) (or a non-empty subset).

   > **Note:** This allows previously malformed or invalid programs to become executable.
   >
   > It also allows program executions that previously trapped to execute successfully, although the intention is to only exercise this where the possibility of such an extension has been previously noted.
   >
   > And it allows reducing the set of observable behaviours of a program execution, e.g., by reducing non-determinism.
   >
   > In a program linking prior modules with modules using new features, a prior module may encounter new behaviours, e.g., new forms of control flow or side effects when calling into a latter module.

In addition, future versions of WebAssembly will not allocate the [opcode](binary-instr) `0xFF` to represent an instruction or instruction prefix.

### Release 2.0

#### Sign Extension Instructions

Added new numeric instructions for performing sign extension within integer representations. [^proposal-signext]

* New [numeric instructions](syntax-instr-numeric):
  * `i{nn}.extend{N}_s`

#### Non-trapping Float-to-Int Conversions

Added new conversion instructions that avoid trapping when converting a floating-point number to an integer. [^proposal-cvtsat]

* New [numeric instructions](syntax-instr-numeric):
  * `i{nn}.trunc_sat_f{mm}_{sx}`

#### Multiple Values

Generalized the result type of blocks and functions to allow for multiple values; in addition, introduced the ability to have block parameters. [^proposal-multivalue]

* [Function types](syntax-functype) allow more than one result
* [Block types](syntax-blocktype) can be arbitrary function types

#### Reference Types

Added FUNCREF and EXTERNREF as new value types and respective instructions. [^proposal-reftype]

* New [reference](syntax-reftype) [value types](syntax-valtype):
  * FUNCREF
  * EXTERNREF
* New [reference instructions](syntax-instr-ref):
  * REFNULL
  * REFFUNC
  * REFISNULL
* Extended [parametric instruction](syntax-instr-parametric):
  * SELECT with optional type immediate
* New [declarative](syntax-elemmode) form of [element segment](syntax-elem)

#### Table Instructions

Added instructions to directly access and modify tables. [^proposal-reftype]

* [Table types](syntax-tabletype) allow any [reference type](syntax-reftype) as element type
* New [table instructions](syntax-instr-table):
  * TABLEGET
  * TABLESET
  * TABLESIZE
  * TABLEGROW

#### Multiple Tables

Added the ability to use multiple tables per module. [^proposal-reftype]

* [Modules](syntax-module) may
  * [define](syntax-table) multiple tables
  * [import](syntax-import) multiple tables
  * [export](syntax-export) multiple tables
* [Table instructions](syntax-instr-table) take a [table index](syntax-tableidx) immediate:
  * TABLEGET
  * TABLESET
  * TABLESIZE
  * TABLEGROW
  * CALLINDIRECT
* [Element segments](syntax-elem) take a [table index](syntax-tableidx)

#### Bulk Memory and Table Instructions

Added instructions that modify ranges of memory or table entries. [^proposal-reftype] [^proposal-bulk]

* New [memory instructions](syntax-instr-memory):
  * MEMORYFILL
  * MEMORYINIT
  * MEMORYCOPY
  * DATADROP
* New [table instructions](syntax-instr-table):
  * TABLEFILL
  * TABLEINIT
  * TABLECOPY
  * ELEMDROP
* New [passive](syntax-datamode) form of [data segment](syntax-data)
* New [passive](syntax-elemmode) form of [element segment](syntax-elem)
* New [data count section](binary-datacntsec) in binary format
* Active data and element segments boundaries are no longer checked at compile time but may trap instead

#### Vector Instructions

Added vector type and instructions that manipulate multiple numeric values in parallel (also known as *SIMD*, single instruction multiple data) [^proposal-vectype]

* New [value type](syntax-valtype):
  * v128
* New [memory instructions](syntax-instr-memory):
  * `v128.load`
  * `v128.loadNxM_sx`
  * `v128.loadN_zero`
  * `v128.loadN_splat`
  * `v128.loadN_lane`
  * `v128.store`
  * `v128.storeN_lane`
* New constant [vector instruction](syntax-instr-vec):
  * `v128.const`
* New unary [vector instructions](syntax-instr-vec):
  * `v128.not`
  * `iNxM.abs`
  * `iNxM.neg`
  * `i8x16.popcnt`
  * `fNxM.abs`
  * `fNxM.neg`
  * `fNxM.sqrt`
  * `fNxM.ceil`
  * `fNxM.floor`
  * `fNxM.trunc`
  * `fNxM.nearest`
* New binary [vector instructions](syntax-instr-vec):
  * `v128.and`
  * `v128.andnot`
  * `v128.or`
  * `v128.xor`
  * `iNxM.add`
  * `iNxM.sub`
  * `iNxM.mul`
  * `iNxM.add_sat_sx`
  * `iNxM.sub_sat_sx`
  * `iNxM.min_sx`
  * `iNxM.max_sx`
  * `iNxM.shl`
  * `iNxM.shr_sx`
  * `fNxM.add`
  * `fNxM.sub`
  * `fNxM.mul`
  * `fNxM.div`
  * `i16x8.extadd_pairwise_i8x16_sx`
  * `i32x4.extadd_pairwise_i16x8_sx`
  * `iNxM.extmul_half_iN'xM'_sx`
  * `i16x8.q15mulr_sat_s`
  * `i32x4.dot_i16x8_s`
  * `i8x16.avgr_u`
  * `i16x8.avgr_u`
  * `fNxM.min`
  * `fNxM.max`
  * `fNxM.pmin`
  * `fNxM.pmax`
* New ternary [vector instruction](syntax-instr-vec):
  * `v128.bitselect`
* New test [vector instructions](syntax-instr-vec):
  * `v128.any_true`
  * `iNxM.all_true`
* New relational [vector instructions](syntax-instr-vec):
  * `iNxM.eq`
  * `iNxM.ne`
  * `iNxM.lt_sx`
  * `iNxM.gt_sx`
  * `iNxM.le_sx`
  * `iNxM.ge_sx`
  * `fNxM.eq`
  * `fNxM.ne`
  * `fNxM.lt`
  * `fNxM.gt`
  * `fNxM.le`
  * `fNxM.ge`
* New conversion [vector instructions](syntax-instr-vec):
  * `i32x4.trunc_sat_f32x4_sx`
  * `i32x4.trunc_sat_f64x2_sx_zero`
  * `f32x4.convert_i32x4_sx`
  * `f32x4.demote_f64x2_zero`
  * `f64x2.convert_low_i32x4_sx`
  * `f64x2.promote_low_f32x4`
* New lane access [vector instructions](syntax-instr-vec):
  * `iNxM.extract_lane_sx?`
  * `iNxM.replace_lane`
  * `fNxM.extract_lane`
  * `fNxM.replace_lane`
* New lane splitting/combining [vector instructions](syntax-instr-vec):
  * `iNxM.extend_half_iN'xM'_sx`
  * `i8x16.narrow_i16x8_sx`
  * `i16x8.narrow_i32x4_sx`
* New byte reordering [vector instructions](syntax-instr-vec):
  * `i8x16.shuffle`
  * `i8x16.swizzle`
* New injection/projection [vector instructions](syntax-instr-vec):
  * `iNxM.splat`
  * `fNxM.splat`
  * `iNxM.bitmask`

### Release 3.0

#### Extended Constant Expressions

Allowed basic numeric computations in constant expressions. [^proposal-extconst]

* Extended set of [constant instructions](valid-constant) with:
  * `i{nn}.add`
  * `i{nn}.sub`
  * `i{nn}.mul`
  * GLOBALGET for any previously declared immutable [global](syntax-global)

> **Note:** The [garbage collection](extension-gc) extension added further constant instructions.

#### Tail Calls

Added instructions to perform tail calls. [^proposal-tailcall]

* New [control instructions](syntax-instr-control):
  * RETURNCALL
  * RETURNCALLINDIRECT

#### Exception Handling

Added tag definitions, imports, and exports, and instructions to throw and catch exceptions. [^proposal-exn]

* [Modules](syntax-module) may
  * [define](syntax-tag) tags
  * [import](syntax-import) tags
  * [export](syntax-export) tags
* New [heap types](syntax-heaptype):
  * exn
  * noexn
* New [reference type](syntax-reftype) short-hands:
  * exnref
  * nullexnref
* New [control instructions](syntax-instr-control):
  * throw
  * throw_ref
  * try_table
* New [tag section](binary-tagsec) in binary format.

#### Multiple Memories

Added the ability to use multiple memories per module. [^proposal-multimem]

* [Modules](syntax-module) may
  * [define](syntax-mem) multiple memories
  * [import](syntax-import) multiple memories
  * [export](syntax-export) multiple memories
* [Memory instructions](syntax-instr-memory) take a [memory index](syntax-memidx) immediate:
  * MEMORYSIZE
  * MEMORYGROW
  * MEMORYFILL
  * MEMORYCOPY
  * MEMORYINIT
  * `t.load`
  * `t.store`
  * `t.loadN_sx`
  * `t.storeN`
  * `v128.loadNxM_sx`
  * `v128.loadN_zero`
  * `v128.loadN_splat`
  * `v128.loadN_lane`
  * `v128.storeN_lane`
* [Data segments](syntax-elem) take a [memory index](syntax-memidx)

#### 64-bit Address Space

Added the ability to declare an `i64` [address type](syntax-addrtype) for [tables](syntax-tabletype) and [memories](syntax-memtype). [^proposal-addr64]

* [Address types](syntax-addrtype) denote a subset of the integral [number types](syntax-numtype)
* [Table types](syntax-tabletype) include an [address type](syntax-addrtype)
* [Memory types](syntax-memtype) include an [address type](syntax-addrtype)
* Operand types of [table](syntax-instr-table) and [memory](syntax-instr-memory) instructions now depend on the subject's declared address type:
  * TABLEGET
  * TABLESET
  * TABLESIZE
  * TABLEGROW
  * TABLEFILL
  * TABLECOPY
  * TABLEINIT
  * MEMORYSIZE
  * MEMORYGROW
  * MEMORYFILL
  * MEMORYCOPY
  * MEMORYINIT
  * `t.load`
  * `t.store`
  * `t.loadN_sx`
  * `t.storeN`
  * `v128.loadNxM_sx`
  * `v128.loadN_zero`
  * `v128.loadN_splat`
  * `v128.loadN_lane`
  * `v128.storeN_lane`

#### Typeful References

Added more precise types for references. [^proposal-typedref]

* New generalised form of [reference types](syntax-reftype):
  * `(ref null? heaptype)`
* New class of [heap types](syntax-heaptype):
  * func
  * extern
  * `typeidx`
* Basic [subtyping](match) on [reference](match-reftype) and [value](match-valtype) types
* New [reference instructions](syntax-instr-ref):
  * REFASNONNULL
  * BRONNULL
  * BRONNONNULL
* New [control instruction](syntax-instr-control):
  * CALLREF
* Refined typing of [reference instruction](syntax-instr-ref):
  * REFFUNC with more precise result type
* Refined typing of [local instructions](valid-instr-variable) and [instruction sequences](valid-instrs) to track the [initialization status](syntax-init) of [locals](syntax-local) with non-defaultable type
* Refined decoding of [active](syntax-elemmode) [element segments](binary-elem) with implicit element type and plain function indices (opcode `0`) to produce [non-null](syntax-null) [reference type](syntax-reftype)
* Extended [table definitions](syntax-table) with optional initializer expression

#### Garbage Collection

Added managed reference types. [^proposal-gc]

* New forms of [heap types](syntax-heaptype):
  * any
  * eq
  * i31
  * struct
  * array
  * none
  * nofunc
  * noextern
* New [reference type](syntax-reftype) short-hands:
  * anyref
  * eqref
  * i31ref
  * structref
  * arrayref
  * nullref
  * nullfuncref
  * nullexternref
* New forms of type definitions:
  * [structure](syntax-structtype)
  * [array types](syntax-arraytype)
  * [sub types](syntax-subtype)
  * [recursive types](syntax-rectype)
* Enriched [subtyping](match) based on explicitly declared [sub types](syntax-subtype) and the new heap types
* New generic [reference instructions](syntax-instr-ref):
  * REFEQ
  * REFTEST
  * REFCAST
  * BRONCAST
  * BRONCASTFAIL
* New [reference instructions](syntax-instr-ref) for [unboxed scalars](syntax-i31):
  * REFI31
  * `i31get_sx`
* New [reference instructions](syntax-instr-ref) for [structure types](syntax-structtype):
  * STRUCTNEW
  * STRUCTNEWDEFAULT
  * `structget_sx?`
  * STRUCTSET
* New [reference instructions](syntax-instr-ref) for [array types](syntax-structtype):
  * ARRAYNEW
  * ARRAYNEWDEFAULT
  * ARRAYNEWFIXED
  * ARRAYNEWDATA
  * ARRAYNEWELEM
  * `arrayget_sx?`
  * ARRAYSET
  * ARRAYLEN
  * ARRAYFILL
  * ARRAYCOPY
  * ARRAYINITDATA
  * ARRAYINITELEM
* New [reference instructions](syntax-instr-ref) for converting [external types](syntax-externtype):
  * ANYCONVERTEXTERN
  * EXTERNCONVERTANY
* Extended set of [constant instructions](valid-constant) with:
  * REFI31
  * STRUCTNEW
  * STRUCTNEWDEFAULT
  * ARRAYNEW
  * ARRAYNEWDEFAULT
  * ARRAYNEWFIXED
  * ANYCONVERTEXTERN
  * EXTERNCONVERTANY

#### Relaxed Vector Instructions

Added new *relaxed* vector instructions, whose behaviour is non-deterministic and implementation-dependent. [^proposal-relaxed]

* New binary [vector instruction](syntax-instr-vec-relaxed):
  * `fNxM.relaxed_min`
  * `fNxM.relaxed_max`
  * `i16x8.relaxed_q15mulr_s`
  * `i16x8.relaxed_dot_i8x16_i7x16_s`
* New ternary [vector instruction](syntax-instr-vec-relaxed):
  * `fNxM.relaxed_madd`
  * `fNxM.relaxed_nmadd`
  * `iNxM.relaxed_laneselect`
  * `i32x4.relaxed_dot_i8x16_i7x16_add_s`
* New conversion [vector instructions](syntax-instr-vec-relaxed):
  * `i32x4.relaxed_trunc_f32x4_sx`
  * `i32x4.relaxed_trunc_f64x2_sx_zero`
* New byte reordering [vector instruction](syntax-instr-vec-relaxed):
  * `i8x16.relaxed_swizzle`

#### Profiles

Introduced the concept of [profile](profiles) for specifying language subsets.

* A new profile defining a [deterministic](profile-deterministic) mode of execution.

#### Custom Annotations

Added generic syntax for custom annotations in the text format, mirroring the role of custom sections in the binary format. [^proposal-annot]

* [Annotations](text-annot) of the form `(@id ...)` are allowed anywhere in the [text format](text)
* [Identifiers](text-id) can be escaped as `@"..."` with arbitrary [names](text-name)
* Defined [name annotations](text-nameannot) `(@name "...")` for:
  * [module names](text-modulenameannot)
  * [type names](text-typenameannot)
  * [function names](text-funcnameannot)
  * [local names](text-localnameannot)
  * [field names](text-fieldnameannot)
* Defined [custom annotation](text-customannot) `(@custom "...")` to represent arbitrary [custom sections](binary-customsec) in the text format

[^proposal-signext]: https://github.com/WebAssembly/spec/tree/main/proposals/sign-extension-ops/

[^proposal-cvtsat]: https://github.com/WebAssembly/spec/tree/main/proposals/nontrapping-float-to-int-conversion/

[^proposal-multivalue]: https://github.com/WebAssembly/spec/tree/main/proposals/multi-value/

[^proposal-reftype]: https://github.com/WebAssembly/spec/tree/main/proposals/reference-types/

[^proposal-bulk]: https://github.com/WebAssembly/spec/tree/main/proposals/bulk-memory-operations/

[^proposal-vectype]: https://github.com/WebAssembly/spec/tree/main/proposals/simd/

[^proposal-extconst]: https://github.com/WebAssembly/spec/tree/main/proposals/extended-const/

[^proposal-tailcall]: https://github.com/WebAssembly/spec/tree/main/proposals/tail-call/

[^proposal-exn]: https://github.com/WebAssembly/spec/tree/main/proposals/exception-handling/

[^proposal-multimem]: https://github.com/WebAssembly/spec/tree/main/proposals/multi-memory/

[^proposal-addr64]: https://github.com/WebAssembly/spec/tree/main/proposals/memory64/

[^proposal-typedref]: https://github.com/WebAssembly/spec/tree/main/proposals/function-references/

[^proposal-gc]: https://github.com/WebAssembly/spec/tree/main/proposals/gc/

[^proposal-relaxed]: https://github.com/WebAssembly/spec/tree/main/proposals/relaxed-simd/

[^proposal-annot]: https://github.com/WebAssembly/spec/tree/main/proposals/annotations/
