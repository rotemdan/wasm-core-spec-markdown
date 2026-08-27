## Instructions

WebAssembly computation is performed by executing individual [instructions](syntax-instr).

### Parametric Instructions

#### nop

1. Do nothing.

```text
nop -> ε
```

#### unreachable

1. Trap.

```text
unreachable -> trap
```

#### drop

1. Assert: Due to [validation](valid-drop), a value is on the top of the stack.
2. Pop the value `val` from the stack.

```text
val drop -> ε
```

#### select (t*)^?

1. Assert: Due to [validation](valid-select), a value of [number type](syntax-numtype) `i32` is on the top of the stack.
2. Pop the value `(i32.const c)` from the stack.
3. Assert: Due to [validation](valid-select), a value is on the top of the stack.
4. Pop the value `val_2` from the stack.
5. Assert: Due to [validation](valid-select), a value is on the top of the stack.
6. Pop the value `val_1` from the stack.
7. If `c ≠ 0`, then:
   1. Push the value `val_1` to the stack.
8. Else:
   1. Push the value `val_2` to the stack.

```text
val1 val2 (i32.const c) (select (t*)^?) -> val1   (if c ≠ 0)
val1 val2 (i32.const c) (select (t*)^?) -> val2   (if c = 0)
```

> **Note:** In future versions of WebAssembly, `select` may allow more than one value per choice.

### Control Instructions

#### block bt instr*

1. Let `z` be the current state.
2. Let `t1^m ->_{localidx_0*} t2^n` be the destructuring of `fblocktype_z(bt)`.
3. Assert: Due to [validation](valid-block), `localidx_0* = ε`.
4. Assert: Due to [validation](valid-block), there are at least `m` values on the top of the stack.
5. Pop the values `val^m` from the stack.
6. Let `L` be the `label` whose arity is `n` and whose continuation is the end of the block.
7. Enter the block `val^m instr*` with the `label` `L`.

```text
z ; val^m (block bt instr*) -> (label_n { ε } val^m instr*)   (if fblocktype_z(bt) = t1^m -> t2^n)
```

#### loop bt instr*

1. Let `z` be the current state.
2. Let `t1^m ->_{localidx_0*} t2^n` be the destructuring of `fblocktype_z(bt)`.
3. Assert: Due to [validation](valid-loop), `localidx_0* = ε`.
4. Assert: Due to [validation](valid-loop), there are at least `m` values on the top of the stack.
5. Pop the values `val^m` from the stack.
6. Let `L` be the `label` whose arity is `m` and whose continuation is the start of the block.
7. Enter the block `val^m instr*` with the `label` `L`.

```text
z ; val^m (loop bt instr*) -> (label_m { loop bt instr* } val^m instr*)   (if fblocktype_z(bt) = t1^m -> t2^n)
```

#### if bt instr1* else instr2*

1. Assert: Due to validation, a value of [number type](syntax-numtype) `i32` is on the top of the stack.
2. Pop the value `(i32.const c)` from the stack.
3. If `c ≠ 0`, then:
   1. Execute the instruction `(block bt instr1*)`.
4. Else:
   1. Execute the instruction `(block bt instr2*)`.

```text
(i32.const c) (if bt instr1* else instr2*) -> (block bt instr1*)   (if c ≠ 0)
(i32.const c) (if bt instr1* else instr2*) -> (block bt instr2*)   (if c = 0)
```

#### br l

1. If the first non-value entry of the stack is a `label`, then:
   1. Let `L` be the topmost `label`.
   2. Let `n` be the arity of `L`.
   3. If `l = 0`, then:
      1. Assert: Due to [validation](valid-br), there are at least `n` values on the top of the stack.
      2. Pop the values `val^n` from the stack.
      3. Pop all values `val'*` from the top of the stack.
      4. Pop the `label` from the stack.
      5. Push the values `val^n` to the stack.
      6. Jump to the continuation of `L`.
   4. Else:
      1. Pop all values `val*` from the top of the stack.
      2. Pop the `label` from the stack.
      3. Push the values `val*` to the stack.
      4. Execute the instruction `(br l - 1)`.
2. Else:
   1. Assert: Due to [validation](valid-br), the first non-value entry of the stack is a `handler`.
   2. Pop all values `val*` from the top of the stack.
   3. Pop the `handler` from the stack.
   4. Push the values `val*` to the stack.
   5. Execute the instruction `(br l)`.

```text
(label_n { instr'* } val'* val^n (br l) instr*) -> val^n instr'*   (if l = 0)
(label_n { instr'* } val* (br l) instr*) -> val* (br l - 1)   (if l > 0)
(handler_n { catch* } val* (br l) instr*) -> val* (br l)
```

#### br_if l

1. Assert: Due to [validation](valid-br_if), a value of [number type](syntax-numtype) `i32` is on the top of the stack.
2. Pop the value `(i32.const c)` from the stack.
3. If `c ≠ 0`, then:
   1. Execute the instruction `(br l)`.
4. Else:
   1. Do nothing.

```text
(i32.const c) (br_if l) -> (br l)   (if c ≠ 0)
(i32.const c) (br_if l) -> ε   (if c = 0)
```

#### br_table l* l'

1. Assert: Due to [validation](valid-br_table), a value of [number type](syntax-numtype) `i32` is on the top of the stack.
2. Pop the value `(i32.const i)` from the stack.
3. If `i < |l*|`, then:
   1. Execute the instruction `(br l*[i])`.
4. Else:
   1. Execute the instruction `(br l')`.

```text
(i32.const i) (br_table l* l') -> (br l*[i])   (if i < |l*|)
(i32.const i) (br_table l* l') -> (br l')   (if i ≥ |l*|)
```

#### br_on_null l

1. Assert: Due to [validation](valid-br_on_null), a value is on the top of the stack.
2. Pop the value `val` from the stack.
3. If `val = ref.null`, then:
   1. Execute the instruction `(br l)`.
4. Else:
   1. Push the value `val` to the stack.

```text
val (br_on_null l) -> (br l)   (if val = ref.null)
val (br_on_null l) -> val   (otherwise)
```

#### br_on_non_null l

1. Assert: Due to [validation](valid-br_on_non_null), a value is on the top of the stack.
2. Pop the value `val` from the stack.
3. If `val = ref.null`, then:
   1. Do nothing.
4. Else:
   1. Push the value `val` to the stack.
   2. Execute the instruction `(br l)`.

```text
val (br_on_non_null l) -> ε   (if val = ref.null)
val (br_on_non_null l) -> val (br l)   (otherwise)
```

#### br_on_cast l rt1 rt2

1. Let `f` be the topmost `frame`.
2. Assert: Due to [validation](valid-br_on_cast), a [reference value](syntax-ref) is on the top of the stack.
3. Pop the value `reff` from the stack.
4. Push the value `reff` to the stack.
5. If `reff` is [valid](valid-val) with type `insttype_{f.MODULE}(rt2)`, then:
   1. Execute the instruction `(br l)`.
6. Else:
   1. Do nothing.

```text
s ; f ; reff (br_on_cast l rt1 rt2) -> reff (br l)   (if s |- reff : insttype_{f.MODULE}(rt2))
s ; f ; reff (br_on_cast l rt1 rt2) -> reff   (otherwise)
```

#### br_on_cast_fail l rt1 rt2

1. Let `f` be the topmost `frame`.
2. Assert: Due to [validation](valid-br_on_cast_fail), a [reference value](syntax-ref) is on the top of the stack.
3. Pop the value `reff` from the stack.
4. Push the value `reff` to the stack.
5. If `reff` is [valid](valid-val) with type `insttype_{f.MODULE}(rt2)`, then:
   1. Do nothing.
6. Else:
   1. Execute the instruction `(br l)`.

```text
s ; f ; reff (br_on_cast_fail l rt1 rt2) -> reff   (if s |- reff : insttype_{f.MODULE}(rt2))
s ; f ; reff (br_on_cast_fail l rt1 rt2) -> reff (br l)   (otherwise)
```

#### return

1. If the first non-value entry of the stack is a `frame`, then:
   1. Let `f` be the topmost `frame`.
   2. Let `n` be the arity of `f`.
   3. Assert: Due to [validation](valid-return), there are at least `n` values on the top of the stack.
   4. Pop the values `val^n` from the stack.
   5. Pop all values `val'*` from the top of the stack.
   6. Pop the `frame` from the stack.
   7. Push the values `val^n` to the stack.
2. Else if the first non-value entry of the stack is a `label`, then:
   1. Pop all values `val*` from the top of the stack.
   2. Pop the `label` from the stack.
   3. Push the values `val*` to the stack.
   4. Execute the instruction `return`.
3. Else:
   1. Assert: Due to [validation](valid-return), the first non-value entry of the stack is a `handler`.
   2. Pop all values `val*` from the top of the stack.
   3. Pop the `handler` from the stack.
   4. Push the values `val*` to the stack.
   5. Execute the instruction `return`.

```text
(frame_n { f } val'* val^n return instr*) -> val^n
(label_n { instr'* } val* return instr*) -> val* return
(handler_n { catch* } val* return instr*) -> val* return
```

#### call x

1. Let `z` be the current state.
2. Assert: Due to [validation](valid-call), `x < |z.MODULE.IFUNCS|`.
3. Let `a` be the [address](syntax-addr) `z.MODULE.IFUNCS[x]`.
4. Assert: Due to [validation](valid-call), `a < |z.FUNCS|`.
5. Push the value `(ref.func a)` to the stack.
6. Execute the instruction `(call_ref z.FUNCS[a].ITYPE)`.

```text
z ; (call x) -> (ref.func a) (call_ref z.FUNCS[a].ITYPE)   (if z.MODULE.MIFUNCS[x] = a)
```

#### call_ref y

> **Note:** The formal rule for calling a non-null function reference is described [below](exec-invoke).

1. Assert: due to [validation](valid-call_ref), a null or [function reference](syntax-ref) is on the top of the stack.
2. Pop the reference value `r` from the stack.
3. If `r` is `ref.null ht`, then:
   1. Trap.
4. Assert: due to [validation](valid-call_ref), `r` is a [function reference](syntax-ref).
5. Let `ref.func a` be the reference `r`.
6. [Invoke](exec-invoke) the function instance at address `a`.

```text
z ; (ref.null) (call_ref y) -> z ; trap
```

#### call_indirect x y

1. Execute the instruction `(table.get x)`.
2. Execute the instruction `(ref.cast (ref null y))`.
3. Execute the instruction `(call_ref y)`.

```text
(call_indirect x y) -> (table.get x) (ref.cast (ref null y)) (call_ref y)
```

#### return_call x

1. Let `z` be the current state.
2. Assert: Due to [validation](valid-return_call), `x < |z.MODULE.IFUNCS|`.
3. Let `a` be the [address](syntax-addr) `z.MODULE.IFUNCS[x]`.
4. Assert: Due to [validation](valid-return_call), `a < |z.FUNCS|`.
5. Push the value `(ref.func a)` to the stack.
6. Execute the instruction `(return_call_ref z.FUNCS[a].ITYPE)`.

```text
z ; (return_call x) -> (ref.func a) (return_call_ref z.FUNCS[a].ITYPE)   (if z.MODULE.MIFUNCS[x] = a)
```

#### return_call_ref y

1. Let `z` be the current state.
2. If the first non-value entry of the stack is a `label`, then:
   1. Pop all values `val*` from the top of the stack.
   2. Pop the `label` from the stack.
   3. Push the values `val*` to the stack.
   4. Execute the instruction `(return_call_ref y)`.
3. Else if the first non-value entry of the stack is a `handler`, then:
   1. Pop all values `val*` from the top of the stack.
   2. Pop the `handler` from the stack.
   3. Push the values `val*` to the stack.
   4. Execute the instruction `(return_call_ref y)`.
4. Else:
   1. Assert: Due to [validation](valid-return_call_ref), the first non-value entry of the stack is a `frame`.
   2. Assert: Due to [validation](valid-return_call_ref), a value is on the top of the stack.
   3. Pop the value `val''` from the stack.
   4. If `val'' = ref.null`, then:
      1. Trap.
   5. Assert: Due to [validation](valid-return_call_ref), `val''` is some `ref.func funcaddr`.
   6. Let `(ref.func a)` be the destructuring of `val''`.
   7. Assert: Due to [validation](valid-return_call_ref), `a < |z.FUNCS|`.
   8. Assert: Due to [validation](valid-return_call_ref), the [expansion](aux-expand-deftype) of `z.FUNCS[a].ITYPE` is some `func t1^n -> t2^m`.
   9. Let `(func t1^n -> t2^m)` be the destructuring of the [expansion](aux-expand-deftype) of `z.FUNCS[a].ITYPE`.
   10. Assert: Due to [validation](valid-return_call_ref), there are at least `n` values on the top of the stack.
   11. Pop the values `val^n` from the stack.
   12. Pop all values `val'*` from the top of the stack.
   13. Pop the `frame` from the stack.
   14. Push the values `val^n` to the stack.
   15. Push the value `(ref.func a)` to the stack.
   16. Execute the instruction `(call_ref y)`.

```text
z ; (label_k { instr'* } val* (return_call_ref y) instr*) -> val* (return_call_ref y)
z ; (handler_k { catch* } val* (return_call_ref y) instr*) -> val* (return_call_ref y)
z ; (frame_k { f } val* (ref.null) (return_call_ref y) instr*) -> trap
z ; (frame_k { f } val'* val^n (ref.func a) (return_call_ref y) instr*) -> val^n (ref.func a) (call_ref y)   (if z.FUNCS[a].ITYPE ≈ func t1^n -> t2^m)
```

#### return_call_indirect x y

1. Execute the instruction `(table.get x)`.
2. Execute the instruction `(ref.cast (ref null y))`.
3. Execute the instruction `(return_call_ref y)`.

```text
(return_call_indirect x y) -> (table.get x) (ref.cast (ref null y)) (return_call_ref y)
```

#### throw x

