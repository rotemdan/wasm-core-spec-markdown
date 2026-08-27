## Modules

For modules, the execution semantics primarily defines instantiation, which allocates instances for a module and its contained definitions, initializes memories and tables from contained data and element segments, and invokes the start function if present. It also includes invocation of exported functions.

### Allocation

New instances of tags, globals, memories, tables, functions, data segments, and element segments are allocated in a store `s`, as defined by the following auxiliary functions.

#### Tags

#### alloc_tag(s, tagtype)

1. Let `taginst` be the tag instance `{ ITYPE tagtype }`.
2. Let `a` be the length of `s.TAGS`.
3. Append `taginst` to `s.TAGS`.
4. Return `a`.

```text
alloctag(s, tagtype) = (s ⊕ { TAGS taginst }, |s.TAGS|)
  (if taginst = { ITYPE tagtype })
```

#### Globals

#### alloc_global(s, globaltype, val)

1. Let `globalinst` be the global instance `{ ITYPE globaltype, IVALUE val }`.
2. Let `a` be the length of `s.GLOBALS`.
3. Append `globalinst` to `s.GLOBALS`.
4. Return `a`.

```text
allocglobal(s, globaltype, val) = (s ⊕ { GLOBALS globalinst }, |s.GLOBALS|)
  (if globalinst = { ITYPE globaltype, IVALUE val })
```

#### Memories

#### alloc_mem(s, at [ i .. j? ] page)

1. Let `meminst` be the memory instance `{ ITYPE (at [ i .. j? ] page), IBYTES (0x00)^(i * 64 Ki) }`.
2. Let `a` be the length of `s.MEMS`.
3. Append `meminst` to `s.MEMS`.
4. Return `a`.

```text
allocmem(s, at [ i .. j? ] page) = (s ⊕ { MEMS meminst }, |s.MEMS|)
  (if meminst = { ITYPE (at [ i .. j? ] page), IBYTES (0x00)^(i * 64 Ki) })
```

#### Tables

#### alloc_table(s, at [ i .. j? ] rt, reff)

1. Let `tableinst` be the table instance `{ ITYPE (at [ i .. j? ] rt), IREFS reff^i }`.
2. Let `a` be the length of `s.TABLES`.
3. Append `tableinst` to `s.TABLES`.
4. Return `a`.

```text
alloctable(s, at [ i .. j? ] rt, reff) = (s ⊕ { TABLES tableinst }, |s.TABLES|)
  (if tableinst = { ITYPE (at [ i .. j? ] rt), IREFS reff^i })
```

#### Functions

#### alloc_func(s, deftype, funccode, moduleinst)

1. Let `funcinst` be the function instance `{ ITYPE deftype, IMODULE moduleinst, ICODE funccode }`.
2. Let `a` be the length of `s.FUNCS`.
3. Append `funcinst` to `s.FUNCS`.
4. Return `a`.

```text
allocfunc(s, deftype, funccode, moduleinst) = (s ⊕ { FUNCS funcinst }, |s.FUNCS|)
  (if funcinst = { ITYPE deftype, IMODULE moduleinst, ICODE funccode })
```

#### Data segments

#### alloc_data(s, ok, byte*)

1. Let `datainst` be the data instance `{ IBYTES byte* }`.
2. Let `a` be the length of `s.DATAS`.
3. Append `datainst` to `s.DATAS`.
4. Return `a`.

```text
allocdata(s, ok, byte*) = (s ⊕ { DATAS datainst }, |s.DATAS|)
  (if datainst = { IBYTES byte* })
```

#### Element segments

#### alloc_elem(s, elemtype, reff*)

1. Let `eleminst` be the element instance `{ ITYPE elemtype, IREFS reff* }`.
2. Let `a` be the length of `s.ELEMS`.
3. Append `eleminst` to `s.ELEMS`.
4. Return `a`.

```text
allocelem(s, elemtype, reff*) = (s ⊕ { ELEMS eleminst }, |s.ELEMS|)
  (if eleminst = { ITYPE elemtype, IREFS reff* })
```

