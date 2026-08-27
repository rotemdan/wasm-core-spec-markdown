.. index:: instruction, ! opcode
.. _binary-instr:

Instructions
------------

:ref:`Instructions <syntax-instr>` are encoded by *opcodes*.
Each opcode is represented by a single byte,
and is followed by the instruction's immediate arguments, where present.
The only exception are :ref:`structured control instructions <binary-instr-control>`, which consist of several opcodes bracketing their nested instruction sequences.

.. note::
   The byte codes chosen to encode instructions are historical and do not follow a consistent pattern.
   In this section, instructions are hence not presented in opcode order,
   but instead grouped consistently with other sections in this document.
   An instruction index ordered by opcode can be found in the :ref:`Appendix <index-instr>`.

   Gaps in the byte code ranges are reserved for future extensions.


.. index:: parametric instruction, value type, polymorphism
   pair: binary format; instruction
.. _binary-instr-parametric:

Parametric Instructions
~~~~~~~~~~~~~~~~~~~~~~~

:ref:`Parametric instructions <syntax-instr-parametric>` are represented by single byte codes, possibly followed by a type annotation.

.. _binary-nop:
.. _binary-unreachable:
.. _binary-drop:
.. _binary-select:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Binstr} & ::= & \mathtt{0x00} & \quad\Rightarrow\quad{} & \UNREACHABLE \\
   & & | & \mathtt{0x01} & \quad\Rightarrow\quad{} & \NOP \\
   & & | & \mathtt{0x1A} & \quad\Rightarrow\quad{} & \DROP \\
   & & | & \mathtt{0x1B} & \quad\Rightarrow\quad{} & \SELECT \\
   & & | & \mathtt{0x1C}~~{t^\ast}{:}{\Blist}({\Bvaltype}) & \quad\Rightarrow\quad{} & \SELECT~{t^\ast} \\
   \end{array}


.. index:: control instructions, structured control, exception handling, label, block, branch, result type, value type, block type, label index, function index, tag index, type index, list, polymorphism, LEB128
   pair: binary format; instruction
   pair: binary format; block type
.. _binary-instr-control:

Control Instructions
~~~~~~~~~~~~~~~~~~~~

:ref:`Control instructions <syntax-instr-control>` have varying encodings. For structured instructions, the instruction sequences forming nested blocks are delimited with explicit opcodes for :math:`\mathsf{end}` and :math:`\mathsf{else}`.

:ref:`Block types <syntax-blocktype>` are encoded in special compressed form, by either the byte :math:`\mathtt{0x40}` indicating the empty type, as a single :ref:`value type <binary-valtype>`, or as a :ref:`type index <binary-typeidx>` encoded as a positive :ref:`signed integer <binary-sint>`.

.. _binary-blocktype:
.. _binary-block:
.. _binary-loop:
.. _binary-if:
.. _binary-br:
.. _binary-br_if:
.. _binary-br_table:
.. _binary-br_on_null:
.. _binary-br_on_non_null:
.. _binary-br_on_cast:
.. _binary-br_on_cast_fail:
.. _binary-return:
.. _binary-call:
.. _binary-call_ref:
.. _binary-call_indirect:
.. _binary-return_call:
.. _binary-return_call_ref:
.. _binary-return_call_indirect:
.. _binary-throw:
.. _binary-throw_ref:
.. _binary-try_table:
.. _binary-catch:
.. _binary-castop:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Bblocktype} & ::= & \mathtt{0x40} & \quad\Rightarrow\quad{} & \epsilon \\
   & & | & t{:}{\Bvaltype} & \quad\Rightarrow\quad{} & t \\
   & & | & i{:}{\Bs33} & \quad\Rightarrow\quad{} & i & \quad \mbox{if}~ i \geq 0 \\[0.8ex]
   & {\Binstr} & ::= & \dots \\
   & & | & \mathtt{0x02}~~{\mathit{bt}}{:}{\Bblocktype}~~{({\mathit{in}}{:}{\Binstr})^\ast}~~\mathtt{0x0B} & \quad\Rightarrow\quad{} & \BLOCK~{\mathit{bt}}~{{\mathit{in}}^\ast} \\
   & & | & \mathtt{0x03}~~{\mathit{bt}}{:}{\Bblocktype}~~{({\mathit{in}}{:}{\Binstr})^\ast}~~\mathtt{0x0B} & \quad\Rightarrow\quad{} & \LOOP~{\mathit{bt}}~{{\mathit{in}}^\ast} \\
   & & | & \mathtt{0x04}~~{\mathit{bt}}{:}{\Bblocktype}~~{({\mathit{in}}{:}{\Binstr})^\ast}~~\mathtt{0x0B} & \quad\Rightarrow\quad{} & \IF~{\mathit{bt}}~{{\mathit{in}}^\ast}~\ELSE~\epsilon \\
   & & | & \begin{array}[t]{@{}l@{}} \mathtt{0x04}~~{\mathit{bt}}{:}{\Bblocktype}~~{({\mathit{in}}_1{:}{\Binstr})^\ast} \\
     \mathtt{0x05}~~{({\mathit{in}}_2{:}{\Binstr})^\ast}~~\mathtt{0x0B} \end{array} & \quad\Rightarrow\quad{} & \IF~{\mathit{bt}}~{{\mathit{in}}_1^\ast}~\ELSE~{{\mathit{in}}_2^\ast} \\
   & & | & \mathtt{0x08}~~x{:}{\Btagidx} & \quad\Rightarrow\quad{} & \THROW~x \\
   & & | & \mathtt{0x0A} & \quad\Rightarrow\quad{} & \THROWREF \\
   & & | & \mathtt{0x0C}~~l{:}{\Blabelidx} & \quad\Rightarrow\quad{} & \BR~l \\
   & & | & \mathtt{0x0D}~~l{:}{\Blabelidx} & \quad\Rightarrow\quad{} & \BRIF~l \\
   & & | & \mathtt{0x0E}~~{l^\ast}{:}{\Blist}({\Blabelidx})~~l_n{:}{\Blabelidx} & \quad\Rightarrow\quad{} & \BRTABLE~{l^\ast}~l_n \\
   & & | & \mathtt{0x0F} & \quad\Rightarrow\quad{} & \RETURN \\
   & & | & \mathtt{0x10}~~x{:}{\Bfuncidx} & \quad\Rightarrow\quad{} & \CALL~x \\
   & & | & \mathtt{0x11}~~y{:}{\Btypeidx}~~x{:}{\Btableidx} & \quad\Rightarrow\quad{} & \CALLINDIRECT~x~y \\
   & & | & \mathtt{0x12}~~x{:}{\Bfuncidx} & \quad\Rightarrow\quad{} & \RETURNCALL~x \\
   & & | & \mathtt{0x13}~~y{:}{\Btypeidx}~~x{:}{\Btableidx} & \quad\Rightarrow\quad{} & \RETURNCALLINDIRECT~x~y \\
   & & | & \mathtt{0x14}~~x{:}{\Btypeidx} & \quad\Rightarrow\quad{} & \CALLREF~x \\
   & & | & \mathtt{0x15}~~x{:}{\Btypeidx} & \quad\Rightarrow\quad{} & \RETURNCALLREF~x \\
   & & | & \mathtt{0x1F}~~{\mathit{bt}}{:}{\Bblocktype}~~{c^\ast}{:}{\Blist}({\Bcatch})~~{({\mathit{in}}{:}{\Binstr})^\ast}~~\mathtt{0x0B} & \quad\Rightarrow\quad{} & \TRYTABLE~{\mathit{bt}}~{c^\ast}~{{\mathit{in}}^\ast} \\
   & & | & \mathtt{0xD5}~~l{:}{\Blabelidx} & \quad\Rightarrow\quad{} & \BRONNULL~l \\
   & & | & \mathtt{0xD6}~~l{:}{\Blabelidx} & \quad\Rightarrow\quad{} & \BRONNONNULL~l \\
   & & | & \begin{array}[t]{@{}l@{}} \mathtt{0xFB}~~24{:}{\Bu32}~~({{\NULL}_1^?}, {{\NULL}_2^?}){:}{\Bcastop} \\
     l{:}{\Blabelidx}~~{\mathit{ht}}_1{:}{\Bheaptype}~~{\mathit{ht}}_2{:}{\Bheaptype} \end{array} & \quad\Rightarrow\quad{} & \BRONCAST~l~(\REF~{{\NULL}_1^?}~{\mathit{ht}}_1)~(\REF~{{\NULL}_2^?}~{\mathit{ht}}_2) \\
   & & | & \begin{array}[t]{@{}l@{}} \mathtt{0xFB}~~25{:}{\Bu32}~~({{\NULL}_1^?}, {{\NULL}_2^?}){:}{\Bcastop} \\
     l{:}{\Blabelidx}~~{\mathit{ht}}_1{:}{\Bheaptype}~~{\mathit{ht}}_2{:}{\Bheaptype} \end{array} & \quad\Rightarrow\quad{} & \BRONCASTFAIL~l~(\REF~{{\NULL}_1^?}~{\mathit{ht}}_1)~(\REF~{{\NULL}_2^?}~{\mathit{ht}}_2) \\[0.8ex]
   & {\Bcatch} & ::= & \mathtt{0x00}~~x{:}{\Btagidx}~~l{:}{\Blabelidx} & \quad\Rightarrow\quad{} & \CATCH~x~l \\
   & & | & \mathtt{0x01}~~x{:}{\Btagidx}~~l{:}{\Blabelidx} & \quad\Rightarrow\quad{} & \CATCHREF~x~l \\
   & & | & \mathtt{0x02}~~l{:}{\Blabelidx} & \quad\Rightarrow\quad{} & \CATCHALL~l \\
   & & | & \mathtt{0x03}~~l{:}{\Blabelidx} & \quad\Rightarrow\quad{} & \CATCHALLREF~l \\[0.8ex]
   & {\Bcastop} & ::= & \mathtt{0x00} & \quad\Rightarrow\quad{} & (\epsilon, \epsilon) \\
   & & | & \mathtt{0x01} & \quad\Rightarrow\quad{} & (\NULL, \epsilon) \\
   & & | & \mathtt{0x02} & \quad\Rightarrow\quad{} & (\epsilon, \NULL) \\
   & & | & \mathtt{0x03} & \quad\Rightarrow\quad{} & (\NULL, \NULL) \\
   \end{array}


