.. index:: instruction
.. _text-instr:
.. _text-instrs:

Instructions
------------

Instructions are syntactically distinguished into *plain* and *structured* instructions.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tinstr}}_{{\I}} & ::= & {\mathit{in}}{:}{{\Tplaininstr}}_{{\I}} & \quad\Rightarrow\quad{} & {\mathit{in}} \\
   & & | & {\mathit{in}}{:}{{\Tblockinstr}}_{{\I}} & \quad\Rightarrow\quad{} & {\mathit{in}} \\
   & {{\Tinstrs}}_{{\I}} & ::= & {{\mathit{in}}^\ast}{:}{{{\Tinstr}}_{{\I}}^\ast} & \quad\Rightarrow\quad{} & {{\mathit{in}}^\ast} \\
   \end{array}

In addition, as a syntactic abbreviation, instructions can be written as S-expressions in :ref:`folded <text-foldedinstr>` form, to group them visually.


.. index:: index, label index
   pair: text format; label index
.. _text-label:

Labels
~~~~~~

:ref:`Structured control instructions <text-instr-control>` can be annotated with a symbolic :ref:`label identifier <text-id>`.
They are the only :ref:`symbolic identifiers <text-index>` that can be bound locally in an instruction sequence.
The following grammar handles the corresponding update to the :ref:`identifier context <text-context>` by :ref:`composing <notation-compose>` the context with an additional label entry.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tlabel}}_{{\I}} & ::= & \epsilon & \quad\Rightarrow\quad{} & (\epsilon, \{ \ILABELS~\epsilon \} \oplus {\I}) \\
   & & | & {\mathit{id}}{:}{\Tid} & \quad\Rightarrow\quad{} & ({\mathit{id}}, \{ \ILABELS~{\mathit{id}} \} \oplus {\I}) & \quad \mbox{if}~ {\mathit{id}} \notin {\I}{.}\ILABELS \\
   & & | & {\mathit{id}}{:}{\Tid} & \quad\Rightarrow\quad{} & ({\mathit{id}}, \{ \ILABELS~{\mathit{id}} \} \oplus {\I}{}[{.}\ILABELS{}[x] = \epsilon]) & \quad \mbox{if}~ {\mathit{id}} = {\I}{.}\ILABELS{}[x] \\
   \end{array}

.. note::
   The new label entry is inserted at the *beginning* of the label list in the identifier context.
   This effectively shifts all existing labels up by one,
   mirroring the fact that control instructions are indexed relatively not absolutely.

   If a label with the same name already exists,
   then it is shadowed and the earlier label becomes inaccessible.


.. index:: parametric instruction, value type, polymorphism
   pair: text format; instruction
.. _text-instr-parametric:

Parametric Instructions
~~~~~~~~~~~~~~~~~~~~~~~

.. _text-drop:
.. _text-select:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tplaininstr}}_{{\I}} & ::= & \mbox{‘\texttt{unreachable}’} & \quad\Rightarrow\quad{} & \UNREACHABLE \\
   & & | & \mbox{‘\texttt{nop}’} & \quad\Rightarrow\quad{} & \NOP \\
   & & | & \mbox{‘\texttt{drop}’} & \quad\Rightarrow\quad{} & \DROP \\
   & & | & \mbox{‘\texttt{select}’}~~{({t^\ast}{:}{{{\Tresult}}_{{\I}}^\ast})^?} & \quad\Rightarrow\quad{} & \SELECT~{({t^\ast})^?} \\
   \end{array}


.. index:: control instructions, structured control, label, block, branch, result type, label index, function index, tag index, type index, list, polymorphism, reference
   pair: text format; instruction
.. _text-blockinstr:
.. _text-plaininstr:
.. _text-instr-control:

Control Instructions
~~~~~~~~~~~~~~~~~~~~

.. _text-blocktype:
.. _text-block:
.. _text-loop:
.. _text-if:
.. _text-instr-block:
.. _text-try_table:
.. _text-catch:

:ref:`Structured control instructions <syntax-instr-control>` can bind an optional symbolic :ref:`label identifier <text-label>`.
The same label identifier may optionally be repeated after the corresponding :math:`\mbox{‘\texttt{end}’}` or :math:`\mbox{‘\texttt{else}’}` keywords, to indicate the matching delimiters.

Their :ref:`block type <syntax-blocktype>` is given as a :ref:`type use <text-typeuse>`, analogous to the type of :ref:`functions <text-func>`.
However, the special case of a type use that is syntactically empty or consists of only a single :ref:`result <text-result>` is not regarded as an :ref:`abbreviation <text-typeuse-abbrev>` for an inline :ref:`function type <syntax-functype>`, but is parsed directly into an optional :ref:`value type <syntax-valtype>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tblocktype}}_{{\I}} & ::= & {t^?}{:}{{{\Tresult}}_{{\I}}^?} & \quad\Rightarrow\quad{} & {t^?} \\
   & & | & (x, {\I'}){:}{{\Ttypeuse}}_{{\I}} & \quad\Rightarrow\quad{} & x &  \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \quad \mbox{if}~ {\I'} = \{ \ILOCALS~{(\epsilon)^\ast} \}
   } \\[0.8ex]
   & {{\Tblockinstr}}_{{\I}} & ::= & \begin{array}[t]{@{}l@{}} \mbox{‘\texttt{block}’}~~({{\mathit{id}}^?}, {\I'}){:}{{\Tlabel}}_{{\I}}~~{\mathit{bt}}{:}{{\Tblocktype}}_{{\I}} \\
     {{\mathit{in}}^\ast}{:}{{\Tinstrs}}_{{\I'}} \\
     \mbox{‘\texttt{end}’}~~{{\mathit{id}'}^?}{:}{{\Tid}^?} \end{array} & \quad\Rightarrow\quad{} & \BLOCK~{\mathit{bt}}~{{\mathit{in}}^\ast} &  \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \quad \mbox{if}~ {{\mathit{id}'}^?} = \epsilon \lor {{\mathit{id}'}^?} = {{\mathit{id}}^?}
   } \\
   & & | & \begin{array}[t]{@{}l@{}} \mbox{‘\texttt{loop}’}~~({{\mathit{id}}^?}, {\I'}){:}{{\Tlabel}}_{{\I}}~~{\mathit{bt}}{:}{{\Tblocktype}}_{{\I}} \\
     {{\mathit{in}}^\ast}{:}{{\Tinstrs}}_{{\I'}} \\
     \mbox{‘\texttt{end}’}~~{{\mathit{id}'}^?}{:}{{\Tid}^?} \end{array} & \quad\Rightarrow\quad{} & \LOOP~{\mathit{bt}}~{{\mathit{in}}^\ast} &  \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \quad \mbox{if}~ {{\mathit{id}'}^?} = \epsilon \lor {{\mathit{id}'}^?} = {{\mathit{id}}^?}
   } \\
   & & | & \begin{array}[t]{@{}l@{}} \mbox{‘\texttt{if}’}~~({{\mathit{id}}^?}, {\I'}){:}{{\Tlabel}}_{{\I}}~~{\mathit{bt}}{:}{{\Tblocktype}}_{{\I}} \\
     {{\mathit{in}}_1^\ast}{:}{{\Tinstrs}}_{{\I'}} \\
     \mbox{‘\texttt{else}’}~~{{\mathit{id}}_1^?}{:}{{\Tid}^?} \\
     {{\mathit{in}}_2^\ast}{:}{{\Tinstrs}}_{{\I'}} \\
     \mbox{‘\texttt{end}’}~~{{\mathit{id}}_2^?}{:}{{\Tid}^?} \end{array} & \quad\Rightarrow\quad{} & \IF~{\mathit{bt}}~{{\mathit{in}}_1^\ast}~\ELSE~{{\mathit{in}}_2^\ast} &  \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \quad \mbox{if}~ ({{\mathit{id}}_1^?} = \epsilon \lor {{\mathit{id}}_1^?} = {{\mathit{id}}^?}) \land ({{\mathit{id}}_2^?} = \epsilon \lor {{\mathit{id}}_2^?} = {{\mathit{id}}^?})
   } \\
   & & | & \begin{array}[t]{@{}l@{}} \mbox{‘\texttt{try\_table}’}~~({{\mathit{id}}^?}, {\I'}){:}{{\Tlabel}}_{{\I}}~~{\mathit{bt}}{:}{{\Tblocktype}}_{{\I}} \\
     {c^\ast}{:}{{{\Tcatch}}_{{\I}}^\ast} \\
     {{\mathit{in}}^\ast}{:}{{\Tinstrs}}_{{\I'}} \\
     \mbox{‘\texttt{end}’}~~{{\mathit{id}'}^?}{:}{{\Tid}^?} \end{array} & \quad\Rightarrow\quad{} & \TRYTABLE~{\mathit{bt}}~{c^\ast}~{{\mathit{in}}^\ast} &  \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \quad \mbox{if}~ {{\mathit{id}'}^?} = \epsilon \lor {{\mathit{id}'}^?} = {{\mathit{id}}^?}
   } \\[0.8ex]
   & {{\Tcatch}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{catch}’}~~x{:}{{\Ttagidx}}_{{\I}}~~l{:}{{\Tlabelidx}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & \CATCH~x~l \\
   & & | & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{catch\_ref}’}~~x{:}{{\Ttagidx}}_{{\I}}~~l{:}{{\Tlabelidx}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & \CATCHREF~x~l \\
   & & | & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{catch\_all}’}~~l{:}{{\Tlabelidx}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & \CATCHALL~l \\
   & & | & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{catch\_all\_ref}’}~~l{:}{{\Tlabelidx}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & \CATCHALLREF~l \\
   \end{array}

.. note::
   The side condition stating that the :ref:`identifier context <text-context>` :math:`{\I'}` must only contain unnamed entries in the rule for :math:`{\mathtt{typeuse}}` block types enforces that no identifier can be bound in any :math:`{\mathtt{param}}` declaration for a block type.


.. _text-nop:
.. _text-unreachable:
.. _text-br:
.. _text-br_if:
.. _text-br_table:
.. _text-br_on_null:
.. _text-br_on_non_null:
.. _text-br_on_cast:
.. _text-br_on_cast_fail:
.. _text-return:
.. _text-call:
.. _text-call_ref:
.. _text-call_indirect:
.. _text-return_call:
.. _text-return_call_indirect:
.. _text-throw:
.. _text-throw_ref:

All other control instruction are represented verbatim.

.. note::
   The side condition stating that the :ref:`identifier context <text-context>` :math:`{\I'}` must only contain unnamed entries in the rule for |CALLINDIRECT| enforces that no identifier can be bound in any |Tparam| declaration appearing in the type annotation.


Abbreviations
.............

The :math:`\mbox{‘\texttt{else}’}` keyword of an :math:`\mbox{‘\texttt{if}’}` instruction can be omitted if the following instruction sequence is empty.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tblockinstr}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{if}’}~~{{\Tlabel}}_{{\I}}~~{{\Tblocktype}}_{{\I}}~~{{\Tinstrs}}_{{\I}}~~\mbox{‘\texttt{end}’}~~{{\Tid}^?} & \quad\equiv\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \mbox{‘\texttt{if}’}~~{{\Tlabel}}_{{\I}}~~{{\Tblocktype}}_{{\I}}~~{{\Tinstrs}}_{{\I}}~~\mbox{‘\texttt{else}’}~~\mbox{‘\texttt{end}’}~~{{\Tid}^?} \\
   \end{array}
   } \\
   \end{array}

Also, for backwards compatibility, the table index to :math:`\mbox{‘\texttt{call\_indirect}’}` and :math:`\mbox{‘\texttt{return\_call\_indirect}’}` can be omitted, defaulting to :math:`0`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tplaininstr}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{call\_indirect}’}~~{{\Ttypeuse}}_{{\I}} & \quad\equiv\quad{} & \mbox{‘\texttt{call\_indirect}’}~~\mbox{‘\texttt{0}’}~~{{\Ttypeuse}}_{{\I}} \\
   & & | & \mbox{‘\texttt{return\_call\_indirect}’}~~{{\Ttypeuse}}_{{\I}} & \quad\equiv\quad{} & \mbox{‘\texttt{return\_call\_indirect}’}~~\mbox{‘\texttt{0}’}~~{{\Ttypeuse}}_{{\I}} \\
   \end{array}


.. index:: variable instructions, local index, global index
   pair: text format; instruction
.. _text-instr-variable:

Variable Instructions
~~~~~~~~~~~~~~~~~~~~~

