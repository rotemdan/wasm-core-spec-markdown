## Type Soundness

The [type system](type-system) of WebAssembly is *sound*, implying both *type safety* and *memory safety* with respect to the WebAssembly semantics. For example:

* All types declared and derived during validation are respected at run time; e.g., every [local](syntax-local) or [global](syntax-global) variable will only contain type-correct values, every [instruction](syntax-instr) will only be applied to operands of the expected type, and every [function](syntax-func) [invocation](exec-invocation) always evaluates to a result of the right type (if it does not diverge, throw an exception, or [trap](trap)).
* No memory location will be read or written except those explicitly defined by the program, i.e., as a [local](syntax-local), a [global](syntax-global), an element in a [table](syntax-table), or a location within a linear [memory](syntax-mem).
* There is no undefined behavior, i.e., the [execution rules](exec) cover all possible cases that can occur in a [valid](valid) program, and the rules are mutually consistent.

Soundness also is instrumental in ensuring additional properties, most notably, *encapsulation* of function and module scopes: no [locals](syntax-local) can be accessed outside their own function and no [module](syntax-module) components can be accessed outside their own module unless they are explicitly [exported](syntax-export) or [imported](syntax-import).

The typing rules defining WebAssembly [validation](valid) only cover the *static* components of a WebAssembly program. In order to state and prove soundness precisely, the typing rules must be extended to the *dynamic* components of the abstract [runtime](syntax-runtime), that is, the [store](syntax-store), [configurations](syntax-config), and [administrative instructions](syntax-instr-admin). [^cite-pldi2017]

### Contexts

In order to check [rolled up](aux-roll-rectype) recursive types, the [context](context) is locally extended with an additional component that records the [sub type](syntax-subtype) corresponding to each [recursive type index](syntax-rectypeidx) within the current group of [recursive types](syntax-rectype):

```text
context ::= { ..., RECS subtype* }
```

### Types

Well-formedness for [extended type forms](type-ext) is defined as follows.

The [type use](syntax-typeuse) `(rec . i)` is [valid](valid-typeuse) if:

   * The [recursive type](syntax-subtype) `C.RECS[i]` exists.

```text
C.RECS[i] = st
─────────────────────────────
C |- (rec . i) : OK
```

The [heap type](syntax-heaptype) `bot` is always [valid](valid-heaptype).

```text
─────────────────────────────
C |- bot : OK
```

The [value type](syntax-valtype) `bot` is always [valid](valid-valtype).

```text
─────────────────────────────
C |- bot : OK
```

The [recursive type](syntax-rectype) `(rec subtype*)` is [valid](valid-rectype) for `i` if:

   * Either:
      * The sub type sequence `subtype*` is empty.
   * Or:
      * The sub type sequence `subtype*` is of the form `subtype1 subtype'*`.
      * The [sub type](syntax-subtype) `subtype1` is [valid](valid-subtype) for `i`.
      * The [recursive type](syntax-rectype) `(rec subtype*)` is [valid](valid-rectype) for `i + 1`.

```text
─────────────────────────────
C |- rec ε : OK(i)
```

```text
C |- subtype1 : OK(i)
C |- rec subtype* : OK(i + 1)
─────────────────────────────
C |- rec (subtype1 subtype*) : OK(i)
```

The [sub type](syntax-subtype) `(sub final? typeuse* comptype)` is [valid](valid-subtype) for `i` if:

   * The length of `typeuse*` is less than or equal to `1`.
   * For all `typeuse` in `typeuse*`:
      * The [type use](syntax-typeuse) `typeuse` is [valid](valid-typeuse).
      * `typeuse <: i` is true.
      * The [sub type](syntax-subtype) `unrollht_C(typeuse)` is of the form `(sub typeuse'* comptype')`.
   * `comptype'*` is the concatenation of all such `comptype'`.
   * The [composite type](syntax-comptype) `comptype` is [valid](valid-comptype).
   * For all `comptype'` in `comptype'*`:
      * The [composite type](syntax-comptype) `comptype` [matches](match-comptype) the [composite type](syntax-comptype) `comptype'`.

```text
|typeuse*| <= 1
(C |- typeuse : OK)*
(typeuse <: i)*
(unrollht_C(typeuse) = sub typeuse'* comptype')*
C |- comptype : OK
(C |- comptypematch comptype <: comptype')*
─────────────────────────────
C |- sub final? typeuse* comptype : OK(i)
```

where:

```text
unrollht_C(deftype) = unrolldt(deftype)
unrollht_C(typeidx) = unrolldt(C.TYPES[typeidx])
unrollht_C(rec . i) = C.RECS[i]

rec . j <: i = j < i
typeuse <: i = true    (otherwise)
```

> **Note:** The new rules for [recursive types](syntax-rectype) and [sub types](syntax-subtype) complement the ones [previously given](valid-subtype), which only allowed regular [type indices](syntax-typeidx) as supertypes. They define validity of [rolled-up](aux-roll-rectype) recursive types, like they occur in [defined types](syntax-deftype), in turn needed to define [validity](valid-context) of [contexts](context). None of these rules are needed in the implementation of a validator.

The [defined type](syntax-deftype) `(rectype . i)` is [valid](valid-deftype) if:

   * Let `C'` be the same context as `C`, but with the sub type sequence `subtype^n` prepended to the field `RECS`.
   * Under the context `C'`, the [recursive type](syntax-rectype) `rectype` is [valid](valid-rectype) for `0`.
   * The [recursive type](syntax-rectype) `rectype` is of the form `(rec subtype^n)`.
   * `i` is less than `n`.

```text
C, RECS subtype^n |- rectype : OK(0)
rectype = rec subtype^n
i < n
─────────────────────────────
C |- deftype rectype . i : OK
```

### Subtyping

Inside a [rolled-up](aux-roll-rectype) [recursive type](syntax-rectype), a [recursive type index](syntax-rectypeidx) can [match](match-heaptype) another [heap type](syntax-heaptype).

```text
C.RECS[i] = sub final? (struct fieldtype*)
─────────────────────────────
C |- rec . i <: struct
```

```text
C.RECS[i] = sub final? (array fieldtype)
─────────────────────────────
C |- rec . i <: array
```

```text
C.RECS[i] = sub final? (func t1* -> t2*)
─────────────────────────────
C |- rec . i <: func
```

```text
C.RECS[i] = sub final? typeuse* ct
─────────────────────────────
C |- rec . i <: typeuse*[j]
```

