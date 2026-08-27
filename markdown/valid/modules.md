## Modules

Modules are valid when all the components they contain are valid. To verify this, most definitions are themselves classified with a suitable type.

### Types

The sequence of types defined in a module is validated incrementally, yielding a sequence of defined types representing them individually.

The type definition `(type rectype)` is valid with the defined type sequence `dt*` if:

* The length of `C.TYPES` is equal to `x`.
* The defined type sequence `dt*` is of the form `rolldt_x^*(rectype)`.
* Let `C'` be the same context as `C`, but with the defined type sequence `dt*` appended to the field `TYPES`.
* Under the context `C'`, the recursive type `rectype` is valid for the type index `x`.

```text
x = |C.TYPES|
dt* = rolldt_x^*(rectype)
C ⊕ { TYPES dt* } |- rectype : OK(x)
──────────────────────────────────────
C |- type rectype : dt*
```

```text
────────────────
{ } |- ε : ε

C |- type1 : dt1*
C ⊕ { TYPES dt1* } |- type* : dt*
──────────────────────────────────
C |- type1 type* : dt1* dt*
```

### Tags

Tags are classified by their tag types, which are defined types expanding to function types.

The tag `(tag tagtype)` is valid with the tag type `tagtype'` if:

* The tag type `tagtype` is valid.
* The tag type `tagtype'` is `clostype_C(tagtype)`.

```text
C |- tagtype : OK
────────────────────
C |- tag tagtype : clostype_C(tagtype)
```

### Globals

Globals are classified by global types.

The global `(global globaltype expr)` is valid with the global type `globaltype` if:

* The global type `globaltype` is valid.
* The global type `globaltype` is of the form `(mut? t)`.
* The expression `expr` is valid with the value type `t`.
* `expr` is constant.

```text
C |- globaltype : OK
globaltype = mut? t
C |- expr : t const
──────────────────────────
C |- global globaltype expr : globaltype
```

Sequences of globals are handled incrementally, such that each definition has access to previous definitions.

The global sequence `global*` is valid with the global type sequence `gt*` if:

* Either:
   * The global sequence `global*` is empty.
   * The global type sequence `gt*` is empty.
* Or:
   * The global sequence `global*` is of the form `global1 global'*`.
   * The global type sequence `gt*` is of the form `gt1 gt*`.
   * The global `global1` is valid with the global type `gt1`.
   * Let `C'` be the same context as `C`, but with the global type sequence `gt1` appended to the field `GLOBALS`.
   * Under the context `C'`, the global sequence `global'*` is valid with the global type sequence `gt*`.

```text
────────────────
{ } |- ε : ε

C |- global1 : gt1
C ⊕ { GLOBALS gt1 } |- global* : gt*
─────────────────────────────────
C |- global1 global* : gt1 gt*
```

### Memories

Memories are classified by memory types.

The memory `(memory memtype)` is valid with the memory type `memtype` if:

* The memory type `memtype` is valid.

```text
C |- memtype : OK
──────────────────
C |- memory memtype : memtype
```

### Tables

Tables are classified by table types.

The table `(table tabletype expr)` is valid with the table type `tabletype` if:

* The table type `tabletype` is valid.
* The table type `tabletype` is of the form `(at lim rt)`.
* The expression `expr` is valid with the value type `rt`.
* `expr` is constant.

```text
C |- tabletype : OK
tabletype = at lim rt
C |- expr : rt const
──────────────────────────
C |- table tabletype expr : tabletype
```

### Functions

Functions are classified by defined types that expand to function types of the form `func t1* -> t2*`.

The function `(func x local* expr)` is valid with the type `C.TYPES[x]` if:

* The type `C.TYPES[x]` exists.
* The expansion of `C.TYPES[x]` is `(func t1* -> t2*)`.
* For all `local` in `local*`:
   * The local `local` is valid with the local type `lt`.
* `lt*` is the concatenation of all such `lt`.
* Under the context `C` with the field `LOCALS` appended by `(set t1)* lt*` and the field `LABELS` appended by `t2*` and the field `RETURN` appended by `t2*`, the expression `expr` is valid with the result type `t2*`.

```text
C.TYPES[x] ≈ func t1* -> t2*
(C |- local local : lt)*
C ⊕ { LOCALS (set t1)* lt*, LABELS (t2*), RETURN (t2*) } |- expr : t2*
──────────────────────────────────────────────────────────────
C |- func x local* expr : C.TYPES[x]
```

### Locals

Locals are classified with local types.

The local `(local t)` is valid with the local type `(init t)` if:

* The value type `t` is valid.
* Either:
   * The initialization status `init` is of the form `set`.
   * A default value for `t` is defined.
* Or:
   * The initialization status `init` is of the form `unset`.
   * A default value for `t` is not defined.