.. _text-local.get:
.. _text-local.set:
.. _text-local.tee:
.. _text-global.get:
.. _text-global.set:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tplaininstr}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{local.get}’}~~x{:}{{\Tlocalidx}}_{{\I}} & \quad\Rightarrow\quad{} & \LOCALGET~x \\
   & & | & \mbox{‘\texttt{local.set}’}~~x{:}{{\Tlocalidx}}_{{\I}} & \quad\Rightarrow\quad{} & \LOCALSET~x \\
   & & | & \mbox{‘\texttt{local.tee}’}~~x{:}{{\Tlocalidx}}_{{\I}} & \quad\Rightarrow\quad{} & \LOCALTEE~x \\
   & & | & \mbox{‘\texttt{global.get}’}~~x{:}{{\Tglobalidx}}_{{\I}} & \quad\Rightarrow\quad{} & \GLOBALGET~x \\
   & & | & \mbox{‘\texttt{global.set}’}~~x{:}{{\Tglobalidx}}_{{\I}} & \quad\Rightarrow\quad{} & \GLOBALSET~x \\
   \end{array}


.. index:: table instruction, table index
   pair: text format; instruction
.. _text-instr-table:

Table Instructions
~~~~~~~~~~~~~~~~~~

.. _text-table.get:
.. _text-table.set:
.. _text-table.size:
.. _text-table.grow:
.. _text-table.fill:
.. _text-table.copy:
.. _text-table.init:
.. _text-elem.drop:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tplaininstr}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{table.get}’}~~x{:}{{\Ttableidx}}_{{\I}} & \quad\Rightarrow\quad{} & \TABLEGET~x \\
   & & | & \mbox{‘\texttt{table.set}’}~~x{:}{{\Ttableidx}}_{{\I}} & \quad\Rightarrow\quad{} & \TABLESET~x \\
   & & | & \mbox{‘\texttt{table.size}’}~~x{:}{{\Ttableidx}}_{{\I}} & \quad\Rightarrow\quad{} & \TABLESIZE~x \\
   & & | & \mbox{‘\texttt{table.grow}’}~~x{:}{{\Ttableidx}}_{{\I}} & \quad\Rightarrow\quad{} & \TABLEGROW~x \\
   & & | & \mbox{‘\texttt{table.fill}’}~~x{:}{{\Ttableidx}}_{{\I}} & \quad\Rightarrow\quad{} & \TABLEFILL~x \\
   & & | & \mbox{‘\texttt{table.copy}’}~~x_1{:}{{\Ttableidx}}_{{\I}}~~x_2{:}{{\Ttableidx}}_{{\I}} & \quad\Rightarrow\quad{} & \TABLECOPY~x_1~x_2 \\
   & & | & \mbox{‘\texttt{table.init}’}~~x{:}{{\Ttableidx}}_{{\I}}~~y{:}{{\Telemidx}}_{{\I}} & \quad\Rightarrow\quad{} & \TABLEINIT~x~y \\
   & & | & \mbox{‘\texttt{elem.drop}’}~~x{:}{{\Telemidx}}_{{\I}} & \quad\Rightarrow\quad{} & \ELEMDROP~x \\
   \end{array}


Abbreviations
.............

For backwards compatibility, all :ref:`table indices <syntax-tableidx>` may be omitted from table instructions, defaulting to :math:`0`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tplaininstr}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{table.get}’} & \quad\equiv\quad{} & \mbox{‘\texttt{table.get}’}~~\mbox{‘\texttt{0}’} \\
   & & | & \mbox{‘\texttt{table.set}’} & \quad\equiv\quad{} & \mbox{‘\texttt{table.set}’}~~\mbox{‘\texttt{0}’} \\
   & & | & \mbox{‘\texttt{table.size}’} & \quad\equiv\quad{} & \mbox{‘\texttt{table.size}’}~~\mbox{‘\texttt{0}’} \\
   & & | & \mbox{‘\texttt{table.grow}’} & \quad\equiv\quad{} & \mbox{‘\texttt{table.grow}’}~~\mbox{‘\texttt{0}’} \\
   & & | & \mbox{‘\texttt{table.fill}’} & \quad\equiv\quad{} & \mbox{‘\texttt{table.fill}’}~~\mbox{‘\texttt{0}’} \\
   & & | & \mbox{‘\texttt{table.copy}’} & \quad\equiv\quad{} & \mbox{‘\texttt{table.copy}’}~~\mbox{‘\texttt{0}’}~~\mbox{‘\texttt{0}’} \\
   & & | & \mbox{‘\texttt{table.init}’}~~{{\Telemidx}}_{{\I}} & \quad\equiv\quad{} & \mbox{‘\texttt{table.init}’}~~\mbox{‘\texttt{0}’}~~{{\Telemidx}}_{{\I}} \\
   \end{array}


.. index:: memory instruction, memory index
   pair: text format; instruction
.. _text-instr-memory:

Memory Instructions
~~~~~~~~~~~~~~~~~~~

.. _text-memarg:
.. _text-laneidx:
.. _text-load:
.. _text-loadn:
.. _text-store:
.. _text-storen:
.. _text-memory.size:
.. _text-memory.grow:
.. _text-memory.fill:
.. _text-memory.copy:
.. _text-memory.init:
.. _text-data.drop:

The offset and alignment immediates to memory instructions are optional.
The offset defaults to :math:`0`, the alignment to the storage size of the respective memory access, which is its *natural alignment*.
Lexically, an :math:`{\Toffset}` or :math:`{\Talign}` phrase is considered a single :ref:`keyword token <text-keyword>`, so no :ref:`white space <text-space>` is allowed around the :math:`\mbox{‘\texttt{{=}}’}`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tmemarg}}_{N} & ::= & m{:}{\Toffset}~~n{:}{{\Talign}}_{N} & \quad\Rightarrow\quad{} & \{ \ALIGN~n,\;\allowbreak \OFFSET~m \} \\
   & {\Toffset} & ::= & \mbox{‘\texttt{offset{=}}’}~~m{:}{\Tu64} & \quad\Rightarrow\quad{} & m \\
   & & | & \epsilon & \quad\Rightarrow\quad{} & 0 \\
   & {{\Talign}}_{N} & ::= & \mbox{‘\texttt{align{=}}’}~~m{:}{\Tu64} & \quad\Rightarrow\quad{} & n & \quad \mbox{if}~ m = {2^{n}} \\
   & & | & \epsilon & \quad\Rightarrow\quad{} & N \\
   & {\Tlaneidx} & ::= & i{:}{\Tu8} & \quad\Rightarrow\quad{} & i \\
   \end{array}

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tplaininstr}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{i32.load}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{4} & \quad\Rightarrow\quad{} & \I32{.}\LOAD~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{i64.load}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{8} & \quad\Rightarrow\quad{} & \I64{.}\LOAD~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{f32.load}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{4} & \quad\Rightarrow\quad{} & \F32{.}\LOAD~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{f64.load}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{8} & \quad\Rightarrow\quad{} & \F64{.}\LOAD~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{i32.load8\_s}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{1} & \quad\Rightarrow\quad{} & {\I32{.}\LOAD}{{\mathsf{{\scriptstyle 8}}}{\mathsf{\_}}{\S}}~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{i32.load8\_u}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{1} & \quad\Rightarrow\quad{} & {\I32{.}\LOAD}{{\mathsf{{\scriptstyle 8}}}{\mathsf{\_}}{\U}}~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{i32.load16\_s}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{2} & \quad\Rightarrow\quad{} & {\I32{.}\LOAD}{{\mathsf{{\scriptstyle 16}}}{\mathsf{\_}}{\S}}~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{i32.load16\_u}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{2} & \quad\Rightarrow\quad{} & {\I32{.}\LOAD}{{\mathsf{{\scriptstyle 16}}}{\mathsf{\_}}{\U}}~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{i64.load8\_s}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{1} & \quad\Rightarrow\quad{} & {\I64{.}\LOAD}{{\mathsf{{\scriptstyle 8}}}{\mathsf{\_}}{\S}}~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{i64.load8\_u}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{1} & \quad\Rightarrow\quad{} & {\I64{.}\LOAD}{{\mathsf{{\scriptstyle 8}}}{\mathsf{\_}}{\U}}~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{i64.load16\_s}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{2} & \quad\Rightarrow\quad{} & {\I64{.}\LOAD}{{\mathsf{{\scriptstyle 16}}}{\mathsf{\_}}{\S}}~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{i64.load16\_u}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{2} & \quad\Rightarrow\quad{} & {\I64{.}\LOAD}{{\mathsf{{\scriptstyle 16}}}{\mathsf{\_}}{\U}}~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{i64.load32\_s}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{4} & \quad\Rightarrow\quad{} & {\I64{.}\LOAD}{{\mathsf{{\scriptstyle 32}}}{\mathsf{\_}}{\S}}~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{i64.load32\_u}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{4} & \quad\Rightarrow\quad{} & {\I64{.}\LOAD}{{\mathsf{{\scriptstyle 32}}}{\mathsf{\_}}{\U}}~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{v128.load}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{16} & \quad\Rightarrow\quad{} & \V128{.}\VLOAD~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{v128.load8x8\_s}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{8} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{{\mathsf{{\scriptstyle 8}}}{\Xshape}{\mathsf{{\scriptstyle 8}}}{\mathsf{\_}}{\S}}~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{v128.load8x8\_u}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{8} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{{\mathsf{{\scriptstyle 8}}}{\Xshape}{\mathsf{{\scriptstyle 8}}}{\mathsf{\_}}{\U}}~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{v128.load16x4\_s}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{8} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{{\mathsf{{\scriptstyle 16}}}{\Xshape}{\mathsf{{\scriptstyle 4}}}{\mathsf{\_}}{\S}}~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{v128.load16x4\_u}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{8} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{{\mathsf{{\scriptstyle 16}}}{\Xshape}{\mathsf{{\scriptstyle 4}}}{\mathsf{\_}}{\U}}~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{v128.load32x2\_s}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{8} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{{\mathsf{{\scriptstyle 32}}}{\Xshape}{\mathsf{{\scriptstyle 2}}}{\mathsf{\_}}{\S}}~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{v128.load32x2\_u}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{8} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{{\mathsf{{\scriptstyle 32}}}{\Xshape}{\mathsf{{\scriptstyle 2}}}{\mathsf{\_}}{\U}}~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{v128.load8\_splat}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{1} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{{\mathsf{{\scriptstyle 8}}}{\mathsf{\_}}{\LSPLAT}}~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{v128.load16\_splat}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{2} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{{\mathsf{{\scriptstyle 16}}}{\mathsf{\_}}{\LSPLAT}}~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{v128.load32\_splat}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{4} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{{\mathsf{{\scriptstyle 32}}}{\mathsf{\_}}{\LSPLAT}}~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{v128.load64\_splat}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{8} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{{\mathsf{{\scriptstyle 64}}}{\mathsf{\_}}{\LSPLAT}}~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{v128.load32\_zero}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{4} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{{\mathsf{{\scriptstyle 32}}}{\mathsf{\_}}{\LZERO}}~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{v128.load64\_zero}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{8} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{{\mathsf{{\scriptstyle 64}}}{\mathsf{\_}}{\LZERO}}~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{v128.load8\_lane}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{1}~~i{:}{\Tlaneidx} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{\mathsf{{\scriptstyle 8}}}{\mathsf{\_}}{\VLANE}~x~{\mathit{ao}}~i \\
   & & | & \mbox{‘\texttt{v128.load16\_lane}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{2}~~i{:}{\Tlaneidx} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{\mathsf{{\scriptstyle 16}}}{\mathsf{\_}}{\VLANE}~x~{\mathit{ao}}~i \\
   & & | & \mbox{‘\texttt{v128.load32\_lane}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{4}~~i{:}{\Tlaneidx} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{\mathsf{{\scriptstyle 32}}}{\mathsf{\_}}{\VLANE}~x~{\mathit{ao}}~i \\
   & & | & \mbox{‘\texttt{v128.load64\_lane}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{8}~~i{:}{\Tlaneidx} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{\mathsf{{\scriptstyle 64}}}{\mathsf{\_}}{\VLANE}~x~{\mathit{ao}}~i \\
   & & | & \mbox{‘\texttt{i32.store}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{4} & \quad\Rightarrow\quad{} & \I32{.}\STORE~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{i64.store}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{8} & \quad\Rightarrow\quad{} & \I64{.}\STORE~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{f32.store}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{4} & \quad\Rightarrow\quad{} & \F32{.}\STORE~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{f64.store}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{8} & \quad\Rightarrow\quad{} & \F64{.}\STORE~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{i32.store8}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{1} & \quad\Rightarrow\quad{} & {\I32{.}\STORE}{\mathsf{{\scriptstyle 8}}}~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{i32.store16}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{2} & \quad\Rightarrow\quad{} & {\I32{.}\STORE}{\mathsf{{\scriptstyle 16}}}~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{i64.store8}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{1} & \quad\Rightarrow\quad{} & {\I64{.}\STORE}{\mathsf{{\scriptstyle 8}}}~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{i64.store16}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{2} & \quad\Rightarrow\quad{} & {\I64{.}\STORE}{\mathsf{{\scriptstyle 16}}}~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{i64.store32}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{4} & \quad\Rightarrow\quad{} & {\I64{.}\STORE}{\mathsf{{\scriptstyle 32}}}~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{v128.store}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{16} & \quad\Rightarrow\quad{} & \V128{.}\VSTORE~x~{\mathit{ao}} \\
   & & | & \mbox{‘\texttt{v128.store8\_lane}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{1}~~i{:}{\Tlaneidx} & \quad\Rightarrow\quad{} & {\V128{.}\VSTORE}{\mathsf{{\scriptstyle 8}}}{\mathsf{\_}}{\VLANE}~x~{\mathit{ao}}~i \\
   & & | & \mbox{‘\texttt{v128.store16\_lane}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{2}~~i{:}{\Tlaneidx} & \quad\Rightarrow\quad{} & {\V128{.}\VSTORE}{\mathsf{{\scriptstyle 16}}}{\mathsf{\_}}{\VLANE}~x~{\mathit{ao}}~i \\
   & & | & \mbox{‘\texttt{v128.store32\_lane}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{4}~~i{:}{\Tlaneidx} & \quad\Rightarrow\quad{} & {\V128{.}\VSTORE}{\mathsf{{\scriptstyle 32}}}{\mathsf{\_}}{\VLANE}~x~{\mathit{ao}}~i \\
   & & | & \mbox{‘\texttt{v128.store64\_lane}’}~~x{:}{{\Tmemidx}}_{{\I}}~~{\mathit{ao}}{:}{{\Tmemarg}}_{8}~~i{:}{\Tlaneidx} & \quad\Rightarrow\quad{} & {\V128{.}\VSTORE}{\mathsf{{\scriptstyle 64}}}{\mathsf{\_}}{\VLANE}~x~{\mathit{ao}}~i \\
   & & | & \mbox{‘\texttt{memory.size}’}~~x{:}{{\Tmemidx}}_{{\I}} & \quad\Rightarrow\quad{} & \MEMORYSIZE~x \\
   & & | & \mbox{‘\texttt{memory.grow}’}~~x{:}{{\Tmemidx}}_{{\I}} & \quad\Rightarrow\quad{} & \MEMORYGROW~x \\
   & & | & \mbox{‘\texttt{memory.fill}’}~~x{:}{{\Tmemidx}}_{{\I}} & \quad\Rightarrow\quad{} & \MEMORYFILL~x \\
   & & | & \mbox{‘\texttt{memory.copy}’}~~x_1{:}{{\Tmemidx}}_{{\I}}~~x_2{:}{{\Tmemidx}}_{{\I}} & \quad\Rightarrow\quad{} & \MEMORYCOPY~x_1~x_2 \\
   & & | & \mbox{‘\texttt{memory.init}’}~~x{:}{{\Tmemidx}}_{{\I}}~~y{:}{{\Tdataidx}}_{{\I}} & \quad\Rightarrow\quad{} & \MEMORYINIT~x~y \\
   & & | & \mbox{‘\texttt{data.drop}’}~~x{:}{{\Tdataidx}}_{{\I}} & \quad\Rightarrow\quad{} & \DATADROP~x \\
   \end{array}


Abbreviations
.............

As an abbreviation, the memory index can be omitted in all memory instructions, defaulting to :math:`0`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tplaininstr}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{i32.load}’}~~{{\Tmemarg}}_{4} & \quad\equiv\quad{} & \mbox{‘\texttt{i32.load}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{4} \\
   & & | & \mbox{‘\texttt{i64.load}’}~~{{\Tmemarg}}_{8} & \quad\equiv\quad{} & \mbox{‘\texttt{i64.load}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{8} \\
   & & | & \mbox{‘\texttt{f32.load}’}~~{{\Tmemarg}}_{4} & \quad\equiv\quad{} & \mbox{‘\texttt{f32.load}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{4} \\
   & & | & \mbox{‘\texttt{f64.load}’}~~{{\Tmemarg}}_{8} & \quad\equiv\quad{} & \mbox{‘\texttt{f64.load}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{8} \\
   & & | & \mbox{‘\texttt{i32.load8\_s}’}~~{{\Tmemarg}}_{1} & \quad\equiv\quad{} & \mbox{‘\texttt{i32.load8\_s}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{1} \\
   & & | & \mbox{‘\texttt{i32.load8\_u}’}~~{{\Tmemarg}}_{1} & \quad\equiv\quad{} & \mbox{‘\texttt{i32.load8\_u}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{1} \\
   & & | & \mbox{‘\texttt{i32.load16\_s}’}~~{{\Tmemarg}}_{2} & \quad\equiv\quad{} & \mbox{‘\texttt{i32.load16\_s}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{2} \\
   & & | & \mbox{‘\texttt{i32.load16\_u}’}~~{{\Tmemarg}}_{2} & \quad\equiv\quad{} & \mbox{‘\texttt{i32.load16\_u}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{2} \\
   & & | & \mbox{‘\texttt{i64.load8\_s}’}~~{{\Tmemarg}}_{1} & \quad\equiv\quad{} & \mbox{‘\texttt{i64.load8\_s}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{1} \\
   & & | & \mbox{‘\texttt{i64.load8\_u}’}~~{{\Tmemarg}}_{1} & \quad\equiv\quad{} & \mbox{‘\texttt{i64.load8\_u}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{1} \\
   & & | & \mbox{‘\texttt{i64.load16\_s}’}~~{{\Tmemarg}}_{2} & \quad\equiv\quad{} & \mbox{‘\texttt{i64.load16\_s}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{2} \\
   & & | & \mbox{‘\texttt{i64.load16\_u}’}~~{{\Tmemarg}}_{2} & \quad\equiv\quad{} & \mbox{‘\texttt{i64.load16\_u}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{2} \\
   & & | & \mbox{‘\texttt{i64.load32\_s}’}~~{{\Tmemarg}}_{4} & \quad\equiv\quad{} & \mbox{‘\texttt{i64.load32\_s}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{4} \\
   & & | & \mbox{‘\texttt{i64.load32\_u}’}~~{{\Tmemarg}}_{4} & \quad\equiv\quad{} & \mbox{‘\texttt{i64.load32\_u}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{4} \\
   & & | & \mbox{‘\texttt{v128.load}’}~~{{\Tmemarg}}_{16} & \quad\equiv\quad{} & \mbox{‘\texttt{v128.load}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{16} \\
   & & | & \mbox{‘\texttt{v128.load8x8\_s}’}~~{{\Tmemarg}}_{8} & \quad\equiv\quad{} & \mbox{‘\texttt{v128.load8x8\_s}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{8} \\
   & & | & \mbox{‘\texttt{v128.load8x8\_u}’}~~{{\Tmemarg}}_{8} & \quad\equiv\quad{} & \mbox{‘\texttt{v128.load8x8\_u}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{8} \\
   & & | & \mbox{‘\texttt{v128.load16x4\_s}’}~~{{\Tmemarg}}_{8} & \quad\equiv\quad{} & \mbox{‘\texttt{v128.load16x4\_s}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{8} \\
   & & | & \mbox{‘\texttt{v128.load16x4\_u}’}~~{{\Tmemarg}}_{8} & \quad\equiv\quad{} & \mbox{‘\texttt{v128.load16x4\_u}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{8} \\
   & & | & \mbox{‘\texttt{v128.load32x2\_s}’}~~{{\Tmemarg}}_{8} & \quad\equiv\quad{} & \mbox{‘\texttt{v128.load32x2\_s}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{8} \\
   & & | & \mbox{‘\texttt{v128.load32x2\_u}’}~~{{\Tmemarg}}_{8} & \quad\equiv\quad{} & \mbox{‘\texttt{v128.load32x2\_u}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{8} \\
   & & | & \mbox{‘\texttt{v128.load8\_splat}’}~~{{\Tmemarg}}_{1} & \quad\equiv\quad{} & \mbox{‘\texttt{v128.load8\_splat}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{1} \\
   & & | & \mbox{‘\texttt{v128.load16\_splat}’}~~{{\Tmemarg}}_{2} & \quad\equiv\quad{} & \mbox{‘\texttt{v128.load16\_splat}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{2} \\
   & & | & \mbox{‘\texttt{v128.load32\_splat}’}~~{{\Tmemarg}}_{4} & \quad\equiv\quad{} & \mbox{‘\texttt{v128.load32\_splat}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{4} \\
   & & | & \mbox{‘\texttt{v128.load64\_splat}’}~~{{\Tmemarg}}_{8} & \quad\equiv\quad{} & \mbox{‘\texttt{v128.load64\_splat}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{8} \\
   & & | & \mbox{‘\texttt{v128.load32\_zero}’}~~{{\Tmemarg}}_{4} & \quad\equiv\quad{} & \mbox{‘\texttt{v128.load32\_zero}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{4} \\
   & & | & \mbox{‘\texttt{v128.load64\_zero}’}~~{{\Tmemarg}}_{8} & \quad\equiv\quad{} & \mbox{‘\texttt{v128.load64\_zero}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{8} \\
   & & | & \mbox{‘\texttt{v128.load8\_lane}’}~~{{\Tmemarg}}_{1}~~{\Tlaneidx} & \quad\equiv\quad{} & \mbox{‘\texttt{v128.load8\_lane}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{1}~~{\Tlaneidx} \\
   & & | & \mbox{‘\texttt{v128.load16\_lane}’}~~{{\Tmemarg}}_{2}~~{\Tlaneidx} & \quad\equiv\quad{} & \mbox{‘\texttt{v128.load16\_lane}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{2}~~{\Tlaneidx} \\
   & & | & \mbox{‘\texttt{v128.load32\_lane}’}~~{{\Tmemarg}}_{4}~~{\Tlaneidx} & \quad\equiv\quad{} & \mbox{‘\texttt{v128.load32\_lane}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{4}~~{\Tlaneidx} \\
   & & | & \mbox{‘\texttt{v128.load64\_lane}’}~~{{\Tmemarg}}_{8}~~{\Tlaneidx} & \quad\equiv\quad{} & \mbox{‘\texttt{v128.load64\_lane}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{8}~~{\Tlaneidx} \\
   & & | & \mbox{‘\texttt{i32.store}’}~~{{\Tmemarg}}_{4} & \quad\equiv\quad{} & \mbox{‘\texttt{i32.store}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{4} \\
   & & | & \mbox{‘\texttt{i64.store}’}~~{{\Tmemarg}}_{8} & \quad\equiv\quad{} & \mbox{‘\texttt{i64.store}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{8} \\
   & & | & \mbox{‘\texttt{f32.store}’}~~{{\Tmemarg}}_{4} & \quad\equiv\quad{} & \mbox{‘\texttt{f32.store}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{4} \\
   & & | & \mbox{‘\texttt{f64.store}’}~~{{\Tmemarg}}_{8} & \quad\equiv\quad{} & \mbox{‘\texttt{f64.store}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{8} \\
   & & | & \mbox{‘\texttt{i32.store8}’}~~{{\Tmemarg}}_{1} & \quad\equiv\quad{} & \mbox{‘\texttt{i32.store8}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{1} \\
   & & | & \mbox{‘\texttt{i32.store16}’}~~{{\Tmemarg}}_{2} & \quad\equiv\quad{} & \mbox{‘\texttt{i32.store16}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{2} \\
   & & | & \mbox{‘\texttt{i64.store8}’}~~{{\Tmemarg}}_{1} & \quad\equiv\quad{} & \mbox{‘\texttt{i64.store8}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{1} \\
   & & | & \mbox{‘\texttt{i64.store16}’}~~{{\Tmemarg}}_{2} & \quad\equiv\quad{} & \mbox{‘\texttt{i64.store16}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{2} \\
   & & | & \mbox{‘\texttt{i64.store32}’}~~{{\Tmemarg}}_{4} & \quad\equiv\quad{} & \mbox{‘\texttt{i64.store32}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{4} \\
   & & | & \mbox{‘\texttt{v128.store}’}~~{{\Tmemarg}}_{16} & \quad\equiv\quad{} & \mbox{‘\texttt{v128.store}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{16} \\
   & & | & \mbox{‘\texttt{v128.store8\_lane}’}~~{{\Tmemarg}}_{1}~~{\Tlaneidx} & \quad\equiv\quad{} & \mbox{‘\texttt{v128.store8\_lane}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{1}~~{\Tlaneidx} \\
   & & | & \mbox{‘\texttt{v128.store16\_lane}’}~~{{\Tmemarg}}_{2}~~{\Tlaneidx} & \quad\equiv\quad{} & \mbox{‘\texttt{v128.store16\_lane}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{2}~~{\Tlaneidx} \\
   & & | & \mbox{‘\texttt{v128.store32\_lane}’}~~{{\Tmemarg}}_{4}~~{\Tlaneidx} & \quad\equiv\quad{} & \mbox{‘\texttt{v128.store32\_lane}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{4}~~{\Tlaneidx} \\
   & & | & \mbox{‘\texttt{v128.store64\_lane}’}~~{{\Tmemarg}}_{8}~~{\Tlaneidx} & \quad\equiv\quad{} & \mbox{‘\texttt{v128.store64\_lane}’}~~\mbox{‘\texttt{0}’}~~{{\Tmemarg}}_{8}~~{\Tlaneidx} \\
   & & | & \mbox{‘\texttt{memory.size}’} & \quad\equiv\quad{} & \mbox{‘\texttt{memory.size}’}~~\mbox{‘\texttt{0}’} \\
   & & | & \mbox{‘\texttt{memory.grow}’} & \quad\equiv\quad{} & \mbox{‘\texttt{memory.grow}’}~~\mbox{‘\texttt{0}’} \\
   & & | & \mbox{‘\texttt{memory.fill}’} & \quad\equiv\quad{} & \mbox{‘\texttt{memory.fill}’}~~\mbox{‘\texttt{0}’} \\
   & & | & \mbox{‘\texttt{memory.copy}’} & \quad\equiv\quad{} & \mbox{‘\texttt{memory.copy}’}~~\mbox{‘\texttt{0}’}~~\mbox{‘\texttt{0}’} \\
   & & | & \mbox{‘\texttt{memory.init}’}~~{{\Tdataidx}}_{{\I}} & \quad\equiv\quad{} & \mbox{‘\texttt{memory.init}’}~~\mbox{‘\texttt{0}’}~~{{\Tdataidx}}_{{\I}} \\
   \end{array}


.. index:: reference instruction
   pair: text format; instruction
.. _text-instr-ref:

Reference Instructions
~~~~~~~~~~~~~~~~~~~~~~

.. _text-ref.null:
.. _text-ref.func:
.. _text-ref.is_null:
.. _text-ref.as_non_null:
.. _text-ref.test:
.. _text-ref.cast:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tplaininstr}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{ref.null}’}~~{\mathit{ht}}{:}{{\Theaptype}}_{{\I}} & \quad\Rightarrow\quad{} & \REFNULL~{\mathit{ht}} \\
   & & | & \mbox{‘\texttt{ref.func}’}~~x{:}{{\Tfuncidx}}_{{\I}} & \quad\Rightarrow\quad{} & \REFFUNC~x \\
   & & | & \mbox{‘\texttt{ref.is\_null}’} & \quad\Rightarrow\quad{} & \REFISNULL \\
   & & | & \mbox{‘\texttt{ref.as\_non\_null}’} & \quad\Rightarrow\quad{} & \REFASNONNULL \\
   & & | & \mbox{‘\texttt{ref.eq}’} & \quad\Rightarrow\quad{} & \REFEQ \\
   & & | & \mbox{‘\texttt{ref.test}’}~~{\mathit{rt}}{:}{{\Treftype}}_{{\I}} & \quad\Rightarrow\quad{} & \REFTEST~{\mathit{rt}} \\
   & & | & \mbox{‘\texttt{ref.cast}’}~~{\mathit{rt}}{:}{{\Treftype}}_{{\I}} & \quad\Rightarrow\quad{} & \REFCAST~{\mathit{rt}} \\
   \end{array}


.. index:: aggregate instruction
   pair: text format; instruction
.. _text-instr-aggr:

Aggregate Instructions
~~~~~~~~~~~~~~~~~~~~~~

.. _text-ref.i31:
.. _text-i31.get_s:
.. _text-i31.get_u:
.. _text-struct.new:
.. _text-struct.new_default:
.. _text-struct.get:
.. _text-struct.get_s:
.. _text-struct.get_u:
.. _text-struct.set:
.. _text-array.new:
.. _text-array.new_default:
.. _text-array.new_fixed:
.. _text-array.new_elem:
.. _text-array.new_data:
.. _text-array.get:
.. _text-array.get_s:
.. _text-array.get_u:
.. _text-array.set:
.. _text-array.len:
.. _text-array.fill:
.. _text-array.copy:
.. _text-array.init_data:
.. _text-array.init_elem:
.. _text-any.convert_extern:
.. _text-extern.convert_any:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tplaininstr}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{ref.i31}’} & \quad\Rightarrow\quad{} & \REFI31 \\
   & & | & \mbox{‘\texttt{i31.get\_s}’} & \quad\Rightarrow\quad{} & {\I31GET}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i31.get\_u}’} & \quad\Rightarrow\quad{} & {\I31GET}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{struct.new}’}~~x{:}{{\Ttypeidx}}_{{\I}} & \quad\Rightarrow\quad{} & \STRUCTNEW~x \\
   & & | & \mbox{‘\texttt{struct.new\_default}’}~~x{:}{{\Ttypeidx}}_{{\I}} & \quad\Rightarrow\quad{} & \STRUCTNEWDEFAULT~x \\
   & & | & \mbox{‘\texttt{struct.get}’}~~x{:}{{\Ttypeidx}}_{{\I}}~~i{:}{{\Tfieldidx}}_{{\I}, x} & \quad\Rightarrow\quad{} & \STRUCTGET~x~i \\
   & & | & \mbox{‘\texttt{struct.get\_s}’}~~x{:}{{\Ttypeidx}}_{{\I}}~~i{:}{{\Tfieldidx}}_{{\I}, x} & \quad\Rightarrow\quad{} & {\STRUCTGET}{\mathsf{\_}}{\S}~x~i \\
   & & | & \mbox{‘\texttt{struct.get\_u}’}~~x{:}{{\Ttypeidx}}_{{\I}}~~i{:}{{\Tfieldidx}}_{{\I}, x} & \quad\Rightarrow\quad{} & {\STRUCTGET}{\mathsf{\_}}{\U}~x~i \\
   & & | & \mbox{‘\texttt{struct.set}’}~~x{:}{{\Ttypeidx}}_{{\I}}~~i{:}{{\Tfieldidx}}_{{\I}, x} & \quad\Rightarrow\quad{} & \STRUCTSET~x~i \\
   & & | & \mbox{‘\texttt{array.new}’}~~x{:}{{\Ttypeidx}}_{{\I}} & \quad\Rightarrow\quad{} & \ARRAYNEW~x \\
   & & | & \mbox{‘\texttt{array.new\_default}’}~~x{:}{{\Ttypeidx}}_{{\I}} & \quad\Rightarrow\quad{} & \ARRAYNEWDEFAULT~x \\
   & & | & \mbox{‘\texttt{array.new\_fixed}’}~~x{:}{{\Ttypeidx}}_{{\I}}~~n{:}{\Tu32} & \quad\Rightarrow\quad{} & \ARRAYNEWFIXED~x~n \\
   & & | & \mbox{‘\texttt{array.new\_data}’}~~x{:}{{\Ttypeidx}}_{{\I}}~~y{:}{{\Tdataidx}}_{{\I}} & \quad\Rightarrow\quad{} & \ARRAYNEWDATA~x~y \\
   & & | & \mbox{‘\texttt{array.new\_elem}’}~~x{:}{{\Ttypeidx}}_{{\I}}~~y{:}{{\Telemidx}}_{{\I}} & \quad\Rightarrow\quad{} & \ARRAYNEWELEM~x~y \\
   & & | & \mbox{‘\texttt{array.get}’}~~x{:}{{\Ttypeidx}}_{{\I}} & \quad\Rightarrow\quad{} & \ARRAYGET~x \\
   & & | & \mbox{‘\texttt{array.get\_s}’}~~x{:}{{\Ttypeidx}}_{{\I}} & \quad\Rightarrow\quad{} & {\ARRAYGET}{\mathsf{\_}}{\S}~x \\
   & & | & \mbox{‘\texttt{array.get\_u}’}~~x{:}{{\Ttypeidx}}_{{\I}} & \quad\Rightarrow\quad{} & {\ARRAYGET}{\mathsf{\_}}{\U}~x \\
   & & | & \mbox{‘\texttt{array.set}’}~~x{:}{{\Ttypeidx}}_{{\I}} & \quad\Rightarrow\quad{} & \ARRAYSET~x \\
   & & | & \mbox{‘\texttt{array.len}’} & \quad\Rightarrow\quad{} & \ARRAYLEN \\
   & & | & \mbox{‘\texttt{array.fill}’}~~x{:}{{\Ttypeidx}}_{{\I}} & \quad\Rightarrow\quad{} & \ARRAYFILL~x \\
   & & | & \mbox{‘\texttt{array.copy}’}~~x_1{:}{{\Ttypeidx}}_{{\I}}~~x_2{:}{{\Ttypeidx}}_{{\I}} & \quad\Rightarrow\quad{} & \ARRAYCOPY~x_1~x_2 \\
   & & | & \mbox{‘\texttt{array.init\_data}’}~~x{:}{{\Ttypeidx}}_{{\I}}~~y{:}{{\Tdataidx}}_{{\I}} & \quad\Rightarrow\quad{} & \ARRAYINITDATA~x~y \\
   & & | & \mbox{‘\texttt{array.init\_elem}’}~~x{:}{{\Ttypeidx}}_{{\I}}~~y{:}{{\Telemidx}}_{{\I}} & \quad\Rightarrow\quad{} & \ARRAYINITELEM~x~y \\
   & & | & \mbox{‘\texttt{any.convert\_extern}’} & \quad\Rightarrow\quad{} & \ANYCONVERTEXTERN \\
   & & | & \mbox{‘\texttt{extern.convert\_any}’} & \quad\Rightarrow\quad{} & \EXTERNCONVERTANY \\
   \end{array}


.. index:: numeric instruction
   pair: text format; instruction
.. _text-instr-numeric:

Numeric Instructions
~~~~~~~~~~~~~~~~~~~~

.. _text-const:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tplaininstr}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{i32.const}’}~~c{:}{\Ti32} & \quad\Rightarrow\quad{} & \I32{.}\CONST~c \\
   & & | & \mbox{‘\texttt{i64.const}’}~~c{:}{\Ti64} & \quad\Rightarrow\quad{} & \I64{.}\CONST~c \\
   & & | & \mbox{‘\texttt{f32.const}’}~~c{:}{\Tf32} & \quad\Rightarrow\quad{} & \F32{.}\CONST~c \\
   & & | & \mbox{‘\texttt{f64.const}’}~~c{:}{\Tf64} & \quad\Rightarrow\quad{} & \F64{.}\CONST~c \\
   \end{array}

.. _text-testop:
.. _text-relop:
.. _text-unop:
.. _text-binop:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tplaininstr}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{i32.eqz}’} & \quad\Rightarrow\quad{} & \I32 {.} \EQZ \\
   & & | & \mbox{‘\texttt{i32.eq}’} & \quad\Rightarrow\quad{} & \I32 {.} \EQ \\
   & & | & \mbox{‘\texttt{i32.ne}’} & \quad\Rightarrow\quad{} & \I32 {.} \NE \\
   & & | & \mbox{‘\texttt{i32.lt\_s}’} & \quad\Rightarrow\quad{} & \I32 {.} {\LT}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i32.lt\_u}’} & \quad\Rightarrow\quad{} & \I32 {.} {\LT}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i32.gt\_s}’} & \quad\Rightarrow\quad{} & \I32 {.} {\GT}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i32.gt\_u}’} & \quad\Rightarrow\quad{} & \I32 {.} {\GT}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i32.le\_s}’} & \quad\Rightarrow\quad{} & \I32 {.} {\LE}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i32.le\_u}’} & \quad\Rightarrow\quad{} & \I32 {.} {\LE}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i32.ge\_s}’} & \quad\Rightarrow\quad{} & \I32 {.} {\GE}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i32.ge\_u}’} & \quad\Rightarrow\quad{} & \I32 {.} {\GE}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i32.clz}’} & \quad\Rightarrow\quad{} & \I32 {.} \CLZ \\
   & & | & \mbox{‘\texttt{i32.ctz}’} & \quad\Rightarrow\quad{} & \I32 {.} \CTZ \\
   & & | & \mbox{‘\texttt{i32.popcnt}’} & \quad\Rightarrow\quad{} & \I32 {.} \POPCNT \\
   & & | & \mbox{‘\texttt{i32.extend8\_s}’} & \quad\Rightarrow\quad{} & \I32 {.} {\EXTEND}{\mathsf{{\scriptstyle 8}}}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i32.extend16\_s}’} & \quad\Rightarrow\quad{} & \I32 {.} {\EXTEND}{\mathsf{{\scriptstyle 16}}}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i32.add}’} & \quad\Rightarrow\quad{} & \I32 {.} \ADD \\
   & & | & \mbox{‘\texttt{i32.sub}’} & \quad\Rightarrow\quad{} & \I32 {.} \SUB \\
   & & | & \mbox{‘\texttt{i32.mul}’} & \quad\Rightarrow\quad{} & \I32 {.} \MUL \\
   & & | & \mbox{‘\texttt{i32.div\_s}’} & \quad\Rightarrow\quad{} & \I32 {.} {\DIV}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i32.div\_u}’} & \quad\Rightarrow\quad{} & \I32 {.} {\DIV}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i32.rem\_s}’} & \quad\Rightarrow\quad{} & \I32 {.} {\REM}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i32.rem\_u}’} & \quad\Rightarrow\quad{} & \I32 {.} {\REM}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i32.and}’} & \quad\Rightarrow\quad{} & \I32 {.} \AND \\
   & & | & \mbox{‘\texttt{i32.or}’} & \quad\Rightarrow\quad{} & \I32 {.} \OR \\
   & & | & \mbox{‘\texttt{i32.xor}’} & \quad\Rightarrow\quad{} & \I32 {.} \XOR \\
   & & | & \mbox{‘\texttt{i32.shl}’} & \quad\Rightarrow\quad{} & \I32 {.} \SHL \\
   & & | & \mbox{‘\texttt{i32.shr\_s}’} & \quad\Rightarrow\quad{} & \I32 {.} {\SHR}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i32.shr\_u}’} & \quad\Rightarrow\quad{} & \I32 {.} {\SHR}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i32.rotl}’} & \quad\Rightarrow\quad{} & \I32 {.} \ROTL \\
   & & | & \mbox{‘\texttt{i32.rotr}’} & \quad\Rightarrow\quad{} & \I32 {.} \ROTR \\
   \end{array}

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tplaininstr}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{i64.eqz}’} & \quad\Rightarrow\quad{} & \I64 {.} \EQZ \\
   & & | & \mbox{‘\texttt{i64.eq}’} & \quad\Rightarrow\quad{} & \I64 {.} \EQ \\
   & & | & \mbox{‘\texttt{i64.ne}’} & \quad\Rightarrow\quad{} & \I64 {.} \NE \\
   & & | & \mbox{‘\texttt{i64.lt\_s}’} & \quad\Rightarrow\quad{} & \I64 {.} {\LT}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i64.lt\_u}’} & \quad\Rightarrow\quad{} & \I64 {.} {\LT}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i64.gt\_s}’} & \quad\Rightarrow\quad{} & \I64 {.} {\GT}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i64.gt\_u}’} & \quad\Rightarrow\quad{} & \I64 {.} {\GT}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i64.le\_s}’} & \quad\Rightarrow\quad{} & \I64 {.} {\LE}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i64.le\_u}’} & \quad\Rightarrow\quad{} & \I64 {.} {\LE}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i64.ge\_s}’} & \quad\Rightarrow\quad{} & \I64 {.} {\GE}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i64.ge\_u}’} & \quad\Rightarrow\quad{} & \I64 {.} {\GE}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i64.clz}’} & \quad\Rightarrow\quad{} & \I64 {.} \CLZ \\
   & & | & \mbox{‘\texttt{i64.ctz}’} & \quad\Rightarrow\quad{} & \I64 {.} \CTZ \\
   & & | & \mbox{‘\texttt{i64.popcnt}’} & \quad\Rightarrow\quad{} & \I64 {.} \POPCNT \\
   & & | & \mbox{‘\texttt{i64.extend8\_s}’} & \quad\Rightarrow\quad{} & \I64 {.} {\EXTEND}{\mathsf{{\scriptstyle 8}}}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i64.extend16\_s}’} & \quad\Rightarrow\quad{} & \I64 {.} {\EXTEND}{\mathsf{{\scriptstyle 16}}}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i64.extend32\_s}’} & \quad\Rightarrow\quad{} & \I64 {.} {\EXTEND}{\mathsf{{\scriptstyle 32}}}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i64.add}’} & \quad\Rightarrow\quad{} & \I64 {.} \ADD \\
   & & | & \mbox{‘\texttt{i64.sub}’} & \quad\Rightarrow\quad{} & \I64 {.} \SUB \\
   & & | & \mbox{‘\texttt{i64.mul}’} & \quad\Rightarrow\quad{} & \I64 {.} \MUL \\
   & & | & \mbox{‘\texttt{i64.div\_s}’} & \quad\Rightarrow\quad{} & \I64 {.} {\DIV}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i64.div\_u}’} & \quad\Rightarrow\quad{} & \I64 {.} {\DIV}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i64.rem\_s}’} & \quad\Rightarrow\quad{} & \I64 {.} {\REM}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i64.rem\_u}’} & \quad\Rightarrow\quad{} & \I64 {.} {\REM}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i64.and}’} & \quad\Rightarrow\quad{} & \I64 {.} \AND \\
   & & | & \mbox{‘\texttt{i64.or}’} & \quad\Rightarrow\quad{} & \I64 {.} \OR \\
   & & | & \mbox{‘\texttt{i64.xor}’} & \quad\Rightarrow\quad{} & \I64 {.} \XOR \\
   & & | & \mbox{‘\texttt{i64.shl}’} & \quad\Rightarrow\quad{} & \I64 {.} \SHL \\
   & & | & \mbox{‘\texttt{i64.shr\_s}’} & \quad\Rightarrow\quad{} & \I64 {.} {\SHR}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i64.shr\_u}’} & \quad\Rightarrow\quad{} & \I64 {.} {\SHR}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i64.rotl}’} & \quad\Rightarrow\quad{} & \I64 {.} \ROTL \\
   & & | & \mbox{‘\texttt{i64.rotr}’} & \quad\Rightarrow\quad{} & \I64 {.} \ROTR \\
   \end{array}

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tplaininstr}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{f32.eq}’} & \quad\Rightarrow\quad{} & \F32 {.} \EQ \\
   & & | & \mbox{‘\texttt{f32.ne}’} & \quad\Rightarrow\quad{} & \F32 {.} \NE \\
   & & | & \mbox{‘\texttt{f32.lt}’} & \quad\Rightarrow\quad{} & \F32 {.} \LT \\
   & & | & \mbox{‘\texttt{f32.gt}’} & \quad\Rightarrow\quad{} & \F32 {.} \GT \\
   & & | & \mbox{‘\texttt{f32.le}’} & \quad\Rightarrow\quad{} & \F32 {.} \LE \\
   & & | & \mbox{‘\texttt{f32.ge}’} & \quad\Rightarrow\quad{} & \F32 {.} \GE \\
   & & | & \mbox{‘\texttt{f32.abs}’} & \quad\Rightarrow\quad{} & \F32 {.} \ABS \\
   & & | & \mbox{‘\texttt{f32.neg}’} & \quad\Rightarrow\quad{} & \F32 {.} \NEG \\
   & & | & \mbox{‘\texttt{f32.sqrt}’} & \quad\Rightarrow\quad{} & \F32 {.} \SQRT \\
   & & | & \mbox{‘\texttt{f32.ceil}’} & \quad\Rightarrow\quad{} & \F32 {.} \CEIL \\
   & & | & \mbox{‘\texttt{f32.floor}’} & \quad\Rightarrow\quad{} & \F32 {.} \FLOOR \\
   & & | & \mbox{‘\texttt{f32.trunc}’} & \quad\Rightarrow\quad{} & \F32 {.} \TRUNC \\
   & & | & \mbox{‘\texttt{f32.nearest}’} & \quad\Rightarrow\quad{} & \F32 {.} \NEAREST \\
   & & | & \mbox{‘\texttt{f32.add}’} & \quad\Rightarrow\quad{} & \F32 {.} \ADD \\
   & & | & \mbox{‘\texttt{f32.sub}’} & \quad\Rightarrow\quad{} & \F32 {.} \SUB \\
   & & | & \mbox{‘\texttt{f32.mul}’} & \quad\Rightarrow\quad{} & \F32 {.} \MUL \\
   & & | & \mbox{‘\texttt{f32.div}’} & \quad\Rightarrow\quad{} & \F32 {.} \DIV \\
   & & | & \mbox{‘\texttt{f32.min}’} & \quad\Rightarrow\quad{} & \F32 {.} \FMIN \\
   & & | & \mbox{‘\texttt{f32.max}’} & \quad\Rightarrow\quad{} & \F32 {.} \FMAX \\
   & & | & \mbox{‘\texttt{f32.copysign}’} & \quad\Rightarrow\quad{} & \F32 {.} \COPYSIGN \\
   \end{array}

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tplaininstr}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{f64.eq}’} & \quad\Rightarrow\quad{} & \F64 {.} \EQ \\
   & & | & \mbox{‘\texttt{f64.ne}’} & \quad\Rightarrow\quad{} & \F64 {.} \NE \\
   & & | & \mbox{‘\texttt{f64.lt}’} & \quad\Rightarrow\quad{} & \F64 {.} \LT \\
   & & | & \mbox{‘\texttt{f64.gt}’} & \quad\Rightarrow\quad{} & \F64 {.} \GT \\
   & & | & \mbox{‘\texttt{f64.le}’} & \quad\Rightarrow\quad{} & \F64 {.} \LE \\
   & & | & \mbox{‘\texttt{f64.ge}’} & \quad\Rightarrow\quad{} & \F64 {.} \GE \\
   & & | & \mbox{‘\texttt{f64.abs}’} & \quad\Rightarrow\quad{} & \F64 {.} \ABS \\
   & & | & \mbox{‘\texttt{f64.neg}’} & \quad\Rightarrow\quad{} & \F64 {.} \NEG \\
   & & | & \mbox{‘\texttt{f64.sqrt}’} & \quad\Rightarrow\quad{} & \F64 {.} \SQRT \\
   & & | & \mbox{‘\texttt{f64.ceil}’} & \quad\Rightarrow\quad{} & \F64 {.} \CEIL \\
   & & | & \mbox{‘\texttt{f64.floor}’} & \quad\Rightarrow\quad{} & \F64 {.} \FLOOR \\
   & & | & \mbox{‘\texttt{f64.trunc}’} & \quad\Rightarrow\quad{} & \F64 {.} \TRUNC \\
   & & | & \mbox{‘\texttt{f64.nearest}’} & \quad\Rightarrow\quad{} & \F64 {.} \NEAREST \\
   & & | & \mbox{‘\texttt{f64.add}’} & \quad\Rightarrow\quad{} & \F64 {.} \ADD \\
   & & | & \mbox{‘\texttt{f64.sub}’} & \quad\Rightarrow\quad{} & \F64 {.} \SUB \\
   & & | & \mbox{‘\texttt{f64.mul}’} & \quad\Rightarrow\quad{} & \F64 {.} \MUL \\
   & & | & \mbox{‘\texttt{f64.div}’} & \quad\Rightarrow\quad{} & \F64 {.} \DIV \\
   & & | & \mbox{‘\texttt{f64.min}’} & \quad\Rightarrow\quad{} & \F64 {.} \FMIN \\
   & & | & \mbox{‘\texttt{f64.max}’} & \quad\Rightarrow\quad{} & \F64 {.} \FMAX \\
   & & | & \mbox{‘\texttt{f64.copysign}’} & \quad\Rightarrow\quad{} & \F64 {.} \COPYSIGN \\
   \end{array}

