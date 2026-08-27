.. index:: ! instruction, code, stack machine, operand, operand stack
   pair: abstract syntax; instruction
.. _syntax-instr:

Instructions
------------

WebAssembly code consists of sequences of *instructions*.
Its computational model is based on a *stack machine* in that instructions manipulate values on an implicit *operand stack*,
consuming (popping) argument values and producing or returning (pushing) result values.

In addition to dynamic operands from the stack, some instructions also have static *immediate* arguments,
typically :ref:`indices <syntax-index>` or type annotations,
which are part of the instruction itself.

Some instructions are :ref:`structured <syntax-instr-control>` in that they contain nested sequences of instructions.

The following sections group instructions into a number of different categories.

The syntax of instruction is further :ref:`extended <syntax-instr-admin>` with additional forms for the purpose of specifying :ref:`execution <exec>`.


.. index:: ! parametric instruction, value type
   pair: abstract syntax; instruction
.. _syntax-instr-parametric:

Parametric Instructions
~~~~~~~~~~~~~~~~~~~~~~~

Instructions in this group can operate on operands of any :ref:`value type <syntax-valtype>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\instr} & ::= & \NOP \\
   & & | & \UNREACHABLE \\
   & & | & \DROP \\
   & & | & \SELECT~{({{\valtype}^\ast})^?} \\
   & & | & \dots \\
   \end{array}

The :math:`\mathsf{nop}` instruction does nothing.

The :math:`\mathsf{unreachable}` instruction causes an unconditional :ref:`trap <trap>`.

The :math:`\mathsf{drop}` instruction simply throws away a single operand.

The :math:`\mathsf{select}` instruction selects one of its first two operands based on whether its third operand is zero or not.
It may include a :ref:`value type <syntax-valtype>` determining the type of these operands.
If missing, the operands must be of :ref:`numeric <syntax-numtype>` or :ref:`vector <syntax-vectype>` type.

.. note::
   In future versions of WebAssembly, the type annotation on :math:`\mathsf{select}` may allow for more than a single value being selected at the same time.


.. index:: ! control instruction, ! structured control, ! exception, ! label, ! block, ! block type, ! branch, ! unwinding, stack type, label index, function index, type index, list, trap, function, table, tag, function type, value type, tag type, try block, type index
   pair: abstract syntax; instruction
   pair: abstract syntax; block type
   pair: block; type
.. _syntax-nop:
.. _syntax-unreachable:
.. _syntax-block:
.. _syntax-loop:
.. _syntax-if:
.. _syntax-br:
.. _syntax-br_if:
.. _syntax-br_table:
.. _syntax-br_on_null:
.. _syntax-br_on_non_null:
.. _syntax-br_on_cast:
.. _syntax-br_on_cast_fail:
.. _syntax-call:
.. _syntax-call_ref:
.. _syntax-call_indirect:
.. _syntax-return:
.. _syntax-return_call:
.. _syntax-return_call_ref:
.. _syntax-return_call_indirect:
.. _syntax-throw:
.. _syntax-throw_ref:
.. _syntax-try_table:
.. _syntax-catch:
.. _syntax-instrs:
.. _syntax-instr-control:
.. _exception:

Control Instructions
~~~~~~~~~~~~~~~~~~~~

Instructions in this group affect the flow of control.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\instr} & ::= & \dots \\
   & & | & \BLOCK~{\blocktype}~{{\instr}^\ast} \\
   & & | & \LOOP~{\blocktype}~{{\instr}^\ast} \\
   & & | & \IF~{\blocktype}~{{\instr}^\ast}~\ELSE~{{\instr}^\ast} \\
   & & | & \BR~{\labelidx} \\
   & & | & \BRIF~{\labelidx} \\
   & & | & \BRTABLE~{{\labelidx}^\ast}~{\labelidx} \\
   & & | & \BRONNULL~{\labelidx} \\
   & & | & \BRONNONNULL~{\labelidx} \\
   & & | & \BRONCAST~{\labelidx}~{\reftype}~{\reftype} \\
   & & | & \BRONCASTFAIL~{\labelidx}~{\reftype}~{\reftype} \\
   & & | & \CALL~{\funcidx} \\
   & & | & \CALLREF~{\typeuse} \\
   & & | & \CALLINDIRECT~{\tableidx}~{\typeuse} \\
   & & | & \RETURN \\
   & & | & \RETURNCALL~{\funcidx} \\
   & & | & \RETURNCALLREF~{\typeuse} \\
   & & | & \RETURNCALLINDIRECT~{\tableidx}~{\typeuse} \\
   & & | & \THROW~{\tagidx} \\
   & & | & \THROWREF \\
   & & | & \TRYTABLE~{\blocktype}~{\list}({\catch})~{{\instr}^\ast} \\
   & & | & \dots \\[0.8ex]
   & {\catch} & ::= & \CATCH~{\tagidx}~{\labelidx} \\
   & & | & \CATCHREF~{\tagidx}~{\labelidx} \\
   & & | & \CATCHALL~{\labelidx} \\
   & & | & \CATCHALLREF~{\labelidx} \\
   \end{array}

The :math:`\mathsf{block}`, :math:`\mathsf{loop}`, :math:`\mathsf{if}` and :math:`\mathsf{try\_table}` instructions are *structured* instructions.
They bracket nested sequences of instructions, called *blocks*.
As the grammar prescribes, they must be well-nested.

A structured instruction can consume *input* and produce *output* on the operand stack according to its annotated :ref:`block type <syntax-blocktype>`.

Each structured control instruction introduces an implicit *label*.
Labels are targets for branch instructions that reference them with :ref:`label indices <syntax-labelidx>`.
Unlike with other :ref:`index spaces <syntax-index>`, indexing of labels is relative by nesting depth,
that is, label :math:`0` refers to the innermost structured control instruction enclosing the referring branch instruction,
while increasing indices refer to those farther out.
Consequently, labels can only be referenced from *within* the associated structured control instruction.
This also implies that branches can only be directed outwards,
"breaking" from the block of the control construct they target.
The exact effect depends on that control construct.
In case of :math:`\mathsf{block}` or :math:`\mathsf{if}` it is a *forward jump*,
resuming execution after the end of the block.
In case of :math:`\mathsf{loop}` it is a *backward jump* to the beginning of the loop.

.. note::
   This enforces *structured control flow*.
   Intuitively, a branch targeting a :math:`\mathsf{block}` or :math:`\mathsf{if}` behaves like a :math:`\K{break}` statement in most C-like languages,
   while a branch targeting a :math:`\mathsf{loop}` behaves like a :math:`\K{continue}` statement.

Branch instructions come in several flavors:
:math:`\mathsf{br}` performs an unconditional branch,
:math:`\mathsf{br\_if}` performs a conditional branch,
and :math:`\mathsf{br\_table}` performs an indirect branch through an operand indexing into the label list that is an immediate to the instruction, or to a default target if the operand is out of bounds.
The :math:`\mathsf{br\_on\_null}` and :math:`\mathsf{br\_on\_non\_null}` instructions check whether a reference operand is :ref:`null <syntax-nullref>` and branch if that is the case or not the case, respectively.
Similarly, :math:`\mathsf{br\_on\_cast}` and :math:`\mathsf{br\_on\_cast\_fail}` attempt a downcast on a reference operand and branch if that succeeds, or fails, respectively.

The :math:`\mathsf{return}` instruction is a shortcut for an unconditional branch to the outermost block, which implicitly is the body of the current function.
Taking a branch *unwinds* the operand stack up to the height where the targeted structured control instruction was entered.
However, branches may additionally consume operands themselves, which they push back on the operand stack after unwinding.
Forward branches require operands according to the output of the targeted block's type, i.e., represent the values produced by the terminated block.
Backward branches require operands according to the input of the targeted block's type, i.e., represent the values consumed by the restarted block.

The :math:`\mathsf{call}` instruction invokes another :ref:`function <syntax-func>`, consuming the necessary arguments from the stack and returning the result values of the call.
The :math:`\mathsf{call\_ref}` instruction invokes a function indirectly through a :ref:`function reference <syntax-reftype>` operand.
The :math:`\mathsf{call\_indirect}` instruction calls a function indirectly through an operand indexing into a :ref:`table <syntax-table>` that is denoted by a :ref:`table index <syntax-tableidx>` and must contain :ref:`function references <syntax-reftype>`.
Since it may contain functions of heterogeneous type,
the callee is dynamically checked against the :ref:`function type <syntax-functype>` indexed by the instruction's second immediate, and the call is aborted with a :ref:`trap <trap>` if it does not match.

The :math:`\mathsf{return\_call}`, :math:`\mathsf{return\_call\_ref}`, and :math:`\mathsf{return\_call\_indirect}` instructions are *tail-call* variants of the previous ones.
That is, they first return from the current function before actually performing the respective call.
It is guaranteed that no sequence of nested calls using only these instructions can cause resource exhaustion due to hitting an :ref:`implementation's limit <impl-exec>` on the number of active calls.

