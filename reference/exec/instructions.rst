.. index:: instruction, function type, store, validation
.. _exec-instr:

Instructions
------------

WebAssembly computation is performed by executing individual :ref:`instructions <syntax-instr>`.


.. index:: parametric instruction, value
   pair: execution; instruction
   single: abstract syntax; instruction
.. _exec-instr-parametric:

Parametric Instructions
~~~~~~~~~~~~~~~~~~~~~~~

.. _exec-nop:


:math:`\NOP`
............


1. Do nothing.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & \NOP & \stepto & \epsilon \\
   \end{array}


.. _exec-unreachable:


:math:`\UNREACHABLE`
....................


1. Trap.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & \UNREACHABLE & \stepto & \TRAP \\
   \end{array}


.. _exec-drop:


:math:`\DROP`
.............


1. Assert: Due to :ref:`validation <valid-drop>`, a value is on the top of the stack.

#. Pop the value :math:`{\val}` from the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & {\val}~\DROP & \stepto & \epsilon \\
   \end{array}


.. _exec-select:


:math:`\SELECT~{({t^\ast})^?}`
..............................


1. Assert: Due to :ref:`validation <valid-select>`, a value of :ref:`number type <syntax-numtype>` :math:`\I32` is on the top of the stack.

#. Pop the value :math:`(\I32{.}\CONST~c)` from the stack.

#. Assert: Due to :ref:`validation <valid-select>`, a value is on the top of the stack.

#. Pop the value :math:`{\val}_2` from the stack.

#. Assert: Due to :ref:`validation <valid-select>`, a value is on the top of the stack.

#. Pop the value :math:`{\val}_1` from the stack.

#. If :math:`c \neq 0`, then:

   a. Push the value :math:`{\val}_1` to the stack.

#. Else:

   a. Push the value :math:`{\val}_2` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & {\val}_1~{\val}_2~(\I32{.}\CONST~c)~(\SELECT~{({t^\ast})^?}) & \stepto & {\val}_1 & \quad \mbox{if}~ c \neq 0 \\
   & {\val}_1~{\val}_2~(\I32{.}\CONST~c)~(\SELECT~{({t^\ast})^?}) & \stepto & {\val}_2 & \quad \mbox{if}~ c = 0 \\
   \end{array}

.. note::
   In future versions of WebAssembly, :math:`\mathsf{select}` may allow more than one value per choice.


.. index:: control instructions, structured control, label, block, branch, result type, label index, function index, type index, list, address, table address, table instance, store, frame
   pair: execution; instruction
   single: abstract syntax; instruction
.. _exec-label:
.. _exec-instr-control:

Control Instructions
~~~~~~~~~~~~~~~~~~~~

.. _exec-block:


:math:`\BLOCK~{\mathit{bt}}~{{\instr}^\ast}`
............................................


1. Let :math:`z` be the current state.

#. Let :math:`{t_1^{m}}~{\to}_{{{\localidx}_0^\ast}}\,{t_2^{n}}` be the destructuring of :math:`{{\fblocktype}}_{z}({\mathit{bt}})`.

#. Assert: Due to :ref:`validation <valid-block>`, :math:`{{\localidx}_0^\ast} = \epsilon`.

#. Assert: Due to :ref:`validation <valid-block>`, there are at least :math:`m` values on the top of the stack.

#. Pop the values :math:`{{\val}^{m}}` from the stack.

#. Let :math:`L` be the :math:`\LABEL` whose arity is :math:`n` and whose continuation is the end of the block.

#. Enter the block :math:`{{\val}^{m}}~{{\instr}^\ast}` with the :math:`\LABEL` :math:`L`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; {{\val}^{m}}~(\BLOCK~{\mathit{bt}}~{{\instr}^\ast}) & \stepto & ({{\LABEL}_{n}}{\{ \epsilon \}}~{{\val}^{m}}~{{\instr}^\ast}) & \quad \mbox{if}~ {{\fblocktype}}_{z}({\mathit{bt}}) = {t_1^{m}} \rightarrow {t_2^{n}} \\
   \end{array}


.. _exec-loop:


:math:`\LOOP~{\mathit{bt}}~{{\instr}^\ast}`
...........................................


1. Let :math:`z` be the current state.

#. Let :math:`{t_1^{m}}~{\to}_{{{\localidx}_0^\ast}}\,{t_2^{n}}` be the destructuring of :math:`{{\fblocktype}}_{z}({\mathit{bt}})`.

#. Assert: Due to :ref:`validation <valid-loop>`, :math:`{{\localidx}_0^\ast} = \epsilon`.

#. Assert: Due to :ref:`validation <valid-loop>`, there are at least :math:`m` values on the top of the stack.

#. Pop the values :math:`{{\val}^{m}}` from the stack.

#. Let :math:`L` be the :math:`\LABEL` whose arity is :math:`m` and whose continuation is the start of the block.

#. Enter the block :math:`{{\val}^{m}}~{{\instr}^\ast}` with the :math:`\LABEL` :math:`L`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; {{\val}^{m}}~(\LOOP~{\mathit{bt}}~{{\instr}^\ast}) & \stepto & ({{\LABEL}_{m}}{\{ \LOOP~{\mathit{bt}}~{{\instr}^\ast} \}}~{{\val}^{m}}~{{\instr}^\ast}) & \quad \mbox{if}~ {{\fblocktype}}_{z}({\mathit{bt}}) = {t_1^{m}} \rightarrow {t_2^{n}} \\
   \end{array}


.. _exec-if:


:math:`\IF~{\mathit{bt}}~{{\instr}_1^\ast}~\ELSE~{{\instr}_2^\ast}`
...................................................................


1. Assert: Due to validation, a value of :ref:`number type <syntax-numtype>` :math:`\I32` is on the top of the stack.

#. Pop the value :math:`(\I32{.}\CONST~c)` from the stack.

#. If :math:`c \neq 0`, then:

   a. Execute the instruction :math:`(\BLOCK~{\mathit{bt}}~{{\instr}_1^\ast})`.

#. Else:

   a. Execute the instruction :math:`(\BLOCK~{\mathit{bt}}~{{\instr}_2^\ast})`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & (\I32{.}\CONST~c)~(\IF~{\mathit{bt}}~{{\instr}_1^\ast}~\ELSE~{{\instr}_2^\ast}) & \stepto & (\BLOCK~{\mathit{bt}}~{{\instr}_1^\ast}) & \quad \mbox{if}~ c \neq 0 \\
   & (\I32{.}\CONST~c)~(\IF~{\mathit{bt}}~{{\instr}_1^\ast}~\ELSE~{{\instr}_2^\ast}) & \stepto & (\BLOCK~{\mathit{bt}}~{{\instr}_2^\ast}) & \quad \mbox{if}~ c = 0 \\
   \end{array}


.. _exec-br:


:math:`\BR~l`
.............


1. If the first non-value entry of the stack is a :math:`\LABEL`, then:

   a. Let :math:`L` be the topmost :math:`\LABEL`.

   #. Let :math:`n` be the arity of :math:`L`

   #. If :math:`l = 0`, then:

      1) Assert: Due to :ref:`validation <valid-br>`, there are at least :math:`n` values on the top of the stack.

      #) Pop the values :math:`{{\val}^{n}}` from the stack.

      #) Pop all values :math:`{{\val'}^\ast}` from the top of the stack.

      #) Pop the :math:`\LABEL` from the stack.

      #) Push the values :math:`{{\val}^{n}}` to the stack.

      #) Jump to the continuation of :math:`L`.

   #. Else:

      1) Pop all values :math:`{{\val}^\ast}` from the top of the stack.

      #) Pop the :math:`\LABEL` from the stack.

      #) Push the values :math:`{{\val}^\ast}` to the stack.

      #) Execute the instruction :math:`(\BR~l - 1)`.

#. Else:

   a. Assert: Due to :ref:`validation <valid-br>`, the first non-value entry of the stack is a :math:`\HANDLER`.

   #. Pop all values :math:`{{\val}^\ast}` from the top of the stack.

   #. Pop the :math:`\HANDLER` from the stack.

   #. Push the values :math:`{{\val}^\ast}` to the stack.

   #. Execute the instruction :math:`(\BR~l)`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & ({{\LABEL}_{n}}{\{ {{\instr'}^\ast} \}}~{{\val'}^\ast}~{{\val}^{n}}~(\BR~l)~{{\instr}^\ast}) & \stepto & {{\val}^{n}}~{{\instr'}^\ast} & \quad \mbox{if}~ l = 0 \\
   & ({{\LABEL}_{n}}{\{ {{\instr'}^\ast} \}}~{{\val}^\ast}~(\BR~l)~{{\instr}^\ast}) & \stepto & {{\val}^\ast}~(\BR~l - 1) & \quad \mbox{if}~ l > 0 \\
   & ({{\HANDLER}_{n}}{\{ {{\catch}^\ast} \}}~{{\val}^\ast}~(\BR~l)~{{\instr}^\ast}) & \stepto & {{\val}^\ast}~(\BR~l) \\
   \end{array}


.. _exec-br_if:


:math:`\BRIF~l`
...............


1. Assert: Due to :ref:`validation <valid-br_if>`, a value of :ref:`number type <syntax-numtype>` :math:`\I32` is on the top of the stack.

#. Pop the value :math:`(\I32{.}\CONST~c)` from the stack.

#. If :math:`c \neq 0`, then:

   a. Execute the instruction :math:`(\BR~l)`.

#. Else:

   a. Do nothing.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & (\I32{.}\CONST~c)~(\BRIF~l) & \stepto & (\BR~l) & \quad \mbox{if}~ c \neq 0 \\
   & (\I32{.}\CONST~c)~(\BRIF~l) & \stepto & \epsilon & \quad \mbox{if}~ c = 0 \\
   \end{array}


.. _exec-br_table:


:math:`\BRTABLE~{l^\ast}~{l'}`
..............................


1. Assert: Due to :ref:`validation <valid-br_table>`, a value of :ref:`number type <syntax-numtype>` :math:`\I32` is on the top of the stack.

#. Pop the value :math:`(\I32{.}\CONST~i)` from the stack.

#. If :math:`i < {|{l^\ast}|}`, then:

   a. Execute the instruction :math:`(\BR~{l^\ast}{}[i])`.

#. Else:

   a. Execute the instruction :math:`(\BR~{l'})`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & (\I32{.}\CONST~i)~(\BRTABLE~{l^\ast}~{l'}) & \stepto & (\BR~{l^\ast}{}[i]) & \quad \mbox{if}~ i < {|{l^\ast}|} \\
   & (\I32{.}\CONST~i)~(\BRTABLE~{l^\ast}~{l'}) & \stepto & (\BR~{l'}) & \quad \mbox{if}~ i \geq {|{l^\ast}|} \\
   \end{array}


.. _exec-br_on_null:


:math:`\BRONNULL~l`
...................


1. Assert: Due to :ref:`validation <valid-br_on_null>`, a value is on the top of the stack.

#. Pop the value :math:`{\val}` from the stack.

#. If :math:`{\val} = \REFNULLADDR`, then:

   a. Execute the instruction :math:`(\BR~l)`.

#. Else:

   a. Push the value :math:`{\val}` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & {\val}~(\BRONNULL~l) & \stepto & (\BR~l) & \quad \mbox{if}~ {\val} = \REFNULLADDR \\
   & {\val}~(\BRONNULL~l) & \stepto & {\val} & \quad \mbox{otherwise} \\
   \end{array}


.. _exec-br_on_non_null:


:math:`\BRONNONNULL~l`
......................


1. Assert: Due to :ref:`validation <valid-br_on_non_null>`, a value is on the top of the stack.

#. Pop the value :math:`{\val}` from the stack.

#. If :math:`{\val} = \REFNULLADDR`, then:

   a. Do nothing.

#. Else:

   a. Push the value :math:`{\val}` to the stack.

   #. Execute the instruction :math:`(\BR~l)`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & {\val}~(\BRONNONNULL~l) & \stepto & \epsilon & \quad \mbox{if}~ {\val} = \REFNULLADDR \\
   & {\val}~(\BRONNONNULL~l) & \stepto & {\val}~(\BR~l) & \quad \mbox{otherwise} \\
   \end{array}


.. _exec-br_on_cast:


:math:`\BRONCAST~l~{\mathit{rt}}_1~{\mathit{rt}}_2`
...................................................


1. Let :math:`f` be the topmost :math:`\mathsf{frame}`.

#. Assert: Due to :ref:`validation <valid-br_on_cast>`, a :ref:`reference value <syntax-ref>` is on the top of the stack.

#. Pop the value :math:`{\reff}` from the stack.

#. Push the value :math:`{\reff}` to the stack.

#. If :math:`{\reff}` is :ref:`valid <valid-val>` with type :math:`{{\insttype}}_{f{.}\AMODULE}({\mathit{rt}}_2)`, then:

   a. Execute the instruction :math:`(\BR~l)`.

#. Else:

   a. Do nothing.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & s ; f ; {\reff}~(\BRONCAST~l~{\mathit{rt}}_1~{\mathit{rt}}_2) & \stepto & {\reff}~(\BR~l) & \quad \mbox{if}~ s \vdashref {\reff} : {{\insttype}}_{f{.}\AMODULE}({\mathit{rt}}_2) \\
   & s ; f ; {\reff}~(\BRONCAST~l~{\mathit{rt}}_1~{\mathit{rt}}_2) & \stepto & {\reff} & \quad \mbox{otherwise} \\
   \end{array}


.. _exec-br_on_cast_fail:


:math:`\BRONCASTFAIL~l~{\mathit{rt}}_1~{\mathit{rt}}_2`
.......................................................


1. Let :math:`f` be the topmost :math:`\mathsf{frame}`.

#. Assert: Due to :ref:`validation <valid-br_on_cast_fail>`, a :ref:`reference value <syntax-ref>` is on the top of the stack.

#. Pop the value :math:`{\reff}` from the stack.

#. Push the value :math:`{\reff}` to the stack.

#. If :math:`{\reff}` is :ref:`valid <valid-val>` with type :math:`{{\insttype}}_{f{.}\AMODULE}({\mathit{rt}}_2)`, then:

   a. Do nothing.

#. Else:

   a. Execute the instruction :math:`(\BR~l)`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & s ; f ; {\reff}~(\BRONCASTFAIL~l~{\mathit{rt}}_1~{\mathit{rt}}_2) & \stepto & {\reff} & \quad \mbox{if}~ s \vdashref {\reff} : {{\insttype}}_{f{.}\AMODULE}({\mathit{rt}}_2) \\
   & s ; f ; {\reff}~(\BRONCASTFAIL~l~{\mathit{rt}}_1~{\mathit{rt}}_2) & \stepto & {\reff}~(\BR~l) & \quad \mbox{otherwise} \\
   \end{array}


.. _exec-return:


:math:`\RETURN`
...............


1. If the first non-value entry of the stack is a :math:`\FRAME`, then:

   a. Let :math:`f` be the topmost :math:`\FRAME`.

   #. Let :math:`n` be the arity of :math:`f`

   #. Assert: Due to :ref:`validation <valid-return>`, there are at least :math:`n` values on the top of the stack.

   #. Pop the values :math:`{{\val}^{n}}` from the stack.

   #. Pop all values :math:`{{\val'}^\ast}` from the top of the stack.

   #. Pop the :math:`\FRAME` from the stack.

   #. Push the values :math:`{{\val}^{n}}` to the stack.

#. Else if the first non-value entry of the stack is a :math:`\LABEL`, then:

   a. Pop all values :math:`{{\val}^\ast}` from the top of the stack.

   #. Pop the :math:`\LABEL` from the stack.

   #. Push the values :math:`{{\val}^\ast}` to the stack.

   #. Execute the instruction :math:`\RETURN`.

#. Else:

   a. Assert: Due to :ref:`validation <valid-return>`, the first non-value entry of the stack is a :math:`\HANDLER`.

   #. Pop all values :math:`{{\val}^\ast}` from the top of the stack.

   #. Pop the :math:`\HANDLER` from the stack.

   #. Push the values :math:`{{\val}^\ast}` to the stack.

   #. Execute the instruction :math:`\RETURN`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & ({{\FRAME}_{n}}{\{ f \}}~{{\val'}^\ast}~{{\val}^{n}}~\RETURN~{{\instr}^\ast}) & \stepto & {{\val}^{n}} \\
   & ({{\LABEL}_{n}}{\{ {{\instr'}^\ast} \}}~{{\val}^\ast}~\RETURN~{{\instr}^\ast}) & \stepto & {{\val}^\ast}~\RETURN \\
   & ({{\HANDLER}_{n}}{\{ {{\catch}^\ast} \}}~{{\val}^\ast}~\RETURN~{{\instr}^\ast}) & \stepto & {{\val}^\ast}~\RETURN \\
   \end{array}


.. _exec-call:


:math:`\CALL~x`
...............


1. Let :math:`z` be the current state.

#. Assert: Due to :ref:`validation <valid-call>`, :math:`x < {|z{.}\ZMODULE{.}\MIFUNCS|}`.

#. Let :math:`a` be the :ref:`address <syntax-addr>` :math:`z{.}\ZMODULE{.}\MIFUNCS{}[x]`.

#. Assert: Due to :ref:`validation <valid-call>`, :math:`a < {|z{.}\ZFUNCS|}`.

#. Push the value :math:`(\REFFUNCADDR~a)` to the stack.

#. Execute the instruction :math:`(\CALLREF~z{.}\ZFUNCS{}[a]{.}\FITYPE)`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; (\CALL~x) & \stepto & (\REFFUNCADDR~a)~(\CALLREF~z{.}\ZFUNCS{}[a]{.}\FITYPE) & \quad \mbox{if}~ z{.}\ZMODULE{.}\MIFUNCS{}[x] = a \\
   \end{array}


.. _exec-call_ref:

:math:`\CALLREF~x`
..................

.. todo:: (*) Prose not spliced, for the prose merges the two cases of null and non-null references.

1. Assert: due to :ref:`validation <valid-call_ref>`, a null or :ref:`function reference <syntax-ref>` is on the top of the stack.

2. Pop the reference value :math:`r` from the stack.

3. If :math:`r` is :math:`\REFNULL~\X{ht}`, then:

    a. Trap.

4. Assert: due to :ref:`validation <valid-call_ref>`, :math:`r` is a :ref:`function reference <syntax-ref>`.

5. Let :math:`\REFFUNCADDR~a` be the reference :math:`r`.

6. :ref:`Invoke <exec-invoke>` the function instance at address :math:`a`.

.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; (\REFNULLADDR)~(\CALLREF~y) & \stepto & z ; \TRAP \\
   \end{array}

.. note::
   The formal rule for calling a non-null function reference is described :ref:`below <exec-invoke>`.


.. _exec-call_indirect:


:math:`\CALLINDIRECT~x~y`
.........................


1. Execute the instruction :math:`(\TABLEGET~x)`.

#. Execute the instruction :math:`(\REFCAST~(\REF~\NULL~y))`.

#. Execute the instruction :math:`(\CALLREF~y)`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & (\CALLINDIRECT~x~y) & \stepto & (\TABLEGET~x)~(\REFCAST~(\REF~\NULL~y))~(\CALLREF~y) \\
   \end{array}


.. _exec-return_call:


:math:`\RETURNCALL~x`
.....................


1. Let :math:`z` be the current state.

#. Assert: Due to :ref:`validation <valid-return_call>`, :math:`x < {|z{.}\ZMODULE{.}\MIFUNCS|}`.

#. Let :math:`a` be the :ref:`address <syntax-addr>` :math:`z{.}\ZMODULE{.}\MIFUNCS{}[x]`.

#. Assert: Due to :ref:`validation <valid-return_call>`, :math:`a < {|z{.}\ZFUNCS|}`.

#. Push the value :math:`(\REFFUNCADDR~a)` to the stack.

#. Execute the instruction :math:`(\RETURNCALLREF~z{.}\ZFUNCS{}[a]{.}\FITYPE)`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; (\RETURNCALL~x) & \stepto & (\REFFUNCADDR~a)~(\RETURNCALLREF~z{.}\ZFUNCS{}[a]{.}\FITYPE) & \quad \mbox{if}~ z{.}\ZMODULE{.}\MIFUNCS{}[x] = a \\
   \end{array}


.. _exec-return_call_ref:


:math:`\RETURNCALLREF~y`
........................


1. Let :math:`z` be the current state.

#. If the first non-value entry of the stack is a :math:`\LABEL`, then:

   a. Pop all values :math:`{{\val}^\ast}` from the top of the stack.

   #. Pop the :math:`\LABEL` from the stack.

   #. Push the values :math:`{{\val}^\ast}` to the stack.

   #. Execute the instruction :math:`(\RETURNCALLREF~y)`.

#. Else if the first non-value entry of the stack is a :math:`\HANDLER`, then:

   a. Pop all values :math:`{{\val}^\ast}` from the top of the stack.

   #. Pop the :math:`\HANDLER` from the stack.

   #. Push the values :math:`{{\val}^\ast}` to the stack.

   #. Execute the instruction :math:`(\RETURNCALLREF~y)`.

#. Else:

   a. Assert: Due to :ref:`validation <valid-return_call_ref>`, the first non-value entry of the stack is a :math:`\FRAME`.

   #. Assert: Due to :ref:`validation <valid-return_call_ref>`, a value is on the top of the stack.

   #. Pop the value :math:`{\val''}` from the stack.

   #. If :math:`{\val''} = \REFNULLADDR`, then:

      1) Trap.

   #. Assert: Due to :ref:`validation <valid-return_call_ref>`, :math:`{\val''}` is some :math:`\REFFUNCADDR~{\funcaddr}`.

   #. Let :math:`(\REFFUNCADDR~a)` be the destructuring of :math:`{\val''}`.

   #. Assert: Due to :ref:`validation <valid-return_call_ref>`, :math:`a < {|z{.}\ZFUNCS|}`.

   #. Assert: Due to :ref:`validation <valid-return_call_ref>`, the :ref:`expansion <aux-expand-deftype>` of :math:`z{.}\ZFUNCS{}[a]{.}\FITYPE` is some :math:`\TFUNC~{\resulttype} \Tarrow {\resulttype}`.

   #. Let :math:`(\TFUNC~{t_1^{n}}~\Tarrow~{t_2^{m}})` be the destructuring of the :ref:`expansion <aux-expand-deftype>` of :math:`z{.}\ZFUNCS{}[a]{.}\FITYPE`.

   #. Assert: Due to :ref:`validation <valid-return_call_ref>`, there are at least :math:`n` values on the top of the stack.

   #. Pop the values :math:`{{\val}^{n}}` from the stack.

   #. Pop all values :math:`{{\val'}^\ast}` from the top of the stack.

   #. Pop the :math:`\FRAME` from the stack.

   #. Push the values :math:`{{\val}^{n}}` to the stack.

   #. Push the value :math:`(\REFFUNCADDR~a)` to the stack.

   #. Execute the instruction :math:`(\CALLREF~y)`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; ({{\LABEL}_{k}}{\{ {{\instr'}^\ast} \}}~{{\val}^\ast}~(\RETURNCALLREF~y)~{{\instr}^\ast}) & \stepto & {{\val}^\ast}~(\RETURNCALLREF~y) \\
   & z ; ({{\HANDLER}_{k}}{\{ {{\catch}^\ast} \}}~{{\val}^\ast}~(\RETURNCALLREF~y)~{{\instr}^\ast}) & \stepto & {{\val}^\ast}~(\RETURNCALLREF~y) \\
   & z ; ({{\FRAME}_{k}}{\{ f \}}~{{\val}^\ast}~(\REFNULLADDR)~(\RETURNCALLREF~y)~{{\instr}^\ast}) & \stepto & \TRAP \\
   & z ; ({{\FRAME}_{k}}{\{ f \}}~{{\val'}^\ast}~{{\val}^{n}}~(\REFFUNCADDR~a)~(\RETURNCALLREF~y)~{{\instr}^\ast}) & \stepto & {{\val}^{n}}~(\REFFUNCADDR~a)~(\CALLREF~y) &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad \mbox{if}~ z{.}\ZFUNCS{}[a]{.}\FITYPE \approxexpanddt \TFUNC~{t_1^{n}} \Tarrow {t_2^{m}}
   } \\
   \end{array}


.. _exec-return_call_indirect:


:math:`\RETURNCALLINDIRECT~x~y`
...............................


1. Execute the instruction :math:`(\TABLEGET~x)`.

#. Execute the instruction :math:`(\REFCAST~(\REF~\NULL~y))`.

#. Execute the instruction :math:`(\RETURNCALLREF~y)`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & (\RETURNCALLINDIRECT~x~y) & \stepto & (\TABLEGET~x)~(\REFCAST~(\REF~\NULL~y))~(\RETURNCALLREF~y) \\
   \end{array}


.. _exec-throw:


:math:`\THROW~x`
................


1. Let :math:`z` be the current state.

#. Assert: Due to :ref:`validation <valid-throw>`, :math:`x < {|z{.}\ZMODULE{.}\ZTAGS|}`.

#. Assert: Due to :ref:`validation <valid-throw>`, the :ref:`expansion <aux-expand-deftype>` of :math:`z{.}\ZTAGS{}[x]{.}\HITYPE` is some :math:`\TFUNC~{\resulttype} \Tarrow {\resulttype}`.

#. Let :math:`(\TFUNC~{t^{n}}~\Tarrow~{\resulttype}_0)` be the destructuring of the :ref:`expansion <aux-expand-deftype>` of :math:`z{.}\ZTAGS{}[x]{.}\HITYPE`.

#. Assert: Due to :ref:`validation <valid-throw>`, :math:`{\resulttype}_0 = \epsilon`.

#. Let :math:`a` be the length of :math:`z{.}\ZEXNS`.

#. Assert: Due to :ref:`validation <valid-throw>`, there are at least :math:`n` values on the top of the stack.

#. Pop the values :math:`{{\val}^{n}}` from the stack.

#. Let :math:`{\mathit{exn}}` be the :ref:`exception instance <syntax-exninst>` :math:`\{ \EITAG~z{.}\ZMODULE{.}\ZTAGS{}[x],\;\allowbreak \EIFIELDS~{{\val}^{n}} \}`.

#. Append :math:`{\mathit{exn}}` to :math:`z{.}\ZEXNS`.

#. Push the value :math:`(\REFEXNADDR~a)` to the stack.

#. Execute the instruction :math:`\THROWREF`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; {{\val}^{n}}~(\THROW~x) & \stepto & z{}[{.}\ZEXNS \mathrel{{=}{\oplus}} {\mathit{exn}}] ; (\REFEXNADDR~a)~\THROWREF & \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ z{.}\ZTAGS{}[x]{.}\HITYPE \approxexpanddt \TFUNC~{t^{n}} \Tarrow \epsilon \\
   {\land}~ a = {|z{.}\ZEXNS|} \\
   {\land}~ {\mathit{exn}} = \{ \EITAG~z{.}\ZMODULE{.}\ZTAGS{}[x],\;\allowbreak \EIFIELDS~{{\val}^{n}} \} \\
   \end{array} \\
   \end{array}


.. _exec-throw_ref:


:math:`\THROWREF`
.................


1. Let :math:`z` be the current state.

#. Assert: Due to :ref:`validation <valid-throw_ref>`, a value is on the top of the stack.

#. Pop the value :math:`{\val'}` from the stack.

#. If :math:`{\val'} = \REFNULLADDR`, then:

   a. Trap.

#. If :math:`{\val'}` is some :math:`\REFEXNADDR~{\exnaddr}`, then:

   a. Let :math:`(\REFEXNADDR~a)` be the destructuring of :math:`{\val'}`.

   #. Pop all values :math:`{{\val}^\ast}` from the top of the stack.

   #. If :math:`{{\val}^\ast} \neq \epsilon`, then:

      1) Push the value :math:`(\REFEXNADDR~a)` to the stack.

      #) Execute the instruction :math:`\THROWREF`.

   #. Else if the first non-value entry of the stack is a :math:`\LABEL`, then:

      1) Pop the :math:`\LABEL` from the stack.

      #) Push the value :math:`(\REFEXNADDR~a)` to the stack.

      #) Execute the instruction :math:`\THROWREF`.

   #. Else:

      1) If the first non-value entry of the stack is a :math:`\FRAME`, then:

         a) Pop the :math:`\FRAME` from the stack.

         #) Push the value :math:`(\REFEXNADDR~a)` to the stack.

         #) Execute the instruction :math:`\THROWREF`.

      #) Else if the first non-value entry of the stack is not a :math:`\HANDLER`, then:

         a) Throw the exception :math:`{\val'}` as a result.

      #) Else:

         a) Let :math:`H` be the topmost :math:`\HANDLER`.

         #) Let :math:`n` be the arity of :math:`H`

         #) Let :math:`{{\catch''}^\ast}` be the catch handler of :math:`H`

         #) If :math:`{{\catch''}^\ast} = \epsilon`, then:

            1. Pop the :math:`\HANDLER` from the stack.

            #. Push the value :math:`(\REFEXNADDR~a)` to the stack.

            #. Execute the instruction :math:`\THROWREF`.

         #) Else if :math:`a \geq {|z{.}\ZEXNS|}`, then:

            1. Let :math:`{\catch}_0~{{\catch'}^\ast}` be :math:`{{\catch''}^\ast}`.

            #. If :math:`{\catch}_0` is some :math:`\CATCHALL~{\labelidx}`, then:

               a. Let :math:`(\CATCHALL~l)` be the destructuring of :math:`{\catch}_0`.

               #. Pop the :math:`\HANDLER` from the stack.

               #. Execute the instruction :math:`(\BR~l)`.

            #. Else if :math:`{\catch}_0` is not some :math:`\CATCHALLREF~{\labelidx}`, then:

               a. Let :math:`{\catch}~{{\catch'}^\ast}` be :math:`{{\catch''}^\ast}`.

               #. Pop the :math:`\HANDLER` from the stack.

               #. Let :math:`{H'}` be the :math:`\HANDLER` whose arity is :math:`n` and whose catch handler is :math:`{{\catch'}^\ast}`.

               #. Push the :math:`\HANDLER` :math:`{H'}`.

               #. Push the value :math:`(\REFEXNADDR~a)` to the stack.

               #. Execute the instruction :math:`\THROWREF`.

            #. Else:

               a. Let :math:`(\CATCHALLREF~l)` be the destructuring of :math:`{\catch}_0`.

               #. Pop the :math:`\HANDLER` from the stack.

               #. Push the value :math:`(\REFEXNADDR~a)` to the stack.

               #. Execute the instruction :math:`(\BR~l)`.

         #) Else:

            1. Let :math:`{{\val}^\ast}` be :math:`z{.}\ZEXNS{}[a]{.}\EIFIELDS`.

            #. Let :math:`{\catch}_0~{{\catch'}^\ast}` be :math:`{{\catch''}^\ast}`.

            #. If :math:`{\catch}_0` is some :math:`\CATCH~{\tagidx}~{\labelidx}`, then:

               a. Let :math:`(\CATCH~x~l)` be the destructuring of :math:`{\catch}_0`.

               #. If :math:`x < {|z{.}\ZMODULE{.}\ZTAGS|}` and :math:`z{.}\ZEXNS{}[a]{.}\EITAG = z{.}\ZMODULE{.}\ZTAGS{}[x]`, then:

                  1) Pop the :math:`\HANDLER` from the stack.

                  #) Push the values :math:`{{\val}^\ast}` to the stack.

                  #) Execute the instruction :math:`(\BR~l)`.

               #. Else:

                  1) Let :math:`{\catch}~{{\catch'}^\ast}` be :math:`{{\catch''}^\ast}`.

                  #) Pop the :math:`\HANDLER` from the stack.

                  #) Let :math:`{H'}` be the :math:`\HANDLER` whose arity is :math:`n` and whose catch handler is :math:`{{\catch'}^\ast}`.

                  #) Push the :math:`\HANDLER` :math:`{H'}`.

                  #) Push the value :math:`(\REFEXNADDR~a)` to the stack.

                  #) Execute the instruction :math:`\THROWREF`.

            #. Else if :math:`{\catch}_0` is some :math:`\CATCHREF~{\tagidx}~{\labelidx}`, then:

               a. Let :math:`(\CATCHREF~x~l)` be the destructuring of :math:`{\catch}_0`.

               #. If :math:`x \geq {|z{.}\ZMODULE{.}\ZTAGS|}` or :math:`z{.}\ZEXNS{}[a]{.}\EITAG \neq z{.}\ZMODULE{.}\ZTAGS{}[x]`, then:

                  1) Let :math:`{\catch}~{{\catch'}^\ast}` be :math:`{{\catch''}^\ast}`.

                  #) Pop the :math:`\HANDLER` from the stack.

                  #) Let :math:`{H'}` be the :math:`\HANDLER` whose arity is :math:`n` and whose catch handler is :math:`{{\catch'}^\ast}`.

                  #) Push the :math:`\HANDLER` :math:`{H'}`.

                  #) Push the value :math:`(\REFEXNADDR~a)` to the stack.

                  #) Execute the instruction :math:`\THROWREF`.

               #. Else:

                  1) Pop the :math:`\HANDLER` from the stack.

                  #) Push the values :math:`{{\val}^\ast}` to the stack.

                  #) Push the value :math:`(\REFEXNADDR~a)` to the stack.

                  #) Execute the instruction :math:`(\BR~l)`.

            #. Else:

               a. If :math:`{\catch}_0` is some :math:`\CATCHALL~{\labelidx}`, then:

                  1) Let :math:`(\CATCHALL~l)` be the destructuring of :math:`{\catch}_0`.

                  #) Pop the :math:`\HANDLER` from the stack.

                  #) Execute the instruction :math:`(\BR~l)`.

               #. Else if :math:`{\catch}_0` is not some :math:`\CATCHALLREF~{\labelidx}`, then:

                  1) Let :math:`{\catch}~{{\catch'}^\ast}` be :math:`{{\catch''}^\ast}`.

                  #) Pop the :math:`\HANDLER` from the stack.

                  #) Let :math:`H` be the :math:`\HANDLER` whose arity is :math:`n` and whose catch handler is :math:`{{\catch'}^\ast}`.

                  #) Push the :math:`\HANDLER` :math:`H`.

                  #) Push the value :math:`(\REFEXNADDR~a)` to the stack.

                  #) Execute the instruction :math:`\THROWREF`.

               #. Else:

                  1) Let :math:`(\CATCHALLREF~l)` be the destructuring of :math:`{\catch}_0`.

                  #) Pop the :math:`\HANDLER` from the stack.

                  #) Push the value :math:`(\REFEXNADDR~a)` to the stack.

                  #) Execute the instruction :math:`(\BR~l)`.

