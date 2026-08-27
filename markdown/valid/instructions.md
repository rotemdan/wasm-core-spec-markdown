## Instructions

Instructions are classified by instruction types that describe how they manipulate the operand stack and initialize locals: A type `t1* ->_{x*} t2*` describes the required input stack with argument values of types `t1*` that an instruction pops off and the provided output stack with result values of types `t2*` that it pushes back. Moreover, it enumerates the indices `x*` of locals that have been set by the instruction. In most cases, this is empty.

> **Note:** For example, the instruction `i32.add` has type `i32 i32 -> i32`, consuming two `i32` values and producing one. The instruction `(local.set x)` has type `t ->_{x} ε`, provided `t` is the type declared for the local `x`.

Typing extends to instruction sequences `instr*`. Such a sequence has an instruction type `t1* ->_{x*} t2*` if the accumulative effect of executing the instructions is consuming values of types `t1*` off the operand stack, pushing new values of types `t2*`, and setting all locals `x*`.

#### Polymorphism

For some instructions, the typing rules do not fully constrain the type, and therefore allow for multiple types. Such instructions are called *polymorphic*.

Two degrees of polymorphism can be distinguished:

* *value-polymorphic*: the value type `t` of one or several individual operands is unconstrained. That is the case for all parametric instructions like `drop` and `select`.
* *stack-polymorphic*: the entire (or most of the) instruction type `t1* -> t2*` of the instruction is unconstrained. That is the case for all control instructions that perform an unconditional control transfer, such as `br`, or `return`.

In both cases, the unconstrained types or type sequences can be chosen arbitrarily, as long as they are valid in the current context and meet the constraints imposed for the surrounding parts of the program.

> **Note:** For example, the `select` instruction is valid with type `t t i32 -> t`, for any possible number type `t`. Consequently, both instruction sequences `(i32.const 1) (i32.const 2) (i32.const 3) (select)` and `(f64.const +1) (f64.const +2) (i32.const 3) (select)` are valid, with `t` in the typing of `select` being instantiated to `i32` or `f64`, respectively.
>
> The `unreachable` instruction is stack-polymorphic, and hence valid with type `t1* -> t2*` for any possible sequences of value types `t1*` and `t2*`. Consequently, `(unreachable) (i32.add)` is valid by assuming type `ε -> i32` for the `unreachable` instruction. In contrast, `(unreachable) (i64.const 0) (i32.add)` is invalid, because there is no possible type to pick for the `unreachable` instruction that would make the sequence well-typed.

The Appendix describes a type checking algorithm that efficiently implements validation of instruction sequences as prescribed by the rules given here.

### Parametric Instructions

#### NOP

The instruction `nop` is valid with the instruction type `ε -> ε`.

```text
────────────────
C |- nop : ε -> ε
```

#### UNREACHABLE

The instruction `unreachable` is valid with the instruction type `t1* -> t2*` if:

* The instruction type `t1* -> t2*` is valid.

```text
C |- t1* -> t2* : OK
──────────────────────────────────
C |- unreachable : t1* -> t2*
```

> **Note:** The `unreachable` instruction is stack-polymorphic.

#### DROP

The instruction `drop` is valid with the instruction type `t -> ε` if:

* The value type `t` is valid.

```text
C |- t : OK
──────────────────────
C |- drop : t -> ε
```

> **Note:** Both `drop` and `select` without annotation are value-polymorphic instructions.

#### SELECT (t\*)^?

The instruction `(select valtype?)` is valid with the instruction type `t t i32 -> t` if:

* The value type `t` is valid.
* Either:
   * The value type sequence `valtype?` is of the form `t`.
* Or:
   * The value type sequence `valtype?` is absent.
   * The value type `t` matches the value type `t'`.
   * The value type `t'` is of the form `numtype` or `t'` is of the form `vectype`.

```text
C |- t : OK
──────────────────────────
C |- select t : t t i32 -> t

C |- t : OK
C |- t <: t'
t' = numtype ∨ t' = vectype
──────────────────────────────────
C |- select : t t i32 -> t
```

> **Note:** In future versions of WebAssembly, `select` may allow more than one value per choice.

### Control Instructions

#### BLOCK bt instr\*

The instruction `(block bt instr*)` is valid with the instruction type `t1* -> t2*` if:

* The block type `bt` is valid as the instruction type `t1* -> t2*`.
* Let `C'` be the same context as `C`, but with the result type sequence `t2*` prepended to the field `LABELS`.
* Under the context `C'`, the instruction sequence `instr*` is valid with the instruction type `t1* ->_{x*} t2*`.

```text
C |- bt : t1* -> t2*
{ LABELS (t2*) } ⊕ C |- instr* : t1* ->_{x*} t2*
──────────────────────────────────────────────────────
C |- block bt instr* : t1* -> t2*
```

> **Note:** The notation `{ LABELS (t*) } ⊕ C` inserts the new label type at index `0`, shifting all others.
> The same applies to all other block instructions.

#### LOOP bt instr\*

The instruction `(loop bt instr*)` is valid with the instruction type `t1* -> t2*` if:

* The block type `bt` is valid as the instruction type `t1* -> t2*`.
* Let `C'` be the same context as `C`, but with the result type sequence `t1*` prepended to the field `LABELS`.
* Under the context `C'`, the instruction sequence `instr*` is valid with the instruction type `t1* ->_{x*} t2*`.

```text
C |- bt : t1* -> t2*
{ LABELS (t1*) } ⊕ C |- instr* : t1* ->_{x*} t2*
──────────────────────────────────────────────────────
C |- loop bt instr* : t1* -> t2*
```

#### IF bt instr1\* ELSE instr2\*

The instruction `(if bt instr1* else instr2*)` is valid with the instruction type `t1* i32 -> t2*` if:

* The block type `bt` is valid as the instruction type `t1* -> t2*`.
* Let `C'` be the same context as `C`, but with the result type sequence `t2*` prepended to the field `LABELS`.
* Under the context `C'`, the instruction sequence `instr1*` is valid with the instruction type `t1* ->_{x1*} t2*`.
* Under the context `C'`, the instruction sequence `instr2*` is valid with the instruction type `t1* ->_{x2*} t2*`.

```text
C |- bt : t1* -> t2*
{ LABELS (t2*) } ⊕ C |- instr1* : t1* ->_{x1*} t2*
{ LABELS (t2*) } ⊕ C |- instr2* : t1* ->_{x2*} t2*
──────────────────────────────────────────────────────────────────────
C |- if bt instr1* else instr2* : t1* i32 -> t2*
```

#### BR l

The instruction `(br l)` is valid with the instruction type `t1* t* -> t2*` if:

* The label `C.LABELS[l]` exists.
* The label `C.LABELS[l]` is of the form `t*`.
* The instruction type `t1* -> t2*` is valid.

```text
C.LABELS[l] = t*
C |- t1* -> t2* : OK
──────────────────────────────────
C |- br l : t1* t* -> t2*
```

> **Note:** The label index space in the context `C` contains the most recent label first, so that `C.labels[l]` performs a relative lookup as expected. This applies to other branch instructions as well.
>
> The `br` instruction is stack-polymorphic.

#### BRIF l

The instruction `(br_if l)` is valid with the instruction type `t* i32 -> t*` if:

* The label `C.LABELS[l]` exists.
* The label `C.LABELS[l]` is of the form `t*`.