The instructions :math:`\mathsf{throw}`, :math:`\mathsf{throw\_ref}`, and :math:`\mathsf{try\_table}` are concerned with *exceptions*.
The :math:`\mathsf{throw}` and :math:`\mathsf{throw\_ref}` instructions raise and reraise an exception, respectively, and transfers control to the innermost enclosing exception handler that has a matching catch clause.
The :math:`\mathsf{try\_table}` instruction installs an exception *handler* that handles exceptions as specified by its catch clauses.


.. index:: ! variable instruction, local, global, local index, global index
   pair: abstract syntax; instruction
.. _syntax-instr-variable:

Variable Instructions
~~~~~~~~~~~~~~~~~~~~~

Variable instructions are concerned with access to :ref:`local <syntax-local>` or :ref:`global <syntax-global>` variables.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\instr} & ::= & \dots \\
   & & | & \LOCALGET~{\localidx} \\
   & & | & \LOCALSET~{\localidx} \\
   & & | & \LOCALTEE~{\localidx} \\
   & & | & \GLOBALGET~{\globalidx} \\
   & & | & \GLOBALSET~{\globalidx} \\
   & & | & \dots \\
   \end{array}

These instructions get or set the values of respective variables.
The :math:`\mathsf{local{.}tee}` instruction is like :math:`\mathsf{local{.}set}` but also returns its argument.


.. index:: ! table instruction, table, table index, trap
   pair: abstract syntax; instruction
.. _syntax-instr-table:
.. _syntax-table.get:
.. _syntax-table.set:
.. _syntax-table.size:
.. _syntax-table.grow:
.. _syntax-table.fill:

Table Instructions
~~~~~~~~~~~~~~~~~~