.. _text-cvtop:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tplaininstr}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{i32.wrap\_i64}’} & \quad\Rightarrow\quad{} & \I32 {.} {\WRAP}{\mathsf{\_}}{\I64} \\
   & & | & \mbox{‘\texttt{i32.trunc\_f32\_s}’} & \quad\Rightarrow\quad{} & \I32 {.} {{\TRUNC}{\mathsf{\_}}{\S}}{\mathsf{\_}}{\F32} \\
   & & | & \mbox{‘\texttt{i32.trunc\_f32\_u}’} & \quad\Rightarrow\quad{} & \I32 {.} {{\TRUNC}{\mathsf{\_}}{\U}}{\mathsf{\_}}{\F32} \\
   & & | & \mbox{‘\texttt{i32.trunc\_f64\_s}’} & \quad\Rightarrow\quad{} & \I32 {.} {{\TRUNC}{\mathsf{\_}}{\S}}{\mathsf{\_}}{\F64} \\
   & & | & \mbox{‘\texttt{i32.trunc\_f64\_u}’} & \quad\Rightarrow\quad{} & \I32 {.} {{\TRUNC}{\mathsf{\_}}{\U}}{\mathsf{\_}}{\F64} \\
   & & | & \mbox{‘\texttt{i32.trunc\_sat\_f32\_s}’} & \quad\Rightarrow\quad{} & \I32 {.} {{\TRUNCSAT}{\mathsf{\_}}{\S}}{\mathsf{\_}}{\F32} \\
   & & | & \mbox{‘\texttt{i32.trunc\_sat\_f32\_u}’} & \quad\Rightarrow\quad{} & \I32 {.} {{\TRUNCSAT}{\mathsf{\_}}{\U}}{\mathsf{\_}}{\F32} \\
   & & | & \mbox{‘\texttt{i32.trunc\_sat\_f64\_s}’} & \quad\Rightarrow\quad{} & \I32 {.} {{\TRUNCSAT}{\mathsf{\_}}{\S}}{\mathsf{\_}}{\F64} \\
   & & | & \mbox{‘\texttt{i32.trunc\_sat\_f64\_u}’} & \quad\Rightarrow\quad{} & \I32 {.} {{\TRUNCSAT}{\mathsf{\_}}{\U}}{\mathsf{\_}}{\F64} \\
   & & | & \mbox{‘\texttt{i64.extend\_i32\_s}’} & \quad\Rightarrow\quad{} & \I64 {.} {{\EXTEND}{\mathsf{\_}}{\S}}{\mathsf{\_}}{\I32} \\
   & & | & \mbox{‘\texttt{i64.extend\_i32\_u}’} & \quad\Rightarrow\quad{} & \I64 {.} {{\EXTEND}{\mathsf{\_}}{\U}}{\mathsf{\_}}{\I32} \\
   & & | & \mbox{‘\texttt{i64.trunc\_f32\_s}’} & \quad\Rightarrow\quad{} & \I64 {.} {{\TRUNC}{\mathsf{\_}}{\S}}{\mathsf{\_}}{\F32} \\
   & & | & \mbox{‘\texttt{i64.trunc\_f32\_u}’} & \quad\Rightarrow\quad{} & \I64 {.} {{\TRUNC}{\mathsf{\_}}{\U}}{\mathsf{\_}}{\F32} \\
   & & | & \mbox{‘\texttt{i64.trunc\_f64\_s}’} & \quad\Rightarrow\quad{} & \I64 {.} {{\TRUNC}{\mathsf{\_}}{\S}}{\mathsf{\_}}{\F64} \\
   & & | & \mbox{‘\texttt{i64.trunc\_f64\_u}’} & \quad\Rightarrow\quad{} & \I64 {.} {{\TRUNC}{\mathsf{\_}}{\U}}{\mathsf{\_}}{\F64} \\
   & & | & \mbox{‘\texttt{i64.trunc\_sat\_f32\_s}’} & \quad\Rightarrow\quad{} & \I64 {.} {{\TRUNCSAT}{\mathsf{\_}}{\S}}{\mathsf{\_}}{\F32} \\
   & & | & \mbox{‘\texttt{i64.trunc\_sat\_f32\_u}’} & \quad\Rightarrow\quad{} & \I64 {.} {{\TRUNCSAT}{\mathsf{\_}}{\U}}{\mathsf{\_}}{\F32} \\
   & & | & \mbox{‘\texttt{i64.trunc\_sat\_f64\_s}’} & \quad\Rightarrow\quad{} & \I64 {.} {{\TRUNCSAT}{\mathsf{\_}}{\S}}{\mathsf{\_}}{\F64} \\
   & & | & \mbox{‘\texttt{i64.trunc\_sat\_f64\_u}’} & \quad\Rightarrow\quad{} & \I64 {.} {{\TRUNCSAT}{\mathsf{\_}}{\U}}{\mathsf{\_}}{\F64} \\
   & & | & \mbox{‘\texttt{f32.demote\_f64}’} & \quad\Rightarrow\quad{} & \F32 {.} {\DEMOTE}{\mathsf{\_}}{\F64} \\
   & & | & \mbox{‘\texttt{f32.convert\_i32\_s}’} & \quad\Rightarrow\quad{} & \F32 {.} {{\CONVERT}{\mathsf{\_}}{\S}}{\mathsf{\_}}{\I32} \\
   & & | & \mbox{‘\texttt{f32.convert\_i32\_u}’} & \quad\Rightarrow\quad{} & \F32 {.} {{\CONVERT}{\mathsf{\_}}{\U}}{\mathsf{\_}}{\I32} \\
   & & | & \mbox{‘\texttt{f32.convert\_i64\_s}’} & \quad\Rightarrow\quad{} & \F32 {.} {{\CONVERT}{\mathsf{\_}}{\S}}{\mathsf{\_}}{\I64} \\
   & & | & \mbox{‘\texttt{f32.convert\_i64\_u}’} & \quad\Rightarrow\quad{} & \F32 {.} {{\CONVERT}{\mathsf{\_}}{\U}}{\mathsf{\_}}{\I64} \\
   & & | & \mbox{‘\texttt{f64.promote\_f32}’} & \quad\Rightarrow\quad{} & \F64 {.} {\PROMOTE}{\mathsf{\_}}{\F32} \\
   & & | & \mbox{‘\texttt{f64.convert\_i32\_s}’} & \quad\Rightarrow\quad{} & \F64 {.} {{\CONVERT}{\mathsf{\_}}{\S}}{\mathsf{\_}}{\I32} \\
   & & | & \mbox{‘\texttt{f64.convert\_i32\_u}’} & \quad\Rightarrow\quad{} & \F64 {.} {{\CONVERT}{\mathsf{\_}}{\U}}{\mathsf{\_}}{\I32} \\
   & & | & \mbox{‘\texttt{f64.convert\_i64\_s}’} & \quad\Rightarrow\quad{} & \F64 {.} {{\CONVERT}{\mathsf{\_}}{\S}}{\mathsf{\_}}{\I64} \\
   & & | & \mbox{‘\texttt{f64.convert\_i64\_u}’} & \quad\Rightarrow\quad{} & \F64 {.} {{\CONVERT}{\mathsf{\_}}{\U}}{\mathsf{\_}}{\I64} \\
   & & | & \mbox{‘\texttt{i32.reinterpret\_f32}’} & \quad\Rightarrow\quad{} & \I32 {.} {\REINTERPRET}{\mathsf{\_}}{\F32} \\
   & & | & \mbox{‘\texttt{i64.reinterpret\_f64}’} & \quad\Rightarrow\quad{} & \I64 {.} {\REINTERPRET}{\mathsf{\_}}{\F64} \\
   & & | & \mbox{‘\texttt{f32.reinterpret\_i32}’} & \quad\Rightarrow\quad{} & \F32 {.} {\REINTERPRET}{\mathsf{\_}}{\I32} \\
   & & | & \mbox{‘\texttt{f64.reinterpret\_i64}’} & \quad\Rightarrow\quad{} & \F64 {.} {\REINTERPRET}{\mathsf{\_}}{\I64} \\
   \end{array}