> **Note:** These rules complement the previously given rules for [matching heap types](match-heaptype). They are only invoked when checking [validity](valid-rectype-ext) of [rolled-up](aux-roll-rectype) [recursive types](syntax-rectype).

### Results

[Results](syntax-result) can be classified by [result types](syntax-resulttype) as follows.

#### Results `val*`

* For each [value](syntax-val) `val_i` in `val*`:
  * The value `val_i` is [valid](valid-val) with some [value type](syntax-valtype) `t_i`.
* Let `t*` be the concatenation of all `t_i`.
* Then the result is valid with [result type](syntax-resulttype) `[t*]`.

```text
(S |- val : t)*
─────────────────────────────
S |- result val* : [t*]
```

#### Results `(ref.exn a) throw_ref`

* The value `ref.exn a` must be [valid](valid-val).
* Then the result is valid with [result type](syntax-resulttype) `[t*]`, for any [valid](valid-resulttype) [closed](type-closed) [result types](syntax-resulttype).

```text
S |- val ref.exn a : ref exn
|- resulttype [t*] : OK
─────────────────────────────
S |- result (ref.exn a) throw_ref : [t*]
```

#### Results `trap`

* The result is valid with [result type](syntax-resulttype) `[t*]`, for any [valid](valid-resulttype) [closed](type-closed) [result types](syntax-resulttype).

```text
|- resulttype [t*] : OK
─────────────────────────────
S |- result trap : [t*]
```

### Store Validity

The following typing rules specify when a runtime [store](syntax-store) `S` is *valid*. A valid store must consist of [tag](syntax-taginst), [global](syntax-globalinst), [memory](syntax-meminst), [table](syntax-tableinst), [function](syntax-funcinst), [data](syntax-datainst), [element](syntax-eleminst), [structure](syntax-structinst), [array](syntax-arrayinst), [exception](syntax-exninst), and [module](syntax-moduleinst) instances that are themselves valid, relative to `S`.

To that end, each kind of instance is classified by a respective [tag](syntax-tagtype), [global](syntax-globaltype), [memory](syntax-memtype), [table](syntax-tabletype), [function](syntax-functype), or [element](syntax-eleminst) type, or just `ok` in the case of [data](syntax-datainst) [structures](syntax-structinst), [arrays](syntax-arrayinst), or [exceptions](syntax-exninst). Module instances are classified by *module contexts*, which are regular [contexts](context) repurposed as module types describing the [index spaces](syntax-index) defined by a module.

#### Store `S`

* Each [tag instance](syntax-taginst) `taginst_i` in `S.TAGS` must be [valid](valid-taginst) with some [tag type](syntax-tagtype) `tagtype_i`.
* Each [global instance](syntax-globalinst) `globalinst_i` in `S.GLOBALS` must be [valid](valid-globalinst) with some [global type](syntax-globaltype) `globaltype_i`.
* Each [memory instance](syntax-meminst) `meminst_i` in `S.MEMS` must be [valid](valid-meminst) with some [memory type](syntax-memtype) `memtype_i`.
* Each [table instance](syntax-tableinst) `tableinst_i` in `S.TABLES` must be [valid](valid-tableinst) with some [table type](syntax-tabletype) `tabletype_i`.
* Each [function instance](syntax-funcinst) `funcinst_i` in `S.FUNCS` must be [valid](valid-funcinst) with some [defined type](syntax-deftype) `deftype_i`.
* Each [data instance](syntax-datainst) `datainst_i` in `S.DATAS` must be [valid](valid-datainst).
* Each [element instance](syntax-eleminst) `eleminst_i` in `S.ELEMS` must be [valid](valid-eleminst) with some [reference type](syntax-reftype) `reftype_i`.
* Each [structure instance](syntax-structinst) `structinst_i` in `S.STRUCTS` must be [valid](valid-structinst).
* Each [array instance](syntax-arrayinst) `arrayinst_i` in `S.ARRAYS` must be [valid](valid-arrayinst).
* Each [exception instance](syntax-exninst) `exninst_i` in `S.EXNS` must be [valid](valid-exninst).
* No [reference](syntax-ref) to a bound [structure address](syntax-structaddr) must be reachable from itself through a path consisting only of indirections through immutable structure, or array [fields](syntax-fieldtype) or fields of [exception instances](syntax-exninst).
* No [reference](syntax-ref) to a bound [array address](syntax-arrayaddr) must be reachable from itself through a path consisting only of indirections through immutable structure or array [fields](syntax-fieldtype) or fields of [exception instances](syntax-exninst).
* No [reference](syntax-ref) to a bound [exception address](syntax-exnaddr) must be reachable from itself through a path consisting only of indirections through immutable structure or array [fields](syntax-fieldtype) or fields of [exception instances](syntax-exninst).
* Then the store is valid.

```text
(S |- taginst : tagtype)*
(S |- globalinst : globaltype)*
(S |- meminst : memtype)*
(S |- tableinst : tabletype)*
(S |- funcinst : deftype)*
(S |- datainst : OK)*
(S |- eleminst : reftype)*
(S |- structinst : OK)*
(S |- arrayinst : OK)*
(S |- exninst : OK)*
S = { TAGS taginst*, GLOBALS globalinst*, MEMS meminst*, TABLES tableinst*,
      FUNCS funcinst*, DATAS datainst*, ELEMS eleminst*, STRUCTS structinst*,
      ARRAYS arrayinst*, EXNS exninst* }
(S.STRUCTS[a_s] = structinst)*
(not ((ref.struct a_s) ≫^+_S (ref.struct a_s)))*
(S.ARRAYS[a_a] = arrayinst)*
(not ((ref.array a_a) ≫^+_S (ref.array a_a)))*
(S.EXNS[a_e] = exninst)*
(not ((ref.exn a_e) ≫^+_S (ref.exn a_e)))*
─────────────────────────────
|- store S : OK
```

where `val1 ≫^+_S val2` denotes the transitive closure of the following *immutable reachability* relation on [values](syntax-val):

```text
(ref.struct a) ≫_S S.STRUCTS[a].IFIELDS[i]   iff   expanddt(S.STRUCTS[a].ITYPE) = struct ft1^i st ft2*
(ref.array a)  ≫_S S.ARRAYS[a].AIFIELDS[i]   iff   expanddt(S.ARRAYS[a].AITYPE) = array st
(ref.exn a)    ≫_S S.EXNS[a].EIFIELDS[i]
(ref.extern reff) ≫_S reff
```