1. Let `z` be the current state.
2. Assert: Due to [validation](valid-throw), `x < |z.MODULE.TAGS|`.
3. Assert: Due to [validation](valid-throw), the [expansion](aux-expand-deftype) of `z.TAGS[x].ITYPE` is some `func t^n -> resulttype`.
4. Let `(func t^n -> resulttype_0)` be the destructuring of the [expansion](aux-expand-deftype) of `z.TAGS[x].ITYPE`.
5. Assert: Due to [validation](valid-throw), `resulttype_0 = ε`.
6. Let `a` be the length of `z.EXNS`.
7. Assert: Due to [validation](valid-throw), there are at least `n` values on the top of the stack.
8. Pop the values `val^n` from the stack.
9. Let `exn` be the [exception instance](syntax-exninst) `{ itag z.MODULE.TAGS[x], ifields val^n }`.
10. Append `exn` to `z.EXNS`.
11. Push the value `(ref.exn a)` to the stack.
12. Execute the instruction `throw_ref`.

```text
z ; val^n (throw x) -> z[.EXNS =⊕ exn] ; (ref.exn a) throw_ref
  (if z.TAGS[x].ITYPE ≈ func t^n -> ε
   ∧ a = |z.EXNS|
   ∧ exn = { itag z.MODULE.TAGS[x], ifields val^n })
```

#### throw_ref

1. Let `z` be the current state.
2. Assert: Due to [validation](valid-throw_ref), a value is on the top of the stack.
3. Pop the value `val'` from the stack.
4. If `val' = ref.null`, then:
   1. Trap.
5. If `val'` is some `ref.exn exnaddr`, then:
   1. Let `(ref.exn a)` be the destructuring of `val'`.
   2. Pop all values `val*` from the top of the stack.
   3. If `val* ≠ ε`, then:
      1. Push the value `(ref.exn a)` to the stack.
      2. Execute the instruction `throw_ref`.
   4. Else if the first non-value entry of the stack is a `label`, then:
      1. Pop the `label` from the stack.
      2. Push the value `(ref.exn a)` to the stack.
      3. Execute the instruction `throw_ref`.
   5. Else:
      1. If the first non-value entry of the stack is a `frame`, then:
         1. Pop the `frame` from the stack.
         2. Push the value `(ref.exn a)` to the stack.
         3. Execute the instruction `throw_ref`.
      2. Else if the first non-value entry of the stack is not a `handler`, then:
         1. Throw the exception `val'` as a result.
      3. Else:
         1. Let `H` be the topmost `handler`.
         2. Let `n` be the arity of `H`.
         3. Let `catch''*` be the catch handler of `H`.
         4. If `catch''* = ε`, then:
            1. Pop the `handler` from the stack.
            2. Push the value `(ref.exn a)` to the stack.
            3. Execute the instruction `throw_ref`.
         5. Else if `a ≥ |z.EXNS|`, then:
            1. Let `catch_0 catch'*` be `catch''*`.
            2. If `catch_0` is some `catch_all labelidx`, then:
               1. Let `(catch_all l)` be the destructuring of `catch_0`.
               2. Pop the `handler` from the stack.
               3. Execute the instruction `(br l)`.
            3. Else if `catch_0` is not some `catch_all_ref labelidx`, then:
               1. Let `catch catch'*` be `catch''*`.
               2. Pop the `handler` from the stack.
               3. Let `H'` be the `handler` whose arity is `n` and whose catch handler is `catch'*`.
               4. Push the `handler` `H'`.
               5. Push the value `(ref.exn a)` to the stack.
               6. Execute the instruction `throw_ref`.
            4. Else:
               1. Let `(catch_all_ref l)` be the destructuring of `catch_0`.
               2. Pop the `handler` from the stack.
               3. Push the value `(ref.exn a)` to the stack.
               4. Execute the instruction `(br l)`.
         6. Else:
            1. Let `val*` be `z.EXNS[a].IFIELDS`.
            2. Let `catch_0 catch'*` be `catch''*`.
            3. If `catch_0` is some `catch tagidx labelidx`, then:
               1. Let `(catch x l)` be the destructuring of `catch_0`.
               2. If `x < |z.MODULE.TAGS|` and `z.EXNS[a].ITAG = z.MODULE.TAGS[x]`, then:
                  1. Pop the `handler` from the stack.
                  2. Push the values `val*` to the stack.
                  3. Execute the instruction `(br l)`.
               3. Else:
                  1. Let `catch catch'*` be `catch''*`.
                  2. Pop the `handler` from the stack.
                  3. Let `H'` be the `handler` whose arity is `n` and whose catch handler is `catch'*`.
                  4. Push the `handler` `H'`.
                  5. Push the value `(ref.exn a)` to the stack.
                  6. Execute the instruction `throw_ref`.
            4. Else if `catch_0` is some `catch_ref tagidx labelidx`, then:
               1. Let `(catch_ref x l)` be the destructuring of `catch_0`.
               2. If `x ≥ |z.MODULE.TAGS|` or `z.EXNS[a].ITAG ≠ z.MODULE.TAGS[x]`, then:
                  1. Let `catch catch'*` be `catch''*`.
                  2. Pop the `handler` from the stack.
                  3. Let `H'` be the `handler` whose arity is `n` and whose catch handler is `catch'*`.
                  4. Push the `handler` `H'`.
                  5. Push the value `(ref.exn a)` to the stack.
                  6. Execute the instruction `throw_ref`.
               3. Else:
                  1. Pop the `handler` from the stack.
                  2. Push the values `val*` to the stack.
                  3. Push the value `(ref.exn a)` to the stack.
                  4. Execute the instruction `(br l)`.
            5. Else:
               1. If `catch_0` is some `catch_all labelidx`, then:
                  1. Let `(catch_all l)` be the destructuring of `catch_0`.
                  2. Pop the `handler` from the stack.
                  3. Execute the instruction `(br l)`.
               2. Else if `catch_0` is not some `catch_all_ref labelidx`, then:
                  1. Let `catch catch'*` be `catch''*`.
                  2. Pop the `handler` from the stack.
                  3. Let `H` be the `handler` whose arity is `n` and whose catch handler is `catch'*`.
                  4. Push the `handler` `H`.
                  5. Push the value `(ref.exn a)` to the stack.
                  6. Execute the instruction `throw_ref`.
               3. Else:
                  1. Let `(catch_all_ref l)` be the destructuring of `catch_0`.
                  2. Pop the `handler` from the stack.
                  3. Push the value `(ref.exn a)` to the stack.
                  4. Execute the instruction `(br l)`.
6. Else:
   1. Assert: Due to [validation](valid-throw_ref), the first non-value entry of the stack is not a `label`.
   2. Assert: Due to [validation](valid-throw_ref), the first non-value entry of the stack is not a `frame`.
   3. Assert: Due to [validation](valid-throw_ref), the first non-value entry of the stack is not a `handler`.
   4. Throw the exception `val'` as a result.

```text
z ; (ref.null) throw_ref -> trap

z ; val* (ref.exn a) throw_ref instr* -> (ref.exn a) throw_ref
  (if val* ≠ ε ∨ instr* ≠ ε)

z ; (label_n { instr'* } (ref.exn a) throw_ref) -> (ref.exn a) throw_ref

z ; (frame_n { f } (ref.exn a) throw_ref) -> (ref.exn a) throw_ref

z ; (handler_n { ε } (ref.exn a) throw_ref) -> (ref.exn a) throw_ref

z ; (handler_n { (catch x l) catch'* } (ref.exn a) throw_ref) -> val* (br l)
  (if z.EXNS[a].ITAG = z.MODULE.TAGS[x]
   ∧ val* = z.EXNS[a].IFIELDS)

z ; (handler_n { (catch_ref x l) catch'* } (ref.exn a) throw_ref) -> val* (ref.exn a) (br l)
  (if z.EXNS[a].ITAG = z.MODULE.TAGS[x]
   ∧ val* = z.EXNS[a].IFIELDS)

z ; (handler_n { (catch_all l) catch'* } (ref.exn a) throw_ref) -> (br l)

z ; (handler_n { (catch_all_ref l) catch'* } (ref.exn a) throw_ref) -> (ref.exn a) (br l)

z ; (handler_n { catch catch'* } (ref.exn a) throw_ref) -> (handler_n { catch'* } (ref.exn a) throw_ref)   (otherwise)
```

#### try_table bt catch* instr*

1. Let `z` be the current state.
2. Let `t1^m ->_{localidx_0*} t2^n` be the destructuring of `fblocktype_z(bt)`.
3. Assert: Due to [validation](valid-try_table), `localidx_0* = ε`.
4. Assert: Due to [validation](valid-try_table), there are at least `m` values on the top of the stack.
5. Pop the values `val^m` from the stack.
6. Let `H` be the `handler` whose arity is `n` and whose catch handler is `catch*`.
7. Push the `handler` `H`.
8. Let `L` be the `label` whose arity is `n` and whose continuation is the end of the block.
9. Enter the block `val^m instr*` with the `label` `L`.

```text
z ; val^m (try_table bt catch* instr*) -> (handler_n { catch* } (label_n { ε } val^m instr*))
  (if fblocktype_z(bt) = t1^m -> t2^n)
```

### Blocks

The following auxiliary rules define the semantics of executing an [instruction sequence](syntax-instrs) that forms a [block](exec-instr-control).

#### Entering instr* with label L and values val*

1. Push `L` to the stack.
2. Push the values `val*` to the stack.
3. Jump to the start of the instruction sequence `instr*`.

> **Note:** No formal reduction rule is needed for entering an instruction sequence, because the label `L` is embedded in the [administrative instruction](syntax-instr-admin) that structured control instructions reduce to directly.

#### Exiting instr* with label L

When the end of a block is reached without a jump, [exception](exception), or [trap](trap) aborting it, then the following steps are performed.

1. Pop all values `val*` from the top of the stack.
2. Assert: due to [validation](valid-instrs), the label `L` is now on the top of the stack.
3. Pop the label from the stack.
4. Push `val*` back to the stack.
5. Jump to the position after the end of the [structured control instruction](syntax-instr-control) associated with the label `L`.

```text
(label_n { instr* } val*) -> val*
```

> **Note:** This semantics also applies to the instruction sequence contained in a `loop` instruction. Therefore, execution of a loop falls off the end, unless a backwards branch is performed explicitly.

### Exception Handling

The following auxiliary rules define the semantics of entering and exiting `try_table` blocks.

#### Entering instr* with label L and exception handler H

1. Push `H` to the stack.
2. Push `L` onto the stack.
3. Jump to the start of the instruction sequence `instr*`.

> **Note:** No formal reduction rule is needed for entering an exception [handler](syntax-handler) because it is an [administrative instruction](syntax-instr-admin) that the `try_table` instruction reduces to directly.

#### Exiting an exception handler

When the end of a `try_table` block is reached without a jump, [exception](exception), or [trap](trap), then the following steps are performed.

1. Let `m` be the number of values on the top of the stack.
2. Pop the values `val^m` from the stack.
3. Assert: due to [validation](valid-instrs), a handler and a label are now on the top of the stack.
4. Pop the label from the stack.
5. Pop the handler `H` from the stack.
6. Push `val^m` back to the stack.
7. Jump to the position after the end of the administrative instruction associated with the handler `H`.

```text
(handler_n { catch* } val*) -> val*
```

### Function Calls

The following auxiliary rules define the semantics of invoking a [function instance](syntax-funcinst) through one of the [call instructions](exec-instr-control) and returning from it.

#### Invocation of function reference (ref.func a)

1. Let `z` be the current state.
2. Assert: due to [validation](valid-call), `z.FUNCS[a]` exists.
3. Let `fi` be the [function instance](syntax-funcinst), `z.FUNCS[a]`.
4. Let `func [t1^n] -> [t2^m]` be the [composite type](syntax-comptype) `expanddt(fi.ITYPE)`.
5. Let `func x (local t)* instr*` be the [function](syntax-func) `fi.ICODE`.
6. Assert: due to [validation](valid-call), `n` values are on the top of the stack.
7. Pop the values `val^n` from the stack.
8. Let `f` be the [frame](syntax-frame) `{ LOCALS val^n (default_t)*, MODULE fi.IMODULE }`.
9. Push the activation of `f` with arity `m` to the stack.
10. Let `L` be the [label](syntax-label) whose arity is `m` and whose continuation is the end of the function.
11. [Enter](exec-instrs-enter) the instruction sequence `instr*` with label `L` and no values.

```text
z ; val^n (ref.func a) (call_ref y) -> z ; (frame_m { f } (label_m { ε } instr*))
  (if z.FUNCS[a] = fi
   ∧ fi.ITYPE ≈ func t1^n -> t2^m
   ∧ fi.ICODE = func x (local t)* (instr*)
   ∧ f = { LOCALS val^n (default_t)*, MODULE fi.IMODULE })
```

> **Note:** For non-defaultable types, the respective local is left uninitialized by these rules.

#### Returning from a function

When the end of a function is reached without a jump (including through `return`), or an [exception](exception) or [trap](trap) aborting it, then the following steps are performed.

1. Let `F` be the [current](exec-notation-textual) [frame](syntax-frame).
2. Let `n` be the arity of the activation of `F`.
3. Assert: due to [validation](valid-instrs), there are `n` values on the top of the stack.
4. Pop the results `val^n` from the stack.
5. Assert: due to [validation](valid-func), the frame `F` is now on the top of the stack.
6. Pop the frame from the stack.
7. Push `val^n` back to the stack.
8. Jump to the instruction after the original call.

```text
(frame_n { f } val^n) -> val^n
```

#### Host Functions

Invoking a [host function](syntax-hostfunc) has non-deterministic behavior.

It may either terminate with a [trap](trap), an [exception](exception), or return regularly.

However, in the latter case, it must consume and produce the right number and types of WebAssembly [values](syntax-val) on the stack, according to its [function type](syntax-functype).

A host function may also modify the [store](syntax-store).

However, all store modifications must result in an [extension](extend-store) of the original store, i.e., they must only modify mutable contents and must not have instances removed.

Furthermore, the resulting store must be [valid](valid-store), i.e., all data and code in it is well-typed.