.. note::
   The :math:`\mathsf{else}` opcode :math:`\mathtt{0x05}` in the encoding of an :math:`\mathsf{if}` instruction can be omitted if the following instruction sequence is empty.

   Unlike any :ref:`other occurrence <binary-typeidx>`, the :ref:`type index <syntax-typeidx>` in a :ref:`block type <syntax-blocktype>` is encoded as a positive :ref:`signed integer <syntax-sint>`, so that its |SignedLEB128| bit pattern cannot collide with the encoding of :ref:`value types <binary-valtype>` or the special code :math:`\mathtt{0x40}`, which correspond to the LEB128 encoding of negative integers.
   To avoid any loss in the range of allowed indices, it is treated as a 33 bit signed integer.


.. index:: variable instructions, local index, global index
   pair: binary format; instruction
.. _binary-instr-variable:

Variable Instructions
~~~~~~~~~~~~~~~~~~~~~

:ref:`Variable instructions <syntax-instr-variable>` are represented by byte codes followed by the encoding of the respective :ref:`index <syntax-index>`.

.. _binary-local.get:
.. _binary-local.set:
.. _binary-local.tee:
.. _binary-global.get:
.. _binary-global.set:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Binstr} & ::= & \dots \\
   & & | & \mathtt{0x20}~~x{:}{\Blocalidx} & \quad\Rightarrow\quad{} & \LOCALGET~x \\
   & & | & \mathtt{0x21}~~x{:}{\Blocalidx} & \quad\Rightarrow\quad{} & \LOCALSET~x \\
   & & | & \mathtt{0x22}~~x{:}{\Blocalidx} & \quad\Rightarrow\quad{} & \LOCALTEE~x \\
   & & | & \mathtt{0x23}~~x{:}{\Bglobalidx} & \quad\Rightarrow\quad{} & \GLOBALGET~x \\
   & & | & \mathtt{0x24}~~x{:}{\Bglobalidx} & \quad\Rightarrow\quad{} & \GLOBALSET~x \\
   \end{array}


.. index:: table instruction, table index
   pair: binary format; instruction
.. _binary-instr-table:
.. _binary-table.get:
.. _binary-table.set:
.. _binary-table.size:
.. _binary-table.grow:
.. _binary-table.fill:
.. _binary-table.copy:
.. _binary-table.init:
.. _binary-elem.drop:

Table Instructions
~~~~~~~~~~~~~~~~~~