Instructions in this group are concerned with tables :ref:`table <syntax-table>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\instr} & ::= & \dots \\
   & & | & \TABLEGET~{\tableidx} \\
   & & | & \TABLESET~{\tableidx} \\
   & & | & \TABLESIZE~{\tableidx} \\
   & & | & \TABLEGROW~{\tableidx} \\
   & & | & \TABLEFILL~{\tableidx} \\
   & & | & \TABLECOPY~{\tableidx}~{\tableidx} \\
   & & | & \TABLEINIT~{\tableidx}~{\elemidx} \\
   & & | & \ELEMDROP~{\elemidx} \\
   & & | & \dots \\
   \end{array}

The :math:`\mathsf{table{.}get}` and :math:`\mathsf{table{.}set}` instructions load or store an element in a table, respectively.

The :math:`\mathsf{table{.}size}` instruction returns the current size of a table.
The :math:`\mathsf{table{.}grow}` instruction grows table by a given delta and returns the previous size, or :math:`{-1}` if enough space cannot be allocated.
It also takes an initialization value for the newly allocated entries.

The :math:`\mathsf{table{.}fill}` instruction sets all entries in a range to a given value.
The :math:`\mathsf{table{.}copy}` instruction copies elements from a source table region to a possibly overlapping destination region; the first index denotes the destination.
The :math:`\mathsf{table{.}init}` instruction copies elements from a :ref:`passive element segment <syntax-elem>` into a table.

The :math:`\mathsf{elem{.}drop}` instruction prevents further use of a passive element segment. This instruction is intended to be used as an optimization hint. After an element segment is dropped its elements can no longer be retrieved, so the memory used by this segment may be freed.

.. note::
   An additional instruction that accesses a table is the :ref:`control instruction <syntax-instr-control>` :math:`\mathsf{call\_indirect}`.


.. index:: ! memory instruction, memory, memory index, page size, little endian, trap
   pair: abstract syntax; instruction
.. _syntax-loadn:
.. _syntax-storen:
.. _syntax-memarg:
.. _syntax-loadop:
.. _syntax-storeop:
.. _syntax-vloadop:
.. _syntax-lanewidth:
.. _syntax-instr-memory:

Memory Instructions
~~~~~~~~~~~~~~~~~~~

Instructions in this group are concerned with linear :ref:`memory <syntax-mem>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\memarg} & ::= & \{ \begin{array}[t]{@{}l@{}l@{}}
   \ALIGN~{\u32} ,  \OFFSET~{\u64} \} \\
   \end{array} \\[0.8ex]
   & {{\loadop}}_{{\ntI}{{\ntN}}} & ::= & {{\sz}}{\mathsf{\_}}{{\sx}} & \quad \mbox{if}~ {\sz} < N \\[0.8ex]
   & {{\storeop}}_{{\ntI}{{\ntN}}} & ::= & {\sz} & \quad \mbox{if}~ {\sz} < N \\[0.8ex]
   & {{\vloadop}}_{{\vectype}} & ::= & {{\sz}}{\Xshape}{M}{\mathsf{\_}}{{\sx}} & \quad \mbox{if}~ {\sz} \cdot M = {|{\vectype}|} / 2 \\
   & & | & {{\sz}}{\mathsf{\_}}{\LSPLAT} \\
   & & | & {{\sz}}{\mathsf{\_}}{\LZERO} & \quad \mbox{if}~ {\sz} \geq \mathsf{{\scriptstyle 32}} \\[0.8ex]
   & {\instr} & ::= & \dots \\
   & & | & {{\numtype}{.}\LOAD}{{{{\loadop}}_{{\numtype}}^?}}~{\memidx}~{\memarg} \\
   & & | & {{\numtype}{.}\STORE}{{{{\storeop}}_{{\numtype}}^?}}~{\memidx}~{\memarg} \\
   & & | & {{\vectype}{.}\VLOAD}{{{{\vloadop}}_{{\vectype}}^?}}~{\memidx}~{\memarg} \\
   & & | & {{\vectype}{.}\VLOAD}{{\sz}}{\mathsf{\_}}{\VLANE}~{\memidx}~{\memarg}~{\laneidx} \\
   & & | & {\vectype}{.}\VSTORE~{\memidx}~{\memarg} \\
   & & | & {{\vectype}{.}\VSTORE}{{\sz}}{\mathsf{\_}}{\VLANE}~{\memidx}~{\memarg}~{\laneidx} \\
   & & | & \MEMORYSIZE~{\memidx} \\
   & & | & \MEMORYGROW~{\memidx} \\
   & & | & \MEMORYFILL~{\memidx} \\
   & & | & \MEMORYCOPY~{\memidx}~{\memidx} \\
   & & | & \MEMORYINIT~{\memidx}~{\dataidx} \\
   & & | & \DATADROP~{\dataidx} \\
   & & | & \dots \\
   \end{array}

Memory is accessed with :math:`\mathsf{load}` and :math:`\mathsf{store}` instructions for the different :ref:`number types <syntax-numtype>` and :ref:`vector types <syntax-vectype>`.
They all take a :ref:`memory index <syntax-memidx>` and a *memory argument* :math:`{\memarg}` that contains an address *offset* and the expected *alignment* (expressed as the exponent of a power of 2).

Integer loads and stores can optionally specify a *storage size* :math:`{\sz}` that is smaller than the :ref:`bit width <syntax-numtype>` of the respective value type.
In the case of loads, a sign extension mode :math:`{\sx}` is then required to select appropriate behavior.

Vector loads can specify a shape that is half the :ref:`bit width <syntax-valtype>` of :math:`\mathsf{v{\scriptstyle 128}}`. Each lane is half its usual size, and the sign extension mode :math:`{\sx}` then specifies how the smaller lane is extended to the larger lane.
Alternatively, vector loads can perform a *splat*, such that only a single lane of the specified storage size is loaded, and the result is duplicated to all lanes.

The static address offset is added to the dynamic address operand, yielding a 33-bit or 65-bit *effective address* that is the zero-based index at which the memory is accessed.
All values are read and written in |LittleEndian|_ byte order.
A :ref:`trap <trap>` results if any of the accessed memory bytes lies outside the address range implied by the memory's current size.

The :math:`\mathsf{memory{.}size}` instruction returns the current size of a memory.
The :math:`\mathsf{memory{.}grow}` instruction grows a memory by a given delta and returns the previous size, or :math:`{-1}` if enough memory cannot be allocated.
Both instructions operate in units of :ref:`page size <page-size>`.

The :math:`\mathsf{memory{.}fill}` instruction sets all values in a region of memory to a given byte.
The :math:`\mathsf{memory{.}copy}` instruction copies data from a source memory region to a possibly overlapping destination region in another or the same memory; the first index denotes the destination.
The :math:`\mathsf{memory{.}init}` instruction copies data from a :ref:`passive data segment <syntax-data>` into a memory.

The :math:`\mathsf{data{.}drop}` instruction prevents further use of a passive data segment. This instruction is intended to be used as an optimization hint. After a data segment is dropped its data can no longer be retrieved, so the memory used by this segment may be freed.


.. index:: ! reference instruction, reference, null, cast, heap type, reference type
   pair: abstract syntax; instruction
.. _syntax-ref.null:
.. _syntax-ref.func:
.. _syntax-ref.is_null:
.. _syntax-ref.as_non_null:
.. _syntax-ref.eq:
.. _syntax-ref.test:
.. _syntax-ref.cast:
.. _syntax-instr-ref:

Reference Instructions
~~~~~~~~~~~~~~~~~~~~~~