```text
C |- t : OK
default_t ≠ ε
──────────────────
C |- local t : set t

C |- t : OK
default_t = ε
──────────────────
C |- local t : unset t
```

> **Note:** For cases where both rules are applicable, the former yields the more permissable type.

### Data Segments

Data segments are classified by the singleton data type, which merely expresses well-formedness.

The memory segment `(DATA b* datamode)` is valid if:

* The data mode `datamode` is valid.

```text
C |- datamode : OK
────────────────────
C |- data DATA b* datamode : OK
```

#### Data Mode

The data mode `datamode` is valid if:

* Either:
   * The data mode `datamode` is of the form `passive`.
* Or:
   * The data mode `datamode` is of the form `(active x expr)`.
   * The memory `C.MEMS[x]` exists.
   * The memory `C.MEMS[x]` is of the form `(at lim page)`.
   * The expression `expr` is valid with the value type `at`.
   * `expr` is constant.

```text
────────────────
C |- passive : OK

C.MEMS[x] = at lim page
C |- expr : at const
────────────────────────
C |- active x expr : OK
```

### Element Segments

Element segments are classified by their element type.

The table segment `(ELEM elemtype expr* elemmode)` is valid with the element type `elemtype` if:

* The reference type `elemtype` is valid.
* For all `expr` in `expr*`:
   * The expression `expr` is valid with the value type `elemtype`.
   * `expr` is constant.
* The element mode `elemmode` is valid with the element type `elemtype`.

```text
C |- elemtype : OK
(C |- expr : elemtype const)*
C |- elemmode : elemtype
──────────────────────────────────
C |- elem ELEM elemtype expr* elemmode : elemtype
```

#### Element Mode

The element mode `elemmode` is valid with the element type `rt` if:

* Either:
   * The element mode `elemmode` is of the form `passive`.
* Or:
   * The element mode `elemmode` is of the form `declare`.
* Or:
   * The element mode `elemmode` is of the form `(active x expr)`.
   * The table `C.TABLES[x]` exists.
   * The table `C.TABLES[x]` is of the form `(at lim rt')`.
   * The reference type `rt` matches the reference type `rt'`.
   * The expression `expr` is valid with the value type `at`.
   * `expr` is constant.

```text
────────────────
C |- passive : rt

────────────────
C |- declare : rt

C.TABLES[x] = at lim rt'
C |- rt <: rt'
C |- expr : at const
────────────────────────
C |- active x expr : rt
```

### Start Function

The start function `(start x)` is valid if:

* The function `C.FUNCS[x]` exists.
* The expansion of `C.FUNCS[x]` is `(func ->)`.

```text
C.FUNCS[x] ≈ func ε -> ε
──────────────────────────
C |- start x : OK
```

### Imports

Imports are classified by external types.

The import `(import name1 name2 xt)` is valid with the external type `externtype` if:

* The external type `xt` is valid.
* The external type `externtype` is `clostype_C(xt)`.

```text
C |- xt : OK
────────────────────────────
C |- import name1 name2 xt : clostype_C(xt)
```

### Exports

Exports are classified by their external type.

The export `(export name externidx)` is valid with the name `name` and the external type `xt` if:

* The external index `externidx` is valid with the external type `xt`.

```text
C |- externidx : xt
──────────────────────
C |- export name externidx : name xt
```

#### XXTAG x

The external index `(tag x)` is valid with the external type `(tag jt)` if:

* The tag `C.TAGS[x]` exists.
* The tag `C.TAGS[x]` is of the form `jt`.

```text
C.TAGS[x] = jt
──────────────────
C |- tag x : tag jt
```

#### XXGLOBAL x

The external index `(global x)` is valid with the external type `(global gt)` if:

* The global `C.GLOBALS[x]` exists.
* The global `C.GLOBALS[x]` is of the form `gt`.

```text
C.GLOBALS[x] = gt
────────────────────
C |- global x : global gt
```

#### XXMEM x

The external index `(mem x)` is valid with the external type `(mem mt)` if:

* The memory `C.MEMS[x]` exists.
* The memory `C.MEMS[x]` is of the form `mt`.

```text
C.MEMS[x] = mt
─────────────────
C |- mem x : mem mt
```

#### XXTABLE x

The external index `(table x)` is valid with the external type `(table tt)` if:

* The table `C.TABLES[x]` exists.
* The table `C.TABLES[x]` is of the form `tt`.

```text
C.TABLES[x] = tt
────────────────────
C |- table x : table tt
```

#### XXFUNC x

The external index `(func x)` is valid with the external type `(func dt)` if:

* The function `C.FUNCS[x]` exists.
* The function `C.FUNCS[x]` is of the form `dt`.

```text
C.FUNCS[x] = dt
─────────────────
C |- func x : func dt
```