:ref:`Table instructions <syntax-instr-table>` are represented either by a single byte or a one byte prefix followed by a variable-length :ref:`unsigned integer <binary-uint>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Binstr} & ::= & \dots \\
   & & | & \mathtt{0x25}~~x{:}{\Btableidx} & \quad\Rightarrow\quad{} & \TABLEGET~x \\
   & & | & \mathtt{0x26}~~x{:}{\Btableidx} & \quad\Rightarrow\quad{} & \TABLESET~x \\
   & & | & \mathtt{0xFC}~~12{:}{\Bu32}~~y{:}{\Belemidx}~~x{:}{\Btableidx} & \quad\Rightarrow\quad{} & \TABLEINIT~x~y \\
   & & | & \mathtt{0xFC}~~13{:}{\Bu32}~~x{:}{\Belemidx} & \quad\Rightarrow\quad{} & \ELEMDROP~x \\
   & & | & \mathtt{0xFC}~~14{:}{\Bu32}~~x_1{:}{\Btableidx}~~x_2{:}{\Btableidx} & \quad\Rightarrow\quad{} & \TABLECOPY~x_1~x_2 \\
   & & | & \mathtt{0xFC}~~15{:}{\Bu32}~~x{:}{\Btableidx} & \quad\Rightarrow\quad{} & \TABLEGROW~x \\
   & & | & \mathtt{0xFC}~~16{:}{\Bu32}~~x{:}{\Btableidx} & \quad\Rightarrow\quad{} & \TABLESIZE~x \\
   & & | & \mathtt{0xFC}~~17{:}{\Bu32}~~x{:}{\Btableidx} & \quad\Rightarrow\quad{} & \TABLEFILL~x \\
   \end{array}


.. index:: memory instruction, memory index
   pair: binary format; instruction
.. _binary-instr-memory:

Memory Instructions
~~~~~~~~~~~~~~~~~~~

Each variant of :ref:`memory instruction <syntax-instr-memory>` is encoded with a different byte code. Loads and stores are followed by the encoding of their |memarg| immediate, which includes the :ref:`memory index <binary-memidx>` if bit 6 of the flags field containing alignment is set; the memory index defaults to 0 otherwise.

.. _binary-memarg:
.. _binary-load:
.. _binary-loadn:
.. _binary-store:
.. _binary-storen:
.. _binary-memory.size:
.. _binary-memory.grow:
.. _binary-memory.fill:
.. _binary-memory.copy:
.. _binary-memory.init:
.. _binary-data.drop:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Bmemarg} & ::= & n{:}{\Bu32}~~m{:}{\Bu64} & \quad\Rightarrow\quad{} & (0, \{ \ALIGN~n,\;\allowbreak \OFFSET~m \}) & \quad \mbox{if}~ n < {2^{6}} \\
   & & | & n{:}{\Bu32}~~x{:}{\Bmemidx}~~m{:}{\Bu64} & \quad\Rightarrow\quad{} & (x, \{ \ALIGN~(n - {2^{6}}),\;\allowbreak \OFFSET~m \}) & \quad \mbox{if}~ {2^{6}} \leq n < {2^{7}} \\[0.8ex]
   & {\Binstr} & ::= & \dots \\
   & & | & \mathtt{0x28}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & \I32{.}\LOAD~x~{\mathit{ao}} \\
   & & | & \mathtt{0x29}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & \I64{.}\LOAD~x~{\mathit{ao}} \\
   & & | & \mathtt{0x2A}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & \F32{.}\LOAD~x~{\mathit{ao}} \\
   & & | & \mathtt{0x2B}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & \F64{.}\LOAD~x~{\mathit{ao}} \\
   & & | & \mathtt{0x2C}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & {\I32{.}\LOAD}{{\mathsf{{\scriptstyle 8}}}{\mathsf{\_}}{\S}}~x~{\mathit{ao}} \\
   & & | & \mathtt{0x2D}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & {\I32{.}\LOAD}{{\mathsf{{\scriptstyle 8}}}{\mathsf{\_}}{\U}}~x~{\mathit{ao}} \\
   & & | & \mathtt{0x2E}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & {\I32{.}\LOAD}{{\mathsf{{\scriptstyle 16}}}{\mathsf{\_}}{\S}}~x~{\mathit{ao}} \\
   & & | & \mathtt{0x2F}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & {\I32{.}\LOAD}{{\mathsf{{\scriptstyle 16}}}{\mathsf{\_}}{\U}}~x~{\mathit{ao}} \\
   & & | & \mathtt{0x30}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & {\I64{.}\LOAD}{{\mathsf{{\scriptstyle 8}}}{\mathsf{\_}}{\S}}~x~{\mathit{ao}} \\
   & & | & \mathtt{0x31}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & {\I64{.}\LOAD}{{\mathsf{{\scriptstyle 8}}}{\mathsf{\_}}{\U}}~x~{\mathit{ao}} \\
   & & | & \mathtt{0x32}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & {\I64{.}\LOAD}{{\mathsf{{\scriptstyle 16}}}{\mathsf{\_}}{\S}}~x~{\mathit{ao}} \\
   & & | & \mathtt{0x33}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & {\I64{.}\LOAD}{{\mathsf{{\scriptstyle 16}}}{\mathsf{\_}}{\U}}~x~{\mathit{ao}} \\
   & & | & \mathtt{0x34}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & {\I64{.}\LOAD}{{\mathsf{{\scriptstyle 32}}}{\mathsf{\_}}{\S}}~x~{\mathit{ao}} \\
   & & | & \mathtt{0x35}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & {\I64{.}\LOAD}{{\mathsf{{\scriptstyle 32}}}{\mathsf{\_}}{\U}}~x~{\mathit{ao}} \\
   & & | & \mathtt{0x36}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & \I32{.}\STORE~x~{\mathit{ao}} \\
   & & | & \mathtt{0x37}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & \I64{.}\STORE~x~{\mathit{ao}} \\
   & & | & \mathtt{0x38}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & \F32{.}\STORE~x~{\mathit{ao}} \\
   & & | & \mathtt{0x39}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & \F64{.}\STORE~x~{\mathit{ao}} \\
   & & | & \mathtt{0x3A}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & {\I32{.}\STORE}{\mathsf{{\scriptstyle 8}}}~x~{\mathit{ao}} \\
   & & | & \mathtt{0x3B}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & {\I32{.}\STORE}{\mathsf{{\scriptstyle 16}}}~x~{\mathit{ao}} \\
   & & | & \mathtt{0x3C}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & {\I64{.}\STORE}{\mathsf{{\scriptstyle 8}}}~x~{\mathit{ao}} \\
   & & | & \mathtt{0x3D}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & {\I64{.}\STORE}{\mathsf{{\scriptstyle 16}}}~x~{\mathit{ao}} \\
   & & | & \mathtt{0x3E}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & {\I64{.}\STORE}{\mathsf{{\scriptstyle 32}}}~x~{\mathit{ao}} \\
   & & | & \mathtt{0x3F}~~x{:}{\Bmemidx} & \quad\Rightarrow\quad{} & \MEMORYSIZE~x \\
   & & | & \mathtt{0x40}~~x{:}{\Bmemidx} & \quad\Rightarrow\quad{} & \MEMORYGROW~x \\
   & & | & \mathtt{0xFC}~~8{:}{\Bu32}~~y{:}{\Bdataidx}~~x{:}{\Bmemidx} & \quad\Rightarrow\quad{} & \MEMORYINIT~x~y \\
   & & | & \mathtt{0xFC}~~9{:}{\Bu32}~~x{:}{\Bdataidx} & \quad\Rightarrow\quad{} & \DATADROP~x \\
   & & | & \mathtt{0xFC}~~10{:}{\Bu32}~~x_1{:}{\Bmemidx}~~x_2{:}{\Bmemidx} & \quad\Rightarrow\quad{} & \MEMORYCOPY~x_1~x_2 \\
   & & | & \mathtt{0xFC}~~11{:}{\Bu32}~~x{:}{\Bmemidx} & \quad\Rightarrow\quad{} & \MEMORYFILL~x \\
   \end{array}


.. index:: reference instruction
   pair: binary format; instruction
.. _binary-instr-ref:

Reference Instructions
~~~~~~~~~~~~~~~~~~~~~~

Generic :ref:`reference instructions <syntax-instr-ref>` are represented by single byte codes, others use prefixes and type operands.

.. _binary-ref.null:
.. _binary-ref.func:
.. _binary-ref.is_null:
.. _binary-ref.as_non_null:
.. _binary-ref.test:
.. _binary-ref.cast:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Binstr} & ::= & \dots \\
   & & | & \mathtt{0xD0}~~{\mathit{ht}}{:}{\Bheaptype} & \quad\Rightarrow\quad{} & \REFNULL~{\mathit{ht}} \\
   & & | & \mathtt{0xD1} & \quad\Rightarrow\quad{} & \REFISNULL \\
   & & | & \mathtt{0xD2}~~x{:}{\Bfuncidx} & \quad\Rightarrow\quad{} & \REFFUNC~x \\
   & & | & \mathtt{0xD3} & \quad\Rightarrow\quad{} & \REFEQ \\
   & & | & \mathtt{0xD4} & \quad\Rightarrow\quad{} & \REFASNONNULL \\
   & & | & \mathtt{0xFB}~~20{:}{\Bu32}~~{\mathit{ht}}{:}{\Bheaptype} & \quad\Rightarrow\quad{} & \REFTEST~(\REF~{\mathit{ht}}) \\
   & & | & \mathtt{0xFB}~~21{:}{\Bu32}~~{\mathit{ht}}{:}{\Bheaptype} & \quad\Rightarrow\quad{} & \REFTEST~(\REF~\NULL~{\mathit{ht}}) \\
   & & | & \mathtt{0xFB}~~22{:}{\Bu32}~~{\mathit{ht}}{:}{\Bheaptype} & \quad\Rightarrow\quad{} & \REFCAST~(\REF~{\mathit{ht}}) \\
   & & | & \mathtt{0xFB}~~23{:}{\Bu32}~~{\mathit{ht}}{:}{\Bheaptype} & \quad\Rightarrow\quad{} & \REFCAST~(\REF~\NULL~{\mathit{ht}}) \\
   \end{array}


.. index:: aggregate instruction
   pair: binary format; instruction
.. _binary-instr-aggr:

Aggregate Instructions
~~~~~~~~~~~~~~~~~~~~~~

:ref:`Aggregate instructions <syntax-instr-aggr>` all use a prefix.

.. _binary-ref.i31:
.. _binary-i31.get_s:
.. _binary-i31.get_u:
.. _binary-struct.new:
.. _binary-struct.new_default:
.. _binary-struct.get:
.. _binary-struct.get_s:
.. _binary-struct.get_u:
.. _binary-struct.set:
.. _binary-array.new:
.. _binary-array.new_default:
.. _binary-array.new_fixed:
.. _binary-array.new_elem:
.. _binary-array.new_data:
.. _binary-array.get:
.. _binary-array.get_s:
.. _binary-array.get_u:
.. _binary-array.set:
.. _binary-array.len:
.. _binary-array.fill:
.. _binary-array.copy:
.. _binary-array.init_data:
.. _binary-array.init_elem:
.. _binary-any.convert_extern:
.. _binary-extern.convert_any:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Binstr} & ::= & \dots \\
   & & | & \mathtt{0xFB}~~0{:}{\Bu32}~~x{:}{\Btypeidx} & \quad\Rightarrow\quad{} & \STRUCTNEW~x \\
   & & | & \mathtt{0xFB}~~1{:}{\Bu32}~~x{:}{\Btypeidx} & \quad\Rightarrow\quad{} & \STRUCTNEWDEFAULT~x \\
   & & | & \mathtt{0xFB}~~2{:}{\Bu32}~~x{:}{\Btypeidx}~~i{:}{\Bfieldidx} & \quad\Rightarrow\quad{} & \STRUCTGET~x~i \\
   & & | & \mathtt{0xFB}~~3{:}{\Bu32}~~x{:}{\Btypeidx}~~i{:}{\Bfieldidx} & \quad\Rightarrow\quad{} & {\STRUCTGET}{\mathsf{\_}}{\S}~x~i \\
   & & | & \mathtt{0xFB}~~4{:}{\Bu32}~~x{:}{\Btypeidx}~~i{:}{\Bfieldidx} & \quad\Rightarrow\quad{} & {\STRUCTGET}{\mathsf{\_}}{\U}~x~i \\
   & & | & \mathtt{0xFB}~~5{:}{\Bu32}~~x{:}{\Btypeidx}~~i{:}{\Bfieldidx} & \quad\Rightarrow\quad{} & \STRUCTSET~x~i \\
   & & | & \mathtt{0xFB}~~6{:}{\Bu32}~~x{:}{\Btypeidx} & \quad\Rightarrow\quad{} & \ARRAYNEW~x \\
   & & | & \mathtt{0xFB}~~7{:}{\Bu32}~~x{:}{\Btypeidx} & \quad\Rightarrow\quad{} & \ARRAYNEWDEFAULT~x \\
   & & | & \mathtt{0xFB}~~8{:}{\Bu32}~~x{:}{\Btypeidx}~~n{:}{\Bu32} & \quad\Rightarrow\quad{} & \ARRAYNEWFIXED~x~n \\
   & & | & \mathtt{0xFB}~~9{:}{\Bu32}~~x{:}{\Btypeidx}~~y{:}{\Bdataidx} & \quad\Rightarrow\quad{} & \ARRAYNEWDATA~x~y \\
   & & | & \mathtt{0xFB}~~10{:}{\Bu32}~~x{:}{\Btypeidx}~~y{:}{\Belemidx} & \quad\Rightarrow\quad{} & \ARRAYNEWELEM~x~y \\
   & & | & \mathtt{0xFB}~~11{:}{\Bu32}~~x{:}{\Btypeidx} & \quad\Rightarrow\quad{} & \ARRAYGET~x \\
   & & | & \mathtt{0xFB}~~12{:}{\Bu32}~~x{:}{\Btypeidx} & \quad\Rightarrow\quad{} & {\ARRAYGET}{\mathsf{\_}}{\S}~x \\
   & & | & \mathtt{0xFB}~~13{:}{\Bu32}~~x{:}{\Btypeidx} & \quad\Rightarrow\quad{} & {\ARRAYGET}{\mathsf{\_}}{\U}~x \\
   & & | & \mathtt{0xFB}~~14{:}{\Bu32}~~x{:}{\Btypeidx} & \quad\Rightarrow\quad{} & \ARRAYSET~x \\
   & & | & \mathtt{0xFB}~~15{:}{\Bu32} & \quad\Rightarrow\quad{} & \ARRAYLEN \\
   & & | & \mathtt{0xFB}~~16{:}{\Bu32}~~x{:}{\Btypeidx} & \quad\Rightarrow\quad{} & \ARRAYFILL~x \\
   & & | & \mathtt{0xFB}~~17{:}{\Bu32}~~x_1{:}{\Btypeidx}~~x_2{:}{\Btypeidx} & \quad\Rightarrow\quad{} & \ARRAYCOPY~x_1~x_2 \\
   & & | & \mathtt{0xFB}~~18{:}{\Bu32}~~x{:}{\Btypeidx}~~y{:}{\Bdataidx} & \quad\Rightarrow\quad{} & \ARRAYINITDATA~x~y \\
   & & | & \mathtt{0xFB}~~19{:}{\Bu32}~~x{:}{\Btypeidx}~~y{:}{\Belemidx} & \quad\Rightarrow\quad{} & \ARRAYINITELEM~x~y \\
   & & | & \mathtt{0xFB}~~26{:}{\Bu32} & \quad\Rightarrow\quad{} & \ANYCONVERTEXTERN \\
   & & | & \mathtt{0xFB}~~27{:}{\Bu32} & \quad\Rightarrow\quad{} & \EXTERNCONVERTANY \\
   & & | & \mathtt{0xFB}~~28{:}{\Bu32} & \quad\Rightarrow\quad{} & \REFI31 \\
   & & | & \mathtt{0xFB}~~29{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I31GET}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFB}~~30{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I31GET}{\mathsf{\_}}{\U} \\
   \end{array}


.. index:: numeric instruction
   pair: binary format; instruction
.. _binary-instr-numeric:

Numeric Instructions
~~~~~~~~~~~~~~~~~~~~

All variants of :ref:`numeric instructions <syntax-instr-numeric>` are represented by separate byte codes.

The :math:`\mathsf{const}` instructions are followed by the respective literal.

.. _binary-const:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Binstr} & ::= & \dots \\
   & & | & \mathtt{0x41}~~i{:}{\Bi32} & \quad\Rightarrow\quad{} & \I32{.}\CONST~i \\
   & & | & \mathtt{0x42}~~i{:}{\Bi64} & \quad\Rightarrow\quad{} & \I64{.}\CONST~i \\
   & & | & \mathtt{0x43}~~p{:}{\Bf32} & \quad\Rightarrow\quad{} & \F32{.}\CONST~p \\
   & & | & \mathtt{0x44}~~p{:}{\Bf64} & \quad\Rightarrow\quad{} & \F64{.}\CONST~p \\
   \end{array}

All other numeric instructions are plain opcodes without any immediates.

.. _binary-testop:
.. _binary-relop:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Binstr} & ::= & \dots \\
   & & | & \mathtt{0x45} & \quad\Rightarrow\quad{} & \I32 {.} \EQZ \\
   & & | & \mathtt{0x46} & \quad\Rightarrow\quad{} & \I32 {.} \EQ \\
   & & | & \mathtt{0x47} & \quad\Rightarrow\quad{} & \I32 {.} \NE \\
   & & | & \mathtt{0x48} & \quad\Rightarrow\quad{} & \I32 {.} {\LT}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0x49} & \quad\Rightarrow\quad{} & \I32 {.} {\LT}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0x4A} & \quad\Rightarrow\quad{} & \I32 {.} {\GT}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0x4B} & \quad\Rightarrow\quad{} & \I32 {.} {\GT}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0x4C} & \quad\Rightarrow\quad{} & \I32 {.} {\LE}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0x4D} & \quad\Rightarrow\quad{} & \I32 {.} {\LE}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0x4E} & \quad\Rightarrow\quad{} & \I32 {.} {\GE}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0x4F} & \quad\Rightarrow\quad{} & \I32 {.} {\GE}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0x50} & \quad\Rightarrow\quad{} & \I64 {.} \EQZ \\
   & & | & \mathtt{0x51} & \quad\Rightarrow\quad{} & \I64 {.} \EQ \\
   & & | & \mathtt{0x52} & \quad\Rightarrow\quad{} & \I64 {.} \NE \\
   & & | & \mathtt{0x53} & \quad\Rightarrow\quad{} & \I64 {.} {\LT}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0x54} & \quad\Rightarrow\quad{} & \I64 {.} {\LT}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0x55} & \quad\Rightarrow\quad{} & \I64 {.} {\GT}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0x56} & \quad\Rightarrow\quad{} & \I64 {.} {\GT}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0x57} & \quad\Rightarrow\quad{} & \I64 {.} {\LE}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0x58} & \quad\Rightarrow\quad{} & \I64 {.} {\LE}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0x59} & \quad\Rightarrow\quad{} & \I64 {.} {\GE}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0x5A} & \quad\Rightarrow\quad{} & \I64 {.} {\GE}{\mathsf{\_}}{\U} \\
   \end{array}

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Binstr} & ::= & \dots \\
   & & | & \mathtt{0x5B} & \quad\Rightarrow\quad{} & \F32 {.} \EQ \\
   & & | & \mathtt{0x5C} & \quad\Rightarrow\quad{} & \F32 {.} \NE \\
   & & | & \mathtt{0x5D} & \quad\Rightarrow\quad{} & \F32 {.} \LT \\
   & & | & \mathtt{0x5E} & \quad\Rightarrow\quad{} & \F32 {.} \GT \\
   & & | & \mathtt{0x5F} & \quad\Rightarrow\quad{} & \F32 {.} \LE \\
   & & | & \mathtt{0x60} & \quad\Rightarrow\quad{} & \F32 {.} \GE \\
   & & | & \mathtt{0x61} & \quad\Rightarrow\quad{} & \F64 {.} \EQ \\
   & & | & \mathtt{0x62} & \quad\Rightarrow\quad{} & \F64 {.} \NE \\
   & & | & \mathtt{0x63} & \quad\Rightarrow\quad{} & \F64 {.} \LT \\
   & & | & \mathtt{0x64} & \quad\Rightarrow\quad{} & \F64 {.} \GT \\
   & & | & \mathtt{0x65} & \quad\Rightarrow\quad{} & \F64 {.} \LE \\
   & & | & \mathtt{0x66} & \quad\Rightarrow\quad{} & \F64 {.} \GE \\
   \end{array}

.. _binary-unop:
.. _binary-binop:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Binstr} & ::= & \dots \\
   & & | & \mathtt{0x67} & \quad\Rightarrow\quad{} & \I32 {.} \CLZ \\
   & & | & \mathtt{0x68} & \quad\Rightarrow\quad{} & \I32 {.} \CTZ \\
   & & | & \mathtt{0x69} & \quad\Rightarrow\quad{} & \I32 {.} \POPCNT \\
   & & | & \mathtt{0x6A} & \quad\Rightarrow\quad{} & \I32 {.} \ADD \\
   & & | & \mathtt{0x6B} & \quad\Rightarrow\quad{} & \I32 {.} \SUB \\
   & & | & \mathtt{0x6C} & \quad\Rightarrow\quad{} & \I32 {.} \MUL \\
   & & | & \mathtt{0x6D} & \quad\Rightarrow\quad{} & \I32 {.} {\DIV}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0x6E} & \quad\Rightarrow\quad{} & \I32 {.} {\DIV}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0x6F} & \quad\Rightarrow\quad{} & \I32 {.} {\REM}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0x70} & \quad\Rightarrow\quad{} & \I32 {.} {\REM}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0x71} & \quad\Rightarrow\quad{} & \I32 {.} \AND \\
   & & | & \mathtt{0x72} & \quad\Rightarrow\quad{} & \I32 {.} \OR \\
   & & | & \mathtt{0x73} & \quad\Rightarrow\quad{} & \I32 {.} \XOR \\
   & & | & \mathtt{0x74} & \quad\Rightarrow\quad{} & \I32 {.} \SHL \\
   & & | & \mathtt{0x75} & \quad\Rightarrow\quad{} & \I32 {.} {\SHR}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0x76} & \quad\Rightarrow\quad{} & \I32 {.} {\SHR}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0x77} & \quad\Rightarrow\quad{} & \I32 {.} \ROTL \\
   & & | & \mathtt{0x78} & \quad\Rightarrow\quad{} & \I32 {.} \ROTR \\
   & & | & \mathtt{0x79} & \quad\Rightarrow\quad{} & \I64 {.} \CLZ \\
   & & | & \mathtt{0x7A} & \quad\Rightarrow\quad{} & \I64 {.} \CTZ \\
   & & | & \mathtt{0x7B} & \quad\Rightarrow\quad{} & \I64 {.} \POPCNT \\
   & & | & \mathtt{0x7C} & \quad\Rightarrow\quad{} & \I64 {.} \ADD \\
   & & | & \mathtt{0x7D} & \quad\Rightarrow\quad{} & \I64 {.} \SUB \\
   & & | & \mathtt{0x7E} & \quad\Rightarrow\quad{} & \I64 {.} \MUL \\
   & & | & \mathtt{0x7F} & \quad\Rightarrow\quad{} & \I64 {.} {\DIV}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0x80} & \quad\Rightarrow\quad{} & \I64 {.} {\DIV}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0x81} & \quad\Rightarrow\quad{} & \I64 {.} {\REM}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0x82} & \quad\Rightarrow\quad{} & \I64 {.} {\REM}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0x83} & \quad\Rightarrow\quad{} & \I64 {.} \AND \\
   & & | & \mathtt{0x84} & \quad\Rightarrow\quad{} & \I64 {.} \OR \\
   & & | & \mathtt{0x85} & \quad\Rightarrow\quad{} & \I64 {.} \XOR \\
   & & | & \mathtt{0x86} & \quad\Rightarrow\quad{} & \I64 {.} \SHL \\
   & & | & \mathtt{0x87} & \quad\Rightarrow\quad{} & \I64 {.} {\SHR}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0x88} & \quad\Rightarrow\quad{} & \I64 {.} {\SHR}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0x89} & \quad\Rightarrow\quad{} & \I64 {.} \ROTL \\
   & & | & \mathtt{0x8A} & \quad\Rightarrow\quad{} & \I64 {.} \ROTR \\
   \end{array}

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Binstr} & ::= & \dots \\
   & & | & \mathtt{0x8B} & \quad\Rightarrow\quad{} & \F32 {.} \ABS \\
   & & | & \mathtt{0x8C} & \quad\Rightarrow\quad{} & \F32 {.} \NEG \\
   & & | & \mathtt{0x8D} & \quad\Rightarrow\quad{} & \F32 {.} \CEIL \\
   & & | & \mathtt{0x8E} & \quad\Rightarrow\quad{} & \F32 {.} \FLOOR \\
   & & | & \mathtt{0x8F} & \quad\Rightarrow\quad{} & \F32 {.} \TRUNC \\
   & & | & \mathtt{0x90} & \quad\Rightarrow\quad{} & \F32 {.} \NEAREST \\
   & & | & \mathtt{0x91} & \quad\Rightarrow\quad{} & \F32 {.} \SQRT \\
   & & | & \mathtt{0x92} & \quad\Rightarrow\quad{} & \F32 {.} \ADD \\
   & & | & \mathtt{0x93} & \quad\Rightarrow\quad{} & \F32 {.} \SUB \\
   & & | & \mathtt{0x94} & \quad\Rightarrow\quad{} & \F32 {.} \MUL \\
   & & | & \mathtt{0x95} & \quad\Rightarrow\quad{} & \F32 {.} \DIV \\
   & & | & \mathtt{0x96} & \quad\Rightarrow\quad{} & \F32 {.} \FMIN \\
   & & | & \mathtt{0x97} & \quad\Rightarrow\quad{} & \F32 {.} \FMAX \\
   & & | & \mathtt{0x98} & \quad\Rightarrow\quad{} & \F32 {.} \COPYSIGN \\
   & & | & \mathtt{0x99} & \quad\Rightarrow\quad{} & \F64 {.} \ABS \\
   & & | & \mathtt{0x9A} & \quad\Rightarrow\quad{} & \F64 {.} \NEG \\
   & & | & \mathtt{0x9B} & \quad\Rightarrow\quad{} & \F64 {.} \CEIL \\
   & & | & \mathtt{0x9C} & \quad\Rightarrow\quad{} & \F64 {.} \FLOOR \\
   & & | & \mathtt{0x9D} & \quad\Rightarrow\quad{} & \F64 {.} \TRUNC \\
   & & | & \mathtt{0x9E} & \quad\Rightarrow\quad{} & \F64 {.} \NEAREST \\
   & & | & \mathtt{0x9F} & \quad\Rightarrow\quad{} & \F64 {.} \SQRT \\
   & & | & \mathtt{0xA0} & \quad\Rightarrow\quad{} & \F64 {.} \ADD \\
   & & | & \mathtt{0xA1} & \quad\Rightarrow\quad{} & \F64 {.} \SUB \\
   & & | & \mathtt{0xA2} & \quad\Rightarrow\quad{} & \F64 {.} \MUL \\
   & & | & \mathtt{0xA3} & \quad\Rightarrow\quad{} & \F64 {.} \DIV \\
   & & | & \mathtt{0xA4} & \quad\Rightarrow\quad{} & \F64 {.} \FMIN \\
   & & | & \mathtt{0xA5} & \quad\Rightarrow\quad{} & \F64 {.} \FMAX \\
   & & | & \mathtt{0xA6} & \quad\Rightarrow\quad{} & \F64 {.} \COPYSIGN \\
   \end{array}

.. _binary-cvtop:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Binstr} & ::= & \dots \\
   & & | & \mathtt{0xA7} & \quad\Rightarrow\quad{} & \I32 {.} {\WRAP}{\mathsf{\_}}{\I64} \\
   & & | & \mathtt{0xA8} & \quad\Rightarrow\quad{} & \I32 {.} {{\TRUNC}{\mathsf{\_}}{\S}}{\mathsf{\_}}{\F32} \\
   & & | & \mathtt{0xA9} & \quad\Rightarrow\quad{} & \I32 {.} {{\TRUNC}{\mathsf{\_}}{\U}}{\mathsf{\_}}{\F32} \\
   & & | & \mathtt{0xAA} & \quad\Rightarrow\quad{} & \I32 {.} {{\TRUNC}{\mathsf{\_}}{\S}}{\mathsf{\_}}{\F64} \\
   & & | & \mathtt{0xAB} & \quad\Rightarrow\quad{} & \I32 {.} {{\TRUNC}{\mathsf{\_}}{\U}}{\mathsf{\_}}{\F64} \\
   & & | & \mathtt{0xAC} & \quad\Rightarrow\quad{} & \I64 {.} {{\EXTEND}{\mathsf{\_}}{\S}}{\mathsf{\_}}{\I32} \\
   & & | & \mathtt{0xAD} & \quad\Rightarrow\quad{} & \I64 {.} {{\EXTEND}{\mathsf{\_}}{\U}}{\mathsf{\_}}{\I32} \\
   & & | & \mathtt{0xAE} & \quad\Rightarrow\quad{} & \I64 {.} {{\TRUNC}{\mathsf{\_}}{\S}}{\mathsf{\_}}{\F32} \\
   & & | & \mathtt{0xAF} & \quad\Rightarrow\quad{} & \I64 {.} {{\TRUNC}{\mathsf{\_}}{\U}}{\mathsf{\_}}{\F32} \\
   & & | & \mathtt{0xB0} & \quad\Rightarrow\quad{} & \I64 {.} {{\TRUNC}{\mathsf{\_}}{\S}}{\mathsf{\_}}{\F64} \\
   & & | & \mathtt{0xB1} & \quad\Rightarrow\quad{} & \I64 {.} {{\TRUNC}{\mathsf{\_}}{\U}}{\mathsf{\_}}{\F64} \\
   & & | & \mathtt{0xB2} & \quad\Rightarrow\quad{} & \F32 {.} {{\CONVERT}{\mathsf{\_}}{\S}}{\mathsf{\_}}{\I32} \\
   & & | & \mathtt{0xB3} & \quad\Rightarrow\quad{} & \F32 {.} {{\CONVERT}{\mathsf{\_}}{\U}}{\mathsf{\_}}{\I32} \\
   & & | & \mathtt{0xB4} & \quad\Rightarrow\quad{} & \F32 {.} {{\CONVERT}{\mathsf{\_}}{\S}}{\mathsf{\_}}{\I64} \\
   & & | & \mathtt{0xB5} & \quad\Rightarrow\quad{} & \F32 {.} {{\CONVERT}{\mathsf{\_}}{\U}}{\mathsf{\_}}{\I64} \\
   & & | & \mathtt{0xB6} & \quad\Rightarrow\quad{} & \F32 {.} {\DEMOTE}{\mathsf{\_}}{\F64} \\
   & & | & \mathtt{0xB7} & \quad\Rightarrow\quad{} & \F64 {.} {{\CONVERT}{\mathsf{\_}}{\S}}{\mathsf{\_}}{\I32} \\
   & & | & \mathtt{0xB8} & \quad\Rightarrow\quad{} & \F64 {.} {{\CONVERT}{\mathsf{\_}}{\U}}{\mathsf{\_}}{\I32} \\
   & & | & \mathtt{0xB9} & \quad\Rightarrow\quad{} & \F64 {.} {{\CONVERT}{\mathsf{\_}}{\S}}{\mathsf{\_}}{\I64} \\
   & & | & \mathtt{0xBA} & \quad\Rightarrow\quad{} & \F64 {.} {{\CONVERT}{\mathsf{\_}}{\U}}{\mathsf{\_}}{\I64} \\
   & & | & \mathtt{0xBB} & \quad\Rightarrow\quad{} & \F64 {.} {\PROMOTE}{\mathsf{\_}}{\F32} \\
   & & | & \mathtt{0xBC} & \quad\Rightarrow\quad{} & \I32 {.} {\REINTERPRET}{\mathsf{\_}}{\F32} \\
   & & | & \mathtt{0xBD} & \quad\Rightarrow\quad{} & \I64 {.} {\REINTERPRET}{\mathsf{\_}}{\F64} \\
   & & | & \mathtt{0xBE} & \quad\Rightarrow\quad{} & \F32 {.} {\REINTERPRET}{\mathsf{\_}}{\I32} \\
   & & | & \mathtt{0xBF} & \quad\Rightarrow\quad{} & \F64 {.} {\REINTERPRET}{\mathsf{\_}}{\I64} \\
   \end{array}

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Binstr} & ::= & \dots \\
   & & | & \mathtt{0xC0} & \quad\Rightarrow\quad{} & \I32 {.} {\EXTEND}{\mathsf{{\scriptstyle 8}}}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xC1} & \quad\Rightarrow\quad{} & \I32 {.} {\EXTEND}{\mathsf{{\scriptstyle 16}}}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xC2} & \quad\Rightarrow\quad{} & \I64 {.} {\EXTEND}{\mathsf{{\scriptstyle 8}}}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xC3} & \quad\Rightarrow\quad{} & \I64 {.} {\EXTEND}{\mathsf{{\scriptstyle 16}}}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xC4} & \quad\Rightarrow\quad{} & \I64 {.} {\EXTEND}{\mathsf{{\scriptstyle 32}}}{\mathsf{\_}}{\S} \\
   \end{array}

.. _binary-cvtop-trunc-sat:

The saturating truncation instructions all have a one byte prefix,
whereas the actual opcode is encoded by a variable-length :ref:`unsigned integer <binary-uint>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Binstr} & ::= & \dots \\
   & & | & \mathtt{0xFC}~~0{:}{\Bu32} & \quad\Rightarrow\quad{} & \I32 {.} {{\TRUNCSAT}{\mathsf{\_}}{\S}}{\mathsf{\_}}{\F32} \\
   & & | & \mathtt{0xFC}~~1{:}{\Bu32} & \quad\Rightarrow\quad{} & \I32 {.} {{\TRUNCSAT}{\mathsf{\_}}{\U}}{\mathsf{\_}}{\F32} \\
   & & | & \mathtt{0xFC}~~2{:}{\Bu32} & \quad\Rightarrow\quad{} & \I32 {.} {{\TRUNCSAT}{\mathsf{\_}}{\S}}{\mathsf{\_}}{\F64} \\
   & & | & \mathtt{0xFC}~~3{:}{\Bu32} & \quad\Rightarrow\quad{} & \I32 {.} {{\TRUNCSAT}{\mathsf{\_}}{\U}}{\mathsf{\_}}{\F64} \\
   & & | & \mathtt{0xFC}~~4{:}{\Bu32} & \quad\Rightarrow\quad{} & \I64 {.} {{\TRUNCSAT}{\mathsf{\_}}{\S}}{\mathsf{\_}}{\F32} \\
   & & | & \mathtt{0xFC}~~5{:}{\Bu32} & \quad\Rightarrow\quad{} & \I64 {.} {{\TRUNCSAT}{\mathsf{\_}}{\U}}{\mathsf{\_}}{\F32} \\
   & & | & \mathtt{0xFC}~~6{:}{\Bu32} & \quad\Rightarrow\quad{} & \I64 {.} {{\TRUNCSAT}{\mathsf{\_}}{\S}}{\mathsf{\_}}{\F64} \\
   & & | & \mathtt{0xFC}~~7{:}{\Bu32} & \quad\Rightarrow\quad{} & \I64 {.} {{\TRUNCSAT}{\mathsf{\_}}{\U}}{\mathsf{\_}}{\F64} \\
   \end{array}


