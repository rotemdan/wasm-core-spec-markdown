## Values

### Value Typing

For the purpose of checking argument values against the parameter types of exported functions, values are classified by value types. The following auxiliary typing rules specify this typing relation relative to a store `S` in which possibly referenced addresses live.

#### Numeric Values

The number value `(nt.const c)` is valid with the number type `nt`.

```text
s |- nt.const c : nt
```

#### Vector Values

The vector value `(vt.vconst c)` is valid with the vector type `vt`.

```text
s |- vt.vconst c : vt
```

#### Null References

The reference value `ref.null` is valid with the reference type `(ref null bot)`.

```text
s |- ref.null : ref null bot
```

#### Scalar References

The reference value `(ref.i31 i)` is valid with the reference type `(ref i31)`.

```text
s |- ref.i31 i : ref i31
```

#### Structure References

The reference value `(ref.struct a)` is valid with the reference type `(ref dt)` if:

* The structure instance `s.STRUCTS[a]` exists.
* The defined type `s.STRUCTS[a].ITYPE` is of the form `dt`.

```text
s.STRUCTS[a].ITYPE = dt
─────────────────────────────
s |- ref.struct a : ref dt
```

#### Array References

The reference value `(ref.array a)` is valid with the reference type `(ref dt)` if:

* The array instance `s.ARRAYS[a]` exists.
* The defined type `s.ARRAYS[a].ITYPE` is of the form `dt`.

```text
s.ARRAYS[a].ITYPE = dt
─────────────────────────────
s |- ref.array a : ref dt
```

#### Exception References

The reference value `(ref.exn a)` is valid with the reference type `(ref exn)` if:

* The exception instance `s.EXNS[a]` exists.

```text
s.EXNS[a] = exn
─────────────────────────────
s |- ref.exn a : ref exn
```

#### Function References

The reference value `(ref.func a)` is valid with the reference type `(ref dt)` if:

* The function instance `s.FUNCS[a]` exists.
* The defined type `s.FUNCS[a].ITYPE` is of the form `dt`.

```text
s.FUNCS[a].ITYPE = dt
─────────────────────────────
s |- ref.func a : ref dt
```

#### Host References

The reference value `(ref.host a)` is valid with the reference type `(ref any)`.

```text
s |- ref.host a : ref any
```

> **Note:** A bare host reference is considered internalized.

#### External References

The reference value `(ref.extern reff)` is valid with the reference type `(ref extern)` if:

* The reference value `reff` is valid with the reference type `(ref any)`.
* The reference value `reff` is not of the form `ref.null`.

```text
s |- reff : ref any
reff != ref.null
─────────────────────────────
s |- ref.extern reff : ref extern
```

#### Subsumption

The reference value `reff` is valid with the reference type `rt` if:

* The reference value `reff` is valid with the reference type `rt'`.
* Under the context `{ RETURN ε }`, the reference type `rt` is valid.
* The reference type `rt'` matches the reference type `rt`.

```text
s |- reff : rt'
{ } |- rt : OK
{ } |- rt' <: rt
─────────────────────────────
s |- reff : rt
```

### External Typing

For the purpose of checking external addresses against imports, such values are classified by external types. The following auxiliary typing rules specify this typing relation relative to a store `S` in which the referenced instances live.

#### Functions

The external address `(func a)` is valid with the external type `(func funcinst.ITYPE)` if:

* The function instance `s.FUNCS[a]` exists.
* The function instance `s.FUNCS[a]` is of the form `funcinst`.

```text
s.FUNCS[a] = funcinst
─────────────────────────────
s |- func a : func funcinst.ITYPE
```

#### Tables

The external address `(table a)` is valid with the external type `(table tableinst.ITYPE)` if:

* The table instance `s.TABLES[a]` exists.
* The table instance `s.TABLES[a]` is of the form `tableinst`.

```text
s.TABLES[a] = tableinst
─────────────────────────────
s |- table a : table tableinst.ITYPE
```

#### Memories

The external address `(mem a)` is valid with the external type `(mem meminst.ITYPE)` if:

* The memory instance `s.MEMS[a]` exists.
* The memory instance `s.MEMS[a]` is of the form `meminst`.

```text
s.MEMS[a] = meminst
─────────────────────────────
s |- mem a : mem meminst.ITYPE
```

#### Globals

The external address `(global a)` is valid with the external type `(global globalinst.ITYPE)` if:

* The global instance `s.GLOBALS[a]` exists.
* The global instance `s.GLOBALS[a]` is of the form `globalinst`.

```text
s.GLOBALS[a] = globalinst
─────────────────────────────
s |- global a : global globalinst.ITYPE
```

#### Tags

The external address `(tag a)` is valid with the external type `(tag taginst.ITYPE)` if:

* The tag instance `s.TAGS[a]` exists.
* The tag instance `s.TAGS[a]` is of the form `taginst`.

```text
s.TAGS[a] = taginst
─────────────────────────────
s |- tag a : tag taginst.ITYPE
```

#### Subsumption

The external address `externaddr` is valid with the external type `xt` if:

* The external address `externaddr` is valid with the external type `xt'`.
* Under the context `{ RETURN ε }`, the external type `xt` is valid.
* The external type `xt'` matches the external type `xt`.

```text
s |- externaddr : xt'
{ } |- xt : OK
{ } |- xt' <: xt
─────────────────────────────
s |- externaddr : xt
```
