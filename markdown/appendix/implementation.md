## Implementation Limitations

Implementations typically impose additional restrictions on a number of aspects of a WebAssembly module or execution.

These may stem from:

* physical resource limits,
* constraints imposed by the embedder or its environment,
* limitations of selected implementation strategies.

This section lists allowed limitations.

Where restrictions take the form of numeric limits, no minimum requirements are given, nor are the limits assumed to be concrete, fixed numbers.

However, it is expected that all implementations have "reasonably" large limits to enable common applications.

> **Note:** A conforming implementation is not allowed to leave out individual *features*. However, designated subsets of WebAssembly may be specified in the future.

### Syntactic Limits

#### Structure

An implementation may impose restrictions on the following dimensions of a module:

* the number of [types](syntax-type) in a [module](syntax-module)
* the number of [functions](syntax-func) in a [module](syntax-module), including imports
* the number of [tables](syntax-table) in a [module](syntax-module), including imports
* the number of [memories](syntax-mem) in a [module](syntax-module), including imports
* the number of [globals](syntax-global) in a [module](syntax-module), including imports
* the number of [tags](syntax-tag) in a [module](syntax-module), including imports
* the number of [element segments](syntax-elem) in a [module](syntax-module)
* the number of [data segments](syntax-data) in a [module](syntax-module)
* the number of [imports](syntax-import) to a [module](syntax-module)
* the number of [exports](syntax-export) from a [module](syntax-module)
* the number of [sub types](syntax-subtype) in a [recursive type](syntax-rectype)
* the subtyping depth of a [sub type](syntax-subtype)
* the number of fields in a [structure type](syntax-structtype)
* the number of parameters in a [function type](syntax-functype)
* the number of results in a [function type](syntax-functype)
* the number of parameters in a [block type](syntax-blocktype)
* the number of results in a [block type](syntax-blocktype)
* the number of [locals](syntax-local) in a [function](syntax-func)
* the number of [instructions](syntax-instr) in a [function](syntax-func) body
* the number of [instructions](syntax-instr) in a [structured control instruction](syntax-instr-control)
* the number of [structured control instructions](syntax-instr-control) in a [function](syntax-func)
* the nesting depth of [structured control instructions](syntax-instr-control)
* the number of [label indices](syntax-labelidx) in a BRTABLE instruction
* the number of instructions in a [constant](valid-constant) [expression](syntax-expr)
* the number of instructions between a [branch instruction](syntax-br) and its target
* the nesting depth of blocks between a [branch instruction](syntax-br) and its target
* the length of the array in a ARRAYNEWFIXED instruction
* the length of an [element segment](syntax-elem)
* the length of a [data segment](syntax-data)
* the length of a [name](syntax-name)
* the range of [characters](syntax-char) in a [name](syntax-name)

If the limits of an implementation are exceeded for a given module, then the implementation may reject the [validation](valid), compilation, or [instantiation](exec-instantiation) of that module with an implementation-specific error.

> **Note:** The last item allows [embedders](embedder) that operate in limited environments without support for Unicode to limit the names of [imports](syntax-import) and [exports](syntax-export) to common subsets like ASCII.

#### Binary Format

For a module given in [binary format](binary), additional limitations may be imposed on the following dimensions:

* the size of a [module](binary-module)
* the size of any [section](binary-section)
* the size of an individual [function](syntax-func)'s [code](binary-code)
* the size of a [structured control instruction](syntax-instr-control)
* the size of an individual [constant](valid-constant) [expression](syntax-expr)'s instruction sequence
* the number of [sections](binary-section)

#### Text Format

For a module given in [text format](text), additional limitations may be imposed on the following dimensions:

* the size of the [source text](source)
* the size of any syntactic element
* the size of an individual [token](text-token)
* the nesting depth of [folded instructions](text-foldedinstr)
* the length of symbolic [identifiers](text-id)
* the range of literal [characters](text-char) allowed in the [source text](source)

#### Validation

An implementation may defer [validation](valid) of individual [functions](syntax-func) until they are first [invoked](exec-invoke).

If a function turns out to be invalid, then the invocation, and every consecutive call to the same function, results in a [trap](trap).

> **Note:** This is to allow implementations to use interpretation or just-in-time compilation for functions. The function must still be fully validated before execution of its body begins.

### Execution

Restrictions on the following dimensions may be imposed during [execution](exec) of a WebAssembly program:

* the number of allocated [module instances](syntax-moduleinst)
* the number of allocated [function instances](syntax-funcinst)
* the number of allocated [table instances](syntax-tableinst)
* the number of allocated [memory instances](syntax-meminst)
* the number of allocated [global instances](syntax-globalinst)
* the number of allocated [tag instances](syntax-taginst)
* the number of allocated [structure instances](syntax-structinst)
* the number of allocated [array instances](syntax-arrayinst)
* the number of allocated [exception instances](syntax-exninst)
* the size of a [table instance](syntax-tableinst)
* the size of a [memory instance](syntax-meminst)
* the size of an [array instance](syntax-arrayinst)
* the number of [frames](syntax-frame) on the [stack](stack)
* the number of [labels](syntax-label) on the [stack](stack)
* the number of [values](syntax-val) on the [stack](stack)
* the number of [handlers](syntax-handler) on the [stack](stack)
* the number of [labels](syntax-val) dropped from the [stack](stack) by a [branch instruction](syntax-br)
* the number of [values](syntax-val) dropped from the [stack](stack) by a [branch instruction](syntax-br)

If the runtime limits of an implementation are exceeded during execution of a computation, then it may terminate that computation and report an implementation-specific error to the invoking code.

Some of the above limits may already be verified during [validation](valid) or [instantiation](exec-instantiation), in which case an implementation may report exceedance in the same manner as for [syntactic limits](impl-syntax).

> **Note:** Concrete limits are usually not fixed but may be dependent on specifics, interdependent, vary over time, or depend on other implementation- or embedder-specific situations or events.