```text
C.LABELS[l] = t*
──────────────────────
C |- br_if l : t* i32 -> t*
```

#### BRTABLE l\* lN

The instruction `(br_table l* l')` is valid with the instruction type `t1* t* i32 -> t2*` if:

* For all `l` in `l*`:
   * The label `C.LABELS[l]` exists.
   * The result type `t*` matches the label `C.LABELS[l]`.
* The label `C.LABELS[l']` exists.
* The result type `t*` matches the label `C.LABELS[l']`.
* The instruction type `t1* t* i32 -> t2*` is valid.

```text
(C |- t* <: C.LABELS[l])*
C |- t* <: C.LABELS[l']
C |- t1* t* i32 -> t2* : OK
──────────────────────────────────────────────────────────
C |- br_table l* l' : t1* t* i32 -> t2*
```

> **Note:** The `br_table` instruction is stack-polymorphic.
>
> Furthermore, the result type `t*` is also chosen non-deterministically in this rule. Although it may seem necessary to compute `t*` as the greatest lower bound of all label types in practice, a simple sequential algorithm does not require this.

#### BRONNULL l

The instruction `(br_on_null l)` is valid with the instruction type `t* (ref null ht) -> t* (ref ht)` if:

* The label `C.LABELS[l]` exists.
* The label `C.LABELS[l]` is of the form `t*`.
* The heap type `ht` is valid.

```text
C.LABELS[l] = t*
C |- ht : OK
──────────────────────────────
C |- br_on_null l : t* (ref null ht) -> t* (ref ht)
```

#### BRONNONNULL l

The instruction `(br_on_non_null l)` is valid with the instruction type `t* (ref null ht) -> t*` if:

* The label `C.LABELS[l]` exists.
* The label `C.LABELS[l]` is of the form `t* (ref null^? ht)`.

```text
C.LABELS[l] = t* (ref null^? ht)
────────────────────────────────────
C |- br_on_non_null l : t* (ref null ht) -> t*
```

#### BRONCAST l rt1 rt2

The instruction `(br_on_cast l rt1 rt2)` is valid with the instruction type `t* rt1 -> t* reftype` if:

* The label `C.LABELS[l]` exists.
* The label `C.LABELS[l]` is of the form `t* rt`.
* The reference type `rt1` is valid.
* The reference type `rt2` is valid.
* The reference type `rt2` matches the reference type `rt1`.
* The reference type `rt2` matches the reference type `rt`.
* The reference type `reftype` is `rt1 reftypediff rt2`.

```text
C.LABELS[l] = t* rt
C |- rt1 : OK
C |- rt2 : OK
C |- rt2 <: rt1
C |- rt2 <: rt
────────────────────────────────────────────────────────
C |- br_on_cast l rt1 rt2 : t* rt1 -> t* (rt1 reftypediff rt2)
```

#### BRONCASTFAIL l rt1 rt2

The instruction `(br_on_cast_fail l rt1 rt2)` is valid with the instruction type `t* rt1 -> t* rt2` if:

* The label `C.LABELS[l]` exists.
* The label `C.LABELS[l]` is of the form `t* rt`.
* The reference type `rt1` is valid.
* The reference type `rt2` is valid.
* The reference type `rt2` matches the reference type `rt1`.
* The reference type `rt1 reftypediff rt2` matches the reference type `rt`.

```text
C.LABELS[l] = t* rt
C |- rt1 : OK
C |- rt2 : OK
C |- rt2 <: rt1
C |- rt1 reftypediff rt2 <: rt
────────────────────────────────────────────────────────────
C |- br_on_cast_fail l rt1 rt2 : t* rt1 -> t* rt2
```

#### CALL x

The instruction `(call x)` is valid with the instruction type `t1* -> t2*` if:

* The function `C.FUNCS[x]` exists.
* The expansion of `C.FUNCS[x]` is `(func t1* -> t2*)`.

```text
C.FUNCS[x] ≈ func t1* -> t2*
──────────────────────────────
C |- call x : t1* -> t2*
```

#### CALLREF x

The instruction `(call_ref x)` is valid with the instruction type `t1* (ref null x) -> t2*` if:

* The type `C.TYPES[x]` exists.
* The expansion of `C.TYPES[x]` is `(func t1* -> t2*)`.

```text
C.TYPES[x] ≈ func t1* -> t2*
──────────────────────────────────
C |- call_ref x : t1* (ref null x) -> t2*
```

#### CALLINDIRECT x y

The instruction `(call_indirect x y)` is valid with the instruction type `t1* at -> t2*` if:

* The table `C.TABLES[x]` exists.
* The table `C.TABLES[x]` is of the form `(at lim rt)`.
* The reference type `rt` matches the reference type `(ref null func)`.
* The type `C.TYPES[y]` exists.
* The expansion of `C.TYPES[y]` is `(func t1* -> t2*)`.

```text
C.TABLES[x] = at lim rt
C |- rt <: (ref null func)
C.TYPES[y] ≈ func t1* -> t2*
──────────────────────────────────────────────
C |- call_indirect x y : t1* at -> t2*
```

#### RETURN

The instruction `return` is valid with the instruction type `t1* t* -> t2*` if:

* The result type `C.RETURN` is of the form `t*`.
* The instruction type `t1* -> t2*` is valid.

```text
C.RETURN = (t*)
C |- t1* -> t2* : OK
─────────────────────────────────
C |- return : t1* t* -> t2*
```

> **Note:** The `return` instruction is stack-polymorphic.
>
> `C.RETURN` is absent (set to `ε`) when validating an expression that is not a function body. This differs from it being set to the empty result type `[ε]`, which is the case for functions not returning anything.

#### RETURNCALL x

The instruction `(return_call x)` is valid with the instruction type `t3* t1* -> t4*` if:

* The function `C.FUNCS[x]` exists.
* The expansion of `C.FUNCS[x]` is `(func t1* -> t2*)`.
* The result type `C.RETURN` is of the form `t'2*`.
* The result type `t2*` matches the result type `t'2*`.
* The instruction type `t3* -> t4*` is valid.

```text
C.FUNCS[x] ≈ func t1* -> t2*
C.RETURN = (t'2*)
C |- t2* <: t'2*
C |- t3* -> t4* : OK
──────────────────────────────────────────────
C |- return_call x : t3* t1* -> t4*
```

> **Note:** The `return_call` instruction is stack-polymorphic.

#### RETURNCALLREF x

The instruction `(return_call_ref x)` is valid with the instruction type `t3* t1* (ref null x) -> t4*` if:

* The type `C.TYPES[x]` exists.
* The expansion of `C.TYPES[x]` is `(func t1* -> t2*)`.
* The result type `C.RETURN` is of the form `t'2*`.
* The result type `t2*` matches the result type `t'2*`.
* The instruction type `t3* -> t4*` is valid.

```text
C.TYPES[x] ≈ func t1* -> t2*
C.RETURN = (t'2*)
C |- t2* <: t'2*
C |- t3* -> t4* : OK
────────────────────────────────────────────────
C |- return_call_ref x : t3* t1* (ref null x) -> t4*
```

> **Note:** The `return_call_ref` instruction is stack-polymorphic.

#### RETURNCALLINDIRECT x y

The instruction `(return_call_indirect x y)` is valid with the instruction type `t3* t1* at -> t4*` if:

* The table `C.TABLES[x]` exists.
* The table `C.TABLES[x]` is of the form `(at lim rt)`.
* The reference type `rt` matches the reference type `(ref null func)`.
* The type `C.TYPES[y]` exists.
* The expansion of `C.TYPES[y]` is `(func t1* -> t2*)`.
* The result type `C.RETURN` is of the form `t'2*`.
* The result type `t2*` matches the result type `t'2*`.
* The instruction type `t3* -> t4*` is valid.

```text
C.TABLES[x] = at lim rt
C |- rt <: (ref null func)
C.TYPES[y] ≈ func t1* -> t2*
C.RETURN = (t'2*)
C |- t2* <: t'2*
C |- t3* -> t4* : OK
──────────────────────────────────────────────────────────────
C |- return_call_indirect x y : t3* t1* at -> t4*
```

> **Note:** The `return_call_indirect` instruction is stack-polymorphic.

#### THROW x

The instruction `(throw x)` is valid with the instruction type `t1* t* -> t2*` if:

* The tag `C.TAGS[x]` exists.
* The expansion of `C.TAGS[x]` is `(func t* ->)`.
* The instruction type `t1* -> t2*` is valid.

```text
C.TAGS[x] ≈ func t* -> ε
C |- t1* -> t2* : OK
─────────────────────────────────
C |- throw x : t1* t* -> t2*
```

> **Note:** The `throw` instruction is stack-polymorphic.

#### THROWREF

The instruction `throw_ref` is valid with the instruction type `t1* (ref null exn) -> t2*` if:

* The instruction type `t1* -> t2*` is valid.

```text
C |- t1* -> t2* : OK
────────────────────────────────────
C |- throw_ref : t1* (ref null exn) -> t2*
```

> **Note:** The `throw_ref` instruction is stack-polymorphic.

#### TRYTABLE bt catch\* instr\*

The instruction `(try_table bt catch* instr*)` is valid with the instruction type `t1* -> t2*` if:

* The block type `bt` is valid as the instruction type `t1* -> t2*`.
* Let `C'` be the same context as `C`, but with the result type sequence `t2*` prepended to the field `LABELS`.
* Under the context `C'`, the instruction sequence `instr*` is valid with the instruction type `t1* ->_{x*} t2*`.
* For all `catch` in `catch*`:
   * The catch clause `catch` is valid.

```text
C |- bt : t1* -> t2*
{ LABELS (t2*) } ⊕ C |- instr* : t1* ->_{x*} t2*
(C |- catch : OK)*
──────────────────────────────────────────────────────────
C |- try_table bt catch* instr* : t1* -> t2*
```

#### CATCH x l

The catch clause `(catch x l)` is valid if:

* The tag `C.TAGS[x]` exists.
* The expansion of `C.TAGS[x]` is `(func t* ->)`.
* The label `C.LABELS[l]` exists.
* The result type `t*` matches the label `C.LABELS[l]`.

```text
C.TAGS[x] ≈ func t* -> ε
C |- t* <: C.LABELS[l]
──────────────────────────────────────────────
C |- catch x l : OK
```

#### CATCHREF x l

The catch clause `(catch_ref x l)` is valid if:

* The tag `C.TAGS[x]` exists.
* The expansion of `C.TAGS[x]` is `(func t* ->)`.
* The label `C.LABELS[l]` exists.
* The result type `t* (ref exn)` matches the label `C.LABELS[l]`.

```text
C.TAGS[x] ≈ func t* -> ε
C |- t* (ref exn) <: C.LABELS[l]
────────────────────────────────────────────────────────
C |- catch_ref x l : OK
```

#### CATCHALL l

The catch clause `(catch_all l)` is valid if:

* The label `C.LABELS[l]` exists.
* The result type `ε` matches the label `C.LABELS[l]`.

```text
C |- ε <: C.LABELS[l]
──────────────────────────────────────────────
C |- catch_all l : OK
```

#### CATCHALLREF l

The catch clause `(catch_all_ref l)` is valid if:

* The label `C.LABELS[l]` exists.
* The result type `(ref exn)` matches the label `C.LABELS[l]`.

```text
C |- (ref exn) <: C.LABELS[l]
──────────────────────────────────────────────────────
C |- catch_all_ref l : OK
```

### Variable Instructions

#### LOCALGET x

The instruction `(local.get x)` is valid with the instruction type `ε -> t` if:

* The local `C.LOCALS[x]` exists.
* The local `C.LOCALS[x]` is of the form `(set t)`.

```text
C.LOCALS[x] = set t
──────────────────────
C |- local.get x : ε -> t
```

#### LOCALSET x

The instruction `(local.set x)` is valid with the instruction type `t ->_{x} ε` if:

* The local `C.LOCALS[x]` exists.
* The local `C.LOCALS[x]` is of the form `(init t)`.

```text
C.LOCALS[x] = init t
──────────────────────
C |- local.set x : t ->_{x} ε
```

#### LOCALTEE x

The instruction `(local.tee x)` is valid with the instruction type `t ->_{x} t` if:

* The local `C.LOCALS[x]` exists.
* The local `C.LOCALS[x]` is of the form `(init t)`.

```text
C.LOCALS[x] = init t
──────────────────────
C |- local.tee x : t ->_{x} t
```

#### GLOBALGET x

The instruction `(global.get x)` is valid with the instruction type `ε -> t` if:

* The global `C.GLOBALS[x]` exists.
* The global `C.GLOBALS[x]` is of the form `(mut? t)`.

```text
C.GLOBALS[x] = mut? t
──────────────────────
C |- global.get x : ε -> t
```

#### GLOBALSET x

The instruction `(global.set x)` is valid with the instruction type `t -> ε` if:

* The global `C.GLOBALS[x]` exists.
* The global `C.GLOBALS[x]` is of the form `(mut t)`.

```text
C.GLOBALS[x] = mut t
──────────────────────
C |- global.set x : t -> ε
```

### Table Instructions

#### TABLEGET x

The instruction `(table.get x)` is valid with the instruction type `at -> rt` if:

* The table `C.TABLES[x]` exists.
* The table `C.TABLES[x]` is of the form `(at lim rt)`.

```text
C.TABLES[x] = at lim rt
────────────────────────
C |- table.get x : at -> rt
```

#### TABLESET x

The instruction `(table.set x)` is valid with the instruction type `at rt -> ε` if:

* The table `C.TABLES[x]` exists.
* The table `C.TABLES[x]` is of the form `(at lim rt)`.

```text
C.TABLES[x] = at lim rt
────────────────────────────
C |- table.set x : at rt -> ε
```

#### TABLESIZE x

The instruction `(table.size x)` is valid with the instruction type `ε -> at` if:

* The table `C.TABLES[x]` exists.
* The table `C.TABLES[x]` is of the form `(at lim rt)`.

```text
C.TABLES[x] = at lim rt
──────────────────────────
C |- table.size x : ε -> at
```

#### TABLEGROW x

The instruction `(table.grow x)` is valid with the instruction type `rt at -> at` if:

* The table `C.TABLES[x]` exists.
* The table `C.TABLES[x]` is of the form `(at lim rt)`.

```text
C.TABLES[x] = at lim rt
────────────────────────────
C |- table.grow x : rt at -> at
```

#### TABLEFILL x

The instruction `(table.fill x)` is valid with the instruction type `at rt at -> ε` if:

* The table `C.TABLES[x]` exists.
* The table `C.TABLES[x]` is of the form `(at lim rt)`.

```text
C.TABLES[x] = at lim rt
────────────────────────────────
C |- table.fill x : at rt at -> ε
```

#### TABLECOPY x y

The instruction `(table.copy x1 x2)` is valid with the instruction type `at1 at2 addrtype -> ε` if:

* The table `C.TABLES[x1]` exists.
* The table `C.TABLES[x1]` is of the form `(at1 lim1 rt1)`.
* The table `C.TABLES[x2]` exists.
* The table `C.TABLES[x2]` is of the form `(at2 lim2 rt2)`.
* The reference type `rt2` matches the reference type `rt1`.
* The address type `addrtype` is `addrtypemin(at1, at2)`.

```text
C.TABLES[x1] = at1 lim1 rt1
C.TABLES[x2] = at2 lim2 rt2
C |- rt2 <: rt1
──────────────────────────────────────────
C |- table.copy x1 x2 : at1 at2 addrtypemin(at1, at2) -> ε
```

#### TABLEINIT x y

The instruction `(table.init x y)` is valid with the instruction type `at i32 i32 -> ε` if:

* The table `C.TABLES[x]` exists.
* The table `C.TABLES[x]` is of the form `(at lim rt1)`.
* The element segment `C.ELEMS[y]` exists.
* The element segment `C.ELEMS[y]` is of the form `rt2`.
* The reference type `rt2` matches the reference type `rt1`.

```text
C.TABLES[x] = at lim rt1
C.ELEMS[y] = rt2
C |- rt2 <: rt1
──────────────────────────────────────
C |- table.init x y : at i32 i32 -> ε
```

#### ELEMDROP x

The instruction `(elem.drop x)` is valid with the instruction type `ε -> ε` if:

* The element segment `C.ELEMS[x]` exists.

```text
C.ELEMS[x] = rt
──────────────────
C |- elem.drop x : ε -> ε
```

### Memory Instructions

Memory instructions use memory arguments, which are classified by the address type and the bit width of the access they are suitable for.

#### memarg

`{ align n, offset m }` is valid for `at` and `N` if:

* `2^n` is less than or equal to `N / 8`.
* `m` is less than `2^|at|`.

```text
2^n ≤ N / 8
m < 2^|at|
──────────────────────────
|- { align n, offset m } : at -> N
```

#### t.LOAD x memarg

The instruction `(nt.LOAD x memarg)` is valid with the instruction type `at -> nt` if:

* The memory `C.MEMS[x]` exists.
* The memory `C.MEMS[x]` is of the form `(at lim page)`.
* `memarg` is valid for `at` and `|nt|`.

```text
C.MEMS[x] = at lim page
|- memarg : at -> |nt|
──────────────────────────────────────
C |- nt.LOAD x memarg : at -> nt
```

#### t.LOAD{N}\_sx x memarg

The instruction `(ntN.LOAD K_sx x memarg)` is valid with the instruction type `at -> ntN` if:

* The memory `C.MEMS[x]` exists.
* The memory `C.MEMS[x]` is of the form `(at lim page)`.
* `memarg` is valid for `at` and `K`.

```text
C.MEMS[x] = at lim page
|- memarg : at -> K
──────────────────────────────────────────
C |- ntN.LOAD K_sx x memarg : at -> ntN
```

#### t.STORE x memarg

The instruction `(nt.STORE x memarg)` is valid with the instruction type `at nt -> ε` if:

* The memory `C.MEMS[x]` exists.
* The memory `C.MEMS[x]` is of the form `(at lim page)`.
* `memarg` is valid for `at` and `|nt|`.

```text
C.MEMS[x] = at lim page
|- memarg : at -> |nt|
──────────────────────────────────────
C |- nt.STORE x memarg : at nt -> ε
```

#### t.STORE{N} x memarg

The instruction `(ntN.STORE K x memarg)` is valid with the instruction type `at ntN -> ε` if:

* The memory `C.MEMS[x]` exists.
* The memory `C.MEMS[x]` is of the form `(at lim page)`.
* `memarg` is valid for `at` and `K`.

```text
C.MEMS[x] = at lim page
|- memarg : at -> K
──────────────────────────────────────────
C |- ntN.STORE K x memarg : at ntN -> ε
```

#### v128.LOAD x memarg

The instruction `(v128.LOAD x memarg)` is valid with the instruction type `at -> v128` if:

* The memory `C.MEMS[x]` exists.
* The memory `C.MEMS[x]` is of the form `(at lim page)`.
* `memarg` is valid for `at` and `|v128|`.

```text
C.MEMS[x] = at lim page
|- memarg : at -> |v128|
────────────────────────────────────────
C |- v128.LOAD x memarg : at -> v128
```

#### v128.LOAD{N}xM\_sx x memarg

The instruction `(v128.LOAD NshapeM_sx x memarg)` is valid with the instruction type `at -> v128` if:

* The memory `C.MEMS[x]` exists.
* The memory `C.MEMS[x]` is of the form `(at lim page)`.
* `memarg` is valid for `at` and `N · M`.

```text
C.MEMS[x] = at lim page
|- memarg : at -> N · M
──────────────────────────────────────────────
C |- v128.LOAD NshapeM_sx x memarg : at -> v128
```

#### v128.LOAD{N}\_splat x memarg

The instruction `(v128.LOAD N_splat x memarg)` is valid with the instruction type `at -> v128` if:

* The memory `C.MEMS[x]` exists.
* The memory `C.MEMS[x]` is of the form `(at lim page)`.
* `memarg` is valid for `at` and `N`.

```text
C.MEMS[x] = at lim page
|- memarg : at -> N
──────────────────────────────────────────────
C |- v128.LOAD N_splat x memarg : at -> v128
```

#### v128.LOAD{N}\_zero x memarg

The instruction `(v128.LOAD N_zero x memarg)` is valid with the instruction type `at -> v128` if:

* The memory `C.MEMS[x]` exists.
* The memory `C.MEMS[x]` is of the form `(at lim page)`.
* `memarg` is valid for `at` and `N`.

```text
C.MEMS[x] = at lim page
|- memarg : at -> N
──────────────────────────────────────────────
C |- v128.LOAD N_zero x memarg : at -> v128
```

#### v128.LOAD{N}\_lane x memarg laneidx

The instruction `(v128.LOAD N_lane x memarg i)` is valid with the instruction type `at v128 -> v128` if:

* The memory `C.MEMS[x]` exists.
* The memory `C.MEMS[x]` is of the form `(at lim page)`.
* `memarg` is valid for `at` and `N`.
* `i` is less than `128 / N`.

```text
C.MEMS[x] = at lim page
|- memarg : at -> N
i < 128 / N
──────────────────────────────────────────────────
C |- v128.LOAD N_lane x memarg i : at v128 -> v128
```

#### v128.STORE x memarg

The instruction `(v128.STORE x memarg)` is valid with the instruction type `at v128 -> ε` if:

* The memory `C.MEMS[x]` exists.
* The memory `C.MEMS[x]` is of the form `(at lim page)`.
* `memarg` is valid for `at` and `|v128|`.

```text
C.MEMS[x] = at lim page
|- memarg : at -> |v128|
────────────────────────────────────────
C |- v128.STORE x memarg : at v128 -> ε
```