.. index:: vector instruction
   pair: text format; instruction
.. _text-instr-vec:

Vector Instructions
~~~~~~~~~~~~~~~~~~~

Vector constant instructions have a mandatory :ref:`shape <syntax-shape>` descriptor, which determines how the following values are parsed.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tplaininstr}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{v128.const}’}~~\mbox{‘\texttt{i8x16}’}~~{c^\ast}{:}{{\Ti8}^{16}} & \quad\Rightarrow\quad{} & \V128{.}\CONST~{{{{\bytes}}_{{\INX}{128}}^{{-1}}}}{({\bigcat}\, {{{\bytes}}_{{\INX}{8}}(c)^\ast})} \\
   & & | & \mbox{‘\texttt{v128.const}’}~~\mbox{‘\texttt{i16x8}’}~~{c^\ast}{:}{{\Ti16}^{8}} & \quad\Rightarrow\quad{} & \V128{.}\CONST~{{{{\bytes}}_{{\INX}{128}}^{{-1}}}}{({\bigcat}\, {{{\bytes}}_{{\INX}{16}}(c)^\ast})} \\
   & & | & \mbox{‘\texttt{v128.const}’}~~\mbox{‘\texttt{i32x4}’}~~{c^\ast}{:}{{\Ti32}^{4}} & \quad\Rightarrow\quad{} & \V128{.}\CONST~{{{{\bytes}}_{{\INX}{128}}^{{-1}}}}{({\bigcat}\, {{{\bytes}}_{{\INX}{32}}(c)^\ast})} \\
   & & | & \mbox{‘\texttt{v128.const}’}~~\mbox{‘\texttt{i64x2}’}~~{c^\ast}{:}{{\Ti64}^{2}} & \quad\Rightarrow\quad{} & \V128{.}\CONST~{{{{\bytes}}_{{\INX}{128}}^{{-1}}}}{({\bigcat}\, {{{\bytes}}_{{\INX}{64}}(c)^\ast})} \\
   & & | & \mbox{‘\texttt{v128.const}’}~~\mbox{‘\texttt{f32x4}’}~~{c^\ast}{:}{{\Tf32}^{4}} & \quad\Rightarrow\quad{} & \V128{.}\CONST~{{{{\bytes}}_{{\INX}{128}}^{{-1}}}}{({\bigcat}\, {{{\bytes}}_{{\FNX}{32}}(c)^\ast})} \\
   & & | & \mbox{‘\texttt{v128.const}’}~~\mbox{‘\texttt{f64x2}’}~~{c^\ast}{:}{{\Tf64}^{2}} & \quad\Rightarrow\quad{} & \V128{.}\CONST~{{{{\bytes}}_{{\INX}{128}}^{{-1}}}}{({\bigcat}\, {{{\bytes}}_{{\FNX}{64}}(c)^\ast})} \\
   \end{array}

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tplaininstr}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{i8x16.shuffle}’}~~{i^\ast}{:}{{\Tlaneidx}^{16}} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}{.}\VSHUFFLE~{i^\ast} \\
   & & | & \mbox{‘\texttt{i8x16.swizzle}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} \VSWIZZLE \\
   & & | & \mbox{‘\texttt{i8x16.relaxed\_swizzle}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} \VRELAXEDSWIZZLE \\
   & & | & \mbox{‘\texttt{i8x16.splat}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}{.}\VSPLAT \\
   & & | & \mbox{‘\texttt{i16x8.splat}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}{.}\VSPLAT \\
   & & | & \mbox{‘\texttt{i32x4.splat}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}{.}\VSPLAT \\
   & & | & \mbox{‘\texttt{i64x2.splat}’} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}}{.}\VSPLAT \\
   & & | & \mbox{‘\texttt{f32x4.splat}’} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}}{.}\VSPLAT \\
   & & | & \mbox{‘\texttt{f64x2.splat}’} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}}{.}\VSPLAT \\
   & & | & \mbox{‘\texttt{i8x16.extract\_lane\_s}’}~~i{:}{\Tlaneidx} & \quad\Rightarrow\quad{} & {{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}{.}\VEXTRACTLANE}{\mathsf{\_}}{\S}~i \\
   & & | & \mbox{‘\texttt{i8x16.extract\_lane\_u}’}~~i{:}{\Tlaneidx} & \quad\Rightarrow\quad{} & {{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}{.}\VEXTRACTLANE}{\mathsf{\_}}{\U}~i \\
   & & | & \mbox{‘\texttt{i16x8.extract\_lane\_s}’}~~i{:}{\Tlaneidx} & \quad\Rightarrow\quad{} & {{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}{.}\VEXTRACTLANE}{\mathsf{\_}}{\S}~i \\
   & & | & \mbox{‘\texttt{i16x8.extract\_lane\_u}’}~~i{:}{\Tlaneidx} & \quad\Rightarrow\quad{} & {{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}{.}\VEXTRACTLANE}{\mathsf{\_}}{\U}~i \\
   & & | & \mbox{‘\texttt{i32x4.extract\_lane}’}~~i{:}{\Tlaneidx} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}{.}\VEXTRACTLANE~i \\
   & & | & \mbox{‘\texttt{i64x2.extract\_lane}’}~~i{:}{\Tlaneidx} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}}{.}\VEXTRACTLANE~i \\
   & & | & \mbox{‘\texttt{f32x4.extract\_lane}’}~~i{:}{\Tlaneidx} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}}{.}\VEXTRACTLANE~i \\
   & & | & \mbox{‘\texttt{f64x2.extract\_lane}’}~~i{:}{\Tlaneidx} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}}{.}\VEXTRACTLANE~i \\
   & & | & \mbox{‘\texttt{i8x16.replace\_lane}’}~~i{:}{\Tlaneidx} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}{.}\VREPLACELANE~i \\
   & & | & \mbox{‘\texttt{i16x8.replace\_lane}’}~~i{:}{\Tlaneidx} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}{.}\VREPLACELANE~i \\
   & & | & \mbox{‘\texttt{i32x4.replace\_lane}’}~~i{:}{\Tlaneidx} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}{.}\VREPLACELANE~i \\
   & & | & \mbox{‘\texttt{i64x2.replace\_lane}’}~~i{:}{\Tlaneidx} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}}{.}\VREPLACELANE~i \\
   & & | & \mbox{‘\texttt{f32x4.replace\_lane}’}~~i{:}{\Tlaneidx} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}}{.}\VREPLACELANE~i \\
   & & | & \mbox{‘\texttt{f64x2.replace\_lane}’}~~i{:}{\Tlaneidx} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}}{.}\VREPLACELANE~i \\
   \end{array}