#. Else:

   a. Assert: Due to :ref:`validation <valid-throw_ref>`, the first non-value entry of the stack is not a :math:`\LABEL`.

   #. Assert: Due to :ref:`validation <valid-throw_ref>`, the first non-value entry of the stack is not a :math:`\FRAME`.

   #. Assert: Due to :ref:`validation <valid-throw_ref>`, the first non-value entry of the stack is not a :math:`\HANDLER`.

   #. Throw the exception :math:`{\val'}` as a result.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; (\REFNULLADDR)~\THROWREF & \stepto & \TRAP \\[0.8ex]
   & z ; {{\val}^\ast}~(\REFEXNADDR~a)~\THROWREF~{{\instr}^\ast} & \stepto & (\REFEXNADDR~a)~\THROWREF &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad \mbox{if}~ {{\val}^\ast} \neq \epsilon \lor {{\instr}^\ast} \neq \epsilon
   } \\[0.8ex]
   & z ; ({{\LABEL}_{n}}{\{ {{\instr'}^\ast} \}}~(\REFEXNADDR~a)~\THROWREF) & \stepto & (\REFEXNADDR~a)~\THROWREF \\[0.8ex]
   & z ; ({{\FRAME}_{n}}{\{ f \}}~(\REFEXNADDR~a)~\THROWREF) & \stepto & (\REFEXNADDR~a)~\THROWREF \\[0.8ex]
   & z ; ({{\HANDLER}_{n}}{\{ \epsilon \}}~(\REFEXNADDR~a)~\THROWREF) & \stepto & (\REFEXNADDR~a)~\THROWREF \\[0.8ex]
   & z ; ({{\HANDLER}_{n}}{\{ (\CATCH~x~l)~{{\catch'}^\ast} \}}~(\REFEXNADDR~a)~\THROWREF) & \stepto & {{\val}^\ast}~(\BR~l) &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ z{.}\ZEXNS{}[a]{.}\EITAG = z{.}\ZMODULE{.}\ZTAGS{}[x] \\
   {\land}~ {{\val}^\ast} = z{.}\ZEXNS{}[a]{.}\EIFIELDS \\
   \end{array}
   } \\[0.8ex]
   & z ; ({{\HANDLER}_{n}}{\{ (\CATCHREF~x~l)~{{\catch'}^\ast} \}}~(\REFEXNADDR~a)~\THROWREF) & \stepto & {{\val}^\ast}~(\REFEXNADDR~a)~(\BR~l) &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ z{.}\ZEXNS{}[a]{.}\EITAG = z{.}\ZMODULE{.}\ZTAGS{}[x] \\
   {\land}~ {{\val}^\ast} = z{.}\ZEXNS{}[a]{.}\EIFIELDS \\
   \end{array}
   } \\[0.8ex]
   & z ; ({{\HANDLER}_{n}}{\{ (\CATCHALL~l)~{{\catch'}^\ast} \}}~(\REFEXNADDR~a)~\THROWREF) & \stepto & (\BR~l) \\[0.8ex]
   & z ; ({{\HANDLER}_{n}}{\{ (\CATCHALLREF~l)~{{\catch'}^\ast} \}}~(\REFEXNADDR~a)~\THROWREF) & \stepto & (\REFEXNADDR~a)~(\BR~l) \\[0.8ex]
   & z ; ({{\HANDLER}_{n}}{\{ {\catch}~{{\catch'}^\ast} \}}~(\REFEXNADDR~a)~\THROWREF) & \stepto & ({{\HANDLER}_{n}}{\{ {{\catch'}^\ast} \}}~(\REFEXNADDR~a)~\THROWREF) &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad \mbox{otherwise}
   } \\
   \end{array}


.. _exec-try_table:


:math:`\TRYTABLE~{\mathit{bt}}~{{\catch}^\ast}~{{\instr}^\ast}`
...............................................................


1. Let :math:`z` be the current state.

#. Let :math:`{t_1^{m}}~{\to}_{{{\localidx}_0^\ast}}\,{t_2^{n}}` be the destructuring of :math:`{{\fblocktype}}_{z}({\mathit{bt}})`.

#. Assert: Due to :ref:`validation <valid-try_table>`, :math:`{{\localidx}_0^\ast} = \epsilon`.

#. Assert: Due to :ref:`validation <valid-try_table>`, there are at least :math:`m` values on the top of the stack.

#. Pop the values :math:`{{\val}^{m}}` from the stack.

#. Let :math:`H` be the :math:`\HANDLER` whose arity is :math:`n` and whose catch handler is :math:`{{\catch}^\ast}`.

#. Push the :math:`\HANDLER` :math:`H`.

#. Let :math:`L` be the :math:`\LABEL` whose arity is :math:`n` and whose continuation is the end of the block.

#. Enter the block :math:`{{\val}^{m}}~{{\instr}^\ast}` with the :math:`\LABEL` :math:`L`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; {{\val}^{m}}~(\TRYTABLE~{\mathit{bt}}~{{\catch}^\ast}~{{\instr}^\ast}) & \stepto & ({{\HANDLER}_{n}}{\{ {{\catch}^\ast} \}}~({{\LABEL}_{n}}{\{ \epsilon \}}~{{\val}^{m}}~{{\instr}^\ast})) &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad \mbox{if}~ {{\fblocktype}}_{z}({\mathit{bt}}) = {t_1^{m}} \rightarrow {t_2^{n}}
   } \\
   \end{array}


.. index:: instruction, instruction sequence, block, exception, trap
.. _exec-instrs:

Blocks
~~~~~~

The following auxiliary rules define the semantics of executing an :ref:`instruction sequence <syntax-instrs>`
that forms a :ref:`block <exec-instr-control>`.


.. _exec-instrs-enter:

Entering :math:`\instr^\ast` with label :math:`L` and values :math:`\val^\ast`
..............................................................................

1. Push :math:`L` to the stack.

2. Push the values :math:`\val^\ast` to the stack.

3. Jump to the start of the instruction sequence :math:`\instr^\ast`.

.. note::
   No formal reduction rule is needed for entering an instruction sequence,
   because the label :math:`L` is embedded in the :ref:`administrative instruction <syntax-instr-admin>` that structured control instructions reduce to directly.


.. _exec-instrs-exit:

Exiting :math:`\instr^\ast` with label :math:`L`
................................................

When the end of a block is reached without a jump, :ref:`exception <exception>`, or :ref:`trap <trap>` aborting it, then the following steps are performed.

1. Pop all values :math:`\val^\ast` from the top of the stack.

2. Assert: due to :ref:`validation <valid-instrs>`, the label :math:`L` is now on the top of the stack.

3. Pop the label from the stack.

4. Push :math:`\val^\ast` back to the stack.

5. Jump to the position after the end of the :ref:`structured control instruction <syntax-instr-control>` associated with the label :math:`L`.

.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & ({{\LABEL}_{n}}{\{ {{\instr}^\ast} \}}~{{\val}^\ast}) & \stepto & {{\val}^\ast} \\
   \end{array}

.. note::
   This semantics also applies to the instruction sequence contained in a :math:`\mathsf{loop}` instruction.
   Therefore, execution of a loop falls off the end, unless a backwards branch is performed explicitly.


.. index:: exception, handler, throw context, tag, exception tag

.. _exec-handler:

Exception Handling
~~~~~~~~~~~~~~~~~~

The following auxiliary rules define the semantics of entering and exiting :math:`\mathsf{try\_table}` blocks.

.. _exec-handler-enter:

Entering :math:`\instr^\ast` with label :math:`L` and exception handler :math:`H`
.................................................................................

1. Push :math:`H` to the stack.

2. Push :math:`L` onto the stack.

3. Jump to the start of the instruction sequence :math:`\instr^\ast`.

.. note::
   No formal reduction rule is needed for entering an exception :ref:`handler <syntax-handler>`
   because it is an :ref:`administrative instruction <syntax-instr-admin>`
   that the :math:`\mathsf{try\_table}` instruction reduces to directly.


.. _exec-handler-exit:

Exiting an exception handler
............................

When the end of a :math:`\mathsf{try\_table}` block is reached without a jump, :ref:`exception <exception>`, or :ref:`trap <trap>`, then the following steps are performed.

1. Let :math:`m` be the number of values on the top of the stack.

2. Pop the values :math:`\val^m` from the stack.

3. Assert: due to :ref:`validation <valid-instrs>`, a handler and a label are now on the top of the stack.

4. Pop the label from the stack.

5. Pop the handler :math:`H` from the stack.

6. Push :math:`\val^m` back to the stack.

7. Jump to the position after the end of the administrative instruction associated with the handler :math:`H`.

.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & ({{\HANDLER}_{n}}{\{ {{\catch}^\ast} \}}~{{\val}^\ast}) & \stepto & {{\val}^\ast} \\
   \end{array}


.. index:: ! call, function, function instance, label, frame

Function Calls
~~~~~~~~~~~~~~

The following auxiliary rules define the semantics of invoking a :ref:`function instance <syntax-funcinst>`
through one of the :ref:`call instructions <exec-instr-control>`
and returning from it.


.. _exec-invoke:

Invocation of :ref:`function reference <syntax-ref.func>` :math:`(\REFFUNCADDR~a)`
..................................................................................

1. Let :math:`z` be the current state.

2. Assert: due to :ref:`validation <valid-call>`, :math:`z.\SFUNCS[a]` exists.

3. Let :math:`\X{fi}` be the :ref:`function instance <syntax-funcinst>`, :math:`z.\SFUNCS[a]`.

4. Let :math:`\TFUNC~[t_1^n] \Tarrow [t_2^m]` be the :ref:`composite type <syntax-comptype>` :math:`\expanddt(\X{fi}.\FITYPE)`.

5. Let :math:`\FUNC~x~(\local~t)^\ast~\instr^\ast` be the :ref:`function <syntax-func>` :math:`\X{fi}.\FICODE`.

6. Assert: due to :ref:`validation <valid-call>`, :math:`n` values are on the top of the stack.

7. Pop the values :math:`\val^n` from the stack.

8. Let :math:`f` be the :ref:`frame <syntax-frame>` :math:`\{ \ALOCALS~\val^n~(\default_t)^\ast, \AMODULE~\X{fi}.\FIMODULE \}`.

9. Push the activation of :math:`f` with arity :math:`m` to the stack.

10. Let :math:`L` be the :ref:`label <syntax-label>` whose arity is :math:`m` and whose continuation is the end of the function.

11. :ref:`Enter <exec-instrs-enter>` the instruction sequence :math:`\instr^\ast` with label :math:`L` and no values.

.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; {{\val}^{n}}~(\REFFUNCADDR~a)~(\CALLREF~y) & \stepto & z ; ({{\FRAME}_{m}}{\{ f \}}~({{\LABEL}_{m}}{\{ \epsilon \}}~{{\instr}^\ast})) &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ z{.}\ZFUNCS{}[a] = {\mathit{fi}} \\
   {\land}~ {\mathit{fi}}{.}\FITYPE \approxexpanddt \TFUNC~{t_1^{n}} \Tarrow {t_2^{m}} \\
   {\land}~ {\mathit{fi}}{.}\FICODE = \FUNC~x~{(\LOCAL~t)^\ast}~({{\instr}^\ast}) \\
   {\land}~ f = \{ \ALOCALS~{{\val}^{n}}~{({{\default}}_{t})^\ast},\;\allowbreak \AMODULE~{\mathit{fi}}{.}\FIMODULE \} \\
   \end{array}
   } \\
   \end{array}

.. note::
   For non-defaultable types, the respective local is left uninitialized by these rules.


.. _exec-invoke-exit:

Returning from a function
.........................

When the end of a function is reached without a jump (including through |RETURN|), or an :ref:`exception <exception>` or :ref:`trap <trap>` aborting it, then the following steps are performed.

1. Let :math:`F` be the :ref:`current <exec-notation-textual>` :ref:`frame <syntax-frame>`.

2. Let :math:`n` be the arity of the activation of :math:`F`.

3. Assert: due to :ref:`validation <valid-instrs>`, there are :math:`n` values on the top of the stack.

4. Pop the results :math:`\val^n` from the stack.

5. Assert: due to :ref:`validation <valid-func>`, the frame :math:`F` is now on the top of the stack.

6. Pop the frame from the stack.

7. Push :math:`\val^n` back to the stack.

8. Jump to the instruction after the original call.

.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & ({{\FRAME}_{n}}{\{ f \}}~{{\val}^{n}}) & \stepto & {{\val}^{n}} \\
   \end{array}


.. index:: host function, store
.. _exec-invoke-host:

Host Functions
..............

Invoking a :ref:`host function <syntax-hostfunc>` has non-deterministic behavior.
It may either terminate with a :ref:`trap <trap>`, an :ref:`exception <exception>`, or return regularly.
However, in the latter case, it must consume and produce the right number and types of WebAssembly :ref:`values <syntax-val>` on the stack,
according to its :ref:`function type <syntax-functype>`.

A host function may also modify the :ref:`store <syntax-store>`.
However, all store modifications must result in an :ref:`extension <extend-store>` of the original store, i.e., they must only modify mutable contents and must not have instances removed.
Furthermore, the resulting store must be :ref:`valid <valid-store>`, i.e., all data and code in it is well-typed.

.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & s ; f ; {{\val}^{n}}~(\REFFUNCADDR~a)~(\CALLREF~y) & \stepto & {s'} ; f ; {\result} &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ (s ; f){.}\ZFUNCS{}[a] = {\mathit{fi}} \\
   {\land}~ {\mathit{fi}}{.}\FITYPE \approxexpanddt \TFUNC~{t_1^{n}} \Tarrow {t_2^{m}} \\
   {\land}~ {\mathit{fi}}{.}\FICODE = {\mathit{hf}} \\
   {\land}~ ({s'}, {\result}) \in {{\mathit{hf}}}{(s, {{\val}^{n}})} \\
   \end{array}
   } \\
   & s ; f ; {{\val}^{n}}~(\REFFUNCADDR~a)~(\CALLREF~y) & \stepto & s ; f ; (\REFFUNCADDR~a)~(\CALLREF~y) &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ (s ; f){.}\ZFUNCS{}[a] = {\mathit{fi}} \\
   {\land}~ {\mathit{fi}}{.}\FITYPE \approxexpanddt \TFUNC~{t_1^{n}} \Tarrow {t_2^{m}} \\
   {\land}~ {\mathit{fi}}{.}\FICODE = {\mathit{hf}} \\
   {\land}~ \bot \in {{\mathit{hf}}}{(s, {{\val}^{n}})} \\
   \end{array}
   } \\
   \end{array}

Here, :math:`{{\mathit{hf}}}{(s, {{\val}^{n}})}` denotes the implementation-defined execution of host function :math:`{\mathit{hf}}` in current store :math:`s` with arguments :math:`{{\val}^{n}}`.
It yields a set of possible outcomes, where each element is either a pair of a modified store :math:`{s'}` and a :ref:`result <syntax-result>`
or the special value :math:`\bot` indicating divergence.
A host function is non-deterministic if there is at least one argument for which the set of outcomes is not singular.

For a WebAssembly implementation to be :ref:`sound <soundness>` in the presence of host functions,
every :ref:`host function instance <syntax-funcinst>` must be :ref:`valid <valid-hostfuncinst>`,
which means that it adheres to suitable pre- and post-conditions:
under a :ref:`valid store <valid-store>` :math:`s`, and given arguments :math:`{{\val}^{n}}` matching the ascribed parameter types :math:`{t_1^{n}}`,
executing the host function must yield a non-empty set of possible outcomes each of which is either divergence or consists of a valid store :math:`{s'}` that is an :ref:`extension <extend-store>` of :math:`s` and a result matching the ascribed return types :math:`{t_2^{m}}`.
All these notions are made precise in the :ref:`Appendix <soundness>`.

.. note::
   A host function can call back into WebAssembly by :ref:`invoking <exec-invocation>` a function :ref:`exported <syntax-export>` from a :ref:`module <syntax-module>`.
   However, the effects of any such call are subsumed by the non-deterministic behavior allowed for the host function.


.. index:: variable instructions, local index, global index, address, global address, global instance, store, frame, value
   pair: execution; instruction
   single: abstract syntax; instruction
.. _exec-instr-variable:

Variable Instructions
~~~~~~~~~~~~~~~~~~~~~

.. _exec-local.get:


:math:`\LOCALGET~x`
...................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, :math:`z{.}\ZLOCALS{}[x]` is defined.

#. Let :math:`{\val}` be :math:`z{.}\ZLOCALS{}[x]`.

#. Push the value :math:`{\val}` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; (\LOCALGET~x) & \stepto & {\val} & \quad \mbox{if}~ z{.}\ZLOCALS{}[x] = {\val} \\
   \end{array}


.. _exec-local.set:


:math:`\LOCALSET~x`
...................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, a value is on the top of the stack.

#. Pop the value :math:`{\val}` from the stack.

#. Replace :math:`z{.}\ZLOCALS{}[x]` with :math:`{\val}`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; {\val}~(\LOCALSET~x) & \stepto & z{}[{.}\ZLOCALS{}[x] = {\val}] ; \epsilon \\
   \end{array}


.. _exec-local.tee:


:math:`\LOCALTEE~x`
...................


1. Assert: Due to validation, a value is on the top of the stack.

#. Pop the value :math:`{\val}` from the stack.

#. Push the value :math:`{\val}` to the stack.

#. Push the value :math:`{\val}` to the stack.

#. Execute the instruction :math:`(\LOCALSET~x)`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & {\val}~(\LOCALTEE~x) & \stepto & {\val}~{\val}~(\LOCALSET~x) \\
   \end{array}


.. _exec-global.get:


:math:`\GLOBALGET~x`
....................


1. Let :math:`z` be the current state.

#. Let :math:`{\val}` be the :ref:`value <syntax-val>` :math:`z{.}\ZGLOBALS{}[x]{.}\GIVALUE`.

#. Push the value :math:`{\val}` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; (\GLOBALGET~x) & \stepto & {\val} & \quad \mbox{if}~ z{.}\ZGLOBALS{}[x]{.}\GIVALUE = {\val} \\
   \end{array}


.. _exec-global.set:


:math:`\GLOBALSET~x`
....................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, a value is on the top of the stack.

#. Pop the value :math:`{\val}` from the stack.

#. Replace :math:`z{.}\ZGLOBALS{}[x]{.}\mathsf{value}` with :math:`{\val}`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; {\val}~(\GLOBALSET~x) & \stepto & z{}[{.}\ZGGLOBALS{}[x]{.}\ZGVALUE = {\val}] ; \epsilon \\
   \end{array}


.. index:: table instruction, table index, store, frame, address, table address, table instance, element address, element instance, value, integer, limits, reference, reference type
   pair: execution; instruction
   single: abstract syntax; instruction
.. _exec-instr-table:

Table Instructions
~~~~~~~~~~~~~~~~~~

.. _exec-table.get:


:math:`\TABLEGET~x`
...................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, a :ref:`number value <syntax-num>` is on the top of the stack.

#. Pop the value :math:`({\mathit{at}}{.}\CONST~i)` from the stack.

#. If :math:`i \geq {|z{.}\ZTABLES{}[x]{.}\TIREFS|}`, then:

   a. Trap.

#. Push the value :math:`z{.}\ZTABLES{}[x]{.}\TIREFS{}[i]` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; ({\mathit{at}}{.}\CONST~i)~(\TABLEGET~x) & \stepto & \TRAP & \quad \mbox{if}~ i \geq {|z{.}\ZTABLES{}[x]{.}\TIREFS|} \\
   & z ; ({\mathit{at}}{.}\CONST~i)~(\TABLEGET~x) & \stepto & z{.}\ZTABLES{}[x]{.}\TIREFS{}[i] & \quad \mbox{if}~ i < {|z{.}\ZTABLES{}[x]{.}\TIREFS|} \\
   \end{array}


.. _exec-table.set:


:math:`\TABLESET~x`
...................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, a :ref:`reference value <syntax-ref>` is on the top of the stack.

#. Pop the value :math:`{\reff}` from the stack.

#. Assert: Due to validation, a :ref:`number value <syntax-num>` is on the top of the stack.

#. Pop the value :math:`({\mathit{at}}{.}\CONST~i)` from the stack.

#. If :math:`i \geq {|z{.}\ZTABLES{}[x]{.}\TIREFS|}`, then:

   a. Trap.

#. Replace :math:`z{.}\ZTABLES{}[x]{.}\mathsf{refs}{}[i]` with :math:`{\reff}`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; ({\mathit{at}}{.}\CONST~i)~{\reff}~(\TABLESET~x) & \stepto & z ; \TRAP & \quad \mbox{if}~ i \geq {|z{.}\ZTABLES{}[x]{.}\TIREFS|} \\
   & z ; ({\mathit{at}}{.}\CONST~i)~{\reff}~(\TABLESET~x) & \stepto & z{}[{.}\ZTTABLES{}[x]{.}\ZTREFS{}[i] = {\reff}] ; \epsilon & \quad \mbox{if}~ i < {|z{.}\ZTABLES{}[x]{.}\TIREFS|} \\
   \end{array}


.. _exec-table.size:


:math:`\TABLESIZE~x`
....................


1. Let :math:`z` be the current state.

#. Let :math:`({\mathit{at}}~{\mathit{lim}}~{\mathit{rt}})` be the destructuring of :math:`z{.}\ZTABLES{}[x]{.}\TITYPE`.

#. Let :math:`n` be the length of :math:`z{.}\ZTABLES{}[x]{.}\TIREFS`.

#. Push the value :math:`({\mathit{at}}{.}\CONST~n)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; (\TABLESIZE~x) & \stepto & ({\mathit{at}}{.}\CONST~n) & \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ {|z{.}\ZTABLES{}[x]{.}\TIREFS|} = n \\
   {\land}~ z{.}\ZTABLES{}[x]{.}\TITYPE = {\mathit{at}}~{\mathit{lim}}~{\mathit{rt}} \\
   \end{array} \\
   \end{array}


.. index:: determinism, non-determinism
.. _exec-table.grow:


:math:`\TABLEGROW~x`
....................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, a :ref:`number value <syntax-num>` is on the top of the stack.

#. Pop the value :math:`({\mathit{at}}{.}\CONST~n)` from the stack.

#. Assert: Due to validation, a :ref:`reference value <syntax-ref>` is on the top of the stack.

#. Pop the value :math:`{\reff}` from the stack.

#. Either:

   a. Let :math:`{\mathit{ti}}` be the :ref:`table instance <syntax-tableinst>` :math:`{\growtable}(z{.}\ZTABLES{}[x], n, {\reff})`.

   #. Push the value :math:`({\mathit{at}}{.}\CONST~{|z{.}\ZTABLES{}[x]{.}\TIREFS|})` to the stack.

   #. Replace :math:`z{.}\ZTABLES{}[x]` with :math:`{\mathit{ti}}`.

#. Or:

   a. Push the value :math:`({\mathit{at}}{.}\CONST~{{{{\signed}}_{{|{\mathit{at}}|}}^{{-1}}}}{({-1})})` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; {\reff}~({\mathit{at}}{.}\CONST~n)~(\TABLEGROW~x) & \stepto & z{}[{.}\ZTABLES{}[x] = {\mathit{ti}}] ; ({\mathit{at}}{.}\CONST~{|z{.}\ZTABLES{}[x]{.}\TIREFS|}) &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad \mbox{if}~ {\mathit{ti}} = {\growtable}(z{.}\ZTABLES{}[x], n, {\reff})
   } \\
   & z ; {\reff}~({\mathit{at}}{.}\CONST~n)~(\TABLEGROW~x) & \stepto & z ; ({\mathit{at}}{.}\CONST~{{{{\signed}}_{{|{\mathit{at}}|}}^{{-1}}}}{({-1})}) \\
   \end{array}

.. note::
   The |TABLEGROW| instruction is non-deterministic.
   It may either succeed, returning the old table size :math:`\X{sz}`,
   or fail, returning :math:`{-1}`.
   Failure *must* occur if the referenced table instance has a maximum size defined that would be exceeded.
   However, failure *can* occur in other cases as well.
   In practice, the choice depends on the :ref:`resources <impl-exec>` available to the :ref:`embedder <embedder>`.


.. _exec-table.fill:


:math:`\TABLEFILL~x`
....................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, a :ref:`number value <syntax-num>` is on the top of the stack.

#. Pop the value :math:`({\mathit{at}}{.}\CONST~n)` from the stack.

#. Assert: Due to validation, a value is on the top of the stack.

#. Pop the value :math:`{\val}` from the stack.

#. Assert: Due to validation, a value of :ref:`number type <syntax-numtype>` :math:`{\mathit{at}}` is on the top of the stack.

#. Pop the value :math:`({\numtype}_0{.}\CONST~i)` from the stack.

#. If :math:`i + n > {|z{.}\ZTABLES{}[x]{.}\TIREFS|}`, then:

   a. Trap.

#. If :math:`n = 0`, then:

   a. Do nothing.

#. Else:

   a. Push the value :math:`({\mathit{at}}{.}\CONST~i)` to the stack.

   #. Push the value :math:`{\val}` to the stack.

   #. Execute the instruction :math:`(\TABLESET~x)`.

   #. Push the value :math:`({\mathit{at}}{.}\CONST~i + 1)` to the stack.

   #. Push the value :math:`{\val}` to the stack.

   #. Push the value :math:`({\mathit{at}}{.}\CONST~n - 1)` to the stack.

   #. Execute the instruction :math:`(\TABLEFILL~x)`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; ({\mathit{at}}{.}\CONST~i)~{\val}~({\mathit{at}}{.}\CONST~n)~(\TABLEFILL~x) & \stepto & \TRAP & \quad \mbox{if}~ i + n > {|z{.}\ZTABLES{}[x]{.}\TIREFS|} \\
   & z ; ({\mathit{at}}{.}\CONST~i)~{\val}~({\mathit{at}}{.}\CONST~n)~(\TABLEFILL~x) & \stepto & \epsilon & \quad \mbox{otherwise, if}~ n = 0 \\
   & z ; ({\mathit{at}}{.}\CONST~i)~{\val}~({\mathit{at}}{.}\CONST~n)~(\TABLEFILL~x) & \stepto & & \\
   & \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}l@{}}
   \begin{array}[t]{@{}l@{}} ({\mathit{at}}{.}\CONST~i)~{\val}~(\TABLESET~x) \\
     ({\mathit{at}}{.}\CONST~i + 1)~{\val}~({\mathit{at}}{.}\CONST~n - 1)~(\TABLEFILL~x) \end{array} & \quad \mbox{otherwise} \\
   \end{array}
   } \\
   \end{array}


.. _exec-table.copy:


:math:`\TABLECOPY~x_1~x_2`
..........................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, a :ref:`number value <syntax-num>` is on the top of the stack.

#. Pop the value :math:`({\mathit{at}}{.}\CONST~n)` from the stack.

#. Assert: Due to validation, a :ref:`number value <syntax-num>` is on the top of the stack.

#. Pop the value :math:`({\mathit{at}}_2{.}\CONST~i_2)` from the stack.

#. Assert: Due to validation, a :ref:`number value <syntax-num>` is on the top of the stack.

#. Pop the value :math:`({\mathit{at}}_1{.}\CONST~i_1)` from the stack.

#. If :math:`i_1 + n > {|z{.}\ZTABLES{}[x_1]{.}\TIREFS|}`, then:

   a. Trap.

#. If :math:`i_2 + n > {|z{.}\ZTABLES{}[x_2]{.}\TIREFS|}`, then:

   a. Trap.

#. If :math:`n = 0`, then:

   a. Do nothing.

#. Else:

   a. If :math:`i_1 \leq i_2`, then:

      1) Push the value :math:`({\mathit{at}}_1{.}\CONST~i_1)` to the stack.

      #) Push the value :math:`({\mathit{at}}_2{.}\CONST~i_2)` to the stack.

      #) Execute the instruction :math:`(\TABLEGET~x_2)`.

      #) Execute the instruction :math:`(\TABLESET~x_1)`.

      #) Push the value :math:`({\mathit{at}}_1{.}\CONST~i_1 + 1)` to the stack.

      #) Push the value :math:`({\mathit{at}}_2{.}\CONST~i_2 + 1)` to the stack.

   #. Else:

      1) Push the value :math:`({\mathit{at}}_1{.}\CONST~i_1 + n - 1)` to the stack.

      #) Push the value :math:`({\mathit{at}}_2{.}\CONST~i_2 + n - 1)` to the stack.

      #) Execute the instruction :math:`(\TABLEGET~x_2)`.

      #) Execute the instruction :math:`(\TABLESET~x_1)`.

      #) Push the value :math:`({\mathit{at}}_1{.}\CONST~i_1)` to the stack.

      #) Push the value :math:`({\mathit{at}}_2{.}\CONST~i_2)` to the stack.

   #. Push the value :math:`({\mathit{at}}{.}\CONST~n - 1)` to the stack.

   #. Execute the instruction :math:`(\TABLECOPY~x_1~x_2)`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; ({\mathit{at}}_1{.}\CONST~i_1)~({\mathit{at}}_2{.}\CONST~i_2)~({\mathit{at}'}{.}\CONST~n)~(\TABLECOPY~x_1~x_2) & \stepto & \TRAP &  \\
   & \multicolumn{4}{@{}l@{}}{\quad
   \quad \mbox{if}~ i_1 + n > {|z{.}\ZTABLES{}[x_1]{.}\TIREFS|} \lor i_2 + n > {|z{.}\ZTABLES{}[x_2]{.}\TIREFS|}
   } \\
   & z ; ({\mathit{at}}_1{.}\CONST~i_1)~({\mathit{at}}_2{.}\CONST~i_2)~({\mathit{at}'}{.}\CONST~n)~(\TABLECOPY~x~y) & \stepto & \epsilon & \quad \mbox{otherwise, if}~ n = 0 \\
   & z ; ({\mathit{at}}_1{.}\CONST~i_1)~({\mathit{at}}_2{.}\CONST~i_2)~({\mathit{at}'}{.}\CONST~n)~(\TABLECOPY~x~y) & \stepto & & \\
   & \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}l@{}}
   \begin{array}[t]{@{}l@{}} ({\mathit{at}}_1{.}\CONST~i_1)~({\mathit{at}}_2{.}\CONST~i_2)~(\TABLEGET~y)~(\TABLESET~x) \\
     ({\mathit{at}}_1{.}\CONST~i_1 + 1)~({\mathit{at}}_2{.}\CONST~i_2 + 1)~({\mathit{at}'}{.}\CONST~n - 1)~(\TABLECOPY~x~y) \end{array} & \quad \mbox{otherwise, if}~ i_1 \leq i_2 \\
   \end{array}
   } \\
   & z ; ({\mathit{at}}_1{.}\CONST~i_1)~({\mathit{at}}_2{.}\CONST~i_2)~({\mathit{at}'}{.}\CONST~n)~(\TABLECOPY~x~y) & \stepto & & \\
   & \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}l@{}}
   \begin{array}[t]{@{}l@{}} ({\mathit{at}}_1{.}\CONST~i_1 + n - 1)~({\mathit{at}}_2{.}\CONST~i_2 + n - 1)~(\TABLEGET~y)~(\TABLESET~x) \\
     ({\mathit{at}}_1{.}\CONST~i_1)~({\mathit{at}}_2{.}\CONST~i_2)~({\mathit{at}'}{.}\CONST~n - 1)~(\TABLECOPY~x~y) \end{array} & \quad \mbox{otherwise} \\
   \end{array}
   } \\
   \end{array}


.. _exec-table.init:


:math:`\TABLEINIT~x~y`
......................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, a value of :ref:`number type <syntax-numtype>` :math:`\I32` is on the top of the stack.

#. Pop the value :math:`(\I32{.}\CONST~n)` from the stack.

#. Assert: Due to validation, a value of :ref:`number type <syntax-numtype>` :math:`\I32` is on the top of the stack.

#. Pop the value :math:`(\I32{.}\CONST~j)` from the stack.

#. Assert: Due to validation, a :ref:`number value <syntax-num>` is on the top of the stack.

#. Pop the value :math:`({\mathit{at}}{.}\CONST~i)` from the stack.

#. If :math:`i + n > {|z{.}\ZTABLES{}[x]{.}\TIREFS|}`, then:

   a. Trap.

#. If :math:`j + n > {|z{.}\ZELEMS{}[y]{.}\EIREFS|}`, then:

   a. Trap.

#. If :math:`n = 0`, then:

   a. Do nothing.

#. Else:

   a. Assert: Due to validation, :math:`j < {|z{.}\ZELEMS{}[y]{.}\EIREFS|}`.

   #. Push the value :math:`({\mathit{at}}{.}\CONST~i)` to the stack.

   #. Push the value :math:`z{.}\ZELEMS{}[y]{.}\EIREFS{}[j]` to the stack.

   #. Execute the instruction :math:`(\TABLESET~x)`.

   #. Push the value :math:`({\mathit{at}}{.}\CONST~i + 1)` to the stack.

   #. Push the value :math:`(\I32{.}\CONST~j + 1)` to the stack.

   #. Push the value :math:`(\I32{.}\CONST~n - 1)` to the stack.

   #. Execute the instruction :math:`(\TABLEINIT~x~y)`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; ({\mathit{at}}{.}\CONST~i)~(\I32{.}\CONST~j)~(\I32{.}\CONST~n)~(\TABLEINIT~x~y) & \stepto & \TRAP &  \\
   & \multicolumn{4}{@{}l@{}}{\quad
   \quad \mbox{if}~ i + n > {|z{.}\ZTABLES{}[x]{.}\TIREFS|} \lor j + n > {|z{.}\ZELEMS{}[y]{.}\EIREFS|}
   } \\
   & z ; ({\mathit{at}}{.}\CONST~i)~(\I32{.}\CONST~j)~(\I32{.}\CONST~n)~(\TABLEINIT~x~y) & \stepto & \epsilon & \quad \mbox{otherwise, if}~ n = 0 \\
   & z ; ({\mathit{at}}{.}\CONST~i)~(\I32{.}\CONST~j)~(\I32{.}\CONST~n)~(\TABLEINIT~x~y) & \stepto & & \\
   & \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}l@{}}
   \begin{array}[t]{@{}l@{}} ({\mathit{at}}{.}\CONST~i)~z{.}\ZELEMS{}[y]{.}\EIREFS{}[j]~(\TABLESET~x) \\
     ({\mathit{at}}{.}\CONST~i + 1)~(\I32{.}\CONST~j + 1)~(\I32{.}\CONST~n - 1)~(\TABLEINIT~x~y) \end{array} & \quad \mbox{otherwise} \\
   \end{array}
   } \\
   \end{array}


.. _exec-elem.drop:


:math:`\ELEMDROP~x`
...................


1. Let :math:`z` be the current state.

#. Replace :math:`z{.}\ZELEMS{}[x]{.}\mathsf{refs}` with :math:`\epsilon`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; (\ELEMDROP~x) & \stepto & z{}[{.}\ZEELEMS{}[x]{.}\ZEREFS = \epsilon] ; \epsilon \\
   \end{array}


.. index:: memory instruction, memory index, store, frame, address, memory address, memory instance, value, integer, limits, value type, bit width
   pair: execution; instruction
   single: abstract syntax; instruction
.. _exec-memarg:
.. _exec-instr-memory:

Memory Instructions
~~~~~~~~~~~~~~~~~~~

.. note::
   The alignment :math:`\memarg.\ALIGN` in load and store instructions does not affect the semantics.
   It is a hint that the offset :math:`\X{ea}` at which the memory is accessed is intended to satisfy the property :math:`\X{ea} \mod 2^{\memarg.\ALIGN} = 0`.
   A WebAssembly implementation can use this hint to optimize for the intended use.
   Unaligned access violating that property is still allowed and must succeed regardless of the annotation.
   However, it may be substantially slower on some hardware.


.. _exec-load-val:
.. _exec-load-pack:
.. _exec-vload-val:


:math:`{{\mathit{nt}}{.}\LOAD}{{{\loadop}^?}}~x~{\mathit{ao}}`
..............................................................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, a :ref:`number value <syntax-num>` is on the top of the stack.

#. Pop the value :math:`({\mathit{at}}{.}\CONST~i)` from the stack.

#. If :math:`{{\loadop}^?}` is not defined, then:

   a. If :math:`i + {\mathit{ao}}{.}\OFFSET + {|{\mathit{nt}}|} / 8 > {|z{.}\ZMEMS{}[x]{.}\MIBYTES|}`, then:

      1) Trap.

   #. Let :math:`c` be the result for which :math:`{{\bytes}}_{{\mathit{nt}}}(c)` :math:`=` :math:`z{.}\ZMEMS{}[x]{.}\MIBYTES{}[i + {\mathit{ao}}{.}\OFFSET : {|{\mathit{nt}}|} / 8]`.

   #. Push the value :math:`({\mathit{nt}}{.}\CONST~c)` to the stack.

#. Else:

   a. Assert: Due to validation, :math:`{\mathit{nt}}` is :math:`{\ntI}{{\ntN}}`.

   #. Let :math:`{\loadop}_0` be :math:`{{\loadop}^?}`.

   #. Let :math:`{n}{\mathsf{\_}}{{\sx}}` be the destructuring of :math:`{\loadop}_0`.

   #. If :math:`i + {\mathit{ao}}{.}\OFFSET + n / 8 > {|z{.}\ZMEMS{}[x]{.}\MIBYTES|}`, then:

      1) Trap.

   #. Let :math:`c` be the result for which :math:`{{\bytes}}_{{\INX}{n}}(c)` :math:`=` :math:`z{.}\ZMEMS{}[x]{.}\MIBYTES{}[i + {\mathit{ao}}{.}\OFFSET : n / 8]`.

   #. Push the value :math:`({\mathit{nt}}{.}\CONST~{{{{\extend}}_{n, {|{\mathit{nt}}|}}^{{\sx}}}}{(c)})` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; ({\mathit{at}}{.}\CONST~i)~({\mathit{nt}}{.}\LOAD~x~{\mathit{ao}}) & \stepto & \TRAP &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad \mbox{if}~ i + {\mathit{ao}}{.}\OFFSET + {|{\mathit{nt}}|} / 8 > {|z{.}\ZMEMS{}[x]{.}\MIBYTES|}
   } \\
   & z ; ({\mathit{at}}{.}\CONST~i)~({\mathit{nt}}{.}\LOAD~x~{\mathit{ao}}) & \stepto & ({\mathit{nt}}{.}\CONST~c) &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad \mbox{if}~ {{\bytes}}_{{\mathit{nt}}}(c) = z{.}\ZMEMS{}[x]{.}\MIBYTES{}[i + {\mathit{ao}}{.}\OFFSET : {|{\mathit{nt}}|} / 8]
   } \\
   & z ; ({\mathit{at}}{.}\CONST~i)~({{\ntI}{{\ntN}}{.}\LOAD}{{n}{\mathsf{\_}}{{\sx}}}~x~{\mathit{ao}}) & \stepto & \TRAP &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad \mbox{if}~ i + {\mathit{ao}}{.}\OFFSET + n / 8 > {|z{.}\ZMEMS{}[x]{.}\MIBYTES|}
   } \\
   & z ; ({\mathit{at}}{.}\CONST~i)~({{\ntI}{{\ntN}}{.}\LOAD}{{n}{\mathsf{\_}}{{\sx}}}~x~{\mathit{ao}}) & \stepto & ({\ntI}{{\ntN}}{.}\CONST~{{{{\extend}}_{n, {|{\ntI}{{\ntN}}|}}^{{\sx}}}}{(c)}) &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad \mbox{if}~ {{\bytes}}_{{\INX}{n}}(c) = z{.}\ZMEMS{}[x]{.}\MIBYTES{}[i + {\mathit{ao}}{.}\OFFSET : n / 8]
   } \\
   \end{array}


.. _exec-vload-pack:


:math:`{\V128{.}\VLOAD}{{K}{\Xshape}{M}{\mathsf{\_}}{{\sx}}}~x~{\mathit{ao}}`
.............................................................................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, a :ref:`number value <syntax-num>` is on the top of the stack.

#. Pop the value :math:`({\mathit{at}}{.}\CONST~i)` from the stack.

#. If :math:`i + {\mathit{ao}}{.}\OFFSET + K \cdot M / 8 > {|z{.}\ZMEMS{}[x]{.}\MIBYTES|}`, then:

   a. Trap.

#. Let :math:`{j^{M}}` be the result for which :math:`{({{\bytes}}_{{\INX}{K}}({j^{M}}) = z{.}\ZMEMS{}[x]{.}\MIBYTES{}[i + {\mathit{ao}}{.}\OFFSET + k \cdot K / 8 : K / 8])^{k<M}}`.

#. Let :math:`{\ntI}{{\ntN}}` be the result for which :math:`N` :math:`=` :math:`K \cdot 2`.

#. Let :math:`c` be :math:`{{{{\lanes}}_{{{\ntI}{{\ntN}}}{\Xshape}{M}}^{{-1}}}}{({{{{{\extend}}_{K, N}^{{\sx}}}}{(j)}^{M}})}`.

#. Push the value :math:`(\V128{.}\CONST~c)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; ({\mathit{at}}{.}\CONST~i)~({\V128{.}\VLOAD}{{K}{\Xshape}{M}{\mathsf{\_}}{{\sx}}}~x~{\mathit{ao}}) & \stepto & \TRAP & \quad \mbox{if}~ i + {\mathit{ao}}{.}\OFFSET + K \cdot M / 8 > {|z{.}\ZMEMS{}[x]{.}\MIBYTES|} \\
   & z ; ({\mathit{at}}{.}\CONST~i)~({\V128{.}\VLOAD}{{K}{\Xshape}{M}{\mathsf{\_}}{{\sx}}}~x~{\mathit{ao}}) & \stepto & (\V128{.}\CONST~c) &  \\
   & \multicolumn{4}{@{}l@{}}{\quad
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ ({{\bytes}}_{{\INX}{K}}(j) = z{.}\ZMEMS{}[x]{.}\MIBYTES{}[i + {\mathit{ao}}{.}\OFFSET + k \cdot K / 8 : K / 8])^{k<M} \\
   {\land}~ c = {{{{\lanes}}_{{{\ntI}{{\ntN}}}{\Xshape}{M}}^{{-1}}}}{({{{{{\extend}}_{K, N}^{{\sx}}}}{(j)}^{M}})} \land N = K \cdot 2 \\
   \end{array}
   } \\
   \end{array}


.. _exec-vload-splat:


:math:`{\V128{.}\VLOAD}{{N}{\mathsf{\_}}{\LSPLAT}}~x~{\mathit{ao}}`
...................................................................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, a :ref:`number value <syntax-num>` is on the top of the stack.

#. Pop the value :math:`({\mathit{at}}{.}\CONST~i)` from the stack.

#. If :math:`i + {\mathit{ao}}{.}\OFFSET + N / 8 > {|z{.}\ZMEMS{}[x]{.}\MIBYTES|}`, then:

   a. Trap.

#. Let :math:`M` be :math:`128 / N`.

#. Let :math:`{\ntI}{{\ntN}}` be the result for which :math:`{|{\ntI}{{\ntN}}|}` :math:`=` :math:`N`.

#. Let :math:`j` be the result for which :math:`{{\bytes}}_{{\INX}{N}}(j)` :math:`=` :math:`z{.}\ZMEMS{}[x]{.}\MIBYTES{}[i + {\mathit{ao}}{.}\OFFSET : N / 8]`.

#. Let :math:`c` be :math:`{{{{\lanes}}_{{{\ntI}{{\ntN}}}{\Xshape}{M}}^{{-1}}}}{({j^{M}})}`.

#. Push the value :math:`(\V128{.}\CONST~c)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; ({\mathit{at}}{.}\CONST~i)~({\V128{.}\VLOAD}{{N}{\mathsf{\_}}{\LSPLAT}}~x~{\mathit{ao}}) & \stepto & \TRAP & \quad \mbox{if}~ i + {\mathit{ao}}{.}\OFFSET + N / 8 > {|z{.}\ZMEMS{}[x]{.}\MIBYTES|} \\
   & z ; ({\mathit{at}}{.}\CONST~i)~({\V128{.}\VLOAD}{{N}{\mathsf{\_}}{\LSPLAT}}~x~{\mathit{ao}}) & \stepto & (\V128{.}\CONST~c) &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ {{\bytes}}_{{\INX}{N}}(j) = z{.}\ZMEMS{}[x]{.}\MIBYTES{}[i + {\mathit{ao}}{.}\OFFSET : N / 8] \\
   {\land}~ N = {|{\ntI}{{\ntN}}|} \\
   {\land}~ M = 128 / N \\
   {\land}~ c = {{{{\lanes}}_{{{\ntI}{{\ntN}}}{\Xshape}{M}}^{{-1}}}}{({j^{M}})} \\
   \end{array}
   } \\
   \end{array}


.. _exec-vload-zero:


:math:`{\V128{.}\VLOAD}{{N}{\mathsf{\_}}{\LZERO}}~x~{\mathit{ao}}`
..................................................................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, a :ref:`number value <syntax-num>` is on the top of the stack.

#. Pop the value :math:`({\mathit{at}}{.}\CONST~i)` from the stack.

#. If :math:`i + {\mathit{ao}}{.}\OFFSET + N / 8 > {|z{.}\ZMEMS{}[x]{.}\MIBYTES|}`, then:

   a. Trap.

#. Let :math:`j` be the result for which :math:`{{\bytes}}_{{\INX}{N}}(j)` :math:`=` :math:`z{.}\ZMEMS{}[x]{.}\MIBYTES{}[i + {\mathit{ao}}{.}\OFFSET : N / 8]`.

#. Let :math:`c` be :math:`{{{{\extend}}_{N, 128}^{\U}}}{(j)}`.

#. Push the value :math:`(\V128{.}\CONST~c)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; ({\mathit{at}}{.}\CONST~i)~({\V128{.}\VLOAD}{{N}{\mathsf{\_}}{\LZERO}}~x~{\mathit{ao}}) & \stepto & \TRAP & \quad \mbox{if}~ i + {\mathit{ao}}{.}\OFFSET + N / 8 > {|z{.}\ZMEMS{}[x]{.}\MIBYTES|} \\
   & z ; ({\mathit{at}}{.}\CONST~i)~({\V128{.}\VLOAD}{{N}{\mathsf{\_}}{\LZERO}}~x~{\mathit{ao}}) & \stepto & (\V128{.}\CONST~c) &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ {{\bytes}}_{{\INX}{N}}(j) = z{.}\ZMEMS{}[x]{.}\MIBYTES{}[i + {\mathit{ao}}{.}\OFFSET : N / 8] \\
   {\land}~ c = {{{{\extend}}_{N, 128}^{\U}}}{(j)} \\
   \end{array}
   } \\
   \end{array}


.. _exec-vload_lane:


:math:`{\V128{.}\VLOAD}{N}{\mathsf{\_}}{\VLANE}~x~{\mathit{ao}}~j`
..................................................................


1. Let :math:`z` be the current state.

#. Assert: Due to :ref:`validation <valid-vload_lane>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_1)` from the stack.

#. Assert: Due to :ref:`validation <valid-vload_lane>`, a :ref:`number value <syntax-num>` is on the top of the stack.

#. Pop the value :math:`({\mathit{at}}{.}\CONST~i)` from the stack.

#. If :math:`i + {\mathit{ao}}{.}\OFFSET + N / 8 > {|z{.}\ZMEMS{}[x]{.}\MIBYTES|}`, then:

   a. Trap.

#. Let :math:`M` be :math:`{|\V128|} / N`.

#. Let :math:`{\ntI}{{\ntN}}` be the result for which :math:`{|{\ntI}{{\ntN}}|}` :math:`=` :math:`N`.

#. Let :math:`k` be the result for which :math:`{{\bytes}}_{{\INX}{N}}(k)` :math:`=` :math:`z{.}\ZMEMS{}[x]{.}\MIBYTES{}[i + {\mathit{ao}}{.}\OFFSET : N / 8]`.

#. Let :math:`c` be :math:`{{{{\lanes}}_{{{\ntI}{{\ntN}}}{\Xshape}{M}}^{{-1}}}}{({{\lanes}}_{{{\ntI}{{\ntN}}}{\Xshape}{M}}(c_1){}[{}[j] = k])}`.

#. Push the value :math:`(\V128{.}\CONST~c)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; ({\mathit{at}}{.}\CONST~i)~(\V128{.}\CONST~c_1)~({\V128{.}\VLOAD}{N}{\mathsf{\_}}{\VLANE}~x~{\mathit{ao}}~j) & \stepto & \TRAP & \quad \mbox{if}~ i + {\mathit{ao}}{.}\OFFSET + N / 8 > {|z{.}\ZMEMS{}[x]{.}\MIBYTES|} \\
   & z ; ({\mathit{at}}{.}\CONST~i)~(\V128{.}\CONST~c_1)~({\V128{.}\VLOAD}{N}{\mathsf{\_}}{\VLANE}~x~{\mathit{ao}}~j) & \stepto & (\V128{.}\CONST~c) &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ {{\bytes}}_{{\INX}{N}}(k) = z{.}\ZMEMS{}[x]{.}\MIBYTES{}[i + {\mathit{ao}}{.}\OFFSET : N / 8] \\
   {\land}~ N = {|{\ntI}{{\ntN}}|} \\
   {\land}~ M = {|\V128|} / N \\
   {\land}~ c = {{{{\lanes}}_{{{\ntI}{{\ntN}}}{\Xshape}{M}}^{{-1}}}}{({{\lanes}}_{{{\ntI}{{\ntN}}}{\Xshape}{M}}(c_1){}[{}[j] = k])} \\
   \end{array}
   } \\
   \end{array}


.. _exec-store-val:
.. _exec-store-pack:
.. _exec-vstore:


:math:`{{\mathit{nt}}{.}\STORE}{{{\storeop}^?}}~x~{\mathit{ao}}`
................................................................


1. Let :math:`z` be the current state.

#. Assert: Due to :ref:`validation <valid-store>`, a :ref:`number value <syntax-num>` is on the top of the stack.

#. Pop the value :math:`({\mathit{nt}'}{.}\CONST~c)` from the stack.

#. Assert: Due to :ref:`validation <valid-store>`, a value is on the top of the stack.

#. Pop the value :math:`({\mathit{at}}{.}\CONST~i)` from the stack.

#. Assert: Due to :ref:`validation <valid-store>`, :math:`{\mathit{nt}} = {\mathit{nt}'}`.

#. If :math:`{{\storeop}^?}` is not defined, then:

   a. If :math:`i + {\mathit{ao}}{.}\OFFSET + {|{\mathit{nt}'}|} / 8 > {|z{.}\ZMEMS{}[x]{.}\MIBYTES|}`, then:

      1) Trap.

   #. Let :math:`{b^\ast}` be :math:`{{\bytes}}_{{\mathit{nt}'}}(c)`.

   #. Replace :math:`z{.}\ZMEMS{}[x]{.}\mathsf{bytes}{}[i + {\mathit{ao}}{.}\OFFSET : {|{\mathit{nt}'}|} / 8]` with :math:`{b^\ast}`.

#. Else:

   a. Assert: Due to :ref:`validation <valid-store>`, :math:`{\mathit{nt}'}` is :math:`{\ntI}{{\ntN}}`.

   #. Let :math:`n` be :math:`{{\storeop}^?}`.

   #. If :math:`i + {\mathit{ao}}{.}\OFFSET + n / 8 > {|z{.}\ZMEMS{}[x]{.}\MIBYTES|}`, then:

      1) Trap.

   #. Let :math:`{b^\ast}` be :math:`{{\bytes}}_{{\INX}{n}}({{\wrap}}_{{|{\mathit{nt}'}|}, n}(c))`.

   #. Replace :math:`z{.}\ZMEMS{}[x]{.}\mathsf{bytes}{}[i + {\mathit{ao}}{.}\OFFSET : n / 8]` with :math:`{b^\ast}`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; ({\mathit{at}}{.}\CONST~i)~({\mathit{nt}}{.}\CONST~c)~({\mathit{nt}}{.}\STORE~x~{\mathit{ao}}) & \stepto & z ; \TRAP &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad \mbox{if}~ i + {\mathit{ao}}{.}\OFFSET + {|{\mathit{nt}}|} / 8 > {|z{.}\ZMEMS{}[x]{.}\MIBYTES|}
   } \\
   & z ; ({\mathit{at}}{.}\CONST~i)~({\mathit{nt}}{.}\CONST~c)~({\mathit{nt}}{.}\STORE~x~{\mathit{ao}}) & \stepto & z{}[{.}\ZMMEMS{}[x]{.}\ZMBYTES{}[i + {\mathit{ao}}{.}\OFFSET : {|{\mathit{nt}}|} / 8] = {b^\ast}] ; \epsilon &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad \mbox{if}~ {b^\ast} = {{\bytes}}_{{\mathit{nt}}}(c)
   } \\
   & z ; ({\mathit{at}}{.}\CONST~i)~({\ntI}{{\ntN}}{.}\CONST~c)~({{\ntI}{{\ntN}}{.}\STORE}{n}~x~{\mathit{ao}}) & \stepto & z ; \TRAP &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad \mbox{if}~ i + {\mathit{ao}}{.}\OFFSET + n / 8 > {|z{.}\ZMEMS{}[x]{.}\MIBYTES|}
   } \\
   & z ; ({\mathit{at}}{.}\CONST~i)~({\ntI}{{\ntN}}{.}\CONST~c)~({{\ntI}{{\ntN}}{.}\STORE}{n}~x~{\mathit{ao}}) & \stepto & z{}[{.}\ZMMEMS{}[x]{.}\ZMBYTES{}[i + {\mathit{ao}}{.}\OFFSET : n / 8] = {b^\ast}] ; \epsilon &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad \mbox{if}~ {b^\ast} = {{\bytes}}_{{\INX}{n}}({{\wrap}}_{{|{\ntI}{{\ntN}}|}, n}(c))
   } \\
   & z ; ({\mathit{at}}{.}\CONST~i)~(\V128{.}\CONST~c)~(\V128{.}\VSTORE~x~{\mathit{ao}}) & \stepto & z ; \TRAP &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad \mbox{if}~ i + {\mathit{ao}}{.}\OFFSET + {|\V128|} / 8 > {|z{.}\ZMEMS{}[x]{.}\MIBYTES|}
   } \\
   & z ; ({\mathit{at}}{.}\CONST~i)~(\V128{.}\CONST~c)~(\V128{.}\VSTORE~x~{\mathit{ao}}) & \stepto & z{}[{.}\ZMMEMS{}[x]{.}\ZMBYTES{}[i + {\mathit{ao}}{.}\OFFSET : {|\V128|} / 8] = {b^\ast}] ; \epsilon &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad \mbox{if}~ {b^\ast} = {{\bytes}}_{\V128}(c)
   } \\
   \end{array}