#### Growing memories

#### grow_mem(meminst, n)

1. Let `{ ITYPE (at [ i .. j? ] page), IBYTES b* }` be the destructuring of `meminst`.
2. Let `i'` be `|b*| / (64 Ki) + n`.
3. If not `(i' <= j)?`, then:
   1. Fail.
4. If `i' <= 2^(|at| - 16)`, then:
   1. Let `meminst'` be the memory instance `{ ITYPE (at [ i' .. j? ] page), IBYTES b* (0x00)^(n * 64 Ki) }`.
   2. Return `meminst'`.
5. Fail.

```text
growmem(meminst, n) = meminst'
  (if meminst = { ITYPE (at [ i .. j? ] page), IBYTES b* }
   ∧ meminst' = { ITYPE (at [ i' .. j? ] page), IBYTES b* (0x00)^(n * 64 Ki) }
   ∧ i' = |b*| / (64 Ki) + n
   ∧ (i' <= j)?
   ∧ i' <= 2^(|at| - 16))
```

#### Growing tables

#### grow_table(tableinst, n, r)

1. Let `{ ITYPE (at [ i .. j? ] rt), IREFS r'* }` be the destructuring of `tableinst`.
2. Let `i'` be `|r'*| + n`.
3. If not `(i' <= j)?`, then:
   1. Fail.
4. If `i' <= 2^|at| - 1`, then:
   1. Let `tableinst'` be the table instance `{ ITYPE (at [ i' .. j? ] rt), IREFS r'* r^n }`.
   2. Return `tableinst'`.
5. Fail.

```text
growtable(tableinst, n, r) = tableinst'
  (if tableinst = { ITYPE (at [ i .. j? ] rt), IREFS r'* }
   ∧ tableinst' = { ITYPE (at [ i' .. j? ] rt), IREFS r'* r^n }
   ∧ i' = |r'*| + n
   ∧ (i' <= j)?
   ∧ i' <= 2^|at| - 1)
```

#### Modules

#### alloc_module(s, module, externaddr*, val_g*, reff_t*, (reff_e*)*)

1. Let `(MODULE type* import* tag* global* mem* table* func* data* elem* start? export*)` be the destructuring of `module`.
2. Let `aa_i*` be `tagsxa(externaddr*)`.
3. Let `ga_i*` be `globalsxa(externaddr*)`.
4. Let `fa_i*` be `funcsxa(externaddr*)`.
5. Let `ma_i*` be `memsxa(externaddr*)`.
6. Let `ta_i*` be `tablesxa(externaddr*)`.
7. Let `fa*` be `|s.FUNCS| + i_f` for all `i_f` from `0` to `|func*| - 1`.
8. Let `tagtype*` be the tag type sequence `ε`.
9. For each `tag` in `tag*`, do:
   1. Let `(TAG tagtype)` be the destructuring of `tag`.
   2. Append `tagtype` to `tagtype*`.
10. Let `byte**` be the byte sequence sequence `ε`.
11. For each `data` in `data*`, do:
    1. Let `(DATA byte* datamode)` be the destructuring of `data`.
    2. Append `byte*` to `byte**`.
12. Let `globaltype*` be the global type sequence `ε`.
13. For each `global` in `global*`, do:
    1. Let `(GLOBAL globaltype expr_g)` be the destructuring of `global`.
    2. Append `globaltype` to `globaltype*`.
14. Let `tabletype*` be the table type sequence `ε`.
15. For each `table` in `table*`, do:
    1. Let `(TABLE tabletype expr_t)` be the destructuring of `table`.
    2. Append `tabletype` to `tabletype*`.
16. Let `memtype*` be the memory type sequence `ε`.
17. For each `mem` in `mem*`, do:
    1. Let `(MEMORY memtype)` be the destructuring of `mem`.
    2. Append `memtype` to `memtype*`.
18. Let `dt*` be `alloctype*(type*)`.
19. Let `elemtype*` be the reference type sequence `ε`.
20. For each `elem` in `elem*`, do:
    1. Let `(ELEM elemtype expr_e* elemmode)` be the destructuring of `elem`.
    2. Append `elemtype` to `elemtype*`.