> **Note:** The constraint on reachability through immutable fields prevents the presence of cyclic data structures that can not be constructed in the language. Cycles can only be formed using mutation.

#### Tag Instances `{ TYPE tagtype }`

* The [tag type](syntax-tagtype) `tagtype` must be [valid](valid-tagtype) under the empty [context](context).
* Then the tag instance is valid with [tag type](syntax-tagtype) `tagtype`.

```text
|- tagtype : OK
─────────────────────────────
S |- taginst { TYPE tagtype } : tagtype
```

#### Global Instances `{ TYPE mut t, VALUE val }`

* The [global type](syntax-globaltype) `mut t` must be [valid](valid-globaltype) under the empty [context](context).
* The [value](syntax-val) `val` must be [valid](valid-val) with some [value type](syntax-valtype) `t'`.
* The [value type](syntax-valtype) `t'` must [match](match-valtype) the [value type](syntax-valtype) `t`.
* Then the global instance is valid with [global type](syntax-globaltype) `mut t`.

```text
|- globaltype mut t : OK
S |- val : t'
|- valtypematch t' <: t
─────────────────────────────
S |- globalinst { TYPE mut t, VALUE val } : mut t
```

#### Memory Instances `{ TYPE (addrtype [n .. m]), BYTES b* }`

* The [memory type](syntax-memtype) `addrtype [n .. m]` must be [valid](valid-memtype) under the empty [context](context).
* Let `limits` be `[n .. m]`.
* The length of `b*` must equal `m` multiplied by the [page size](page-size) `64 Ki`.
* Then the memory instance is valid with [memory type](syntax-memtype) `addrtype [n .. m]`.

```text
|- memtype addrtype [n .. m] : OK
|b*| = n * 64 Ki
─────────────────────────────
S |- meminst { TYPE (addrtype [n .. m]), BYTES b* } : addrtype [n .. m]
```

#### Table Instances `{ TYPE (addrtype [n .. m] t), REFS reff* }`

* The [table type](syntax-tabletype) `addrtype [n .. m] t` must be [valid](valid-tabletype) under the empty [context](context).
* Let `limits` be `[n .. m]`.
* The length of `reff*` must equal `n`.
* For each [reference](syntax-ref) `reff_i` in the table's elements `reff^n`:
  * The [reference](syntax-ref) `reff_i` must be [valid](valid-ref) with some [reference type](syntax-reftype) `t'_i`.
  * The [reference type](syntax-reftype) `t'_i` must [match](match-reftype) the [reference type](syntax-reftype) `t`.
* Then the table instance is valid with [table type](syntax-tabletype) `addrtype [n .. m] t`.

```text
|- tabletype addrtype [n .. m] t : OK
|reff*| = n
(S |- reff : t')*
(|- reftypematch t' <: t)*
─────────────────────────────
S |- tableinst { TYPE (addrtype [n .. m] t), REFS reff* } : addrtype [n .. m] t
```

#### Function Instances `{ TYPE deftype, MODULE moduleinst, CODE func }`

* The [defined type](syntax-deftype) `deftype` must be [valid](valid-deftype) under an empty [context](context).
* The [module instance](syntax-moduleinst) `moduleinst` must be [valid](valid-moduleinst) with some [context](context) `C`.
* Under [context](context) `C`:
  * The [function](syntax-func) `func` must be [valid](valid-func) with some [defined type](syntax-deftype) `deftype'`.
  * The [defined type](syntax-deftype) `deftype'` must [match](match-deftype) `deftype`.
* Then the function instance is valid with [defined type](syntax-deftype) `deftype`.

```text
|- deftype : OK
S |- moduleinst : C
C |- func : deftype'
C |- deftypematch deftype' <: deftype
─────────────────────────────
S |- funcinst { TYPE deftype, MODULE moduleinst, CODE func } : deftype
```

#### Host Function Instances `{ TYPE deftype, HOSTFUNC hf }`

* The [defined type](syntax-deftype) `deftype` must be [valid](valid-deftype) under an empty [context](context).
* The [expansion](aux-expand-deftype) of [defined type](syntax-deftype) `deftype` must be some [function type](syntax-functype) `func [t1*] -> [t2*]`.
* For every [valid](valid-store) [store](syntax-store) `S1` [extending](extend-store) `S` and every sequence `val*` of [values](syntax-val) whose [types](valid-val) coincide with `t1*`:
  * [Executing](exec-invoke-host) `hf` in store `S1` with arguments `val*` has a non-empty set of possible outcomes.
  * For every element `R` of this set:
    * Either `R` must be `bot` (i.e., divergence).
    * Or `R` consists of a [valid](valid-store) [store](syntax-store) `S2` [extending](extend-store) `S1` and a [result](syntax-result) `result` whose [type](valid-result) coincides with `[t2*]`.
* Then the function instance is valid with [defined type](syntax-deftype) `deftype`.

```text
|- deftype : OK
deftype ≈ func [t1*] -> [t2*]
∀ S1, val* , (|- store S1 : OK ∧ |- storeextends S extendsto S1 ∧ S1 |- result val* : [t1*]) =>
  hf(S1; val*) ⊃ ∅ ∧
  ∀ R ∈ hf(S1; val*), R = bot ∨ ∃ S2, result ,
    (|- store S2 : OK ∧ |- storeextends S1 extendsto S2 ∧ S2 |- result result : [t2*]) ∧ R = (S2; result)
─────────────────────────────
S |- funcinst { TYPE deftype, HOSTFUNC hf } : deftype
```

> **Note:** This rule states that, if appropriate pre-conditions about store and arguments are satisfied, then executing the host function must satisfy appropriate post-conditions about store and results. The post-conditions match the ones in the [execution rule](exec-invoke-host) for invoking host functions.
>
> Any store under which the function is invoked is assumed to be an extension of the current store. That way, the function itself is able to make sufficient assumptions about future stores.

#### Data Instances `{ BYTES b* }`

* The data instance is valid.

```text
─────────────────────────────
S |- datainst { BYTES b* } : OK
```

#### Element Instances `{ TYPE t, REFS reff* }`

