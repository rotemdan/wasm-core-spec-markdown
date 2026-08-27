## Instructions

WebAssembly code consists of sequences of *instructions*. Its computational model is based on a *stack machine* in that instructions manipulate values on an implicit *operand stack*, consuming (popping) argument values and producing or returning (pushing) result values.

In addition to dynamic operands from the stack, some instructions also have static *immediate* arguments, typically indices or type annotations, which are part of the instruction itself.

Some instructions are *structured* in that they contain nested sequences of instructions.

The following sections group instructions into a number of different categories.

The syntax of instruction is further extended with additional forms for the purpose of specifying execution.

### Parametric Instructions

Instructions in this group can operate on operands of any value type.

```text
instr ::= nop
        | unreachable
        | drop
        | select (valtype*)^?
        | ...
```

The `nop` instruction does nothing.

The `unreachable` instruction causes an unconditional trap.

The `drop` instruction simply throws away a single operand.

The `select` instruction selects one of its first two operands based on whether its third operand is zero or not. It may include a value type determining the type of these operands. If missing, the operands must be of numeric or vector type.

> **Note:** In future versions of WebAssembly, the type annotation on `select` may allow for more than a single value being selected at the same time.

### Control Instructions

Instructions in this group affect the flow of control.

```text
instr ::= ...
        | block blocktype instr*
        | loop blocktype instr*
        | if blocktype instr* else instr*
        | br labelidx
        | br_if labelidx
        | br_table labelidx* labelidx
        | br_on_null labelidx
        | br_on_non_null labelidx
        | br_on_cast labelidx reftype reftype
        | br_on_cast_fail labelidx reftype reftype
        | call funcidx
        | call_ref typeuse
        | call_indirect tableidx typeuse
        | return
        | return_call funcidx
        | return_call_ref typeuse
        | return_call_indirect tableidx typeuse
        | throw tagidx
        | throw_ref
        | try_table blocktype list(catch) instr*
        | ...

catch ::= catch tagidx labelidx
        | catch_ref tagidx labelidx
        | catch_all labelidx
        | catch_all_ref labelidx
```

The `block`, `loop`, `if` and `try_table` instructions are *structured* instructions. They bracket nested sequences of instructions, called *blocks*. As the grammar prescribes, they must be well-nested.

A structured instruction can consume *input* and produce *output* on the operand stack according to its annotated block type.

Each structured control instruction introduces an implicit *label*. Labels are targets for branch instructions that reference them with label indices. Unlike with other index spaces, indexing of labels is relative by nesting depth, that is, label `0` refers to the innermost structured control instruction enclosing the referring branch instruction, while increasing indices refer to those farther out. Consequently, labels can only be referenced from *within* the associated structured control instruction. This also implies that branches can only be directed outwards, "breaking" from the block of the control construct they target. The exact effect depends on that control construct. In case of `block` or `if` it is a *forward jump*, resuming execution after the end of the block. In case of `loop` it is a *backward jump* to the beginning of the loop.

> **Note:** This enforces *structured control flow*. Intuitively, a branch targeting a `block` or `if` behaves like a `break` statement in most C-like languages, while a branch targeting a `loop` behaves like a `continue` statement.

Branch instructions come in several flavors: `br` performs an unconditional branch, `br_if` performs a conditional branch, and `br_table` performs an indirect branch through an operand indexing into the label list that is an immediate to the instruction, or to a default target if the operand is out of bounds. The `br_on_null` and `br_on_non_null` instructions check whether a reference operand is null and branch if that is the case or not the case, respectively. Similarly, `br_on_cast` and `br_on_cast_fail` attempt a downcast on a reference operand and branch if that succeeds, or fails, respectively.

The `return` instruction is a shortcut for an unconditional branch to the outermost block, which implicitly is the body of the current function. Taking a branch *unwinds* the operand stack up to the height where the targeted structured control instruction was entered. However, branches may additionally consume operands themselves, which they push back on the operand stack after unwinding. Forward branches require operands according to the output of the targeted block's type, i.e., represent the values produced by the terminated block. Backward branches require operands according to the input of the targeted block's type, i.e., represent the values consumed by the restarted block.

The `call` instruction invokes another function, consuming the necessary arguments from the stack and returning the result values of the call. The `call_ref` instruction invokes a function indirectly through a function reference operand. The `call_indirect` instruction calls a function indirectly through an operand indexing into a table that is denoted by a table index and must contain function references. Since it may contain functions of heterogeneous type, the callee is dynamically checked against the function type indexed by the instruction's second immediate, and the call is aborted with a trap if it does not match.