Instructions in this group are concerned with accessing :ref:`references <syntax-reftype>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\instr} & ::= & \dots \\
   & & | & \REFFUNC~{\funcidx} \\
   & & | & \REFNULL~{\heaptype} \\
   & & | & \REFISNULL \\
   & & | & \REFASNONNULL \\
   & & | & \REFEQ \\
   & & | & \REFTEST~{\reftype} \\
   & & | & \REFCAST~{\reftype} \\
   & & | & \dots \\
   \end{array}

The :math:`\mathsf{ref{.}null}` and :math:`\mathsf{ref{.}func}` instructions produce a :ref:`null <syntax-nullref>` reference or a reference to a given function, respectively.

The instruction :math:`\mathsf{ref{.}is\_null}` checks for null,
while :math:`\mathsf{ref{.}as\_non\_null}` converts a :ref:`nullable <syntax-reftype>` to a non-null one, and :ref:`traps <trap>` if it encounters null.

The :math:`\mathsf{ref{.}eq}` compares two references.

The instructions :math:`\mathsf{ref{.}test}` and :math:`\mathsf{ref{.}cast}` test the :ref:`dynamic type <type-inst>` of a reference operand.
The former merely returns the result of the test,
while the latter performs a downcast and :ref:`traps <trap>` if the operand's type does not match.

.. note::
   The :math:`\mathsf{br\_on\_null}` and :math:`\mathsf{br\_on\_non\_null}` instructions provide versions of :math:`\mathsf{ref{.}as\_non\_null}` that branch depending on the success or failure of a null test instead of trapping.
   Similarly, the :math:`\mathsf{br\_on\_cast}` and :math:`\mathsf{br\_on\_cast\_fail}` instructions provides versions of :math:`\mathsf{ref{.}cast}` that branch depending on the success of the downcast instead of trapping.

   An additional instruction operating on function references is the :ref:`control instruction <syntax-instr-control>` :math:`\mathsf{call\_ref}`.


.. index:: reference instruction, reference, null, heap type, reference type
   pair: abstract syntax; instruction

.. _syntax-struct.new:
.. _syntax-struct.new_default:
.. _syntax-struct.get:
.. _syntax-struct.get_s:
.. _syntax-struct.get_u:
.. _syntax-struct.set:
.. _syntax-array.new:
.. _syntax-array.new_default:
.. _syntax-array.new_fixed:
.. _syntax-array.new_data:
.. _syntax-array.new_elem:
.. _syntax-array.get:
.. _syntax-array.get_s:
.. _syntax-array.get_u:
.. _syntax-array.set:
.. _syntax-array.len:
.. _syntax-array.fill:
.. _syntax-array.copy:
.. _syntax-array.init_data:
.. _syntax-array.init_elem:
.. _syntax-ref.i31:
.. _syntax-i31.get_s:
.. _syntax-i31.get_u:
.. _syntax-any.convert_extern:
.. _syntax-extern.convert_any:
.. _syntax-instr-aggr:
.. _syntax-instr-struct:
.. _syntax-instr-array:
.. _syntax-instr-i31:
.. _syntax-instr-extern:

Aggregate Instructions
~~~~~~~~~~~~~~~~~~~~~~

Instructions in this group are concerned with creating and accessing :ref:`references <syntax-reftype>` to :ref:`aggregate <syntax-aggrtype>` types.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\instr} & ::= & \dots \\
   & & | & \STRUCTNEW~{\typeidx} \\
   & & | & \STRUCTNEWDEFAULT~{\typeidx} \\
   & & | & {\STRUCTGET}{\mathsf{\_}}{{{\sx}^?}}~{\typeidx}~{\fieldidx} \\
   & & | & \STRUCTSET~{\typeidx}~{\fieldidx} \\
   & & | & \ARRAYNEW~{\typeidx} \\
   & & | & \ARRAYNEWDEFAULT~{\typeidx} \\
   & & | & \ARRAYNEWFIXED~{\typeidx}~{\u32} \\
   & & | & \ARRAYNEWDATA~{\typeidx}~{\dataidx} \\
   & & | & \ARRAYNEWELEM~{\typeidx}~{\elemidx} \\
   & & | & {\ARRAYGET}{\mathsf{\_}}{{{\sx}^?}}~{\typeidx} \\
   & & | & \ARRAYSET~{\typeidx} \\
   & & | & \ARRAYLEN \\
   & & | & \ARRAYFILL~{\typeidx} \\
   & & | & \ARRAYCOPY~{\typeidx}~{\typeidx} \\
   & & | & \ARRAYINITDATA~{\typeidx}~{\dataidx} \\
   & & | & \ARRAYINITELEM~{\typeidx}~{\elemidx} \\
   & & | & \REFI31 \\
   & & | & {\I31GET}{\mathsf{\_}}{{\sx}} \\
   & & | & \EXTERNCONVERTANY \\
   & & | & \ANYCONVERTEXTERN \\
   & & | & \dots \\
   \end{array}

The instructions :math:`\mathsf{struct{.}new}` and :math:`\mathsf{struct{.}new\_default}` allocate a new :ref:`structure <syntax-structtype>`, initializing them either with operands or with default values.
The remaining instructions on structs access individual fields,
allowing for different sign extension modes in the case of :ref:`packed <syntax-packtype>` storage types.

Similarly, :ref:`arrays <syntax-arraytype>` can be allocated either with an explicit initialization operand or a default value.
Furthermore, :math:`\mathsf{array{.}new\_fixed}` allocates an array with statically fixed size,
and :math:`\mathsf{array{.}new\_data}` and :math:`\mathsf{array{.}new\_elem}` allocate an array and initialize it from a :ref:`data <syntax-data>` or :ref:`element <syntax-elem>` segment, respectively.
The instructions :math:`\mathsf{array{.}get}`, :math:`\mathsf{array{.}get}~{\sx}`, and :math:`\mathsf{array{.}set}` access individual slots,
again allowing for different sign extension modes in the case of a :ref:`packed <syntax-packtype>` storage type;
:math:`\mathsf{array{.}len}` produces the length of an array;
:math:`\mathsf{array{.}fill}` fills a specified slice of an array with a given value and :math:`\mathsf{array{.}copy}`, :math:`\mathsf{array{.}init\_data}`, and :math:`\mathsf{array{.}init\_elem}` copy elements to a specified slice of an array from a given array, data segment, or element segment, respectively.

The instructions :math:`\mathsf{ref{.}i{\scriptstyle 31}}` and :math:`\mathsf{i{\scriptstyle 31}{.}get}~{\sx}` convert between type :math:`\mathsf{i{\scriptstyle 32}}` and an unboxed :ref:`scalar <syntax-i31>`.

The instructions :math:`\mathsf{any{.}convert\_extern}` and :math:`\mathsf{extern{.}convert\_any}` allow lossless conversion between references represented as type :math:`(\REF~\NULL~\EXTERN)` and as :math:`(\REF~\NULL~\ANY)`.


.. index:: ! numeric instruction, value, value type, integer, floating-point, two's complement
   pair: abstract syntax; instruction
.. _syntax-sx:
.. _syntax-sz:
.. _syntax-num_:
.. _syntax-const:
.. _syntax-unop:
.. _syntax-binop:
.. _syntax-testop:
.. _syntax-relop:
.. _syntax-cvtop:
.. _syntax-instr-numeric:

Numeric Instructions
~~~~~~~~~~~~~~~~~~~~

Numeric instructions provide basic operations over numeric :ref:`values <syntax-value>` of specific :ref:`type <syntax-numtype>`.
These operations closely match respective operations available in hardware.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\sz} & ::= & \mathsf{{\scriptstyle 8}} ~~|~~ \mathsf{{\scriptstyle 16}} ~~|~~ \mathsf{{\scriptstyle 32}} ~~|~~ \mathsf{{\scriptstyle 64}} \\
   & {\sx} & ::= & \U ~~|~~ \S \\[0.8ex]
   & {{\num}}_{{\ntI}{{\ntN}}} & ::= & {{\iNX}}{N} \\
   & {{\num}}_{{\ntF}{{\ntN}}} & ::= & {{\fNX}}{N} \\[0.8ex]
   & {\instr} & ::= & \dots \\
   & & | & {\numtype}{.}\CONST~{{\num}}_{{\numtype}} \\
   & & | & {\numtype} {.} {{\unop}}_{{\numtype}} \\
   & & | & {\numtype} {.} {{\binop}}_{{\numtype}} \\
   & & | & {\numtype} {.} {{\testop}}_{{\numtype}} \\
   & & | & {\numtype} {.} {{\relop}}_{{\numtype}} \\
   & & | & {\numtype}_1 {.} {{{\cvtop}}_{{\numtype}_2, {\numtype}_1}}{\mathsf{\_}}{{\numtype}_2} \\
   & & | & \dots \\[0.8ex]
   & {{\unop}}_{{\ntI}{{\ntN}}} & ::= & \CLZ ~~|~~ \CTZ ~~|~~ \POPCNT ~~|~~ {\EXTEND}{{\sz}}{\mathsf{\_}}{\S} & \quad \mbox{if}~ {\sz} < N \\
   & {{\unop}}_{{\ntF}{{\ntN}}} & ::= & \ABS ~~|~~ \NEG ~~|~~ \SQRT ~~|~~ \CEIL ~~|~~ \FLOOR ~~|~~ \TRUNC ~~|~~ \NEAREST \\[0.8ex]
   & {{\binop}}_{{\ntI}{{\ntN}}} & ::= & \ADD ~~|~~ \SUB ~~|~~ \MUL ~~|~~ {\DIV}{\mathsf{\_}}{{\sx}} ~~|~~ {\REM}{\mathsf{\_}}{{\sx}} \\
   & & | & \AND ~~|~~ \OR ~~|~~ \XOR ~~|~~ \SHL ~~|~~ {\SHR}{\mathsf{\_}}{{\sx}} ~~|~~ \ROTL ~~|~~ \ROTR \\
   & {{\binop}}_{{\ntF}{{\ntN}}} & ::= & \ADD ~~|~~ \SUB ~~|~~ \MUL ~~|~~ \DIV ~~|~~ \FMIN ~~|~~ \FMAX ~~|~~ \COPYSIGN \\[0.8ex]
   & {{\testop}}_{{\ntI}{{\ntN}}} & ::= & \EQZ \\[0.8ex]
   & {{\relop}}_{{\ntI}{{\ntN}}} & ::= & \EQ ~~|~~ \NE ~~|~~ {\LT}{\mathsf{\_}}{{\sx}} ~~|~~ {\GT}{\mathsf{\_}}{{\sx}} ~~|~~ {\LE}{\mathsf{\_}}{{\sx}} ~~|~~ {\GE}{\mathsf{\_}}{{\sx}} \\
   & {{\relop}}_{{\ntF}{{\ntN}}} & ::= & \EQ ~~|~~ \NE ~~|~~ \LT ~~|~~ \GT ~~|~~ \LE ~~|~~ \GE \\[0.8ex]
   & {{\cvtop}}_{{{\ntI}{{\ntN}}}_1, {{\ntI}{{\ntN}}}_2} & ::= & {\EXTEND}{\mathsf{\_}}{{\sx}} & \quad \mbox{if}~ N_1 < N_2 \\
   & & | & \WRAP & \quad \mbox{if}~ N_1 > N_2 \\
   & {{\cvtop}}_{{{\ntI}{{\ntN}}}_1, {{\ntF}{{\ntN}}}_2} & ::= & {\CONVERT}{\mathsf{\_}}{{\sx}} \\
   & & | & \REINTERPRET & \quad \mbox{if}~ N_1 = N_2 \\
   & {{\cvtop}}_{{{\ntF}{{\ntN}}}_1, {{\ntI}{{\ntN}}}_2} & ::= & {\TRUNC}{\mathsf{\_}}{{\sx}} \\
   & & | & {\TRUNCSAT}{\mathsf{\_}}{{\sx}} \\
   & & | & \REINTERPRET & \quad \mbox{if}~ N_1 = N_2 \\
   & {{\cvtop}}_{{{\ntF}{{\ntN}}}_1, {{\ntF}{{\ntN}}}_2} & ::= & \PROMOTE & \quad \mbox{if}~ N_1 < N_2 \\
   & & | & \DEMOTE & \quad \mbox{if}~ N_1 > N_2 \\
   \end{array}

Numeric instructions are divided by :ref:`number type <syntax-numtype>`.
For each type, several subcategories can be distinguished:

* *Constants*: return a static constant.

* *Unary operations*: consume one operand and produce one result of the respective type.

* *Binary operations*: consume two operands and produce one result of the respective type.

* *Tests*: consume one operand of the respective type and produce a Boolean integer result.

* *Comparisons*: consume two operands of the respective type and produce a Boolean integer result.

* *Conversions*: consume a value of one type and produce a result of another
  (the source type of the conversion is the one after the ":math:`\mathsf{\_}`").

Some integer instructions come in two flavors,
where a signedness annotation :math:`{\sx}` distinguishes whether the operands are to be :ref:`interpreted <aux-signed>` as :ref:`unsigned <syntax-uint>` or :ref:`signed <syntax-sint>` integers.
For the other integer instructions, the use of two's complement for the signed interpretation means that they behave the same regardless of signedness.


.. index:: ! vector instruction, numeric vector, number, value, value type, SIMD
   pair: abstract syntax; instruction
.. _syntax-laneidx:
.. _syntax-lanetype:
.. _syntax-dim:
.. _syntax-shape:
.. _syntax-half:
.. _syntax-zero:
.. _syntax-vvunop:
.. _syntax-vvbinop:
.. _syntax-vvternop:
.. _syntax-vvtestop:
.. _syntax-vtestop:
.. _syntax-vrelop:
.. _syntax-vswizzlop:
.. _syntax-vshiftop:
.. _syntax-vunop:
.. _syntax-vbinop:
.. _syntax-vternop:
.. _syntax-vextunop:
.. _syntax-vextbinop:
.. _syntax-vextternop:
.. _syntax-vcvtop:
.. _syntax-instr-vec:
.. _syntax-instr-vec-relaxed:

Vector Instructions
~~~~~~~~~~~~~~~~~~~

Vector instructions (also known as *SIMD* instructions, *single instruction multiple data*) provide basic operations over :ref:`values <syntax-value>` of :ref:`vector type <syntax-vectype>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\lanetype} & ::= & {\numtype} ~~|~~ {\packtype} \\
   & {\dim} & ::= & \mathsf{{\scriptstyle 1}} ~~|~~ \mathsf{{\scriptstyle 2}} ~~|~~ \mathsf{{\scriptstyle 4}} ~~|~~ \mathsf{{\scriptstyle 8}} ~~|~~ \mathsf{{\scriptstyle 16}} \\
   & {\shape} & ::= & {{\lanetype}}{\Xshape}{{\dim}} & \quad \mbox{if}~ {|{\lanetype}|} \cdot {\dim} = 128 \\
   & {\ishape} & ::= & {\shape} & \quad \mbox{if}~ {\shlanetype}({\shape}) = {\ntI}{{\ntN}} \\
   & {\bshape} & ::= & {\shape} & \quad \mbox{if}~ {\shlanetype}({\shape}) = \I8 \\[0.8ex]
   & {\half} & ::= & \LOW ~~|~~ \HIGH \\[0.8ex]
   & {\zero} & ::= & \ZERO \\[0.8ex]
   & {\laneidx} & ::= & {\u8} \\[0.8ex]
   & {\instr} & ::= & \dots \\
   & & | & {\vectype}{.}\CONST~{{\vec}}_{{\vectype}} \\
   & & | & {\vectype} {.} {\vvunop} \\
   & & | & {\vectype} {.} {\vvbinop} \\
   & & | & {\vectype} {.} {\vvternop} \\
   & & | & {\vectype} {.} {\vvtestop} \\
   & & | & {\shape} {.} {{\vunop}}_{{\shape}} \\
   & & | & {\shape} {.} {{\vbinop}}_{{\shape}} \\
   & & | & {\shape} {.} {{\vternop}}_{{\shape}} \\
   & & | & {\shape} {.} {{\vtestop}}_{{\shape}} \\
   & & | & {\shape} {.} {{\vrelop}}_{{\shape}} \\
   & & | & {\ishape} {.} {{\vshiftop}}_{{\ishape}} \\
   & & | & {\ishape}{.}\VBITMASK \\
   & & | & {\bshape} {.} {{\vswizzlop}}_{{\bshape}} \\
   & & | & {\bshape}{.}\VSHUFFLE~{{\laneidx}^\ast} & \quad \mbox{if}~ {|{{\laneidx}^\ast}|} = {\shdim}({\bshape}) \\
   & & | & {\ishape}_1 {.} {{{\vextunop}}_{{\ishape}_2, {\ishape}_1}}{\mathsf{\_}}{{\ishape}_2} \\
   & & | & {\ishape}_1 {.} {{{\vextbinop}}_{{\ishape}_2, {\ishape}_1}}{\mathsf{\_}}{{\ishape}_2} \\
   & & | & {\ishape}_1 {.} {{{\vextternop}}_{{\ishape}_2, {\ishape}_1}}{\mathsf{\_}}{{\ishape}_2} \\
   & & | & {{\ishape}_1{.}\VNARROW}{\mathsf{\_}}{{\ishape}_2}{\mathsf{\_}}{{\sx}} & \quad \mbox{if}~ {|{\shlanetype}({\ishape}_2)|} = 2 \cdot {|{\shlanetype}({\ishape}_1)|} \leq \mathsf{{\scriptstyle 32}} \\
   & & | & {\shape}_1 {.} {{{\vcvtop}}_{{\shape}_2, {\shape}_1}}{\mathsf{\_}}{{\shape}_2} \\
   & & | & {\shape}{.}\VSPLAT \\
   & & | & {{\shape}{.}\VEXTRACTLANE}{\mathsf{\_}}{{{\sx}^?}}~{\laneidx} & \quad \mbox{if}~ {{\sx}^?} = \epsilon \Leftrightarrow {\shlanetype}({\shape}) \in \I32~\I64~\F32~\F64 \\
   & & | & {\shape}{.}\VREPLACELANE~{\laneidx} \\
   & & | & \dots \\
   \end{array}