* The [reference type](syntax-reftype) `t` must be [valid](valid-reftype) under the empty [context](context).
* For each [reference](syntax-ref) `reff_i` in the elements `reff^n`:
  * The [reference](syntax-ref) `reff_i` must be [valid](valid-ref) with some [reference type](syntax-reftype) `t'_i`.
  * The [reference type](syntax-reftype) `t'_i` must [match](match-reftype) the [reference type](syntax-reftype) `t`.
* Then the element instance is valid with [reference type](syntax-reftype) `t`.

```text
|- reftype t : OK
(S |- val reff : t')*
(|- reftypematch t' <: t)*
─────────────────────────────
S |- eleminst { TYPE t, REFS reff* } : t
```

#### Structure Instances `{ TYPE dt, FIELDS fieldval* }`

* The [defined type](syntax-deftype) `dt` must be [valid](valid-deftype) under the empty [context](context).
* The [expansion](aux-expand-deftype) of `dt` must be a [structure type](syntax-structtype) `struct fieldtype*`.
* The length of the sequence of [field values](syntax-fieldval) `fieldval*` must be the same as the length of the sequence of [field types](syntax-fieldtype) `fieldtype*`.
* For each [field value](syntax-fieldval) `fieldval_i` in `fieldval*` and corresponding [field type](syntax-fieldtype) `fieldtype_i` in `fieldtype*`:
  * Let `fieldtype_i` be `mut storagetype_i`.
  * The [field value](syntax-fieldval) `fieldval_i` must be [valid](valid-fieldval) with [storage type](syntax-storagetype) `storagetype_i`.
* Then the structure instance is valid.

```text
|- deftype dt : OK
expanddt(dt) = struct (mut st)*
(S |- fieldval fv : st)*
─────────────────────────────
S |- structinst { TYPE dt, FIELDS fv* } : OK
```

#### Array Instances `{ TYPE dt, FIELDS fieldval* }`

* The [defined type](syntax-deftype) `dt` must be [valid](valid-deftype) under the empty [context](context).
* The [expansion](aux-expand-deftype) of `dt` must be an [array type](syntax-arraytype) `array fieldtype`.
* Let `fieldtype` be `mut storagetype`.
* For each [field value](syntax-fieldval) `fieldval_i` in `fieldval*`:
  * The [field value](syntax-fieldval) `fieldval_i` must be [valid](valid-fieldval) with [storage type](syntax-storagetype) `storagetype`.
* Then the array instance is valid.

```text
|- deftype dt : OK
expanddt(dt) = array (mut st)
(S |- fieldval fv : st)*
─────────────────────────────
S |- arrayinst { TYPE dt, FIELDS fv* } : OK
```

#### Field Values `fieldval`

* If `fieldval` is a [value](syntax-val) `val`, then:
  * The value `val` must be [valid](valid-val) with [value type](syntax-valtype) `t`.
  * Then the field value is valid with [value type](syntax-valtype) `t`.
* Else, `fieldval` is a [packed value](syntax-packval) `packval`:
  * Let `packtype.pack i` be the field value `fieldval`.
  * Then the field value is valid with [packed type](syntax-packtype) `packtype`.

```text
─────────────────────────────
S |- packval pt.pack i : pt
```

#### Exception Instances `{ TAG a, FIELDS val* }`

* The store entry `S.TAGS[a]` must exist.
* The [expansion](aux-expand-deftype) of the [tag type](syntax-tagtype) `S.TAGS[a].TYPE` must be some [function type](syntax-functype) `func [t*] -> []`.
* The [result type](syntax-resulttype) `[]` must be empty.
* The sequence `val*` of [values](syntax-val) must have the same length as the sequence `t*` of [value types](syntax-valtype).
* For each value `val_i` in `val*` and corresponding value type `t_i` in `t*`, the value `val_i` must be valid with type `t_i`.
* Then the exception instance is valid.

```text
S.TAGS[a].TYPE ≈ func [t*] -> []
(S |- val : t)*
─────────────────────────────
S |- exninst { TAG a, FIELDS val* } : OK
```

#### Export Instances `{ NAME name, ADDR externaddr }`

* The [external address](syntax-externaddr) `externaddr` must be [valid](valid-externaddr) with some [external type](syntax-externtype) `externtype`.
* Then the export instance is valid.

```text
S |- externaddr : externtype
─────────────────────────────
S |- exportinst { NAME name, ADDR externaddr } : OK
```

#### Module Instances `moduleinst`

* Each [defined type](syntax-deftype) `deftype_i` in `moduleinst.TYPES` must be [valid](valid-deftype) under the empty [context](context).
* For each [tag address](syntax-tagaddr) `tagaddr_i` in `moduleinst.TAGS`, the [external address](syntax-externaddr) `TAG tagaddr_i` must be [valid](valid-externaddr-tag) with some [external type](syntax-externtype) `tag tagtype_i`.
* For each [global address](syntax-globaladdr) `globaladdr_i` in `moduleinst.GLOBALS`, the [external address](syntax-externaddr) `GLOBAL globaladdr_i` must be [valid](valid-externaddr-global) with some [external type](syntax-externtype) `global globaltype_i`.
* For each [memory address](syntax-memaddr) `memaddr_i` in `moduleinst.MEMS`, the [external address](syntax-externaddr) `MEM memaddr_i` must be [valid](valid-externaddr-mem) with some [external type](syntax-externtype) `mem memtype_i`.
* For each [table address](syntax-tableaddr) `tableaddr_i` in `moduleinst.TABLES`, the [external address](syntax-externaddr) `TABLE tableaddr_i` must be [valid](valid-externaddr-table) with some [external type](syntax-externtype) `table tabletype_i`.
* For each [function address](syntax-funcaddr) `funcaddr_i` in `moduleinst.FUNCS`, the [external address](syntax-externaddr) `FUNC funcaddr_i` must be [valid](valid-externaddr-func) with some [external type](syntax-externtype) `func deftype_Fi`.
* For each [data address](syntax-dataaddr) `dataaddr_i` in `moduleinst.DATAS`, the [data instance](syntax-datainst) `S.DATAS[dataaddr_i]` must be [valid](valid-datainst) with `ok_i`.
* For each [element address](syntax-elemaddr) `elemaddr_i` in `moduleinst.ELEMS`, the [element instance](syntax-eleminst) `S.ELEMS[elemaddr_i]` must be [valid](valid-eleminst) with some [reference type](syntax-reftype) `reftype_i`.
* Each [export instance](syntax-exportinst) `exportinst_i` in `moduleinst.EXPORTS` must be [valid](valid-exportinst).
* For each [export instance](syntax-exportinst) `exportinst_i` in `moduleinst.EXPORTS`, the [name](syntax-name) `exportinst_i.NAME` must be different from any other name occurring in `moduleinst.EXPORTS`.
* Let `deftype*` be the concatenation of all `deftype_i` in order.
* Let `tagtype*` be the concatenation of all `tagtype_i` in order.
* Let `globaltype*` be the concatenation of all `globaltype_i` in order.
* Let `memtype*` be the concatenation of all `memtype_i` in order.
* Let `tabletype*` be the concatenation of all `tabletype_i` in order.
* Let `deftype_F*` be the concatenation of all `deftype_Fi` in order.
* Let `reftype*` be the concatenation of all `reftype_i` in order.
* Let `ok*` be the concatenation of all `ok_i` in order.
* Let `m` be the length of `moduleinst.FUNCS`.
* Let `x*` be the sequence of [function indices](syntax-funcidx) from `0` to `k-1` for some `k` that is smaller than or equal to the number of functions in the instance.
* Then the module instance is valid with [context](context) `{ TYPES deftype*, TAGS tagtype*, GLOBALS globaltype*, MEMS memtype*, TABLES tabletype*, FUNCS deftype_F*, DATAS ok*, ELEMS reftype*, REFS x* }`.

