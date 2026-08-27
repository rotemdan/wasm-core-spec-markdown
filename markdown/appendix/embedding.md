## Embedding

A WebAssembly implementation will typically be *embedded* into a *host* environment.

An *embedder* implements the connection between such a host environment and the WebAssembly semantics as defined in the main body of this specification.

An embedder is expected to interact with the semantics in well-defined ways.

This section defines a suitable interface to the WebAssembly semantics in the form of entry points through which an embedder can access it.

The interface is intended to be complete, in the sense that an embedder does not need to reference other functional parts of the WebAssembly specification directly.

> **Note:** On the other hand, an embedder does not need to provide the host environment with access to all functionality defined in this interface. For example, an implementation may not support [parsing](embed-module-parse) of the [text format](text).

### Types

In the description of the embedder interface, syntactic classes from the [abstract syntax](syntax) and the [runtime's abstract machine](syntax-runtime) are used as names for variables that range over the possible objects from that class.

Hence, these syntactic classes can also be interpreted as types.

For numeric parameters, notation like `i:u64` is used to specify a symbolic name in addition to the respective value range.

### Booleans

Interface operation that are predicates return Boolean values:

```text
bool ::= false | true
```

### Exceptions and Errors

Invoking an exported function may throw or propagate exceptions, expressed by an auxiliary syntactic class:

```text
exception ::= exception exnaddr
```

The exception address `exnaddr` identifies the exception thrown.

Failure of an interface operation is also indicated by an auxiliary syntactic class:

```text
error ::= error
```

In addition to the error conditions specified explicitly in this section, such as invalid arguments or [exceptions](exception) and [traps](trap) resulting from [execution](exec), implementations may also return errors when specific [implementation limitations](impl) are reached.

> **Note:** Errors are abstract and unspecific with this definition. Implementations can refine it to carry suitable classifications and diagnostic messages.

### Pre- and Post-Conditions

Some operations state *pre-conditions* about their arguments or *post-conditions* about their results.

It is the embedder's responsibility to meet the pre-conditions.

If it does, the post conditions are guaranteed by the semantics.

In addition to pre- and post-conditions explicitly stated with each operation, the specification adopts the following conventions for [runtime objects](syntax-runtime) (`store`, `moduleinst`, [addresses](syntax-addr)):

* Every runtime object passed as a parameter must be [valid](valid-store) per an implicit pre-condition.
* Every runtime object returned as a result is [valid](valid-store) per an implicit post-condition.

> **Note:** As long as an embedder treats runtime objects as abstract and only creates and manipulates them through the interface defined here, all implicit pre-conditions are automatically met.

### Store

#### store_init() : store

1. Return the empty [store](syntax-store).

```text
store_init() = { }
```

### Modules

#### module_decode(byte* : module | error)

1. If there exists a derivation for the [byte](syntax-byte) sequence `byte*` as a `module` according to the [binary grammar for modules](binary-module), yielding a [module](syntax-module) `m`, then return `m`.
2. Else, return `ERROR`.

```text
module_decode(b*) = m      (if module =>* m : b*)
module_decode(b*) = ERROR   (otherwise)
```

#### module_parse(char* : module | error)

1. If there exists a derivation for the [source](text-source) `char*` as a `module` according to the [text grammar for modules](text-module), yielding a [module](syntax-module) `m`, then return `m`.
2. Else, return `ERROR`.

```text
module_parse(c*) = m      (if module =>* m : c*)
module_parse(c*) = ERROR   (otherwise)
```

#### module_validate(module : error?)

1. If `module` is [valid](valid-module), then return nothing.
2. Else, return `ERROR`.

```text
module_validate(m) = ε      (if |- module m : externtype* -> externtype'*)
module_validate(m) = ERROR   (otherwise)
```

#### module_instantiate(store, module, externaddr* : (store, moduleinst | exception | error))

1. Try [instantiating](exec-instantiation) `module` in `store` with [external addresses](syntax-externaddr) `externaddr*` as imports:

  a. If it succeeds with a [module instance](syntax-moduleinst) `moduleinst`, then let `result` be `moduleinst`.
  b. Else, let `result` be `ERROR`.

2. Return the new store paired with `result`.

```text
module_instantiate(S, m, ev*) = (S', F.MODULE)   (if instantiate(S, m, ev*) stepto* S'; F; ε)
module_instantiate(S, m, ev*) = (S', ERROR)        (otherwise, if instantiate(S, m, ev*) stepto* S'; F; result)
```

> **Note:** The store may be modified even in case of an error.

#### module_imports(module : (name, name, externtype)*)

1. Pre-condition: `module` is [valid](valid-module) with the external import types `externtype*` and external export types `externtype'*`.
2. Let `import*` be the [imports](syntax-import) of `module`.
3. Assert: the length of `import*` equals the length of `externtype*`.
4. For each `import_i` in `import*` and corresponding `externtype_i` in `externtype*`, do:

  a. Let `IMPORT nm_{i1} nm_{i2} xt_i` be the deconstruction of `import_i`.
  b. Let `result_i` be the triple `(nm_{i1}, nm_{i2}, externtype_i)`.

5. Return the concatenation of all `result_i`, in index order.
6. Post-condition: each `externtype_i` is [valid](valid-externtype) under the empty [context](context).

```text
module_imports(m) = (nm1, nm2, externtype)*
  (if (IMPORT nm1 nm2 xt*)* in m ∧ |- module m : externtype* -> externtype'*)
```

#### module_exports(module : (name, externtype)*)

1. Pre-condition: `module` is [valid](valid-module) with the external import types `externtype*` and external export types `externtype'*`.
2. Let `export*` be the [exports](syntax-export) of `module`.
3. Assert: the length of `export*` equals the length of `externtype'*`.
4. For each `export_i` in `export*` and corresponding `externtype'_i` in `externtype'*`, do:

  a. Let `EXPORT nm_i externidx_i` be the deconstruction of `export_i`.
  b. Let `result_i` be the pair `(nm_i, externtype'_i)`.

5. Return the concatenation of all `result_i`, in index order.
6. Post-condition: each `externtype'_i` is [valid](valid-externtype) under the empty [context](context).

```text
module_exports(m) = (nm, externtype')*
  (if (EXPORT nm xt*)* in m ∧ |- module m : externtype* -> externtype'*)
```

### Module Instances

#### instance_export(moduleinst, name : externaddr | error)

1. Assert: due to [validity](valid-moduleinst) of the [module instance](syntax-moduleinst) `moduleinst`, all its [export names](syntax-exportinst) are different.
2. If there exists an `exportinst_i` in `moduleinst.MIEXPORTS` such that [name](syntax-name) `exportinst_i.NAME` equals `name`, then:

  a. Return the [external address](syntax-externaddr) `exportinst_i.ADDR`.

3. Else, return `ERROR`.

```text
instance_export(m, name) = m.MIEXPORTS[i].ADDR   (if m.MIEXPORTS[i].NAME = name)
instance_export(m, name) = ERROR                  (otherwise)
```

### Functions

#### func_alloc(store, deftype, hostfunc : (store, funcaddr))

1. Pre-condition: the [defined type](syntax-deftype) `deftype` is [valid](valid-deftype) under the empty [context](context) and [expands](aux-expand-deftype) to a [function type](syntax-functype).
2. Let `funcaddr` be the result of [allocating a host function](alloc-func) in `store` with [defined type](syntax-deftype) `deftype`, host function code `hostfunc` and an empty [module instance](syntax-moduleinst).
3. Return the new store paired with `funcaddr`.

```text
func_alloc(S, dt, code) = (S', a)   (if allocfunc(S, dt, code, {}) = S', a)
```

> **Note:** This operation assumes that `hostfunc` satisfies the [pre- and post-conditions](exec-invoke-host) required for a function instance with type `deftype`.
>
> Regular (non-host) function instances can only be created indirectly through [module instantiation](embed-module-instantiate).

#### func_type(store, funcaddr : deftype)

1. Let `deftype` be the [defined type](syntax-deftype) `S.SFUNCS[a].FITYPE`.
2. Return `deftype`.
3. Post-condition: the returned [defined type](syntax-deftype) is [valid](valid-deftype) and [expands](aux-expand-deftype) to a [function type](syntax-functype).

```text
func_type(S, a) = S.SFUNCS[a].FITYPE
```

#### func_invoke(store, funcaddr, val* : (store, val* | exception | error))

1. Try [invoking](exec-invocation) the function `funcaddr` in `store` with [values](syntax-val) `val*` as arguments:

  a. If it succeeds with [values](syntax-val) `v'*` as results, then let `result` be `v'*`.
  b. Else if the outcome is an exception with a thrown [exception](exec-throw_ref) `refexnaddr exnaddr` as the result, then let `result` be `exception exnaddr`.
  c. Else it has trapped, hence let `result` be `ERROR`.

2. Return the new store paired with `result`.

```text
func_invoke(S, a, v*) = (S', v'*)      (if invoke(S, a, v*) stepto* S'; F; v'*)
func_invoke(S, a, v*) = (S', exception a')   (if invoke(S, a, v*) stepto* S'; F; (refexnaddr a') throw_ref)
func_invoke(S, a, v*) = (S', ERROR)      (if invoke(S, a, v*) stepto* S'; F; trap)
```

> **Note:** The store may be modified even in case of an error.

### Tables

#### table_alloc(store, tabletype, reff : (store, tableaddr))

1. Pre-condition: the `tabletype` is [valid](valid-tabletype) under the empty [context](context).
2. Let `tableaddr` be the result of [allocating a table](alloc-table) in `store` with [table type](syntax-tabletype) `tabletype` and initialization value `reff`.
3. Return the new store paired with `tableaddr`.

```text
table_alloc(S, tt, r) = (S', a)   (if alloctable(S, tt, r) = S', a)
```

#### table_type(store, tableaddr : tabletype)

1. Return `S.TABLES[a].TYPE`.
2. Post-condition: the returned [table type](syntax-tabletype) is [valid](valid-tabletype) under the empty [context](context).

```text
table_type(S, a) = S.TABLES[a].TYPE
```

#### table_read(store, tableaddr, i:u64 : reff | error)

1. Let `ti` be the [table instance](syntax-tableinst) `store.TABLES[tableaddr]`.
2. If `i` is larger than or equal to the length of `ti.TIREFS`, then return `ERROR`.
3. Else, return the [reference value](syntax-ref) `ti.TIREFS[i]`.

```text
table_read(S, a, i) = r      (if S.TABLES[a].TIREFS[i] = r)
table_read(S, a, i) = ERROR   (otherwise)
```

#### table_write(store, tableaddr, i:u64, reff : store | error)

1. Let `ti` be the [table instance](syntax-tableinst) `store.TABLES[tableaddr]`.
2. If `i` is larger than or equal to the length of `ti.TIREFS`, then return `ERROR`.
3. Replace `ti.TIREFS[i]` with the [reference value](syntax-ref) `reff`.
4. Return the updated store.

```text
table_write(S, a, i, r) = S'   (if S' = S with TABLES[a].TIREFS[i] = r)
table_write(S, a, i, r) = ERROR   (otherwise)
```

#### table_size(store, tableaddr : u64)

1. Return the length of `store.TABLES[tableaddr].TIREFS`.

```text
table_size(S, a) = n   (if |S.TABLES[a].TIREFS| = n)
```

#### table_grow(store, tableaddr, n:u64, reff : store | error)

1. Try [growing](grow-table) the [table instance](syntax-tableinst) `store.TABLES[tableaddr]` by `n` elements with initialization value `reff`:

  a. If it succeeds, return the updated store.
  b. Else, return `ERROR`.

```text
table_grow(S, a, n, r) = S'   (if S' = S with TABLES[a] = growtable(S.TABLES[a], n, r))
table_grow(S, a, n, r) = ERROR   (otherwise)
```

### Memories

#### mem_alloc(store, memtype : (store, memaddr))

1. Pre-condition: the `memtype` is [valid](valid-memtype) under the empty [context](context).
2. Let `memaddr` be the result of [allocating a memory](alloc-mem) in `store` with [memory type](syntax-memtype) `memtype`.
3. Return the new store paired with `memaddr`.

```text
mem_alloc(S, mt) = (S', a)   (if allocmem(S, mt) = S', a)
```

#### mem_type(store, memaddr : memtype)

1. Return `S.MEMS[a].TYPE`.
2. Post-condition: the returned [memory type](syntax-memtype) is [valid](valid-memtype) under the empty [context](context).

```text
mem_type(S, a) = S.MEMS[a].TYPE
```

#### mem_read(store, memaddr, i:u64 : byte | error)

1. Let `mi` be the [memory instance](syntax-meminst) `store.MEMS[memaddr]`.
2. If `i` is larger than or equal to the length of `mi.MIBYTES`, then return `ERROR`.
3. Else, return the [byte](syntax-byte) `mi.MIBYTES[i]`.

```text
mem_read(S, a, i) = b      (if S.MEMS[a].MIBYTES[i] = b)
mem_read(S, a, i) = ERROR   (otherwise)
```

#### mem_write(store, memaddr, i:u64, byte : store | error)

1. Let `mi` be the [memory instance](syntax-meminst) `store.MEMS[memaddr]`.
2. If `i` is larger than or equal to the length of `mi.MIBYTES`, then return `ERROR`.
3. Replace `mi.MIBYTES[i]` with `byte`.
4. Return the updated store.

```text
mem_write(S, a, i, b) = S'   (if S' = S with MEMS[a].MIBYTES[i] = b)
mem_write(S, a, i, b) = ERROR   (otherwise)
```

#### mem_size(store, memaddr : u64)

1. Return the length of `store.MEMS[memaddr].MIBYTES` divided by the [page size](page-size).

```text
mem_size(S, a) = n   (if |S.MEMS[a].MIBYTES| = n * 64 Ki)
```

#### mem_grow(store, memaddr, n:u64 : store | error)

1. Try [growing](grow-mem) the [memory instance](syntax-meminst) `store.MEMS[memaddr]` by `n` [pages](page-size):

  a. If it succeeds, return the updated store.
  b. Else, return `ERROR`.

```text
mem_grow(S, a, n) = S'   (if S' = S with MEMS[a] = growmem(S.MEMS[a], n))
mem_grow(S, a, n) = ERROR   (otherwise)
```

### Tags

#### tag_alloc(store, tagtype : (store, tagaddr))

1. Pre-condition: `tagtype` is [valid](valid-tagtype).
2. Let `tagaddr` be the result of [allocating a tag](alloc-tag) in `store` with [tag type](syntax-tagtype) `tagtype`.
3. Return the new store paired with `tagaddr`.

```text
tag_alloc(S, tt) = (S', a)   (if alloctag(S, tt) = S', a)
```

#### tag_type(store, tagaddr : tagtype)

1. Return `S.TAGS[a].TYPE`.
2. Post-condition: the returned [tag type](syntax-tagtype) is [valid](valid-tagtype).

```text
tag_type(S, a) = S.TAGS[a].TYPE
```

### Exceptions

#### exn_alloc(store, tagaddr, val* : (store, exnaddr))

1. Pre-condition: `tagaddr` is an allocated [tag address](syntax-tagaddr).
2. Let `exnaddr` be the result of [allocating an exception instance](syntax-exninst) in `store` with [tag address](syntax-tagaddr) `tagaddr` and initialization values `val*`.
3. Return the new store paired with `exnaddr`.

```text
exn_alloc(S, tagaddr, val*) = (S compose {SEXNS exninst}, |S.SEXNS|)   (if exninst = {EITAG tagaddr, EIFIELDS val*})
```

#### exn_tag(store, exnaddr : tagaddr)

1. Let `exninst` be the [exception instance](syntax-exninst) `store.SEXNS[exnaddr]`.
2. Return the [tag address](syntax-tagaddr) `exninst.EITAG`.

```text
exn_tag(S, a) = exninst.EITAG   (if exninst = S.SEXNS[a])
```

#### exn_read(store, exnaddr : val*)

1. Let `exninst` be the [exception instance](syntax-exninst) `store.SEXNS[exnaddr]`.
2. Return the [values](syntax-val) `exninst.EIFIELDS`.

```text
exn_read(S, a) = exninst.EIFIELDS   (if exninst = S.SEXNS[a])
```

### Globals

#### global_alloc(store, globaltype, val : (store, globaladdr))

1. Pre-condition: the `globaltype` is [valid](valid-globaltype) under the empty [context](context).
2. Let `globaladdr` be the result of [allocating a global](alloc-global) in `store` with [global type](syntax-globaltype) `globaltype` and initialization value `val`.
3. Return the new store paired with `globaladdr`.

```text
global_alloc(S, gt, v) = (S', a)   (if allocglobal(S, gt, v) = S', a)
```

#### global_type(store, globaladdr : globaltype)

1. Return `S.GLOBALS[a].TYPE`.
2. Post-condition: the returned [global type](syntax-globaltype) is [valid](valid-globaltype) under the empty [context](context).

```text
global_type(S, a) = S.GLOBALS[a].TYPE
```

#### global_read(store, globaladdr : val)

1. Let `gi` be the [global instance](syntax-globalinst) `store.GLOBALS[globaladdr]`.
2. Return the [value](syntax-val) `gi.GIVALUE`.

```text
global_read(S, a) = v   (if S.GLOBALS[a].GIVALUE = v)
```

#### global_write(store, globaladdr, val : store | error)

1. Let `gi` be the [global instance](syntax-globalinst) `store.GLOBALS[globaladdr]`.
2. Let `mut t` be the structure of the [global type](syntax-globaltype) `gi.GITYPE`.
3. If `mut` is empty, then return `ERROR`.
4. Replace `gi.GIVALUE` with the [value](syntax-val) `val`.
5. Return the updated store.

```text
global_write(S, a, v) = S'   (if S.GLOBALS[a].GITYPE = mut t ∧ S' = S with GLOBALS[a].GIVALUE = v)
global_write(S, a, v) = ERROR   (otherwise)
```

### Values

#### ref_type(store, reff : reftype)

1. Pre-condition: the [reference](syntax-ref) `reff` is [valid](valid-val) under store `S`.
2. Return the [reference type](syntax-reftype) `t` with which `reff` is valid.
3. Post-condition: the returned [reference type](syntax-reftype) is [valid](valid-reftype) under the empty [context](context).

```text
ref_type(S, r) = t   (if S |-val r : t)
```

> **Note:** In future versions of WebAssembly, not all references may carry precise type information at run time. In such cases, this function may return a less precise supertype.

#### val_default(valtype : val)

1. If `default_valtype` is not defined, then return `ERROR`.
2. Else, return the [value](syntax-val) `default_valtype`.

```text
val_default(t) = v      (if default_t = v)
val_default(t) = ERROR   (if default_t = ε)
```

### Matching

#### match_valtype(valtype1, valtype2 : bool)

1. Pre-condition: the [value types](syntax-valtype) `valtype1` and `valtype2` are [valid](valid-valtype) under the empty [context](context).
2. If `valtype1` [matches](match-valtype) `valtype2`, then return `TRUE`.
3. Else, return `FALSE`.

```text
match_reftype(t1, t2) = TRUE    (if |- valtypematch t1 <: t2)
match_reftype(t1, t2) = FALSE   (otherwise)
```

#### match_externtype(externtype1, externtype2 : bool)

1. Pre-condition: the [extern types](syntax-externtype) `externtype1` and `externtype2` are [valid](valid-externtype) under the empty [context](context).
2. If `externtype1` [matches](match-externtype) `externtype2`, then return `TRUE`.
3. Else, return `FALSE`.

```text
match_externtype(et1, et2) = TRUE    (if |- externtypematch et1 <: et2)
match_externtype(et1, et2) = FALSE   (otherwise)
```
