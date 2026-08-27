## Conventions

Validation checks that a WebAssembly module is well-formed. Only valid modules can be instantiated.

Validity is defined by a *type system* over the abstract syntax of a module and its contents. For each piece of abstract syntax, there is a typing rule that specifies the constraints that apply to it.

All rules are given in two *equivalent* forms:

1. In *prose*, describing the meaning in intuitive form.
2. In *formal notation*, describing the rule in mathematical form. [^cite-pldi2017]

> **Note:** The prose and formal rules are equivalent, so that understanding of the formal notation is *not* required to read this specification.
>
> The formalism offers a more concise description in notation that is used widely in programming languages semantics and is readily amenable to mathematical proof.

In both cases, the rules are formulated in a *declarative* manner. That is, they only formulate the constraints, they do not define an algorithm. The skeleton of a sound and complete algorithm for type-checking instruction sequences according to this specification is provided in the appendix.

### Types

To define the semantics, the definition of some sorts of types is extended to include additional forms. By virtue of not being representable in either the binary format or the text format, these forms cannot be used in a program; they only occur during validation or execution.

```text
valtype ::= ... | bot

absheaptype ::= ... | bot

typeuse ::= ... | deftype | rec . n
```

The unique value type `bot` is a *bottom type* that matches all value types. Similarly, `bot` is also used as a bottom type of all heap types.

> **Note:** No validation rule uses bottom types explicitly, but various rules can pick any value or heap type, including bottom.
>
> This ensures the existence of principal types, and thus a validation algorithm without back tracking.

A type use can consist directly of a defined type. This occurs as the result of substituting a type index with its definition.

A type use may also be a *recursive type index*. Such an index refers to the `i`-th component of a surrounding recursive type. It occurs as the result of rolling up the definition of a recursive type.

Both extensions affect occurrences of type uses in concrete heap types, in sub types and in instructions.

A type of any form is *closed* when it does not contain a heap type that is a type index or a recursive type index without a surrounding recursive type, i.e., all type indices have been substituted with their defined type and all free recursive type indices have been unrolled.

#### Convention

* The *difference* `rt1 reftypediff rt2` between two reference types is defined as follows:

  ```text
  (ref null? ht1) reftypediff (ref null ht2) = (ref ht1)
  (ref null? ht1) reftypediff (ref ht2) = (ref null? ht1)
  ```

> **Note:** This definition computes an approximation of the reference type that is inhabited by all values from `rt1` except those from `rt2`. Since the type system does not have general union types, the definition only affects the presence of null and cannot express the absence of other values.

### Defined Types

*Defined types* denote the individual types defined in a module.

Each such type is represented as a projection from the recursive type group it originates from, indexed by its position in that group.

```text
deftype ::= rectype . n
```

Defined types do not occur in the binary or text format, but are formed by rolling up the recursive types defined in a module.

> **Note:** It is an invariant of the semantics that all recursive types occurring in defined types are rolled up.

#### Conventions

* `t[ x* assignsubst dt* ]` denotes the parallel *substitution* of type indices `x*` with corresponding defined types `dt*` in type `t`, provided `|x*| = |dt*|`.
* `t[ (rec i)* assignsubst dt* ]` denotes the parallel substitution of recursive type indices `(rec i)*` with defined types `dt*` in type `t`, provided `|(rec i)*| = |dt*|`. This substitution does not proceed under recursive types, since they are considered local *binders* for all recursive type indices.

* `t[ assignsubst dt* ]` is shorthand for the substitution `t[ x* assignsubst dt* ]`, where `x* = 0 ... (|dt*| - 1)`.

> **Note:** All recursive types formed by the semantics are closed with respect to recursive type indices that occur inside them. Hence, substitution of recursive type indices never needs to modify the bodies of recursive types.
>
> In addition, all types used for substitution are closed with respect to recursive type indices, such that name capture of recursive type indices cannot occur.

### Rolling and Unrolling

In order to allow comparing recursive types for equivalence, their representation is changed such that all type indices internal to the same recursive type are replaced by recursive type indices.