.. _exec-vstore_lane:


:math:`{\V128{.}\VSTORE}{N}{\mathsf{\_}}{\VLANE}~x~{\mathit{ao}}~j`
...................................................................


1. Let :math:`z` be the current state.

#. Assert: Due to :ref:`validation <valid-vstore_lane>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c)` from the stack.

#. Assert: Due to :ref:`validation <valid-vstore_lane>`, a :ref:`number value <syntax-num>` is on the top of the stack.

#. Pop the value :math:`({\mathit{at}}{.}\CONST~i)` from the stack.

#. If :math:`i + {\mathit{ao}}{.}\OFFSET + N / 8 > {|z{.}\ZMEMS{}[x]{.}\MIBYTES|}`, then:

   a. Trap.

#. Let :math:`M` be :math:`128 / N`.

#. Let :math:`{\ntI}{{\ntN}}` be the result for which :math:`{|{\ntI}{{\ntN}}|}` :math:`=` :math:`N`.

#. Assert: Due to :ref:`validation <valid-vstore_lane>`, :math:`j < {|{{\lanes}}_{{{\ntI}{{\ntN}}}{\Xshape}{M}}(c)|}`.

#. Let :math:`{b^\ast}` be :math:`{{\bytes}}_{{\INX}{N}}({{\lanes}}_{{{\ntI}{{\ntN}}}{\Xshape}{M}}(c){}[j])`.

