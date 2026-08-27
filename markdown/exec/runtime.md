## Runtime Structure

Store, stack, and other runtime structure forming the WebAssembly abstract machine, such as values or module instances, are made precise in terms of additional auxiliary syntax.

### Values

WebAssembly computations manipulate *values* of either the four basic number types, i.e., integers and floating-point data of 32 or 64 bit width each, or vectors of 128 bit width, or of reference type.

In most places of the semantics, values of different types can occur.

In order to avoid ambiguities, values are therefore represented with an abstract syntax that makes their type explicit.

It is convenient to reuse the same notation as for the `const` instructions and `ref.null` producing them.

References other than null are represented with additional administrative instructions.

They either are *scalar references*, containing a 31-bit integer, *null references*, *structure references*, pointing to a specific structure address, *array references*, pointing to a specific array address, *function references*, pointing to a specific function address, *exception references*, pointing to a specific exception address, or *host references* pointing to an uninterpreted form of host address defined by the embedder.

Any of the aforementioned references can furthermore be wrapped up as an *external reference*.

```text
val ::= num | vec | reff

num ::= numtype.const num_numtype

vec ::= vectype.vconst vec_vectype

reff ::= ref.i31 u31
  | ref.null
  | ref.struct structaddr
  | ref.array arrayaddr
  | ref.func funcaddr
  | ref.exn exnaddr
  | ref.host hostaddr
  | ref.extern reff
```

> **Note:** Future versions of WebAssembly may add additional forms of values.

Value types can have an associated *default value*; it is the respective value `0` for number types, `0` for vector types, and null for nullable reference types.

For other references, no default value is defined, `default_t` hence is an optional value `val^?`.

```text
default_iN = (iN.const 0)
default_fN = (fN.const +0)
default_vN = (vN.vconst 0)
default_(ref null ht) = ref.null
default_(ref ht) = ε
```

#### Convention

* The meta variable `r` ranges over reference values where clear from context.

### Results

A *result* is the outcome of a computation.

It is either a sequence of values, a thrown exception, or a trap.

```text
result ::= val* | (ref.exn exnaddr) throwref | trap
```

### Store

The *store* represents all global state that can be manipulated by WebAssembly programs.

It consists of the runtime representation of all instances of functions, tables, memories, globals, tags, element segments, data segments, and structures, arrays or exceptions that have been allocated during the life time of the abstract machine.

It is an invariant of the semantics that no element or data instance is addressed from anywhere else but the owning module instances.

Syntactically, the store is defined as a record listing the existing instances of each category:

```text
store ::=
  { TAGS taginst*
    GLOBALS globalinst*
    MEMS meminst*
    TABLES tableinst*
    FUNCS funcinst*
    DATAS datainst*
    ELEMS eleminst*
    STRUCTS structinst*
    ARRAYS arrayinst*
    EXNS exninst* }
```

> **Note:** In practice, implementations may apply techniques like garbage collection or reference counting to remove objects from the store that are no longer referenced.
>
> However, such techniques are not semantically observable, and hence outside the scope of this specification.

#### Convention

* The meta variable `s` ranges over stores where clear from context.

### Addresses

Function instances, table instances, memory instances, global instances, tag instances, element instances, data instances and structure, array or exception instances in the store are referenced with abstract *addresses*.

These are simply indices into the respective store component.

In addition, an embedder may supply an uninterpreted set of *host addresses*.

```text
addr ::= 0 | 1 | 2 | ...

funcaddr ::= addr

tableaddr ::= addr

memaddr ::= addr

globaladdr ::= addr

tagaddr ::= addr

elemaddr ::= addr

dataaddr ::= addr

structaddr ::= addr

arrayaddr ::= addr

exnaddr ::= addr

hostaddr ::= addr
```

An embedder may assign identity to exported store objects corresponding to their addresses, even where this identity is not observable from within WebAssembly code itself (such as for function instances or immutable globals).

> **Note:** Addresses are *dynamic*, globally unique references to runtime objects, in contrast to indices, which are *static*, module-local references to their original definitions.
>
> A *memory address* `memaddr` denotes the abstract address *of* a memory *instance* in the store, not an offset *inside* a memory instance.
>
> There is no specific limit on the number of allocations of store objects, hence logical addresses can be arbitrarily large natural numbers.