The `return_call`, `return_call_ref`, and `return_call_indirect` instructions are *tail-call* variants of the previous ones. That is, they first return from the current function before actually performing the respective call. It is guaranteed that no sequence of nested calls using only these instructions can cause resource exhaustion due to hitting an implementation's limit on the number of active calls.

The instructions `throw`, `throw_ref`, and `try_table` are concerned with *exceptions*. The `throw` and `throw_ref` instructions raise and reraise an exception, respectively, and transfers control to the innermost enclosing exception handler that has a matching catch clause. The `try_table` instruction installs an exception *handler* that handles exceptions as specified by its catch clauses.

### Variable Instructions

Variable instructions are concerned with access to local or global variables.

```text
instr ::= ...
        | local.get localidx
        | local.set localidx
        | local.tee localidx
        | global.get globalidx
        | global.set globalidx
        | ...
```

These instructions get or set the values of respective variables. The `local.tee` instruction is like `local.set` but also returns its argument.

### Table Instructions

Instructions in this group are concerned with tables.

```text
instr ::= ...
        | table.get tableidx
        | table.set tableidx
        | table.size tableidx
        | table.grow tableidx
        | table.fill tableidx
        | table.copy tableidx tableidx
        | table.init tableidx elemidx
        | elem.drop elemidx
        | ...
```

The `table.get` and `table.set` instructions load or store an element in a table, respectively.

The `table.size` instruction returns the current size of a table. The `table.grow` instruction grows table by a given delta and returns the previous size, or `-1` if enough space cannot be allocated. It also takes an initialization value for the newly allocated entries.

The `table.fill` instruction sets all entries in a range to a given value. The `table.copy` instruction copies elements from a source table region to a possibly overlapping destination region; the first index denotes the destination. The `table.init` instruction copies elements from a passive element segment into a table.

The `elem.drop` instruction prevents further use of a passive element segment. This instruction is intended to be used as an optimization hint. After an element segment is dropped its elements can no longer be retrieved, so the memory used by this segment may be freed.

> **Note:** An additional instruction that accesses a table is the control instruction `call_indirect`.

### Memory Instructions

Instructions in this group are concerned with linear memory.

```text
memarg ::= { align u32 , offset u64 }

loadop_{iN} ::= sz_sx   (if sz < N)

storeop_{iN} ::= sz   (if sz < N)

vloadop_{vectype} ::= sz x M_sx   (if sz * M = |vectype| / 2)
                   | sz_splat
                   | sz_zero   (if sz >= 32)

instr ::= ...
        | numtype.load loadop_{numtype}? memidx memarg
        | numtype.store storeop_{numtype}? memidx memarg
        | vectype.vload vloadop_{vectype}? memidx memarg
        | vectype.vload sz_lane memidx memarg laneidx
        | vectype.vstore memidx memarg
        | vectype.vstore sz_lane memidx memarg laneidx
        | memory.size memidx
        | memory.grow memidx
        | memory.fill memidx
        | memory.copy memidx memidx
        | memory.init memidx dataidx
        | data.drop dataidx
        | ...
```

Memory is accessed with `load` and `store` instructions for the different number types and vector types. They all take a memory index and a *memory argument* `memarg` that contains an address *offset* and the expected *alignment* (expressed as the exponent of a power of 2).

Integer loads and stores can optionally specify a *storage size* `sz` that is smaller than the bit width of the respective value type. In the case of loads, a sign extension mode `sx` is then required to select appropriate behavior.

Vector loads can specify a shape that is half the bit width of `v128`. Each lane is half its usual size, and the sign extension mode `sx` then specifies how the smaller lane is extended to the larger lane. Alternatively, vector loads can perform a *splat*, such that only a single lane of the specified storage size is loaded, and the result is duplicated to all lanes.

The static address offset is added to the dynamic address operand, yielding a 33-bit or 65-bit *effective address* that is the zero-based index at which the memory is accessed. All values are read and written in LittleEndian byte order. A trap results if any of the accessed memory bytes lies outside the address range implied by the memory's current size.

The `memory.size` instruction returns the current size of a memory. The `memory.grow` instruction grows a memory by a given delta and returns the previous size, or `-1` if enough memory cannot be allocated. Both instructions operate in units of page size.