#### v128.STORE{N}\_lane x memarg laneidx

The instruction `(v128.STORE N_lane x memarg i)` is valid with the instruction type `at v128 -> ε` if:

* The memory `C.MEMS[x]` exists.
* The memory `C.MEMS[x]` is of the form `(at lim page)`.
* `memarg` is valid for `at` and `N`.
* `i` is less than `128 / N`.

```text
C.MEMS[x] = at lim page
|- memarg : at -> N
i < 128 / N
──────────────────────────────────────────────────
C |- v128.STORE N_lane x memarg i : at v128 -> ε
```

#### MEMORYSIZE x

The instruction `(memory.size x)` is valid with the instruction type `ε -> at` if:

* The memory `C.MEMS[x]` exists.
* The memory `C.MEMS[x]` is of the form `(at lim page)`.

```text
C.MEMS[x] = at lim page
──────────────────────────
C |- memory.size x : ε -> at
```

#### MEMORYGROW x

The instruction `(memory.grow x)` is valid with the instruction type `at -> at` if:

* The memory `C.MEMS[x]` exists.
* The memory `C.MEMS[x]` is of the form `(at lim page)`.

```text
C.MEMS[x] = at lim page
──────────────────────────
C |- memory.grow x : at -> at
```

#### MEMORYFILL x

The instruction `(memory.fill x)` is valid with the instruction type `at i32 at -> ε` if:

* The memory `C.MEMS[x]` exists.
* The memory `C.MEMS[x]` is of the form `(at lim page)`.

```text
C.MEMS[x] = at lim page
──────────────────────────────
C |- memory.fill x : at i32 at -> ε
```

#### MEMORYCOPY x y

The instruction `(memory.copy x1 x2)` is valid with the instruction type `at1 at2 addrtype -> ε` if:

* The memory `C.MEMS[x1]` exists.
* The memory `C.MEMS[x1]` is of the form `(at1 lim1 page)`.
* The memory `C.MEMS[x2]` exists.
* The memory `C.MEMS[x2]` is of the form `(at2 lim2 page)`.
* The address type `addrtype` is `addrtypemin(at1, at2)`.

```text
C.MEMS[x1] = at1 lim1 page
C.MEMS[x2] = at2 lim2 page
──────────────────────────────────────────────────────
C |- memory.copy x1 x2 : at1 at2 addrtypemin(at1, at2) -> ε
```

#### MEMORYINIT x y

The instruction `(memory.init x y)` is valid with the instruction type `at i32 i32 -> ε` if:

* The memory `C.MEMS[x]` exists.
* The memory `C.MEMS[x]` is of the form `(at lim page)`.
* The data segment `C.DATAS[y]` exists.
* The data segment `C.DATAS[y]` is of the form `OK`.

```text
C.MEMS[x] = at lim page
C.DATAS[y] = OK
──────────────────────────────────
C |- memory.init x y : at i32 i32 -> ε
```

#### DATADROP x

The instruction `(data.drop x)` is valid with the instruction type `ε -> ε` if:

* The data segment `C.DATAS[x]` exists.
* The data segment `C.DATAS[x]` is of the form `OK`.

```text
C.DATAS[x] = OK
──────────────────
C |- data.drop x : ε -> ε
```

### Reference Instructions

#### REFNULL ht

The instruction `(ref.null ht)` is valid with the instruction type `ε -> (ref null ht)` if:

* The heap type `ht` is valid.

```text
C |- ht : OK
──────────────────────────
C |- ref.null ht : ε -> (ref null ht)
```

#### REFFUNC x

The instruction `(ref.func x)` is valid with the instruction type `ε -> (ref dt)` if:

* The function `C.FUNCS[x]` exists.
* The function `C.FUNCS[x]` is of the form `dt`.
* `x` is contained in `C.REFS`.

```text
C.FUNCS[x] = dt
x ∈ C.REFS
──────────────────────────────
C |- ref.func x : ε -> (ref dt)
```

#### REFISNULL

The instruction `ref.is_null` is valid with the instruction type `(ref null ht) -> i32` if:

* The heap type `ht` is valid.

```text
C |- ht : OK
────────────────────────────
C |- ref.is_null : (ref null ht) -> i32
```

#### REFASNONNULL

The instruction `ref.as_non_null` is valid with the instruction type `(ref null ht) -> (ref ht)` if:

* The heap type `ht` is valid.

```text
C |- ht : OK
──────────────────────────────
C |- ref.as_non_null : (ref null ht) -> (ref ht)
```

#### REFEQ

The instruction `ref.eq` is valid with the instruction type `(ref null eq) (ref null eq) -> i32`.

```text
──────────────────────────────────────────
C |- ref.eq : (ref null eq) (ref null eq) -> i32
```

#### REFTEST rt

The instruction `(ref.test rt)` is valid with the instruction type `rt' -> i32` if:

* The reference type `rt` is valid.
* The reference type `rt'` is valid.
* The reference type `rt` matches the reference type `rt'`.

```text
C |- rt : OK
C |- rt' : OK
C |- rt <: rt'
──────────────────────────────────────────
C |- ref.test rt : rt' -> i32
```

> **Note:** The liberty to pick a supertype `rt'` allows typing the instruction with the least precise super type of `rt` as input, that is, the top type in the corresponding heap subtyping hierarchy.

#### REFCAST rt

The instruction `(ref.cast rt)` is valid with the instruction type `rt' -> rt` if:

* The reference type `rt` is valid.
* The reference type `rt'` is valid.
* The reference type `rt` matches the reference type `rt'`.

```text
C |- rt : OK
C |- rt' : OK
C |- rt <: rt'
──────────────────────────────────────────
C |- ref.cast rt : rt' -> rt
```

> **Note:** The liberty to pick a supertype `rt'` allows typing the instruction with the least precise super type of `rt` as input, that is, the top type in the corresponding heap subtyping hierarchy.

### Aggregate Reference Instructions

#### STRUCTNEW x

The instruction `(struct.new x)` is valid with the instruction type `t* -> (ref x)` if:

* The type `C.TYPES[x]` exists.
* The expansion of `C.TYPES[x]` is `(struct (mut? zt)*)`.
* The value type sequence `t*` is `unpack(zt)*`.

```text
C.TYPES[x] ≈ struct (mut? zt)*
──────────────────────────────────
C |- struct.new x : unpack(zt)* -> (ref x)
```

#### STRUCTNEWDEFAULT x

The instruction `(struct.new_default x)` is valid with the instruction type `ε -> (ref x)` if:

* The type `C.TYPES[x]` exists.
* The expansion of `C.TYPES[x]` is `(struct (mut? zt)*)`.
* For all `zt` in `zt*`:
   * A default value for `unpack(zt)` is defined.

```text
C.TYPES[x] ≈ struct (mut? zt)*
(default_unpack(zt) ≠ ε)*
──────────────────────────────────────────
C |- struct.new_default x : ε -> (ref x)
```

#### STRUCTGET \_sx^? x y

The instruction `(struct.get_sx^? x i)` is valid with the instruction type `(ref null x) -> t` if:

* The type `C.TYPES[x]` exists.
* The expansion of `C.TYPES[x]` is `(struct ft*)`.
* The length of `ft*` is greater than `i`.
* The field type `ft*[i]` is of the form `(mut? zt)`.
* The signedness `sx^?` is present if and only if `zt` is a packed type.
* The value type `t` is `unpack(zt)`.