#. Replace :math:`z{.}\ZMEMS{}[x]{.}\mathsf{bytes}{}[i + {\mathit{ao}}{.}\OFFSET : N / 8]` with :math:`{b^\ast}`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; ({\mathit{at}}{.}\CONST~i)~(\V128{.}\CONST~c)~({\V128{.}\VSTORE}{N}{\mathsf{\_}}{\VLANE}~x~{\mathit{ao}}~j) & \stepto & z ; \TRAP &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad \mbox{if}~ i + {\mathit{ao}}{.}\OFFSET + N / 8 > {|z{.}\ZMEMS{}[x]{.}\MIBYTES|}
   } \\
   & z ; ({\mathit{at}}{.}\CONST~i)~(\V128{.}\CONST~c)~({\V128{.}\VSTORE}{N}{\mathsf{\_}}{\VLANE}~x~{\mathit{ao}}~j) & \stepto & z{}[{.}\ZMMEMS{}[x]{.}\ZMBYTES{}[i + {\mathit{ao}}{.}\OFFSET : N / 8] = {b^\ast}] ; \epsilon &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ N = {|{\ntI}{{\ntN}}|} \\
   {\land}~ M = 128 / N \\
   {\land}~ {b^\ast} = {{\bytes}}_{{\INX}{N}}({{\lanes}}_{{{\ntI}{{\ntN}}}{\Xshape}{M}}(c){}[j]) \\
   \end{array}
   } \\
   \end{array}


.. _exec-memory.size:


:math:`\MEMORYSIZE~x`
.....................


1. Let :math:`z` be the current state.

#. Let :math:`({\mathit{at}}~{\mathit{lim}}~\PAGE)` be the destructuring of :math:`z{.}\ZMEMS{}[x]{.}\MITYPE`.

#. Let :math:`n \cdot 64 \, {\mathrm{Ki}}` be the length of :math:`z{.}\ZMEMS{}[x]{.}\MIBYTES`.

#. Push the value :math:`({\mathit{at}}{.}\CONST~n)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; (\MEMORYSIZE~x) & \stepto & ({\mathit{at}}{.}\CONST~n) & \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ n \cdot 64 \, {\mathrm{Ki}} = {|z{.}\ZMEMS{}[x]{.}\MIBYTES|} \\
   {\land}~ z{.}\ZMEMS{}[x]{.}\MITYPE = {\mathit{at}}~{\mathit{lim}}~\PAGE \\
   \end{array} \\
   \end{array}


.. index:: determinism, non-determinism
.. _exec-memory.grow:


:math:`\MEMORYGROW~x`
.....................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, a :ref:`number value <syntax-num>` is on the top of the stack.

#. Pop the value :math:`({\mathit{at}}{.}\CONST~n)` from the stack.

#. Either:

   a. Let :math:`{\mathit{mi}}` be the :ref:`memory instance <syntax-meminst>` :math:`{\growmem}(z{.}\ZMEMS{}[x], n)`.

   #. Push the value :math:`({\mathit{at}}{.}\CONST~{|z{.}\ZMEMS{}[x]{.}\MIBYTES|} / (64 \, {\mathrm{Ki}}))` to the stack.

   #. Replace :math:`z{.}\ZMEMS{}[x]` with :math:`{\mathit{mi}}`.

#. Or:

   a. Push the value :math:`({\mathit{at}}{.}\CONST~{{{{\signed}}_{{|{\mathit{at}}|}}^{{-1}}}}{({-1})})` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; ({\mathit{at}}{.}\CONST~n)~(\MEMORYGROW~x) & \stepto & z{}[{.}\ZMEMS{}[x] = {\mathit{mi}}] ; ({\mathit{at}}{.}\CONST~{|z{.}\ZMEMS{}[x]{.}\MIBYTES|} / 64 \, {\mathrm{Ki}}) &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad \mbox{if}~ {\mathit{mi}} = {\growmem}(z{.}\ZMEMS{}[x], n)
   } \\
   & z ; ({\mathit{at}}{.}\CONST~n)~(\MEMORYGROW~x) & \stepto & z ; ({\mathit{at}}{.}\CONST~{{{{\signed}}_{{|{\mathit{at}}|}}^{{-1}}}}{({-1})}) \\
   \end{array}

.. note::
   The |MEMORYGROW| instruction is non-deterministic.
   It may either succeed, returning the old memory size :math:`\X{sz}`,
   or fail, returning :math:`{-1}`.
   Failure *must* occur if the referenced memory instance has a maximum size defined that would be exceeded.
   However, failure *can* occur in other cases as well.
   In practice, the choice depends on the :ref:`resources <impl-exec>` available to the :ref:`embedder <embedder>`.


.. _exec-memory.fill:


:math:`\MEMORYFILL~x`
.....................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, a :ref:`number value <syntax-num>` is on the top of the stack.

#. Pop the value :math:`({\mathit{at}}{.}\CONST~n)` from the stack.

#. Assert: Due to validation, a value is on the top of the stack.

#. Pop the value :math:`{\val}` from the stack.

#. Assert: Due to validation, a value of :ref:`number type <syntax-numtype>` :math:`{\mathit{at}}` is on the top of the stack.

#. Pop the value :math:`({\numtype}_0{.}\CONST~i)` from the stack.

#. If :math:`i + n > {|z{.}\ZMEMS{}[x]{.}\MIBYTES|}`, then:

   a. Trap.

#. If :math:`n = 0`, then:

   a. Do nothing.

#. Else:

   a. Push the value :math:`({\mathit{at}}{.}\CONST~i)` to the stack.

   #. Push the value :math:`{\val}` to the stack.

   #. Execute the instruction :math:`({\I32{.}\STORE}{8}~x)`.

   #. Push the value :math:`({\mathit{at}}{.}\CONST~i + 1)` to the stack.

   #. Push the value :math:`{\val}` to the stack.

   #. Push the value :math:`({\mathit{at}}{.}\CONST~n - 1)` to the stack.

   #. Execute the instruction :math:`(\MEMORYFILL~x)`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; ({\mathit{at}}{.}\CONST~i)~{\val}~({\mathit{at}}{.}\CONST~n)~(\MEMORYFILL~x) & \stepto & \TRAP & \quad \mbox{if}~ i + n > {|z{.}\ZMEMS{}[x]{.}\MIBYTES|} \\
   & z ; ({\mathit{at}}{.}\CONST~i)~{\val}~({\mathit{at}}{.}\CONST~n)~(\MEMORYFILL~x) & \stepto & \epsilon & \quad \mbox{otherwise, if}~ n = 0 \\
   & z ; ({\mathit{at}}{.}\CONST~i)~{\val}~({\mathit{at}}{.}\CONST~n)~(\MEMORYFILL~x) & \stepto & & \\
   & \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}l@{}}
   \begin{array}[t]{@{}l@{}} ({\mathit{at}}{.}\CONST~i)~{\val}~({\I32{.}\STORE}{\mathsf{{\scriptstyle 8}}}~x) \\
     ({\mathit{at}}{.}\CONST~i + 1)~{\val}~({\mathit{at}}{.}\CONST~n - 1)~(\MEMORYFILL~x) \end{array} & \quad \mbox{otherwise} \\
   \end{array}
   } \\
   \end{array}


.. _exec-memory.copy:


:math:`\MEMORYCOPY~x_1~x_2`
...........................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, a :ref:`number value <syntax-num>` is on the top of the stack.

#. Pop the value :math:`({\mathit{at}}{.}\CONST~n)` from the stack.

#. Assert: Due to validation, a :ref:`number value <syntax-num>` is on the top of the stack.

#. Pop the value :math:`({\mathit{at}}_2{.}\CONST~i_2)` from the stack.

#. Assert: Due to validation, a :ref:`number value <syntax-num>` is on the top of the stack.

#. Pop the value :math:`({\mathit{at}}_1{.}\CONST~i_1)` from the stack.

#. If :math:`i_1 + n > {|z{.}\ZMEMS{}[x_1]{.}\MIBYTES|}`, then:

   a. Trap.

#. If :math:`i_2 + n > {|z{.}\ZMEMS{}[x_2]{.}\MIBYTES|}`, then:

   a. Trap.

#. If :math:`n = 0`, then:

   a. Do nothing.