.. index:: vector instruction
   pair: binary format; instruction
.. _binary-instr-vec:

Vector Instructions
~~~~~~~~~~~~~~~~~~~

All variants of :ref:`vector instructions <syntax-instr-vec>` are represented by separate byte codes.
They all have a one byte prefix, whereas the actual opcode is encoded by a variable-length :ref:`unsigned integer <binary-uint>`.

Vector loads and stores are followed by the encoding of their :math:`{\memarg}` immediate.

.. _binary-laneidx:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Blaneidx} & ::= & l{:}{\Bbyte} & \quad\Rightarrow\quad{} & l \\[0.8ex]
   & {\Binstr} & ::= & \dots \\
   & & | & \mathtt{0xFD}~~0{:}{\Bu32}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & \V128{.}\VLOAD~x~{\mathit{ao}} \\
   & & | & \mathtt{0xFD}~~1{:}{\Bu32}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{{\mathsf{{\scriptstyle 8}}}{\Xshape}{\mathsf{{\scriptstyle 8}}}{\mathsf{\_}}{\S}}~x~{\mathit{ao}} \\
   & & | & \mathtt{0xFD}~~2{:}{\Bu32}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{{\mathsf{{\scriptstyle 8}}}{\Xshape}{\mathsf{{\scriptstyle 8}}}{\mathsf{\_}}{\U}}~x~{\mathit{ao}} \\
   & & | & \mathtt{0xFD}~~3{:}{\Bu32}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{{\mathsf{{\scriptstyle 16}}}{\Xshape}{\mathsf{{\scriptstyle 4}}}{\mathsf{\_}}{\S}}~x~{\mathit{ao}} \\
   & & | & \mathtt{0xFD}~~4{:}{\Bu32}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{{\mathsf{{\scriptstyle 16}}}{\Xshape}{\mathsf{{\scriptstyle 4}}}{\mathsf{\_}}{\U}}~x~{\mathit{ao}} \\
   & & | & \mathtt{0xFD}~~5{:}{\Bu32}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{{\mathsf{{\scriptstyle 32}}}{\Xshape}{\mathsf{{\scriptstyle 2}}}{\mathsf{\_}}{\S}}~x~{\mathit{ao}} \\
   & & | & \mathtt{0xFD}~~6{:}{\Bu32}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{{\mathsf{{\scriptstyle 32}}}{\Xshape}{\mathsf{{\scriptstyle 2}}}{\mathsf{\_}}{\U}}~x~{\mathit{ao}} \\
   & & | & \mathtt{0xFD}~~7{:}{\Bu32}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{{\mathsf{{\scriptstyle 8}}}{\mathsf{\_}}{\LSPLAT}}~x~{\mathit{ao}} \\
   & & | & \mathtt{0xFD}~~8{:}{\Bu32}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{{\mathsf{{\scriptstyle 16}}}{\mathsf{\_}}{\LSPLAT}}~x~{\mathit{ao}} \\
   & & | & \mathtt{0xFD}~~9{:}{\Bu32}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{{\mathsf{{\scriptstyle 32}}}{\mathsf{\_}}{\LSPLAT}}~x~{\mathit{ao}} \\
   & & | & \mathtt{0xFD}~~10{:}{\Bu32}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{{\mathsf{{\scriptstyle 64}}}{\mathsf{\_}}{\LSPLAT}}~x~{\mathit{ao}} \\
   & & | & \mathtt{0xFD}~~11{:}{\Bu32}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & \V128{.}\VSTORE~x~{\mathit{ao}} \\
   & & | & \mathtt{0xFD}~~84{:}{\Bu32}~~(x, {\mathit{ao}}){:}{\Bmemarg}~~i{:}{\Blaneidx} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{\mathsf{{\scriptstyle 8}}}{\mathsf{\_}}{\VLANE}~x~{\mathit{ao}}~i \\
   & & | & \mathtt{0xFD}~~85{:}{\Bu32}~~(x, {\mathit{ao}}){:}{\Bmemarg}~~i{:}{\Blaneidx} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{\mathsf{{\scriptstyle 16}}}{\mathsf{\_}}{\VLANE}~x~{\mathit{ao}}~i \\
   & & | & \mathtt{0xFD}~~86{:}{\Bu32}~~(x, {\mathit{ao}}){:}{\Bmemarg}~~i{:}{\Blaneidx} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{\mathsf{{\scriptstyle 32}}}{\mathsf{\_}}{\VLANE}~x~{\mathit{ao}}~i \\
   & & | & \mathtt{0xFD}~~87{:}{\Bu32}~~(x, {\mathit{ao}}){:}{\Bmemarg}~~i{:}{\Blaneidx} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{\mathsf{{\scriptstyle 64}}}{\mathsf{\_}}{\VLANE}~x~{\mathit{ao}}~i \\
   & & | & \mathtt{0xFD}~~88{:}{\Bu32}~~(x, {\mathit{ao}}){:}{\Bmemarg}~~i{:}{\Blaneidx} & \quad\Rightarrow\quad{} & {\V128{.}\VSTORE}{\mathsf{{\scriptstyle 8}}}{\mathsf{\_}}{\VLANE}~x~{\mathit{ao}}~i \\
   & & | & \mathtt{0xFD}~~89{:}{\Bu32}~~(x, {\mathit{ao}}){:}{\Bmemarg}~~i{:}{\Blaneidx} & \quad\Rightarrow\quad{} & {\V128{.}\VSTORE}{\mathsf{{\scriptstyle 16}}}{\mathsf{\_}}{\VLANE}~x~{\mathit{ao}}~i \\
   & & | & \mathtt{0xFD}~~90{:}{\Bu32}~~(x, {\mathit{ao}}){:}{\Bmemarg}~~i{:}{\Blaneidx} & \quad\Rightarrow\quad{} & {\V128{.}\VSTORE}{\mathsf{{\scriptstyle 32}}}{\mathsf{\_}}{\VLANE}~x~{\mathit{ao}}~i \\
   & & | & \mathtt{0xFD}~~91{:}{\Bu32}~~(x, {\mathit{ao}}){:}{\Bmemarg}~~i{:}{\Blaneidx} & \quad\Rightarrow\quad{} & {\V128{.}\VSTORE}{\mathsf{{\scriptstyle 64}}}{\mathsf{\_}}{\VLANE}~x~{\mathit{ao}}~i \\
   & & | & \mathtt{0xFD}~~92{:}{\Bu32}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{{\mathsf{{\scriptstyle 32}}}{\mathsf{\_}}{\LZERO}}~x~{\mathit{ao}} \\
   & & | & \mathtt{0xFD}~~93{:}{\Bu32}~~(x, {\mathit{ao}}){:}{\Bmemarg} & \quad\Rightarrow\quad{} & {\V128{.}\VLOAD}{{\mathsf{{\scriptstyle 64}}}{\mathsf{\_}}{\LZERO}}~x~{\mathit{ao}} \\
   \end{array}