```text
s ; f ; val^n (ref.func a) (call_ref y) -> s' ; f ; result
  (if (s ; f).FUNCS[a] = fi
   ∧ fi.ITYPE ≈ func t1^n -> t2^m
   ∧ fi.ICODE = hf
   ∧ (s', result) ∈ hf(s, val^n))

s ; f ; val^n (ref.func a) (call_ref y) -> s ; f ; (ref.func a) (call_ref y)
  (if (s ; f).FUNCS[a] = fi
   ∧ fi.ITYPE ≈ func t1^n -> t2^m
   ∧ fi.ICODE = hf
   ∧ ⊥ ∈ hf(s, val^n))
```

Here, `hf(s, val^n)` denotes the implementation-defined execution of host function `hf` in current store `s` with arguments `val^n`.

It yields a set of possible outcomes, where each element is either a pair of a modified store `s'` and a [result](syntax-result) or the special value `⊥` indicating divergence.

A host function is non-deterministic if there is at least one argument for which the set of outcomes is not singular.

For a WebAssembly implementation to be [sound](soundness) in the presence of host functions, every [host function instance](syntax-funcinst) must be [valid](valid-hostfuncinst), which means that it adheres to suitable pre- and post-conditions: under a [valid store](valid-store) `s`, and given arguments `val^n` matching the ascribed parameter types `t_1^n`, executing the host function must yield a non-empty set of possible outcomes each of which is either divergence or consists of a valid store `s'` that is an [extension](extend-store) of `s` and a result matching the ascribed return types `t_2^m`.

All these notions are made precise in the [Appendix](soundness).

> **Note:** A host function can call back into WebAssembly by [invoking](exec-invocation) a function [exported](syntax-export) from a [module](syntax-module).
>
> However, the effects of any such call are subsumed by the non-deterministic behavior allowed for the host function.

### Variable Instructions

#### local.get x

1. Let `z` be the current state.
2. Assert: Due to validation, `z.LOCALS[x]` is defined.
3. Let `val` be `z.LOCALS[x]`.
4. Push the value `val` to the stack.

```text
z ; (local.get x) -> val   (if z.LOCALS[x] = val)
```

#### local.set x

1. Let `z` be the current state.
2. Assert: Due to validation, a value is on the top of the stack.
3. Pop the value `val` from the stack.
4. Replace `z.LOCALS[x]` with `val`.

```text
z ; val (local.set x) -> z[.LOCALS[x] = val] ; ε
```

#### local.tee x

1. Assert: Due to validation, a value is on the top of the stack.
2. Pop the value `val` from the stack.
3. Push the value `val` to the stack.
4. Push the value `val` to the stack.
5. Execute the instruction `(local.set x)`.

```text
val (local.tee x) -> val val (local.set x)
```

#### global.get x

1. Let `z` be the current state.
2. Let `val` be the [value](syntax-val) `z.GLOBALS[x].value`.
3. Push the value `val` to the stack.

```text
z ; (global.get x) -> val   (if z.GLOBALS[x].value = val)
```

#### global.set x

1. Let `z` be the current state.
2. Assert: Due to validation, a value is on the top of the stack.
3. Pop the value `val` from the stack.
4. Replace `z.GLOBALS[x].value` with `val`.

```text
z ; val (global.set x) -> z[.GLOBALS[x].value = val] ; ε
```

### Table Instructions

#### table.get x

1. Let `z` be the current state.
2. Assert: Due to validation, a [number value](syntax-num) is on the top of the stack.
3. Pop the value `(at.const i)` from the stack.
4. If `i ≥ |z.TABLES[x].irefs|`, then:
   1. Trap.
5. Push the value `z.TABLES[x].irefs[i]` to the stack.

```text
z ; (at.const i) (table.get x) -> trap   (if i ≥ |z.TABLES[x].irefs|)
z ; (at.const i) (table.get x) -> z.TABLES[x].irefs[i]   (if i < |z.TABLES[x].irefs|)
```

#### table.set x

1. Let `z` be the current state.
2. Assert: Due to validation, a [reference value](syntax-ref) is on the top of the stack.
3. Pop the value `reff` from the stack.
4. Assert: Due to validation, a [number value](syntax-num) is on the top of the stack.
5. Pop the value `(at.const i)` from the stack.
6. If `i ≥ |z.TABLES[x].irefs|`, then:
   1. Trap.
7. Replace `z.TABLES[x].trefs[i]` with `reff`.

```text
z ; (at.const i) reff (table.set x) -> z ; trap   (if i ≥ |z.TABLES[x].irefs|)
z ; (at.const i) reff (table.set x) -> z[.TABLES[x].trefs[i] = reff] ; ε   (if i < |z.TABLES[x].irefs|)
```

#### table.size x

1. Let `z` be the current state.
2. Let `(at lim rt)` be the destructuring of `z.TABLES[x].itype`.
3. Let `n` be the length of `z.TABLES[x].irefs`.
4. Push the value `(at.const n)` to the stack.

```text
z ; (table.size x) -> (at.const n)
  (if |z.TABLES[x].irefs| = n
   ∧ z.TABLES[x].itype = at lim rt)
```

#### table.grow x

1. Let `z` be the current state.
2. Assert: Due to validation, a [number value](syntax-num) is on the top of the stack.
3. Pop the value `(at.const n)` from the stack.
4. Assert: Due to validation, a [reference value](syntax-ref) is on the top of the stack.
5. Pop the value `reff` from the stack.
6. Either:
   1. Let `ti` be the [table instance](syntax-tableinst) `growtable(z.TABLES[x], n, reff)`.
   2. Push the value `(at.const |z.TABLES[x].irefs|)` to the stack.
   3. Replace `z.TABLES[x]` with `ti`.
7. Or:
   1. Push the value `(at.const signed_{|at|}(-1))` to the stack.

```text
z ; reff (at.const n) (table.grow x) -> z[.TABLES[x] = ti] ; (at.const |z.TABLES[x].irefs|)
  (if ti = growtable(z.TABLES[x], n, reff))
z ; reff (at.const n) (table.grow x) -> z ; (at.const signed_{|at|}(-1))
```

> **Note:** The `table.grow` instruction is non-deterministic. It may either succeed, returning the old table size `sz`, or fail, returning `-1`.
>
> Failure *must* occur if the referenced table instance has a maximum size defined that would be exceeded. However, failure *can* occur in other cases as well. In practice, the choice depends on the [resources](impl-exec) available to the [embedder](embedder).

#### table.fill x

1. Let `z` be the current state.
2. Assert: Due to validation, a [number value](syntax-num) is on the top of the stack.
3. Pop the value `(at.const n)` from the stack.
4. Assert: Due to validation, a value is on the top of the stack.
5. Pop the value `val` from the stack.
6. Assert: Due to validation, a value of [number type](syntax-numtype) `at` is on the top of the stack.
7. Pop the value `(numtype0.const i)` from the stack.
8. If `i + n > |z.TABLES[x].irefs|`, then:
   1. Trap.
9. If `n = 0`, then:
   1. Do nothing.
10. Else:
    1. Push the value `(at.const i)` to the stack.
    2. Push the value `val` to the stack.
    3. Execute the instruction `(table.set x)`.
    4. Push the value `(at.const i + 1)` to the stack.
    5. Push the value `val` to the stack.
    6. Push the value `(at.const n - 1)` to the stack.
    7. Execute the instruction `(table.fill x)`.

```text
z ; (at.const i) val (at.const n) (table.fill x) -> trap   (if i + n > |z.TABLES[x].irefs|)
z ; (at.const i) val (at.const n) (table.fill x) -> ε   (otherwise, if n = 0)
z ; (at.const i) val (at.const n) (table.fill x) ->
    (at.const i) val (table.set x)
    (at.const i + 1) val (at.const n - 1) (table.fill x)   (otherwise)
```

#### table.copy x_1 x_2

1. Let `z` be the current state.
2. Assert: Due to validation, a [number value](syntax-num) is on the top of the stack.
3. Pop the value `(at.const n)` from the stack.
4. Assert: Due to validation, a [number value](syntax-num) is on the top of the stack.
5. Pop the value `(at2.const i2)` from the stack.
6. Assert: Due to validation, a [number value](syntax-num) is on the top of the stack.
7. Pop the value `(at1.const i1)` from the stack.
8. If `i1 + n > |z.TABLES[x_1].irefs|`, then:
   1. Trap.
9. If `i2 + n > |z.TABLES[x_2].irefs|`, then:
   1. Trap.
10. If `n = 0`, then:
    1. Do nothing.
11. Else:
    1. If `i1 ≤ i2`, then:
       1. Push the value `(at1.const i1)` to the stack.
       2. Push the value `(at2.const i2)` to the stack.
       3. Execute the instruction `(table.get x2)`.
       4. Execute the instruction `(table.set x1)`.
       5. Push the value `(at1.const i1 + 1)` to the stack.
       6. Push the value `(at2.const i2 + 1)` to the stack.
    2. Else:
       1. Push the value `(at1.const i1 + n - 1)` to the stack.
       2. Push the value `(at2.const i2 + n - 1)` to the stack.
       3. Execute the instruction `(table.get x2)`.
       4. Execute the instruction `(table.set x1)`.
       5. Push the value `(at1.const i1)` to the stack.
       6. Push the value `(at2.const i2)` to the stack.
    3. Push the value `(at.const n - 1)` to the stack.
    4. Execute the instruction `(table.copy x1 x2)`.

```text
z ; (at1.const i1) (at2.const i2) (at'.const n) (table.copy x y) -> trap
  (if i1 + n > |z.TABLES[x_1].irefs| ∨ i2 + n > |z.TABLES[x_2].irefs|)
z ; (at1.const i1) (at2.const i2) (at'.const n) (table.copy x y) -> ε   (otherwise, if n = 0)
z ; (at1.const i1) (at2.const i2) (at'.const n) (table.copy x y) ->
    (at1.const i1) (at2.const i2) (table.get x2) (table.set x1)
    (at1.const i1 + 1) (at2.const i2 + 1) (at'.const n - 1) (table.copy x y)   (otherwise, if i1 ≤ i2)
z ; (at1.const i1) (at2.const i2) (at'.const n) (table.copy x y) ->
    (at1.const i1 + n - 1) (at2.const i2 + n - 1) (table.get x2) (table.set x1)
    (at1.const i1) (at2.const i2) (at'.const n - 1) (table.copy x y)   (otherwise)
```

#### table.init x y

1. Let `z` be the current state.
2. Assert: Due to validation, a value of [number type](syntax-numtype) `i32` is on the top of the stack.
3. Pop the value `(i32.const n)` from the stack.
4. Assert: Due to validation, a value of [number type](syntax-numtype) `i32` is on the top of the stack.
5. Pop the value `(i32.const j)` from the stack.
6. Assert: Due to validation, a [number value](syntax-num) is on the top of the stack.
7. Pop the value `(at.const i)` from the stack.
8. If `i + n > |z.TABLES[x].irefs|`, then:
   1. Trap.
9. If `j + n > |z.ELEMS[y].erefs|`, then:
   1. Trap.
10. If `n = 0`, then:
    1. Do nothing.
11. Else:
    1. Assert: Due to validation, `j < |z.ELEMS[y].erefs|`.
    2. Push the value `(at.const i)` to the stack.
    3. Push the value `z.ELEMS[y].erefs[j]` to the stack.
    4. Execute the instruction `(table.set x)`.
    5. Push the value `(at.const i + 1)` to the stack.
    6. Push the value `(i32.const j + 1)` to the stack.
    7. Push the value `(i32.const n - 1)` to the stack.
    8. Execute the instruction `(table.init x y)`.

```text
z ; (at.const i) (i32.const j) (i32.const n) (table.init x y) -> trap
  (if i + n > |z.TABLES[x].irefs| ∨ j + n > |z.ELEMS[y].erefs|)
z ; (at.const i) (i32.const j) (i32.const n) (table.init x y) -> ε   (otherwise, if n = 0)
z ; (at.const i) (i32.const j) (i32.const n) (table.init x y) ->
    (at.const i) z.ELEMS[y].erefs[j] (table.set x)
    (at.const i + 1) (i32.const j + 1) (i32.const n - 1) (table.init x y)   (otherwise)
```

#### elem.drop x

1. Let `z` be the current state.
2. Replace `z.ELEMS[x].erefs` with `ε`.

```text
z ; (elem.drop x) -> z[.ELEMS[x].erefs = ε] ; ε
```

### Memory Instructions

> **Note:** The alignment `memarg.align` in load and store instructions does not affect the semantics. It is a hint that the offset `ea` at which the memory is accessed is intended to satisfy the property `ea mod 2^{memarg.align} = 0`.
>
> A WebAssembly implementation can use this hint to optimize for the intended use. Unaligned access violating that property is still allowed and must succeed regardless of the annotation. However, it may be substantially slower on some hardware.

#### nt.load loadop? x ao

1. Let `z` be the current state.
2. Assert: Due to validation, a [number value](syntax-num) is on the top of the stack.
3. Pop the value `(at.const i)` from the stack.
4. If `loadop?` is not defined, then:
   1. If `i + ao.offset + |nt| / 8 > |z.MEMS[x].ibytes|`, then:
      1. Trap.
   2. Let `c` be the result for which `bytes_nt(c) = z.MEMS[x].ibytes[i + ao.offset : |nt| / 8]`.
   3. Push the value `(nt.const c)` to the stack.
5. Else:
   1. Assert: Due to validation, `nt` is `iN`.
   2. Let `loadop_0` be `loadop?`.
   3. Let `n_sx` be the destructuring of `loadop_0`.
   4. If `i + ao.offset + n / 8 > |z.MEMS[x].ibytes|`, then:
      1. Trap.
   5. Let `c` be the result for which `bytes_{iN n}(c) = z.MEMS[x].ibytes[i + ao.offset : n / 8]`.
   6. Push the value `(iN.const extend_{n, |nt|}^{sx}(c))` to the stack.