21. Let `expr_f*` be the expression sequence `ε`.
22. Let `local**` be the local sequence sequence `ε`.
23. Let `x*` be the type index sequence `ε`.
24. For each `func` in `func*`, do:
    1. Let `(FUNC x local* expr_f)` be the destructuring of `func`.
    2. Append `expr_f` to `expr_f*`.
    3. Append `local*` to `local**`.
    4. Append `x` to `x*`.
25. Let `aa*` be `ε`.
26. For each `tagtype` in `tagtype*`, do:
    1. Let `aa` be the tag address `alloctag(s, tagtype[assignsubst dt*])`.
    2. Append `aa` to `aa*`.
27. Let `ga*` be `ε`.
28. For each `globaltype` in `globaltype*` and `val_g` in `val_g*`, do:
    1. Let `ga` be the global address `allocglobal(s, globaltype[assignsubst dt*], val_g)`.
    2. Append `ga` to `ga*`.
29. Let `ma*` be `ε`.
30. For each `memtype` in `memtype*`, do:
    1. Let `ma` be the memory address `allocmem(s, memtype[assignsubst dt*])`.
    2. Append `ma` to `ma*`.
31. Let `ta*` be `ε`.
32. For each `tabletype` in `tabletype*` and `reff_t` in `reff_t*`, do:
    1. Let `ta` be the table address `alloctable(s, tabletype[assignsubst dt*], reff_t)`.
    2. Append `ta` to `ta*`.
33. Let `xi*` be `ε`.
34. For each `export` in `export*`, do:
    1. Let `xi` be the export instance `allocexport(moduleinst, export)`.
    2. Append `xi` to `xi*`.
35. Let `da*` be `ε`.
36. For each `byte*` in `byte**`, do:
    1. Let `da` be the data address `allocdata(s, ok, byte*)`.
    2. Append `da` to `da*`.
37. Let `ea*` be `ε`.
38. For each `elemtype` in `elemtype*` and `reff_e*` in `reff_e**`, do:
    1. Let `ea` be the elem address `allocelem(s, elemtype[assignsubst dt*], reff_e*)`.
    2. Append `ea` to `ea*`.
39. Let `moduleinst` be the module instance `{ ITYPES dt*, IGLOBALS ga_i* ga*, IMEMS ma_i* ma*, ITABLES ta_i* ta*, IFUNCS fa_i* fa*, IDATAS da*, IELEMS ea*, IEXPORTS xi* }`.
40. Let `funcaddr_0*` be `ε`.
41. For each `expr_f` in `expr_f*`, `local*` in `local**`, and `x` in `x*`, do:
    1. Let `funcaddr_0` be the function address `allocfunc(s, dt*[x], FUNC x local* expr_f, moduleinst)`.
    2. Append `funcaddr_0` to `funcaddr_0*`.
42. Assert: Due to validation, `funcaddr_0* = fa*`.
43. Return `moduleinst`.