The `memory.fill` instruction sets all values in a region of memory to a given byte. The `memory.copy` instruction copies data from a source memory region to a possibly overlapping destination region in another or the same memory; the first index denotes the destination. The `memory.init` instruction copies data from a passive data segment into a memory.

The `data.drop` instruction prevents further use of a passive data segment. This instruction is intended to be used as an optimization hint. After a data segment is dropped its data can no longer be retrieved, so the memory used by this segment may be freed.

### Reference Instructions

Instructions in this group are concerned with accessing references.

```text
instr ::= ...
        | ref.func funcidx
        | ref.null heaptype
        | ref.is_null
        | ref.as_non_null
        | ref.eq
        | ref.test reftype
        | ref.cast reftype
        | ...
```

The `ref.null` and `ref.func` instructions produce a null reference or a reference to a given function, respectively.

The instruction `ref.is_null` checks for null, while `ref.as_non_null` converts a nullable to a non-null one, and traps if it encounters null.

The `ref.eq` compares two references.

The instructions `ref.test` and `ref.cast` test the dynamic type of a reference operand. The former merely returns the result of the test, while the latter performs a downcast and traps if the operand's type does not match.

> **Note:** The `br_on_null` and `br_on_non_null` instructions provide versions of `ref.as_non_null` that branch depending on the success or failure of a null test instead of trapping. Similarly, the `br_on_cast` and `br_on_cast_fail` instructions provides versions of `ref.cast` that branch depending on the success of the downcast instead of trapping.
>
> An additional instruction operating on function references is the control instruction `call_ref`.

### Aggregate Instructions

Instructions in this group are concerned with creating and accessing references to aggregate types.

```text
instr ::= ...
        | struct.new typeidx
        | struct.new_default typeidx
        | struct.get sx? typeidx fieldidx
        | struct.set typeidx fieldidx
        | array.new typeidx
        | array.new_default typeidx
        | array.new_fixed typeidx u32
        | array.new_data typeidx dataidx
        | array.new_elem typeidx elemidx
        | array.get sx? typeidx
        | array.set typeidx
        | array.len
        | array.fill typeidx
        | array.copy typeidx typeidx
        | array.init_data typeidx dataidx
        | array.init_elem typeidx elemidx
        | ref.i31
        | i31.get sx
        | extern.convert_any
        | any.convert_extern
        | ...
```

The instructions `struct.new` and `struct.new_default` allocate a new structure, initializing them either with operands or with default values. The remaining instructions on structs access individual fields, allowing for different sign extension modes in the case of packed storage types.

Similarly, arrays can be allocated either with an explicit initialization operand or a default value. Furthermore, `array.new_fixed` allocates an array with statically fixed size, and `array.new_data` and `array.new_elem` allocate an array and initialize it from a data or element segment, respectively. The instructions `array.get`, `array.get sx`, and `array.set` access individual slots, again allowing for different sign extension modes in the case of a packed storage type; `array.len` produces the length of an array; `array.fill` fills a specified slice of an array with a given value and `array.copy`, `array.init_data`, and `array.init_elem` copy elements to a specified slice of an array from a given array, data segment, or element segment, respectively.

The instructions `ref.i31` and `i31.get sx` convert between type `i32` and an unboxed scalar.

The instructions `any.convert_extern` and `extern.convert_any` allow lossless conversion between references represented as type `(ref null extern)` and as `(ref null any)`.

### Numeric Instructions

Numeric instructions provide basic operations over numeric values of specific type. These operations closely match respective operations available in hardware.