```text
z ; (at.const i) (nt.load x ao) -> trap
  (if i + ao.offset + |nt| / 8 > |z.MEMS[x].ibytes|)
z ; (at.const i) (nt.load x ao) -> (nt.const c)
  (if bytes_nt(c) = z.MEMS[x].ibytes[i + ao.offset : |nt| / 8])
z ; (at.const i) (iN.load n_sx x ao) -> trap
  (if i + ao.offset + n / 8 > |z.MEMS[x].ibytes|)
z ; (at.const i) (iN.load n_sx x ao) -> (iN.const extend_{n, |iN|}^{sx}(c))
  (if bytes_{iN n}(c) = z.MEMS[x].ibytes[i + ao.offset : n / 8])
```

#### v128.load K shape M_sx x ao

1. Let `z` be the current state.
2. Assert: Due to validation, a [number value](syntax-num) is on the top of the stack.
3. Pop the value `(at.const i)` from the stack.
4. If `i + ao.offset + K * M / 8 > |z.MEMS[x].ibytes|`, then:
   1. Trap.
5. Let `j^M` be the result for which `(bytes_{iN K}(j) = z.MEMS[x].ibytes[i + ao.offset + k * K / 8 : K / 8])^{k<M}`.
6. Let `iN N` be the result for which `N = K * 2`.
7. Let `c` be `lanes^{-1}_{iN N shape M}((extend_{K, N}^{sx}(j))^{M})`.
8. Push the value `(v128.const c)` to the stack.

```text
z ; (at.const i) (v128.load K shape M_sx x ao) -> trap
  (if i + ao.offset + K * M / 8 > |z.MEMS[x].ibytes|)
z ; (at.const i) (v128.load K shape M_sx x ao) -> (v128.const c)
  (if (bytes_{iN K}(j) = z.MEMS[x].ibytes[i + ao.offset + k * K / 8 : K / 8])^{k<M}
   ∧ c = lanes^{-1}_{iN N shape M}((extend_{K, N}^{sx}(j))^{M}) ∧ N = K * 2)
```

#### v128.load N_splat x ao

1. Let `z` be the current state.
2. Assert: Due to validation, a [number value](syntax-num) is on the top of the stack.
3. Pop the value `(at.const i)` from the stack.
4. If `i + ao.offset + N / 8 > |z.MEMS[x].ibytes|`, then:
   1. Trap.
5. Let `M` be `128 / N`.
6. Let `iN N` be the result for which `|iN N| = N`.
7. Let `j` be the result for which `bytes_{iN N}(j) = z.MEMS[x].ibytes[i + ao.offset : N / 8]`.
8. Let `c` be `lanes^{-1}_{iN N shape M}(j^{M})`.
9. Push the value `(v128.const c)` to the stack.

```text
z ; (at.const i) (v128.load N_splat x ao) -> trap
  (if i + ao.offset + N / 8 > |z.MEMS[x].ibytes|)
z ; (at.const i) (v128.load N_splat x ao) -> (v128.const c)
  (if bytes_{iN N}(j) = z.MEMS[x].ibytes[i + ao.offset : N / 8]
   ∧ N = |iN N|
   ∧ M = 128 / N
   ∧ c = lanes^{-1}_{iN N shape M}(j^{M}))
```

#### v128.load N_zero x ao

1. Let `z` be the current state.
2. Assert: Due to validation, a [number value](syntax-num) is on the top of the stack.
3. Pop the value `(at.const i)` from the stack.
4. If `i + ao.offset + N / 8 > |z.MEMS[x].ibytes|`, then:
   1. Trap.
5. Let `j` be the result for which `bytes_{iN N}(j) = z.MEMS[x].ibytes[i + ao.offset : N / 8]`.
6. Let `c` be `extend_{N, 128}^{u}(j)`.
7. Push the value `(v128.const c)` to the stack.

```text
z ; (at.const i) (v128.load N_zero x ao) -> trap
  (if i + ao.offset + N / 8 > |z.MEMS[x].ibytes|)
z ; (at.const i) (v128.load N_zero x ao) -> (v128.const c)
  (if bytes_{iN N}(j) = z.MEMS[x].ibytes[i + ao.offset : N / 8]
   ∧ c = extend_{N, 128}^{u}(j))
```

#### v128.load N_lane x ao j

1. Let `z` be the current state.
2. Assert: Due to [validation](valid-vload_lane), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
3. Pop the value `(v128.const c1)` from the stack.
4. Assert: Due to [validation](valid-vload_lane), a [number value](syntax-num) is on the top of the stack.
5. Pop the value `(at.const i)` from the stack.
6. If `i + ao.offset + N / 8 > |z.MEMS[x].ibytes|`, then:
   1. Trap.
7. Let `M` be `|v128| / N`.
8. Let `iN N` be the result for which `|iN N| = N`.
9. Let `k` be the result for which `bytes_{iN N}(k) = z.MEMS[x].ibytes[i + ao.offset : N / 8]`.
10. Let `c` be `lanes^{-1}_{iN N shape M}(lanes_{iN N shape M}(c1)[[j] = k])`.
11. Push the value `(v128.const c)` to the stack.

```text
z ; (at.const i) (v128.const c1) (v128.load N_lane x ao j) -> trap
  (if i + ao.offset + N / 8 > |z.MEMS[x].ibytes|)
z ; (at.const i) (v128.const c1) (v128.load N_lane x ao j) -> (v128.const c)
  (if bytes_{iN N}(k) = z.MEMS[x].ibytes[i + ao.offset : N / 8]
   ∧ N = |iN N|
   ∧ M = |v128| / N
   ∧ c = lanes^{-1}_{iN N shape M}(lanes_{iN N shape M}(c1)[[j] = k]))
```

#### nt.store storeop? x ao

1. Let `z` be the current state.
2. Assert: Due to [validation](valid-store), a [number value](syntax-num) is on the top of the stack.
3. Pop the value `(nt'.const c)` from the stack.
4. Assert: Due to [validation](valid-store), a value is on the top of the stack.
5. Pop the value `(at.const i)` from the stack.
6. Assert: Due to [validation](valid-store), `nt = nt'`.
7. If `storeop?` is not defined, then:
   1. If `i + ao.offset + |nt'| / 8 > |z.MEMS[x].ibytes|`, then:
      1. Trap.
   2. Let `b*` be `bytes_{nt'}(c)`.
   3. Replace `z.MEMS[x].bytes[i + ao.offset : |nt'| / 8]` with `b*`.
8. Else:
   1. Assert: Due to [validation](valid-store), `nt'` is `iN`.
   2. Let `n` be `storeop?`.
   3. If `i + ao.offset + n / 8 > |z.MEMS[x].ibytes|`, then:
      1. Trap.
   4. Let `b*` be `bytes_{iN n}(wrap_{|nt'|, n}(c))`.
   5. Replace `z.MEMS[x].bytes[i + ao.offset : n / 8]` with `b*`.

```text
z ; (at.const i) (nt.const c) (nt.store x ao) -> z ; trap
  (if i + ao.offset + |nt| / 8 > |z.MEMS[x].ibytes|)
z ; (at.const i) (nt.const c) (nt.store x ao) -> z[.MEMS[x].mbytes[i + ao.offset : |nt| / 8] = b*] ; ε
  (if b* = bytes_nt(c))
z ; (at.const i) (iN.const c) (iN.store n x ao) -> z ; trap
  (if i + ao.offset + n / 8 > |z.MEMS[x].ibytes|)
z ; (at.const i) (iN.const c) (iN.store n x ao) -> z[.MEMS[x].mbytes[i + ao.offset : n / 8] = b*] ; ε
  (if b* = bytes_{iN n}(wrap_{|iN|, n}(c)))
z ; (at.const i) (v128.const c) (v128.store x ao) -> z ; trap
  (if i + ao.offset + |v128| / 8 > |z.MEMS[x].ibytes|)
z ; (at.const i) (v128.const c) (v128.store x ao) -> z[.MEMS[x].mbytes[i + ao.offset : |v128| / 8] = b*] ; ε
  (if b* = bytes_{v128}(c))
```

#### v128.store N_lane x ao j

1. Let `z` be the current state.
2. Assert: Due to [validation](valid-vstore_lane), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
3. Pop the value `(v128.const c)` from the stack.
4. Assert: Due to [validation](valid-vstore_lane), a [number value](syntax-num) is on the top of the stack.
5. Pop the value `(at.const i)` from the stack.
6. If `i + ao.offset + N / 8 > |z.MEMS[x].ibytes|`, then:
   1. Trap.
7. Let `M` be `128 / N`.
8. Let `iN N` be the result for which `|iN N| = N`.
9. Assert: Due to [validation](valid-vstore_lane), `j < |lanes_{iN N shape M}(c)|`.
10. Let `b*` be `bytes_{iN N}(lanes_{iN N shape M}(c)[j])`.
11. Replace `z.MEMS[x].bytes[i + ao.offset : N / 8]` with `b*`.

```text
z ; (at.const i) (v128.const c) (v128.store N_lane x ao j) -> z ; trap
  (if i + ao.offset + N / 8 > |z.MEMS[x].ibytes|)
z ; (at.const i) (v128.const c) (v128.store N_lane x ao j) -> z[.MEMS[x].mbytes[i + ao.offset : N / 8] = b*] ; ε
  (if N = |iN N|
   ∧ M = 128 / N
   ∧ b* = bytes_{iN N}(lanes_{iN N shape M}(c)[j]))
```

#### memory.size x

1. Let `z` be the current state.
2. Let `(at lim page)` be the destructuring of `z.MEMS[x].itype`.
3. Let `n * 64 Ki` be the length of `z.MEMS[x].ibytes`.
4. Push the value `(at.const n)` to the stack.

```text
z ; (memory.size x) -> (at.const n)
  (if n * 64 Ki = |z.MEMS[x].ibytes|
   ∧ z.MEMS[x].itype = at lim page)
```

#### memory.grow x

1. Let `z` be the current state.
2. Assert: Due to validation, a [number value](syntax-num) is on the top of the stack.
3. Pop the value `(at.const n)` from the stack.
4. Either:
   1. Let `mi` be the [memory instance](syntax-meminst) `growmem(z.MEMS[x], n)`.
   2. Push the value `(at.const |z.MEMS[x].ibytes| / (64 Ki))` to the stack.
   3. Replace `z.MEMS[x]` with `mi`.
5. Or:
   1. Push the value `(at.const signed_{|at|}(-1))` to the stack.

```text
z ; (at.const n) (memory.grow x) -> z[.MEMS[x] = mi] ; (at.const |z.MEMS[x].ibytes| / 64 Ki)
  (if mi = growmem(z.MEMS[x], n))
z ; (at.const n) (memory.grow x) -> z ; (at.const signed_{|at|}(-1))
```

> **Note:** The `memory.grow` instruction is non-deterministic. It may either succeed, returning the old memory size `sz`, or fail, returning `-1`.
>
> Failure *must* occur if the referenced memory instance has a maximum size defined that would be exceeded. However, failure *can* occur in other cases as well. In practice, the choice depends on the [resources](impl-exec) available to the [embedder](embedder).

#### memory.fill x

1. Let `z` be the current state.
2. Assert: Due to validation, a [number value](syntax-num) is on the top of the stack.
3. Pop the value `(at.const n)` from the stack.
4. Assert: Due to validation, a value is on the top of the stack.
5. Pop the value `val` from the stack.
6. Assert: Due to validation, a value of [number type](syntax-numtype) `at` is on the top of the stack.
7. Pop the value `(numtype0.const i)` from the stack.
8. If `i + n > |z.MEMS[x].ibytes|`, then:
   1. Trap.
9. If `n = 0`, then:
   1. Do nothing.
10. Else:
    1. Push the value `(at.const i)` to the stack.
    2. Push the value `val` to the stack.
    3. Execute the instruction `(i32.store 8 x)`.
    4. Push the value `(at.const i + 1)` to the stack.
    5. Push the value `val` to the stack.
    6. Push the value `(at.const n - 1)` to the stack.
    7. Execute the instruction `(memory.fill x)`.

```text
z ; (at.const i) val (at.const n) (memory.fill x) -> trap   (if i + n > |z.MEMS[x].ibytes|)
z ; (at.const i) val (at.const n) (memory.fill x) -> ε   (otherwise, if n = 0)
z ; (at.const i) val (at.const n) (memory.fill x) ->
    (at.const i) val (i32.store 8 x)
    (at.const i + 1) val (at.const n - 1) (memory.fill x)   (otherwise)
```

#### memory.copy x_1 x_2

1. Let `z` be the current state.
2. Assert: Due to validation, a [number value](syntax-num) is on the top of the stack.
3. Pop the value `(at.const n)` from the stack.
4. Assert: Due to validation, a [number value](syntax-num) is on the top of the stack.
5. Pop the value `(at2.const i2)` from the stack.
6. Assert: Due to validation, a [number value](syntax-num) is on the top of the stack.
7. Pop the value `(at1.const i1)` from the stack.
8. If `i1 + n > |z.MEMS[x_1].ibytes|`, then:
   1. Trap.
9. If `i2 + n > |z.MEMS[x_2].ibytes|`, then:
   1. Trap.
10. If `n = 0`, then:
    1. Do nothing.
11. Else:
    1. If `i1 ≤ i2`, then:
       1. Push the value `(at1.const i1)` to the stack.
       2. Push the value `(at2.const i2)` to the stack.
       3. Execute the instruction `(i32.load 8_u x2)`.
       4. Execute the instruction `(i32.store 8 x1)`.
       5. Push the value `(at1.const i1 + 1)` to the stack.
       6. Push the value `(at2.const i2 + 1)` to the stack.
    2. Else:
       1. Push the value `(at1.const i1 + n - 1)` to the stack.
       2. Push the value `(at2.const i2 + n - 1)` to the stack.
       3. Execute the instruction `(i32.load 8_u x2)`.
       4. Execute the instruction `(i32.store 8 x1)`.
       5. Push the value `(at1.const i1)` to the stack.
       6. Push the value `(at2.const i2)` to the stack.
    3. Push the value `(at.const n - 1)` to the stack.
    4. Execute the instruction `(memory.copy x1 x2)`.