The :math:`\mathsf{const}` instruction for vectors is followed by 16 immediate bytes, which are converted into an :math:`{\mathit{u{\kern-0.1em\scriptstyle 128}}}` in |littleendian| byte order:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Binstr} & ::= & \dots \\
   & & | & \mathtt{0xFD}~~12{:}{\Bu32}~~{(b{:}{\Bbyte})^{16}} & \quad\Rightarrow\quad{} & \V128{.}\CONST~{{{{\bytes}}_{{\INX}{\mathsf{{\scriptstyle 128}}}}^{{-1}}}}{({(b)^{16}})} \\
   \end{array}

.. _binary-vswizzlop:
.. _binary-vshuffle:

The :math:`\mathsf{shuffle}` instruction is also followed by the encoding of 16 :math:`{\laneidx}` immediates.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Binstr} & ::= & \dots \\
   & & | & \mathtt{0xFD}~~13{:}{\Bu32}~~{(l{:}{\Blaneidx})^{16}} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}{.}\VSHUFFLE~{l^{16}} \\
   & & | & \mathtt{0xFD}~~14{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} \VSWIZZLE \\
   & & | & \mathtt{0xFD}~~256{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} \VRELAXEDSWIZZLE \\
   \end{array}

Lane instructions are followed by the encoding of a :math:`{\laneidx}` immediate.