```text
(|- deftype : OK)*
(S |- externaddr TAG tagaddr : tag tagtype)*
(S |- externaddr GLOBAL globaladdr : global globaltype)*
(S |- externaddr FUNC funcaddr : func deftype_F)*
(S |- externaddr MEM memaddr : mem memtype)*
(S |- externaddr TABLE tableaddr : table tabletype)*
(S |- datainst S.DATAS[dataaddr] : ok)*
(S |- eleminst S.ELEMS[elemaddr] : reftype)*
(S |- exportinst exportinst : OK)*
(exportinst.NAME)* disjoint
k <= |funcaddr*|
─────────────────────────────
S |- moduleinst {
  TYPES deftype*,
  TAGS tagaddr*,
  GLOBALS globaladdr*,
  MEMS memaddr*,
  TABLES tableaddr*,
  FUNCS funcaddr*,
  DATAS dataaddr*,
  ELEMS elemaddr*,
  EXPORTS exportinst* } : {
    TYPES deftype*,
    TAGS tagtype*,
    GLOBALS globaltype*,
    MEMS memtype*,
    TABLES tabletype*,
    FUNCS deftype_F*,
    DATAS ok*,
    ELEMS reftype*,
    REFS 0 .. k-1 }
```

> **Note:** The field `C.refs` from the resulting context is meant to be constructed to contain all [function indices](syntax-funcidx) from the module. However, modules and their instances can contain more than `2^32` functions (at most `2^32` definitions plus `2^32` imports), while the highest representable function index is `2^32 - 1`. The variable `k` in the rule hence allows picking an upper limit for `C.refs` that is smaller than the total number of functions, in case that is necessary for `C.refs` to be syntactically well-formed. In practice, `k = min(|funcaddr*|, 2^32)` always is the maximally permissive choice.

### Configuration Validity

To relate the WebAssembly [type system](valid) to its [execution semantics](exec), the [typing rules for instructions](valid-instr) must be extended to [configurations](syntax-config) `S;T`, which relates the [store](syntax-store) to execution [threads](syntax-thread).

Configurations and threads are classified by their [result type](syntax-resulttype). In addition to the store `S`, threads are typed under a *return type* `resulttype?`, which controls whether and with which type a [return](syntax-return) instruction is allowed. This type is absent (`ε`) except for instruction sequences inside an administrative [frame](syntax-frame) instruction.

Finally, [frames](syntax-frame) are classified with *frame contexts*, which extend the [module contexts](module-context) of a frame's associated [module instance](syntax-moduleinst) with the [locals](syntax-local) that the frame contains.

#### Configurations `S;T`

* The [store](syntax-store) `S` must be [valid](valid-store).
* Under no allowed return type, the [thread](syntax-thread) `T` must be [valid](valid-thread) with some [result type](syntax-resulttype) `[t*]`.
* Then the configuration is valid with the [result type](syntax-resulttype) `[t*]`.

```text
|- store S : OK
S; ε |- thread T : [t*]
─────────────────────────────
|- config S; T : [t*]
```

#### Threads `F;instr*`

* Let `resulttype?` be the current allowed return type.
* The [frame](syntax-frame) `F` must be [valid](valid-frame) with a [context](context) `C`.
* Let `C'` be the same [context](context) as `C`, but with [return](syntax-return) set to `resulttype?`.
* Under context `C'`, the instruction sequence `instr*` must be [valid](valid-instrs) with some type `[] -> [t*]`.
* Then the thread is valid with the [result type](syntax-resulttype) `[t*]`.

```text
S |- frame F : C
S; C, RETURN resulttype? |- instrs instr* : [] -> [t*]
─────────────────────────────
S; resulttype? |- thread F; instr* : [t*]
```

#### Frames `{ LOCALS val*, MODULE moduleinst }`

* The [module instance](syntax-moduleinst) `moduleinst` must be [valid](valid-moduleinst) with some [module context](module-context) `C`.
* Each [value](syntax-val) `val_i` in `val*` must be [valid](valid-val) with some [value type](syntax-valtype) `t_i`.
* Let `t*` be the concatenation of all `t_i` in order.
* Let `C'` be the same [context](context) as `C`, but with the [value types](syntax-valtype) `t*` prepended to the [locals](syntax-local) list.
* Then the frame is valid with [frame context](frame-context) `C'`.

```text
S |- moduleinst : C
(S |- val : t)*
─────────────────────────────
S |- frame { LOCALS val*, MODULE moduleinst } : (C, LOCALS t*)
```

### Administrative Instructions

Typing rules for [administrative instructions](syntax-instr-admin) are specified as follows. In addition to the [context](context) `C`, typing of these instructions is defined under a given [store](syntax-store) `S`.

To that end, all previous typing judgements `C |- prop` are generalized to include the store, as in `S; C |- prop`, by implicitly adding `S` to all rules -- `S` is never modified by the pre-existing rules, but it is accessed in the extra rules for [administrative instructions](valid-instr-admin) given below.

#### `trap`

* The instruction is valid with any [valid](valid-instrtype) [instruction type](syntax-instrtype) of the form `[t1*] -> [t2*]`.

