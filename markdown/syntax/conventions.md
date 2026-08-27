## Conventions

WebAssembly is a programming language that has multiple concrete representations (its binary format and the text format). Both map to a common structure. For conciseness, this structure is described in the form of an *abstract syntax*. All parts of this specification are defined in terms of this abstract syntax.

### Grammar Notation

The following conventions are adopted in defining grammar rules for abstract syntax.

* Terminal symbols (atoms) are written in sans-serif font or in symbolic form: `i32`, `nop`, `->`, `[ , ]`.
* Nonterminal symbols are written in italic font: `valtype`, `instr`.
* `A^n` is a sequence of `n >= 0` iterations of `A`.
* `A*` is a possibly empty sequence of iterations of `A`. (This is a shorthand for `A^n` used where `n` is not relevant.)
* `A+` is a non-empty sequence of iterations of `A`. (This is a shorthand for `A^n` where `n >= 1`.)
* `A?` is an optional occurrence of `A`. (This is a shorthand for `A^n` where `n <= 1`.)
* Productions are written `sym ::= A_1 | ... | A_n`.
* Large productions may be split into multiple definitions, indicated by ending the first one with explicit ellipses, `sym ::= A_1 | ...`, and starting continuations with ellipses, `sym ::= ... | A_2`.
* Some productions are augmented with side conditions, "`iff condition`", that provide a shorthand for a combinatorial expansion of the production into many separate cases.
* If the same meta variable or non-terminal symbol appears multiple times in a production, then all those occurrences must have the same instantiation. (This is a shorthand for a side condition requiring multiple different variables to be equal.)

### Auxiliary Notation

When dealing with syntactic constructs the following notation is also used:

* `ε` denotes the empty sequence.
* `|s|` denotes the length of a sequence `s`.
* `s[i]` denotes the `i`-th element of a sequence `s`, starting from `0`.
* `s[i : n]` denotes the sub-sequence `s[i] ... s[i + n - 1]` of a sequence `s`.
* `s[[i] = A]` denotes the same sequence as `s`, except that the `i`-th element is replaced with `A`.
* `s[[i : n] = A^n]` denotes the same sequence as `s`, except that the sub-sequence `s[i : n]` is replaced with `A^n`.
* `s1 ⊕ s2` denotes the sequence `s1` concatenated with `s2`; this is equivalent to `s1 s2`, but used for clarity.
* `++ s*` denotes the flattened sequence, formed by concatenating all sequences `s_i` in `s*`.
* `A ∈ s` denotes that `A` is a member of the sequence `s`, that is, `s` is of the form `s1 A s2` for some sequences `s1`, `s2`.

Moreover, the following conventions are employed:

* The notation `x^n`, where `x` is a non-terminal symbol, is treated as a meta variable ranging over respective sequences of `x` (similarly for `x*`, `x+`, `x?`).
* When given a sequence `x^n`, then the occurrences of `x` in an iterated sequence `( ... x ... )^n` are assumed to denote the individual elements of `x^n`, respectively (similarly for `x*`, `x+`, `x?`). This implicitly expresses a form of mapping syntactic constructions over a sequence.
* `e^(i<n)` denotes the same sequence as `e^n`, but implicitly also defines `i^n` to be the sequence of values `0` to `(n - 1)`.

> **Note:** For example, if `x^n` is the sequence `a b c`, then `(f(x) + 1)^n` denotes the sequence `(f(a) + 1) (f(b) + 1) (f(c) + 1)`.
>
> The form `e^(i<n)` additionally gives access to an index variable inside the iteration. For example, `(f(x) + i)^(i<n)` denotes the sequence `(f(a) + 0) (f(b) + 1) (f(c) + 2)`.

Productions of the following form are interpreted as *records* that map a fixed set of fields `field_i` to "values" `A_i`, respectively:

```text
r ::= { field_1 A_1 ,  field_2 A_2 ,  ... }
```

The following notation is adopted for manipulating such records:

* Where the type of a record is clear from context, empty fields with value `ε` are often omitted.
* `r.field` denotes the contents of the `field` component of `r`.
* `r[.field = A]` denotes the same record as `r`, except that the value of the `field` component is replaced with `A`.
* `r[.field =⊕ A*]` denotes the same record as `r`, except that `A*` is appended to the sequence value of the `field` component, that is, it is short for `r[.field = r.field ⊕ A*]`.
* `r1 ⊕ r2` denotes the composition of two identically shaped records by concatenating each field of sequences point-wise:

  ```text
  { field_1 A_1* , field_2 A_2* , ... } ⊕ { field_1 B_1* , field_2 B_2* , ... }
    = { field_1 (A_1* ⊕ B_1*) , field_2 (A_2* ⊕ B_2*) , ... }
  ```
* `++ r*` denotes the composition of a sequence of records, respectively; if the sequence is empty, then all fields of the resulting record are empty.

The update notation for sequences and records generalizes recursively to nested components accessed by "paths" `pth ::= ([ i ] | .field)^+`:

* `s[[ i ] pth = A]` is short for `s[[i] = s[i][pth = A]]`,
* `r[.field pth = A]` is short for `r[.field = r.field[pth = A]]`.

### Lists

*Lists* are bounded sequences of the form `A^n` (or `A*`), where the `A` can either be values or complex constructions. A list can have at most `2^32 - 1` elements.

```text
list(X) ::= X*   (if |X*| < 2^32)
```