.. _binary-vextract_lane:
.. _binary-vreplace_lane:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Binstr} & ::= & \dots \\
   & & | & \mathtt{0xFD}~~21{:}{\Bu32}~~l{:}{\Blaneidx} & \quad\Rightarrow\quad{} & {{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}{.}\VEXTRACTLANE}{\mathsf{\_}}{\S}~l \\
   & & | & \mathtt{0xFD}~~22{:}{\Bu32}~~l{:}{\Blaneidx} & \quad\Rightarrow\quad{} & {{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}{.}\VEXTRACTLANE}{\mathsf{\_}}{\U}~l \\
   & & | & \mathtt{0xFD}~~23{:}{\Bu32}~~l{:}{\Blaneidx} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}{.}\VREPLACELANE~l \\
   & & | & \mathtt{0xFD}~~24{:}{\Bu32}~~l{:}{\Blaneidx} & \quad\Rightarrow\quad{} & {{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}{.}\VEXTRACTLANE}{\mathsf{\_}}{\S}~l \\
   & & | & \mathtt{0xFD}~~25{:}{\Bu32}~~l{:}{\Blaneidx} & \quad\Rightarrow\quad{} & {{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}{.}\VEXTRACTLANE}{\mathsf{\_}}{\U}~l \\
   & & | & \mathtt{0xFD}~~26{:}{\Bu32}~~l{:}{\Blaneidx} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}{.}\VREPLACELANE~l \\
   & & | & \mathtt{0xFD}~~27{:}{\Bu32}~~l{:}{\Blaneidx} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}{.}\VEXTRACTLANE~l \\
   & & | & \mathtt{0xFD}~~28{:}{\Bu32}~~l{:}{\Blaneidx} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}{.}\VREPLACELANE~l \\
   & & | & \mathtt{0xFD}~~29{:}{\Bu32}~~l{:}{\Blaneidx} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}}{.}\VEXTRACTLANE~l \\
   & & | & \mathtt{0xFD}~~30{:}{\Bu32}~~l{:}{\Blaneidx} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}}{.}\VREPLACELANE~l \\
   & & | & \mathtt{0xFD}~~31{:}{\Bu32}~~l{:}{\Blaneidx} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}}{.}\VEXTRACTLANE~l \\
   & & | & \mathtt{0xFD}~~32{:}{\Bu32}~~l{:}{\Blaneidx} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}}{.}\VREPLACELANE~l \\
   & & | & \mathtt{0xFD}~~33{:}{\Bu32}~~l{:}{\Blaneidx} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}}{.}\VEXTRACTLANE~l \\
   & & | & \mathtt{0xFD}~~34{:}{\Bu32}~~l{:}{\Blaneidx} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}}{.}\VREPLACELANE~l \\
   \end{array}