```text
C |- instrtype [t1*] -> [t2*] : OK
─────────────────────────────
S; C |- admininstr trap : [t1*] -> [t2*]
```

#### `val`

* The value `val` must be valid with [value type](syntax-valtype) `t`.
* Then it is valid as an instruction with type `[] -> [t]`.

```text
S |- val : t
─────────────────────────────
S; C |- admininstr val : [] -> [t]
```

#### `label_n {instr0*} instr*`

* The instruction sequence `instr0*` must be [valid](valid-instrs) with some type `[t1^n] ->_{x*} [t2*]`.
* Let `C'` be the same [context](context) as `C`, but with the [result type](syntax-resulttype) `[t1^n]` prepended to the [labels](syntax-label) list.
* Under context `C'`, the instruction sequence `instr*` must be [valid](valid-instrs) with type `[] ->_{x'*} [t2*]`.
* Then the compound instruction is valid with type `[] -> [t2*]`.

```text
S; C |- instrs instr0* : [t1^n] ->_{x*} [t2*]
S; C, LABELS [t1^n] |- instrs instr* : [] ->_{x'*} [t2*]
─────────────────────────────
S; C |- admininstr label_n {instr0*} instr* : [] -> [t2*]
```

#### `frame_n {F} instr*`

* Under the [valid](valid-resulttype) return type `[t^n]`, the [thread](syntax-frame) `F; instr*` must be [valid](valid-frame) with [result type](syntax-resulttype) `[t^n]`.
* Then the compound instruction is valid with type `[] -> [t^n]`.

```text
{} |- resulttype [t^n] : OK
S; [t^n] |- instrs F; instr* : [t^n]
─────────────────────────────
S; C |- admininstr frame_n {F} instr* : [] -> [t^n]
```

#### `handler_n {catch*} instr*`

* For every [catch clause](syntax-catch) `catch_i` in `catch*`, `catch_i` must be [valid](valid-catch).
* The instruction sequence `instr*` must be [valid](valid-instrs) with some type `[t1*] -> [t2*]`.
* Then the compound instruction is valid with type `[t1*] -> [t2*]`.

```text
(C |- catch : OK)*
S; C |- instrs instr* : [] -> [t*]
─────────────────────────────
S; C |- admininstr handler_n {catch*} instr* : [] -> [t*]
```

### Store Extension

Programs can mutate the [store](syntax-store) and its contained instances. Any such modification must respect certain invariants, such as not removing allocated instances or changing immutable definitions. While these invariants are inherent to the execution semantics of WebAssembly [instructions](exec-instr) and [modules](exec-instantiation), [host functions](syntax-hostfunc) do not automatically adhere to them. Consequently, the required invariants must be stated as explicit constraints on the [invocation](exec-invoke-host) of host functions. Soundness only holds when the [embedder](embedder) ensures these constraints.

The necessary constraints are codified by the notion of store *extension*: a store state `S'` extends state `S`, written `S extends S'`, when the following rules hold.

> **Note:** Extension does not imply that the new store is valid, which is defined separately [above](valid-store).

#### Store `S`

* The length of `S.TAGS` must not shrink.
* The length of `S.GLOBALS` must not shrink.
* The length of `S.MEMS` must not shrink.
* The length of `S.TABLES` must not shrink.
* The length of `S.FUNCS` must not shrink.
* The length of `S.DATAS` must not shrink.
* The length of `S.ELEMS` must not shrink.
* The length of `S.STRUCTS` must not shrink.
* The length of `S.ARRAYS` must not shrink.
* The length of `S.EXNS` must not shrink.
* For each [tag instance](syntax-taginst) `taginst_i` in the original `S.TAGS`, the new tag instance must be an [extension](extend-taginst) of the old.
* For each [global instance](syntax-globalinst) `globalinst_i` in the original `S.GLOBALS`, the new global instance must be an [extension](extend-globalinst) of the old.
* For each [memory instance](syntax-meminst) `meminst_i` in the original `S.MEMS`, the new memory instance must be an [extension](extend-meminst) of the old.
* For each [table instance](syntax-tableinst) `tableinst_i` in the original `S.TABLES`, the new table instance must be an [extension](extend-tableinst) of the old.
* For each [function instance](syntax-funcinst) `funcinst_i` in the original `S.FUNCS`, the new function instance must be an [extension](extend-funcinst) of the old.
* For each [data instance](syntax-datainst) `datainst_i` in the original `S.DATAS`, the new data instance must be an [extension](extend-datainst) of the old.
* For each [element instance](syntax-eleminst) `eleminst_i` in the original `S.ELEMS`, the new element instance must be an [extension](extend-eleminst) of the old.
* For each [structure instance](syntax-structinst) `structinst_i` in the original `S.STRUCTS`, the new structure instance must be an [extension](extend-structinst) of the old.
* For each [array instance](syntax-arrayinst) `arrayinst_i` in the original `S.ARRAYS`, the new array instance must be an [extension](extend-arrayinst) of the old.
* For each [exception instance](syntax-exninst) `exninst_i` in the original `S.EXNS`, the new exception instance must be an [extension](extend-datainst) of the old.

```text
S1.TAGS = taginst1*        S2.TAGS = taginst1'* taginst2*        (|- taginstextends taginst1 extends taginst1')*
S1.GLOBALS = globalinst1*  S2.GLOBALS = globalinst1'* globalinst2*  (|- globalinstextends globalinst1 extends globalinst1')*
S1.MEMS = meminst1*        S2.MEMS = meminst1'* meminst2*        (|- meminstextends meminst1 extends meminst1')*
S1.TABLES = tableinst1*    S2.TABLES = tableinst1'* tableinst2*  (|- tableinstextends tableinst1 extends tableinst1')*
S1.FUNCS = funcinst1*      S2.FUNCS = funcinst1'* funcinst2*     (|- funcinstextends funcinst1 extends funcinst1')*
S1.DATAS = datainst1*      S2.DATAS = datainst1'* datainst2*     (|- datainstextends datainst1 extends datainst1')*
S1.ELEMS = eleminst1*      S2.ELEMS = eleminst1'* eleminst2*     (|- eleminstextends eleminst1 extends eleminst1')*
S1.STRUCTS = structinst1*  S2.STRUCTS = structinst1'* structinst2*  (|- structinstextends structinst1 extends structinst1')*
S1.ARRAYS = arrayinst1*    S2.ARRAYS = arrayinst1'* arrayinst2*  (|- arrayinstextends arrayinst1 extends arrayinst1')*
S1.EXNS = exninst1*        S2.EXNS = exninst1'* exninst2*        (|- exninstextends exninst1 extends exninst1')*
─────────────────────────────
|- storeextends S1 extends S2
```