#. Else:

   a. If :math:`i_1 \leq i_2`, then:

      1) Push the value :math:`({\mathit{at}}_1{.}\CONST~i_1)` to the stack.

      #) Push the value :math:`({\mathit{at}}_2{.}\CONST~i_2)` to the stack.

      #) Execute the instruction :math:`({\I32{.}\LOAD}{{8}{\mathsf{\_}}{\U}}~x_2)`.

      #) Execute the instruction :math:`({\I32{.}\STORE}{8}~x_1)`.

      #) Push the value :math:`({\mathit{at}}_1{.}\CONST~i_1 + 1)` to the stack.

      #) Push the value :math:`({\mathit{at}}_2{.}\CONST~i_2 + 1)` to the stack.

   #. Else:

      1) Push the value :math:`({\mathit{at}}_1{.}\CONST~i_1 + n - 1)` to the stack.

      #) Push the value :math:`({\mathit{at}}_2{.}\CONST~i_2 + n - 1)` to the stack.

      #) Execute the instruction :math:`({\I32{.}\LOAD}{{8}{\mathsf{\_}}{\U}}~x_2)`.

      #) Execute the instruction :math:`({\I32{.}\STORE}{8}~x_1)`.

      #) Push the value :math:`({\mathit{at}}_1{.}\CONST~i_1)` to the stack.

      #) Push the value :math:`({\mathit{at}}_2{.}\CONST~i_2)` to the stack.

   #. Push the value :math:`({\mathit{at}}{.}\CONST~n - 1)` to the stack.

   #. Execute the instruction :math:`(\MEMORYCOPY~x_1~x_2)`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; ({\mathit{at}}_1{.}\CONST~i_1)~({\mathit{at}}_2{.}\CONST~i_2)~({\mathit{at}'}{.}\CONST~n)~(\MEMORYCOPY~x_1~x_2) & \stepto & \TRAP &  \\
   & \multicolumn{4}{@{}l@{}}{\quad
   \quad \mbox{if}~ i_1 + n > {|z{.}\ZMEMS{}[x_1]{.}\MIBYTES|} \lor i_2 + n > {|z{.}\ZMEMS{}[x_2]{.}\MIBYTES|}
   } \\
   & z ; ({\mathit{at}}_1{.}\CONST~i_1)~({\mathit{at}}_2{.}\CONST~i_2)~({\mathit{at}'}{.}\CONST~n)~(\MEMORYCOPY~x_1~x_2) & \stepto & \epsilon & \quad \mbox{otherwise, if}~ n = 0 \\
   & z ; ({\mathit{at}}_1{.}\CONST~i_1)~({\mathit{at}}_2{.}\CONST~i_2)~({\mathit{at}'}{.}\CONST~n)~(\MEMORYCOPY~x_1~x_2) & \stepto & & \\
   & \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}l@{}}
   \begin{array}[t]{@{}l@{}} ({\mathit{at}}_1{.}\CONST~i_1)~({\mathit{at}}_2{.}\CONST~i_2)~({\I32{.}\LOAD}{{\mathsf{{\scriptstyle 8}}}{\mathsf{\_}}{\U}}~x_2)~({\I32{.}\STORE}{\mathsf{{\scriptstyle 8}}}~x_1) \\
     ({\mathit{at}}_1{.}\CONST~i_1 + 1)~({\mathit{at}}_2{.}\CONST~i_2 + 1)~({\mathit{at}'}{.}\CONST~n - 1)~(\MEMORYCOPY~x_1~x_2) \end{array} & \quad \mbox{otherwise, if}~ i_1 \leq i_2 \\
   \end{array}
   } \\
   & z ; ({\mathit{at}}_1{.}\CONST~i_1)~({\mathit{at}}_2{.}\CONST~i_2)~({\mathit{at}'}{.}\CONST~n)~(\MEMORYCOPY~x_1~x_2) & \stepto & & \\
   & \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}l@{}}
   \begin{array}[t]{@{}l@{}} ({\mathit{at}}_1{.}\CONST~i_1 + n - 1)~({\mathit{at}}_2{.}\CONST~i_2 + n - 1)~({\I32{.}\LOAD}{{\mathsf{{\scriptstyle 8}}}{\mathsf{\_}}{\U}}~x_2)~({\I32{.}\STORE}{\mathsf{{\scriptstyle 8}}}~x_1) \\
     ({\mathit{at}}_1{.}\CONST~i_1)~({\mathit{at}}_2{.}\CONST~i_2)~({\mathit{at}'}{.}\CONST~n - 1)~(\MEMORYCOPY~x_1~x_2) \end{array} & \quad \mbox{otherwise} \\
   \end{array}
   } \\
   \end{array}


.. _exec-memory.init:


:math:`\MEMORYINIT~x~y`
.......................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, a value of :ref:`number type <syntax-numtype>` :math:`\I32` is on the top of the stack.

#. Pop the value :math:`(\I32{.}\CONST~n)` from the stack.

#. Assert: Due to validation, a value of :ref:`number type <syntax-numtype>` :math:`\I32` is on the top of the stack.

#. Pop the value :math:`(\I32{.}\CONST~j)` from the stack.

#. Assert: Due to validation, a :ref:`number value <syntax-num>` is on the top of the stack.

#. Pop the value :math:`({\mathit{at}}{.}\CONST~i)` from the stack.

#. If :math:`i + n > {|z{.}\ZMEMS{}[x]{.}\MIBYTES|}`, then:

   a. Trap.

#. If :math:`j + n > {|z{.}\ZDATAS{}[y]{.}\DIBYTES|}`, then:

   a. Trap.

#. If :math:`n = 0`, then:

   a. Do nothing.

#. Else:

   a. Assert: Due to validation, :math:`j < {|z{.}\ZDATAS{}[y]{.}\DIBYTES|}`.

   #. Push the value :math:`({\mathit{at}}{.}\CONST~i)` to the stack.

   #. Push the value :math:`(\I32{.}\CONST~z{.}\ZDATAS{}[y]{.}\DIBYTES{}[j])` to the stack.

   #. Execute the instruction :math:`({\I32{.}\STORE}{8}~x)`.

   #. Push the value :math:`({\mathit{at}}{.}\CONST~i + 1)` to the stack.

   #. Push the value :math:`(\I32{.}\CONST~j + 1)` to the stack.

   #. Push the value :math:`(\I32{.}\CONST~n - 1)` to the stack.

   #. Execute the instruction :math:`(\MEMORYINIT~x~y)`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; ({\mathit{at}}{.}\CONST~i)~(\I32{.}\CONST~j)~(\I32{.}\CONST~n)~(\MEMORYINIT~x~y) & \stepto & \TRAP &  \\
   & \multicolumn{4}{@{}l@{}}{\quad
   \quad \mbox{if}~ i + n > {|z{.}\ZMEMS{}[x]{.}\MIBYTES|} \lor j + n > {|z{.}\ZDATAS{}[y]{.}\DIBYTES|}
   } \\
   & z ; ({\mathit{at}}{.}\CONST~i)~(\I32{.}\CONST~j)~(\I32{.}\CONST~n)~(\MEMORYINIT~x~y) & \stepto & \epsilon & \quad \mbox{otherwise, if}~ n = 0 \\
   & z ; ({\mathit{at}}{.}\CONST~i)~(\I32{.}\CONST~j)~(\I32{.}\CONST~n)~(\MEMORYINIT~x~y) & \stepto & & \\
   & \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}l@{}}
   \begin{array}[t]{@{}l@{}} ({\mathit{at}}{.}\CONST~i)~(\I32{.}\CONST~z{.}\ZDATAS{}[y]{.}\DIBYTES{}[j])~({\I32{.}\STORE}{\mathsf{{\scriptstyle 8}}}~x) \\
     ({\mathit{at}}{.}\CONST~i + 1)~(\I32{.}\CONST~j + 1)~(\I32{.}\CONST~n - 1)~(\MEMORYINIT~x~y) \end{array} & \quad \mbox{otherwise} \\
   \end{array}
   } \\
   \end{array}


.. _exec-data.drop:


:math:`\DATADROP~x`
...................


1. Let :math:`z` be the current state.

#. Replace :math:`z{.}\ZDATAS{}[x]{.}\mathsf{bytes}` with :math:`\epsilon`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; (\DATADROP~x) & \stepto & z{}[{.}\ZDDATAS{}[x]{.}\ZDBYTES = \epsilon] ; \epsilon \\
   \end{array}


.. index:: reference instructions, reference
   pair: execution; instruction
   single: abstract syntax; instruction
.. _exec-instr-ref:

Reference Instructions
~~~~~~~~~~~~~~~~~~~~~~

.. _exec-ref.null:


:math:`\REFNULL~{\mathit{ht}}`
..............................


1. Push the value :math:`\REFNULLADDR` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; (\REFNULL~{\mathit{ht}}) & \stepto & \REFNULLADDR \\
   \end{array}


.. _exec-ref.func:


:math:`\REFFUNC~x`
..................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, :math:`x < {|z{.}\ZMODULE{.}\MIFUNCS|}`.

#. Push the value :math:`(\REFFUNCADDR~z{.}\ZMODULE{.}\MIFUNCS{}[x])` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; (\REFFUNC~x) & \stepto & (\REFFUNCADDR~z{.}\ZMODULE{.}\MIFUNCS{}[x]) \\
   \end{array}


.. _exec-ref.is_null:


:math:`\REFISNULL`
..................


1. Assert: Due to validation, a :ref:`reference value <syntax-ref>` is on the top of the stack.

#. Pop the value :math:`{\reff}` from the stack.

#. If :math:`{\reff} = \REFNULLADDR`, then:

   a. Push the value :math:`(\I32{.}\CONST~1)` to the stack.

#. Else:

   a. Push the value :math:`(\I32{.}\CONST~0)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & {\reff}~\REFISNULL & \stepto & (\I32{.}\CONST~1) & \quad \mbox{if}~ {\reff} = \REFNULLADDR \\
   & {\reff}~\REFISNULL & \stepto & (\I32{.}\CONST~0) & \quad \mbox{otherwise} \\
   \end{array}


.. _exec-ref.as_non_null:


:math:`\REFASNONNULL`
.....................


1. Assert: Due to validation, a :ref:`reference value <syntax-ref>` is on the top of the stack.

#. Pop the value :math:`{\reff}` from the stack.

#. If :math:`{\reff} = \REFNULLADDR`, then:

   a. Trap.

#. Push the value :math:`{\reff}` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & {\reff}~\REFASNONNULL & \stepto & \TRAP & \quad \mbox{if}~ {\reff} = \REFNULLADDR \\
   & {\reff}~\REFASNONNULL & \stepto & {\reff} & \quad \mbox{otherwise} \\
   \end{array}


.. _exec-ref.eq:


:math:`\REFEQ`
..............


1. Assert: Due to validation, a :ref:`reference value <syntax-ref>` is on the top of the stack.

#. Pop the value :math:`{\reff}_2` from the stack.

#. Assert: Due to validation, a :ref:`reference value <syntax-ref>` is on the top of the stack.

#. Pop the value :math:`{\reff}_1` from the stack.

#. If :math:`{\reff}_1 = \REFNULLADDR` and :math:`{\reff}_2 = \REFNULLADDR`, then:

   a. Push the value :math:`(\I32{.}\CONST~1)` to the stack.

#. Else if :math:`{\reff}_1 = {\reff}_2`, then:

   a. Push the value :math:`(\I32{.}\CONST~1)` to the stack.

#. Else:

   a. Push the value :math:`(\I32{.}\CONST~0)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & {\reff}_1~{\reff}_2~\REFEQ & \stepto & (\I32{.}\CONST~1) & \quad \mbox{if}~ {\reff}_1 = \REFNULLADDR \land {\reff}_2 = \REFNULLADDR \\
   & {\reff}_1~{\reff}_2~\REFEQ & \stepto & (\I32{.}\CONST~1) & \quad \mbox{otherwise, if}~ {\reff}_1 = {\reff}_2 \\
   & {\reff}_1~{\reff}_2~\REFEQ & \stepto & (\I32{.}\CONST~0) & \quad \mbox{otherwise} \\
   \end{array}


.. _exec-ref.test:


:math:`\REFTEST~{\mathit{rt}}`
..............................


1. Let :math:`f` be the topmost :math:`\mathsf{frame}`.

#. Assert: Due to validation, a :ref:`reference value <syntax-ref>` is on the top of the stack.

#. Pop the value :math:`{\reff}` from the stack.

#. If :math:`{\reff}` is :ref:`valid <valid-val>` with type :math:`{{\insttype}}_{f{.}\AMODULE}({\mathit{rt}})`, then:

   a. Push the value :math:`(\I32{.}\CONST~1)` to the stack.

#. Else:

   a. Push the value :math:`(\I32{.}\CONST~0)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & s ; f ; {\reff}~(\REFTEST~{\mathit{rt}}) & \stepto & (\I32{.}\CONST~1) & \quad \mbox{if}~ s \vdashref {\reff} : {{\insttype}}_{f{.}\AMODULE}({\mathit{rt}}) \\
   & s ; f ; {\reff}~(\REFTEST~{\mathit{rt}}) & \stepto & (\I32{.}\CONST~0) & \quad \mbox{otherwise} \\
   \end{array}


.. _exec-ref.cast:


:math:`\REFCAST~{\mathit{rt}}`
..............................


1. Let :math:`f` be the topmost :math:`\mathsf{frame}`.

#. Assert: Due to validation, a :ref:`reference value <syntax-ref>` is on the top of the stack.

#. Pop the value :math:`{\reff}` from the stack.

#. If not :math:`{\reff}` is :ref:`valid <valid-val>` with type :math:`{{\insttype}}_{f{.}\AMODULE}({\mathit{rt}})`, then:

   a. Trap.

#. Push the value :math:`{\reff}` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & s ; f ; {\reff}~(\REFCAST~{\mathit{rt}}) & \stepto & {\reff} & \quad \mbox{if}~ s \vdashref {\reff} : {{\insttype}}_{f{.}\AMODULE}({\mathit{rt}}) \\
   & s ; f ; {\reff}~(\REFCAST~{\mathit{rt}}) & \stepto & \TRAP & \quad \mbox{otherwise} \\
   \end{array}


.. _exec-ref.i31:


:math:`\REFI31`
...............


1. Assert: Due to validation, a value of :ref:`number type <syntax-numtype>` :math:`\I32` is on the top of the stack.

#. Pop the value :math:`(\I32{.}\CONST~i)` from the stack.

#. Push the value :math:`(\REFI31NUM~{{\wrap}}_{32, 31}(i))` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & (\I32{.}\CONST~i)~\REFI31 & \stepto & (\REFI31NUM~{{\wrap}}_{32, 31}(i)) \\
   \end{array}


.. _exec-i31.get:


:math:`{\I31GET}{\mathsf{\_}}{{\sx}}`
.....................................


1. Assert: Due to validation, a value is on the top of the stack.

#. Pop the value :math:`{\val}` from the stack.

#. If :math:`{\val} = \REFNULLADDR`, then:

   a. Trap.

#. Assert: Due to validation, :math:`{\val}` is some :math:`\REFI31NUM~{\u31}`.

#. Let :math:`(\REFI31NUM~i)` be the destructuring of :math:`{\val}`.

#. Push the value :math:`(\I32{.}\CONST~{{{{\extend}}_{31, 32}^{{\sx}}}}{(i)})` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & (\REFNULLADDR)~({\I31GET}{\mathsf{\_}}{{\sx}}) & \stepto & \TRAP \\
   & (\REFI31NUM~i)~({\I31GET}{\mathsf{\_}}{{\sx}}) & \stepto & (\I32{.}\CONST~{{{{\extend}}_{31, 32}^{{\sx}}}}{(i)}) \\
   \end{array}


.. _exec-struct.new:


:math:`\STRUCTNEW~x`
....................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, the :ref:`expansion <aux-expand-deftype>` of :math:`z{.}\ZTYPES{}[x]` is some :math:`\TSTRUCT~{\list}({\fieldtype})`.

#. Let :math:`(\TSTRUCT~{\list}_0)` be the destructuring of the :ref:`expansion <aux-expand-deftype>` of :math:`z{.}\ZTYPES{}[x]`.

#. Let :math:`{({\TMUT^?}~{\mathit{zt}})^{n}}` be :math:`{\list}_0`.

#. Let :math:`a` be the length of :math:`z{.}\ZSTRUCTS`.

#. Assert: Due to validation, there are at least :math:`n` values on the top of the stack.

#. Pop the values :math:`{{\val}^{n}}` from the stack.

#. Let :math:`{\mathit{si}}` be the :ref:`structure instance <syntax-structinst>` :math:`\{ \SITYPE~z{.}\ZTYPES{}[x],\;\allowbreak \SIFIELDS~{{{\packfield}}_{{\mathit{zt}}}({\val})^{n}} \}`.

#. Push the value :math:`(\REFSTRUCTADDR~a)` to the stack.

#. Append :math:`{\mathit{si}}` to :math:`z{.}\ZSTRUCTS`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; {{\val}^{n}}~(\STRUCTNEW~x) & \stepto & z{}[{.}\ZSTRUCTS \mathrel{{=}{\oplus}} {\mathit{si}}] ; (\REFSTRUCTADDR~a) & \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ z{.}\ZTYPES{}[x] \approxexpanddt \TSTRUCT~{({\TMUT^?}~{\mathit{zt}})^{n}} \\
   {\land}~ a = {|z{.}\ZSTRUCTS|} \\
   {\land}~ {\mathit{si}} = \{ \SITYPE~z{.}\ZTYPES{}[x],\;\allowbreak \SIFIELDS~{({{\packfield}}_{{\mathit{zt}}}({\val}))^{n}} \} \\
   \end{array} \\
   \end{array}


.. _exec-struct.new_default:


:math:`\STRUCTNEWDEFAULT~x`
...........................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, the :ref:`expansion <aux-expand-deftype>` of :math:`z{.}\ZTYPES{}[x]` is some :math:`\TSTRUCT~{\list}({\fieldtype})`.

#. Let :math:`(\TSTRUCT~{\list}_0)` be the destructuring of the :ref:`expansion <aux-expand-deftype>` of :math:`z{.}\ZTYPES{}[x]`.

#. Let :math:`{({\TMUT^?}~{\mathit{zt}})^\ast}` be :math:`{\list}_0`.

#. Assert: Due to validation, for all :math:`{\mathit{zt}}` in :math:`{{\mathit{zt}}^\ast}`, :math:`{{\default}}_{{\unpack}({\mathit{zt}})}` is defined.

#. Let :math:`{{\val}^\ast}` be the value sequence :math:`\epsilon`.

#. For each :math:`{\mathit{zt}}` in :math:`{{\mathit{zt}}^\ast}`, do:

   a. Let :math:`{\val}` be :math:`{{\default}}_{{\unpack}({\mathit{zt}})}`.

   #. Append :math:`{\val}` to :math:`{{\val}^\ast}`.

#. Assert: Due to validation, :math:`{|{{\val}^\ast}|} = {|{{\mathit{zt}}^\ast}|}`.

#. Push the values :math:`{{\val}^\ast}` to the stack.

#. Execute the instruction :math:`(\STRUCTNEW~x)`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; (\STRUCTNEWDEFAULT~x) & \stepto & {{\val}^\ast}~(\STRUCTNEW~x) & \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ z{.}\ZTYPES{}[x] \approxexpanddt \TSTRUCT~{({\TMUT^?}~{\mathit{zt}})^\ast} \\
   {\land}~ ({{\default}}_{{\unpack}({\mathit{zt}})} = {\val})^\ast \\
   \end{array} \\
   \end{array}


.. _exec-struct.get:
.. _exec-struct.get_sx:


:math:`{\STRUCTGET}{\mathsf{\_}}{{{\sx}^?}}~x~i`
................................................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, a value is on the top of the stack.

#. Pop the value :math:`{\val}` from the stack.

#. If :math:`{\val} = \REFNULLADDR`, then:

   a. Trap.

#. Assert: Due to validation, :math:`{\val}` is some :math:`\REFSTRUCTADDR~{\structaddr}`.

#. Let :math:`(\REFSTRUCTADDR~a)` be the destructuring of :math:`{\val}`.

#. Assert: Due to validation, :math:`i < {|z{.}\ZSTRUCTS{}[a]{.}\SIFIELDS|}`.

#. Assert: Due to validation, :math:`a < {|z{.}\ZSTRUCTS|}`.

#. Assert: Due to validation, the :ref:`expansion <aux-expand-deftype>` of :math:`z{.}\ZTYPES{}[x]` is some :math:`\TSTRUCT~{\list}({\fieldtype})`.

#. Let :math:`(\TSTRUCT~{\list}_0)` be the destructuring of the :ref:`expansion <aux-expand-deftype>` of :math:`z{.}\ZTYPES{}[x]`.

#. Let :math:`{({\TMUT^?}~{\mathit{zt}})^\ast}` be :math:`{\list}_0`.

#. Assert: Due to validation, :math:`i < {|{{\mathit{zt}}^\ast}|}`.

#. Push the value :math:`{{{{\unpackfield}}_{{{\mathit{zt}}^\ast}{}[i]}^{{{\sx}^?}}}}{(z{.}\ZSTRUCTS{}[a]{.}\SIFIELDS{}[i])}` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; (\REFNULLADDR)~({\STRUCTGET}{\mathsf{\_}}{{{\sx}^?}}~x~i) & \stepto & \TRAP \\
   & z ; (\REFSTRUCTADDR~a)~({\STRUCTGET}{\mathsf{\_}}{{{\sx}^?}}~x~i) & \stepto & {{{{\unpackfield}}_{{{\mathit{zt}}^\ast}{}[i]}^{{{\sx}^?}}}}{(z{.}\ZSTRUCTS{}[a]{.}\SIFIELDS{}[i])} &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad \mbox{if}~ z{.}\ZTYPES{}[x] \approxexpanddt \TSTRUCT~{({\TMUT^?}~{\mathit{zt}})^\ast}
   } \\
   \end{array}


.. _exec-struct.set:


:math:`\STRUCTSET~x~i`
......................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, a value is on the top of the stack.

#. Pop the value :math:`{\val}` from the stack.

#. Assert: Due to validation, a value is on the top of the stack.

#. Pop the value :math:`{\val'}` from the stack.

#. If :math:`{\val'} = \REFNULLADDR`, then:

   a. Trap.

#. Assert: Due to validation, :math:`{\val'}` is some :math:`\REFSTRUCTADDR~{\structaddr}`.

#. Let :math:`(\REFSTRUCTADDR~a)` be the destructuring of :math:`{\val'}`.

#. Assert: Due to validation, the :ref:`expansion <aux-expand-deftype>` of :math:`z{.}\ZTYPES{}[x]` is some :math:`\TSTRUCT~{\list}({\fieldtype})`.

#. Let :math:`(\TSTRUCT~{\list}_0)` be the destructuring of the :ref:`expansion <aux-expand-deftype>` of :math:`z{.}\ZTYPES{}[x]`.

#. Let :math:`{({\TMUT^?}~{\mathit{zt}})^\ast}` be :math:`{\list}_0`.

#. Assert: Due to validation, :math:`i < {|{{\mathit{zt}}^\ast}|}`.

#. Replace :math:`z{.}\ZSTRUCTS{}[a]{.}\mathsf{fields}{}[i]` with :math:`{{\packfield}}_{{{\mathit{zt}}^\ast}{}[i]}({\val})`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; (\REFNULLADDR)~{\val}~(\STRUCTSET~x~i) & \stepto & z ; \TRAP \\
   & z ; (\REFSTRUCTADDR~a)~{\val}~(\STRUCTSET~x~i) & \stepto & z{}[{.}\ZSSTRUCTS{}[a]{.}\ZSFIELDS{}[i] = {{\packfield}}_{{{\mathit{zt}}^\ast}{}[i]}({\val})] ; \epsilon &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad \mbox{if}~ z{.}\ZTYPES{}[x] \approxexpanddt \TSTRUCT~{({\TMUT^?}~{\mathit{zt}})^\ast}
   } \\
   \end{array}
   

.. _exec-array.new:


:math:`\ARRAYNEW~x`
...................


1. Assert: Due to validation, a value of :ref:`number type <syntax-numtype>` :math:`\I32` is on the top of the stack.

#. Pop the value :math:`(\I32{.}\CONST~n)` from the stack.

#. Assert: Due to validation, a value is on the top of the stack.

#. Pop the value :math:`{\val}` from the stack.

#. Push the values :math:`{{\val}^{n}}` to the stack.

#. Execute the instruction :math:`(\ARRAYNEWFIXED~x~n)`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & {\val}~(\I32{.}\CONST~n)~(\ARRAYNEW~x) & \stepto & {{\val}^{n}}~(\ARRAYNEWFIXED~x~n) \\
   \end{array}


.. _exec-array.new_default:


:math:`\ARRAYNEWDEFAULT~x`
..........................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, a value of :ref:`number type <syntax-numtype>` :math:`\I32` is on the top of the stack.

#. Pop the value :math:`(\I32{.}\CONST~n)` from the stack.

#. Assert: Due to validation, the :ref:`expansion <aux-expand-deftype>` of :math:`z{.}\ZTYPES{}[x]` is some :math:`\TARRAY~{\fieldtype}`.

#. Let :math:`(\TARRAY~{\fieldtype}_0)` be the destructuring of the :ref:`expansion <aux-expand-deftype>` of :math:`z{.}\ZTYPES{}[x]`.

#. Let :math:`({\TMUT^?}~{\mathit{zt}})` be the destructuring of :math:`{\fieldtype}_0`.

#. Assert: Due to validation, :math:`{{\default}}_{{\unpack}({\mathit{zt}})}` is defined.

#. Let :math:`{\val}` be :math:`{{\default}}_{{\unpack}({\mathit{zt}})}`.

#. Push the values :math:`{{\val}^{n}}` to the stack.

#. Execute the instruction :math:`(\ARRAYNEWFIXED~x~n)`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; (\I32{.}\CONST~n)~(\ARRAYNEWDEFAULT~x) & \stepto & {{\val}^{n}}~(\ARRAYNEWFIXED~x~n) & \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ z{.}\ZTYPES{}[x] \approxexpanddt \TARRAY~({\TMUT^?}~{\mathit{zt}}) \\
   {\land}~ {{\default}}_{{\unpack}({\mathit{zt}})} = {\val} \\
   \end{array} \\
   \end{array}


.. _exec-array.new_fixed:


:math:`\ARRAYNEWFIXED~x~n`
..........................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, the :ref:`expansion <aux-expand-deftype>` of :math:`z{.}\ZTYPES{}[x]` is some :math:`\TARRAY~{\fieldtype}`.

#. Let :math:`(\TARRAY~{\fieldtype}_0)` be the destructuring of the :ref:`expansion <aux-expand-deftype>` of :math:`z{.}\ZTYPES{}[x]`.

#. Let :math:`({\TMUT^?}~{\mathit{zt}})` be the destructuring of :math:`{\fieldtype}_0`.

#. Let :math:`a` be the length of :math:`z{.}\ZARRAYS`.

#. Assert: Due to validation, there are at least :math:`n` values on the top of the stack.

#. Pop the values :math:`{{\val}^{n}}` from the stack.

#. Let :math:`{\mathit{ai}}` be the :ref:`array instance <syntax-arrayinst>` :math:`\{ \AITYPE~z{.}\ZTYPES{}[x],\;\allowbreak \AIFIELDS~{{{\packfield}}_{{\mathit{zt}}}({\val})^{n}} \}`.

#. Push the value :math:`(\REFARRAYADDR~a)` to the stack.

#. Append :math:`{\mathit{ai}}` to :math:`z{.}\ZARRAYS`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; {{\val}^{n}}~(\ARRAYNEWFIXED~x~n) & \stepto & z{}[{.}\ZARRAYS \mathrel{{=}{\oplus}} {\mathit{ai}}] ; (\REFARRAYADDR~a) &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ z{.}\ZTYPES{}[x] \approxexpanddt \TARRAY~({\TMUT^?}~{\mathit{zt}}) \\
   {\land}~ a = {|z{.}\ZARRAYS|} \land {\mathit{ai}} = \{ \AITYPE~z{.}\ZTYPES{}[x],\;\allowbreak \AIFIELDS~{({{\packfield}}_{{\mathit{zt}}}({\val}))^{n}} \} \\
   \end{array}
   } \\
   \end{array}