Vector instructions have a naming convention involving a *shape* prefix that
determines how their operands will be interpreted,
written :math:`{t}{\mathsf{x}}{N}`, and consisting of a *lane type* :math:`t`, a possibly *packed* :ref:`numeric type <syntax-numtype>`, and its *dimension* :math:`N`, which denotes the number of lanes of that type.
Operations are performed point-wise on the values of each lane.

Instructions prefixed with :math:`\mathsf{v{\scriptstyle 128}}` do not involve a specific interpretation, and treat the :math:`\mathsf{v{\scriptstyle 128}}` as either an :math:`{\i128}` value or a vector of :math:`128` individual bits.

.. note::
   For example, the shape :math:`{\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}` interprets the operand
   as four :math:`{\i32}` values, packed into an :math:`{\i128}`.
   The bit width of the lane type :math:`t` times :math:`N` always is :math:`128`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\vvunop} & ::= & \VNOT \\[0.8ex]
   & {\vvbinop} & ::= & \VAND ~~|~~ \VANDNOT ~~|~~ \VOR ~~|~~ \VXOR \\[0.8ex]
   & {\vvternop} & ::= & \VBITSELECT \\[0.8ex]
   & {\vvtestop} & ::= & \VANYTRUE \\[0.8ex]
   & {{\vunop}}_{{{\ntI}{{\ntN}}}{\Xshape}{M}} & ::= & \VABS ~~|~~ \VNEG \\
   & & | & \VPOPCNT & \quad \mbox{if}~ N = \mathsf{{\scriptstyle 8}} \\
   & {{\vunop}}_{{{\ntF}{{\ntN}}}{\Xshape}{M}} & ::= & \VABS ~~|~~ \VNEG ~~|~~ \VSQRT \\
   & & | & \VCEIL ~~|~~ \VFLOOR ~~|~~ \VTRUNC ~~|~~ \VNEAREST \\[0.8ex]
   & {{\vbinop}}_{{{\ntI}{{\ntN}}}{\Xshape}{M}} & ::= & \VADD \\
   & & | & \VSUB \\
   & & | & {\VADDSAT}{\mathsf{\_}}{{\sx}} & \quad \mbox{if}~ N \leq \mathsf{{\scriptstyle 16}} \\
   & & | & {\VSUBSAT}{\mathsf{\_}}{{\sx}} & \quad \mbox{if}~ N \leq \mathsf{{\scriptstyle 16}} \\
   & & | & \VMUL & \quad \mbox{if}~ N \geq \mathsf{{\scriptstyle 16}} \\
   & & | & {\VAVGR}{\mathsf{\_}}{\VU} & \quad \mbox{if}~ N \leq \mathsf{{\scriptstyle 16}} \\
   & & | & {\VQ15MULRSAT}{\mathsf{\_}}{\VS} & \quad \mbox{if}~ N = \mathsf{{\scriptstyle 16}} \\
   & & | & {\VRELAXEDQ15MULR}{\mathsf{\_}}{\VS} & \quad \mbox{if}~ N = \mathsf{{\scriptstyle 16}} \\
   & & | & {\VMIN}{\mathsf{\_}}{{\sx}} & \quad \mbox{if}~ N \leq \mathsf{{\scriptstyle 32}} \\
   & & | & {\VMAX}{\mathsf{\_}}{{\sx}} & \quad \mbox{if}~ N \leq \mathsf{{\scriptstyle 32}} \\
   & {{\vbinop}}_{{{\ntF}{{\ntN}}}{\Xshape}{M}} & ::= & \VADD ~~|~~ \VSUB ~~|~~ \VMUL ~~|~~ \VDIV \\
   & & | & \VMIN ~~|~~ \VMAX ~~|~~ \VPMIN ~~|~~ \VPMAX \\
   & & | & \VRELAXEDMIN ~~|~~ \VRELAXEDMAX \\[0.8ex]
   & {{\vternop}}_{{{\ntI}{{\ntN}}}{\Xshape}{M}} & ::= & \VRELAXEDLANESELECT \\
   & {{\vternop}}_{{{\ntF}{{\ntN}}}{\Xshape}{M}} & ::= & \VRELAXEDMADD ~~|~~ \VRELAXEDNMADD \\[0.8ex]
   & {{\vtestop}}_{{{\ntI}{{\ntN}}}{\Xshape}{M}} & ::= & \VALLTRUE \\[0.8ex]
   & {{\vrelop}}_{{{\ntI}{{\ntN}}}{\Xshape}{M}} & ::= & \VEQ ~~|~~ \VNE \\
   & & | & {\VLT}{\mathsf{\_}}{{\sx}} & \quad \mbox{if}~ N \neq \mathsf{{\scriptstyle 64}} \lor {\sx} = \S \\
   & & | & {\VGT}{\mathsf{\_}}{{\sx}} & \quad \mbox{if}~ N \neq \mathsf{{\scriptstyle 64}} \lor {\sx} = \S \\
   & & | & {\VLE}{\mathsf{\_}}{{\sx}} & \quad \mbox{if}~ N \neq \mathsf{{\scriptstyle 64}} \lor {\sx} = \S \\
   & & | & {\VGE}{\mathsf{\_}}{{\sx}} & \quad \mbox{if}~ N \neq \mathsf{{\scriptstyle 64}} \lor {\sx} = \S \\
   & {{\vrelop}}_{{{\ntF}{{\ntN}}}{\Xshape}{M}} & ::= & \VEQ ~~|~~ \VNE ~~|~~ \VLT ~~|~~ \VGT ~~|~~ \VLE ~~|~~ \VGE \\[0.8ex]
   & {{\vswizzlop}}_{{\I8}{\Xshape}{M}} & ::= & \VSWIZZLE ~~|~~ \VRELAXEDSWIZZLE \\[0.8ex]
   & {{\vshiftop}}_{{{\ntI}{{\ntN}}}{\Xshape}{M}} & ::= & \VSHL ~~|~~ {\VSHR}{\mathsf{\_}}{{\sx}} \\[0.8ex]
   & {{\vextunop}}_{{{{\ntI}{{\ntN}}}_1}{\Xshape}{M_1}, {{{\ntI}{{\ntN}}}_2}{\Xshape}{M_2}} & ::= & {\VEXTADDPAIRWISE}{\mathsf{\_}}{{\sx}} & \quad \mbox{if}~ 16 \leq 2 \cdot N_1 = N_2 \leq \mathsf{{\scriptstyle 32}} \\[0.8ex]
   & {{\vextbinop}}_{{{{\ntI}{{\ntN}}}_1}{\Xshape}{M_1}, {{{\ntI}{{\ntN}}}_2}{\Xshape}{M_2}} & ::= & {\VEXTMUL}{\mathsf{\_}}{{\half}}{\mathsf{\_}}{{\sx}} & \quad \mbox{if}~ 2 \cdot N_1 = N_2 \geq \mathsf{{\scriptstyle 16}} \\
   & & | & {\VDOT}{\mathsf{\_}}{\VS} & \quad \mbox{if}~ 2 \cdot N_1 = N_2 = \mathsf{{\scriptstyle 32}} \\
   & & | & {\VRELAXEDDOT}{\mathsf{\_}}{\VS} & \quad \mbox{if}~ 2 \cdot N_1 = N_2 = \mathsf{{\scriptstyle 16}} \\[0.8ex]
   & {{\vextternop}}_{{{{\ntI}{{\ntN}}}_1}{\Xshape}{M_1}, {{{\ntI}{{\ntN}}}_2}{\Xshape}{M_2}} & ::= & {\VRELAXEDDOTADD}{\mathsf{\_}}{\VS} & \quad \mbox{if}~ 4 \cdot N_1 = N_2 = \mathsf{{\scriptstyle 32}} \\[0.8ex]
   & {{\vcvtop}}_{{{{\ntI}{{\ntN}}}_1}{\Xshape}{M_1}, {{{\ntI}{{\ntN}}}_2}{\Xshape}{M_2}} & ::= & {\VEXTEND}{\mathsf{\_}}{{\half}}{\mathsf{\_}}{{\sx}} & \quad \mbox{if}~ N_2 = 2 \cdot N_1 \\
   & {{\vcvtop}}_{{{{\ntI}{{\ntN}}}_1}{\Xshape}{M_1}, {{{\ntF}{{\ntN}}}_2}{\Xshape}{M_2}} & ::= & {\VCONVERT}{\mathsf{\_}}{{{\half}^?}}{\mathsf{\_}}{{\sx}} & \quad \mbox{if}~ N_2 = N_1 = \mathsf{{\scriptstyle 32}} \land {{\half}^?} = \epsilon \lor N_2 = 2 \cdot N_1 \land {{\half}^?} = \LOW \\
   & {{\vcvtop}}_{{{{\ntF}{{\ntN}}}_1}{\Xshape}{M_1}, {{{\ntI}{{\ntN}}}_2}{\Xshape}{M_2}} & ::= & {\VTRUNCSAT}{\mathsf{\_}}{{\sx}}{\mathsf{\_}}{{{\zero}^?}} & \quad \mbox{if}~ N_1 = N_2 = \mathsf{{\scriptstyle 32}} \land {{\zero}^?} = \epsilon \lor N_1 = 2 \cdot N_2 \land {{\zero}^?} = \ZERO \\
   & & | & {\VRELAXEDTRUNC}{\mathsf{\_}}{{\sx}}{\mathsf{\_}}{{{\zero}^?}} & \quad \mbox{if}~ N_1 = N_2 = \mathsf{{\scriptstyle 32}} \land {{\zero}^?} = \epsilon \lor N_1 = 2 \cdot N_2 \land {{\zero}^?} = \ZERO \\
   & {{\vcvtop}}_{{{{\ntF}{{\ntN}}}_1}{\Xshape}{M_1}, {{{\ntF}{{\ntN}}}_2}{\Xshape}{M_2}} & ::= & {\VDEMOTE}{\mathsf{\_}}{\VZERO} & \quad \mbox{if}~ N_1 = 2 \cdot N_2 \\
   & & | & {\VPROMOTE}{\mathsf{\_}}{\VLOW} & \quad \mbox{if}~ 2 \cdot N_1 = N_2 \\
   \end{array}