#### Tag Instance `taginst`

* A tag instance must remain unchanged.

```text
─────────────────────────────
|- taginstextends taginst extends taginst
```

#### Global Instance `globalinst`

* The [global type](syntax-globaltype) `globalinst.TYPE` must remain unchanged.
* Let `mut t` be the structure of `globalinst.TYPE`.
* If `mut` is empty, then the [value](syntax-val) `globalinst.VALUE` must remain unchanged.

```text
mut = mut ∨ val1 = val2
─────────────────────────────
|- globalinstextends { TYPE (mut t), VALUE val1 } extends { TYPE (mut t), VALUE val2 }
```

#### Memory Instance `meminst`

* The [memory type](syntax-memtype) `meminst.TYPE` must remain unchanged.
* The length of `meminst.BYTES` must not shrink.

```text
n1 <= n2
─────────────────────────────
|- meminstextends { TYPE mt, BYTES b1^n1 } extends { TYPE mt, BYTES b2^n2 }
```

#### Table Instance `tableinst`

* The [table type](syntax-tabletype) `tableinst.TYPE` must remain unchanged.
* The length of `tableinst.REFS` must not shrink.

```text
n1 <= n2
─────────────────────────────
|- tableinstextends { TYPE tt, REFS (fa1?)^n1 } extends { TYPE tt, REFS (fa2?)^n2 }
```

#### Function Instance `funcinst`

* A function instance must remain unchanged.

```text
─────────────────────────────
|- funcinstextends funcinst extends funcinst
```

#### Data Instance `datainst`

* The list `datainst.BYTES` must:
  * either remain unchanged,
  * or shrink to length `0`.

```text
─────────────────────────────
|- datainstextends { BYTES b* } extends { BYTES b* }
```

```text
─────────────────────────────
|- datainstextends { BYTES b* } extends { BYTES ε }
```

#### Element Instance `eleminst`

* The [reference type](syntax-reftype) `eleminst.TYPE` must remain unchanged.
* The list `eleminst.REFS` must:
  * either remain unchanged,
  * or shrink to length `0`.

```text
─────────────────────────────
|- eleminstextends { TYPE t, REFS a* } extends { TYPE t, REFS a* }
```

```text
─────────────────────────────
|- eleminstextends { TYPE t, REFS a* } extends { TYPE t, REFS ε }
```

#### Structure Instance `structinst`

* The [defined type](syntax-deftype) `structinst.TYPE` must remain unchanged.
* Assert: due to [store well-formedness](valid-structinst), the [expansion](aux-expand-deftype) of `structinst.TYPE` is a [structure type](syntax-structtype).
* Let `struct fieldtype*` be the [expansion](aux-expand-deftype) of `structinst.TYPE`.
* The length of the list `structinst.FIELDS` must remain unchanged.
* Assert: due to [store well-formedness](valid-structinst), the length of `structinst.FIELDS` is the same as the length of `fieldtype*`.
* For each [field value](syntax-fieldval) `fieldval_i` in `structinst.FIELDS` and corresponding [field type](syntax-fieldtype) `fieldtype_i` in `fieldtype*`:
  * Let `mut_i st_i` be the structure of `fieldtype_i`.
  * If `mut_i` is empty, then the [field value](syntax-fieldval) `fieldval_i` must remain unchanged.

```text
(mut = mut ∨ fieldval1 = fieldval2)*
─────────────────────────────
|- structinstextends { TYPE (mut st)*, FIELDS fieldval1* } extends { TYPE (mut st)*, FIELDS fieldval2* }
```

#### Array Instance `arrayinst`

* The [defined type](syntax-deftype) `arrayinst.TYPE` must remain unchanged.
* Assert: due to [store well-formedness](valid-arrayinst), the [expansion](aux-expand-deftype) of `arrayinst.TYPE` is an [array type](syntax-arraytype).
* Let `array fieldtype` be the [expansion](aux-expand-deftype) of `arrayinst.TYPE`.
* The length of the list `arrayinst.FIELDS` must remain unchanged.
* Let `mut st` be the structure of `fieldtype`.
* If `mut` is empty, then the sequence of [field values](syntax-fieldval) `arrayinst.FIELDS` must remain unchanged.

```text
mut = mut ∨ fieldval1* = fieldval2*
─────────────────────────────
|- arrayinstextends { TYPE (mut st), FIELDS fieldval1* } extends { TYPE (mut st), FIELDS fieldval2* }
```

#### Exception Instance `exninst`

* An exception instance must remain unchanged.

```text
─────────────────────────────
|- exninstextends exninst extends exninst
```

### Theorems

Given the definition of [valid configurations](valid-config), the standard soundness theorems hold. [^cite-cpp2018] [^cite-fm2021]

**Theorem (Preservation).** If a [configuration](syntax-config) `S;T` is [valid](valid-config) with [result type](syntax-resulttype) `[t*]` (i.e., `|- config S;T : [t*]`), and steps to `S';T'` (i.e., `S;T stepto S';T'`), then `S';T'` is a valid configuration with the same result type (i.e., `|- config S';T' : [t*]`). Furthermore, `S'` is an [extension](extend-store) of `S` (i.e., `|- storeextends S extends S'`).

A *terminal* [thread](syntax-thread) is one whose sequence of [instructions](syntax-instr) is a [result](syntax-result). A terminal configuration is a configuration whose thread is terminal.

**Theorem (Progress).** If a [configuration](syntax-config) `S;T` is [valid](valid-config) (i.e., `|- config S;T : [t*]` for some [result type](syntax-resulttype) `[t*]`), then either it is terminal, or it can step to some configuration `S';T'` (i.e., `S;T stepto S';T'`).

From Preservation and Progress the soundness of the WebAssembly type system follows directly.

**Corollary (Soundness).** If a [configuration](syntax-config) `S;T` is [valid](valid-config) (i.e., `|- config S;T : [t*]` for some [result type](syntax-resulttype) `[t*]`), then it either diverges or takes a finite number of steps to reach a terminal configuration `S';T'` (i.e., `S;T stepto* S';T'`) that is valid with the same result type (i.e., `|- config S';T' : [t*]`) and where `S'` is an [extension](extend-store) of `S` (i.e., `|- storeextends S extends S'`).