.. _exec-array.new_data:


:math:`\ARRAYNEWDATA~x~y`
.........................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, a value of :ref:`number type <syntax-numtype>` :math:`\I32` is on the top of the stack.

#. Pop the value :math:`(\I32{.}\CONST~n)` from the stack.

#. Assert: Due to validation, a value of :ref:`number type <syntax-numtype>` :math:`\I32` is on the top of the stack.

#. Pop the value :math:`(\I32{.}\CONST~i)` from the stack.

#. Assert: Due to validation, the :ref:`expansion <aux-expand-deftype>` of :math:`z{.}\ZTYPES{}[x]` is some :math:`\TARRAY~{\fieldtype}`.

#. Let :math:`(\TARRAY~{\fieldtype}_0)` be the destructuring of the :ref:`expansion <aux-expand-deftype>` of :math:`z{.}\ZTYPES{}[x]`.

#. Let :math:`({\TMUT^?}~{\mathit{zt}})` be the destructuring of :math:`{\fieldtype}_0`.

#. If :math:`i + n \cdot {|{\mathit{zt}}|} / 8 > {|z{.}\ZDATAS{}[y]{.}\DIBYTES|}`, then:

   a. Trap.

#. Let :math:`{{{\byte}^\ast}^\ast}` be the result for which each :math:`{{\byte}^\ast}` has length :math:`{|{\mathit{zt}}|} / 8`, and the :ref:`concatenation <notation-concat>` of :math:`{{{\byte}^\ast}^\ast}` is :math:`z{.}\ZDATAS{}[y]{.}\DIBYTES{}[i : n \cdot {|{\mathit{zt}}|} / 8]`.

#. Let :math:`{c^{n}}` be the result for which :math:`{({{\bytes}}_{{\mathit{zt}}}({c^{n}}) = {{\byte}^\ast})^\ast}`.

#. Push the values :math:`{{\unpack}({\mathit{zt}}){.}\CONST~{{\unpacknum}}_{{\mathit{zt}}}(c)^{n}}` to the stack.

#. Execute the instruction :math:`(\ARRAYNEWFIXED~x~n)`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; (\I32{.}\CONST~i)~(\I32{.}\CONST~n)~(\ARRAYNEWDATA~x~y) & \stepto & \TRAP &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ z{.}\ZTYPES{}[x] \approxexpanddt \TARRAY~({\TMUT^?}~{\mathit{zt}}) \\
   {\land}~ i + n \cdot {|{\mathit{zt}}|} / 8 > {|z{.}\ZDATAS{}[y]{.}\DIBYTES|} \\
   \end{array}
   } \\
   & z ; (\I32{.}\CONST~i)~(\I32{.}\CONST~n)~(\ARRAYNEWDATA~x~y) & \stepto & {({\unpack}({\mathit{zt}}){.}\CONST~{{\unpacknum}}_{{\mathit{zt}}}(c))^{n}}~(\ARRAYNEWFIXED~x~n) &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ z{.}\ZTYPES{}[x] \approxexpanddt \TARRAY~({\TMUT^?}~{\mathit{zt}}) \\
   {\land}~ {\bigcat}\, {{{\bytes}}_{{\mathit{zt}}}(c)^{n}} = z{.}\ZDATAS{}[y]{.}\DIBYTES{}[i : n \cdot {|{\mathit{zt}}|} / 8] \\
   \end{array}
   } \\
   \end{array}


.. _exec-array.new_elem:


:math:`\ARRAYNEWELEM~x~y`
.........................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, a value of :ref:`number type <syntax-numtype>` :math:`\I32` is on the top of the stack.

#. Pop the value :math:`(\I32{.}\CONST~n)` from the stack.

#. Assert: Due to validation, a value of :ref:`number type <syntax-numtype>` :math:`\I32` is on the top of the stack.

#. Pop the value :math:`(\I32{.}\CONST~i)` from the stack.

#. If :math:`i + n > {|z{.}\ZELEMS{}[y]{.}\EIREFS|}`, then:

   a. Trap.

#. Let :math:`{{\reff}^{n}}` be :math:`z{.}\ZELEMS{}[y]{.}\EIREFS{}[i : n]`.

#. Push the values :math:`{{\reff}^{n}}` to the stack.

#. Execute the instruction :math:`(\ARRAYNEWFIXED~x~n)`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; (\I32{.}\CONST~i)~(\I32{.}\CONST~n)~(\ARRAYNEWELEM~x~y) & \stepto & \TRAP & \quad \mbox{if}~ i + n > {|z{.}\ZELEMS{}[y]{.}\EIREFS|} \\
   & z ; (\I32{.}\CONST~i)~(\I32{.}\CONST~n)~(\ARRAYNEWELEM~x~y) & \stepto & {{\reff}^{n}}~(\ARRAYNEWFIXED~x~n) &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad \mbox{if}~ {{\reff}^{n}} = z{.}\ZELEMS{}[y]{.}\EIREFS{}[i : n]
   } \\
   \end{array}


.. _exec-array.get:
.. _exec-array.get_sx:


:math:`{\ARRAYGET}{\mathsf{\_}}{{{\sx}^?}}~x`
.............................................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, a value of :ref:`number type <syntax-numtype>` :math:`\I32` is on the top of the stack.

#. Pop the value :math:`(\I32{.}\CONST~i)` from the stack.

#. Assert: Due to validation, a value is on the top of the stack.

#. Pop the value :math:`{\val}` from the stack.

#. If :math:`{\val} = \REFNULLADDR`, then:

   a. Trap.

#. Assert: Due to validation, :math:`{\val}` is some :math:`\REFARRAYADDR~{\arrayaddr}`.

#. Let :math:`(\REFARRAYADDR~a)` be the destructuring of :math:`{\val}`.

#. Assert: Due to validation, :math:`a < {|z{.}\ZARRAYS|}`.

#. If :math:`i \geq {|z{.}\ZARRAYS{}[a]{.}\AIFIELDS|}`, then:

   a. Trap.

#. Assert: Due to validation, the :ref:`expansion <aux-expand-deftype>` of :math:`z{.}\ZTYPES{}[x]` is some :math:`\TARRAY~{\fieldtype}`.

#. Let :math:`(\TARRAY~{\fieldtype}_0)` be the destructuring of the :ref:`expansion <aux-expand-deftype>` of :math:`z{.}\ZTYPES{}[x]`.

#. Let :math:`({\TMUT^?}~{\mathit{zt}})` be the destructuring of :math:`{\fieldtype}_0`.

#. Push the value :math:`{{{{\unpackfield}}_{{\mathit{zt}}}^{{{\sx}^?}}}}{(z{.}\ZARRAYS{}[a]{.}\AIFIELDS{}[i])}` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; (\REFNULLADDR)~(\I32{.}\CONST~i)~({\ARRAYGET}{\mathsf{\_}}{{{\sx}^?}}~x) & \stepto & \TRAP \\
   & z ; (\REFARRAYADDR~a)~(\I32{.}\CONST~i)~({\ARRAYGET}{\mathsf{\_}}{{{\sx}^?}}~x) & \stepto & \TRAP & \quad \mbox{if}~ i \geq {|z{.}\ZARRAYS{}[a]{.}\AIFIELDS|} \\
   & z ; (\REFARRAYADDR~a)~(\I32{.}\CONST~i)~({\ARRAYGET}{\mathsf{\_}}{{{\sx}^?}}~x) & \stepto & {{{{\unpackfield}}_{{\mathit{zt}}}^{{{\sx}^?}}}}{(z{.}\ZARRAYS{}[a]{.}\AIFIELDS{}[i])} &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad \mbox{if}~ z{.}\ZTYPES{}[x] \approxexpanddt \TARRAY~({\TMUT^?}~{\mathit{zt}})
   } \\
   \end{array}


.. _exec-array.set:


:math:`\ARRAYSET~x`
...................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, a value is on the top of the stack.

#. Pop the value :math:`{\val}` from the stack.

#. Assert: Due to validation, a value of :ref:`number type <syntax-numtype>` :math:`\I32` is on the top of the stack.

#. Pop the value :math:`(\I32{.}\CONST~i)` from the stack.

#. Assert: Due to validation, a value is on the top of the stack.

#. Pop the value :math:`{\val'}` from the stack.

#. If :math:`{\val'} = \REFNULLADDR`, then:

   a. Trap.

#. Assert: Due to validation, :math:`{\val'}` is some :math:`\REFARRAYADDR~{\arrayaddr}`.

#. Let :math:`(\REFARRAYADDR~a)` be the destructuring of :math:`{\val'}`.

#. If :math:`a < {|z{.}\ZARRAYS|}` and :math:`i \geq {|z{.}\ZARRAYS{}[a]{.}\AIFIELDS|}`, then:

   a. Trap.

#. Assert: Due to validation, the :ref:`expansion <aux-expand-deftype>` of :math:`z{.}\ZTYPES{}[x]` is some :math:`\TARRAY~{\fieldtype}`.

#. Let :math:`(\TARRAY~{\fieldtype}_0)` be the destructuring of the :ref:`expansion <aux-expand-deftype>` of :math:`z{.}\ZTYPES{}[x]`.

#. Let :math:`({\TMUT^?}~{\mathit{zt}})` be the destructuring of :math:`{\fieldtype}_0`.

#. Replace :math:`z{.}\ZARRAYS{}[a]{.}\mathsf{fields}{}[i]` with :math:`{{\packfield}}_{{\mathit{zt}}}({\val})`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; (\REFNULLADDR)~(\I32{.}\CONST~i)~{\val}~(\ARRAYSET~x) & \stepto & z ; \TRAP \\
   & z ; (\REFARRAYADDR~a)~(\I32{.}\CONST~i)~{\val}~(\ARRAYSET~x) & \stepto & z ; \TRAP & \quad \mbox{if}~ i \geq {|z{.}\ZARRAYS{}[a]{.}\AIFIELDS|} \\
   & z ; (\REFARRAYADDR~a)~(\I32{.}\CONST~i)~{\val}~(\ARRAYSET~x) & \stepto & z{}[{.}\ZAARRAYS{}[a]{.}\ZAFIELDS{}[i] = {{\packfield}}_{{\mathit{zt}}}({\val})] ; \epsilon &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad \mbox{if}~ z{.}\ZTYPES{}[x] \approxexpanddt \TARRAY~({\TMUT^?}~{\mathit{zt}})
   } \\
   \end{array}


.. _exec-array.len:


:math:`\ARRAYLEN`
.................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, a value is on the top of the stack.

#. Pop the value :math:`{\val}` from the stack.

#. If :math:`{\val} = \REFNULLADDR`, then:

   a. Trap.

#. Assert: Due to validation, :math:`{\val}` is some :math:`\REFARRAYADDR~{\arrayaddr}`.

#. Let :math:`(\REFARRAYADDR~a)` be the destructuring of :math:`{\val}`.

#. Assert: Due to validation, :math:`a < {|z{.}\ZARRAYS|}`.

#. Push the value :math:`(\I32{.}\CONST~{|z{.}\ZARRAYS{}[a]{.}\AIFIELDS|})` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; (\REFNULLADDR)~\ARRAYLEN & \stepto & \TRAP \\
   & z ; (\REFARRAYADDR~a)~\ARRAYLEN & \stepto & (\I32{.}\CONST~{|z{.}\ZARRAYS{}[a]{.}\AIFIELDS|}) \\
   \end{array}


.. _exec-array.fill:


:math:`\ARRAYFILL~x`
....................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, a value of :ref:`number type <syntax-numtype>` :math:`\I32` is on the top of the stack.

#. Pop the value :math:`(\I32{.}\CONST~n)` from the stack.

#. Assert: Due to validation, a value is on the top of the stack.

#. Pop the value :math:`{\val}` from the stack.

#. Assert: Due to validation, a value of :ref:`number type <syntax-numtype>` :math:`\I32` is on the top of the stack.

#. Pop the value :math:`(\I32{.}\CONST~i)` from the stack.

#. Assert: Due to validation, a value is on the top of the stack.

#. Pop the value :math:`{\val'}` from the stack.

#. If :math:`{\val'} = \REFNULLADDR`, then:

   a. Trap.

#. Assert: Due to validation, :math:`{\val'}` is some :math:`\REFARRAYADDR~{\arrayaddr}`.

#. Let :math:`(\REFARRAYADDR~a)` be the destructuring of :math:`{\val'}`.

#. If :math:`a \geq {|z{.}\ZARRAYS|}`, then:

   a. Do nothing.

#. Else if :math:`i + n > {|z{.}\ZARRAYS{}[a]{.}\AIFIELDS|}`, then:

   a. Trap.

#. If :math:`n = 0`, then:

   a. Do nothing.

#. Else:

   a. Push the value :math:`(\REFARRAYADDR~a)` to the stack.

   #. Push the value :math:`(\I32{.}\CONST~i)` to the stack.

   #. Push the value :math:`{\val}` to the stack.

   #. Execute the instruction :math:`(\ARRAYSET~x)`.

   #. Push the value :math:`(\REFARRAYADDR~a)` to the stack.

   #. Push the value :math:`(\I32{.}\CONST~i + 1)` to the stack.

   #. Push the value :math:`{\val}` to the stack.

   #. Push the value :math:`(\I32{.}\CONST~n - 1)` to the stack.

   #. Execute the instruction :math:`(\ARRAYFILL~x)`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; (\REFNULLADDR)~(\I32{.}\CONST~i)~{\val}~(\I32{.}\CONST~n)~(\ARRAYFILL~x) & \stepto & \TRAP \\
   & z ; (\REFARRAYADDR~a)~(\I32{.}\CONST~i)~{\val}~(\I32{.}\CONST~n)~(\ARRAYFILL~x) & \stepto & \TRAP & \quad \mbox{if}~ i + n > {|z{.}\ZARRAYS{}[a]{.}\AIFIELDS|} \\
   & z ; (\REFARRAYADDR~a)~(\I32{.}\CONST~i)~{\val}~(\I32{.}\CONST~n)~(\ARRAYFILL~x) & \stepto & \epsilon & \quad \mbox{otherwise, if}~ n = 0 \\
   & z ; (\REFARRAYADDR~a)~(\I32{.}\CONST~i)~{\val}~(\I32{.}\CONST~n)~(\ARRAYFILL~x) & \stepto & & \\
   & \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}l@{}}
   \begin{array}[t]{@{}l@{}} (\REFARRAYADDR~a)~(\I32{.}\CONST~i)~{\val}~(\ARRAYSET~x) \\
     (\REFARRAYADDR~a)~(\I32{.}\CONST~i + 1)~{\val}~(\I32{.}\CONST~n - 1)~(\ARRAYFILL~x) \end{array} & \quad \mbox{otherwise} \\
   \end{array}
   } \\
   \end{array}


.. _exec-array.copy:


:math:`\ARRAYCOPY~x_1~x_2`
..........................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, a value of :ref:`number type <syntax-numtype>` :math:`\I32` is on the top of the stack.

#. Pop the value :math:`(\I32{.}\CONST~n)` from the stack.

#. Assert: Due to validation, a value of :ref:`number type <syntax-numtype>` :math:`\I32` is on the top of the stack.

#. Pop the value :math:`(\I32{.}\CONST~i_2)` from the stack.

#. Assert: Due to validation, a value is on the top of the stack.

#. Pop the value :math:`{\val}` from the stack.

#. Assert: Due to validation, a value of :ref:`number type <syntax-numtype>` :math:`\I32` is on the top of the stack.

#. Pop the value :math:`(\I32{.}\CONST~i_1)` from the stack.

#. Assert: Due to validation, a value is on the top of the stack.

#. Pop the value :math:`{\val'}` from the stack.

#. If :math:`{\val'} = \REFNULLADDR` and :math:`{\val}` is reference value, then:

   a. Trap.

#. If :math:`{\val} = \REFNULLADDR` and :math:`{\val'}` is reference value, then:

   a. Trap.

#. If :math:`{\val'}` is some :math:`\REFARRAYADDR~{\arrayaddr}`, then:

   a. Let :math:`(\REFARRAYADDR~a_1)` be the destructuring of :math:`{\val'}`.

   #. If :math:`{\val}` is some :math:`\REFARRAYADDR~{\arrayaddr}`, then:

      1) If :math:`a_1 < {|z{.}\ZARRAYS|}` and :math:`i_1 + n > {|z{.}\ZARRAYS{}[a_1]{.}\AIFIELDS|}`, then:

         a) Trap.

      #) Let :math:`(\REFARRAYADDR~a_2)` be the destructuring of :math:`{\val}`.

      #) If :math:`a_2 \geq {|z{.}\ZARRAYS|}`, then:

         a) Do nothing.

      #) Else if :math:`i_2 + n > {|z{.}\ZARRAYS{}[a_2]{.}\AIFIELDS|}`, then:

         a) Trap.

      #) If :math:`n = 0`, then:

         a) Do nothing.

      #) Else:

         a) Assert: Due to validation, the :ref:`expansion <aux-expand-deftype>` of :math:`z{.}\ZTYPES{}[x_2]` is some :math:`\TARRAY~{\fieldtype}`.

         #) Let :math:`(\TARRAY~{\fieldtype}_0)` be the destructuring of the :ref:`expansion <aux-expand-deftype>` of :math:`z{.}\ZTYPES{}[x_2]`.

         #) Let :math:`({\TMUT^?}~{\mathit{zt}}_2)` be the destructuring of :math:`{\fieldtype}_0`.

         #) Let :math:`{{\sx}^?}` be :math:`{\sx}({\mathit{zt}}_2)`.

         #) Push the value :math:`(\REFARRAYADDR~a_1)` to the stack.

         #) If :math:`i_1 \leq i_2`, then:

            1. Push the value :math:`(\I32{.}\CONST~i_1)` to the stack.

            #. Push the value :math:`(\REFARRAYADDR~a_2)` to the stack.

            #. Push the value :math:`(\I32{.}\CONST~i_2)` to the stack.

            #. Execute the instruction :math:`({\ARRAYGET}{\mathsf{\_}}{{{\sx}^?}}~x_2)`.

            #. Execute the instruction :math:`(\ARRAYSET~x_1)`.

            #. Push the value :math:`(\REFARRAYADDR~a_1)` to the stack.

            #. Push the value :math:`(\I32{.}\CONST~i_1 + 1)` to the stack.

            #. Push the value :math:`(\REFARRAYADDR~a_2)` to the stack.

            #. Push the value :math:`(\I32{.}\CONST~i_2 + 1)` to the stack.

         #) Else:

            1. Push the value :math:`(\I32{.}\CONST~i_1 + n - 1)` to the stack.

            #. Push the value :math:`(\REFARRAYADDR~a_2)` to the stack.

            #. Push the value :math:`(\I32{.}\CONST~i_2 + n - 1)` to the stack.

            #. Execute the instruction :math:`({\ARRAYGET}{\mathsf{\_}}{{{\sx}^?}}~x_2)`.

            #. Execute the instruction :math:`(\ARRAYSET~x_1)`.

            #. Push the value :math:`(\REFARRAYADDR~a_1)` to the stack.

            #. Push the value :math:`(\I32{.}\CONST~i_1)` to the stack.

            #. Push the value :math:`(\REFARRAYADDR~a_2)` to the stack.

            #. Push the value :math:`(\I32{.}\CONST~i_2)` to the stack.

         #) Push the value :math:`(\I32{.}\CONST~n - 1)` to the stack.

         #) Execute the instruction :math:`(\ARRAYCOPY~x_1~x_2)`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; (\REFNULLADDR)~(\I32{.}\CONST~i_1)~{\reff}~(\I32{.}\CONST~i_2)~(\I32{.}\CONST~n)~(\ARRAYCOPY~x_1~x_2) & \stepto & \TRAP \\
   & z ; {\reff}~(\I32{.}\CONST~i_1)~(\REFNULLADDR)~(\I32{.}\CONST~i_2)~(\I32{.}\CONST~n)~(\ARRAYCOPY~x_1~x_2) & \stepto & \TRAP \\
   & z ; (\REFARRAYADDR~a_1)~(\I32{.}\CONST~i_1)~(\REFARRAYADDR~a_2)~(\I32{.}\CONST~i_2)~(\I32{.}\CONST~n)~(\ARRAYCOPY~x_1~x_2) & \stepto & \TRAP &  \\
   & \multicolumn{4}{@{}l@{}}{\quad
   \quad \mbox{if}~ i_1 + n > {|z{.}\ZARRAYS{}[a_1]{.}\AIFIELDS|}
   } \\
   & z ; (\REFARRAYADDR~a_1)~(\I32{.}\CONST~i_1)~(\REFARRAYADDR~a_2)~(\I32{.}\CONST~i_2)~(\I32{.}\CONST~n)~(\ARRAYCOPY~x_1~x_2) & \stepto & \TRAP &  \\
   & \multicolumn{4}{@{}l@{}}{\quad
   \quad \mbox{if}~ i_2 + n > {|z{.}\ZARRAYS{}[a_2]{.}\AIFIELDS|}
   } \\
   & z ; (\REFARRAYADDR~a_1)~(\I32{.}\CONST~i_1)~(\REFARRAYADDR~a_2)~(\I32{.}\CONST~i_2)~(\I32{.}\CONST~n)~(\ARRAYCOPY~x_1~x_2) & \stepto & \epsilon &  \\
   & \multicolumn{4}{@{}l@{}}{\quad
   \quad \mbox{otherwise, if}~ n = 0
   } \\
   & z ; (\REFARRAYADDR~a_1)~(\I32{.}\CONST~i_1)~(\REFARRAYADDR~a_2)~(\I32{.}\CONST~i_2)~(\I32{.}\CONST~n)~(\ARRAYCOPY~x_1~x_2) & \stepto & & \\
   & \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}l@{}l@{}}
   \begin{array}[t]{@{}l@{}} (\REFARRAYADDR~a_1)~(\I32{.}\CONST~i_1) \\
     (\REFARRAYADDR~a_2)~(\I32{.}\CONST~i_2) \\
     ({\ARRAYGET}{\mathsf{\_}}{{{\sx}^?}}~x_2)~(\ARRAYSET~x_1) \\
     (\REFARRAYADDR~a_1)~(\I32{.}\CONST~i_1 + 1)~(\REFARRAYADDR~a_2)~(\I32{.}\CONST~i_2 + 1)~(\I32{.}\CONST~n - 1)~(\ARRAYCOPY~x_1~x_2) \end{array} & & \\
    \multicolumn{3}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{otherwise, if}~ z{.}\ZTYPES{}[x_2] \approxexpanddt \TARRAY~({\TMUT^?}~{\mathit{zt}}_2) \\
   {\land}~ i_1 \leq i_2 \land {{\sx}^?} = {\sx}({\mathit{zt}}_2) \\
   \end{array} \\
   \end{array}
   } \\
   \end{array}
   } \\
   & z ; (\REFARRAYADDR~a_1)~(\I32{.}\CONST~i_1)~(\REFARRAYADDR~a_2)~(\I32{.}\CONST~i_2)~(\I32{.}\CONST~n)~(\ARRAYCOPY~x_1~x_2) & \stepto & & \\
   & \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}l@{}l@{}}
   \begin{array}[t]{@{}l@{}} (\REFARRAYADDR~a_1)~(\I32{.}\CONST~i_1 + n - 1) \\
     (\REFARRAYADDR~a_2)~(\I32{.}\CONST~i_2 + n - 1) \\
     ({\ARRAYGET}{\mathsf{\_}}{{{\sx}^?}}~x_2)~(\ARRAYSET~x_1) \\
     (\REFARRAYADDR~a_1)~(\I32{.}\CONST~i_1)~(\REFARRAYADDR~a_2)~(\I32{.}\CONST~i_2)~(\I32{.}\CONST~n - 1)~(\ARRAYCOPY~x_1~x_2) \end{array} & & \\
    \multicolumn{3}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{otherwise, if}~ z{.}\ZTYPES{}[x_2] \approxexpanddt \TARRAY~({\TMUT^?}~{\mathit{zt}}_2) \\
   {\land}~ {{\sx}^?} = {\sx}({\mathit{zt}}_2) \\
   \end{array} \\
   \end{array}
   } \\
   \end{array}
   } \\
   \end{array}