Vector instructions can be grouped into several subcategories:

* *Constants*: return a static constant.

* *Unary Operations*: consume one :math:`\mathsf{v{\scriptstyle 128}}` operand and produce one :math:`\mathsf{v{\scriptstyle 128}}` result.

* *Binary Operations*: consume two :math:`\mathsf{v{\scriptstyle 128}}` operands and produce one :math:`\mathsf{v{\scriptstyle 128}}` result.

* *Ternary Operations*: consume three :math:`\mathsf{v{\scriptstyle 128}}` operands and produce one :math:`\mathsf{v{\scriptstyle 128}}` result.

* *Tests*: consume one :math:`\mathsf{v{\scriptstyle 128}}` operand and produce a Boolean integer result.

* *Shifts*: consume a :math:`\mathsf{v{\scriptstyle 128}}` operand and an :math:`\mathsf{i{\scriptstyle 32}}` operand, producing one :math:`\mathsf{v{\scriptstyle 128}}` result.

* *Splats*: consume a value of numeric type and produce a :math:`\mathsf{v{\scriptstyle 128}}` result of a specified shape.

* *Extract lanes*: consume a :math:`\mathsf{v{\scriptstyle 128}}` operand and return the numeric value in a given lane.

* *Replace lanes*: consume a :math:`\mathsf{v{\scriptstyle 128}}` operand and a numeric value for a given lane, and produce a :math:`\mathsf{v{\scriptstyle 128}}` result.