#### Conventions

* The notation `addr(A)` denotes the set of addresses from address space `addr` occurring free in `A`. We sometimes reinterpret this set as the list of its elements, without assuming any particular order.

### External Addresses

An *external address* is the runtime address of an entity that can be imported or exported.
It is an address denoting either a function instance, global instance, table instance, memory instance, or tag instance in the shared store.

```text
externaddr ::= tag tagaddr | global globaladdr | mem memaddr | table tableaddr | func funcaddr
```

### Module Instances

A *module instance* is the runtime representation of a module.

It is created by instantiating a module, and collects runtime representations of all entities that are imported, defined, or exported by the module.

```text
moduleinst ::=
  { ITYPES deftype*
    ITAGS tagaddr*
    IGLOBALS globaladdr*
    IMEMS memaddr*
    ITABLES tableaddr*
    IFUNCS funcaddr*
    IDATAS dataaddr*
    IELEMS elemaddr*
    IEXPORTS exportinst* }
```

Each component references runtime instances corresponding to respective declarations from the original module -- whether imported or defined -- in the order of their static indices.

Function instances, table instances, memory instances, global instances, and tag instances are denoted by their respective addresses in the store.

It is an invariant of the semantics that all export instances in a given module instance have different names.

> **Note:** All record fields except `exports` are to be considered *private* components of a module instance.
> They are not accessible to other modules, only to function instances originating from the same module.

### Function Instances

A *function instance* is the runtime representation of a function.

It effectively is a *closure* of the original function over the runtime module instance of its originating module.

The module instance is used to resolve references to other definitions during execution of the function.

```text
funcinst ::=
  { ITYPE deftype , IMODULE moduleinst , ICODE funccode }

funccode ::= func | hostfunc
```

A *host function* is a function expressed outside WebAssembly but passed to a module as an import.

The definition and behavior of host functions are outside the scope of this specification.

For the purpose of this specification, it is assumed that when invoked, a host function behaves non-deterministically, but within certain constraints that ensure the integrity of the runtime.

> **Note:** Function instances are immutable, and their identity is not observable by WebAssembly code.
> However, an embedder might provide implicit or explicit means for distinguishing their addresses.

### Table Instances

A *table instance* is the runtime representation of a table.
It records its type and holds a sequence of reference values.

```text
tableinst ::=
  { ITYPE tabletype , IREFS reff* }
```

Table elements can be mutated through table instructions, the execution of an active element segment, or by external means provided by the embedder.

It is an invariant of the semantics that all table elements have a type matching the element type of `tabletype`.

It also is an invariant that the length of the element sequence never exceeds the maximum size of `tabletype`.

### Memory Instances

A *memory instance* is the runtime representation of a linear memory.
It records its type and holds a sequence of bytes.

```text
meminst ::=
  { ITYPE memtype , IBYTES byte* }
```

The length of the sequence always is a multiple of the WebAssembly *page size*, which is defined to be the constant `65536` -- abbreviated `64 Ki`.

A memory's bytes can be mutated through memory instructions, the execution of an active data segment, or by external means provided by the embedder.

It is an invariant of the semantics that the length of the byte sequence, divided by page size, never exceeds the maximum size of `memtype`.

### Global Instances

A *global instance* is the runtime representation of a global variable.
It records its type and holds an individual value.

```text
globalinst ::=
  { ITYPE globaltype , IVALUE val }
```

The value of mutable globals can be mutated through variable instructions or by external means provided by the embedder.

It is an invariant of the semantics that the value has a type matching the value type of `globaltype`.

### Tag Instances

A *tag instance* is the runtime representation of a tag definition.
It records the defined type of the tag.

```text
taginst ::=
  { ITYPE tagtype }
```

### Element Instances

An *element instance* is the runtime representation of an element segment.
It holds a list of references and its type.

```text
eleminst ::=
  { ITYPE elemtype , IREFS reff* }
```

It is an invariant of the semantics that all elements of a segment have a type matching `elemtype`.

### Data Instances