```text
z ; (at1.const i1) (at2.const i2) (at'.const n) (memory.copy x1 x2) -> trap
  (if i1 + n > |z.MEMS[x_1].ibytes| ∨ i2 + n > |z.MEMS[x_2].ibytes|)
z ; (at1.const i1) (at2.const i2) (at'.const n) (memory.copy x1 x2) -> ε   (otherwise, if n = 0)
z ; (at1.const i1) (at2.const i2) (at'.const n) (memory.copy x1 x2) ->
    (at1.const i1) (at2.const i2) (i32.load 8_u x2) (i32.store 8 x1)
    (at1.const i1 + 1) (at2.const i2 + 1) (at'.const n - 1) (memory.copy x1 x2)   (otherwise, if i1 ≤ i2)
z ; (at1.const i1) (at2.const i2) (at'.const n) (memory.copy x1 x2) ->
    (at1.const i1 + n - 1) (at2.const i2 + n - 1) (i32.load 8_u x2) (i32.store 8 x1)
    (at1.const i1) (at2.const i2) (at'.const n - 1) (memory.copy x1 x2)   (otherwise)
```

#### memory.init x y

1. Let `z` be the current state.
2. Assert: Due to validation, a value of [number type](syntax-numtype) `i32` is on the top of the stack.
3. Pop the value `(i32.const n)` from the stack.
4. Assert: Due to validation, a value of [number type](syntax-numtype) `i32` is on the top of the stack.
5. Pop the value `(i32.const j)` from the stack.
6. Assert: Due to validation, a [number value](syntax-num) is on the top of the stack.
7. Pop the value `(at.const i)` from the stack.
8. If `i + n > |z.MEMS[x].ibytes|`, then:
   1. Trap.
9. If `j + n > |z.DATAS[y].ibytes|`, then:
   1. Trap.
10. If `n = 0`, then:
    1. Do nothing.
11. Else:
    1. Assert: Due to validation, `j < |z.DATAS[y].ibytes|`.
    2. Push the value `(at.const i)` to the stack.
    3. Push the value `(i32.const z.DATAS[y].ibytes[j])` to the stack.
    4. Execute the instruction `(i32.store 8 x)`.
    5. Push the value `(at.const i + 1)` to the stack.
    6. Push the value `(i32.const j + 1)` to the stack.
    7. Push the value `(i32.const n - 1)` to the stack.
    8. Execute the instruction `(memory.init x y)`.

```text
z ; (at.const i) (i32.const j) (i32.const n) (memory.init x y) -> trap
  (if i + n > |z.MEMS[x].ibytes| ∨ j + n > |z.DATAS[y].ibytes|)
z ; (at.const i) (i32.const j) (i32.const n) (memory.init x y) -> ε   (otherwise, if n = 0)
z ; (at.const i) (i32.const j) (i32.const n) (memory.init x y) ->
    (at.const i) (i32.const z.DATAS[y].ibytes[j]) (i32.store 8 x)
    (at.const i + 1) (i32.const j + 1) (i32.const n - 1) (memory.init x y)   (otherwise)
```

#### data.drop x

1. Let `z` be the current state.
2. Replace `z.DATAS[x].dbytes` with `ε`.

```text
z ; (data.drop x) -> z[.DATAS[x].dbytes = ε] ; ε
```

### Reference Instructions

#### ref.null ht

1. Push the value `ref.null` to the stack.

```text
z ; (ref.null ht) -> ref.null
```

#### ref.func x

1. Let `z` be the current state.
2. Assert: Due to validation, `x < |z.MODULE.IFUNCS|`.
3. Push the value `(ref.func z.MODULE.IFUNCS[x])` to the stack.

```text
z ; (ref.func x) -> (ref.func z.MODULE.IFUNCS[x])
```

#### ref.is_null

1. Assert: Due to validation, a [reference value](syntax-ref) is on the top of the stack.
2. Pop the value `reff` from the stack.
3. If `reff = ref.null`, then:
   1. Push the value `(i32.const 1)` to the stack.
4. Else:
   1. Push the value `(i32.const 0)` to the stack.

```text
reff ref.is_null -> (i32.const 1)   (if reff = ref.null)
reff ref.is_null -> (i32.const 0)   (otherwise)
```

#### ref.as_non_null

1. Assert: Due to validation, a [reference value](syntax-ref) is on the top of the stack.
2. Pop the value `reff` from the stack.
3. If `reff = ref.null`, then:
   1. Trap.
4. Push the value `reff` to the stack.

```text
reff ref.as_non_null -> trap   (if reff = ref.null)
reff ref.as_non_null -> reff   (otherwise)
```

#### ref.eq

1. Assert: Due to validation, a [reference value](syntax-ref) is on the top of the stack.
2. Pop the value `reff2` from the stack.
3. Assert: Due to validation, a [reference value](syntax-ref) is on the top of the stack.
4. Pop the value `reff1` from the stack.
5. If `reff1 = ref.null` and `reff2 = ref.null`, then:
   1. Push the value `(i32.const 1)` to the stack.
6. Else if `reff1 = reff2`, then:
   1. Push the value `(i32.const 1)` to the stack.
7. Else:
   1. Push the value `(i32.const 0)` to the stack.

```text
reff1 reff2 ref.eq -> (i32.const 1)   (if reff1 = ref.null ∧ reff2 = ref.null)
reff1 reff2 ref.eq -> (i32.const 1)   (otherwise, if reff1 = reff2)
reff1 reff2 ref.eq -> (i32.const 0)   (otherwise)
```

#### ref.test rt

1. Let `f` be the topmost `frame`.
2. Assert: Due to validation, a [reference value](syntax-ref) is on the top of the stack.
3. Pop the value `reff` from the stack.
4. If `reff` is [valid](valid-val) with type `insttype_{f.MODULE}(rt)`, then:
   1. Push the value `(i32.const 1)` to the stack.
5. Else:
   1. Push the value `(i32.const 0)` to the stack.

```text
s ; f ; reff (ref.test rt) -> (i32.const 1)   (if s |- reff : insttype_{f.MODULE}(rt))
s ; f ; reff (ref.test rt) -> (i32.const 0)   (otherwise)

#### ref.cast rt

1. Let `f` be the topmost `frame`.
2. Assert: Due to validation, a [reference value](syntax-ref) is on the top of the stack.
3. Pop the value `reff` from the stack.
4. If not `reff` is [valid](valid-val) with type `insttype_{f.MODULE}(rt)`, then:
   1. Trap.
5. Push the value `reff` to the stack.

```text
s ; f ; reff (ref.cast rt) -> reff   (if s |- reff : insttype_{f.MODULE}(rt))
s ; f ; reff (ref.cast rt) -> trap   (otherwise)
```

#### ref.i31

1. Assert: Due to validation, a value of [number type](syntax-numtype) `i32` is on the top of the stack.
2. Pop the value `(i32.const i)` from the stack.
3. Push the value `(ref.i31 wrap_{32, 31}(i))` to the stack.

```text
(i32.const i) ref.i31 -> (ref.i31 wrap_{32, 31}(i))
```

#### i31.get_sx

1. Assert: Due to validation, a value is on the top of the stack.
2. Pop the value `val` from the stack.
3. If `val = ref.null`, then:
   1. Trap.
4. Assert: Due to validation, `val` is some `ref.i31 i`.
5. Let `(ref.i31 i)` be the destructuring of `val`.
6. Push the value `(i32.const extend_{31, 32}^{sx}(i))` to the stack.

```text
ref.null (i31.get_sx) -> trap
(ref.i31 i) (i31.get_sx) -> (i32.const extend_{31, 32}^{sx}(i))
```

#### struct.new x

1. Let `z` be the current state.
2. Assert: Due to validation, the [expansion](aux-expand-deftype) of `z.TYPES[x]` is some `struct list(fieldtype)`.
3. Let `(struct list_0)` be the destructuring of the [expansion](aux-expand-deftype) of `z.TYPES[x]`.
4. Let `((mut? zt)^n)` be `list_0`.
5. Let `a` be the length of `z.STRUCTS`.
6. Assert: Due to validation, there are at least `n` values on the top of the stack.
7. Pop the values `val^n` from the stack.
8. Let `si` be the [structure instance](syntax-structinst) `{ itype z.TYPES[x], ifields (packfield_{zt}(val))^n }`.
9. Push the value `(ref.struct a)` to the stack.
10. Append `si` to `z.STRUCTS`.

```text
z ; val^n (struct.new x) -> z[.STRUCTS =⊕ si] ; (ref.struct a)
  (if z.TYPES[x] ≈ struct ((mut? zt)^n)
   ∧ a = |z.STRUCTS|
   ∧ si = { itype z.TYPES[x], ifields (packfield_{zt}(val))^n })
```

#### struct.new_default x

1. Let `z` be the current state.
2. Assert: Due to validation, the [expansion](aux-expand-deftype) of `z.TYPES[x]` is some `struct list(fieldtype)`.
3. Let `(struct list_0)` be the destructuring of the [expansion](aux-expand-deftype) of `z.TYPES[x]`.
4. Let `((mut? zt)*)` be `list_0`.
5. Assert: Due to validation, for all `zt` in `zt*`, `default_{unpack(zt)}` is defined.
6. Let `val*` be the value sequence `ε`.
7. For each `zt` in `zt*`, do:
   1. Let `val` be `default_{unpack(zt)}`.
   2. Append `val` to `val*`.
8. Assert: Due to validation, `|val*| = |zt*|`.
9. Push the values `val*` to the stack.
10. Execute the instruction `(struct.new x)`.

```text
z ; (struct.new_default x) -> val* (struct.new x)
  (if z.TYPES[x] ≈ struct ((mut? zt)*)
   ∧ (default_{unpack(zt)} = val)^*)
```

#### struct.get_sx? x i

1. Let `z` be the current state.
2. Assert: Due to validation, a value is on the top of the stack.
3. Pop the value `val` from the stack.
4. If `val = ref.null`, then:
   1. Trap.
5. Assert: Due to validation, `val` is some `ref.struct structaddr`.
6. Let `(ref.struct a)` be the destructuring of `val`.
7. Assert: Due to validation, `i < |z.STRUCTS[a].ifields|`.
8. Assert: Due to validation, `a < |z.STRUCTS|`.
9. Assert: Due to validation, the [expansion](aux-expand-deftype) of `z.TYPES[x]` is some `struct list(fieldtype)`.
10. Let `(struct list_0)` be the destructuring of the [expansion](aux-expand-deftype) of `z.TYPES[x]`.
11. Let `((mut? zt)*)` be `list_0`.
12. Assert: Due to validation, `i < |zt*|`.
13. Push the value `unpackfield_{zt*[i]}^{sx?}(z.STRUCTS[a].ifields[i])` to the stack.

```text
z ; ref.null (struct.get_sx? x i) -> trap
z ; (ref.struct a) (struct.get_sx? x i) -> unpackfield_{zt*[i]}^{sx?}(z.STRUCTS[a].ifields[i])
  (if z.TYPES[x] ≈ struct ((mut? zt)*))
```

#### struct.set x i

1. Let `z` be the current state.
2. Assert: Due to validation, a value is on the top of the stack.
3. Pop the value `val` from the stack.
4. Assert: Due to validation, a value is on the top of the stack.
5. Pop the value `val'` from the stack.
6. If `val' = ref.null`, then:
   1. Trap.
7. Assert: Due to validation, `val'` is some `ref.struct structaddr`.
8. Let `(ref.struct a)` be the destructuring of `val'`.
9. Assert: Due to validation, the [expansion](aux-expand-deftype) of `z.TYPES[x]` is some `struct list(fieldtype)`.
10. Let `(struct list_0)` be the destructuring of the [expansion](aux-expand-deftype) of `z.TYPES[x]`.
11. Let `((mut? zt)*)` be `list_0`.
12. Assert: Due to validation, `i < |zt*|`.
13. Replace `z.STRUCTS[a].fields[i]` with `packfield_{zt*[i]}(val)`.

```text
z ; ref.null val (struct.set x i) -> z ; trap
z ; (ref.struct a) val (struct.set x i) -> z[.STRUCTS[a].fields[i] = packfield_{zt*[i]}(val)] ; ε
  (if z.TYPES[x] ≈ struct ((mut? zt)*))
```

#### array.new x

1. Assert: Due to validation, a value of [number type](syntax-numtype) `i32` is on the top of the stack.
2. Pop the value `(i32.const n)` from the stack.
3. Assert: Due to validation, a value is on the top of the stack.
4. Pop the value `val` from the stack.
5. Push the values `val^n` to the stack.
6. Execute the instruction `(array.new_fixed x n)`.

```text
val (i32.const n) (array.new x) -> val^n (array.new_fixed x n)
```

#### array.new_default x

1. Let `z` be the current state.
2. Assert: Due to validation, a value of [number type](syntax-numtype) `i32` is on the top of the stack.
3. Pop the value `(i32.const n)` from the stack.
4. Assert: Due to validation, the [expansion](aux-expand-deftype) of `z.TYPES[x]` is some `array fieldtype`.
5. Let `(array fieldtype_0)` be the destructuring of the [expansion](aux-expand-deftype) of `z.TYPES[x]`.
6. Let `(mut? zt)` be the destructuring of `fieldtype_0`.
7. Assert: Due to validation, `default_{unpack(zt)}` is defined.
8. Let `val` be `default_{unpack(zt)}`.
9. Push the values `val^n` to the stack.
10. Execute the instruction `(array.new_fixed x n)`.

```text
z ; (i32.const n) (array.new_default x) -> val^n (array.new_fixed x n)
  (if z.TYPES[x] ≈ array (mut? zt)
   ∧ default_{unpack(zt)} = val)
```

#### array.new_fixed x n