Some vector instructions have a signedness annotation :math:`{\sx}` which distinguishes whether the elements in the operands are to be :ref:`interpreted <aux-signed>` as :ref:`unsigned <syntax-uint>` or :ref:`signed <syntax-sint>` integers.
For the other vector instructions, the use of two's complement for the signed interpretation means that they behave the same regardless of signedness.


.. _aux-lanetype:
.. _aux-dim:
.. _aux-zeroop:
.. _aux-halfop:

Conventions
...........

* The function :math:`{\shlanetype}({\shape})` extracts the lane type of a shape.  

* The function :math:`{\shdim}({\shape})` extracts the dimension of a shape.  

* The function :math:`{\zeroop}({\vcvtop})` extracts the :math:`\mathsf{zero}` flag from a vector conversion operator, or returns :math:`\epsilon` if it does not contain any.  

* The function :math:`{\halfop}({\vcvtop})` extracts the :math:`{\half}` flag from a vector conversion operator, or returns :math:`\epsilon` if it does not contain any.  


.. index:: ! expression, constant, global, offset, element, data, instruction
   pair: abstract syntax; expression
   single: expression; constant
.. _syntax-expr:

Expressions
~~~~~~~~~~~

:ref:`Function <syntax-func>` bodies, initialization values for :ref:`globals <syntax-global>`, elements and offsets of :ref:`element <syntax-elem>` segments, and offsets of :ref:`data <syntax-data>` segments are given as expressions, which are sequences of :ref:`instructions <syntax-instr>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\expr} & ::= & {{\instr}^\ast} \\
   \end{array}

In some places, validation :ref:`restricts <valid-constant>` expressions to be *constant*, which limits the set of allowable instructions.