A *data instance* is the runtime representation of a data segment.

It holds a list of bytes.

```text
datainst ::=
  { IBYTES byte* }
```

### Export Instances

An *export instance* is the runtime representation of an export.

It defines the export's name and the associated external address.

```text
exportinst ::=
  { NAME name , ADDR externaddr }
```

#### Conventions

The following auxiliary functions are assumed on sequences of external addresses.

They extract addresses of a specific kind in an order-preserving fashion:

* `funcsxa(xa*)` extracts all function addresses from `xa*`,
* `tablesxa(xa*)` extracts all table addresses from `xa*`,
* `memsxa(xa*)` extracts all memory addresses from `xa*`,
* `globalsxa(xa*)` extracts all global addresses from `xa*`,
* `tagsxa(xa*)` extracts all tag addresses from `xa*`.

### Aggregate Instances

A *structure instance* is the runtime representation of a heap object allocated from a structure type.

Likewise, an *array instance* is the runtime representation of a heap object allocated from an array type.

Both record their respective defined type and hold a list of the values of their *fields*.

```text
structinst ::=
  { ITYPE deftype , IFIELDS fieldval* }

arrayinst ::=
  { ITYPE deftype , IFIELDS fieldval* }

fieldval ::= val | packval

packval ::= packtype.pack iN N
```

#### Conventions

* Conversion of a regular value to a field value is defined as follows:

  ```text
  packfield_valtype(val) = val
  packfield_packtype(i32.const i) = packtype.pack wrap_(32, |packtype|)(i)
  ```

* The inverse conversion of a field value to a regular value is defined as follows:

  ```text
  unpackfield_valtype^ε(val) = val
  unpackfield_packtype^sx(packtype.pack i) = i32.const extend_(|packtype|, 32)^sx(i)
  ```

### Exception Instances

An *exception instance* is the runtime representation of an exception produced by a `throw` instruction.
It holds the address of the respective tag and the argument values.

```text
exninst ::=
  { ITAG tagaddr , IFIELDS val* }
```

### Stack

Besides the store, most instructions interact with an implicit *stack*.
The stack contains the two kinds of entries:

* *Values*: the *operands* of instructions.
* *Control Frames*: currently active control flow structures.

The latter can in turn be one of the following:

* *Labels*: active structured control instructions that can be targeted by branches.
* *(Call) Frames*: the *activation records* of active function calls.
* *Handlers*: active exception handlers.

> **Note:** Where clear from context, *call frame* is abbreviated to just *frame*.

All these entries can occur on the stack in any order during the execution of a program.
Stack entries are described by abstract syntax as follows.

> **Note:** It is possible to model the WebAssembly semantics using separate stacks for operands, control constructs, and calls.
>
> However, because the stacks are interdependent, additional book keeping about associated stack heights would be required.
>
> For the purpose of this specification, an interleaved representation is simpler.

#### Values

Values are represented by themselves.

#### Labels

Labels carry an argument arity `n` and their associated branch *target*, which is expressed syntactically as an instruction sequence:

```text
label ::= label_n { instr* }
```

Intuitively, `instr*` is the *continuation* to execute when the branch is taken, in place of the original control construct.

> **Note:** For example, a loop label has the form
>
> ```text
> label_n { (loop bt ...) }
> ```
>
> When performing a branch to this label, this executes the loop, effectively restarting it from the beginning. Conversely, a simple block label has the form
>
> ```text
> label_n { ε }
> ```
>
> When branching, the empty continuation ends the targeted block, such that execution can proceed with consecutive instructions.

#### Call Frames

Call frames carry the return arity `n` of the respective function, hold the values of its locals (including arguments) in the order corresponding to their static local indices, and a reference to the function's own module instance:

```text
callframe ::= frame_n { frame }

frame ::=
  { LOCALS (val^?)* , MODULE moduleinst }
```

Locals may be uninitialized, in which case they are empty.

Locals are mutated by respective variable instructions.

#### Exception Handlers

Exception handlers are installed by `try_table` instructions and record the corresponding list of catch clauses:

```text
handler ::= handler_n { catch* }
```

The handlers on the stack are searched when an exception is thrown.