```text
sz ::= 8 | 16 | 32 | 64

sx ::= u | s

num_{iN} ::= iN

num_{fN} ::= fN

instr ::= ...
        | numtype.const num_{numtype}
        | numtype . unop_{numtype}
        | numtype . binop_{numtype}
        | numtype . testop_{numtype}
        | numtype . relop_{numtype}
        | numtype_1 . cvtop_{numtype_2, numtype_1}_numtype_2
        | ...

unop_{iN} ::= clz | ctz | popcnt | extend_sz_s   (if sz < N)

unop_{fN} ::= abs | neg | sqrt | ceil | floor | trunc | nearest

binop_{iN} ::= add | sub | mul | div_sx | rem_sx
             | and | or | xor | shl | shr_sx | rotl | rotr

binop_{fN} ::= add | sub | mul | div | fmin | fmax | copysign

testop_{iN} ::= eqz

relop_{iN} ::= eq | ne | lt_sx | gt_sx | le_sx | ge_sx

relop_{fN} ::= eq | ne | lt | gt | le | ge

cvtop_{iN_1, iN_2} ::= extend_sx   (if N_1 < N_2)
                     | wrap   (if N_1 > N_2)

cvtop_{iN_1, fN_2} ::= convert_sx
                     | reinterpret   (if N_1 = N_2)

cvtop_{fN_1, iN_2} ::= trunc_sx
                     | trunc_sat_sx
                     | reinterpret   (if N_1 = N_2)

cvtop_{fN_1, fN_2} ::= promote   (if N_1 < N_2)
                     | demote   (if N_1 > N_2)
```

Numeric instructions are divided by number type. For each type, several subcategories can be distinguished:

* *Constants*: return a static constant.
* *Unary operations*: consume one operand and produce one result of the respective type.
* *Binary operations*: consume two operands and produce one result of the respective type.
* *Tests*: consume one operand of the respective type and produce a Boolean integer result.
* *Comparisons*: consume two operands of the respective type and produce a Boolean integer result.
* *Conversions*: consume a value of one type and produce a result of another (the source type of the conversion is the one after the "_").

Some integer instructions come in two flavors, where a signedness annotation `sx` distinguishes whether the operands are to be interpreted as unsigned or signed integers. For the other integer instructions, the use of two's complement for the signed interpretation means that they behave the same regardless of signedness.

### Vector Instructions

Vector instructions (also known as *SIMD* instructions, *single instruction multiple data*) provide basic operations over values of vector type.

```text
lanetype ::= numtype | packtype

dim ::= 1 | 2 | 4 | 8 | 16

shape ::= lanetype x dim   (if |lanetype| * dim = 128)

ishape ::= shape   (if shlanetype(shape) = iN)

bshape ::= shape   (if shlanetype(shape) = i8)

half ::= low | high

zero ::= zero

laneidx ::= u8

instr ::= ...
        | vectype.const vec_{vectype}
        | vectype . vvunop
        | vectype . vvbinop
        | vectype . vvternop
        | vectype . vvtestop
        | shape . vunop_{shape}
        | shape . vbinop_{shape}
        | shape . vternop_{shape}
        | shape . vtestop_{shape}
        | shape . vrelop_{shape}
        | ishape . vshiftop_{ishape}
        | ishape . vbitmask
        | bshape . vswizzlop_{bshape}
        | bshape . vshuffle laneidx*   (if |laneidx*| = shdim(bshape))
        | ishape_1 . vextunop_{ishape_2, ishape_1}_ishape_2
        | ishape_1 . vextbinop_{ishape_2, ishape_1}_ishape_2
        | ishape_1 . vextternop_{ishape_2, ishape_1}_ishape_2
        | ishape_1 . vnarrow_ishape_2_sx   (if |shlanetype(ishape_2)| = 2 * |shlanetype(ishape_1)| <= 32)
        | shape_1 . vcvtop_{shape_2, shape_1}_shape_2
        | shape . vsplat
        | shape . extractlane_sx? laneidx   (if sx? = ε <=> shlanetype(shape) ∈ i32 i64 f32 f64)
        | shape . replacelane laneidx
        | ...
```

Vector instructions have a naming convention involving a *shape* prefix that determines how their operands will be interpreted, written `t x N`, and consisting of a *lane type* `t`, a possibly *packed* numeric type, and its *dimension* `N`, which denotes the number of lanes of that type. Operations are performed point-wise on the values of each lane.

Instructions prefixed with `v128` do not involve a specific interpretation, and treat the `v128` as either an `i128` value or a vector of `128` individual bits.

> **Note:** For example, the shape `i32 x 4` interprets the operand as four `i32` values, packed into an `i128`. The bit width of the lane type `t` times `N` always is `128`.