```text
allocmodule(s, module, externaddr*, val_g*, reff_t*, (reff_e*)*) = (s_7, moduleinst)
  (if module = MODULE type* import* tag* global* mem* table* func* data* elem* start? export*
   ∧ tag* = (TAG tagtype)*
   ∧ global* = (GLOBAL globaltype expr_g)*
   ∧ mem* = (MEMORY memtype)*
   ∧ table* = (TABLE tabletype expr_t)*
   ∧ func* = (FUNC x local* expr_f)*
   ∧ data* = (DATA byte* datamode)*
   ∧ elem* = (ELEM elemtype expr_e* elemmode)*
   ∧ aa_i* = tagsxa(externaddr*)
   ∧ ga_i* = globalsxa(externaddr*)
   ∧ ma_i* = memsxa(externaddr*)
   ∧ ta_i* = tablesxa(externaddr*)
   ∧ fa_i* = funcsxa(externaddr*)
   ∧ dt* = alloc_type*(type*)
   ∧ fa* = (|s.FUNCS| + i_f)^(i_f<|func*|)
   ∧ (s_1, aa*) = alloc_tag*(s, (tagtype[assignsubst dt*])*)
   ∧ (s_2, ga*) = alloc_global*(s_1, (globaltype[assignsubst dt*])*, val_g*)
   ∧ (s_3, ma*) = alloc_mem*(s_2, (memtype[assignsubst dt*])*)
   ∧ (s_4, ta*) = alloc_table*(s_3, (tabletype[assignsubst dt*])*, reff_t*)
   ∧ (s_5, da*) = alloc_data*(s_4, ok^(|data*|), (byte*)*)
   ∧ (s_6, ea*) = alloc_elem*(s_5, (elemtype[assignsubst dt*])*, (reff_e*)*)
   ∧ (s_7, fa*) = alloc_func*(s_6, (dt*[x])*, (FUNC x local* expr_f)*, moduleinst^(|func*|))
   ∧ xi* = alloc_export*({ ITAGS aa_i* aa*, IGLOBALS ga_i* ga*, IMEMS ma_i* ma*, ITABLES ta_i* ta*, IFUNCS fa_i* fa* }, export*)
   ∧ moduleinst = { ITYPES dt*, IGLOBALS ga_i* ga*, IMEMS ma_i* ma*, ITABLES ta_i* ta*, IFUNCS fa_i* fa*, IDATAS da*, IELEMS ea*, IEXPORTS xi* })
```

Here, the notation `allocX*` is shorthand for multiple allocations of object kind `X`, defined as follows:

```text
allocX*(s, ε, ε) = (s, ε)
allocX*(s, X X'*, Y Y'*) = (s_2, a a'*)
  (if (s_1, a) = allocX(X, Y, s, X, Y)
   ∧ (s_2, a'*) = allocX*(s_1, X'*, Y'*))
```

For types, however, allocation is defined in terms of rolling and substitution of all preceding types to produce a list of closed defined types:

#### alloc_type*(type''*)

1. If `type''* = ε`, then:
   1. Return `ε`.
2. Let `type'* type` be `type''*`.
3. Let `(TYPE rectype)` be the destructuring of `type`.
4. Let `deftype'*` be `alloc_type*(type'*)`.
5. Let `x` be the length of `deftype'*`.
6. Let `deftype*` be `rolldt_x*(rectype)[assignsubst deftype'*]`.
7. Return `deftype'* deftype*`.

```text
alloc_type*(ε) = ε
alloc_type*(type'* type) = deftype'* deftype*
  (if deftype'* = alloc_type*(type'*)
   ∧ type = TYPE rectype
   ∧ deftype* = rolldt_x*(rectype)[assignsubst deftype'*]
   ∧ x = |deftype'*|)
```

Finally, export instances are produced with the help of the following definition:

#### alloc_export(moduleinst, EXPORT name externidx)

1. If `externidx` is some `tag tagidx`, then:
   1. Let `(tag x)` be the destructuring of `externidx`.
   2. Return `{ NAME name, ADDR (tag moduleinst.ITAGS[x]) }`.
2. If `externidx` is some `global globalidx`, then:
   1. Let `(global x)` be the destructuring of `externidx`.
   2. Return `{ NAME name, ADDR (global moduleinst.IGLOBALS[x]) }`.
3. If `externidx` is some `mem memidx`, then:
   1. Let `(mem x)` be the destructuring of `externidx`.
   2. Return `{ NAME name, ADDR (mem moduleinst.IMEMS[x]) }`.
4. If `externidx` is some `table tableidx`, then:
   1. Let `(table x)` be the destructuring of `externidx`.
   2. Return `{ NAME name, ADDR (table moduleinst.ITABLES[x]) }`.
5. Assert: Due to validation, `externidx` is some `func funcidx`.
6. Let `(func x)` be the destructuring of `externidx`.
7. Return `{ NAME name, ADDR (func moduleinst.IFUNCS[x]) }`.