### Modules

Modules are classified by their mapping from the external types of their imports to those of their exports.

A module is entirely closed, that is, its components can only refer to definitions that appear in the module itself. Consequently, no initial context is required. Instead, the context `C` for validation of the module's content is constructed from the definitions in the module.

The module `(module type* import* tag* global* mem* table* func* data* elem* start? export*)` is valid with the module type `moduletype` if:

* Under the context `{ RETURN ε }`, the type definition sequence `type*` is valid with the defined type sequence `dt'*`.
* For all `import` in `import*`:
   * Under the context `{ TYPES dt'*, RETURN ε }`, the import `import` is valid with the external type `xt_i`.
* `xt_i*` is the concatenation of all such `xt_i`.
* For all `tag` in `tag*`:
   * Under the context `C'`, the tag `tag` is valid with the tag type `jt`.
* `jt*` is the concatenation of all such `jt`.
* Under the context `C'`, the global sequence `global*` is valid with the global type sequence `gt*`.
* For all `mem` in `mem*`:
   * Under the context `C'`, the memory `mem` is valid with the memory type `mt`.
* `mt*` is the concatenation of all such `mt`.
* For all `table` in `table*`:
   * Under the context `C'`, the table `table` is valid with the table type `tt`.
* `tt*` is the concatenation of all such `tt`.
* For all `func` in `func*`:
   * The function `func` is valid with the defined type `dt`.
* `dt*` is the concatenation of all such `dt`.
* For all `data` in `data*`:
   * The memory segment `data` is valid.
* `ok*` is the concatenation of all such `ok`.
* For all `elem` in `elem*`:
   * The table segment `elem` is valid with the element type `rt`.
* `rt*` is the concatenation of all such `rt`.
* If `start` is defined, then:
   * The start function `start` is valid.
* For all `export` in `export*`:
   * The export `export` is valid with the name `nm` and the external type `xt_e`.
* `nm*` is the concatenation of all such `nm`.
* `xt_e*` is the concatenation of all such `xt_e`.
* `nm* disjoint` is true.
* The context `C` is of the form `C'` with the field `TAGS` appended by `jt_i* jt*` and the field `GLOBALS` appended by `gt*` and the field `MEMS` appended by `mt_i* mt*` and the field `TABLES` appended by `tt_i* tt*` and the field `DATAS` appended by `ok*` and the field `ELEMS` appended by `rt*`.
* The context `C'` is of the form `{ TYPES dt'*, GLOBALS gt_i*, FUNCS dt_i* dt*, RETURN ε, REFS x* }`.
* The function index sequence `x*` is of the form `freefuncidx(global* mem* table* elem* export*)`.
* The tag type sequence `jt_i*` is of the form `tagsxt(xt_i*)`.
* The global type sequence `gt_i*` is of the form `globalsxt(xt_i*)`.
* The memory type sequence `mt_i*` is of the form `memsxt(xt_i*)`.
* The table type sequence `tt_i*` is of the form `tablesxt(xt_i*)`.
* The defined type sequence `dt_i*` is of the form `funcsxt(xt_i*)`.
* The module type `moduletype` is `clostype_C(xt_i* -> xt_e*)`.

```text
{ } |- type* : dt'*
(TYPES dt'* |- import : xt_i)*
(C' |- tag : jt)*
C' |- globals global* : gt*
(C' |- mem : mt)*
(C' |- table : tt)*
(C |- func : dt)*
(C |- data : ok)*
(C |- elem : rt)*
(C |- start : OK)?
(C |- export : nm xt_e)*
nm* disjoint
C = C' ⊕ { TAGS jt_i* jt*, GLOBALS gt*, MEMS mt_i* mt*, TABLES tt_i* tt*, DATAS ok*, ELEMS rt* }
C' = { TYPES dt'*, GLOBALS gt_i*, FUNCS dt_i* dt*, REFS x* }
x* = freefuncidx(global* mem* table* elem* export*)
jt_i* = tagsxt(xt_i*)
gt_i* = globalsxt(xt_i*)
mt_i* = memsxt(xt_i*)
tt_i* = tablesxt(xt_i*)
dt_i* = funcsxt(xt_i*)
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
|- module type* import* tag* global* mem* table* func* data* elem* start? export* : clostype_C(xt_i* -> xt_e*)
```

> **Note:** All functions in a module are mutually recursive. Consequently, the definition of the context `C` in this rule is recursive: it depends on the outcome of validation of the function, table, memory, and global definitions contained in the module, which itself depends on `C`. However, this recursion is just a specification device. All types needed to construct `C` can easily be determined from a simple pre-pass over the module that does not perform any actual validation.
>
> Globals, however, are not recursive but evaluated sequentially, such that each constant expressions only has access to imported or previously defined globals.