In other words, every thread in a valid configuration either runs forever, traps, throws an exception, or terminates with a result that has the expected type. Consequently, given a [valid store](valid-store), no computation defined by [instantiation](exec-instantiation) or [invocation](exec-invocation) of a valid module can "crash" or otherwise (mis)behave in ways not covered by the [execution](exec) semantics given in this specification.

[^cite-pldi2017]: The formalization and theorems are derived from the following article: Andreas Haas, Andreas Rossberg, Derek Schuff, Ben Titzer, Dan Gohman, Luke Wagner, Alon Zakai, JF Bastien, Michael Holman. PLDI2017. Proceedings of the 38th ACM SIGPLAN Conference on Programming Language Design and Implementation (PLDI 2017). ACM 2017.

[^cite-cpp2018]: A machine-verified version of the formalization and soundness proof of the PLDI 2017 paper is described in the following article: Conrad Watt. CPP2018. Proceedings of the 7th ACM SIGPLAN Conference on Certified Programs and Proofs (CPP 2018). ACM 2018.

[^cite-fm2021]: Machine-verified formalizations and soundness proofs of the semantics from the official specification are described in the following article: Conrad Watt, Xiaojia Rao, Jean Pichon-Pharabod, Martin Bodin, Philippa Gardner. FM2021. Proceedings of the 24th International Symposium on Formal Methods (FM 2021). Springer 2021.

## Type System Properties

### Principal Types

The [type system](type-system) of WebAssembly features both [subtyping](match) and simple forms of [polymorphism](polymorphism) for [instruction types](syntax-instrtype). That has the effect that every instruction or instruction sequence can be classified with multiple different instruction types.

However, the typing rules still allow deriving *principal types* for instruction sequences. That is, every valid instruction sequence has one particular type scheme, possibly containing some unconstrained place holder *type variables*, that is a subtype of all its valid instruction types, after substituting its type variables with suitable specific types.

Moreover, when deriving an instruction type in a "forward" manner, i.e., the *input* of the instruction sequence is already fixed to specific types, then it has a principal *output* type expressible without type variables, up to a possibly [polymorphic stack](polymorphism) bottom representable with one single variable. In other words, "forward" principal types are effectively *closed*.

> **Note:** For example, in isolation, the instruction `ref.as_non_null` has the type `[(ref null ht)] -> [(ref ht)]` for any choice of valid [heap type](syntax-type) `ht`. Moreover, if the input type `[(ref null ht)]` is already determined, i.e., a specific `ht` is given, then the output type `[(ref ht)]` is fully determined as well.
>
> The implication of the latter property is that a validator for *complete* instruction sequences (as they occur in valid modules) can be implemented with a simple left-to-right [algorithm](algo-valid) that does not require the introduction of type variables.
>
> A typing algorithm capable of handling *partial* instruction sequences (as might be considered for program analysis or program manipulation) needs to introduce type variables and perform substitutions, but it does not need to perform backtracking or record any non-syntactic constraints on these type variables.

Technically, the [syntax](syntax-type) of [heap](syntax-heaptype), [value](syntax-valtype), and [result](syntax-resulttype) types can be enriched with type variables as follows:

```text
null      ::= null? | α_null

heaptype  ::= ... | α_heaptype

reftype   ::= ref null heaptype

valtype   ::= ... | α_valtype | α_numvectype

resulttype ::= [α_valtype*^? valtype*]
```

where each `α_xyz` ranges over a set of type variables for syntactic class `xyz`, respectively. The special class `numvectype` is defined as `numtype | vectype | bot`, and is only needed to handle unannotated [select](syntax-select) instructions.

A type is *closed* when it does not contain any type variables, and *open* otherwise. A *type substitution* `σ` is a finite mapping from type variables to closed types of the respective syntactic class. When applied to an open type, it replaces the type variables `α` from its domain with the respective `σ(α)`.

**Theorem (Principal Types).** If an instruction sequence `instr*` is [valid](valid-config) with some closed [instruction type](syntax-instrtype) `instrtype` (i.e., `C |- instrs instr* : instrtype`), then it is also valid with a possibly open instruction type `instrtype_min` (i.e., `C |- instrs instr* : instrtype_min`), such that for *every* closed type `instrtype'` with which `instr*` is valid (i.e., for all `C |- instrs instr* : instrtype'`), there exists a substitution `σ`, such that `σ(instrtype_min)` is a subtype of `instrtype'` (i.e., `C |- instrtypematch σ(instrtype_min) <: instrtype'`). Furthermore, `instrtype_min` is unique up to the choice of type variables.

**Theorem (Closed Principal Forward Types).** If closed input type `[t1*]` is given and the instruction sequence `instr*` is [valid](valid-config) with [instruction type](syntax-instrtype) `[t1*] ->_{x*} [t2*]` (i.e., `C |- instrs instr* : [t1*] ->_{x*} [t2*]`), then it is also valid with instruction type `[t1*] ->_{x*} [α_valtype* t*]` (i.e., `C |- instrs instr* : [t1*] ->_{x*} [α_valtype* t*]`), where all `t*` are closed, such that for *every* closed result type `[t'2*]` with which `instr*` is valid (i.e., for all `C |- instrs instr* : [t1*] ->_{x*} [t'2*]`), there exists a substitution `σ`, such that `[t'2*] = [σ(α_valtype*) t*]`.

### Type Lattice

The [Principal Types](principality) property depends on the existence of a *greatest lower bound* for any pair of types.

**Theorem (Greatest Lower Bounds for Value Types).** For any two value types `t1` and `t2` that are [valid](valid-valtype) (i.e., `C |- valtype t1 : OK` and `C |- valtype t2 : OK`), there exists a valid value type `t` that is a subtype of both `t1` and `t2` (i.e., `C |- valtype t : OK` and `C |- valtypematch t <: t1` and `C |- valtypematch t <: t2`), such that *every* valid value type `t'` that also is a subtype of both `t1` and `t2` (i.e., for all `C |- valtype t' : OK` and `C |- valtypematch t' <: t1` and `C |- valtypematch t' <: t2`), is a subtype of `t` (i.e., `C |- valtypematch t' <: t`).

> **Note:** The greatest lower bound of two types may be BOT.