```text
allocexport(moduleinst, EXPORT name (tag x)) = { NAME name, ADDR (tag moduleinst.ITAGS[x]) }
allocexport(moduleinst, EXPORT name (global x)) = { NAME name, ADDR (global moduleinst.IGLOBALS[x]) }
allocexport(moduleinst, EXPORT name (mem x)) = { NAME name, ADDR (mem moduleinst.IMEMS[x]) }
allocexport(moduleinst, EXPORT name (table x)) = { NAME name, ADDR (table moduleinst.ITABLES[x]) }
allocexport(moduleinst, EXPORT name (func x)) = { NAME name, ADDR (func moduleinst.IFUNCS[x]) }
```

> **Note:** The definition of module allocation is mutually recursive with the allocation of its associated functions, because the resulting module instance is passed to the allocators as an argument, in order to form the necessary closures.
>
> In an implementation, this recursion is easily unraveled by mutating one or the other in a secondary step.

### Instantiation

Given a store `s`, a `module` is instantiated with a list of external addresses `externaddr*` supplying the required imports as follows.

Instantiation checks that the module is valid and the provided imports match the declared types, and may fail with an error otherwise.
Instantiation can also result in an exception or trap when initializing a table or memory from an active segment or when executing the start function.
It is up to the embedder to define how such conditions are reported.

#### instantiate(s, module, externaddr*)

1. If `module` is not valid, then:
   1. Fail.
2. Let `xt_i* ->M xt_e*` be the destructuring of the type of `module`.
3. Let `(MODULE type* import* tag* global* mem* table* func* data* elem* start? export*)` be the destructuring of `module`.
4. If `|externaddr*| != |xt_i*|`, then:
   1. Fail.
5. For all `externaddr` in `externaddr*`, and corresponding `xt_i` in `xt_i*`:
   1. If `externaddr` is not valid with type `xt_i`, then:
      1. Fail.
6. Let `instr_d*` be the concatenation of `rundata_i_d(data*[i_d])^(i_d<|data*|)`.
7. Let `instr_e*` be the concatenation of `runelem_i_e(elem*[i_e])^(i_e<|elem*|)`.
8. Let `moduleinst_0` be the module instance `{ ITYPES alloc_type*(type*), IGLOBALS globalsxa(externaddr*), IFUNCS funcsxa(externaddr*) (|s.FUNCS| + i_f)^(i_f<|func*|) }`.
9. Let `expr_t*` be the expression sequence `ε`.
10. For each `table` in `table*`, do:
    1. Let `(TABLE tabletype expr_t)` be the destructuring of `table`.
    2. Append `expr_t` to `expr_t*`.
11. Let `expr_g*` be the expression sequence `ε`.
12. Let `globaltype*` be the global type sequence `ε`.
13. For each `global` in `global*`, do:
    1. Let `(GLOBAL globaltype expr_g)` be the destructuring of `global`.
    2. Append `expr_g` to `expr_g*`.
    3. Append `globaltype` to `globaltype*`.
14. Let `expr_e**` be the expression sequence sequence `ε`.
15. For each `elem` in `elem*`, do:
    1. Let `(ELEM reftype expr_e* elemmode)` be the destructuring of `elem`.
    2. Append `expr_e*` to `expr_e**`.
16. Let `z` be the state `(s, { AMODULE moduleinst_0 })`.
17. Let `F` be the frame `z.ZFRAME`.
18. Push the frame `F`.
19. Let `val_g*` be `evalglobal*(z, globaltype*, expr_g*)`.
20. Let `reff_t*` be `evalexpr*(z, expr_t*)`.
21. Let `reff_e**` be `evalexpr**(z, expr_e**)`.
22. Pop the frame from the stack.
23. Let `(s, f)` be the destructuring of `z`.
24. Let `moduleinst` be `allocmodule(s, module, externaddr*, val_g*, reff_t*, reff_e**)`.
25. Let `F'` be the frame `{ AMODULE moduleinst }`.
26. Push the frame `F'`.
27. Execute the sequence `instr_e*`.
28. Execute the sequence `instr_d*`.
29. If `start?` is defined, then:
    1. Let `(START x)` be `start?`.
    2. Let `instr_s` be the instruction `(call x)`.
    3. Execute the instruction `instr_s`.