```text
C.TYPES[x] ≈ struct ft*
ft*[i] = mut? zt
sx^? ≠ ε ⇔ zt ≠ unpack(zt)
──────────────────────────────────────────────
C |- struct.get_sx^? x i : (ref null x) -> unpack(zt)
```

#### STRUCTSET x y

The instruction `(struct.set x i)` is valid with the instruction type `(ref null x) t -> ε` if:

* The type `C.TYPES[x]` exists.
* The expansion of `C.TYPES[x]` is `(struct ft*)`.
* The length of `ft*` is greater than `i`.
* The field type `ft*[i]` is of the form `(mut zt)`.
* The value type `t` is `unpack(zt)`.

```text
C.TYPES[x] ≈ struct ft*
ft*[i] = mut zt
────────────────────────────────────────
C |- struct.set x i : (ref null x) unpack(zt) -> ε
```

#### ARRAYNEW x

The instruction `(array.new x)` is valid with the instruction type `t i32 -> (ref x)` if:

* The type `C.TYPES[x]` exists.
* The expansion of `C.TYPES[x]` is `(array (mut? zt))`.
* The value type `t` is `unpack(zt)`.

```text
C.TYPES[x] ≈ array (mut? zt)
──────────────────────────────────
C |- array.new x : unpack(zt) i32 -> (ref x)
```

#### ARRAYNEWDEFAULT x

The instruction `(array.new_default x)` is valid with the instruction type `i32 -> (ref x)` if:

* The type `C.TYPES[x]` exists.
* The expansion of `C.TYPES[x]` is `(array (mut? zt))`.
* A default value for `unpack(zt)` is defined.

```text
C.TYPES[x] ≈ array (mut? zt)
default_unpack(zt) ≠ ε
──────────────────────────────────────────
C |- array.new_default x : i32 -> (ref x)
```

#### ARRAYNEWFIXED x n

The instruction `(array.new_fixed x n)` is valid with the instruction type `t^n -> (ref x)` if:

* The type `C.TYPES[x]` exists.
* The expansion of `C.TYPES[x]` is `(array (mut? zt))`.
* The value type `t` is `unpack(zt)`.

```text
C.TYPES[x] ≈ array (mut? zt)
────────────────────────────────────
C |- array.new_fixed x n : unpack(zt)^n -> (ref x)
```

#### ARRAYNEWELEM x y

The instruction `(array.new_elem x y)` is valid with the instruction type `i32 i32 -> (ref x)` if:

* The type `C.TYPES[x]` exists.
* The expansion of `C.TYPES[x]` is `(array (mut? rt))`.
* The element segment `C.ELEMS[y]` exists.
* The element segment `C.ELEMS[y]` matches the reference type `rt`.

```text
C.TYPES[x] ≈ array (mut? rt)
C |- C.ELEMS[y] <: rt
──────────────────────────────────────────────
C |- array.new_elem x y : i32 i32 -> (ref x)
```

#### ARRAYNEWDATA x y

The instruction `(array.new_data x y)` is valid with the instruction type `i32 i32 -> (ref x)` if:

* The type `C.TYPES[x]` exists.
* The expansion of `C.TYPES[x]` is `(array (mut? zt))`.
* The value type `unpack(zt)` is of the form `numtype` or `unpack(zt)` is of the form `vectype`.
* The data segment `C.DATAS[y]` exists.
* The data segment `C.DATAS[y]` is of the form `OK`.

```text
C.TYPES[x] ≈ array (mut? zt)
unpack(zt) = numtype ∨ unpack(zt) = vectype
C.DATAS[y] = OK
──────────────────────────────────────────────────
C |- array.new_data x y : i32 i32 -> (ref x)
```

#### ARRAYGET \_sx^? x

The instruction `(array.get_sx^? x)` is valid with the instruction type `(ref null x) i32 -> t` if:

* The type `C.TYPES[x]` exists.
* The expansion of `C.TYPES[x]` is `(array (mut? zt))`.
* The signedness `sx^?` is present if and only if `zt` is a packed type.
* The value type `t` is `unpack(zt)`.

```text
C.TYPES[x] ≈ array (mut? zt)
sx^? ≠ ε ⇔ zt ≠ unpack(zt)
──────────────────────────────────────────────
C |- array.get_sx^? x : (ref null x) i32 -> unpack(zt)
```

#### ARRAYSET x

The instruction `(array.set x)` is valid with the instruction type `(ref null x) i32 t -> ε` if:

* The type `C.TYPES[x]` exists.
* The expansion of `C.TYPES[x]` is `(array (mut zt))`.
* The value type `t` is `unpack(zt)`.

```text
C.TYPES[x] ≈ array (mut zt)
──────────────────────────────────────
C |- array.set x : (ref null x) i32 unpack(zt) -> ε
```

#### ARRAYLEN

The instruction `array.len` is valid with the instruction type `(ref null array) -> i32`.

```text
──────────────────────────────────
C |- array.len : (ref null array) -> i32
```

#### ARRAYFILL x

The instruction `(array.fill x)` is valid with the instruction type `(ref null x) i32 t i32 -> ε` if:

* The type `C.TYPES[x]` exists.
* The expansion of `C.TYPES[x]` is `(array (mut zt))`.
* The value type `t` is `unpack(zt)`.

```text
C.TYPES[x] ≈ array (mut zt)
────────────────────────────────────────────
C |- array.fill x : (ref null x) i32 unpack(zt) i32 -> ε
```

#### ARRAYCOPY x y

The instruction `(array.copy x1 x2)` is valid with the instruction type `(ref null x1) i32 (ref null x2) i32 i32 -> ε` if:

* The type `C.TYPES[x1]` exists.
* The expansion of `C.TYPES[x1]` is `(array (mut zt1))`.
* The type `C.TYPES[x2]` exists.
* The expansion of `C.TYPES[x2]` is `(array (mut? zt2))`.
* The storage type `zt2` matches the storage type `zt1`.

```text
C.TYPES[x1] ≈ array (mut zt1)
C.TYPES[x2] ≈ array (mut? zt2)
C |- zt2 <: zt1
────────────────────────────────────────────────────
C |- array.copy x1 x2 : (ref null x1) i32 (ref null x2) i32 i32 -> ε
```

#### ARRAYINITELEM x y

The instruction `(array.init_elem x y)` is valid with the instruction type `(ref null x) i32 i32 i32 -> ε` if:

* The type `C.TYPES[x]` exists.
* The expansion of `C.TYPES[x]` is `(array (mut zt))`.
* The element segment `C.ELEMS[y]` exists.
* The element segment `C.ELEMS[y]` matches the storage type `zt`.

```text
C.TYPES[x] ≈ array (mut zt)
C |- C.ELEMS[y] <: zt
────────────────────────────────────────────────────
C |- array.init_elem x y : (ref null x) i32 i32 i32 -> ε
```

#### ARRAYINITDATA x y

The instruction `(array.init_data x y)` is valid with the instruction type `(ref null x) i32 i32 i32 -> ε` if:

* The type `C.TYPES[x]` exists.
* The expansion of `C.TYPES[x]` is `(array (mut zt))`.
* The value type `unpack(zt)` is of the form `numtype` or `unpack(zt)` is of the form `vectype`.
* The data segment `C.DATAS[y]` exists.
* The data segment `C.DATAS[y]` is of the form `OK`.