#### Conventions

* The meta variable `L` ranges over labels where clear from context.
* The meta variable `f` ranges over frame states where clear from context.
* The meta variable `H` ranges over exception handlers where clear from context.
* The following auxiliary definition takes a block type and looks up the instruction type that it denotes in the current frame:

  ```text
  fblocktype_z(x) = t1* -> t2*    (if z.TYPES[x] ≈ func t1* -> t2*)
  fblocktype_z(t?) = ε -> t?
  ```

### Administrative Instructions

> **Note:** This section is only relevant for the formal notation.

In order to express the reduction of traps, calls, exception handling, and control instructions, the syntax of instructions is extended to include the following *administrative instructions*:

```text
instr ::= ...
  | reff
  | label_n { instr* } instr*
  | frame_n { frame } instr*
  | handler_n { catch* } instr*
  | trap
```

A reference represents a reference value of respective form "on the stack".

The `label`, `frame`, and `handler` instructions model labels, frames, and active exception handlers, respectively, "on the stack".

Moreover, the administrative syntax maintains the nesting structure of the original structured control instruction or function body and their instruction sequences.

The `trap` instruction represents the occurrence of a trap.

Traps are bubbled up through nested instruction sequences, ultimately reducing the entire program to a single `trap` instruction, signaling abrupt termination.

> **Note:** For example, the reduction rule for `block` is:
>
> ```text
> (block bt instr*) -> (label_n { ε } instr*)
> ```
>
> if the block type `bt` denotes a function type `func t1^m -> t2^n`, such that `n` is the block's result arity. This rule replaces the block with a label instruction, which can be interpreted as "pushing" the label on the stack. When its end is reached, i.e., the inner instruction sequence has been reduced to the empty sequence -- or rather, a sequence of `n` values representing the results -- then the `label` instruction is eliminated courtesy of its own reduction rule:
>
> ```text
> (label_n { instr* } val*) -> val*
> ```
>
> This can be interpreted as removing the label from the stack and only leaving the locally accumulated operand values. Validation guarantees that `n` matches the number `|val*|` of resulting values at this point.

### Configurations

A *configuration* describes the current computation.

It consists of the computation's *state* and the sequence of instructions left to execute.

The state in turn consists of a global store and a current frame referring to the module instance in which the computation runs, i.e., where the current function originates from.

```text
config ::= state ; instr*

state ::= store ; frame
```

> **Note:** The current version of WebAssembly is single-threaded, but configurations with multiple threads may be supported in the future.

#### Conventions

* The meta variable `z` ranges over frame states where clear from context.
* The following shorthands are defined for accessing a state `z = (s ; f)`:
  * `z.TYPES[x] = f.MODULE.ITYPES[x]`
  * `z.TAGS[x] = s.TAGS[f.MODULE.ITAGS[x]]`
  * `z.GLOBALS[x] = s.GLOBALS[f.MODULE.IGLOBALS[x]]`
  * `z.MEMS[x] = s.MEMS[f.MODULE.IMEMS[x]]`
  * `z.TABLES[x] = s.TABLES[f.MODULE.ITABLES[x]]`
  * `z.FUNCS[x] = s.FUNCS[f.MODULE.IFUNCS[x]]`
  * `z.DATAS[x] = s.DATAS[f.MODULE.IDATAS[x]]`
  * `z.ELEMS[x] = s.ELEMS[f.MODULE.IELEMS[x]]`
  * `z.LOCALS[x] = f.LOCALS[x]`
* These shorthands also extend to notation for updating state:
  * `z[.GGLOBALS[x].GVALUE = v] = s[.GLOBALS[f.MODULE.IGLOBALS[x]].IVALUE = v] ; f`
  * `z[.MMEMS[x].MBYTES[i : j] = b*] = s[.MEMS[f.MODULE.IMEMS[x]].IBYTES[i : j] = b*] ; f`
  * `z[.TTABLES[x].TREFS[i] = r] = s[.TABLES[f.MODULE.ITABLES[x]].IREFS[i] = r] ; f`
  * `z[.LOCALS[x] = v] = s ; f[.LOCALS[x] = v]`