30. Pop the frame from the stack.
31. Return `moduleinst`.

```text
instantiate(s, module, externaddr*) = s'''' ; { AMODULE moduleinst } ; instr_e* instr_d* instr_s?
  (if |- module : xt_i* ->M xt_e*
   ∧ (s |- externaddr : xt_i)*
   ∧ module = MODULE type* import* tag* global* mem* table* func* data* elem* start? export*
   ∧ global* = (GLOBAL globaltype expr_g)*
   ∧ table* = (TABLE tabletype expr_t)*
   ∧ data* = (DATA byte* datamode)*
   ∧ elem* = (ELEM reftype expr_e* elemmode)*
   ∧ start? = (START x)?
   ∧ moduleinst_0 = { ITYPES alloc_type*(type*), IGLOBALS globalsxa(externaddr*), IFUNCS funcsxa(externaddr*) (|s.FUNCS| + i_f)^(i_f<|func*|) }
   ∧ z = s ; { AMODULE moduleinst_0 }
   ∧ (z', val_g*) = evalglobal*(z, globaltype*, expr_g*)
   ∧ (z'', reff_t*) = evalexpr*(z', expr_t*)
   ∧ (z''', reff_e**) = evalexpr**(z'', expr_e**)
   ∧ z''' = s''' ; f
   ∧ (s'''', moduleinst) = allocmodule(s''', module, externaddr*, val_g*, reff_t*, (reff_e*)*)
   ∧ instr_d* = ++ rundata_i_d(data*[i_d])^(i_d<|data*|)
   ∧ instr_e* = ++ runelem_i_e(elem*[i_e])^(i_e<|elem*|)
   ∧ instr_s? = (call x)?)
```

where:

#### evalexpr*(z, expr''*)

1. If `expr''* = ε`, then:
   1. Return `ε`.
2. Let `expr expr'*` be `expr''*`.
3. Let `reff` be the result of evaluating `expr` with state `z`.
4. Let `reff'*` be `evalexpr*(z, expr'*)`.
5. Return `reff reff'*`.

```text
evalexpr*(z, ε) = (z, ε)
evalexpr*(z, expr expr'*) = (z'', reff reff'*)
  (if z ; expr ->* z' ; reff
   ∧ (z'', reff'*) = evalexpr*(z', expr'*))
```

#### evalglobal*(z, globaltype*, expr''*)

1. If `expr''* = ε`, then:
   1. Assert: Due to validation, `globaltype* = ε`.
   2. Return `ε`.
2. Else:
   1. Let `expr expr'*` be `expr''*`.
   2. Assert: Due to validation, `|globaltype*| >= 1`.
   3. Let `gt gt'*` be `globaltype*`.
   4. Let `val` be the result of evaluating `expr` with state `z`.
   5. Let `(s, f)` be the destructuring of `z`.
   6. Let `a` be `allocglobal(s, gt, val)`.
   7. Append `a` to `f.AMODULE.IGLOBALS`.
   8. Let `val'*` be `evalglobal*((s, f), gt'*, expr'*)`.
   9. Return `val val'*`.

```text
evalglobal*(z, ε, ε) = (z, ε)
evalglobal*(z, gt gt'*, expr expr'*) = (z'', val val'*)
  (if z ; expr ->* z' ; val
   ∧ z' = s ; f
   ∧ (s', a) = allocglobal(s, gt, val)
   ∧ (z'', val'*) = evalglobal*((s' ; f[.AMODULE.IGLOBALS =⊕ a]), gt'*, expr'*))
```

#### rundata_x(DATA b^n datamode)

1. If `datamode = dpassive`, then:
   1. Return `ε`.
2. Assert: Due to validation, `datamode` is some `dactive memidx expr`.
3. Let `(dactive y instr*)` be the destructuring of `datamode`.
4. Return `instr* (i32.const 0) (i32.const n) (memory.init y x) (data.drop x)`.