1. Let `z` be the current state.
2. Assert: Due to validation, the [expansion](aux-expand-deftype) of `z.TYPES[x]` is some `array fieldtype`.
3. Let `(array fieldtype_0)` be the destructuring of the [expansion](aux-expand-deftype) of `z.TYPES[x]`.
4. Let `(mut? zt)` be the destructuring of `fieldtype_0`.
5. Let `a` be the length of `z.ARRAYS`.
6. Assert: Due to validation, there are at least `n` values on the top of the stack.
7. Pop the values `val^n` from the stack.
8. Let `ai` be the [array instance](syntax-arrayinst) `{ itype z.TYPES[x], ifields (packfield_{zt}(val))^n }`.
9. Push the value `(ref.array a)` to the stack.
10. Append `ai` to `z.ARRAYS`.

```text
z ; val^n (array.new_fixed x n) -> z[.ARRAYS =⊕ ai] ; (ref.array a)
  (if z.TYPES[x] ≈ array (mut? zt)
   ∧ a = |z.ARRAYS| ∧ ai = { itype z.TYPES[x], ifields (packfield_{zt}(val))^n })
```

#### array.new_data x y

1. Let `z` be the current state.
2. Assert: Due to validation, a value of [number type](syntax-numtype) `i32` is on the top of the stack.
3. Pop the value `(i32.const n)` from the stack.
4. Assert: Due to validation, a value of [number type](syntax-numtype) `i32` is on the top of the stack.
5. Pop the value `(i32.const i)` from the stack.
6. Assert: Due to validation, the [expansion](aux-expand-deftype) of `z.TYPES[x]` is some `array fieldtype`.
7. Let `(array fieldtype_0)` be the destructuring of the [expansion](aux-expand-deftype) of `z.TYPES[x]`.
8. Let `(mut? zt)` be the destructuring of `fieldtype_0`.
9. If `i + n * |zt| / 8 > |z.DATAS[y].ibytes|`, then:
   1. Trap.
10. Let `byte**` be the result for which each `byte*` has length `|zt| / 8`, and the [concatenation](notation-concat) of `byte**` is `z.DATAS[y].ibytes[i : n * |zt| / 8]`.
11. Let `c^n` be the result for which `(bytes_{zt}(c^n) = byte*)^*`.
12. Push the values `(unpack(zt).const unpacknum_{zt}(c))^n` to the stack.
13. Execute the instruction `(array.new_fixed x n)`.

```text
z ; (i32.const i) (i32.const n) (array.new_data x y) -> trap
  (if z.TYPES[x] ≈ array (mut? zt)
   ∧ i + n * |zt| / 8 > |z.DATAS[y].ibytes|)
z ; (i32.const i) (i32.const n) (array.new_data x y) -> (unpack(zt).const unpacknum_{zt}(c))^n (array.new_fixed x n)
  (if z.TYPES[x] ≈ array (mut? zt)
   ∧ concat bytes_{zt}(c)^n = z.DATAS[y].ibytes[i : n * |zt| / 8])
```

#### array.new_elem x y

1. Let `z` be the current state.
2. Assert: Due to validation, a value of [number type](syntax-numtype) `i32` is on the top of the stack.
3. Pop the value `(i32.const n)` from the stack.
4. Assert: Due to validation, a value of [number type](syntax-numtype) `i32` is on the top of the stack.
5. Pop the value `(i32.const i)` from the stack.
6. If `i + n > |z.ELEMS[y].erefs|`, then:
   1. Trap.
7. Let `reff^n` be `z.ELEMS[y].erefs[i : n]`.
8. Push the values `reff^n` to the stack.
9. Execute the instruction `(array.new_fixed x n)`.

```text
z ; (i32.const i) (i32.const n) (array.new_elem x y) -> trap   (if i + n > |z.ELEMS[y].erefs|)
z ; (i32.const i) (i32.const n) (array.new_elem x y) -> reff^n (array.new_fixed x n)
  (if reff^n = z.ELEMS[y].erefs[i : n])
```

#### array.get_sx? x

1. Let `z` be the current state.
2. Assert: Due to validation, a value of [number type](syntax-numtype) `i32` is on the top of the stack.
3. Pop the value `(i32.const i)` from the stack.
4. Assert: Due to validation, a value is on the top of the stack.
5. Pop the value `val` from the stack.
6. If `val = ref.null`, then:
   1. Trap.
7. Assert: Due to validation, `val` is some `ref.array arrayaddr`.
8. Let `(ref.array a)` be the destructuring of `val`.
9. Assert: Due to validation, `a < |z.ARRAYS|`.
10. If `i ≥ |z.ARRAYS[a].ifields|`, then:
    1. Trap.
11. Assert: Due to validation, the [expansion](aux-expand-deftype) of `z.TYPES[x]` is some `array fieldtype`.
12. Let `(array fieldtype_0)` be the destructuring of the [expansion](aux-expand-deftype) of `z.TYPES[x]`.
13. Let `(mut? zt)` be the destructuring of `fieldtype_0`.
14. Push the value `unpackfield_{zt}^{sx?}(z.ARRAYS[a].ifields[i])` to the stack.

```text
z ; ref.null (i32.const i) (array.get_sx? x) -> trap
z ; (ref.array a) (i32.const i) (array.get_sx? x) -> trap   (if i ≥ |z.ARRAYS[a].ifields|)
z ; (ref.array a) (i32.const i) (array.get_sx? x) -> unpackfield_{zt}^{sx?}(z.ARRAYS[a].ifields[i])
  (if z.TYPES[x] ≈ array (mut? zt))
```

#### array.set x

1. Let `z` be the current state.
2. Assert: Due to validation, a value is on the top of the stack.
3. Pop the value `val` from the stack.
4. Assert: Due to validation, a value of [number type](syntax-numtype) `i32` is on the top of the stack.
5. Pop the value `(i32.const i)` from the stack.
6. Assert: Due to validation, a value is on the top of the stack.
7. Pop the value `val'` from the stack.
8. If `val' = ref.null`, then:
   1. Trap.
9. Assert: Due to validation, `val'` is some `ref.array arrayaddr`.
10. Let `(ref.array a)` be the destructuring of `val'`.
11. If `a < |z.ARRAYS|` and `i ≥ |z.ARRAYS[a].ifields|`, then:
    1. Trap.
12. Assert: Due to validation, the [expansion](aux-expand-deftype) of `z.TYPES[x]` is some `array fieldtype`.
13. Let `(array fieldtype_0)` be the destructuring of the [expansion](aux-expand-deftype) of `z.TYPES[x]`.
14. Let `(mut? zt)` be the destructuring of `fieldtype_0`.
15. Replace `z.ARRAYS[a].fields[i]` with `packfield_{zt}(val)`.

```text
z ; ref.null (i32.const i) val (array.set x) -> z ; trap
z ; (ref.array a) (i32.const i) val (array.set x) -> z ; trap   (if i ≥ |z.ARRAYS[a].ifields|)
z ; (ref.array a) (i32.const i) val (array.set x) -> z[.ARRAYS[a].fields[i] = packfield_{zt}(val)] ; ε
  (if z.TYPES[x] ≈ array (mut? zt))
```

#### array.len

1. Let `z` be the current state.
2. Assert: Due to validation, a value is on the top of the stack.
3. Pop the value `val` from the stack.
4. If `val = ref.null`, then:
   1. Trap.
5. Assert: Due to validation, `val` is some `ref.array arrayaddr`.
6. Let `(ref.array a)` be the destructuring of `val`.
7. Assert: Due to validation, `a < |z.ARRAYS|`.
8. Push the value `(i32.const |z.ARRAYS[a].ifields|)` to the stack.

```text
z ; ref.null array.len -> trap
z ; (ref.array a) array.len -> (i32.const |z.ARRAYS[a].ifields|)

#### array.fill x

1. Let `z` be the current state.
2. Assert: Due to validation, a value of [number type](syntax-numtype) `i32` is on the top of the stack.
3. Pop the value `(i32.const n)` from the stack.
4. Assert: Due to validation, a value is on the top of the stack.
5. Pop the value `val` from the stack.
6. Assert: Due to validation, a value of [number type](syntax-numtype) `i32` is on the top of the stack.
7. Pop the value `(i32.const i)` from the stack.
8. Assert: Due to validation, a value is on the top of the stack.
9. Pop the value `val'` from the stack.
10. If `val' = ref.null`, then:
    1. Trap.
11. Assert: Due to validation, `val'` is some `ref.array arrayaddr`.
12. Let `(ref.array a)` be the destructuring of `val'`.
13. If `a ≥ |z.ARRAYS|`, then:
    1. Do nothing.
14. Else if `i + n > |z.ARRAYS[a].ifields|`, then:
    1. Trap.
15. If `n = 0`, then:
    1. Do nothing.
16. Else:
    1. Push the value `(ref.array a)` to the stack.
    2. Push the value `(i32.const i)` to the stack.
    3. Push the value `val` to the stack.
    4. Execute the instruction `(array.set x)`.
    5. Push the value `(ref.array a)` to the stack.
    6. Push the value `(i32.const i + 1)` to the stack.
    7. Push the value `val` to the stack.
    8. Push the value `(i32.const n - 1)` to the stack.
    9. Execute the instruction `(array.fill x)`.

```text
z ; ref.null (i32.const i) val (i32.const n) (array.fill x) -> trap
z ; (ref.array a) (i32.const i) val (i32.const n) (array.fill x) -> trap   (if i + n > |z.ARRAYS[a].ifields|)
z ; (ref.array a) (i32.const i) val (i32.const n) (array.fill x) -> ε   (otherwise, if n = 0)
z ; (ref.array a) (i32.const i) val (i32.const n) (array.fill x) ->
    (ref.array a) (i32.const i) val (array.set x)
    (ref.array a) (i32.const i + 1) val (i32.const n - 1) (array.fill x)   (otherwise)
```

#### array.copy x_1 x_2

1. Let `z` be the current state.
2. Assert: Due to validation, a value of [number type](syntax-numtype) `i32` is on the top of the stack.
3. Pop the value `(i32.const n)` from the stack.
4. Assert: Due to validation, a value of [number type](syntax-numtype) `i32` is on the top of the stack.
5. Pop the value `(i32.const i2)` from the stack.
6. Assert: Due to validation, a value is on the top of the stack.
7. Pop the value `val` from the stack.
8. Assert: Due to validation, a value of [number type](syntax-numtype) `i32` is on the top of the stack.
9. Pop the value `(i32.const i1)` from the stack.
10. Assert: Due to validation, a value is on the top of the stack.
11. Pop the value `val'` from the stack.
12. If `val' = ref.null` and `val` is reference value, then:
    1. Trap.
13. If `val = ref.null` and `val'` is reference value, then:
    1. Trap.
14. If `val'` is some `ref.array arrayaddr`, then:
    1. Let `(ref.array a1)` be the destructuring of `val'`.
    2. If `val` is some `ref.array arrayaddr`, then:
       1. If `a1 < |z.ARRAYS|` and `i1 + n > |z.ARRAYS[a1].ifields|`, then:
          1. Trap.
       2. Let `(ref.array a2)` be the destructuring of `val`.
       3. If `a2 ≥ |z.ARRAYS|`, then:
          1. Do nothing.
       4. Else if `i2 + n > |z.ARRAYS[a2].ifields|`, then:
          1. Trap.
       5. If `n = 0`, then:
          1. Do nothing.
       6. Else:
          1. Assert: Due to validation, the [expansion](aux-expand-deftype) of `z.TYPES[x2]` is some `array fieldtype`.
          2. Let `(array fieldtype_0)` be the destructuring of the [expansion](aux-expand-deftype) of `z.TYPES[x2]`.
          3. Let `(mut? zt2)` be the destructuring of `fieldtype_0`.
          4. Let `sx?` be `sx(zt2)`.
          5. Push the value `(ref.array a1)` to the stack.
          6. If `i1 ≤ i2`, then:
             1. Push the value `(i32.const i1)` to the stack.
             2. Push the value `(ref.array a2)` to the stack.
             3. Push the value `(i32.const i2)` to the stack.
             4. Execute the instruction `(array.get_sx? x2)`.
             5. Execute the instruction `(array.set x1)`.
             6. Push the value `(ref.array a1)` to the stack.
             7. Push the value `(i32.const i1 + 1)` to the stack.
             8. Push the value `(ref.array a2)` to the stack.
             9. Push the value `(i32.const i2 + 1)` to the stack.
          7. Else:
             1. Push the value `(i32.const i1 + n - 1)` to the stack.
             2. Push the value `(ref.array a2)` to the stack.
             3. Push the value `(i32.const i2 + n - 1)` to the stack.
             4. Execute the instruction `(array.get_sx? x2)`.
             5. Execute the instruction `(array.set x1)`.
             6. Push the value `(ref.array a1)` to the stack.
             7. Push the value `(i32.const i1)` to the stack.
             8. Push the value `(ref.array a2)` to the stack.
             9. Push the value `(i32.const i2)` to the stack.
          8. Push the value `(i32.const n - 1)` to the stack.
          9. Execute the instruction `(array.copy x1 x2)`.

```text
z ; ref.null (i32.const i1) reff (i32.const i2) (i32.const n) (array.copy x1 x2) -> trap
z ; reff (i32.const i1) ref.null (i32.const i2) (i32.const n) (array.copy x1 x2) -> trap
z ; (ref.array a1) (i32.const i1) (ref.array a2) (i32.const i2) (i32.const n) (array.copy x1 x2) -> trap   (if i1 + n > |z.ARRAYS[a1].ifields|)
z ; (ref.array a1) (i32.const i1) (ref.array a2) (i32.const i2) (i32.const n) (array.copy x1 x2) -> trap   (if i2 + n > |z.ARRAYS[a2].ifields|)
z ; (ref.array a1) (i32.const i1) (ref.array a2) (i32.const i2) (i32.const n) (array.copy x1 x2) -> ε   (otherwise, if n = 0)
z ; (ref.array a1) (i32.const i1) (ref.array a2) (i32.const i2) (i32.const n) (array.copy x1 x2) ->
    (ref.array a1) (i32.const i1) (ref.array a2) (i32.const i2) (array.get_sx? x2) (array.set x1)
    (ref.array a1) (i32.const i1 + 1) (ref.array a2) (i32.const i2 + 1) (i32.const n - 1) (array.copy x1 x2)
    (otherwise, if z.TYPES[x2] ≈ array (mut? zt2) ∧ i1 ≤ i2 ∧ sx? = sx(zt2))
z ; (ref.array a1) (i32.const i1 + n - 1) (ref.array a2) (i32.const i2 + n - 1) (array.get_sx? x2) (array.set x1)
    (ref.array a1) (i32.const i1) (ref.array a2) (i32.const i2) (i32.const n - 1) (array.copy x1 x2)
    (otherwise, if z.TYPES[x2] ≈ array (mut? zt2) ∧ sx? = sx(zt2))
```