> **Note:** This representation is independent of the type index space, so that it is meaningful across module boundaries. Moreover, this representation ensures that types with equivalent recursive structure are also syntactically equal, hence allowing a simple equality check on (closed) types. It gives rise to an *iso-recursive* interpretation of types.

The representation change is performed by two auxiliary operations on the syntax of recursive types:

* *Rolling up* a recursive type substitutes its internal type indices with corresponding recursive type indices.
* *Unrolling* a recursive type substitutes its recursive type indices with the corresponding defined types.

These operations are extended to defined types and defined as follows:

```text
rollrt_x(rectype) = rec ((subtype[ (x + i)^(i<n) assignsubst (rec . i)^(i<n) ])^n)   (if rectype = rec subtype^{n})
unrollrt(rectype) = rec ((subtype[ (rec . i)^(i<n) assignsubst (rectype . i)^(i<n) ])^n)   (if rectype = rec subtype^{n})
rolldt_x^* (rectype) = ((rec subtype^{n}) . i)^(i<n)   (if rollrt_x(rectype) = rec subtype^{n})
unrolldt(rectype . i) = subtype^* [i]   (if unrollrt(rectype) = rec subtype^*)
```

In addition, the following auxiliary relation denotes the *expansion* of a defined type or type use:

```text
deftype ≈ comptype   (if unrolldt(deftype) = sub final? typeuse^* comptype)
deftype ≈_C comptype   (if deftype ≈ comptype)
typeidx ≈_C comptype   (if C.TYPES[typeidx] ≈ comptype)
```

### Instruction Types

*Instruction types* classify the behaviour of instructions or instruction sequences, by describing how they manipulate the operand stack and the initialization status of locals:

```text
instrtype ::= resulttype ->_{localidx^*} resulttype
```

An instruction type `t1* ->_{x*} t2*` describes the required input stack with argument values of types `t1*` that an instruction pops off and the provided output stack with result values of types `t2*` that it pushes back. Moreover, it enumerates the indices `x*` of locals that have been set by the instruction or sequence.

> **Note:** Instruction types are only used for validation, they do not occur in programs.

### Local Types

*Local types* classify locals, by describing their value type as well as their *initialization status*:

```text
localtype ::= init valtype

init ::= set | unset
```

> **Note:** Local types are only used for validation, they do not occur in programs.

### Contexts

Validity of an individual definition is specified relative to a *context*,
which collects relevant information about the surrounding module and the definitions in scope:

* *Types*: the list of types defined in the current module.
* *Functions*: the list of functions declared in the current module, represented by a defined type that expands to their function type.
* *Tables*: the list of tables declared in the current module, represented by their table type.
* *Memories*: the list of memories declared in the current module, represented by their memory type.
* *Globals*: the list of globals declared in the current module, represented by their global type.
* *Tags*: the list of tags declared in the current module, represented by their tag type.
* *Element Segments*: the list of element segments declared in the current module, represented by the elements' reference type.
* *Data Segments*: the list of data segments declared in the current module, each represented by an `OKdata` entry.
* *Locals*: the list of locals declared in the function (including parameters), represented by their local type.
* *Labels*: the stack of labels accessible from the current position, represented by their result type.
* *Return*: the return type of the current function, represented as an optional result type that is absent when no return is allowed, as in free-standing expressions.
* *References*: the list of function indices that occur in the module outside functions and can hence be used to form references inside them.

In other words, a context contains a sequence of suitable types for each index space, describing each defined entry in that space. Locals, labels and return type are only used for validating instructions in function bodies, and are left empty elsewhere. The label stack is the only part of the context that changes as validation of an instruction sequence proceeds.

More concretely, contexts are defined as records `C` with abstract syntax:

```text
context ::=
  { TYPES deftype*
    TAGS tagtype*
    GLOBALS globaltype*
    MEMS memtype*
    TABLES tabletype*
    FUNCS deftype*
    DATAS datatype*
    ELEMS elemtype*
    LOCALS localtype*
    LABELS resulttype*
    RETURN resulttype?
    REFS funcidx*
    ... }
```

> **Note:** The definition of contexts needs to be extended with additional fields for the purpose of proving type soundness.

#### Convention