#### runelem_x(ELEM rt e^n elemmode)

1. If `elemmode = epassive`, then:
   1. Return `ε`.
2. If `elemmode = edeclare`, then:
   1. Return `(elem.drop x)`.
3. Assert: Due to validation, `elemmode` is some `eactive tableidx expr`.
4. Let `(eactive y instr*)` be the destructuring of `elemmode`.
5. Return `instr* (i32.const 0) (i32.const n) (table.init y x) (elem.drop x)`.

```text
rundata_x(DATA b^n (dpassive)) = ε
rundata_x(DATA b^n (dactive y instr*)) = instr* (i32.const 0) (i32.const n) (memory.init y x) (data.drop x)

runelem_x(ELEM rt e^n (epassive)) = ε
runelem_x(ELEM rt e^n (edeclare)) = (elem.drop x)
runelem_x(ELEM rt e^n (eactive y instr*)) = instr* (i32.const 0) (i32.const n) (table.init y x) (elem.drop x)
```

> **Note:** Checking import types assumes that the module instance has already been allocated to compute the respective closed defined types.
>
> However, this forward reference merely is a way to simplify the specification.
>
> In practice, implementations will likely allocate or canonicalize types beforehand, when *compiling* a module, in a stage before instantiation and before imports are checked.
>
> Similarly, module allocation and the evaluation of global and table initializers as well as element segments are mutually recursive because the global initialization values `val_g*`, `reff_t`, and element segment contents `reff_e**` are passed to the module allocator while depending on the module instance `moduleinst` and store `s'` returned by allocation.
>
> Again, this recursion is just a specification device.
>
> In practice, the initialization values can be determined beforehand by staging module allocation such that first, the module's own function instances are pre-allocated in the store, then the initializer expressions are evaluated in order, allocating globals on the way, then the rest of the module instance is allocated, and finally the new function instances' `module` fields are set to that module instance.
>
> This is possible because validation ensures that initialization expressions cannot actually call a function, only take their reference.
>
> All failure conditions are checked before any observable mutation of the store takes place.
> Store mutation is not atomic; it happens in individual steps that may be interleaved with other threads.
>
> Evaluation of constant expressions does not affect the store.

### Invocation

Once a module has been instantiated, any exported function can be invoked externally via its function address `funcaddr` in the store `s` and an appropriate list `val*` of argument values.

Invocation may fail with an error if the arguments do not fit the function type. Invocation can also result in an exception or trap. It is up to the embedder to define how such conditions are reported.

> **Note:** If the embedder API performs type checks itself, either statically or dynamically, before performing an invocation, then no failure other than traps or exceptions can occur.

#### invoke(s, funcaddr, val*)

1. Assert: Due to validation, the expansion of `s.FUNCS[funcaddr].FITYPE` is some `func t1* -> t2*`.
2. Let `(func t1* -> t2*)` be the destructuring of the expansion of `s.FUNCS[funcaddr].FITYPE`.
3. If `|t1*| != |val*|`, then:
   1. Fail.
4. For all `t1` in `t1*`, and corresponding `val` in `val*`:
   1. If `val` is not valid with type `t1`, then:
      1. Fail.
5. Let `k` be the length of `t2*`.
6. Let `F` be the frame `{ AMODULE { } }` whose arity is `k`.
7. Push the frame `F`.
8. Push the values `val*` to the stack.
9. Push the value `(ref.func funcaddr)` to the stack.
10. Execute the instruction `(call_ref s.FUNCS[funcaddr].FITYPE)`.
11. Pop the values `val'^k` from the stack.
12. Pop the frame from the stack.
13. Return `val'^k`.

```text
invoke(s, funcaddr, val*) = s ; { AMODULE { } } ; val* (ref.func funcaddr) (call_ref s.FUNCS[funcaddr].FITYPE)
  (if s.FUNCS[funcaddr].FITYPE ≈ func t1* -> t2*
   ∧ (s |- val : t1)*)
```