All other vector instructions are plain opcodes without any immediates.

.. _binary-vsplat:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Binstr} & ::= & \dots \\
   & & | & \mathtt{0xFD}~~15{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}{.}\VSPLAT \\
   & & | & \mathtt{0xFD}~~16{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}{.}\VSPLAT \\
   & & | & \mathtt{0xFD}~~17{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}{.}\VSPLAT \\
   & & | & \mathtt{0xFD}~~18{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}}{.}\VSPLAT \\
   & & | & \mathtt{0xFD}~~19{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}}{.}\VSPLAT \\
   & & | & \mathtt{0xFD}~~20{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}}{.}\VSPLAT \\
   \end{array}

.. _binary-virelop:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Binstr} & ::= & \dots \\
   & & | & \mathtt{0xFD}~~35{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} \VEQ \\
   & & | & \mathtt{0xFD}~~36{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} \VNE \\
   & & | & \mathtt{0xFD}~~37{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VLT}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~38{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VLT}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0xFD}~~39{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VGT}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~40{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VGT}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0xFD}~~41{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VLE}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~42{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VLE}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0xFD}~~43{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VGE}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~44{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VGE}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0xFD}~~45{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} \VEQ \\
   & & | & \mathtt{0xFD}~~46{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} \VNE \\
   & & | & \mathtt{0xFD}~~47{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VLT}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~48{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VLT}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0xFD}~~49{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VGT}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~50{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VGT}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0xFD}~~51{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VLE}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~52{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VLE}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0xFD}~~53{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VGE}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~54{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VGE}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0xFD}~~55{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VEQ \\
   & & | & \mathtt{0xFD}~~56{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VNE \\
   & & | & \mathtt{0xFD}~~57{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {\VLT}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~58{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {\VLT}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0xFD}~~59{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {\VGT}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~60{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {\VGT}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0xFD}~~61{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {\VLE}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~62{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {\VLE}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0xFD}~~63{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {\VGE}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~64{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {\VGE}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0xFD}~~214{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VEQ \\
   & & | & \mathtt{0xFD}~~215{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VNE \\
   & & | & \mathtt{0xFD}~~216{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {\VLT}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~217{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {\VGT}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~218{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {\VLE}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~219{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {\VGE}{\mathsf{\_}}{\S} \\
   \end{array}

.. _binary-vfrelop:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Binstr} & ::= & \dots \\
   & & | & \mathtt{0xFD}~~65{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VEQ \\
   & & | & \mathtt{0xFD}~~66{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VNE \\
   & & | & \mathtt{0xFD}~~67{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VLT \\
   & & | & \mathtt{0xFD}~~68{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VGT \\
   & & | & \mathtt{0xFD}~~69{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VLE \\
   & & | & \mathtt{0xFD}~~70{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VGE \\
   & & | & \mathtt{0xFD}~~71{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VEQ \\
   & & | & \mathtt{0xFD}~~72{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VNE \\
   & & | & \mathtt{0xFD}~~73{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VLT \\
   & & | & \mathtt{0xFD}~~74{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VGT \\
   & & | & \mathtt{0xFD}~~75{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VLE \\
   & & | & \mathtt{0xFD}~~76{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VGE \\
   \end{array}

.. _binary-vvunop:
.. _binary-vvbinop:
.. _binary-vvternop:
.. _binary-vvtestop:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Binstr} & ::= & \dots \\
   & & | & \mathtt{0xFD}~~77{:}{\Bu32} & \quad\Rightarrow\quad{} & \V128 {.} \VNOT \\
   & & | & \mathtt{0xFD}~~78{:}{\Bu32} & \quad\Rightarrow\quad{} & \V128 {.} \VAND \\
   & & | & \mathtt{0xFD}~~79{:}{\Bu32} & \quad\Rightarrow\quad{} & \V128 {.} \VANDNOT \\
   & & | & \mathtt{0xFD}~~80{:}{\Bu32} & \quad\Rightarrow\quad{} & \V128 {.} \VOR \\
   & & | & \mathtt{0xFD}~~81{:}{\Bu32} & \quad\Rightarrow\quad{} & \V128 {.} \VXOR \\
   & & | & \mathtt{0xFD}~~82{:}{\Bu32} & \quad\Rightarrow\quad{} & \V128 {.} \VBITSELECT \\
   & & | & \mathtt{0xFD}~~83{:}{\Bu32} & \quad\Rightarrow\quad{} & \V128 {.} \VANYTRUE \\
   \end{array}

.. _binary-vitestop:
.. _binary-vshiftop:
.. _binary-viunop:
.. _binary-vibinop:
.. _binary-viternop:
.. _binary-viextunop:
.. _binary-viextbinop:
.. _binary-viextternop:
.. _binary-viminmaxop:
.. _binary-vsatbinop:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Binstr} & ::= & \dots \\
   & & | & \mathtt{0xFD}~~96{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} \VABS \\
   & & | & \mathtt{0xFD}~~97{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} \VNEG \\
   & & | & \mathtt{0xFD}~~98{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} \VPOPCNT \\
   & & | & \mathtt{0xFD}~~99{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} \VALLTRUE \\
   & & | & \mathtt{0xFD}~~100{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}{.}\VBITMASK \\
   & & | & \mathtt{0xFD}~~101{:}{\Bu32} & \quad\Rightarrow\quad{} & {{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}{.}\VNARROW}{\mathsf{\_}}{{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~102{:}{\Bu32} & \quad\Rightarrow\quad{} & {{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}{.}\VNARROW}{\mathsf{\_}}{{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0xFD}~~107{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} \VSHL \\
   & & | & \mathtt{0xFD}~~108{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VSHR}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~109{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VSHR}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0xFD}~~110{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} \VADD \\
   & & | & \mathtt{0xFD}~~111{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VADDSAT}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~112{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VADDSAT}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0xFD}~~113{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} \VSUB \\
   & & | & \mathtt{0xFD}~~114{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VSUBSAT}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~115{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VSUBSAT}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0xFD}~~118{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VMIN}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~119{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VMIN}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0xFD}~~120{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VMAX}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~121{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VMAX}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0xFD}~~123{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} {\VAVGR}{\mathsf{\_}}{\VU} \\
   \end{array}

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Binstr} & ::= & \dots \\
   & & | & \mathtt{0xFD}~~124{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {{\VEXTADDPAIRWISE}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}} \\
   & & | & \mathtt{0xFD}~~125{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {{\VEXTADDPAIRWISE}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}} \\
   & & | & \mathtt{0xFD}~~128{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} \VABS \\
   & & | & \mathtt{0xFD}~~129{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} \VNEG \\
   & & | & \mathtt{0xFD}~~131{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} \VALLTRUE \\
   & & | & \mathtt{0xFD}~~132{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}{.}\VBITMASK \\
   & & | & \mathtt{0xFD}~~133{:}{\Bu32} & \quad\Rightarrow\quad{} & {{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}{.}\VNARROW}{\mathsf{\_}}{{\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~134{:}{\Bu32} & \quad\Rightarrow\quad{} & {{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}{.}\VNARROW}{\mathsf{\_}}{{\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0xFD}~~135{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {{\VEXTEND}{\mathsf{\_}}{\LOW}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}} \\
   & & | & \mathtt{0xFD}~~136{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {{\VEXTEND}{\mathsf{\_}}{\HIGH}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}} \\
   & & | & \mathtt{0xFD}~~137{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {{\VEXTEND}{\mathsf{\_}}{\LOW}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}} \\
   & & | & \mathtt{0xFD}~~138{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {{\VEXTEND}{\mathsf{\_}}{\HIGH}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}} \\
   & & | & \mathtt{0xFD}~~139{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} \VSHL \\
   & & | & \mathtt{0xFD}~~140{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VSHR}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~141{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VSHR}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0xFD}~~130{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VQ15MULRSAT}{\mathsf{\_}}{\VS} \\
   & & | & \mathtt{0xFD}~~142{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} \VADD \\
   & & | & \mathtt{0xFD}~~143{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VADDSAT}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~144{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VADDSAT}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0xFD}~~145{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} \VSUB \\
   & & | & \mathtt{0xFD}~~146{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VSUBSAT}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~147{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VSUBSAT}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0xFD}~~149{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} \VMUL \\
   & & | & \mathtt{0xFD}~~150{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VMIN}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~151{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VMIN}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0xFD}~~152{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VMAX}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~153{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VMAX}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0xFD}~~155{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VAVGR}{\mathsf{\_}}{\VU} \\
   & & | & \mathtt{0xFD}~~273{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {\VRELAXEDQ15MULR}{\mathsf{\_}}{\VS} \\
   & & | & \mathtt{0xFD}~~156{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {{\VEXTMUL}{\mathsf{\_}}{\LOW}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}} \\
   & & | & \mathtt{0xFD}~~157{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {{\VEXTMUL}{\mathsf{\_}}{\HIGH}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}} \\
   & & | & \mathtt{0xFD}~~158{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {{\VEXTMUL}{\mathsf{\_}}{\LOW}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}} \\
   & & | & \mathtt{0xFD}~~159{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {{\VEXTMUL}{\mathsf{\_}}{\HIGH}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}} \\
   & & | & \mathtt{0xFD}~~274{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} {{\VRELAXEDDOT}{\mathsf{\_}}{\VS}}{\mathsf{\_}}{{\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}}} \\
   \end{array}

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Binstr} & ::= & \dots \\
   & & | & \mathtt{0xFD}~~126{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VEXTADDPAIRWISE}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}} \\
   & & | & \mathtt{0xFD}~~127{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VEXTADDPAIRWISE}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}} \\
   & & | & \mathtt{0xFD}~~160{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VABS \\
   & & | & \mathtt{0xFD}~~161{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VNEG \\
   & & | & \mathtt{0xFD}~~163{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VALLTRUE \\
   & & | & \mathtt{0xFD}~~164{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}{.}\VBITMASK \\
   & & | & \mathtt{0xFD}~~167{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VEXTEND}{\mathsf{\_}}{\LOW}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}} \\
   & & | & \mathtt{0xFD}~~168{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VEXTEND}{\mathsf{\_}}{\HIGH}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}} \\
   & & | & \mathtt{0xFD}~~169{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VEXTEND}{\mathsf{\_}}{\LOW}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}} \\
   & & | & \mathtt{0xFD}~~170{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VEXTEND}{\mathsf{\_}}{\HIGH}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}} \\
   & & | & \mathtt{0xFD}~~171{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VSHL \\
   & & | & \mathtt{0xFD}~~172{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {\VSHR}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~173{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {\VSHR}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0xFD}~~174{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VADD \\
   & & | & \mathtt{0xFD}~~177{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VSUB \\
   & & | & \mathtt{0xFD}~~181{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VMUL \\
   & & | & \mathtt{0xFD}~~182{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {\VMIN}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~183{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {\VMIN}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0xFD}~~184{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {\VMAX}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~185{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {\VMAX}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0xFD}~~186{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VDOT}{\mathsf{\_}}{\VS}}{\mathsf{\_}}{{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}} \\
   & & | & \mathtt{0xFD}~~188{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VEXTMUL}{\mathsf{\_}}{\LOW}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}} \\
   & & | & \mathtt{0xFD}~~189{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VEXTMUL}{\mathsf{\_}}{\HIGH}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}} \\
   & & | & \mathtt{0xFD}~~190{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VEXTMUL}{\mathsf{\_}}{\LOW}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}} \\
   & & | & \mathtt{0xFD}~~191{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VEXTMUL}{\mathsf{\_}}{\HIGH}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}} \\
   & & | & \mathtt{0xFD}~~275{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VRELAXEDDOTADD}{\mathsf{\_}}{\VS}}{\mathsf{\_}}{{\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}}} \\
   \end{array}

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Binstr} & ::= & \dots \\
   & & | & \mathtt{0xFD}~~192{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VABS \\
   & & | & \mathtt{0xFD}~~193{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VNEG \\
   & & | & \mathtt{0xFD}~~195{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VALLTRUE \\
   & & | & \mathtt{0xFD}~~196{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}}{.}\VBITMASK \\
   & & | & \mathtt{0xFD}~~199{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {{\VEXTEND}{\mathsf{\_}}{\LOW}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mathtt{0xFD}~~200{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {{\VEXTEND}{\mathsf{\_}}{\HIGH}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mathtt{0xFD}~~201{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {{\VEXTEND}{\mathsf{\_}}{\LOW}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mathtt{0xFD}~~202{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {{\VEXTEND}{\mathsf{\_}}{\HIGH}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mathtt{0xFD}~~203{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VSHL \\
   & & | & \mathtt{0xFD}~~204{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {\VSHR}{\mathsf{\_}}{\S} \\
   & & | & \mathtt{0xFD}~~205{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {\VSHR}{\mathsf{\_}}{\U} \\
   & & | & \mathtt{0xFD}~~206{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VADD \\
   & & | & \mathtt{0xFD}~~209{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VSUB \\
   & & | & \mathtt{0xFD}~~213{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VMUL \\
   & & | & \mathtt{0xFD}~~220{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {{\VEXTMUL}{\mathsf{\_}}{\LOW}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mathtt{0xFD}~~221{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {{\VEXTMUL}{\mathsf{\_}}{\HIGH}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mathtt{0xFD}~~222{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {{\VEXTMUL}{\mathsf{\_}}{\LOW}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mathtt{0xFD}~~223{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {{\VEXTMUL}{\mathsf{\_}}{\HIGH}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   \end{array}

.. _binary-vfunop:
.. _binary-vfbinop:
.. _binary-vfternop:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Binstr} & ::= & \dots \\
   & & | & \mathtt{0xFD}~~103{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VCEIL \\
   & & | & \mathtt{0xFD}~~104{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VFLOOR \\
   & & | & \mathtt{0xFD}~~105{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VTRUNC \\
   & & | & \mathtt{0xFD}~~106{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VNEAREST \\
   & & | & \mathtt{0xFD}~~224{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VABS \\
   & & | & \mathtt{0xFD}~~225{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VNEG \\
   & & | & \mathtt{0xFD}~~227{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VSQRT \\
   & & | & \mathtt{0xFD}~~228{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VADD \\
   & & | & \mathtt{0xFD}~~229{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VSUB \\
   & & | & \mathtt{0xFD}~~230{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VMUL \\
   & & | & \mathtt{0xFD}~~231{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VDIV \\
   & & | & \mathtt{0xFD}~~232{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VMIN \\
   & & | & \mathtt{0xFD}~~233{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VMAX \\
   & & | & \mathtt{0xFD}~~234{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VPMIN \\
   & & | & \mathtt{0xFD}~~235{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VPMAX \\
   & & | & \mathtt{0xFD}~~269{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VRELAXEDMIN \\
   & & | & \mathtt{0xFD}~~270{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VRELAXEDMAX \\
   & & | & \mathtt{0xFD}~~261{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VRELAXEDMADD \\
   & & | & \mathtt{0xFD}~~262{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VRELAXEDNMADD \\
   \end{array}

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Binstr} & ::= & \dots \\
   & & | & \mathtt{0xFD}~~116{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VCEIL \\
   & & | & \mathtt{0xFD}~~117{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VFLOOR \\
   & & | & \mathtt{0xFD}~~122{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VTRUNC \\
   & & | & \mathtt{0xFD}~~148{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VNEAREST \\
   & & | & \mathtt{0xFD}~~236{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VABS \\
   & & | & \mathtt{0xFD}~~237{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VNEG \\
   & & | & \mathtt{0xFD}~~239{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VSQRT \\
   & & | & \mathtt{0xFD}~~240{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VADD \\
   & & | & \mathtt{0xFD}~~241{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VSUB \\
   & & | & \mathtt{0xFD}~~242{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VMUL \\
   & & | & \mathtt{0xFD}~~243{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VDIV \\
   & & | & \mathtt{0xFD}~~244{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VMIN \\
   & & | & \mathtt{0xFD}~~245{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VMAX \\
   & & | & \mathtt{0xFD}~~246{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VPMIN \\
   & & | & \mathtt{0xFD}~~247{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VPMAX \\
   & & | & \mathtt{0xFD}~~271{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VRELAXEDMIN \\
   & & | & \mathtt{0xFD}~~272{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VRELAXEDMAX \\
   & & | & \mathtt{0xFD}~~263{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VRELAXEDMADD \\
   & & | & \mathtt{0xFD}~~264{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VRELAXEDNMADD \\
   & & | & \mathtt{0xFD}~~265{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I8}{\Xshape}{\mathsf{{\scriptstyle 16}}} {.} \VRELAXEDLANESELECT \\
   & & | & \mathtt{0xFD}~~266{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I16}{\Xshape}{\mathsf{{\scriptstyle 8}}} {.} \VRELAXEDLANESELECT \\
   & & | & \mathtt{0xFD}~~267{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} \VRELAXEDLANESELECT \\
   & & | & \mathtt{0xFD}~~268{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} \VRELAXEDLANESELECT \\
   \end{array}

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Binstr} & ::= & \dots \\
   & & | & \mathtt{0xFD}~~94{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VDEMOTE}{\mathsf{\_}}{\VZERO}}{\mathsf{\_}}{{\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}}} \\
   & & | & \mathtt{0xFD}~~95{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {{\VPROMOTE}{\mathsf{\_}}{\VLOW}}{\mathsf{\_}}{{\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mathtt{0xFD}~~248{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VTRUNCSAT}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mathtt{0xFD}~~249{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VTRUNCSAT}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mathtt{0xFD}~~250{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VCONVERT}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mathtt{0xFD}~~251{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VCONVERT}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mathtt{0xFD}~~252{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VTRUNCSAT}{\mathsf{\_}}{\S}{\mathsf{\_}}{\ZERO}}{\mathsf{\_}}{{\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}}} \\
   & & | & \mathtt{0xFD}~~253{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VTRUNCSAT}{\mathsf{\_}}{\U}{\mathsf{\_}}{\ZERO}}{\mathsf{\_}}{{\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}}} \\
   & & | & \mathtt{0xFD}~~254{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {{\VCONVERT}{\mathsf{\_}}{\LOW}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mathtt{0xFD}~~255{:}{\Bu32} & \quad\Rightarrow\quad{} & {\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}} {.} {{\VCONVERT}{\mathsf{\_}}{\LOW}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mathtt{0xFD}~~257{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VRELAXEDTRUNC}{\mathsf{\_}}{\S}}{\mathsf{\_}}{{\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mathtt{0xFD}~~258{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VRELAXEDTRUNC}{\mathsf{\_}}{\U}}{\mathsf{\_}}{{\F32}{\Xshape}{\mathsf{{\scriptstyle 4}}}} \\
   & & | & \mathtt{0xFD}~~259{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VRELAXEDTRUNC}{\mathsf{\_}}{\S}{\mathsf{\_}}{\ZERO}}{\mathsf{\_}}{{\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}}} \\
   & & | & \mathtt{0xFD}~~260{:}{\Bu32} & \quad\Rightarrow\quad{} & {\I32}{\Xshape}{\mathsf{{\scriptstyle 4}}} {.} {{\VRELAXEDTRUNC}{\mathsf{\_}}{\U}{\mathsf{\_}}{\ZERO}}{\mathsf{\_}}{{\F64}{\Xshape}{\mathsf{{\scriptstyle 2}}}} \\
   \end{array}


.. index:: expression
   pair: binary format; expression
   single: expression; constant
.. _binary-expr:

Expressions
~~~~~~~~~~~

:ref:`Expressions <syntax-expr>` are encoded by their instruction sequence terminated with an explicit :math:`\mathtt{0x0B}` opcode for :math:`\mathsf{end}`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Bexpr} & ::= & {({\mathit{in}}{:}{\Binstr})^\ast}~~\mathtt{0x0B} & \quad\Rightarrow\quad{} & {{\mathit{in}}^\ast} \\
   \end{array}