```text
C.TYPES[x] ≈ array (mut zt)
unpack(zt) = numtype ∨ unpack(zt) = vectype
C.DATAS[y] = OK
──────────────────────────────────────────────────
C |- array.init_data x y : (ref null x) i32 i32 i32 -> ε
```

### Scalar Reference Instructions

#### REFI31

The instruction `ref.i31` is valid with the instruction type `i32 -> (ref i31)`.

```text
──────────────────────────
C |- ref.i31 : i32 -> (ref i31)
```

#### I31GET \_sx

The instruction `(i31.get_sx)` is valid with the instruction type `(ref null i31) -> i32`.

```text
──────────────────────────────────
C |- i31.get_sx : (ref null i31) -> i32
```

### External Reference Instructions

#### ANYCONVERTEXTERN

The instruction `any.convert_extern` is valid with the instruction type `(ref null?1 extern) -> (ref null?2 any)` if:

* `null?1` is of the form `null?2`.

```text
null?1 = null?2
──────────────────────────────────────────────
C |- any.convert_extern : (ref null?1 extern) -> (ref null?2 any)
```

#### EXTERNCONVERTANY

The instruction `extern.convert_any` is valid with the instruction type `(ref null?1 any) -> (ref null?2 extern)` if:

* `null?1` is of the form `null?2`.

```text
null?1 = null?2
──────────────────────────────────────────────
C |- extern.convert_any : (ref null?1 any) -> (ref null?2 extern)
```

### Numeric Instructions

#### t.CONST c

The instruction `(nt.CONST c_nt)` is valid with the instruction type `ε -> nt`.

```text
────────────────────────────
C |- nt.CONST c_nt : ε -> nt
```

#### t.unop

The instruction `(nt.unop_nt)` is valid with the instruction type `nt -> nt`.

```text
────────────────────────────
C |- nt.unop_nt : nt -> nt
```

#### t.binop

The instruction `(nt.binop_nt)` is valid with the instruction type `nt nt -> nt`.

```text
────────────────────────────────
C |- nt.binop_nt : nt nt -> nt
```

#### t.testop

The instruction `(nt.testop_nt)` is valid with the instruction type `nt -> i32`.

```text
────────────────────────────
C |- nt.testop_nt : nt -> i32
```

#### t.relop

The instruction `(nt.relop_nt)` is valid with the instruction type `nt nt -> i32`.

```text
────────────────────────────────
C |- nt.relop_nt : nt nt -> i32
```

#### t1.cvtop\_t2\_sx^?

The instruction `(nt1.cvtop_nt2)` is valid with the instruction type `nt2 -> nt1`.

```text
────────────────────────────────────
C |- nt1.cvtop_nt2 : nt2 -> nt1
```

### Vector Instructions

Vector instructions can have a prefix to describe the shape of the operand. Packed numeric types, `i8` and `i16`, are not value types. An auxiliary function maps such packed type shapes to value types:

```text
unpack(ntN shape M) = unpack(ntN)
```

#### V128.VCONST c

The instruction `(v128.const c)` is valid with the instruction type `ε -> v128`.

```text
──────────────────────────
C |- v128.const c : ε -> v128
```

#### V128.vvunop

The instruction `(v128.vvunop)` is valid with the instruction type `v128 -> v128`.

```text
────────────────────────────
C |- v128.vvunop : v128 -> v128
```

#### V128.vvbinop

The instruction `(v128.vvbinop)` is valid with the instruction type `v128 v128 -> v128`.

```text
────────────────────────────────
C |- v128.vvbinop : v128 v128 -> v128
```

#### V128.vvternop

The instruction `(v128.vvternop)` is valid with the instruction type `v128 v128 v128 -> v128`.

```text
────────────────────────────────────
C |- v128.vvternop : v128 v128 v128 -> v128
```

#### V128.vvtestop

The instruction `(v128.vvtestop)` is valid with the instruction type `v128 -> i32`.

```text
────────────────────────────
C |- v128.vvtestop : v128 -> i32
```

#### shape.vunop

The instruction `(sh.vunop)` is valid with the instruction type `v128 -> v128`.

```text
──────────────────────────
C |- sh.vunop : v128 -> v128
```

#### shape.vbinop

The instruction `(sh.vbinop)` is valid with the instruction type `v128 v128 -> v128`.

```text
──────────────────────────────
C |- sh.vbinop : v128 v128 -> v128
```

#### shape.vternop

The instruction `(sh.vternop)` is valid with the instruction type `v128 v128 v128 -> v128`.

```text
──────────────────────────────────
C |- sh.vternop : v128 v128 v128 -> v128
```

#### shape.vtestop

The instruction `(sh.vtestop)` is valid with the instruction type `v128 -> i32`.

```text
──────────────────────────
C |- sh.vtestop : v128 -> i32
```

#### ishape.vishiftop

The instruction `(sh.vishiftop)` is valid with the instruction type `v128 i32 -> v128`.

```text
────────────────────────────
C |- sh.vishiftop : v128 i32 -> v128
```

#### ishape.VBITMASK

The instruction `(sh.bitmask)` is valid with the instruction type `v128 -> i32`.

```text
────────────────────────────
C |- sh.bitmask : v128 -> i32
```

#### i8x16.vswizzlop

The instruction `(sh.vswizzle)` is valid with the instruction type `v128 v128 -> v128`.

```text
──────────────────────────────
C |- sh.vswizzle : v128 v128 -> v128
```

#### i8x16.VSHUFFLE laneidx^16

The instruction `(sh.shuffle i*)` is valid with the instruction type `v128 v128 -> v128` if:

* For all `i` in `i*`:
   * The lane index `i` is less than `2 · shdim(sh)`.

```text
(i < 2 · shdim(sh))*
──────────────────────────────────
C |- sh.shuffle i* : v128 v128 -> v128
```

#### shape.VSPLAT

The instruction `(sh.splat)` is valid with the instruction type `numtype -> v128` if:

* The number type `numtype` is `unpack(sh)`.

```text
──────────────────────────
C |- sh.splat : unpack(sh) -> v128
```

#### shape.VEXTRACTLANE \_sx^? laneidx

The instruction `(sh.extract_lane_sx^? i)` is valid with the instruction type `v128 -> numtype` if:

* The lane index `i` is less than `shdim(sh)`.
* The number type `numtype` is `unpack(sh)`.

```text
i < shdim(sh)
──────────────────────────────────────────
C |- sh.extract_lane_sx^? i : v128 -> unpack(sh)
```

#### shape.VREPLACELANE laneidx

The instruction `(sh.replace_lane i)` is valid with the instruction type `v128 numtype -> v128` if:

* The lane index `i` is less than `shdim(sh)`.
* The number type `numtype` is `unpack(sh)`.

```text
i < shdim(sh)
──────────────────────────────────────────
C |- sh.replace_lane i : v128 unpack(sh) -> v128
```

#### ishape1.vextunop\_ishape2

The instruction `(sh1.vextunop_sh2)` is valid with the instruction type `v128 -> v128`.

```text
──────────────────────────────────────
C |- sh1.vextunop_sh2 : v128 -> v128
```

#### ishape1.vextbinop\_ishape2

The instruction `(sh1.vextbinop_sh2)` is valid with the instruction type `v128 v128 -> v128`.