Where:

.. _aux-sx:

.. math::
   \begin{array}[t]{@{}lcl@{}l@{}}
   {\sx}({\consttype}) & = & \epsilon \\
   {\sx}({\packtype}) & = & \S \\
   \end{array}

.. _exec-array.init_data:


:math:`\ARRAYINITDATA~x~y`
..........................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, a value of :ref:`number type <syntax-numtype>` :math:`\I32` is on the top of the stack.

#. Pop the value :math:`(\I32{.}\CONST~n)` from the stack.

#. Assert: Due to validation, a value of :ref:`number type <syntax-numtype>` :math:`\I32` is on the top of the stack.

#. Pop the value :math:`(\I32{.}\CONST~j)` from the stack.

#. Assert: Due to validation, a value of :ref:`number type <syntax-numtype>` :math:`\I32` is on the top of the stack.

#. Pop the value :math:`(\I32{.}\CONST~i)` from the stack.

#. Assert: Due to validation, a value is on the top of the stack.

#. Pop the value :math:`{\val}` from the stack.

#. If :math:`{\val} = \REFNULLADDR`, then:

   a. Trap.

#. Assert: Due to validation, :math:`{\val}` is some :math:`\REFARRAYADDR~{\arrayaddr}`.

#. Let :math:`(\REFARRAYADDR~a)` be the destructuring of :math:`{\val}`.

#. If :math:`a < {|z{.}\ZARRAYS|}` and :math:`i + n > {|z{.}\ZARRAYS{}[a]{.}\AIFIELDS|}`, then:

   a. Trap.

#. If the :ref:`expansion <aux-expand-deftype>` of :math:`z{.}\ZTYPES{}[x]` is some :math:`\TARRAY~{\fieldtype}`, then:

   a. Let :math:`(\TARRAY~{\fieldtype}_0)` be the destructuring of the :ref:`expansion <aux-expand-deftype>` of :math:`z{.}\ZTYPES{}[x]`.

   #. Let :math:`({\TMUT^?}~{\mathit{zt}})` be the destructuring of :math:`{\fieldtype}_0`.

   #. If :math:`j + n \cdot {|{\mathit{zt}}|} / 8 > {|z{.}\ZDATAS{}[y]{.}\DIBYTES|}`, then:

      1) Trap.

   #. If :math:`n = 0`, then:

      1) Do nothing.

   #. Else:

      1) Let :math:`c` be the result for which :math:`{{\bytes}}_{{\mathit{zt}}}(c)` :math:`=` :math:`z{.}\ZDATAS{}[y]{.}\DIBYTES{}[j : {|{\mathit{zt}}|} / 8]`.

      #) Push the value :math:`(\REFARRAYADDR~a)` to the stack.

      #) Push the value :math:`(\I32{.}\CONST~i)` to the stack.

      #) Push the value :math:`{\unpack}({\mathit{zt}}){.}\CONST~{{\unpacknum}}_{{\mathit{zt}}}(c)` to the stack.

      #) Execute the instruction :math:`(\ARRAYSET~x)`.

      #) Push the value :math:`(\REFARRAYADDR~a)` to the stack.

      #) Push the value :math:`(\I32{.}\CONST~i + 1)` to the stack.

      #) Push the value :math:`(\I32{.}\CONST~j + {|{\mathit{zt}}|} / 8)` to the stack.

      #) Push the value :math:`(\I32{.}\CONST~n - 1)` to the stack.

      #) Execute the instruction :math:`(\ARRAYINITDATA~x~y)`.

#. Else if :math:`n = 0`, then:

   a. Do nothing.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; (\REFNULLADDR)~(\I32{.}\CONST~i)~(\I32{.}\CONST~j)~(\I32{.}\CONST~n)~(\ARRAYINITDATA~x~y) & \stepto & \TRAP \\
   & z ; (\REFARRAYADDR~a)~(\I32{.}\CONST~i)~(\I32{.}\CONST~j)~(\I32{.}\CONST~n)~(\ARRAYINITDATA~x~y) & \stepto & \TRAP &  \\
   & \multicolumn{4}{@{}l@{}}{\quad
   \quad \mbox{if}~ i + n > {|z{.}\ZARRAYS{}[a]{.}\AIFIELDS|}
   } \\
   & z ; (\REFARRAYADDR~a)~(\I32{.}\CONST~i)~(\I32{.}\CONST~j)~(\I32{.}\CONST~n)~(\ARRAYINITDATA~x~y) & \stepto & \TRAP &  \\
   & \multicolumn{4}{@{}l@{}}{\quad
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ z{.}\ZTYPES{}[x] \approxexpanddt \TARRAY~({\TMUT^?}~{\mathit{zt}}) \\
   {\land}~ j + n \cdot {|{\mathit{zt}}|} / 8 > {|z{.}\ZDATAS{}[y]{.}\DIBYTES|} \\
   \end{array}
   } \\
   & z ; (\REFARRAYADDR~a)~(\I32{.}\CONST~i)~(\I32{.}\CONST~j)~(\I32{.}\CONST~n)~(\ARRAYINITDATA~x~y) & \stepto & \epsilon &  \\
   & \multicolumn{4}{@{}l@{}}{\quad
   \quad \mbox{otherwise, if}~ n = 0
   } \\
   & z ; (\REFARRAYADDR~a)~(\I32{.}\CONST~i)~(\I32{.}\CONST~j)~(\I32{.}\CONST~n)~(\ARRAYINITDATA~x~y) & \stepto & & \\
   & \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}l@{}l@{}}
   \begin{array}[t]{@{}l@{}} (\REFARRAYADDR~a)~(\I32{.}\CONST~i)~({\unpack}({\mathit{zt}}){.}\CONST~{{\unpacknum}}_{{\mathit{zt}}}(c))~(\ARRAYSET~x) \\
     (\REFARRAYADDR~a)~(\I32{.}\CONST~i + 1)~(\I32{.}\CONST~j + {|{\mathit{zt}}|} / 8)~(\I32{.}\CONST~n - 1)~(\ARRAYINITDATA~x~y) \end{array} & & \\
    \multicolumn{3}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{otherwise, if}~ z{.}\ZTYPES{}[x] \approxexpanddt \TARRAY~({\TMUT^?}~{\mathit{zt}}) \\
   {\land}~ {{\bytes}}_{{\mathit{zt}}}(c) = z{.}\ZDATAS{}[y]{.}\DIBYTES{}[j : {|{\mathit{zt}}|} / 8] \\
   \end{array} \\
   \end{array}
   } \\
   \end{array}
   } \\
   \end{array}


.. _exec-array.init_elem:


:math:`\ARRAYINITELEM~x~y`
..........................


1. Let :math:`z` be the current state.

#. Assert: Due to validation, a value of :ref:`number type <syntax-numtype>` :math:`\I32` is on the top of the stack.

#. Pop the value :math:`(\I32{.}\CONST~n)` from the stack.

#. Assert: Due to validation, a value of :ref:`number type <syntax-numtype>` :math:`\I32` is on the top of the stack.

#. Pop the value :math:`(\I32{.}\CONST~j)` from the stack.

#. Assert: Due to validation, a value of :ref:`number type <syntax-numtype>` :math:`\I32` is on the top of the stack.

#. Pop the value :math:`(\I32{.}\CONST~i)` from the stack.

#. Assert: Due to validation, a value is on the top of the stack.

#. Pop the value :math:`{\val}` from the stack.

#. If :math:`{\val} = \REFNULLADDR`, then:

   a. Trap.

#. Assert: Due to validation, :math:`{\val}` is some :math:`\REFARRAYADDR~{\arrayaddr}`.

#. Let :math:`(\REFARRAYADDR~a)` be the destructuring of :math:`{\val}`.

#. If :math:`a < {|z{.}\ZARRAYS|}` and :math:`i + n > {|z{.}\ZARRAYS{}[a]{.}\AIFIELDS|}`, then:

   a. Trap.

#. If :math:`j + n > {|z{.}\ZELEMS{}[y]{.}\EIREFS|}`, then:

   a. Trap.

#. If :math:`n = 0`, then:

   a. Do nothing.

#. Else if :math:`j < {|z{.}\ZELEMS{}[y]{.}\EIREFS|}`, then:

   a. Let :math:`{\reff}` be the :ref:`reference value <syntax-ref>` :math:`z{.}\ZELEMS{}[y]{.}\EIREFS{}[j]`.

   #. Push the value :math:`(\REFARRAYADDR~a)` to the stack.

   #. Push the value :math:`(\I32{.}\CONST~i)` to the stack.

   #. Push the value :math:`{\reff}` to the stack.

   #. Execute the instruction :math:`(\ARRAYSET~x)`.

   #. Push the value :math:`(\REFARRAYADDR~a)` to the stack.

   #. Push the value :math:`(\I32{.}\CONST~i + 1)` to the stack.

   #. Push the value :math:`(\I32{.}\CONST~j + 1)` to the stack.

   #. Push the value :math:`(\I32{.}\CONST~n - 1)` to the stack.

   #. Execute the instruction :math:`(\ARRAYINITELEM~x~y)`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; (\REFNULLADDR)~(\I32{.}\CONST~i)~(\I32{.}\CONST~j)~(\I32{.}\CONST~n)~(\ARRAYINITELEM~x~y) & \stepto & \TRAP \\
   & z ; (\REFARRAYADDR~a)~(\I32{.}\CONST~i)~(\I32{.}\CONST~j)~(\I32{.}\CONST~n)~(\ARRAYINITELEM~x~y) & \stepto & \TRAP &  \\
   & \multicolumn{4}{@{}l@{}}{\quad
   \quad \mbox{if}~ i + n > {|z{.}\ZARRAYS{}[a]{.}\AIFIELDS|}
   } \\
   & z ; (\REFARRAYADDR~a)~(\I32{.}\CONST~i)~(\I32{.}\CONST~j)~(\I32{.}\CONST~n)~(\ARRAYINITELEM~x~y) & \stepto & \TRAP &  \\
   & \multicolumn{4}{@{}l@{}}{\quad
   \quad \mbox{if}~ j + n > {|z{.}\ZELEMS{}[y]{.}\EIREFS|}
   } \\
   & z ; (\REFARRAYADDR~a)~(\I32{.}\CONST~i)~(\I32{.}\CONST~j)~(\I32{.}\CONST~n)~(\ARRAYINITELEM~x~y) & \stepto & \epsilon &  \\
   & \multicolumn{4}{@{}l@{}}{\quad
   \quad \mbox{otherwise, if}~ n = 0
   } \\
   & z ; (\REFARRAYADDR~a)~(\I32{.}\CONST~i)~(\I32{.}\CONST~j)~(\I32{.}\CONST~n)~(\ARRAYINITELEM~x~y) & \stepto & & \\
   & \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}l@{}l@{}}
   \begin{array}[t]{@{}l@{}} (\REFARRAYADDR~a)~(\I32{.}\CONST~i)~{\reff}~(\ARRAYSET~x) \\
     (\REFARRAYADDR~a)~(\I32{.}\CONST~i + 1)~(\I32{.}\CONST~j + 1)~(\I32{.}\CONST~n - 1)~(\ARRAYINITELEM~x~y) \end{array} & & \\
    \multicolumn{3}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \quad \mbox{otherwise, if}~ {\reff} = z{.}\ZELEMS{}[y]{.}\EIREFS{}[j] \\
   \end{array}
   } \\
   \end{array}
   } \\
   \end{array}


.. _exec-any.convert_extern:


:math:`\ANYCONVERTEXTERN`
.........................


1. Assert: Due to validation, a value is on the top of the stack.

#. Pop the value :math:`{\val}` from the stack.

#. If :math:`{\val} = \REFNULLADDR`, then:

   a. Push the value :math:`\REFNULLADDR` to the stack.

#. If :math:`{\val}` is some :math:`\REFEXTERN~{\reff}`, then:

   a. Let :math:`(\REFEXTERN~{\reff})` be the destructuring of :math:`{\val}`.

   #. Push the value :math:`{\reff}` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & (\REFNULLADDR)~\ANYCONVERTEXTERN & \stepto & \REFNULLADDR \\
   & (\REFEXTERN~{\reff})~\ANYCONVERTEXTERN & \stepto & {\reff} \\
   \end{array}


.. _exec-extern.convert_any:


:math:`\EXTERNCONVERTANY`
.........................


1. Assert: Due to validation, a :ref:`reference value <syntax-ref>` is on the top of the stack.

#. Pop the value :math:`{\reff}` from the stack.

#. If :math:`{\reff} = \REFNULLADDR`, then:

   a. Push the value :math:`\REFNULLADDR` to the stack.

#. Else:

   a. Push the value :math:`(\REFEXTERN~{\reff})` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & {\reff}~\EXTERNCONVERTANY & \stepto & \REFNULLADDR & \quad \mbox{if}~ {\reff} = \REFNULLADDR \\
   & {\reff}~\EXTERNCONVERTANY & \stepto & (\REFEXTERN~{\reff}) & \quad \mbox{otherwise} \\
   \end{array}


.. index:: numeric instruction, determinism, non-determinism, trap, NaN, value, value type
   pair: execution; instruction
   single: abstract syntax; instruction
.. _exec-instr-numeric:

Numeric Instructions
~~~~~~~~~~~~~~~~~~~~

Numeric instructions are defined in terms of the generic :ref:`numeric operators <exec-numeric>`.
The mapping of numeric instructions to their underlying operators is expressed by the following definition:

.. math::
   \begin{array}{lll@{\qquad}l}
   \X{op}_{\IN}(i_1,\dots,i_k) &=& \xref{Step_pure/numerics}{int-ops}{\F{i}\X{op}}_N(i_1,\dots,i_k) \\
   \X{op}_{\FN}(z_1,\dots,z_k) &=& \xref{Step_pure/numerics}{float-ops}{\F{f}\X{op}}_N(z_1,\dots,z_k) \\
   \end{array}

And for :ref:`conversion operators <exec-cvtop>`:

.. math::
   \begin{array}{lll@{\qquad}l}
   \cvtop^{\sx^?}_{t_1,t_2}(c) &=& \xref{Step_pure/numerics}{convert-ops}{\X{cvtop}}^{\sx^?}_{|t_1|,|t_2|}(c) \\
   \end{array}

Where the underlying operators are partial, the corresponding instruction will :ref:`trap <trap>` when the result is not defined.
Where the underlying operators are non-deterministic, because they may return one of multiple possible :ref:`NaN <syntax-nan>` values, so are the corresponding instructions.

.. note::
   For example, the result of instruction :math:`\I32.\ADD` applied to operands :math:`i_1, i_2`
   invokes :math:`\ADD_{\I32}(i_1, i_2)`,
   which maps to the generic :math:`\iadd_{32}(i_1, i_2)` via the above definition.
   Similarly, :math:`\I64.\TRUNC\K{\_}\F32\K{\_s}` applied to :math:`z`
   invokes :math:`\TRUNC^{\K{s}}_{\F32,\I64}(z)`,
   which maps to the generic :math:`\truncs_{32,64}(z)`.


.. _exec-const:

:math:`\X{nt}\K{.}\CONST~c`
...........................

1. Push the value :math:`({\mathit{nt}}{.}\CONST~c)` to the stack.

.. note::
   No formal reduction rule is required for this instruction, since :math:`\mathsf{const}` instructions already are :ref:`values <syntax-val>`.


.. _exec-unop:


:math:`{\mathit{nt}} {.} {\unop}`
.................................


1. Assert: Due to :ref:`validation <valid-unop>`, a value of :ref:`number type <syntax-numtype>` :math:`{\mathit{nt}}` is on the top of the stack.

#. Pop the value :math:`({\numtype}_0{.}\CONST~c_1)` from the stack.

#. If :math:`{{\unop}}{{}_{{\mathit{nt}}}(c_1)}` is empty, then:

   a. Trap.

#. Let :math:`c` be an element of :math:`{{\unop}}{{}_{{\mathit{nt}}}(c_1)}`.

#. Push the value :math:`({\mathit{nt}}{.}\CONST~c)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & ({\mathit{nt}}{.}\CONST~c_1)~({\mathit{nt}} {.} {\unop}) & \stepto & ({\mathit{nt}}{.}\CONST~c) & \quad \mbox{if}~ c \in {{\unop}}{{}_{{\mathit{nt}}}(c_1)} \\
   & ({\mathit{nt}}{.}\CONST~c_1)~({\mathit{nt}} {.} {\unop}) & \stepto & \TRAP & \quad \mbox{if}~ {{\unop}}{{}_{{\mathit{nt}}}(c_1)} = \epsilon \\
   \end{array}


.. _exec-binop:


:math:`{\mathit{nt}} {.} {\binop}`
..................................


1. Assert: Due to :ref:`validation <valid-binop>`, a value of :ref:`number type <syntax-numtype>` :math:`{\mathit{nt}}` is on the top of the stack.

#. Pop the value :math:`({\numtype}_0{.}\CONST~c_2)` from the stack.

#. Assert: Due to :ref:`validation <valid-binop>`, a :ref:`number value <syntax-num>` is on the top of the stack.

#. Pop the value :math:`({\numtype}_0{.}\CONST~c_1)` from the stack.

#. If :math:`{{\binop}}{{}_{{\mathit{nt}}}(c_1, c_2)}` is empty, then:

   a. Trap.

#. Let :math:`c` be an element of :math:`{{\binop}}{{}_{{\mathit{nt}}}(c_1, c_2)}`.

#. Push the value :math:`({\mathit{nt}}{.}\CONST~c)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & ({\mathit{nt}}{.}\CONST~c_1)~({\mathit{nt}}{.}\CONST~c_2)~({\mathit{nt}} {.} {\binop}) & \stepto & ({\mathit{nt}}{.}\CONST~c) & \quad \mbox{if}~ c \in {{\binop}}{{}_{{\mathit{nt}}}(c_1, c_2)} \\
   & ({\mathit{nt}}{.}\CONST~c_1)~({\mathit{nt}}{.}\CONST~c_2)~({\mathit{nt}} {.} {\binop}) & \stepto & \TRAP & \quad \mbox{if}~ {{\binop}}{{}_{{\mathit{nt}}}(c_1, c_2)} = \epsilon \\
   \end{array}


.. _exec-testop:


:math:`{\mathit{nt}} {.} {\testop}`
...................................


1. Assert: Due to :ref:`validation <valid-testop>`, a value of :ref:`number type <syntax-numtype>` :math:`{\mathit{nt}}` is on the top of the stack.

#. Pop the value :math:`({\numtype}_0{.}\CONST~c_1)` from the stack.

#. Let :math:`c` be :math:`{{\testop}}{{}_{{\mathit{nt}}}(c_1)}`.

#. Push the value :math:`(\I32{.}\CONST~c)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & ({\mathit{nt}}{.}\CONST~c_1)~({\mathit{nt}} {.} {\testop}) & \stepto & (\I32{.}\CONST~c) & \quad \mbox{if}~ c = {{\testop}}{{}_{{\mathit{nt}}}(c_1)} \\
   \end{array}


.. _exec-relop:


:math:`{\mathit{nt}} {.} {\relop}`
..................................


1. Assert: Due to :ref:`validation <valid-relop>`, a value of :ref:`number type <syntax-numtype>` :math:`{\mathit{nt}}` is on the top of the stack.

#. Pop the value :math:`({\numtype}_0{.}\CONST~c_2)` from the stack.

#. Assert: Due to :ref:`validation <valid-relop>`, a :ref:`number value <syntax-num>` is on the top of the stack.

#. Pop the value :math:`({\numtype}_0{.}\CONST~c_1)` from the stack.

#. Let :math:`c` be :math:`{{\relop}}{{}_{{\mathit{nt}}}(c_1, c_2)}`.

#. Push the value :math:`(\I32{.}\CONST~c)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & ({\mathit{nt}}{.}\CONST~c_1)~({\mathit{nt}}{.}\CONST~c_2)~({\mathit{nt}} {.} {\relop}) & \stepto & (\I32{.}\CONST~c) & \quad \mbox{if}~ c = {{\relop}}{{}_{{\mathit{nt}}}(c_1, c_2)} \\
   \end{array}


.. _exec-cvtop:


:math:`{\mathit{nt}}_2 {.} {{\cvtop}}{\mathsf{\_}}{{\mathit{nt}}_1}`
....................................................................


1. Assert: Due to :ref:`validation <valid-cvtop>`, a value of :ref:`number type <syntax-numtype>` :math:`{\mathit{nt}}_1` is on the top of the stack.

#. Pop the value :math:`({\numtype}_0{.}\CONST~c_1)` from the stack.

#. If :math:`{{\cvtop}}{{}_{{\mathit{nt}}_1, {\mathit{nt}}_2}(c_1)}` is empty, then:

   a. Trap.

#. Let :math:`c` be an element of :math:`{{\cvtop}}{{}_{{\mathit{nt}}_1, {\mathit{nt}}_2}(c_1)}`.

#. Push the value :math:`({\mathit{nt}}_2{.}\CONST~c)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & ({\mathit{nt}}_1{.}\CONST~c_1)~({\mathit{nt}}_2 {.} {{\cvtop}}{\mathsf{\_}}{{\mathit{nt}}_1}) & \stepto & ({\mathit{nt}}_2{.}\CONST~c) & \quad \mbox{if}~ c \in {{\cvtop}}{{}_{{\mathit{nt}}_1, {\mathit{nt}}_2}(c_1)} \\
   & ({\mathit{nt}}_1{.}\CONST~c_1)~({\mathit{nt}}_2 {.} {{\cvtop}}{\mathsf{\_}}{{\mathit{nt}}_1}) & \stepto & \TRAP & \quad \mbox{if}~ {{\cvtop}}{{}_{{\mathit{nt}}_1, {\mathit{nt}}_2}(c_1)} = \epsilon \\
   \end{array}


.. index:: vector instruction
   pair: execution; instruction
   single: abstract syntax; instruction
.. _exec-instr-vec:

Vector Instructions
~~~~~~~~~~~~~~~~~~~

Vector instructions that operate bitwise are handled as integer operations of respective bit width.

.. math::
   \begin{array}{lll@{\qquad}l}
   \X{op}_{\VN}(i_1,\dots,i_k) &=& \xref{Step_pure/numerics}{int-ops}{\F{i}\X{op}}_N(i_1,\dots,i_k) \\
   \end{array}

Most other vector instructions are defined in terms of :ref:`numeric operators <exec-numeric>` that are applied lane-wise according to the given :ref:`shape <syntax-shape>`.