Where:

```text
sx(consttype) = ε
sx(packtype) = s
```

#### array.init_data x y

1. Let `z` be the current state.
2. Assert: Due to validation, a value of [number type](syntax-numtype) `i32` is on the top of the stack.
3. Pop the value `(i32.const n)` from the stack.
4. Assert: Due to validation, a value of [number type](syntax-numtype) `i32` is on the top of the stack.
5. Pop the value `(i32.const j)` from the stack.
6. Assert: Due to validation, a value of [number type](syntax-numtype) `i32` is on the top of the stack.
7. Pop the value `(i32.const i)` from the stack.
8. Assert: Due to validation, a value is on the top of the stack.
9. Pop the value `val` from the stack.
10. If `val = ref.null`, then:
    1. Trap.
11. Assert: Due to validation, `val` is some `ref.array arrayaddr`.
12. Let `(ref.array a)` be the destructuring of `val`.
13. If `a < |z.ARRAYS|` and `i + n > |z.ARRAYS[a].ifields|`, then:
    1. Trap.
14. If the [expansion](aux-expand-deftype) of `z.TYPES[x]` is some `array fieldtype`, then:
    1. Let `(array fieldtype_0)` be the destructuring of the [expansion](aux-expand-deftype) of `z.TYPES[x]`.
    2. Let `(mut? zt)` be the destructuring of `fieldtype_0`.
    3. If `j + n * |zt| / 8 > |z.DATAS[y].ibytes|`, then:
       1. Trap.
    4. If `n = 0`, then:
       1. Do nothing.
    5. Else:
       1. Let `c` be the result for which `bytes_{zt}(c) = z.DATAS[y].ibytes[j : |zt| / 8]`.
       2. Push the value `(ref.array a)` to the stack.
       3. Push the value `(i32.const i)` to the stack.
       4. Push the value `(unpack(zt).const unpacknum_{zt}(c))` to the stack.
       5. Execute the instruction `(array.set x)`.
       6. Push the value `(ref.array a)` to the stack.
       7. Push the value `(i32.const i + 1)` to the stack.
       8. Push the value `(i32.const j + |zt| / 8)` to the stack.
       9. Push the value `(i32.const n - 1)` to the stack.
       10. Execute the instruction `(array.init_data x y)`.
15. Else if `n = 0`, then:
    1. Do nothing.

```text
z ; ref.null (i32.const i) (i32.const j) (i32.const n) (array.init_data x y) -> trap
z ; (ref.array a) (i32.const i) (i32.const j) (i32.const n) (array.init_data x y) -> trap   (if i + n > |z.ARRAYS[a].ifields|)
z ; (ref.array a) (i32.const i) (i32.const j) (i32.const n) (array.init_data x y) -> trap
   (if z.TYPES[x] ≈ array (mut? zt) ∧ j + n * |zt| / 8 > |z.DATAS[y].ibytes|)
z ; (ref.array a) (i32.const i) (i32.const j) (i32.const n) (array.init_data x y) -> ε   (otherwise, if n = 0)
z ; (ref.array a) (i32.const i) (i32.const j) (i32.const n) (array.init_data x y) ->
    (ref.array a) (i32.const i) (unpack(zt).const unpacknum_{zt}(c)) (array.set x)
    (ref.array a) (i32.const i + 1) (i32.const j + |zt| / 8) (i32.const n - 1) (array.init_data x y)
    (otherwise, if z.TYPES[x] ≈ array (mut? zt) ∧ bytes_{zt}(c) = z.DATAS[y].ibytes[j : |zt| / 8])
```

#### array.init_elem x y

1. Let `z` be the current state.
2. Assert: Due to validation, a value of [number type](syntax-numtype) `i32` is on the top of the stack.
3. Pop the value `(i32.const n)` from the stack.
4. Assert: Due to validation, a value of [number type](syntax-numtype) `i32` is on the top of the stack.
5. Pop the value `(i32.const j)` from the stack.
6. Assert: Due to validation, a value of [number type](syntax-numtype) `i32` is on the top of the stack.
7. Pop the value `(i32.const i)` from the stack.
8. Assert: Due to validation, a value is on the top of the stack.
9. Pop the value `val` from the stack.
10. If `val = ref.null`, then:
    1. Trap.
11. Assert: Due to validation, `val` is some `ref.array arrayaddr`.
12. Let `(ref.array a)` be the destructuring of `val`.
13. If `a < |z.ARRAYS|` and `i + n > |z.ARRAYS[a].ifields|`, then:
    1. Trap.
14. If `j + n > |z.ELEMS[y].erefs|`, then:
    1. Trap.
15. If `n = 0`, then:
    1. Do nothing.
16. Else if `j < |z.ELEMS[y].erefs|`, then:
    1. Let `reff` be the [reference value](syntax-ref) `z.ELEMS[y].erefs[j]`.
    2. Push the value `(ref.array a)` to the stack.
    3. Push the value `(i32.const i)` to the stack.
    4. Push the value `reff` to the stack.
    5. Execute the instruction `(array.set x)`.
    6. Push the value `(ref.array a)` to the stack.
    7. Push the value `(i32.const i + 1)` to the stack.
    8. Push the value `(i32.const j + 1)` to the stack.
    9. Push the value `(i32.const n - 1)` to the stack.
    10. Execute the instruction `(array.init_elem x y)`.

```text
z ; ref.null (i32.const i) (i32.const j) (i32.const n) (array.init_elem x y) -> trap
z ; (ref.array a) (i32.const i) (i32.const j) (i32.const n) (array.init_elem x y) -> trap   (if i + n > |z.ARRAYS[a].ifields|)
z ; (ref.array a) (i32.const i) (i32.const j) (i32.const n) (array.init_elem x y) -> trap   (if j + n > |z.ELEMS[y].erefs|)
z ; (ref.array a) (i32.const i) (i32.const j) (i32.const n) (array.init_elem x y) -> ε   (otherwise, if n = 0)
z ; (ref.array a) (i32.const i) (i32.const j) (i32.const n) (array.init_elem x y) ->
    (ref.array a) (i32.const i) reff (array.set x)
    (ref.array a) (i32.const i + 1) (i32.const j + 1) (i32.const n - 1) (array.init_elem x y)
    (otherwise, if reff = z.ELEMS[y].erefs[j])
```

#### any.convert_extern

1. Assert: Due to validation, a value is on the top of the stack.
2. Pop the value `val` from the stack.
3. If `val = ref.null`, then:
   1. Push the value `ref.null` to the stack.
4. If `val` is some `ref.extern reff`, then:
   1. Let `(ref.extern reff)` be the destructuring of `val`.
   2. Push the value `reff` to the stack.

```text
ref.null any.convert_extern -> ref.null
(ref.extern reff) any.convert_extern -> reff
```

#### extern.convert_any

1. Assert: Due to validation, a [reference value](syntax-ref) is on the top of the stack.
2. Pop the value `reff` from the stack.
3. If `reff = ref.null`, then:
   1. Push the value `ref.null` to the stack.
4. Else:
   1. Push the value `(ref.extern reff)` to the stack.

```text
reff extern.convert_any -> ref.null   (if reff = ref.null)
reff extern.convert_any -> (ref.extern reff)   (otherwise)
```

### Numeric Instructions

Numeric instructions are defined in terms of the generic [numeric operators](exec-numeric).
The mapping of numeric instructions to their underlying operators is expressed by the following definition:

```text
op_{iN}(i_1,...,i_k) = i op_N(i_1,...,i_k)
op_{fN}(z_1,...,z_k) = f op_N(z_1,...,z_k)
```

And for [conversion operators](exec-cvtop):

```text
cvtop^{sx?}_{t1,t2}(c) = cvtop^{sx?}_{|t1|,|t2|}(c)
```

Where the underlying operators are partial, the corresponding instruction will [trap](trap) when the result is not defined.
Where the underlying operators are non-deterministic, because they may return one of multiple possible [NaN](syntax-nan) values, so are the corresponding instructions.

> **Note:** For example, the result of instruction `i32.add` applied to operands `i_1, i_2` invokes `add_{i32}(i_1, i_2)`, which maps to the generic `iadd_{32}(i_1, i_2)` via the above definition. Similarly, `i64.trunc_f32_s` applied to `z` invokes `trunc^{s}_{f32,i64}(z)`, which maps to the generic `truncs_{32,64}(z)`.

#### nt.const c

1. Push the value `(nt.const c)` to the stack.

> **Note:** No formal reduction rule is required for this instruction, since `const` instructions already are [values](syntax-val).

#### nt . unop

1. Assert: Due to [validation](valid-unop), a value of [number type](syntax-numtype) `nt` is on the top of the stack.
2. Pop the value `(numtype0.const c1)` from the stack.
3. If `unop_{nt}(c1)` is empty, then:
   1. Trap.
4. Let `c` be an element of `unop_{nt}(c1)`.
5. Push the value `(nt.const c)` to the stack.

```text
(nt.const c1) (nt . unop) -> (nt.const c)   (if c ∈ unop_{nt}(c1))
(nt.const c1) (nt . unop) -> trap   (if unop_{nt}(c1) = ε)
```

#### nt . binop

1. Assert: Due to [validation](valid-binop), a value of [number type](syntax-numtype) `nt` is on the top of the stack.
2. Pop the value `(numtype0.const c2)` from the stack.
3. Assert: Due to [validation](valid-binop), a [number value](syntax-num) is on the top of the stack.
4. Pop the value `(numtype0.const c1)` from the stack.
5. If `binop_{nt}(c1, c2)` is empty, then:
   1. Trap.
6. Let `c` be an element of `binop_{nt}(c1, c2)`.
7. Push the value `(nt.const c)` to the stack.

```text
(nt.const c1) (nt.const c2) (nt . binop) -> (nt.const c)   (if c ∈ binop_{nt}(c1, c2))
(nt.const c1) (nt.const c2) (nt . binop) -> trap   (if binop_{nt}(c1, c2) = ε)
```

#### nt . testop

1. Assert: Due to [validation](valid-testop), a value of [number type](syntax-numtype) `nt` is on the top of the stack.
2. Pop the value `(numtype0.const c1)` from the stack.
3. Let `c` be `testop_{nt}(c1)`.
4. Push the value `(i32.const c)` to the stack.

```text
(nt.const c1) (nt . testop) -> (i32.const c)   (if c = testop_{nt}(c1))
```

#### nt . relop

1. Assert: Due to [validation](valid-relop), a value of [number type](syntax-numtype) `nt` is on the top of the stack.
2. Pop the value `(numtype0.const c2)` from the stack.
3. Assert: Due to [validation](valid-relop), a [number value](syntax-num) is on the top of the stack.
4. Pop the value `(numtype0.const c1)` from the stack.
5. Let `c` be `relop_{nt}(c1, c2)`.
6. Push the value `(i32.const c)` to the stack.

```text
(nt.const c1) (nt.const c2) (nt . relop) -> (i32.const c)   (if c = relop_{nt}(c1, c2))
```

#### nt2 . cvtop_ nt1

1. Assert: Due to [validation](valid-cvtop), a value of [number type](syntax-numtype) `nt1` is on the top of the stack.
2. Pop the value `(numtype0.const c1)` from the stack.
3. If `cvtop_{nt1, nt2}(c1)` is empty, then:
   1. Trap.
4. Let `c` be an element of `cvtop_{nt1, nt2}(c1)`.
5. Push the value `(nt2.const c)` to the stack.

```text
(nt1.const c1) (nt2 . cvtop_ nt1) -> (nt2.const c)   (if c ∈ cvtop_{nt1, nt2}(c1))
(nt1.const c1) (nt2 . cvtop_ nt1) -> trap   (if cvtop_{nt1, nt2}(c1) = ε)
```

### Vector Instructions

Vector instructions that operate bitwise are handled as integer operations of respective bit width.

```text
op_{vN}(i_1,...,i_k) = i op_N(i_1,...,i_k)
```

Most other vector instructions are defined in terms of [numeric operators](exec-numeric) that are applied lane-wise according to the given [shape](syntax-shape).