```text
────────────────────────────────────────
C |- sh1.vextbinop_sh2 : v128 v128 -> v128
```

#### ishape1.vextternop\_ishape2

The instruction `(sh1.vextternop_sh2)` is valid with the instruction type `v128 v128 v128 -> v128`.

```text
────────────────────────────────────────────
C |- sh1.vextternop_sh2 : v128 v128 v128 -> v128
```

#### ishape1.VNARROW\_ishape2\_sx

The instruction `(sh1.narrow_sh2_sx)` is valid with the instruction type `v128 v128 -> v128`.

```text
────────────────────────────────────────
C |- sh1.narrow_sh2_sx : v128 v128 -> v128
```

#### shape.vcvtop\_half^?\_shape\_sx^?\_zero^?

The instruction `(sh1.vcvtop_sh2)` is valid with the instruction type `v128 -> v128`.

```text
──────────────────────────────────────
C |- sh1.vcvtop_sh2 : v128 -> v128
```

### Instruction Sequences

Typing of instruction sequences is defined recursively.

The instruction sequence `instr*` is valid with the instruction type `it` if:

* Either:
   * The instruction sequence `instr*` is empty.
   * The instruction type `it` is of the form `ε -> ε`.
* Or:
   * The instruction sequence `instr*` is of the form `instr'`.
   * The instruction type `it` is of the form `t1* ->_{x*} t2*`.
   * The instruction `instr'` is valid with the instruction type `t1* ->_{x*} t2*`.
* Or:
   * The instruction sequence `instr*` is of the form `instr1* instr2*`.
   * The instruction type `it` is of the form `t1* ->_{x1* x2*} t3*`.
   * The instruction sequence `instr1*` is valid with the instruction type `t1* ->_{x1*} t2*`.
   * For all `x1` in `x1*`:
      * The local `C.LOCALS[x1]` exists.
      * The local `C.LOCALS[x1]` is of the form `(init t)`.
   * Under the context `C` with the local types of `x1*` updated to `(set t)*`, the instruction sequence `instr2*` is valid with the instruction type `t2* ->_{x2*} t3*`.
* Or:
   * The instruction sequence `instr*` is valid with the instruction type `it''`.
   * The instruction type `it''` matches the instruction type `it`.
   * The instruction type `it` is valid.
* Or:
   * The instruction type `it` is of the form `t* t1* ->_{x*} t* t2*`.
   * The instruction sequence `instr*` is valid with the instruction type `t1* ->_{x*} t2*`.
   * The result type `t*` is valid.

```text
────────────────
C |- ε : ε -> ε
```

```text
C |- instr : t1* ->_{x*} t2*
──────────────────────────────────
C |- instr : t1* ->_{x*} t2*
```

```text
C |- instr1* : t1* ->_{x1*} t2*
(C.LOCALS[x1] = init t)*
C[.LOCAL[x1*] = (set t)*] |- instr2* : t2* ->_{x2*} t3*
──────────────────────────────────────────────────────────────────────
C |- instr1* instr2* : t1* ->_{x1* x2*} t3*
```

> **Note:** This *subsumption rule* allows to weaken the type of an instruction sequence to a supertype, which includes the ability to drop init variables `x*` from the instruction type in a context where they are not needed, for example, at the end of the body of a block.

```text
C |- instr* : it
C |- it <: it'
C |- it' : OK
────────────────────────────────────────────
C |- instr* : it'
```

```text
C |- instr* : t1* ->_{x*} t2*
C |- t* : OK
────────────────────────────────────────────────
C |- instr* : (t* t1*) ->_{x*} (t* t2*)
```

> **Note:** In combination with the previous two rules, this *frame rule* allows to compose instructions whose types would not directly fit otherwise. For example, consider the instruction sequence `(i32.const 1) (i32.const 2) (i32.add)`
>
> To type this sequence, its subsequence `(i32.const 2) (i32.add)` needs to be valid with an intermediate type.
>
> But the direct type of `(i32.const 2)` is `ε -> i32`, not matching the two inputs expected by `i32.add`.
>
> The rule allows to weaken the type of `(i32.const 2)` to the supertype `i32 -> i32 i32`, such that it can be composed with `i32.add` and yields the intermediate type `i32 -> i32 i32` for the subsequence. That can in turn be composed with the first constant.

### Expressions

Expressions `expr` are classified by result types `t*`.

The expression `instr*` is valid with the result type `t*` if:

* The instruction sequence `instr*` is valid with the instruction type `ε -> t*`.

```text
C |- instr* : ε -> t*
──────────────────────────────
C |- instr* : t*
```

#### Constant Expressions

In a *constant* expression, all instructions must be constant.

`instr*` is constant if:

* For all `instr` in `instr*`:
   * `instr` is constant.

`instr` is constant if:

* Either:
   * The instruction `instr` is of the form `(nt.CONST c_nt)`.
* Or:
   * The instruction `instr` is of the form `(vt.CONST c_vt)`.
* Or:
   * The instruction `instr` is of the form `(ref.null ht)`.
* Or:
   * The instruction `instr` is of the form `ref.i31`.
* Or:
   * The instruction `instr` is of the form `(ref.func x)`.
* Or:
   * The instruction `instr` is of the form `(struct.new x)`.
* Or:
   * The instruction `instr` is of the form `(struct.new_default x)`.
* Or:
   * The instruction `instr` is of the form `(array.new x)`.
* Or:
   * The instruction `instr` is of the form `(array.new_default x)`.
* Or:
   * The instruction `instr` is of the form `(array.new_fixed x n)`.
* Or:
   * The instruction `instr` is of the form `any.convert_extern`.
* Or:
   * The instruction `instr` is of the form `extern.convert_any`.
* Or:
   * The instruction `instr` is of the form `(global.get x)`.
      * The global `C.GLOBALS[x]` exists.
      * The global `C.GLOBALS[x]` is of the form `(ε t)`.
* Or:
   * The instruction `instr` is of the form `(ntI ntN.binop)`.
      * `ntI ntN` is contained in `[i32; i64]`.
      * `binop` is contained in `[add; sub; mul]`.

```text
(C |- instrconst instr const)*
──────────────────────────────────
C |- exprconst instr* const

C |- instrconst (nt.CONST c_nt) const
C |- instrconst (vt.CONST c_vt) const
C |- instrconst (ref.null ht) const
C |- instrconst (ref.i31) const
C |- instrconst (ref.func x) const
C |- instrconst (struct.new x) const
C |- instrconst (struct.new_default x) const
C |- instrconst (array.new x) const
C |- instrconst (array.new_default x) const
C |- instrconst (array.new_fixed x n) const
C |- instrconst (any.convert_extern) const
C |- instrconst (extern.convert_any) const

ntI ntN ∈ i32 i64      binop ∈ add sub mul
────────────────────────────────────────────
C |- instrconst (ntI ntN.binop) const

C.GLOBALS[x] = t
──────────────────────
C |- instrconst (global.get x) const
```

> **Note:** Currently, constant expressions occurring in globals are further constrained in that contained `global.get` instructions are only allowed to refer to *imported* or *previously defined* globals. Constant expressions occurring in tables may only have `global.get` instructions that refer to *imported* globals.
>
> This is enforced in the validation rule for modules by constraining the context `C` accordingly.
>
> The definition of constant expression may be extended in future versions of WebAssembly.