A type of any shape can be *closed* to bring it into closed form relative to a context it is valid in, by substituting each type index `x` occurring in it with its own corresponding defined type `C.TYPES[x]`, after first closing the types in `C.TYPES` themselves.

```text
clostype_C(t) = t[ assignsubst dt* ]   (if dt* = clostype^*(C.TYPES))
clostype^*(ε) = ε
clostype^*(dt* dt_n) = dt'^* dt_n[ assignsubst dt'^* ]   (if dt'^* = clostype^*(dt*))
```

> **Note:** Free type indices referring to types within the same recursive type are handled separately by rolling up recursive types before closing them.

### Prose Notation

Validation is specified by stylised rules for each relevant part of the abstract syntax. The rules not only state constraints defining when a phrase is valid, they also classify it with a type.
The following conventions are adopted in stating these rules.

* A phrase `A` is said to be "valid with type `T`" if and only if all constraints expressed by the respective rules are met. The form of `T` depends on the syntactic class of `A`.

  > **Note:** For example, if `A` is a function, then `T` is a defined function type; for an `A` that is a global, `T` is a global type; and so on.

* The rules implicitly assume a given context `C`.
* In some places, this context is locally extended to a context `C'` with additional entries.
  The formulation "Under context `C'`, ... *statement* ..." is adopted to express that the following statement must apply under the assumptions embodied in the extended context.

### Formal Notation

> **Note:** This section gives a brief explanation of the notation for specifying typing rules formally. For the interested reader, a more thorough introduction can be found in respective text books. [^cite-tapl]

The proposition that a phrase `A` has a respective type `T` is written `A : T`.

In general, however, typing is dependent on a context `C`.

To express this explicitly, the complete form is a *judgement* `C |- A : T`,

which says that `A : T` holds under the assumptions encoded in `C`.

The formal typing rules use a standard approach for specifying type systems, rendering them into *deduction rules*.

Every rule has the following general form:

```text
premise1
premise2
...
premisen
────────────
conclusion
```

Such a rule is read as a big implication: if all premises hold, then the conclusion holds.

Some rules have no premises; they are *axioms* whose conclusion holds unconditionally.

The conclusion always is a judgment `C |- A : T`, and there usually is one respective rule for each relevant construct `A` of the abstract syntax.

> **Note:** For example, the typing rule for the `i32.add` instruction can be given as an axiom:
>
> ```text
> C |- i32.add : i32 i32 -> i32
> ```
>
> The instruction is always valid with type `i32 i32 -> i32` (saying that it consumes two `i32` values and produces one), independent of any side conditions.
>
> An instruction like `global.get` can be typed as follows:
>
> ```text
> C.GLOBALS[x] = mut t
> ─────────────────────
> C |- global.get x : ε -> t
> ```
>
> Here, the premise enforces that the immediate global index `x` exists in the context. The instruction produces a value of its respective type `t` (and does not consume any values). If `C.GLOBALS[x]` does not exist then the premise does not hold, and the instruction is ill-typed.
>
> Finally, a structured instruction requires a recursive rule, where the premise is itself a typing judgement:
>
> ```text
> C |- blocktype : t1* -> t2*
> { LABELS (t2*) } ⊕ C |- instr* : t1* -> t2*
> ───────────────────────────────────────────
> C |- block blocktype instr* : t1* -> t2*
> ```
>
> A `block` instruction is only valid when the instruction sequence in its body is. Moreover, the result type must match the block's annotation `blocktype`. If so, then the `block` instruction has the same type as the body. Inside the body an additional label of the corresponding result type is available, which is expressed by extending the context `C` with the additional label information for the premise.

[^cite-pldi2017]: The semantics is derived from the following article:
Andreas Haas, Andreas Rossberg, Derek Schuff, Ben Titzer, Dan Gohman, Luke Wagner, Alon Zakai, JF Bastien, Michael Holman. PLDI2017. Proceedings of the 38th ACM SIGPLAN Conference on Programming Language Design and Implementation (PLDI 2017). ACM 2017.

[^cite-tapl]: For example: Benjamin Pierce. TAPL. The MIT Press 2002
