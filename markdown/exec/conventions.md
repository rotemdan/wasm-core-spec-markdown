## Conventions

WebAssembly code is *executed* when instantiating a module or invoking an exported function on the resulting module instance.

Execution behavior is defined in terms of an *abstract machine* that models the *program state*.

It includes a *stack*, which records operand values and control constructs, and an abstract *store* containing global state.

For each instruction, there is a rule that specifies the effect of its execution on the program state.

Furthermore, there are rules describing the instantiation of a module.

As with validation, all rules are given in two *equivalent* forms:

1. In *prose*, describing the execution in intuitive form.
2. In *formal notation*, describing the rule in mathematical form. [^cite-pldi2017]

> **Note:** As with validation, the prose and formal rules are equivalent,
> so that understanding of the formal notation is *not* required to read this specification.
>
> The formalism offers a more concise description in notation that is used widely in programming languages semantics and is readily amenable to mathematical proof.

### Prose Notation

Execution is specified by stylised, step-wise rules for each instruction of the abstract syntax.
The following conventions are adopted in stating these rules.

* The execution rules implicitly assume a given store `s`.
* The execution rules also assume the presence of an implicit stack that is modified by *pushing* or *popping* values, labels, and frames.
* Certain rules require the stack to contain at least one frame. The most recent frame is referred to as the *current* frame.
* Both the store and the current frame are mutated by *replacing* some of their components. Such replacement is assumed to apply globally.
* The execution of an instruction may *trap*, in which case the entire computation is aborted and no further modifications to the store are performed by it. (Other computations can still be initiated afterwards.)
* The execution of an instruction may also end in a *jump* to a designated target, which defines the next instruction to execute.
* Execution can *enter* and *exit* instruction sequences that form blocks.
* Instruction sequences are implicitly executed in order, unless a trap, jump, or exception occurs.
* In various places the rules contain *assertions* expressing crucial invariants about the program state.

### Formal Notation

> **Note:** This section gives a brief explanation of the notation for specifying execution formally.
> For the interested reader, a more thorough introduction can be found in respective text books. [^cite-tapl]

The formal execution rules use a standard approach for specifying operational semantics, rendering them into *reduction rules*.

Every rule has the following general form:

```text
configuration -> configuration
```

A *configuration* is a syntactic description of a program state.

Each rule specifies one *step* of execution.

As long as there is at most one reduction rule applicable to a given configuration, reduction -- and thereby execution -- is *deterministic*.

WebAssembly has only very few exceptions to this, which are noted explicitly in this specification.

For WebAssembly, a configuration typically is a tuple `(s ; f ; instr*)` consisting of the current store `s`, the call frame `f` of the current function, and the sequence of instructions that is to be executed. (A more precise definition is given later.)

To avoid unnecessary clutter, the store `s` and the frame `f` are often combined into a *state* `z`, which is a pair `(s ; f)`. Moreover, `z` is omitted from reduction rules that do not touch them.

There is no separate representation of the stack.

Instead, it is conveniently represented as part of the configuration's instruction sequence.

In particular, values are defined to coincide with `const` and `ref` instructions, and a sequence of such instructions can be interpreted as an operand "stack" that grows to the right.

> **Note:** For example, the reduction rule for the `i32.add` instruction can be given as follows:
>
> ```text
> i32.const n1 i32.const n2 i32.add -> i32.const (n1 + n2 mod 2^32)
> ```
>
> Per this rule, two `const` instructions and the `add` instruction itself are removed from the instruction stream and replaced with one new `const` instruction.
>
> This can be interpreted as popping two values off the stack and pushing the result.
>
> When no result is produced, an instruction reduces to the empty sequence:
>
> ```text
> nop -> ε
> ```

Labels and frames are similarly defined to be part of an instruction sequence.

The order of reduction is determined by the details of the reduction rules. Usually, the left-most instruction that is not a constant will be the subject of the next reduction *step*.

Reduction *terminates* when no more reduction rules are applicable. Soundness of the WebAssembly type system guarantees that this is only the case when the original instruction sequence has either been reduced to a sequence of value instructions, which can be interpreted as the values of the resulting operand stack, or if an exception or trap occurred.

> **Note:** For example, the following instruction sequence,
>
> ```text
> (f64.const q1) (f64.const q2) (f64.neg) (f64.const q3) (f64.add) (f64.mul)
> ```
>
> terminates after three steps:
>
> ```text
> -> (f64.const q1) (f64.const q4) (f64.const q3) (f64.add) (f64.mul)
> -> (f64.const q5) (f64.mul)
> -> (f64.const q6)
> ```
>
> where `q4 = -q2` and `q5 = -q2 + q3` and `q6 = q1 * (-q2 + q3)`.

[^cite-pldi2017]: The semantics is derived from the following article:
   Andreas Haas, Andreas Rossberg, Derek Schuff, Ben Titzer, Dan Gohman, Luke Wagner, Alon Zakai, JF Bastien, Michael Holman. PLDI2017. Proceedings of the 38th ACM SIGPLAN Conference on Programming Language Design and Implementation (PLDI 2017). ACM 2017.

[^cite-tapl]: For example: Benjamin Pierce. TAPL. The MIT Press 2002