```text
op_{t x N}(n_1,...,n_k) = lanes^{-1}_{t x N}(op_t(i_1,...,i_k)^*)
   (iff i_1^* = lanes_{t x N}(n_1) ∧ ... ∧ i_k^* = lanes_{t x N}(n_k))

For non-deterministic operators this definition is generalized to sets:

```text
op_{t x N}(n_1,...,n_k) = { lanes^{-1}_{t x N}(i*) | i* ∈ ×(op_t(i_1,...,i_k)^*) ∧ i_1^* = lanes_{t x N}(n_1) ∧ ... ∧ i_k^* = lanes_{t x N}(n_k) }
```

where `×(S_1 ... S_N)` transforms a sequence of `N` sets of values into a set of sequences of `N` values by computing the set product:

```text
×(S_1 ... S_N) = { x_1 ... x_N | x_1 ∈ S_1 ∧ ... ∧ x_N ∈ S_N }
```

The remaining vector operators use [individual definitions](op-vec).

> **Note:** For example, the result of instruction `i32x4.add` applied to operands `v_1, v_2` invokes `add_{i32x4}(v_1, v_2)`, which maps to `lanes^{-1}_{i32x4}(add_{i32}(i_1, i_2)^*)`, where `i_1^*` and `i_2^*` are sequences resulting from invoking `lanes_{i32x4}(v_1)` and `lanes_{i32x4}(v_2)` respectively.

#### v128.const c

1. Push the value `(v128.const c)` to the stack.

> **Note:** No formal reduction rule is required for this instruction, since `const` instructions are already [values](syntax-val).

#### v128 . vvunop

1. Assert: Due to [validation](valid-vvunop), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
2. Pop the value `(v128.const c1)` from the stack.
3. Assert: Due to [validation](valid-vvunop), `|vvunop_{v128}(c1)| > 0`.
4. Let `c` be an element of `vvunop_{v128}(c1)`.
5. Push the value `(v128.const c)` to the stack.

```text
(v128.const c1) (v128 . vvunop) -> (v128.const c)   (if c ∈ vvunop_{v128}(c1))
```

#### v128 . vvbinop

1. Assert: Due to [validation](valid-vvbinop), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
2. Pop the value `(v128.const c2)` from the stack.
3. Assert: Due to [validation](valid-vvbinop), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
4. Pop the value `(v128.const c1)` from the stack.
5. Assert: Due to [validation](valid-vvbinop), `|vvbinop_{v128}(c1, c2)| > 0`.
6. Let `c` be an element of `vvbinop_{v128}(c1, c2)`.
7. Push the value `(v128.const c)` to the stack.

```text
(v128.const c1) (v128.const c2) (v128 . vvbinop) -> (v128.const c)   (if c ∈ vvbinop_{v128}(c1, c2))
```

#### v128 . vvternop

1. Assert: Due to [validation](valid-vvternop), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
2. Pop the value `(v128.const c3)` from the stack.
3. Assert: Due to [validation](valid-vvternop), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
4. Pop the value `(v128.const c2)` from the stack.
5. Assert: Due to [validation](valid-vvternop), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
6. Pop the value `(v128.const c1)` from the stack.
7. Assert: Due to [validation](valid-vvternop), `|vvternop_{v128}(c1, c2, c3)| > 0`.
8. Let `c` be an element of `vvternop_{v128}(c1, c2, c3)`.
9. Push the value `(v128.const c)` to the stack.

```text
(v128.const c1) (v128.const c2) (v128.const c3) (v128 . vvternop) -> (v128.const c)
  (if c ∈ vvternop_{v128}(c1, c2, c3))
```

#### v128 . vany_true

1. Assert: Due to [validation](valid-vvtestop), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
2. Pop the value `(v128.const c1)` from the stack.
3. Let `c` be `inez_{|v128|}(c1)`.
4. Push the value `(i32.const c)` to the stack.

```text
(v128.const c1) (v128 . vany_true) -> (i32.const c)   (if c = inez_{|v128|}(c1))
```

#### sh . vunop

1. Assert: Due to [validation](valid-vunop), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
2. Pop the value `(v128.const c1)` from the stack.
3. If `vunop_{sh}(c1)` is empty, then:
   1. Trap.
4. Let `c` be an element of `vunop_{sh}(c1)`.
5. Push the value `(v128.const c)` to the stack.

```text
(v128.const c1) (sh . vunop) -> (v128.const c)   (if c ∈ vunop_{sh}(c1))
(v128.const c1) (sh . vunop) -> trap   (if vunop_{sh}(c1) = ε)
```

#### sh . vbinop

1. Assert: Due to [validation](valid-vbinop), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
2. Pop the value `(v128.const c2)` from the stack.
3. Assert: Due to [validation](valid-vbinop), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
4. Pop the value `(v128.const c1)` from the stack.
5. If `vbinop_{sh}(c1, c2)` is empty, then:
   1. Trap.
6. Let `c` be an element of `vbinop_{sh}(c1, c2)`.
7. Push the value `(v128.const c)` to the stack.

```text
(v128.const c1) (v128.const c2) (sh . vbinop) -> (v128.const c)   (if c ∈ vbinop_{sh}(c1, c2))
(v128.const c1) (v128.const c2) (sh . vbinop) -> trap   (if vbinop_{sh}(c1, c2) = ε)
```

#### sh . vternop

1. Assert: Due to [validation](valid-vternop), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
2. Pop the value `(v128.const c3)` from the stack.
3. Assert: Due to [validation](valid-vternop), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
4. Pop the value `(v128.const c2)` from the stack.
5. Assert: Due to [validation](valid-vternop), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
6. Pop the value `(v128.const c1)` from the stack.
7. If `vternop_{sh}(c1, c2, c3)` is empty, then:
   1. Trap.
8. Let `c` be an element of `vternop_{sh}(c1, c2, c3)`.
9. Push the value `(v128.const c)` to the stack.

```text
(v128.const c1) (v128.const c2) (v128.const c3) (sh . vternop) -> (v128.const c)   (if c ∈ vternop_{sh}(c1, c2, c3))
(v128.const c1) (v128.const c2) (v128.const c3) (sh . vternop) -> trap   (if vternop_{sh}(c1, c2, c3) = ε)
```

#### iN shape M . all_true

1. Assert: Due to [validation](valid-vtestop), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
2. Pop the value `(v128.const c1)` from the stack.
3. Let `i*` be `lanes_{iN N shape M}(c1)`.
4. Let `c` be `Π (inez_N(i)^*)`.
5. Push the value `(i32.const c)` to the stack.

```text
(v128.const c1) (iN shape M . all_true) -> (i32.const c)
  (if i* = lanes_{iN N shape M}(c1)
   ∧ c = Π (inez_N(i)^*))
```

#### sh . vrelop

1. Assert: Due to [validation](valid-vrelop), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
2. Pop the value `(v128.const c2)` from the stack.
3. Assert: Due to [validation](valid-vrelop), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
4. Pop the value `(v128.const c1)` from the stack.
5. Let `c` be `vrelop_{sh}(c1, c2)`.
6. Push the value `(v128.const c)` to the stack.

```text
(v128.const c1) (v128.const c2) (sh . vrelop) -> (v128.const c)   (if c = vrelop_{sh}(c1, c2))
```

#### sh . vshiftop

1. Assert: Due to [validation](valid-vshiftop), a value of [number type](syntax-numtype) `i32` is on the top of the stack.
2. Pop the value `(i32.const i)` from the stack.
3. Assert: Due to [validation](valid-vshiftop), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
4. Pop the value `(v128.const c1)` from the stack.
5. Let `c` be `vshiftop_{sh}(c1, i)`.
6. Push the value `(v128.const c)` to the stack.

```text
(v128.const c1) (i32.const i) (sh . vshiftop) -> (v128.const c)   (if c = vshiftop_{sh}(c1, i))
```

#### sh . bitmask

1. Assert: Due to [validation](valid-vbitmask), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
2. Pop the value `(v128.const c1)` from the stack.
3. Let `c` be `bitmask_{sh}(c1)`.
4. Push the value `(i32.const c)` to the stack.

```text
(v128.const c1) (sh . bitmask) -> (i32.const c)   (if c = bitmask_{sh}(c1))
```

#### sh . swizzle

1. Assert: Due to [validation](valid-vswizzlop), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
2. Pop the value `(v128.const c2)` from the stack.
3. Assert: Due to [validation](valid-vswizzlop), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
4. Pop the value `(v128.const c1)` from the stack.
5. Let `c` be `swizzlop_{sh}(c1, c2)`.
6. Push the value `(v128.const c)` to the stack.

```text
(v128.const c1) (v128.const c2) (sh . swizzle) -> (v128.const c)   (if c = swizzle_{sh}(c1, c2))
```

#### sh . shuffle

1. Assert: Due to [validation](valid-vshuffle), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
2. Pop the value `(v128.const c2)` from the stack.
3. Assert: Due to [validation](valid-vshuffle), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
4. Pop the value `(v128.const c1)` from the stack.
5. Let `c` be `vshuffle_{sh}(i*, c1, c2)`.
6. Push the value `(v128.const c)` to the stack.

```text
(v128.const c1) (v128.const c2) (sh . shuffle i*) -> (v128.const c)   (if c = vshuffle_{sh}(i*, c1, c2))
```

#### iN shape M . splat

1. Assert: Due to [validation](valid-vsplat), a value is on the top of the stack.
2. Pop the value `(numtype0.const c1)` from the stack.
3. Assert: Due to [validation](valid-vsplat), `numtype0 = unpack(iN N)`.
4. Let `c` be `lanes^{-1}_{iN N shape M}((packnum_{iN N}(c1))^M)`.
5. Push the value `(v128.const c)` to the stack.

```text
(unpack(iN N).const c1) (iN shape M . splat) -> (v128.const c)   (if c = lanes^{-1}_{iN N shape M}((packnum_{iN N}(c1))^M))
```

#### lanetype shape M . extract_lane sx'? i

1. Assert: Due to [validation](valid-vextract_lane), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
2. Pop the value `(v128.const c1)` from the stack.
3. If `sx'?` is not defined, then:
   1. Assert: Due to [validation](valid-vextract_lane), `lanetype` is number type.
   2. Assert: Due to [validation](valid-vextract_lane), `i < |lanes_{lanetype shape M}(c1)|`.
   3. Let `c2` be `lanes_{lanetype shape M}(c1)[i]`.
   4. Push the value `(lanetype.const c2)` to the stack.
4. Else:
   1. Assert: Due to [validation](valid-vextract_lane), `lanetype` is packed type.
   2. Let `sx` be `sx'?`.
   3. Assert: Due to [validation](valid-vextract_lane), `i < |lanes_{lanetype shape M}(c1)|`.
   4. Let `c2` be `extend_{|lanetype|, 32}^{sx}(lanes_{lanetype shape M}(c1)[i])`.
   5. Push the value `(i32.const c2)` to the stack.

```text
(v128.const c1) (lanetype shape M . extract_lane i) -> (lanetype.const c2)   (if c2 = lanes_{lanetype shape M}(c1)[i])
(v128.const c1) (pt shape M . extract_lane sx i) -> (i32.const c2)   (if c2 = extend_{|pt|, 32}^{sx}(lanes_{pt shape M}(c1)[i]))
```

#### iN shape M . replace_lane i

1. Assert: Due to [validation](valid-vreplace_lane), a value is on the top of the stack.
2. Pop the value `(numtype0.const c2)` from the stack.
3. Assert: Due to [validation](valid-vreplace_lane), `numtype0 = unpack(iN N)`.
4. Assert: Due to [validation](valid-vreplace_lane), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
5. Pop the value `(v128.const c1)` from the stack.
6. Let `c` be `lanes^{-1}_{iN N shape M}(lanes_{iN N shape M}(c1)[[i] = packnum_{iN N}(c2)])`.
7. Push the value `(v128.const c)` to the stack.

```text
(v128.const c1) (unpack(iN N).const c2) (iN shape M . replace_lane i) -> (v128.const c)
  (if c = lanes^{-1}_{iN N shape M}(lanes_{iN N shape M}(c1)[[i] = packnum_{iN N}(c2)]))
```

#### sh2 . extunop_ sh1

1. Assert: Due to [validation](valid-vextunop), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
2. Pop the value `(v128.const c1)` from the stack.
3. Let `c` be `extunop_{sh1, sh2}(c1)`.
4. Push the value `(v128.const c)` to the stack.

```text
(v128.const c1) (sh2 . extunop_ sh1) -> (v128.const c)   (if extunop_{sh1, sh2}(c1) = c)
```

#### sh2 . extbinop_ sh1

1. Assert: Due to [validation](valid-vextbinop), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
2. Pop the value `(v128.const c2)` from the stack.
3. Assert: Due to [validation](valid-vextbinop), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
4. Pop the value `(v128.const c1)` from the stack.
5. Let `c` be `extbinop_{sh1, sh2}(c1, c2)`.
6. Push the value `(v128.const c)` to the stack.

```text
(v128.const c1) (v128.const c2) (sh2 . extbinop_ sh1) -> (v128.const c)   (if extbinop_{sh1, sh2}(c1, c2) = c)
```

#### sh2 . extternop_ sh1

1. Assert: Due to [validation](valid-vextternop), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
2. Pop the value `(v128.const c3)` from the stack.
3. Assert: Due to [validation](valid-vextternop), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
4. Pop the value `(v128.const c2)` from the stack.
5. Assert: Due to [validation](valid-vextternop), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
6. Pop the value `(v128.const c1)` from the stack.
7. Let `c` be `extternop_{sh1, sh2}(c1, c2, c3)`.
8. Push the value `(v128.const c)` to the stack.

```text
(v128.const c1) (v128.const c2) (v128.const c3) (sh2 . extternop_ sh1) -> (v128.const c)
  (if extternop_{sh1, sh2}(c1, c2, c3) = c)
```

#### sh2 . narrow_ sh1_ sx

1. Assert: Due to [validation](valid-vnarrow), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
2. Pop the value `(v128.const c2)` from the stack.
3. Assert: Due to [validation](valid-vnarrow), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
4. Pop the value `(v128.const c1)` from the stack.
5. Let `c` be `narrow_{sh1, sh2}^{sx}(c1, c2)`.
6. Push the value `(v128.const c)` to the stack.

```text
(v128.const c1) (v128.const c2) (sh2 . narrow_ sh1_ sx) -> (v128.const c)   (if c = narrow_{sh1, sh2}^{sx}(c1, c2))
```

#### sh2 . cvtop_ sh1

1. Assert: Due to [validation](valid-vcvtop), a value of [vector type](syntax-vectype) `v128` is on the top of the stack.
2. Pop the value `(v128.const c1)` from the stack.
3. Let `c` be `cvtop_{sh1, sh2}(cvtop, c1)`.
4. Push the value `(v128.const c)` to the stack.

```text
(v128.const c1) (sh2 . cvtop_ sh1) -> (v128.const c)   (if c = cvtop_{sh1, sh2}(cvtop, c1))
```

### Expressions

An [expression](syntax-expr) is *evaluated* relative to a [current](exec-notation-textual) [frame](syntax-frame) pointing to its containing [module instance](syntax-moduleinst).

#### eval_expr instr*

1. Execute the sequence `instr*`.
2. Pop the value `val` from the stack.
3. Return `val`.

```text
z ; instr* ->* z' ; val*   (if z ; instr* ->* z' ; val*)
```

> **Note:** Evaluation iterates this reduction rule until reaching a value. Expressions constituting [function](syntax-func) bodies are executed during function [invocation](exec-invoke).