```text
vvunop ::= not

vvbinop ::= and | andnot | or | xor

vvternop ::= bitselect

vvtestop ::= any_true

vunop_{iN x M} ::= abs | neg
                 | popcnt   (if N = 8)

vunop_{fN x M} ::= abs | neg | sqrt
                 | ceil | floor | trunc | nearest

vbinop_{iN x M} ::= add
                  | sub
                  | add_sat_sx   (if N <= 16)
                  | sub_sat_sx   (if N <= 16)
                  | mul   (if N >= 16)
                  | avgr_u   (if N <= 16)
                  | q15mulr_sat_s   (if N = 16)
                  | relaxed_q15mulr_s   (if N = 16)
                  | min_sx   (if N <= 32)
                  | max_sx   (if N <= 32)

vbinop_{fN x M} ::= add | sub | mul | div
                  | min | max | pmin | pmax
                  | relaxed_min | relaxed_max

vternop_{iN x M} ::= relaxed_laneselect

vternop_{fN x M} ::= relaxed_madd | relaxed_nmadd

vtestop_{iN x M} ::= all_true

vrelop_{iN x M} ::= eq | ne
                  | lt_sx   (if N != 64 || sx = s)
                  | gt_sx   (if N != 64 || sx = s)
                  | le_sx   (if N != 64 || sx = s)
                  | ge_sx   (if N != 64 || sx = s)

vrelop_{fN x M} ::= eq | ne | lt | gt | le | ge

vswizzlop_{i8 x M} ::= swizzle | relaxed_swizzle

vshiftop_{iN x M} ::= shl | shr_sx

vextunop_{iN_1 x M_1, iN_2 x M_2} ::= extadd_pairwise_sx   (if 16 <= 2 * N_1 = N_2 <= 32)

vextbinop_{iN_1 x M_1, iN_2 x M_2} ::= extmul_half_sx   (if 2 * N_1 = N_2 >= 16)
                                     | dot_s   (if 2 * N_1 = N_2 = 32)
                                     | relaxed_dot_s   (if 2 * N_1 = N_2 = 16)

vextternop_{iN_1 x M_1, iN_2 x M_2} ::= relaxed_dot_add_s   (if 4 * N_1 = N_2 = 32)

vcvtop_{iN_1 x M_1, iN_2 x M_2} ::= extend_half_sx   (if N_2 = 2 * N_1)

vcvtop_{iN_1 x M_1, fN_2 x M_2} ::= convert_half?_sx   (if N_2 = N_1 = 32 && half? = ε || N_2 = 2 * N_1 && half? = low)

vcvtop_{fN_1 x M_1, iN_2 x M_2} ::= trunc_sat_sx_zero?   (if N_1 = N_2 = 32 && zero? = ε || N_1 = 2 * N_2 && zero? = zero)
                                    | relaxed_trunc_sx_zero?   (if N_1 = N_2 = 32 && zero? = ε || N_1 = 2 * N_2 && zero? = zero)

vcvtop_{fN_1 x M_1, fN_2 x M_2} ::= demote_zero   (if N_1 = 2 * N_2)
                                    | promote_low   (if 2 * N_1 = N_2)
```

Vector instructions can be grouped into several subcategories:

* *Constants*: return a static constant.
* *Unary Operations*: consume one `v128` operand and produce one `v128` result.
* *Binary Operations*: consume two `v128` operands and produce one `v128` result.
* *Ternary Operations*: consume three `v128` operands and produce one `v128` result.
* *Tests*: consume one `v128` operand and produce a Boolean integer result.
* *Shifts*: consume a `v128` operand and an `i32` operand, producing one `v128` result.
* *Splats*: consume a value of numeric type and produce a `v128` result of a specified shape.
* *Extract lanes*: consume a `v128` operand and return the numeric value in a given lane.
* *Replace lanes*: consume a `v128` operand and a numeric value for a given lane, and produce a `v128` result.

Some vector instructions have a signedness annotation `sx` which distinguishes whether the elements in the operands are to be interpreted as unsigned or signed integers. For the other vector instructions, the use of two's complement for the signed interpretation means that they behave the same regardless of signedness.

#### Conventions

* The function `shlanetype(shape)` extracts the lane type of a shape.
* The function `shdim(shape)` extracts the dimension of a shape.
* The function `zeroop(vcvtop)` extracts the `zero` flag from a vector conversion operator, or returns `ε` if it does not contain any.
* The function `halfop(vcvtop)` extracts the `half` flag from a vector conversion operator, or returns `ε` if it does not contain any.

### Expressions

Function bodies, initialization values for globals, elements and offsets of element segments, and offsets of data segments are given as expressions, which are sequences of instructions.

```text
expr ::= instr*
```

In some places, validation restricts expressions to be *constant*, which limits the set of allowable instructions.