.. math::
   \begin{array}{llll}
   \X{op}_{t\K{x}N}(n_1,\dots,n_k) &=&
     \lanes^{-1}_{t\K{x}N}(\xref{Step_pure/instructions}{exec-instr-numeric}{\X{op}}_t(i_1,\dots,i_k)^\ast) & \qquad(\iff i_1^\ast = \lanes_{t\K{x}N}(n_1) \land \dots \land i_k^\ast = \lanes_{t\K{x}N}(n_k) \\
   \end{array}

.. note::
   For example, the result of instruction :math:`\K{i32x4}.\ADD` applied to operands :math:`v_1, v_2`
   invokes :math:`\ADD_{\K{i32x4}}(v_1, v_2)`, which maps to
   :math:`\lanes^{-1}_{\K{i32x4}}(\ADD_{\I32}(i_1, i_2)^\ast)`,
   where :math:`i_1^\ast` and :math:`i_2^\ast` are sequences resulting from invoking
   :math:`\lanes_{\K{i32x4}}(v_1)` and :math:`\lanes_{\K{i32x4}}(v_2)`
   respectively.

For non-deterministic operators this definition is generalized to sets:

.. math::
   \begin{array}{lll}
   \X{op}_{t\K{x}N}(n_1,\dots,n_k) &=&
     \{ \lanes^{-1}_{t\K{x}N}(i^\ast) ~|~ i^\ast \in {\Large\times}(\xref{Step_pure/instructions}{exec-instr-numeric}{\X{op}}_t(i_1,\dots,i_k)^\ast) \land i_1^\ast = \lanes_{t\K{x}N}(n_1) \land \dots \land i_k^\ast = \lanes_{t\K{x}N}(n_k) \} \\
   \end{array}

where :math:`{\Large\times} \{x^\ast\}^N` transforms a sequence of :math:`N` sets of values into a set of sequences of :math:`N` values by computing the set product:

.. math::
   \begin{array}{lll}
   {\Large\times} (S_1 \dots S_N) &=& \{ x_1 \dots x_N ~|~ x_1 \in S_1 \land \dots \land x_N \in S_N \}
   \end{array}

The remaining vector operators use :ref:`individual definitions <op-vec>`.


.. _exec-vconst:

:math:`\V128\K{.}\VCONST~c`
...........................

1. Push the value :math:`(\V128{.}\CONST~c)` to the stack.

.. note::
   No formal reduction rule is required for this instruction, since :math:`\mathsf{const}` instructions are already :ref:`values <syntax-val>`.


.. _exec-vvunop:


:math:`\V128 {.} {\vvunop}`
...........................


1. Assert: Due to :ref:`validation <valid-vvunop>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_1)` from the stack.

#. Assert: Due to :ref:`validation <valid-vvunop>`, :math:`{|{{\vvunop}}{{}_{\V128}(c_1)}|} > 0`.

#. Let :math:`c` be an element of :math:`{{\vvunop}}{{}_{\V128}(c_1)}`.

#. Push the value :math:`(\V128{.}\CONST~c)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & (\V128{.}\CONST~c_1)~(\V128 {.} {\vvunop}) & \stepto & (\V128{.}\CONST~c) & \quad \mbox{if}~ c \in {{\vvunop}}{{}_{\V128}(c_1)} \\
   \end{array}


.. _exec-vvbinop:


:math:`\V128 {.} {\vvbinop}`
............................


1. Assert: Due to :ref:`validation <valid-vvbinop>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_2)` from the stack.

#. Assert: Due to :ref:`validation <valid-vvbinop>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_1)` from the stack.

#. Assert: Due to :ref:`validation <valid-vvbinop>`, :math:`{|{{\vvbinop}}{{}_{\V128}(c_1, c_2)}|} > 0`.

#. Let :math:`c` be an element of :math:`{{\vvbinop}}{{}_{\V128}(c_1, c_2)}`.

#. Push the value :math:`(\V128{.}\CONST~c)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & (\V128{.}\CONST~c_1)~(\V128{.}\CONST~c_2)~(\V128 {.} {\vvbinop}) & \stepto & (\V128{.}\CONST~c) & \quad \mbox{if}~ c \in {{\vvbinop}}{{}_{\V128}(c_1, c_2)} \\
   \end{array}


.. _exec-vvternop:


:math:`\V128 {.} {\vvternop}`
.............................


1. Assert: Due to :ref:`validation <valid-vvternop>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_3)` from the stack.

#. Assert: Due to :ref:`validation <valid-vvternop>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_2)` from the stack.

#. Assert: Due to :ref:`validation <valid-vvternop>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_1)` from the stack.

#. Assert: Due to :ref:`validation <valid-vvternop>`, :math:`{|{{\vvternop}}{{}_{\V128}(c_1, c_2, c_3)}|} > 0`.

#. Let :math:`c` be an element of :math:`{{\vvternop}}{{}_{\V128}(c_1, c_2, c_3)}`.

#. Push the value :math:`(\V128{.}\CONST~c)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & (\V128{.}\CONST~c_1)~(\V128{.}\CONST~c_2)~(\V128{.}\CONST~c_3)~(\V128 {.} {\vvternop}) & \stepto & (\V128{.}\CONST~c) &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad \mbox{if}~ c \in {{\vvternop}}{{}_{\V128}(c_1, c_2, c_3)}
   } \\
   \end{array}


.. _exec-vvtestop:


:math:`\V128 {.} \VANYTRUE`
...........................


1. Assert: Due to :ref:`validation <valid-vvtestop>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_1)` from the stack.

#. Let :math:`c` be :math:`{{\inez}}_{{|\V128|}}(c_1)`.

#. Push the value :math:`(\I32{.}\CONST~c)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & (\V128{.}\CONST~c_1)~(\V128 {.} \VANYTRUE) & \stepto & (\I32{.}\CONST~c) & \quad \mbox{if}~ c = {{\inez}}_{{|\V128|}}(c_1) \\
   \end{array}


.. _exec-vunop:


:math:`{\mathit{sh}} {.} {\vunop}`
..................................


1. Assert: Due to :ref:`validation <valid-vunop>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_1)` from the stack.

#. If :math:`{{\vunop}}{{}_{{\mathit{sh}}}(c_1)}` is empty, then:

   a. Trap.

#. Let :math:`c` be an element of :math:`{{\vunop}}{{}_{{\mathit{sh}}}(c_1)}`.

#. Push the value :math:`(\V128{.}\CONST~c)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & (\V128{.}\CONST~c_1)~({\mathit{sh}} {.} {\vunop}) & \stepto & (\V128{.}\CONST~c) & \quad \mbox{if}~ c \in {{\vunop}}{{}_{{\mathit{sh}}}(c_1)} \\
   & (\V128{.}\CONST~c_1)~({\mathit{sh}} {.} {\vunop}) & \stepto & \TRAP & \quad \mbox{if}~ {{\vunop}}{{}_{{\mathit{sh}}}(c_1)} = \epsilon \\
   \end{array}


.. _exec-vbinop:


:math:`{\mathit{sh}} {.} {\vbinop}`
...................................


1. Assert: Due to :ref:`validation <valid-vbinop>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_2)` from the stack.

#. Assert: Due to :ref:`validation <valid-vbinop>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_1)` from the stack.

#. If :math:`{{\vbinop}}{{}_{{\mathit{sh}}}(c_1, c_2)}` is empty, then:

   a. Trap.

#. Let :math:`c` be an element of :math:`{{\vbinop}}{{}_{{\mathit{sh}}}(c_1, c_2)}`.

#. Push the value :math:`(\V128{.}\CONST~c)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & (\V128{.}\CONST~c_1)~(\V128{.}\CONST~c_2)~({\mathit{sh}} {.} {\vbinop}) & \stepto & (\V128{.}\CONST~c) & \quad \mbox{if}~ c \in {{\vbinop}}{{}_{{\mathit{sh}}}(c_1, c_2)} \\
   & (\V128{.}\CONST~c_1)~(\V128{.}\CONST~c_2)~({\mathit{sh}} {.} {\vbinop}) & \stepto & \TRAP & \quad \mbox{if}~ {{\vbinop}}{{}_{{\mathit{sh}}}(c_1, c_2)} = \epsilon \\
   \end{array}


.. _exec-vternop:


:math:`{\mathit{sh}} {.} {\vternop}`
....................................


1. Assert: Due to :ref:`validation <valid-vternop>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_3)` from the stack.

#. Assert: Due to :ref:`validation <valid-vternop>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_2)` from the stack.

#. Assert: Due to :ref:`validation <valid-vternop>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_1)` from the stack.

#. If :math:`{{\vternop}}{{}_{{\mathit{sh}}}(c_1, c_2, c_3)}` is empty, then:

   a. Trap.

#. Let :math:`c` be an element of :math:`{{\vternop}}{{}_{{\mathit{sh}}}(c_1, c_2, c_3)}`.

#. Push the value :math:`(\V128{.}\CONST~c)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & (\V128{.}\CONST~c_1)~(\V128{.}\CONST~c_2)~(\V128{.}\CONST~c_3)~({\mathit{sh}} {.} {\vternop}) & \stepto & (\V128{.}\CONST~c) & \quad \mbox{if}~ c \in {{\vternop}}{{}_{{\mathit{sh}}}(c_1, c_2, c_3)} \\
   & (\V128{.}\CONST~c_1)~(\V128{.}\CONST~c_2)~(\V128{.}\CONST~c_3)~({\mathit{sh}} {.} {\vternop}) & \stepto & \TRAP & \quad \mbox{if}~ {{\vternop}}{{}_{{\mathit{sh}}}(c_1, c_2, c_3)} = \epsilon \\
   \end{array}


.. _exec-vtestop:


:math:`{{\ntI}{{\ntN}}}{\Xshape}{M} {.} \VALLTRUE`
..................................................


1. Assert: Due to :ref:`validation <valid-vtestop>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_1)` from the stack.

#. Let :math:`{i^\ast}` be :math:`{{\lanes}}_{{{\ntI}{{\ntN}}}{\Xshape}{M}}(c_1)`.

#. Let :math:`c` be :math:`{\Pi}\, {{{\inez}}_{N}(i)^\ast}`.

#. Push the value :math:`(\I32{.}\CONST~c)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & (\V128{.}\CONST~c_1)~({{\ntI}{{\ntN}}}{\Xshape}{M} {.} \VALLTRUE) & \stepto & (\I32{.}\CONST~c) & \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ {i^\ast} = {{\lanes}}_{{{\ntI}{{\ntN}}}{\Xshape}{M}}(c_1) \\
   {\land}~ c = {\Pi}\, ({{{\inez}}_{N}(i)^\ast}) \\
   \end{array} \\
   \end{array}


.. _exec-vrelop:


:math:`{\mathit{sh}} {.} {\vrelop}`
...................................


1. Assert: Due to :ref:`validation <valid-vrelop>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_2)` from the stack.

#. Assert: Due to :ref:`validation <valid-vrelop>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_1)` from the stack.

#. Let :math:`c` be :math:`{{\vrelop}}{{}_{{\mathit{sh}}}(c_1, c_2)}`.

#. Push the value :math:`(\V128{.}\CONST~c)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & (\V128{.}\CONST~c_1)~(\V128{.}\CONST~c_2)~({\mathit{sh}} {.} {\vrelop}) & \stepto & (\V128{.}\CONST~c) & \quad \mbox{if}~ c = {{\vrelop}}{{}_{{\mathit{sh}}}(c_1, c_2)} \\
   \end{array}


.. _exec-vshiftop:


:math:`{\mathit{sh}} {.} {\vshiftop}`
.....................................


1. Assert: Due to :ref:`validation <valid-vshiftop>`, a value of :ref:`number type <syntax-numtype>` :math:`\I32` is on the top of the stack.

#. Pop the value :math:`(\I32{.}\CONST~i)` from the stack.

#. Assert: Due to :ref:`validation <valid-vshiftop>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_1)` from the stack.

#. Let :math:`c` be :math:`{{\vshiftop}}{{}_{{\mathit{sh}}}}{(c_1, i)}`.

#. Push the value :math:`(\V128{.}\CONST~c)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & (\V128{.}\CONST~c_1)~(\I32{.}\CONST~i)~({\mathit{sh}} {.} {\vshiftop}) & \stepto & (\V128{.}\CONST~c) & \quad \mbox{if}~ c = {{\vshiftop}}{{}_{{\mathit{sh}}}}{(c_1, i)} \\
   \end{array}


.. _exec-vbitmask:


:math:`{\mathit{sh}}{.}\VBITMASK`
.................................


1. Assert: Due to :ref:`validation <valid-vbitmask>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_1)` from the stack.

#. Let :math:`c` be :math:`{\VBITMASK}{{}_{{\mathit{sh}}}(c_1)}`.

#. Push the value :math:`(\I32{.}\CONST~c)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & (\V128{.}\CONST~c_1)~({\mathit{sh}}{.}\VBITMASK) & \stepto & (\I32{.}\CONST~c) & \quad \mbox{if}~ c = {\VBITMASK}{{}_{{\mathit{sh}}}(c_1)} \\
   \end{array}


.. _exec-vswizzlop:


:math:`{\mathit{sh}} {.} {\mathit{swizzlop}}`
.............................................


1. Assert: Due to :ref:`validation <valid-vswizzlop>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_2)` from the stack.

#. Assert: Due to :ref:`validation <valid-vswizzlop>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_1)` from the stack.

#. Let :math:`c` be :math:`{{\mathit{swizzlop}}}{{}_{{\mathit{sh}}}(c_1, c_2)}`.

#. Push the value :math:`(\V128{.}\CONST~c)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & (\V128{.}\CONST~c_1)~(\V128{.}\CONST~c_2)~({\mathit{sh}} {.} {\mathit{swizzlop}}) & \stepto & (\V128{.}\CONST~c) & \quad \mbox{if}~ c = {{\mathit{swizzlop}}}{{}_{{\mathit{sh}}}(c_1, c_2)} \\
   \end{array}


.. _exec-vshuffle:


:math:`{\mathit{sh}}{.}\VSHUFFLE~{i^\ast}`
..........................................


1. Assert: Due to :ref:`validation <valid-vshuffle>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_2)` from the stack.

#. Assert: Due to :ref:`validation <valid-vshuffle>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_1)` from the stack.

#. Let :math:`c` be :math:`{\VSHUFFLE}{{}_{{\mathit{sh}}}({i^\ast}, c_1, c_2)}`.

#. Push the value :math:`(\V128{.}\CONST~c)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & (\V128{.}\CONST~c_1)~(\V128{.}\CONST~c_2)~({\mathit{sh}}{.}\VSHUFFLE~{i^\ast}) & \stepto & (\V128{.}\CONST~c) & \quad \mbox{if}~ c = {\VSHUFFLE}{{}_{{\mathit{sh}}}({i^\ast}, c_1, c_2)} \\
   \end{array}


.. _exec-vsplat:


:math:`{{\ntI}{{\ntN}}}{\Xshape}{M}{.}\VSPLAT`
..............................................


1. Assert: Due to :ref:`validation <valid-vsplat>`, a value is on the top of the stack.

#. Pop the value :math:`({\numtype}_0{.}\CONST~c_1)` from the stack.

#. Assert: Due to :ref:`validation <valid-vsplat>`, :math:`{\numtype}_0 = {\unpack}({\ntI}{{\ntN}})`.

#. Let :math:`c` be :math:`{{{{\lanes}}_{{{\ntI}{{\ntN}}}{\Xshape}{M}}^{{-1}}}}{({{{\packnum}}_{{\ntI}{{\ntN}}}(c_1)^{M}})}`.

#. Push the value :math:`(\V128{.}\CONST~c)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & ({\unpack}({\ntI}{{\ntN}}){.}\CONST~c_1)~({{\ntI}{{\ntN}}}{\Xshape}{M}{.}\VSPLAT) & \stepto & (\V128{.}\CONST~c) & \quad \mbox{if}~ c = {{{{\lanes}}_{{{\ntI}{{\ntN}}}{\Xshape}{M}}^{{-1}}}}{({{{\packnum}}_{{\ntI}{{\ntN}}}(c_1)^{M}})} \\
   \end{array}


.. _exec-vextract_lane:


:math:`{{{\lanetype}}{\Xshape}{M}{.}\VEXTRACTLANE}{\mathsf{\_}}{{{\sx'}^?}}~i`
..............................................................................


1. Assert: Due to :ref:`validation <valid-vextract_lane>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_1)` from the stack.

#. If :math:`{{\sx'}^?}` is not defined, then:

   a. Assert: Due to :ref:`validation <valid-vextract_lane>`, :math:`{\lanetype}` is number type.

   #. Assert: Due to :ref:`validation <valid-vextract_lane>`, :math:`i < {|{{\lanes}}_{{{\lanetype}}{\Xshape}{M}}(c_1)|}`.

   #. Let :math:`c_2` be :math:`{{\lanes}}_{{{\lanetype}}{\Xshape}{M}}(c_1){}[i]`.

   #. Push the value :math:`({\lanetype}{.}\CONST~c_2)` to the stack.

#. Else:

   a. Assert: Due to :ref:`validation <valid-vextract_lane>`, :math:`{\lanetype}` is packed type.

   #. Let :math:`{\sx}` be :math:`{{\sx'}^?}`.

   #. Assert: Due to :ref:`validation <valid-vextract_lane>`, :math:`i < {|{{\lanes}}_{{{\lanetype}}{\Xshape}{M}}(c_1)|}`.

   #. Let :math:`c_2` be :math:`{{{{\extend}}_{{|{\lanetype}|}, 32}^{{\sx}}}}{({{\lanes}}_{{{\lanetype}}{\Xshape}{M}}(c_1){}[i])}`.

   #. Push the value :math:`(\I32{.}\CONST~c_2)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & (\V128{.}\CONST~c_1)~({{\mathit{nt}}}{\Xshape}{M}{.}\VEXTRACTLANE~i) & \stepto & ({\mathit{nt}}{.}\CONST~c_2) & \quad \mbox{if}~ c_2 = {{\lanes}}_{{{\mathit{nt}}}{\Xshape}{M}}(c_1){}[i] \\
   & (\V128{.}\CONST~c_1)~({{{\mathit{pt}}}{\Xshape}{M}{.}\VEXTRACTLANE}{\mathsf{\_}}{{\sx}}~i) & \stepto & (\I32{.}\CONST~c_2) & \quad \mbox{if}~ c_2 = {{{{\extend}}_{{|{\mathit{pt}}|}, 32}^{{\sx}}}}{({{\lanes}}_{{{\mathit{pt}}}{\Xshape}{M}}(c_1){}[i])} \\
   \end{array}


.. _exec-vreplace_lane:


:math:`{{\ntI}{{\ntN}}}{\Xshape}{M}{.}\VREPLACELANE~i`
......................................................


1. Assert: Due to :ref:`validation <valid-vreplace_lane>`, a value is on the top of the stack.

#. Pop the value :math:`({\numtype}_0{.}\CONST~c_2)` from the stack.

#. Assert: Due to :ref:`validation <valid-vreplace_lane>`, :math:`{\numtype}_0 = {\unpack}({\ntI}{{\ntN}})`.

#. Assert: Due to :ref:`validation <valid-vreplace_lane>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_1)` from the stack.

#. Let :math:`c` be :math:`{{{{\lanes}}_{{{\ntI}{{\ntN}}}{\Xshape}{M}}^{{-1}}}}{({{\lanes}}_{{{\ntI}{{\ntN}}}{\Xshape}{M}}(c_1){}[{}[i] = {{\packnum}}_{{\ntI}{{\ntN}}}(c_2)])}`.

#. Push the value :math:`(\V128{.}\CONST~c)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & (\V128{.}\CONST~c_1)~({\unpack}({\ntI}{{\ntN}}){.}\CONST~c_2)~({{\ntI}{{\ntN}}}{\Xshape}{M}{.}\VREPLACELANE~i) & \stepto & (\V128{.}\CONST~c) &  \\
   & \multicolumn{4}{@{}l@{}}{\quad
   \quad \mbox{if}~ c = {{{{\lanes}}_{{{\ntI}{{\ntN}}}{\Xshape}{M}}^{{-1}}}}{({{\lanes}}_{{{\ntI}{{\ntN}}}{\Xshape}{M}}(c_1){}[{}[i] = {{\packnum}}_{{\ntI}{{\ntN}}}(c_2)])}
   } \\
   \end{array}


.. _exec-vextunop:


:math:`{\mathit{sh}}_2 {.} {{\vextunop}}{\mathsf{\_}}{{\mathit{sh}}_1}`
.......................................................................


1. Assert: Due to :ref:`validation <valid-vextunop>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_1)` from the stack.

#. Let :math:`c` be :math:`{{\vextunop}}{{}_{{\mathit{sh}}_1, {\mathit{sh}}_2}(c_1)}`.

#. Push the value :math:`(\V128{.}\CONST~c)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & (\V128{.}\CONST~c_1)~({\mathit{sh}}_2 {.} {{\vextunop}}{\mathsf{\_}}{{\mathit{sh}}_1}) & \stepto & (\V128{.}\CONST~c) & \quad \mbox{if}~ {{\vextunop}}{{}_{{\mathit{sh}}_1, {\mathit{sh}}_2}(c_1)} = c \\
   \end{array}


.. _exec-vextbinop:


:math:`{\mathit{sh}}_2 {.} {{\vextbinop}}{\mathsf{\_}}{{\mathit{sh}}_1}`
........................................................................


1. Assert: Due to :ref:`validation <valid-vextbinop>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_2)` from the stack.

#. Assert: Due to :ref:`validation <valid-vextbinop>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_1)` from the stack.

#. Let :math:`c` be :math:`{{\vextbinop}}{{}_{{\mathit{sh}}_1, {\mathit{sh}}_2}(c_1, c_2)}`.

#. Push the value :math:`(\V128{.}\CONST~c)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & (\V128{.}\CONST~c_1)~(\V128{.}\CONST~c_2)~({\mathit{sh}}_2 {.} {{\vextbinop}}{\mathsf{\_}}{{\mathit{sh}}_1}) & \stepto & (\V128{.}\CONST~c) & \quad \mbox{if}~ {{\vextbinop}}{{}_{{\mathit{sh}}_1, {\mathit{sh}}_2}(c_1, c_2)} = c \\
   \end{array}


.. _exec-vextternop:


:math:`{\mathit{sh}}_2 {.} {{\vextternop}}{\mathsf{\_}}{{\mathit{sh}}_1}`
.........................................................................


1. Assert: Due to :ref:`validation <valid-vextternop>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_3)` from the stack.

#. Assert: Due to :ref:`validation <valid-vextternop>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_2)` from the stack.

#. Assert: Due to :ref:`validation <valid-vextternop>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_1)` from the stack.

#. Let :math:`c` be :math:`{{\vextternop}}{{}_{{\mathit{sh}}_1, {\mathit{sh}}_2}(c_1, c_2, c_3)}`.

#. Push the value :math:`(\V128{.}\CONST~c)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & (\V128{.}\CONST~c_1)~(\V128{.}\CONST~c_2)~(\V128{.}\CONST~c_3)~({\mathit{sh}}_2 {.} {{\vextternop}}{\mathsf{\_}}{{\mathit{sh}}_1}) & \stepto & (\V128{.}\CONST~c) &  \\
   &&& \multicolumn{2}{@{}l@{}}{\quad
   \quad \mbox{if}~ {{\vextternop}}{{}_{{\mathit{sh}}_1, {\mathit{sh}}_2}(c_1, c_2, c_3)} = c
   } \\
   \end{array}


.. _exec-vnarrow:


:math:`{{\mathit{sh}}_2{.}\VNARROW}{\mathsf{\_}}{{\mathit{sh}}_1}{\mathsf{\_}}{{\sx}}`
......................................................................................


1. Assert: Due to :ref:`validation <valid-vnarrow>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_2)` from the stack.

#. Assert: Due to :ref:`validation <valid-vnarrow>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_1)` from the stack.

#. Let :math:`c` be :math:`{\VNARROW}{{{}_{{\mathit{sh}}_1, {\mathit{sh}}_2}^{{\sx}}}}{(c_1, c_2)}`.

#. Push the value :math:`(\V128{.}\CONST~c)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & (\V128{.}\CONST~c_1)~(\V128{.}\CONST~c_2)~({{\mathit{sh}}_2{.}\VNARROW}{\mathsf{\_}}{{\mathit{sh}}_1}{\mathsf{\_}}{{\sx}}) & \stepto & (\V128{.}\CONST~c) & \quad \mbox{if}~ c = {\VNARROW}{{{}_{{\mathit{sh}}_1, {\mathit{sh}}_2}^{{\sx}}}}{(c_1, c_2)} \\
   \end{array}


.. _exec-vcvtop:


:math:`{\mathit{sh}}_2 {.} {{\vcvtop}}{\mathsf{\_}}{{\mathit{sh}}_1}`
.....................................................................


1. Assert: Due to :ref:`validation <valid-vcvtop>`, a value of :ref:`vector type <syntax-vectype>` :math:`\V128` is on the top of the stack.

#. Pop the value :math:`(\V128{.}\CONST~c_1)` from the stack.

#. Let :math:`c` be :math:`{{\vcvtop}}_{{\mathit{sh}}_1, {\mathit{sh}}_2}({\vcvtop}, c_1)`.

#. Push the value :math:`(\V128{.}\CONST~c)` to the stack.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & (\V128{.}\CONST~c_1)~({\mathit{sh}}_2 {.} {{\vcvtop}}{\mathsf{\_}}{{\mathit{sh}}_1}) & \stepto & (\V128{.}\CONST~c) & \quad \mbox{if}~ c = {{\vcvtop}}_{{\mathit{sh}}_1, {\mathit{sh}}_2}({\vcvtop}, c_1) \\
   \end{array}


.. index:: expression
   pair: execution; expression
   single: abstract syntax; expression
.. _exec-expr:

Expressions
~~~~~~~~~~~

An :ref:`expression <syntax-expr>` is *evaluated* relative to a :ref:`current <exec-notation-textual>` :ref:`frame <syntax-frame>` pointing to its containing :ref:`module instance <syntax-moduleinst>`.


:math:`\mathsf{eval\_expr}~{{\instr}^\ast}`
...........................................


1. Execute the sequence :math:`{{\instr}^\ast}`.

#. Pop the value :math:`{\val}` from the stack.

#. Return :math:`{\val}`.



.. math::
   \begin{array}[t]{@{}l@{}rcl@{}l@{}}
   & z ; {{\instr}^\ast} & \steptostar & {z'} ; {{\val}^\ast} & \quad \mbox{if}~ z ; {{\instr}^\ast} \steptostar {z'} ; {{\val}^\ast} \\
   \end{array}

.. note::
   Evaluation iterates this reduction rule until reaching a value.
   Expressions constituting :ref:`function <syntax-func>` bodies are executed during function :ref:`invocation <exec-invoke>`.