.. _text-vvunop:
.. _text-vvbinop:
.. _text-vvternop:
.. _text-vitestop:
.. _text-virelop:
.. _text-vfrelop:
.. _text-viunop:
.. _text-vfunop:
.. _text-vibinop:
.. _text-vfbinop:
.. _text-vishiftop:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tplaininstr}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{v128.any\_true}’} & \quad\Rightarrow\quad{} & \V128 {.} \VANYTRUE \\
   & & | & \mbox{‘\texttt{v128.not}’} & \quad\Rightarrow\quad{} & \V128 {.} \VNOT \\
   & & | & \mbox{‘\texttt{v128.and}’} & \quad\Rightarrow\quad{} & \V128 {.} \VAND \\
   & & | & \mbox{‘\texttt{v128.andnot}’} & \quad\Rightarrow\quad{} & \V128 {.} \VANDNOT \\
   & & | & \mbox{‘\texttt{v128.or}’} & \quad\Rightarrow\quad{} & \V128 {.} \VOR \\
   & & | & \mbox{‘\texttt{v128.xor}’} & \quad\Rightarrow\quad{} & \V128 {.} \VXOR \\
   & & | & \mbox{‘\texttt{v128.bitselect}’} & \quad\Rightarrow\quad{} & \V128 {.} \VBITSELECT \\
   \end{array}

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tplaininstr}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{i8x16.all\_true}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} \VALLTRUE \\
   & & | & \mbox{‘\texttt{i8x16.eq}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} \VEQ \\
   & & | & \mbox{‘\texttt{i8x16.ne}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} \VNE \\
   & & | & \mbox{‘\texttt{i8x16.lt\_s}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VLT}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i8x16.lt\_u}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VLT}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i8x16.gt\_s}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VGT}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i8x16.gt\_u}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VGT}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i8x16.le\_s}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VLE}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i8x16.le\_u}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VLE}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i8x16.ge\_s}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VGE}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i8x16.ge\_u}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VGE}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i8x16.abs}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} \VABS \\
   & & | & \mbox{‘\texttt{i8x16.neg}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} \VNEG \\
   & & | & \mbox{‘\texttt{i8x16.popcnt}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} \VPOPCNT \\
   & & | & \mbox{‘\texttt{i8x16.add}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} \VADD \\
   & & | & \mbox{‘\texttt{i8x16.add\_sat\_s}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VADDSAT}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i8x16.add\_sat\_u}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VADDSAT}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i8x16.sub}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} \VSUB \\
   & & | & \mbox{‘\texttt{i8x16.sub\_sat\_s}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VSUBSAT}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i8x16.sub\_sat\_u}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VSUBSAT}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i8x16.min\_s}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VMIN}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i8x16.min\_u}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VMIN}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i8x16.max\_s}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VMAX}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i8x16.max\_u}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VMAX}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i8x16.avgr\_u}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VAVGR}{\mathsf{\_}}{\VU} \\
   & & | & \mbox{‘\texttt{i8x16.relaxed\_laneselect}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} \VRELAXEDLANESELECT \\
   & & | & \mbox{‘\texttt{i8x16.shl}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} \VSHL \\
   & & | & \mbox{‘\texttt{i8x16.shr\_s}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VSHR}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i8x16.shr\_u}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VSHR}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i8x16.bitmask}’} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}{.}\VBITMASK \\
   & & | & \mbox{‘\texttt{i8x16.narrow\_i16x8\_s}’} & \quad\Rightarrow\quad{} & {{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}{.}\VNARROW}{\mathsf{\_}}{{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i8x16.narrow\_i16x8\_u}’} & \quad\Rightarrow\quad{} & {{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}{.}\VNARROW}{\mathsf{\_}}{{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}}{\mathsf{\_}}{\U} \\
   \end{array}

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tplaininstr}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{i16x8.all\_true}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} \VALLTRUE \\
   & & | & \mbox{‘\texttt{i16x8.eq}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} \VEQ \\
   & & | & \mbox{‘\texttt{i16x8.ne}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} \VNE \\
   & & | & \mbox{‘\texttt{i16x8.lt\_s}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VLT}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i16x8.lt\_u}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VLT}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i16x8.gt\_s}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VGT}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i16x8.gt\_u}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VGT}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i16x8.le\_s}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VLE}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i16x8.le\_u}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VLE}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i16x8.ge\_s}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VGE}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i16x8.ge\_u}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VGE}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i16x8.abs}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} \VABS \\
   & & | & \mbox{‘\texttt{i16x8.neg}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} \VNEG \\
   & & | & \mbox{‘\texttt{i16x8.add}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} \VADD \\
   & & | & \mbox{‘\texttt{i16x8.add\_sat\_s}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VADDSAT}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i16x8.add\_sat\_u}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VADDSAT}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i16x8.sub}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} \VSUB \\
   & & | & \mbox{‘\texttt{i16x8.sub\_sat\_s}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VSUBSAT}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i16x8.sub\_sat\_u}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VSUBSAT}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i16x8.mul}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} \VMUL \\
   & & | & \mbox{‘\texttt{i16x8.min\_s}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VMIN}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i16x8.min\_u}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VMIN}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i16x8.max\_s}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VMAX}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i16x8.max\_u}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VMAX}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i16x8.avgr\_u}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VAVGR}{\mathsf{\_}}{\VU} \\
   & & | & \mbox{‘\texttt{i16x8.q15mulr\_sat\_s}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VQ15MULRSAT}{\mathsf{\_}}{\VS} \\
   & & | & \mbox{‘\texttt{i16x8.relaxed\_q15mulr\_s}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VRELAXEDQ15MULR}{\mathsf{\_}}{\VS} \\
   & & | & \mbox{‘\texttt{i16x8.relaxed\_laneselect}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} \VRELAXEDLANESELECT \\
   & & | & \mbox{‘\texttt{i16x8.shl}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} \VSHL \\
   & & | & \mbox{‘\texttt{i16x8.shr\_s}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VSHR}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i16x8.shr\_u}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VSHR}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i16x8.bitmask}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}{.}\VBITMASK \\
   & & | & \mbox{‘\texttt{i16x8.narrow\_i32x4\_s}’} & \quad\Rightarrow\quad{} & {{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}{.}\VNARROW}{\mathsf{\_}}{{\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i16x8.narrow\_i32x4\_u}’} & \quad\Rightarrow\quad{} & {{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}{.}\VNARROW}{\mathsf{\_}}{{\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}}{\mathsf{\_}}{\U} \\
   \end{array}

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tplaininstr}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{i32x4.all\_true}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VALLTRUE \\
   & & | & \mbox{‘\texttt{i32x4.eq}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VEQ \\
   & & | & \mbox{‘\texttt{i32x4.ne}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VNE \\
   & & | & \mbox{‘\texttt{i32x4.lt\_s}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {\VLT}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i32x4.lt\_u}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {\VLT}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i32x4.gt\_s}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {\VGT}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i32x4.gt\_u}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {\VGT}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i32x4.le\_s}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {\VLE}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i32x4.le\_u}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {\VLE}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i32x4.ge\_s}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {\VGE}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i32x4.ge\_u}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {\VGE}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i32x4.abs}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VABS \\
   & & | & \mbox{‘\texttt{i32x4.neg}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VNEG \\
   & & | & \mbox{‘\texttt{i32x4.add}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VADD \\
   & & | & \mbox{‘\texttt{i32x4.sub}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VSUB \\
   & & | & \mbox{‘\texttt{i32x4.mul}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VMUL \\
   & & | & \mbox{‘\texttt{i32x4.min\_s}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {\VMIN}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i32x4.min\_u}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {\VMIN}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i32x4.max\_s}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {\VMAX}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i32x4.max\_u}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {\VMAX}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i32x4.relaxed\_laneselect}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VRELAXEDLANESELECT \\
   & & | & \mbox{‘\texttt{i32x4.shl}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VSHL \\
   & & | & \mbox{‘\texttt{i32x4.shr\_s}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {\VSHR}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i32x4.shr\_u}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {\VSHR}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i32x4.bitmask}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}{.}\VBITMASK \\
   \end{array}

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tplaininstr}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{i64x2.all\_true}’} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VALLTRUE \\
   & & | & \mbox{‘\texttt{i64x2.eq}’} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VEQ \\
   & & | & \mbox{‘\texttt{i64x2.ne}’} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VNE \\
   & & | & \mbox{‘\texttt{i64x2.lt\_s}’} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {\VLT}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i64x2.gt\_s}’} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {\VGT}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i64x2.le\_s}’} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {\VLE}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i64x2.ge\_s}’} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {\VGE}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i64x2.abs}’} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VABS \\
   & & | & \mbox{‘\texttt{i64x2.neg}’} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VNEG \\
   & & | & \mbox{‘\texttt{i64x2.add}’} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VADD \\
   & & | & \mbox{‘\texttt{i64x2.sub}’} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VSUB \\
   & & | & \mbox{‘\texttt{i64x2.mul}’} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VMUL \\
   & & | & \mbox{‘\texttt{i64x2.relaxed\_laneselect}’} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VRELAXEDLANESELECT \\
   & & | & \mbox{‘\texttt{i64x2.shl}’} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VSHL \\
   & & | & \mbox{‘\texttt{i64x2.shr\_s}’} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {\VSHR}{\mathsf{\_}}{\S} \\
   & & | & \mbox{‘\texttt{i64x2.shr\_u}’} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {\VSHR}{\mathsf{\_}}{\U} \\
   & & | & \mbox{‘\texttt{i64x2.bitmask}’} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}}{.}\VBITMASK \\
   \end{array}

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tplaininstr}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{f32x4.eq}’} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VEQ \\
   & & | & \mbox{‘\texttt{f32x4.ne}’} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VNE \\
   & & | & \mbox{‘\texttt{f32x4.lt}’} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VLT \\
   & & | & \mbox{‘\texttt{f32x4.gt}’} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VGT \\
   & & | & \mbox{‘\texttt{f32x4.le}’} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VLE \\
   & & | & \mbox{‘\texttt{f32x4.ge}’} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VGE \\
   & & | & \mbox{‘\texttt{f32x4.abs}’} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VABS \\
   & & | & \mbox{‘\texttt{f32x4.neg}’} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VNEG \\
   & & | & \mbox{‘\texttt{f32x4.sqrt}’} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VSQRT \\
   & & | & \mbox{‘\texttt{f32x4.ceil}’} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VCEIL \\
   & & | & \mbox{‘\texttt{f32x4.floor}’} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VFLOOR \\
   & & | & \mbox{‘\texttt{f32x4.trunc}’} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VTRUNC \\
   & & | & \mbox{‘\texttt{f32x4.nearest}’} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VNEAREST \\
   & & | & \mbox{‘\texttt{f32x4.add}’} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VADD \\
   & & | & \mbox{‘\texttt{f32x4.sub}’} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VSUB \\
   & & | & \mbox{‘\texttt{f32x4.mul}’} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VMUL \\
   & & | & \mbox{‘\texttt{f32x4.div}’} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VDIV \\
   & & | & \mbox{‘\texttt{f32x4.min}’} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VMIN \\
   & & | & \mbox{‘\texttt{f32x4.max}’} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VMAX \\
   & & | & \mbox{‘\texttt{f32x4.pmin}’} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VPMIN \\
   & & | & \mbox{‘\texttt{f32x4.pmax}’} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VPMAX \\
   & & | & \mbox{‘\texttt{f32x4.relaxed\_min}’} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VRELAXEDMIN \\
   & & | & \mbox{‘\texttt{f32x4.relaxed\_max}’} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VRELAXEDMAX \\
   & & | & \mbox{‘\texttt{f32x4.relaxed\_madd}’} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VRELAXEDMADD \\
   & & | & \mbox{‘\texttt{f32x4.relaxed\_nmadd}’} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VRELAXEDNMADD \\
   \end{array}

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tplaininstr}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{f64x2.eq}’} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VEQ \\
   & & | & \mbox{‘\texttt{f64x2.ne}’} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VNE \\
   & & | & \mbox{‘\texttt{f64x2.lt}’} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VLT \\
   & & | & \mbox{‘\texttt{f64x2.gt}’} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VGT \\
   & & | & \mbox{‘\texttt{f64x2.le}’} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VLE \\
   & & | & \mbox{‘\texttt{f64x2.ge}’} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VGE \\
   & & | & \mbox{‘\texttt{f64x2.abs}’} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VABS \\
   & & | & \mbox{‘\texttt{f64x2.neg}’} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VNEG \\
   & & | & \mbox{‘\texttt{f64x2.sqrt}’} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VSQRT \\
   & & | & \mbox{‘\texttt{f64x2.ceil}’} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VCEIL \\
   & & | & \mbox{‘\texttt{f64x2.floor}’} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VFLOOR \\
   & & | & \mbox{‘\texttt{f64x2.trunc}’} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VTRUNC \\
   & & | & \mbox{‘\texttt{f64x2.nearest}’} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VNEAREST \\
   & & | & \mbox{‘\texttt{f64x2.add}’} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VADD \\
   & & | & \mbox{‘\texttt{f64x2.sub}’} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VSUB \\
   & & | & \mbox{‘\texttt{f64x2.mul}’} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VMUL \\
   & & | & \mbox{‘\texttt{f64x2.div}’} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VDIV \\
   & & | & \mbox{‘\texttt{f64x2.min}’} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VMIN \\
   & & | & \mbox{‘\texttt{f64x2.max}’} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VMAX \\
   & & | & \mbox{‘\texttt{f64x2.pmin}’} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VPMIN \\
   & & | & \mbox{‘\texttt{f64x2.pmax}’} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VPMAX \\
   & & | & \mbox{‘\texttt{f64x2.relaxed\_min}’} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VRELAXEDMIN \\
   & & | & \mbox{‘\texttt{f64x2.relaxed\_max}’} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VRELAXEDMAX \\
   & & | & \mbox{‘\texttt{f64x2.relaxed\_madd}’} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VRELAXEDMADD \\
   & & | & \mbox{‘\texttt{f64x2.relaxed\_nmadd}’} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VRELAXEDNMADD \\
   \end{array}

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tplaininstr}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{i16x8.extend\_low\_i8x16\_s}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {{\VEXTEND}{\mathsf{\_}}{\LOW}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}} \\
   & & | & \mbox{‘\texttt{i16x8.extend\_low\_i8x16\_u}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {{\VEXTEND}{\mathsf{\_}}{\LOW}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}} \\
   & & | & \mbox{‘\texttt{i16x8.extend\_high\_i8x16\_s}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {{\VEXTEND}{\mathsf{\_}}{\HIGH}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}} \\
   & & | & \mbox{‘\texttt{i16x8.extend\_high\_i8x16\_u}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {{\VEXTEND}{\mathsf{\_}}{\HIGH}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}} \\
   & & | & \mbox{‘\texttt{i32x4.extend\_low\_i16x8\_s}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VEXTEND}{\mathsf{\_}}{\LOW}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}} \\
   & & | & \mbox{‘\texttt{i32x4.extend\_low\_i16x8\_u}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VEXTEND}{\mathsf{\_}}{\LOW}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}} \\
   & & | & \mbox{‘\texttt{i32x4.extend\_high\_i16x8\_s}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VEXTEND}{\mathsf{\_}}{\HIGH}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}} \\
   & & | & \mbox{‘\texttt{i32x4.extend\_high\_i16x8\_u}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VEXTEND}{\mathsf{\_}}{\HIGH}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}} \\
   & & | & \mbox{‘\texttt{i32x4.trunc\_sat\_f32x4\_s}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VTRUNCSAT}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mbox{‘\texttt{i32x4.trunc\_sat\_f32x4\_u}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VTRUNCSAT}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mbox{‘\texttt{i32x4.trunc\_sat\_f64x2\_s\_zero}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VTRUNCSAT}{\mathsf{\_}}{\S}{\mathsf{\_}}{\ZERO}}{\mathsf{\_}}{{\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}}} \\
   & & | & \mbox{‘\texttt{i32x4.trunc\_sat\_f64x2\_u\_zero}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VTRUNCSAT}{\mathsf{\_}}{\U}{\mathsf{\_}}{\ZERO}}{\mathsf{\_}}{{\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}}} \\
   & & | & \mbox{‘\texttt{i32x4.relaxed\_trunc\_f32x4\_s}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VRELAXEDTRUNC}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mbox{‘\texttt{i32x4.relaxed\_trunc\_f32x4\_u}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VRELAXEDTRUNC}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mbox{‘\texttt{i32x4.relaxed\_trunc\_f64x2\_s\_zero}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VRELAXEDTRUNC}{\mathsf{\_}}{\S}{\mathsf{\_}}{\ZERO}}{\mathsf{\_}}{{\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}}} \\
   & & | & \mbox{‘\texttt{i32x4.relaxed\_trunc\_f64x2\_u\_zero}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VRELAXEDTRUNC}{\mathsf{\_}}{\U}{\mathsf{\_}}{\ZERO}}{\mathsf{\_}}{{\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}}} \\
   & & | & \mbox{‘\texttt{i64x2.extend\_low\_i32x4\_s}’} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {{\VEXTEND}{\mathsf{\_}}{\LOW}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mbox{‘\texttt{i64x2.extend\_low\_i32x4\_u}’} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {{\VEXTEND}{\mathsf{\_}}{\LOW}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mbox{‘\texttt{i64x2.extend\_high\_i32x4\_s}’} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {{\VEXTEND}{\mathsf{\_}}{\HIGH}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mbox{‘\texttt{i64x2.extend\_high\_i32x4\_u}’} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {{\VEXTEND}{\mathsf{\_}}{\HIGH}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mbox{‘\texttt{f32x4.demote\_f64x2\_zero}’} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VDEMOTE}{\mathsf{\_}}{\VZERO}}{\mathsf{\_}}{{\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}}} \\
   & & | & \mbox{‘\texttt{f32x4.convert\_i32x4\_s}’} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VCONVERT}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mbox{‘\texttt{f32x4.convert\_i32x4\_u}’} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VCONVERT}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mbox{‘\texttt{f64x2.promote\_low\_f32x4}’} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {{\VPROMOTE}{\mathsf{\_}}{\VLOW}}{\mathsf{\_}}{{\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mbox{‘\texttt{f64x2.convert\_low\_i32x4\_s}’} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {{\VCONVERT}{\mathsf{\_}}{\LOW}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mbox{‘\texttt{f64x2.convert\_low\_i32x4\_u}’} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {{\VCONVERT}{\mathsf{\_}}{\LOW}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   \end{array}

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tplaininstr}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{i16x8.extadd\_pairwise\_i8x16\_s}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {{\VEXTADDPAIRWISE}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}} \\
   & & | & \mbox{‘\texttt{i16x8.extadd\_pairwise\_i8x16\_u}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {{\VEXTADDPAIRWISE}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}} \\
   & & | & \mbox{‘\texttt{i16x8.extmul\_low\_i8x16\_s}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {{\VEXTMUL}{\mathsf{\_}}{\LOW}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}} \\
   & & | & \mbox{‘\texttt{i16x8.extmul\_low\_i8x16\_u}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {{\VEXTMUL}{\mathsf{\_}}{\LOW}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}} \\
   & & | & \mbox{‘\texttt{i16x8.extmul\_high\_i8x16\_s}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {{\VEXTMUL}{\mathsf{\_}}{\HIGH}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}} \\
   & & | & \mbox{‘\texttt{i16x8.extmul\_high\_i8x16\_u}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {{\VEXTMUL}{\mathsf{\_}}{\HIGH}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}} \\
   & & | & \mbox{‘\texttt{i16x8.relaxed\_dot\_i8x16\_i7x16\_s}’} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {{\VRELAXEDDOT}{\mathsf{\_}}{\VS}}{\mathsf{\_}}{{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}} \\
   & & | & \mbox{‘\texttt{i32x4.extadd\_pairwise\_i16x8\_s}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VEXTADDPAIRWISE}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}} \\
   & & | & \mbox{‘\texttt{i32x4.extadd\_pairwise\_i16x8\_u}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VEXTADDPAIRWISE}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}} \\
   & & | & \mbox{‘\texttt{i32x4.extmul\_low\_i16x8\_s}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VEXTMUL}{\mathsf{\_}}{\LOW}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}} \\
   & & | & \mbox{‘\texttt{i32x4.extmul\_low\_i16x8\_u}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VEXTMUL}{\mathsf{\_}}{\LOW}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}} \\
   & & | & \mbox{‘\texttt{i32x4.extmul\_high\_i16x8\_s}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VEXTMUL}{\mathsf{\_}}{\HIGH}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}} \\
   & & | & \mbox{‘\texttt{i32x4.extmul\_high\_i16x8\_u}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VEXTMUL}{\mathsf{\_}}{\HIGH}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}} \\
   & & | & \mbox{‘\texttt{i32x4.dot\_i16x8\_s}’} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VDOT}{\mathsf{\_}}{\VS}}{\mathsf{\_}}{{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}} \\
   & & | & \mbox{‘\texttt{i32x4.relaxed\_dot\_i8x16\_i7x16\_add\_s}’} ~\Rightarrow~ {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VRELAXEDDOTADD}{\mathsf{\_}}{\VS}}{\mathsf{\_}}{{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}} ~~|~~ \mbox{‘\texttt{i64x2.extmul\_low\_i32x4\_s}’} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {{\VEXTMUL}{\mathsf{\_}}{\LOW}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mbox{‘\texttt{i64x2.extmul\_low\_i32x4\_u}’} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {{\VEXTMUL}{\mathsf{\_}}{\LOW}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mbox{‘\texttt{i64x2.extmul\_high\_i32x4\_s}’} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {{\VEXTMUL}{\mathsf{\_}}{\HIGH}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mbox{‘\texttt{i64x2.extmul\_high\_i32x4\_u}’} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {{\VEXTMUL}{\mathsf{\_}}{\HIGH}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   \end{array}


.. index:: ! folded instruction, S-expression
.. _text-foldedinstr:

Folded Instructions
~~~~~~~~~~~~~~~~~~~

Instructions can be written as S-expressions by grouping them into *folded* form. In that notation, an instruction is wrapped in parentheses and optionally includes nested folded instructions to indicate its operands.

In the case of :ref:`block instructions <text-instr-block>`, the folded form omits the :math:`\mbox{‘\texttt{end}’}` delimiter.
For :math:`\mathsf{if}` instructions, both branches have to be wrapped into nested S-expressions, headed by the keywords :math:`\mbox{‘\texttt{then}’}` and :math:`\mbox{‘\texttt{else}’}`.

The set of all phrases defined by the following abbreviations recursively forms the auxiliary syntactic class :math:`{\mathtt{foldedinstr}}`.
Such a folded instruction can appear anywhere a regular instruction can.

.. MathJax doesn't handle LaTex multicolumns, thus the spacing hack in the following formula.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tfoldedinstr}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~{{\Tplaininstr}}_{{\I}}~~{{\Tinstrs}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\equiv\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   {{\Tinstrs}}_{{\I}}~~{{\Tplaininstr}}_{{\I}} \\
   \end{array}
   } \\
   & & | & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{block}’}~~{{\Tlabel}}_{{\I}}~~{{\Tblocktype}}_{{\I}}~~{{\Tinstrs}}_{{\I'}}~~\mbox{‘\texttt{{)}}’} & \quad\equiv\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \mbox{‘\texttt{block}’}~~{{\Tlabel}}_{{\I}}~~{{\Tblocktype}}_{{\I}}~~{{\Tinstrs}}_{{\I'}}~~\mbox{‘\texttt{end}’} \\
   \end{array}
   } \\
   & & | & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{loop}’}~~{{\Tlabel}}_{{\I}}~~{{\Tblocktype}}_{{\I}}~~{{\Tinstrs}}_{{\I'}}~~\mbox{‘\texttt{{)}}’} & \quad\equiv\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \mbox{‘\texttt{loop}’}~~{{\Tlabel}}_{{\I}}~~{{\Tblocktype}}_{{\I}}~~{{\Tinstrs}}_{{\I'}}~~\mbox{‘\texttt{end}’} \\
   \end{array}
   } \\
   & & | & \begin{array}[t]{@{}l@{}} \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{if}’}~~{{\Tlabel}}_{{\I}}~~{{\Tblocktype}}_{{\I}}~~{{{\Tfoldedinstr}}_{{\I}}^\ast} \\
     \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{then}’}~~{{\mathit{in}}_1^\ast}{:}{{\Tinstrs}}_{{\I'}}~~\mbox{‘\texttt{{)}}’}~~{(\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{else}’}~~{{\mathit{in}}_2^\ast}{:}{{\Tinstrs}}_{{\I'}}~~\mbox{‘\texttt{{)}}’})^?}~~\mbox{‘\texttt{{)}}’} \end{array} & \quad\equiv\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   {{{\Tfoldedinstr}}_{{\I}}^\ast}~~\mbox{‘\texttt{if}’}~~{{\Tlabel}}_{{\I}}~~{{\Tblocktype}}_{{\I}}~~{{\mathit{in}}_1^\ast}{:}{{\Tinstrs}}_{{\I'}}~~{(\mbox{‘\texttt{else}’}~~{{\mathit{in}}_2^\ast}{:}{{\Tinstrs}}_{{\I'}})^?}~~\mbox{‘\texttt{end}’} \\
   \end{array}
   } \\
   & & | & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{try\_table}’}~~{{\Tlabel}}_{{\I}}~~{{\Tblocktype}}_{{\I}}~~{{{\Tcatch}}_{{\I}}^\ast}~~{{\Tinstrs}}_{{\I'}}~~\mbox{‘\texttt{{)}}’} & \quad\equiv\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \mbox{‘\texttt{try\_table}’}~~{{\Tlabel}}_{{\I}}~~{{\Tblocktype}}_{{\I}}~~{{{\Tcatch}}_{{\I}}^\ast}~~{{\Tinstrs}}_{{\I'}}~~\mbox{‘\texttt{end}’} \\
   \end{array}
   } \\
   \end{array}

.. note::
   For example, the instruction sequence

   .. math::
      \mathtt{(local.get~\$x)~(i32.const~2)~i32.add~(i32.const~3)~i32.mul}

   can be folded into

   .. math::
      \mathtt{(i32.mul~(i32.add~(local.get~\$x)~(i32.const~2))~(i32.const~3))}

   Folded instructions are solely syntactic sugar,
   no additional syntactic or type-based checking is implied.


.. index:: expression
   pair: text format; expression
   single: expression; constant
.. _text-expr:

Expressions
~~~~~~~~~~~

Expressions are written as instruction sequences.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Texpr}}_{{\I}} & ::= & {{\mathit{in}}^\ast}{:}{{\Tinstrs}}_{{\I}} & \quad\Rightarrow\quad{} & {{\mathit{in}}^\ast} \\
   \end{array}
