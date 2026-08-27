.. index:: instruction, ! instruction type, context, value, operand stack, ! polymorphism
.. _valid-instr:

Instructions
------------

:ref:`Instructions <syntax-instr>` are classified by :ref:`instruction types <syntax-instrtype>` that describe how they manipulate the :ref:`operand stack <stack>` and initialize :ref:`locals <syntax-local>`:
A type :math:`{t_1^\ast} \to_{{x^\ast}} {t_2^\ast}` describes the required input stack with argument values of types :math:`{t_1^\ast}`` that an instruction pops off
and the provided output stack with result values of types :math:`{t_2^\ast}` that it pushes back.
Moreover, it enumerates the :ref:`indices <syntax-localidx>` :math:`{x^\ast}` of locals that have been set by the instruction.
In most cases, this is empty.

.. note::
   For example, the instruction :math:`\I32 {.} \ADD` has type :math:`\I32~\I32 \rightarrow \I32`,
   consuming two :math:`\I32` values and producing one.
   The instruction :math:`(\LOCALSET~x)` has type :math:`t \to_{x} \epsilon`, provided :math:`t` is the type declared for the local :math:`x`.

Typing extends to :ref:`instruction sequences <valid-instrs>` :math:`{{\instr}^\ast}`.
Such a sequence has an instruction type :math:`{t_1^\ast} \to_{{x^\ast}} {t_2^\ast}` if the accumulative effect of executing the instructions is consuming values of types :math:`{t_1^\ast}` off the operand stack, pushing new values of types :math:`{t_2^\ast}`, and setting all locals :math:`{x^\ast}`.

.. _polymorphism:

For some instructions, the typing rules do not fully constrain the type,
and therefore allow for multiple types.
Such instructions are called *polymorphic*.
Two degrees of polymorphism can be distinguished:

* *value-polymorphic*:
  the :ref:`value type <syntax-valtype>` :math:`t` of one or several individual operands is unconstrained.
  That is the case for all :ref:`parametric instructions <valid-instr-parametric>` like :math:`\mathsf{drop}` and :math:`\mathsf{select}`.

* *stack-polymorphic*:
  the entire (or most of the) :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast} \rightarrow {t_2^\ast}` of the instruction is unconstrained.
  That is the case for all :ref:`control instructions <valid-instr-control>` that perform an *unconditional control transfer*, such as :math:`\mathsf{unreachable}`, :math:`\mathsf{br}`, or :math:`\mathsf{return}`.

In both cases, the unconstrained types or type sequences can be chosen arbitrarily, as long as they are valid in the current :ref:`context <context>` and meet the constraints imposed for the surrounding parts of the program.

.. note::
   For example, the :math:`\mathsf{select}` instruction is valid with type :math:`t~t~\I32 \rightarrow t`, for any possible :ref:`number type <syntax-numtype>` :math:`t`.
   Consequently, both instruction sequences

   .. math::
      (\I32{.}\CONST~1)~(\I32{.}\CONST~2)~(\I32{.}\CONST~3)~(\SELECT)

   and

   .. math::
      (\F64{.}\CONST~{+1})~(\F64{.}\CONST~{+2})~(\I32{.}\CONST~3)~(\SELECT)

   are valid, with :math:`t` in the typing of :math:`\mathsf{select}` being instantiated to :math:`\mathsf{i{\scriptstyle 32}}` or :math:`\mathsf{f{\scriptstyle 64}}`, respectively.

   The :math:`\mathsf{unreachable}` instruction is stack-polymorphic,
   and hence valid with type :math:`{t_1^\ast} \rightarrow {t_2^\ast}` for any possible sequences of value types :math:`{t_1^\ast}` and :math:`{t_2^\ast}`.
   Consequently,

   .. math::
      (\UNREACHABLE)~(\I32 {.} \ADD)

   is valid by assuming type :math:`\epsilon \rightarrow \I32` for the :math:`\mathsf{unreachable}` instruction.
   In contrast,

   .. math::
      (\UNREACHABLE)~(\I64{.}\CONST~0)~(\I32 {.} \ADD)

   is invalid, because there is no possible type to pick for the :math:`\mathsf{unreachable}` instruction that would make the sequence well-typed.

The :ref:`Appendix <algo-valid>` describes a type checking :ref:`algorithm <algo-valid>` that efficiently implements validation of instruction sequences as prescribed by the rules given here.


.. index:: parametric instructions, value type, polymorphism
   pair: validation; instruction
   single: abstract syntax; instruction
.. _valid-instr-parametric:

Parametric Instructions
~~~~~~~~~~~~~~~~~~~~~~~

.. _valid-nop:

:math:`\NOP`
............




The :ref:`instruction <syntax-instr>` :math:`\NOP` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\epsilon~\rightarrow~\epsilon`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr \NOP : \epsilon \rightarrow \epsilon
   }
   \qquad
   \end{array}


.. _valid-unreachable:

:math:`\UNREACHABLE`
....................




The :ref:`instruction <syntax-instr>` :math:`\UNREACHABLE` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~\rightarrow~{t_2^\ast}` if:


   * The :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~\rightarrow~{t_2^\ast}` is :ref:`valid <valid-instrtype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashinstrtype {t_1^\ast} \rightarrow {t_2^\ast} : \OKinstrtype
   }{
   C \vdashinstr \UNREACHABLE : {t_1^\ast} \rightarrow {t_2^\ast}
   }
   \qquad
   \end{array}

.. note::
   The :math:`\mathsf{unreachable}` instruction is :ref:`stack-polymorphic <polymorphism>`.


.. _valid-drop:

:math:`\DROP`
.............




The :ref:`instruction <syntax-instr>` :math:`\DROP` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`t~\rightarrow~\epsilon` if:


   * The :ref:`value type <syntax-valtype>` :math:`t` is :ref:`valid <valid-valtype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashvaltype t : \OKvaltype
   }{
   C \vdashinstr \DROP : t \rightarrow \epsilon
   }
   \qquad
   \end{array}

.. note::
   Both :math:`\mathsf{drop}` and :math:`\mathsf{select}` without annotation are :ref:`value-polymorphic <polymorphism>` instructions.


.. _valid-select:

:math:`\SELECT~(t^\ast)^?`
..........................




The :ref:`instruction <syntax-instr>` :math:`(\SELECT~{{\valtype}^?})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`t~t~\I32~\rightarrow~t` if:


   * The :ref:`value type <syntax-valtype>` :math:`t` is :ref:`valid <valid-valtype>`.

   * Either:

      * The value type sequence :math:`{{\valtype}^?}` is of the form :math:`t`.

   * Or:

      * The value type sequence :math:`{{\valtype}^?}` is absent.

      * The :ref:`value type <syntax-valtype>` :math:`t` :ref:`matches <match-valtype>` the :ref:`value type <syntax-valtype>` :math:`{t'}`.

      * The :ref:`value type <syntax-valtype>` :math:`{t'}` is of the form :math:`{\numtype}` or :math:`{t'}` is of the form :math:`{\vectype}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashvaltype t : \OKvaltype
   }{
   C \vdashinstr \SELECT~t : t~t~\I32 \rightarrow t
   }
   \qquad
   \frac{
   C \vdashvaltype t : \OKvaltype
    \qquad
   C \vdashvaltypematch t \subvaltypematch {t'}
    \qquad
   {t'} = {\numtype} \lor {t'} = {\vectype}
   }{
   C \vdashinstr \SELECT : t~t~\I32 \rightarrow t
   }
   \qquad
   \end{array}

.. note::
   In future versions of WebAssembly, :math:`\mathsf{select}` may allow more than one value per choice.


.. index:: control instructions, structured control, label, block, branch, block type, label index, result type, function index, type index, tag index, list, polymorphism, context
   pair: validation; instruction
   single: abstract syntax; instruction
.. _valid-label:
.. _valid-instr-control:

Control Instructions
~~~~~~~~~~~~~~~~~~~~

.. _valid-block:

:math:`\BLOCK~\blocktype~\instr^\ast`
.....................................




The :ref:`instruction <syntax-instr>` :math:`(\BLOCK~{\mathit{bt}}~{{\instr}^\ast})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~\rightarrow~{t_2^\ast}` if:


   * The :ref:`block type <syntax-blocktype>` :math:`{\mathit{bt}}` is :ref:`valid <valid-blocktype>` as the :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~\rightarrow~{t_2^\ast}`.

   * Let :math:`{C'}` be the same context as :math:`C`, but with the result type sequence :math:`{t_2^\ast}` prepended to the field :math:`\CLABELS`.

   * Under the context :math:`{C'}`, the instruction sequence :math:`{{\instr}^\ast}` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~{\to}_{{x^\ast}}\,{t_2^\ast}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashblocktype {\mathit{bt}} : {t_1^\ast} \rightarrow {t_2^\ast}
    \qquad
   \{ \CLABELS~({t_2^\ast}) \} \oplus C \vdashinstrs {{\instr}^\ast} : {t_1^\ast} \to_{{x^\ast}} {t_2^\ast}
   }{
   C \vdashinstr \BLOCK~{\mathit{bt}}~{{\instr}^\ast} : {t_1^\ast} \rightarrow {t_2^\ast}
   }
   \qquad
   \end{array}

.. note::
   The :ref:`notation <notation-concat>` :math:`\{ \CLABELS~({t^\ast}) \} \oplus C` inserts the new label type at index :math:`0`, shifting all others.
   The same applies to all other block instructions.


.. _valid-loop:

:math:`\LOOP~\blocktype~\instr^\ast`
....................................




The :ref:`instruction <syntax-instr>` :math:`(\LOOP~{\mathit{bt}}~{{\instr}^\ast})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~\rightarrow~{t_2^\ast}` if:


   * The :ref:`block type <syntax-blocktype>` :math:`{\mathit{bt}}` is :ref:`valid <valid-blocktype>` as the :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~\rightarrow~{t_2^\ast}`.

   * Let :math:`{C'}` be the same context as :math:`C`, but with the result type sequence :math:`{t_1^\ast}` prepended to the field :math:`\CLABELS`.

   * Under the context :math:`{C'}`, the instruction sequence :math:`{{\instr}^\ast}` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~{\to}_{{x^\ast}}\,{t_2^\ast}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashblocktype {\mathit{bt}} : {t_1^\ast} \rightarrow {t_2^\ast}
    \qquad
   \{ \CLABELS~({t_1^\ast}) \} \oplus C \vdashinstrs {{\instr}^\ast} : {t_1^\ast} \to_{{x^\ast}} {t_2^\ast}
   }{
   C \vdashinstr \LOOP~{\mathit{bt}}~{{\instr}^\ast} : {t_1^\ast} \rightarrow {t_2^\ast}
   }
   \qquad
   \end{array}


.. _valid-if:

:math:`\IF~\blocktype~\instr_1^\ast~\ELSE~\instr_2^\ast`
........................................................




The :ref:`instruction <syntax-instr>` :math:`(\IF~{\mathit{bt}}~{{\instr}_1^\ast}~\ELSE~{{\instr}_2^\ast})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~\I32~\rightarrow~{t_2^\ast}` if:


   * The :ref:`block type <syntax-blocktype>` :math:`{\mathit{bt}}` is :ref:`valid <valid-blocktype>` as the :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~\rightarrow~{t_2^\ast}`.

   * Let :math:`{C'}` be the same context as :math:`C`, but with the result type sequence :math:`{t_2^\ast}` prepended to the field :math:`\CLABELS`.

   * Under the context :math:`{C'}`, the instruction sequence :math:`{{\instr}_1^\ast}` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~{\to}_{{x_1^\ast}}\,{t_2^\ast}`.

   * Under the context :math:`{C'}`, the instruction sequence :math:`{{\instr}_2^\ast}` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~{\to}_{{x_2^\ast}}\,{t_2^\ast}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashblocktype {\mathit{bt}} : {t_1^\ast} \rightarrow {t_2^\ast}
    \qquad
   \{ \CLABELS~({t_2^\ast}) \} \oplus C \vdashinstrs {{\instr}_1^\ast} : {t_1^\ast} \to_{{x_1^\ast}} {t_2^\ast}
    \qquad
   \{ \CLABELS~({t_2^\ast}) \} \oplus C \vdashinstrs {{\instr}_2^\ast} : {t_1^\ast} \to_{{x_2^\ast}} {t_2^\ast}
   }{
   C \vdashinstr \IF~{\mathit{bt}}~{{\instr}_1^\ast}~\ELSE~{{\instr}_2^\ast} : {t_1^\ast}~\I32 \rightarrow {t_2^\ast}
   }
   \qquad
   \end{array}


.. _valid-br:

:math:`\BR~l`
.............




The :ref:`instruction <syntax-instr>` :math:`(\BR~l)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~{t^\ast}~\rightarrow~{t_2^\ast}` if:


   * The :ref:`label <syntax-resulttype>` :math:`C{.}\CLABELS{}[l]` exists.

   * The :ref:`label <syntax-resulttype>` :math:`C{.}\CLABELS{}[l]` is of the form :math:`{t^\ast}`.

   * The :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~\rightarrow~{t_2^\ast}` is :ref:`valid <valid-instrtype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CLABELS{}[l] = {t^\ast}
    \qquad
   C \vdashinstrtype {t_1^\ast} \rightarrow {t_2^\ast} : \OKinstrtype
   }{
   C \vdashinstr \BR~l : {t_1^\ast}~{t^\ast} \rightarrow {t_2^\ast}
   }
   \qquad
   \end{array}

.. note::
   The :ref:`label index <syntax-labelidx>` space in the :ref:`context <context>` :math:`C` contains the most recent label first, so that :math:`C{.}\mathsf{labels}{}[l]` performs a relative lookup as expected.
   This applies to other branch instructions as well.

   The :math:`\mathsf{br}` instruction is :ref:`stack-polymorphic <polymorphism>`.


.. _valid-br_if:

:math:`\BRIF~l`
...............




The :ref:`instruction <syntax-instr>` :math:`(\BRIF~l)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t^\ast}~\I32~\rightarrow~{t^\ast}` if:


   * The :ref:`label <syntax-resulttype>` :math:`C{.}\CLABELS{}[l]` exists.

   * The :ref:`label <syntax-resulttype>` :math:`C{.}\CLABELS{}[l]` is of the form :math:`{t^\ast}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CLABELS{}[l] = {t^\ast}
   }{
   C \vdashinstr \BRIF~l : {t^\ast}~\I32 \rightarrow {t^\ast}
   }
   \qquad
   \end{array}


.. _valid-br_table:

:math:`\BRTABLE~l^\ast~l_N`
...........................




The :ref:`instruction <syntax-instr>` :math:`(\BRTABLE~{l^\ast}~{l'})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~{t^\ast}~\I32~\rightarrow~{t_2^\ast}` if:


   * For all :math:`l` in :math:`{l^\ast}`:

      * The :ref:`label <syntax-resulttype>` :math:`C{.}\CLABELS{}[l]` exists.

      * The :ref:`result type <syntax-resulttype>` :math:`{t^\ast}` :ref:`matches <match-resulttype>` the :ref:`label <syntax-resulttype>` :math:`C{.}\CLABELS{}[l]`.

   * The :ref:`label <syntax-resulttype>` :math:`C{.}\CLABELS{}[{l'}]` exists.

   * The :ref:`result type <syntax-resulttype>` :math:`{t^\ast}` :ref:`matches <match-resulttype>` the :ref:`label <syntax-resulttype>` :math:`C{.}\CLABELS{}[{l'}]`.

   * The :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~{t^\ast}~\I32~\rightarrow~{t_2^\ast}` is :ref:`valid <valid-instrtype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   (C \vdashresulttypematch {t^\ast} \subresulttypematch C{.}\CLABELS{}[l])^\ast
    \qquad
   C \vdashresulttypematch {t^\ast} \subresulttypematch C{.}\CLABELS{}[{l'}]
    \qquad
   C \vdashinstrtype {t_1^\ast}~{t^\ast}~\I32 \rightarrow {t_2^\ast} : \OKinstrtype
   }{
   C \vdashinstr \BRTABLE~{l^\ast}~{l'} : {t_1^\ast}~{t^\ast}~\I32 \rightarrow {t_2^\ast}
   }
   \qquad
   \end{array}

.. note::
   The :math:`\mathsf{br\_table}` instruction is :ref:`stack-polymorphic <polymorphism>`.

   Furthermore, the :ref:`result type <syntax-resulttype>` :math:`{t^\ast}` is also chosen non-deterministically in this rule.
   Although it may seem necessary to compute :math:`{t^\ast}` as the greatest lower bound of all label types in practice,
   a simple :ref:`sequential algorithm <algo-valid>` does not require this.


.. _valid-br_on_null:

:math:`\BRONNULL~l`
...................




The :ref:`instruction <syntax-instr>` :math:`(\BRONNULL~l)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t^\ast}~(\REF~\NULL~{\mathit{ht}})~\rightarrow~{t^\ast}~(\REF~{\mathit{ht}})` if:


   * The :ref:`label <syntax-resulttype>` :math:`C{.}\CLABELS{}[l]` exists.

   * The :ref:`label <syntax-resulttype>` :math:`C{.}\CLABELS{}[l]` is of the form :math:`{t^\ast}`.

   * The :ref:`heap type <syntax-heaptype>` :math:`{\mathit{ht}}` is :ref:`valid <valid-heaptype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CLABELS{}[l] = {t^\ast}
    \qquad
   C \vdashheaptype {\mathit{ht}} : \OKheaptype
   }{
   C \vdashinstr \BRONNULL~l : {t^\ast}~(\REF~\NULL~{\mathit{ht}}) \rightarrow {t^\ast}~(\REF~{\mathit{ht}})
   }
   \qquad
   \end{array}


.. _valid-br_on_non_null:

:math:`\BRONNONNULL~l`
......................




The :ref:`instruction <syntax-instr>` :math:`(\BRONNONNULL~l)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t^\ast}~(\REF~\NULL~{\mathit{ht}})~\rightarrow~{t^\ast}` if:


   * The :ref:`label <syntax-resulttype>` :math:`C{.}\CLABELS{}[l]` exists.

   * The :ref:`label <syntax-resulttype>` :math:`C{.}\CLABELS{}[l]` is of the form :math:`{t^\ast}~(\REF~{\NULL^?}~{\mathit{ht}})`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CLABELS{}[l] = {t^\ast}~(\REF~{\NULL^?}~{\mathit{ht}})
   }{
   C \vdashinstr \BRONNONNULL~l : {t^\ast}~(\REF~\NULL~{\mathit{ht}}) \rightarrow {t^\ast}
   }
   \qquad
   \end{array}


.. _valid-br_on_cast:

:math:`\BRONCAST~l~\X{rt}_1~\X{rt}_2`
.....................................




The :ref:`instruction <syntax-instr>` :math:`(\BRONCAST~l~{\mathit{rt}}_1~{\mathit{rt}}_2)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t^\ast}~{\mathit{rt}}_1~\rightarrow~{t^\ast}~{\reftype}` if:


   * The :ref:`label <syntax-resulttype>` :math:`C{.}\CLABELS{}[l]` exists.

   * The :ref:`label <syntax-resulttype>` :math:`C{.}\CLABELS{}[l]` is of the form :math:`{t^\ast}~{\mathit{rt}}`.

   * The :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}}_1` is :ref:`valid <valid-reftype>`.

   * The :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}}_2` is :ref:`valid <valid-reftype>`.

   * The :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}}_2` :ref:`matches <match-reftype>` the :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}}_1`.

   * The :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}}_2` :ref:`matches <match-reftype>` the :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}}`.

   * The :ref:`reference type <syntax-reftype>` :math:`{\reftype}` is :math:`{\mathit{rt}}_1 \reftypediff {\mathit{rt}}_2`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CLABELS{}[l] = {t^\ast}~{\mathit{rt}}
    \qquad
   C \vdashreftype {\mathit{rt}}_1 : \OKreftype
    \qquad
   C \vdashreftype {\mathit{rt}}_2 : \OKreftype
    \qquad
   C \vdashreftypematch {\mathit{rt}}_2 \subreftypematch {\mathit{rt}}_1
    \qquad
   C \vdashreftypematch {\mathit{rt}}_2 \subreftypematch {\mathit{rt}}
   }{
   C \vdashinstr \BRONCAST~l~{\mathit{rt}}_1~{\mathit{rt}}_2 : {t^\ast}~{\mathit{rt}}_1 \rightarrow {t^\ast}~({\mathit{rt}}_1 \reftypediff {\mathit{rt}}_2)
   }
   \qquad
   \end{array}


.. _valid-br_on_cast_fail:

:math:`\BRONCASTFAIL~l~\X{rt}_1~\X{rt}_2`
.........................................




The :ref:`instruction <syntax-instr>` :math:`(\BRONCASTFAIL~l~{\mathit{rt}}_1~{\mathit{rt}}_2)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t^\ast}~{\mathit{rt}}_1~\rightarrow~{t^\ast}~{\mathit{rt}}_2` if:


   * The :ref:`label <syntax-resulttype>` :math:`C{.}\CLABELS{}[l]` exists.

   * The :ref:`label <syntax-resulttype>` :math:`C{.}\CLABELS{}[l]` is of the form :math:`{t^\ast}~{\mathit{rt}}`.

   * The :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}}_1` is :ref:`valid <valid-reftype>`.

   * The :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}}_2` is :ref:`valid <valid-reftype>`.

   * The :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}}_2` :ref:`matches <match-reftype>` the :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}}_1`.

   * The :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}}_1 \reftypediff {\mathit{rt}}_2` :ref:`matches <match-reftype>` the :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CLABELS{}[l] = {t^\ast}~{\mathit{rt}}
    \qquad
   C \vdashreftype {\mathit{rt}}_1 : \OKreftype
    \qquad
   C \vdashreftype {\mathit{rt}}_2 : \OKreftype
    \qquad
   C \vdashreftypematch {\mathit{rt}}_2 \subreftypematch {\mathit{rt}}_1
    \qquad
   C \vdashreftypematch {\mathit{rt}}_1 \reftypediff {\mathit{rt}}_2 \subreftypematch {\mathit{rt}}
   }{
   C \vdashinstr \BRONCASTFAIL~l~{\mathit{rt}}_1~{\mathit{rt}}_2 : {t^\ast}~{\mathit{rt}}_1 \rightarrow {t^\ast}~{\mathit{rt}}_2
   }
   \qquad
   \end{array}


.. _valid-call:

:math:`\CALL~x`
...............




The :ref:`instruction <syntax-instr>` :math:`(\CALL~x)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~\rightarrow~{t_2^\ast}` if:


   * The :ref:`function <syntax-deftype>` :math:`C{.}\CFUNCS{}[x]` exists.

   * The :ref:`expansion <aux-expand-deftype>` of :math:`C{.}\CFUNCS{}[x]` is :math:`(\TFUNC~{t_1^\ast}~\Tarrow~{t_2^\ast})`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CFUNCS{}[x] \approxexpanddt \TFUNC~{t_1^\ast} \Tarrow {t_2^\ast}
   }{
   C \vdashinstr \CALL~x : {t_1^\ast} \rightarrow {t_2^\ast}
   }
   \qquad
   \end{array}


.. _valid-call_ref:

:math:`\CALLREF~x`
..................




The :ref:`instruction <syntax-instr>` :math:`(\CALLREF~x)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~(\REF~\NULL~x)~\rightarrow~{t_2^\ast}` if:


   * The :ref:`type <syntax-deftype>` :math:`C{.}\CTYPES{}[x]` exists.

   * The :ref:`expansion <aux-expand-deftype>` of :math:`C{.}\CTYPES{}[x]` is :math:`(\TFUNC~{t_1^\ast}~\Tarrow~{t_2^\ast})`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTYPES{}[x] \approxexpanddt \TFUNC~{t_1^\ast} \Tarrow {t_2^\ast}
   }{
   C \vdashinstr \CALLREF~x : {t_1^\ast}~(\REF~\NULL~x) \rightarrow {t_2^\ast}
   }
   \qquad
   \end{array}


.. _valid-call_indirect:

:math:`\CALLINDIRECT~x~y`
.........................




The :ref:`instruction <syntax-instr>` :math:`(\CALLINDIRECT~x~y)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~{\mathit{at}}~\rightarrow~{t_2^\ast}` if:


   * The :ref:`table <syntax-tabletype>` :math:`C{.}\CTABLES{}[x]` exists.

   * The :ref:`table <syntax-tabletype>` :math:`C{.}\CTABLES{}[x]` is of the form :math:`({\mathit{at}}~{\mathit{lim}}~{\mathit{rt}})`.

   * The :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}}` :ref:`matches <match-reftype>` the :ref:`reference type <syntax-reftype>` :math:`(\REF~\NULL~\FUNCT)`.

   * The :ref:`type <syntax-deftype>` :math:`C{.}\CTYPES{}[y]` exists.

   * The :ref:`expansion <aux-expand-deftype>` of :math:`C{.}\CTYPES{}[y]` is :math:`(\TFUNC~{t_1^\ast}~\Tarrow~{t_2^\ast})`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTABLES{}[x] = {\mathit{at}}~{\mathit{lim}}~{\mathit{rt}}
    \qquad
   C \vdashreftypematch {\mathit{rt}} \subreftypematch (\REF~\NULL~\FUNCT)
    \qquad
   C{.}\CTYPES{}[y] \approxexpanddt \TFUNC~{t_1^\ast} \Tarrow {t_2^\ast}
   }{
   C \vdashinstr \CALLINDIRECT~x~y : {t_1^\ast}~{\mathit{at}} \rightarrow {t_2^\ast}
   }
   \qquad
   \end{array}


.. _valid-return:

:math:`\RETURN`
...............




The :ref:`instruction <syntax-instr>` :math:`\RETURN` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~{t^\ast}~\rightarrow~{t_2^\ast}` if:


   * The result type :math:`C{.}\CRETURN` is of the form :math:`{t^\ast}`.

   * The :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~\rightarrow~{t_2^\ast}` is :ref:`valid <valid-instrtype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CRETURN = ({t^\ast})
    \qquad
   C \vdashinstrtype {t_1^\ast} \rightarrow {t_2^\ast} : \OKinstrtype
   }{
   C \vdashinstr \RETURN : {t_1^\ast}~{t^\ast} \rightarrow {t_2^\ast}
   }
   \qquad
   \end{array}

.. note::
   The :math:`\mathsf{return}` instruction is :ref:`stack-polymorphic <polymorphism>`.

   :math:`C{.}\CRETURN` is absent (set to :math:`\epsilon`) when validating an :ref:`expression <valid-expr>` that is not a function body.
   This differs from it being set to the empty result type :math:`{}[\epsilon]`,
   which is the case for functions not returning anything.


.. _valid-return_call:

:math:`\RETURNCALL~x`
.....................




The :ref:`instruction <syntax-instr>` :math:`(\RETURNCALL~x)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t_3^\ast}~{t_1^\ast}~\rightarrow~{t_4^\ast}` if:


   * The :ref:`function <syntax-deftype>` :math:`C{.}\CFUNCS{}[x]` exists.

   * The :ref:`expansion <aux-expand-deftype>` of :math:`C{.}\CFUNCS{}[x]` is :math:`(\TFUNC~{t_1^\ast}~\Tarrow~{t_2^\ast})`.

   * The result type :math:`C{.}\CRETURN` is of the form :math:`{{t'}_2^\ast}`.

   * The :ref:`result type <syntax-resulttype>` :math:`{t_2^\ast}` :ref:`matches <match-resulttype>` the :ref:`result type <syntax-resulttype>` :math:`{{t'}_2^\ast}`.

   * The :ref:`instruction type <syntax-instrtype>` :math:`{t_3^\ast}~\rightarrow~{t_4^\ast}` is :ref:`valid <valid-instrtype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CFUNCS{}[x] \approxexpanddt \TFUNC~{t_1^\ast} \Tarrow {t_2^\ast}
    \qquad
   C{.}\CRETURN = ({{t'}_2^\ast})
    \qquad
   C \vdashresulttypematch {t_2^\ast} \subresulttypematch {{t'}_2^\ast}
    \qquad
   C \vdashinstrtype {t_3^\ast} \rightarrow {t_4^\ast} : \OKinstrtype
   }{
   C \vdashinstr \RETURNCALL~x : {t_3^\ast}~{t_1^\ast} \rightarrow {t_4^\ast}
   }
   \qquad
   \end{array}

.. note::
   The :math:`\mathsf{return\_call}` instruction is :ref:`stack-polymorphic <polymorphism>`.


.. _valid-return_call_ref:

:math:`\RETURNCALLREF~x`
........................




The :ref:`instruction <syntax-instr>` :math:`(\RETURNCALLREF~x)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t_3^\ast}~{t_1^\ast}~(\REF~\NULL~x)~\rightarrow~{t_4^\ast}` if:


   * The :ref:`type <syntax-deftype>` :math:`C{.}\CTYPES{}[x]` exists.

   * The :ref:`expansion <aux-expand-deftype>` of :math:`C{.}\CTYPES{}[x]` is :math:`(\TFUNC~{t_1^\ast}~\Tarrow~{t_2^\ast})`.

   * The result type :math:`C{.}\CRETURN` is of the form :math:`{{t'}_2^\ast}`.

   * The :ref:`result type <syntax-resulttype>` :math:`{t_2^\ast}` :ref:`matches <match-resulttype>` the :ref:`result type <syntax-resulttype>` :math:`{{t'}_2^\ast}`.

   * The :ref:`instruction type <syntax-instrtype>` :math:`{t_3^\ast}~\rightarrow~{t_4^\ast}` is :ref:`valid <valid-instrtype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTYPES{}[x] \approxexpanddt \TFUNC~{t_1^\ast} \Tarrow {t_2^\ast}
    \qquad
   C{.}\CRETURN = ({{t'}_2^\ast})
    \qquad
   C \vdashresulttypematch {t_2^\ast} \subresulttypematch {{t'}_2^\ast}
    \qquad
   C \vdashinstrtype {t_3^\ast} \rightarrow {t_4^\ast} : \OKinstrtype
   }{
   C \vdashinstr \RETURNCALLREF~x : {t_3^\ast}~{t_1^\ast}~(\REF~\NULL~x) \rightarrow {t_4^\ast}
   }
   \qquad
   \end{array}

.. note::
   The :math:`\mathsf{return\_call\_ref}` instruction is :ref:`stack-polymorphic <polymorphism>`.


.. _valid-return_call_indirect:

:math:`\RETURNCALLINDIRECT~x~y`
...............................




The :ref:`instruction <syntax-instr>` :math:`(\RETURNCALLINDIRECT~x~y)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t_3^\ast}~{t_1^\ast}~{\mathit{at}}~\rightarrow~{t_4^\ast}` if:


   * The :ref:`table <syntax-tabletype>` :math:`C{.}\CTABLES{}[x]` exists.

   * The :ref:`table <syntax-tabletype>` :math:`C{.}\CTABLES{}[x]` is of the form :math:`({\mathit{at}}~{\mathit{lim}}~{\mathit{rt}})`.

   * The :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}}` :ref:`matches <match-reftype>` the :ref:`reference type <syntax-reftype>` :math:`(\REF~\NULL~\FUNCT)`.

   * The :ref:`type <syntax-deftype>` :math:`C{.}\CTYPES{}[y]` exists.

   * The :ref:`expansion <aux-expand-deftype>` of :math:`C{.}\CTYPES{}[y]` is :math:`(\TFUNC~{t_1^\ast}~\Tarrow~{t_2^\ast})`.

   * The result type :math:`C{.}\CRETURN` is of the form :math:`{{t'}_2^\ast}`.

   * The :ref:`result type <syntax-resulttype>` :math:`{t_2^\ast}` :ref:`matches <match-resulttype>` the :ref:`result type <syntax-resulttype>` :math:`{{t'}_2^\ast}`.

   * The :ref:`instruction type <syntax-instrtype>` :math:`{t_3^\ast}~\rightarrow~{t_4^\ast}` is :ref:`valid <valid-instrtype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   \begin{array}{@{}c@{}}
   C{.}\CTABLES{}[x] = {\mathit{at}}~{\mathit{lim}}~{\mathit{rt}}
    \qquad
   C \vdashreftypematch {\mathit{rt}} \subreftypematch (\REF~\NULL~\FUNCT)
    \\
   C{.}\CTYPES{}[y] \approxexpanddt \TFUNC~{t_1^\ast} \Tarrow {t_2^\ast}
    \qquad
   C{.}\CRETURN = ({{t'}_2^\ast})
    \qquad
   C \vdashresulttypematch {t_2^\ast} \subresulttypematch {{t'}_2^\ast}
    \qquad
   C \vdashinstrtype {t_3^\ast} \rightarrow {t_4^\ast} : \OKinstrtype
   \end{array}
   }{
   C \vdashinstr \RETURNCALLINDIRECT~x~y : {t_3^\ast}~{t_1^\ast}~{\mathit{at}} \rightarrow {t_4^\ast}
   }
   \qquad
   \end{array}

.. note::
   The :math:`\mathsf{return\_call\_indirect}` instruction is :ref:`stack-polymorphic <polymorphism>`.


.. _valid-throw:

:math:`\THROW~x`
................




The :ref:`instruction <syntax-instr>` :math:`(\THROW~x)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~{t^\ast}~\rightarrow~{t_2^\ast}` if:


   * The :ref:`tag <syntax-tagtype>` :math:`C{.}\CTAGS{}[x]` exists.

   * The :ref:`expansion <aux-expand-deftype>` of :math:`C{.}\CTAGS{}[x]` is :math:`(\TFUNC~{t^\ast}~\Tarrow)`.

   * The :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~\rightarrow~{t_2^\ast}` is :ref:`valid <valid-instrtype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTAGS{}[x] \approxexpanddt \TFUNC~{t^\ast} \Tarrow \epsilon
    \qquad
   C \vdashinstrtype {t_1^\ast} \rightarrow {t_2^\ast} : \OKinstrtype
   }{
   C \vdashinstr \THROW~x : {t_1^\ast}~{t^\ast} \rightarrow {t_2^\ast}
   }
   \qquad
   \end{array}

.. note::
   The :math:`\mathsf{throw}` instruction is :ref:`stack-polymorphic <polymorphism>`.


.. _valid-throw_ref:

:math:`\THROWREF`
.................




The :ref:`instruction <syntax-instr>` :math:`\THROWREF` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~(\REF~\NULL~\EXN)~\rightarrow~{t_2^\ast}` if:


   * The :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~\rightarrow~{t_2^\ast}` is :ref:`valid <valid-instrtype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashinstrtype {t_1^\ast} \rightarrow {t_2^\ast} : \OKinstrtype
   }{
   C \vdashinstr \THROWREF : {t_1^\ast}~(\REF~\NULL~\EXN) \rightarrow {t_2^\ast}
   }
   \qquad
   \end{array}

.. note::
   The :math:`\mathsf{throw\_ref}` instruction is :ref:`stack-polymorphic <polymorphism>`.


.. _valid-try_table:

:math:`\TRYTABLE~\blocktype~\catch^\ast~\instr^\ast`
....................................................




The :ref:`instruction <syntax-instr>` :math:`(\TRYTABLE~{\mathit{bt}}~{{\catch}^\ast}~{{\instr}^\ast})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~\rightarrow~{t_2^\ast}` if:


   * The :ref:`block type <syntax-blocktype>` :math:`{\mathit{bt}}` is :ref:`valid <valid-blocktype>` as the :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~\rightarrow~{t_2^\ast}`.

   * Let :math:`{C'}` be the same context as :math:`C`, but with the result type sequence :math:`{t_2^\ast}` prepended to the field :math:`\CLABELS`.

   * Under the context :math:`{C'}`, the instruction sequence :math:`{{\instr}^\ast}` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~{\to}_{{x^\ast}}\,{t_2^\ast}`.

   * For all :math:`{\catch}` in :math:`{{\catch}^\ast}`:

      * The :ref:`catch clause <syntax-catch>` :math:`{\catch}` is :ref:`valid <valid-catch>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashblocktype {\mathit{bt}} : {t_1^\ast} \rightarrow {t_2^\ast}
    \qquad
   \{ \CLABELS~({t_2^\ast}) \} \oplus C \vdashinstrs {{\instr}^\ast} : {t_1^\ast} \to_{{x^\ast}} {t_2^\ast}
    \qquad
   (C \vdashcatch {\catch} : \OKcatch)^\ast
   }{
   C \vdashinstr \TRYTABLE~{\mathit{bt}}~{{\catch}^\ast}~{{\instr}^\ast} : {t_1^\ast} \rightarrow {t_2^\ast}
   }
   \qquad
   \end{array}


.. _valid-catch:

:math:`\CATCH~x~l`
..................




The :ref:`catch clause <syntax-catch>` :math:`(\CATCH~x~l)` is :ref:`valid <valid-catch>` if:


   * The :ref:`tag <syntax-tagtype>` :math:`C{.}\CTAGS{}[x]` exists.

   * The :ref:`expansion <aux-expand-deftype>` of :math:`C{.}\CTAGS{}[x]` is :math:`(\TFUNC~{t^\ast}~\Tarrow)`.

   * The :ref:`label <syntax-resulttype>` :math:`C{.}\CLABELS{}[l]` exists.

   * The :ref:`result type <syntax-resulttype>` :math:`{t^\ast}` :ref:`matches <match-resulttype>` the :ref:`label <syntax-resulttype>` :math:`C{.}\CLABELS{}[l]`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTAGS{}[x] \approxexpanddt \TFUNC~{t^\ast} \Tarrow \epsilon
    \qquad
   C \vdashresulttypematch {t^\ast} \subresulttypematch C{.}\CLABELS{}[l]
   }{
   C \vdashcatch \CATCH~x~l : \OKcatch
   }
   \qquad
   \end{array}


:math:`\CATCHREF~x~l`
.....................




The :ref:`catch clause <syntax-catch>` :math:`(\CATCHREF~x~l)` is :ref:`valid <valid-catch>` if:


   * The :ref:`tag <syntax-tagtype>` :math:`C{.}\CTAGS{}[x]` exists.

   * The :ref:`expansion <aux-expand-deftype>` of :math:`C{.}\CTAGS{}[x]` is :math:`(\TFUNC~{t^\ast}~\Tarrow)`.

   * The :ref:`label <syntax-resulttype>` :math:`C{.}\CLABELS{}[l]` exists.

   * The :ref:`result type <syntax-resulttype>` :math:`{t^\ast}~(\REF~\EXN)` :ref:`matches <match-resulttype>` the :ref:`label <syntax-resulttype>` :math:`C{.}\CLABELS{}[l]`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTAGS{}[x] \approxexpanddt \TFUNC~{t^\ast} \Tarrow \epsilon
    \qquad
   C \vdashresulttypematch {t^\ast}~(\REF~\EXN) \subresulttypematch C{.}\CLABELS{}[l]
   }{
   C \vdashcatch \CATCHREF~x~l : \OKcatch
   }
   \qquad
   \end{array}


:math:`\CATCHALL~l`
...................




The :ref:`catch clause <syntax-catch>` :math:`(\CATCHALL~l)` is :ref:`valid <valid-catch>` if:


   * The :ref:`label <syntax-resulttype>` :math:`C{.}\CLABELS{}[l]` exists.

   * The :ref:`result type <syntax-resulttype>` :math:`\epsilon` :ref:`matches <match-resulttype>` the :ref:`label <syntax-resulttype>` :math:`C{.}\CLABELS{}[l]`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashresulttypematch \epsilon \subresulttypematch C{.}\CLABELS{}[l]
   }{
   C \vdashcatch \CATCHALL~l : \OKcatch
   }
   \qquad
   \end{array}


:math:`\CATCHALLREF~l`
......................




The :ref:`catch clause <syntax-catch>` :math:`(\CATCHALLREF~l)` is :ref:`valid <valid-catch>` if:


   * The :ref:`label <syntax-resulttype>` :math:`C{.}\CLABELS{}[l]` exists.

   * The :ref:`result type <syntax-resulttype>` :math:`(\REF~\EXN)` :ref:`matches <match-resulttype>` the :ref:`label <syntax-resulttype>` :math:`C{.}\CLABELS{}[l]`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashresulttypematch (\REF~\EXN) \subresulttypematch C{.}\CLABELS{}[l]
   }{
   C \vdashcatch \CATCHALLREF~l : \OKcatch
   }
   \qquad
   \end{array}


.. index:: variable instructions, local index, global index, context
   pair: validation; instruction
   single: abstract syntax; instruction
.. _valid-instr-variable:

Variable Instructions
~~~~~~~~~~~~~~~~~~~~~

.. _valid-local.get:

:math:`\LOCALGET~x`
...................




The :ref:`instruction <syntax-instr>` :math:`(\LOCALGET~x)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\epsilon~\rightarrow~t` if:


   * The :ref:`local <syntax-localtype>` :math:`C{.}\CLOCALS{}[x]` exists.

   * The :ref:`local <syntax-localtype>` :math:`C{.}\CLOCALS{}[x]` is of the form :math:`(\SET~t)`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CLOCALS{}[x] = \SET~t
   }{
   C \vdashinstr \LOCALGET~x : \epsilon \rightarrow t
   }
   \qquad
   \end{array}


.. _valid-local.set:

:math:`\LOCALSET~x`
...................




The :ref:`instruction <syntax-instr>` :math:`(\LOCALSET~x)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`t~{\to}_{x}\,\epsilon` if:


   * The :ref:`local <syntax-localtype>` :math:`C{.}\CLOCALS{}[x]` exists.

   * The :ref:`local <syntax-localtype>` :math:`C{.}\CLOCALS{}[x]` is of the form :math:`({\init}~t)`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CLOCALS{}[x] = {\init}~t
   }{
   C \vdashinstr \LOCALSET~x : t \to_{x} \epsilon
   }
   \qquad
   \end{array}


.. _valid-local.tee:

:math:`\LOCALTEE~x`
...................




The :ref:`instruction <syntax-instr>` :math:`(\LOCALTEE~x)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`t~{\to}_{x}\,t` if:


   * The :ref:`local <syntax-localtype>` :math:`C{.}\CLOCALS{}[x]` exists.

   * The :ref:`local <syntax-localtype>` :math:`C{.}\CLOCALS{}[x]` is of the form :math:`({\init}~t)`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CLOCALS{}[x] = {\init}~t
   }{
   C \vdashinstr \LOCALTEE~x : t \to_{x} t
   }
   \qquad
   \end{array}


.. _valid-global.get:

:math:`\GLOBALGET~x`
....................




The :ref:`instruction <syntax-instr>` :math:`(\GLOBALGET~x)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\epsilon~\rightarrow~t` if:


   * The :ref:`global <syntax-globaltype>` :math:`C{.}\CGLOBALS{}[x]` exists.

   * The :ref:`global <syntax-globaltype>` :math:`C{.}\CGLOBALS{}[x]` is of the form :math:`({\TMUT^?}~t)`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CGLOBALS{}[x] = {\TMUT^?}~t
   }{
   C \vdashinstr \GLOBALGET~x : \epsilon \rightarrow t
   }
   \qquad
   \end{array}


.. _valid-global.set:

:math:`\GLOBALSET~x`
....................




The :ref:`instruction <syntax-instr>` :math:`(\GLOBALSET~x)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`t~\rightarrow~\epsilon` if:


   * The :ref:`global <syntax-globaltype>` :math:`C{.}\CGLOBALS{}[x]` exists.

   * The :ref:`global <syntax-globaltype>` :math:`C{.}\CGLOBALS{}[x]` is of the form :math:`(\TMUT~t)`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CGLOBALS{}[x] = \TMUT~t
   }{
   C \vdashinstr \GLOBALSET~x : t \rightarrow \epsilon
   }
   \qquad
   \end{array}


.. index:: table instruction, table index, context
   pair: validation; instruction
   single: abstract syntax; instruction
.. _valid-instr-table:

Table Instructions
~~~~~~~~~~~~~~~~~~

.. _valid-table.get:

:math:`\TABLEGET~x`
...................




The :ref:`instruction <syntax-instr>` :math:`(\TABLEGET~x)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{at}}~\rightarrow~{\mathit{rt}}` if:


   * The :ref:`table <syntax-tabletype>` :math:`C{.}\CTABLES{}[x]` exists.

   * The :ref:`table <syntax-tabletype>` :math:`C{.}\CTABLES{}[x]` is of the form :math:`({\mathit{at}}~{\mathit{lim}}~{\mathit{rt}})`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTABLES{}[x] = {\mathit{at}}~{\mathit{lim}}~{\mathit{rt}}
   }{
   C \vdashinstr \TABLEGET~x : {\mathit{at}} \rightarrow {\mathit{rt}}
   }
   \qquad
   \end{array}


.. _valid-table.set:

:math:`\TABLESET~x`
...................




The :ref:`instruction <syntax-instr>` :math:`(\TABLESET~x)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{at}}~{\mathit{rt}}~\rightarrow~\epsilon` if:


   * The :ref:`table <syntax-tabletype>` :math:`C{.}\CTABLES{}[x]` exists.

   * The :ref:`table <syntax-tabletype>` :math:`C{.}\CTABLES{}[x]` is of the form :math:`({\mathit{at}}~{\mathit{lim}}~{\mathit{rt}})`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTABLES{}[x] = {\mathit{at}}~{\mathit{lim}}~{\mathit{rt}}
   }{
   C \vdashinstr \TABLESET~x : {\mathit{at}}~{\mathit{rt}} \rightarrow \epsilon
   }
   \qquad
   \end{array}


.. _valid-table.size:

:math:`\TABLESIZE~x`
....................




The :ref:`instruction <syntax-instr>` :math:`(\TABLESIZE~x)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\epsilon~\rightarrow~{\mathit{at}}` if:


   * The :ref:`table <syntax-tabletype>` :math:`C{.}\CTABLES{}[x]` exists.

   * The :ref:`table <syntax-tabletype>` :math:`C{.}\CTABLES{}[x]` is of the form :math:`({\mathit{at}}~{\mathit{lim}}~{\mathit{rt}})`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTABLES{}[x] = {\mathit{at}}~{\mathit{lim}}~{\mathit{rt}}
   }{
   C \vdashinstr \TABLESIZE~x : \epsilon \rightarrow {\mathit{at}}
   }
   \qquad
   \end{array}


.. _valid-table.grow:

:math:`\TABLEGROW~x`
....................




The :ref:`instruction <syntax-instr>` :math:`(\TABLEGROW~x)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{rt}}~{\mathit{at}}~\rightarrow~{\mathit{at}}` if:


   * The :ref:`table <syntax-tabletype>` :math:`C{.}\CTABLES{}[x]` exists.

   * The :ref:`table <syntax-tabletype>` :math:`C{.}\CTABLES{}[x]` is of the form :math:`({\mathit{at}}~{\mathit{lim}}~{\mathit{rt}})`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTABLES{}[x] = {\mathit{at}}~{\mathit{lim}}~{\mathit{rt}}
   }{
   C \vdashinstr \TABLEGROW~x : {\mathit{rt}}~{\mathit{at}} \rightarrow {\mathit{at}}
   }
   \qquad
   \end{array}


.. _valid-table.fill:

:math:`\TABLEFILL~x`
....................




The :ref:`instruction <syntax-instr>` :math:`(\TABLEFILL~x)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{at}}~{\mathit{rt}}~{\mathit{at}}~\rightarrow~\epsilon` if:


   * The :ref:`table <syntax-tabletype>` :math:`C{.}\CTABLES{}[x]` exists.

   * The :ref:`table <syntax-tabletype>` :math:`C{.}\CTABLES{}[x]` is of the form :math:`({\mathit{at}}~{\mathit{lim}}~{\mathit{rt}})`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTABLES{}[x] = {\mathit{at}}~{\mathit{lim}}~{\mathit{rt}}
   }{
   C \vdashinstr \TABLEFILL~x : {\mathit{at}}~{\mathit{rt}}~{\mathit{at}} \rightarrow \epsilon
   }
   \qquad
   \end{array}


.. _valid-table.copy:

:math:`\TABLECOPY~x~y`
......................




The :ref:`instruction <syntax-instr>` :math:`(\TABLECOPY~x_1~x_2)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{at}}_1~{\mathit{at}}_2~{\addrtype}~\rightarrow~\epsilon` if:


   * The :ref:`table <syntax-tabletype>` :math:`C{.}\CTABLES{}[x_1]` exists.

   * The :ref:`table <syntax-tabletype>` :math:`C{.}\CTABLES{}[x_1]` is of the form :math:`({\mathit{at}}_1~{\mathit{lim}}_1~{\mathit{rt}}_1)`.

   * The :ref:`table <syntax-tabletype>` :math:`C{.}\CTABLES{}[x_2]` exists.

   * The :ref:`table <syntax-tabletype>` :math:`C{.}\CTABLES{}[x_2]` is of the form :math:`({\mathit{at}}_2~{\mathit{lim}}_2~{\mathit{rt}}_2)`.

   * The :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}}_2` :ref:`matches <match-reftype>` the :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}}_1`.

   * The :ref:`address type <syntax-addrtype>` :math:`{\addrtype}` is :math:`{\addrtypemin}({\mathit{at}}_1, {\mathit{at}}_2)`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTABLES{}[x_1] = {\mathit{at}}_1~{\mathit{lim}}_1~{\mathit{rt}}_1
    \qquad
   C{.}\CTABLES{}[x_2] = {\mathit{at}}_2~{\mathit{lim}}_2~{\mathit{rt}}_2
    \qquad
   C \vdashreftypematch {\mathit{rt}}_2 \subreftypematch {\mathit{rt}}_1
   }{
   C \vdashinstr \TABLECOPY~x_1~x_2 : {\mathit{at}}_1~{\mathit{at}}_2~{\addrtypemin}({\mathit{at}}_1, {\mathit{at}}_2) \rightarrow \epsilon
   }
   \qquad
   \end{array}


.. _valid-table.init:

:math:`\TABLEINIT~x~y`
......................




The :ref:`instruction <syntax-instr>` :math:`(\TABLEINIT~x~y)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{at}}~\I32~\I32~\rightarrow~\epsilon` if:


   * The :ref:`table <syntax-tabletype>` :math:`C{.}\CTABLES{}[x]` exists.

   * The :ref:`table <syntax-tabletype>` :math:`C{.}\CTABLES{}[x]` is of the form :math:`({\mathit{at}}~{\mathit{lim}}~{\mathit{rt}}_1)`.

   * The :ref:`element segment <syntax-elemtype>` :math:`C{.}\CELEMS{}[y]` exists.

   * The :ref:`element segment <syntax-reftype>` :math:`C{.}\CELEMS{}[y]` is of the form :math:`{\mathit{rt}}_2`.

   * The :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}}_2` :ref:`matches <match-reftype>` the :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}}_1`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTABLES{}[x] = {\mathit{at}}~{\mathit{lim}}~{\mathit{rt}}_1
    \qquad
   C{.}\CELEMS{}[y] = {\mathit{rt}}_2
    \qquad
   C \vdashreftypematch {\mathit{rt}}_2 \subreftypematch {\mathit{rt}}_1
   }{
   C \vdashinstr \TABLEINIT~x~y : {\mathit{at}}~\I32~\I32 \rightarrow \epsilon
   }
   \qquad
   \end{array}


.. _valid-elem.drop:

:math:`\ELEMDROP~x`
...................




The :ref:`instruction <syntax-instr>` :math:`(\ELEMDROP~x)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\epsilon~\rightarrow~\epsilon` if:


   * The :ref:`element segment <syntax-elemtype>` :math:`C{.}\CELEMS{}[x]` exists.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CELEMS{}[x] = {\mathit{rt}}
   }{
   C \vdashinstr \ELEMDROP~x : \epsilon \rightarrow \epsilon
   }
   \qquad
   \end{array}


.. index:: memory instruction, memory index, context, memmory argument
   pair: validation; instruction
   single: abstract syntax; instruction
.. _valid-memarg:
.. _valid-instr-memory:

Memory Instructions
~~~~~~~~~~~~~~~~~~~

Memory instructions use :ref:`memory arguments <syntax-memarg>`,
which are classified by the :ref:`address type <syntax-addrtype>` and the
and :ref:`bit width <bitwidth-valtype>` of the access they are suitable for.

:math:`\memarg`
...............




:math:`\{ \ALIGN~n,\;\allowbreak \OFFSET~m \}` is valid for :math:`{\mathit{at}}` and :math:`N` if:


   * :math:`{2^{n}}` is less than or equal to :math:`N / 8`.

   * :math:`m` is less than :math:`{2^{{|{\mathit{at}}|}}}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   {2^{n}} \leq N / 8
    \qquad
   m < {2^{{|{\mathit{at}}|}}}
   }{
   {\vdashmemarg}\, \{ \ALIGN~n,\;\allowbreak \OFFSET~m \} : {\mathit{at}} \arrowmemarg N
   }
   \qquad
   \end{array}


.. _valid-load-val:

:math:`t\K{.}\LOAD~x~\memarg`
.............................




The :ref:`instruction <syntax-instr>` :math:`({\mathit{nt}}{.}\LOAD~x~{\memarg})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{at}}~\rightarrow~{\mathit{nt}}` if:


   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` exists.

   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` is of the form :math:`({\mathit{at}}~{\mathit{lim}}~\PAGE)`.

   * :math:`{\memarg}` is valid for :math:`{\mathit{at}}` and :math:`{|{\mathit{nt}}|}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CMEMS{}[x] = {\mathit{at}}~{\mathit{lim}}~\PAGE
    \qquad
   {\vdashmemarg}\, {\memarg} : {\mathit{at}} \arrowmemarg {|{\mathit{nt}}|}
   }{
   C \vdashinstr {\mathit{nt}}{.}\LOAD~x~{\memarg} : {\mathit{at}} \rightarrow {\mathit{nt}}
   }
   \qquad
   \end{array}


.. _valid-load-pack:

:math:`t\K{.}\LOAD{N}\K{\_}\sx~x~\memarg`
.........................................




The :ref:`instruction <syntax-instr>` :math:`({{\ntI}{{\ntN}}{.}\LOAD}{{K}{\mathsf{\_}}{{\sx}}}~x~{\memarg})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{at}}~\rightarrow~{\ntI}{{\ntN}}` if:


   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` exists.

   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` is of the form :math:`({\mathit{at}}~{\mathit{lim}}~\PAGE)`.

   * :math:`{\memarg}` is valid for :math:`{\mathit{at}}` and :math:`K`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CMEMS{}[x] = {\mathit{at}}~{\mathit{lim}}~\PAGE
    \qquad
   {\vdashmemarg}\, {\memarg} : {\mathit{at}} \arrowmemarg K
   }{
   C \vdashinstr {{\ntI}{{\ntN}}{.}\LOAD}{{K}{\mathsf{\_}}{{\sx}}}~x~{\memarg} : {\mathit{at}} \rightarrow {\ntI}{{\ntN}}
   }
   \qquad
   \end{array}


.. _valid-store-val:

:math:`t\K{.}\STORE~x~\memarg`
..............................




The :ref:`instruction <syntax-instr>` :math:`({\mathit{nt}}{.}\STORE~x~{\memarg})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{at}}~{\mathit{nt}}~\rightarrow~\epsilon` if:


   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` exists.

   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` is of the form :math:`({\mathit{at}}~{\mathit{lim}}~\PAGE)`.

   * :math:`{\memarg}` is valid for :math:`{\mathit{at}}` and :math:`{|{\mathit{nt}}|}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CMEMS{}[x] = {\mathit{at}}~{\mathit{lim}}~\PAGE
    \qquad
   {\vdashmemarg}\, {\memarg} : {\mathit{at}} \arrowmemarg {|{\mathit{nt}}|}
   }{
   C \vdashinstr {\mathit{nt}}{.}\STORE~x~{\memarg} : {\mathit{at}}~{\mathit{nt}} \rightarrow \epsilon
   }
   \qquad
   \end{array}


.. _valid-store-pack:

:math:`t\K{.}\STORE{N}~x~\memarg`
.................................




The :ref:`instruction <syntax-instr>` :math:`({{\ntI}{{\ntN}}{.}\STORE}{K}~x~{\memarg})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{at}}~{\ntI}{{\ntN}}~\rightarrow~\epsilon` if:


   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` exists.

   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` is of the form :math:`({\mathit{at}}~{\mathit{lim}}~\PAGE)`.

   * :math:`{\memarg}` is valid for :math:`{\mathit{at}}` and :math:`K`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CMEMS{}[x] = {\mathit{at}}~{\mathit{lim}}~\PAGE
    \qquad
   {\vdashmemarg}\, {\memarg} : {\mathit{at}} \arrowmemarg K
   }{
   C \vdashinstr {{\ntI}{{\ntN}}{.}\STORE}{K}~x~{\memarg} : {\mathit{at}}~{\ntI}{{\ntN}} \rightarrow \epsilon
   }
   \qquad
   \end{array}


.. _valid-vload-val:

:math:`\K{v128.}\LOAD~x~\memarg`
.....................................




The :ref:`instruction <syntax-instr>` :math:`(\V128{.}\VLOAD~x~{\memarg})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{at}}~\rightarrow~\V128` if:


   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` exists.

   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` is of the form :math:`({\mathit{at}}~{\mathit{lim}}~\PAGE)`.

   * :math:`{\memarg}` is valid for :math:`{\mathit{at}}` and :math:`{|\V128|}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CMEMS{}[x] = {\mathit{at}}~{\mathit{lim}}~\PAGE
    \qquad
   {\vdashmemarg}\, {\memarg} : {\mathit{at}} \arrowmemarg {|\V128|}
   }{
   C \vdashinstr \V128{.}\VLOAD~x~{\memarg} : {\mathit{at}} \rightarrow \V128
   }
   \qquad
   \end{array}


.. _valid-vload-pack:

:math:`\K{v128.}\LOAD{N}\K{x}M\_\sx~x~\memarg`
..............................................




The :ref:`instruction <syntax-instr>` :math:`({\V128{.}\VLOAD}{{N}{\Xshape}{M}{\mathsf{\_}}{{\sx}}}~x~{\memarg})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{at}}~\rightarrow~\V128` if:


   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` exists.

   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` is of the form :math:`({\mathit{at}}~{\mathit{lim}}~\PAGE)`.

   * :math:`{\memarg}` is valid for :math:`{\mathit{at}}` and :math:`N \cdot M`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CMEMS{}[x] = {\mathit{at}}~{\mathit{lim}}~\PAGE
    \qquad
   {\vdashmemarg}\, {\memarg} : {\mathit{at}} \arrowmemarg N \cdot M
   }{
   C \vdashinstr {\V128{.}\VLOAD}{{N}{\Xshape}{M}{\mathsf{\_}}{{\sx}}}~x~{\memarg} : {\mathit{at}} \rightarrow \V128
   }
   \qquad
   \end{array}


.. _valid-vload-splat:

:math:`\K{v128.}\LOAD{N}\K{\_splat}~x~\memarg`
..............................................




The :ref:`instruction <syntax-instr>` :math:`({\V128{.}\VLOAD}{{N}{\mathsf{\_}}{\LSPLAT}}~x~{\memarg})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{at}}~\rightarrow~\V128` if:


   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` exists.

   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` is of the form :math:`({\mathit{at}}~{\mathit{lim}}~\PAGE)`.

   * :math:`{\memarg}` is valid for :math:`{\mathit{at}}` and :math:`N`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CMEMS{}[x] = {\mathit{at}}~{\mathit{lim}}~\PAGE
    \qquad
   {\vdashmemarg}\, {\memarg} : {\mathit{at}} \arrowmemarg N
   }{
   C \vdashinstr {\V128{.}\VLOAD}{{N}{\mathsf{\_}}{\LSPLAT}}~x~{\memarg} : {\mathit{at}} \rightarrow \V128
   }
   \qquad
   \end{array}


.. _valid-vload-zero:

:math:`\K{v128.}\LOAD{N}\K{\_zero}~x~\memarg`
.............................................




The :ref:`instruction <syntax-instr>` :math:`({\V128{.}\VLOAD}{{N}{\mathsf{\_}}{\LZERO}}~x~{\memarg})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{at}}~\rightarrow~\V128` if:


   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` exists.

   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` is of the form :math:`({\mathit{at}}~{\mathit{lim}}~\PAGE)`.

   * :math:`{\memarg}` is valid for :math:`{\mathit{at}}` and :math:`N`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CMEMS{}[x] = {\mathit{at}}~{\mathit{lim}}~\PAGE
    \qquad
   {\vdashmemarg}\, {\memarg} : {\mathit{at}} \arrowmemarg N
   }{
   C \vdashinstr {\V128{.}\VLOAD}{{N}{\mathsf{\_}}{\LZERO}}~x~{\memarg} : {\mathit{at}} \rightarrow \V128
   }
   \qquad
   \end{array}


.. _valid-vload_lane:

:math:`\K{v128.}\LOAD{N}\K{\_lane}~x~\memarg~\laneidx`
......................................................




The :ref:`instruction <syntax-instr>` :math:`({\V128{.}\VLOAD}{N}{\mathsf{\_}}{\VLANE}~x~{\memarg}~i)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{at}}~\V128~\rightarrow~\V128` if:


   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` exists.

   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` is of the form :math:`({\mathit{at}}~{\mathit{lim}}~\PAGE)`.

   * :math:`{\memarg}` is valid for :math:`{\mathit{at}}` and :math:`N`.

   * :math:`i` is less than :math:`128 / N`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CMEMS{}[x] = {\mathit{at}}~{\mathit{lim}}~\PAGE
    \qquad
   {\vdashmemarg}\, {\memarg} : {\mathit{at}} \arrowmemarg N
    \qquad
   i < 128 / N
   }{
   C \vdashinstr {\V128{.}\VLOAD}{N}{\mathsf{\_}}{\VLANE}~x~{\memarg}~i : {\mathit{at}}~\V128 \rightarrow \V128
   }
   \qquad
   \end{array}


.. _valid-vstore:

:math:`\K{v128.}\STORE~x~\memarg`
.................................




The :ref:`instruction <syntax-instr>` :math:`(\V128{.}\VSTORE~x~{\memarg})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{at}}~\V128~\rightarrow~\epsilon` if:


   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` exists.

   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` is of the form :math:`({\mathit{at}}~{\mathit{lim}}~\PAGE)`.

   * :math:`{\memarg}` is valid for :math:`{\mathit{at}}` and :math:`{|\V128|}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CMEMS{}[x] = {\mathit{at}}~{\mathit{lim}}~\PAGE
    \qquad
   {\vdashmemarg}\, {\memarg} : {\mathit{at}} \arrowmemarg {|\V128|}
   }{
   C \vdashinstr \V128{.}\VSTORE~x~{\memarg} : {\mathit{at}}~\V128 \rightarrow \epsilon
   }
   \qquad
   \end{array}


.. _valid-vstore_lane:

:math:`\K{v128.}\STORE{N}\K{\_lane}~x~\memarg~\laneidx`
.......................................................




The :ref:`instruction <syntax-instr>` :math:`({\V128{.}\VSTORE}{N}{\mathsf{\_}}{\VLANE}~x~{\memarg}~i)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{at}}~\V128~\rightarrow~\epsilon` if:


   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` exists.

   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` is of the form :math:`({\mathit{at}}~{\mathit{lim}}~\PAGE)`.

   * :math:`{\memarg}` is valid for :math:`{\mathit{at}}` and :math:`N`.

   * :math:`i` is less than :math:`128 / N`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CMEMS{}[x] = {\mathit{at}}~{\mathit{lim}}~\PAGE
    \qquad
   {\vdashmemarg}\, {\memarg} : {\mathit{at}} \arrowmemarg N
    \qquad
   i < 128 / N
   }{
   C \vdashinstr {\V128{.}\VSTORE}{N}{\mathsf{\_}}{\VLANE}~x~{\memarg}~i : {\mathit{at}}~\V128 \rightarrow \epsilon
   }
   \qquad
   \end{array}


.. _valid-memory.size:

:math:`\MEMORYSIZE~x`
.....................




The :ref:`instruction <syntax-instr>` :math:`(\MEMORYSIZE~x)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\epsilon~\rightarrow~{\mathit{at}}` if:


   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` exists.

   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` is of the form :math:`({\mathit{at}}~{\mathit{lim}}~\PAGE)`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CMEMS{}[x] = {\mathit{at}}~{\mathit{lim}}~\PAGE
   }{
   C \vdashinstr \MEMORYSIZE~x : \epsilon \rightarrow {\mathit{at}}
   }
   \qquad
   \end{array}


.. _valid-memory.grow:

:math:`\MEMORYGROW~x`
.....................




The :ref:`instruction <syntax-instr>` :math:`(\MEMORYGROW~x)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{at}}~\rightarrow~{\mathit{at}}` if:


   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` exists.

   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` is of the form :math:`({\mathit{at}}~{\mathit{lim}}~\PAGE)`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CMEMS{}[x] = {\mathit{at}}~{\mathit{lim}}~\PAGE
   }{
   C \vdashinstr \MEMORYGROW~x : {\mathit{at}} \rightarrow {\mathit{at}}
   }
   \qquad
   \end{array}


.. _valid-memory.fill:

:math:`\MEMORYFILL~x`
.....................




The :ref:`instruction <syntax-instr>` :math:`(\MEMORYFILL~x)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{at}}~\I32~{\mathit{at}}~\rightarrow~\epsilon` if:


   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` exists.

   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` is of the form :math:`({\mathit{at}}~{\mathit{lim}}~\PAGE)`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CMEMS{}[x] = {\mathit{at}}~{\mathit{lim}}~\PAGE
   }{
   C \vdashinstr \MEMORYFILL~x : {\mathit{at}}~\I32~{\mathit{at}} \rightarrow \epsilon
   }
   \qquad
   \end{array}


.. _valid-memory.copy:

:math:`\MEMORYCOPY~x~y`
.......................




The :ref:`instruction <syntax-instr>` :math:`(\MEMORYCOPY~x_1~x_2)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{at}}_1~{\mathit{at}}_2~{\addrtype}~\rightarrow~\epsilon` if:


   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x_1]` exists.

   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x_1]` is of the form :math:`({\mathit{at}}_1~{\mathit{lim}}_1~\PAGE)`.

   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x_2]` exists.

   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x_2]` is of the form :math:`({\mathit{at}}_2~{\mathit{lim}}_2~\PAGE)`.

   * The :ref:`address type <syntax-addrtype>` :math:`{\addrtype}` is :math:`{\addrtypemin}({\mathit{at}}_1, {\mathit{at}}_2)`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CMEMS{}[x_1] = {\mathit{at}}_1~{\mathit{lim}}_1~\PAGE
    \qquad
   C{.}\CMEMS{}[x_2] = {\mathit{at}}_2~{\mathit{lim}}_2~\PAGE
   }{
   C \vdashinstr \MEMORYCOPY~x_1~x_2 : {\mathit{at}}_1~{\mathit{at}}_2~{\addrtypemin}({\mathit{at}}_1, {\mathit{at}}_2) \rightarrow \epsilon
   }
   \qquad
   \end{array}


.. _valid-memory.init:

:math:`\MEMORYINIT~x~y`
.......................




The :ref:`instruction <syntax-instr>` :math:`(\MEMORYINIT~x~y)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{at}}~\I32~\I32~\rightarrow~\epsilon` if:


   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` exists.

   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` is of the form :math:`({\mathit{at}}~{\mathit{lim}}~\PAGE)`.

   * The :ref:`data segment <syntax-datatype>` :math:`C{.}\CDATAS{}[y]` exists.

   * The :ref:`data segment <syntax-datatype>` :math:`C{.}\CDATAS{}[y]` is of the form :math:`\OKdata`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CMEMS{}[x] = {\mathit{at}}~{\mathit{lim}}~\PAGE
    \qquad
   C{.}\CDATAS{}[y] = \OKdata
   }{
   C \vdashinstr \MEMORYINIT~x~y : {\mathit{at}}~\I32~\I32 \rightarrow \epsilon
   }
   \qquad
   \end{array}


.. _valid-data.drop:

:math:`\DATADROP~x`
...................




The :ref:`instruction <syntax-instr>` :math:`(\DATADROP~x)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\epsilon~\rightarrow~\epsilon` if:


   * The :ref:`data segment <syntax-datatype>` :math:`C{.}\CDATAS{}[x]` exists.

   * The :ref:`data segment <syntax-datatype>` :math:`C{.}\CDATAS{}[x]` is of the form :math:`\OKdata`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CDATAS{}[x] = \OKdata
   }{
   C \vdashinstr \DATADROP~x : \epsilon \rightarrow \epsilon
   }
   \qquad
   \end{array}


.. index:: reference instructions, reference type
   pair: validation; instruction
   single: abstract syntax; instruction
.. _valid-instr-ref:

Reference Instructions
~~~~~~~~~~~~~~~~~~~~~~

.. _valid-ref.null:

:math:`\REFNULL~\X{ht}`
.......................




The :ref:`instruction <syntax-instr>` :math:`(\REFNULL~{\mathit{ht}})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\epsilon~\rightarrow~(\REF~\NULL~{\mathit{ht}})` if:


   * The :ref:`heap type <syntax-heaptype>` :math:`{\mathit{ht}}` is :ref:`valid <valid-heaptype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashheaptype {\mathit{ht}} : \OKheaptype
   }{
   C \vdashinstr \REFNULL~{\mathit{ht}} : \epsilon \rightarrow (\REF~\NULL~{\mathit{ht}})
   }
   \qquad
   \end{array}


.. _valid-ref.func:

:math:`\REFFUNC~x`
..................




The :ref:`instruction <syntax-instr>` :math:`(\REFFUNC~x)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\epsilon~\rightarrow~(\REF~{\mathit{dt}})` if:


   * The :ref:`function <syntax-deftype>` :math:`C{.}\CFUNCS{}[x]` exists.

   * The :ref:`function <syntax-deftype>` :math:`C{.}\CFUNCS{}[x]` is of the form :math:`{\mathit{dt}}`.

   * :math:`x` is contained in :math:`C{.}\CREFS`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CFUNCS{}[x] = {\mathit{dt}}
    \qquad
   x \in C{.}\CREFS
   }{
   C \vdashinstr \REFFUNC~x : \epsilon \rightarrow (\REF~{\mathit{dt}})
   }
   \qquad
   \end{array}


.. _valid-ref.is_null:

:math:`\REFISNULL`
..................




The :ref:`instruction <syntax-instr>` :math:`\REFISNULL` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`(\REF~\NULL~{\mathit{ht}})~\rightarrow~\I32` if:


   * The :ref:`heap type <syntax-heaptype>` :math:`{\mathit{ht}}` is :ref:`valid <valid-heaptype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashheaptype {\mathit{ht}} : \OKheaptype
   }{
   C \vdashinstr \REFISNULL : (\REF~\NULL~{\mathit{ht}}) \rightarrow \I32
   }
   \qquad
   \end{array}


.. _valid-ref.as_non_null:

:math:`\REFASNONNULL`
.....................




The :ref:`instruction <syntax-instr>` :math:`\REFASNONNULL` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`(\REF~\NULL~{\mathit{ht}})~\rightarrow~(\REF~{\mathit{ht}})` if:


   * The :ref:`heap type <syntax-heaptype>` :math:`{\mathit{ht}}` is :ref:`valid <valid-heaptype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashheaptype {\mathit{ht}} : \OKheaptype
   }{
   C \vdashinstr \REFASNONNULL : (\REF~\NULL~{\mathit{ht}}) \rightarrow (\REF~{\mathit{ht}})
   }
   \qquad
   \end{array}


.. _valid-ref.eq:

:math:`\REFEQ`
..............




The :ref:`instruction <syntax-instr>` :math:`\REFEQ` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`(\REF~\NULL~\EQT)~(\REF~\NULL~\EQT)~\rightarrow~\I32`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr \REFEQ : (\REF~\NULL~\EQT)~(\REF~\NULL~\EQT) \rightarrow \I32
   }
   \qquad
   \end{array}


.. _valid-ref.test:

:math:`\REFTEST~\X{rt}`
.......................




The :ref:`instruction <syntax-instr>` :math:`(\REFTEST~{\mathit{rt}})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{rt}'}~\rightarrow~\I32` if:


   * The :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}}` is :ref:`valid <valid-reftype>`.

   * The :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}'}` is :ref:`valid <valid-reftype>`.

   * The :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}}` :ref:`matches <match-reftype>` the :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}'}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashreftype {\mathit{rt}} : \OKreftype
    \qquad
   C \vdashreftype {\mathit{rt}'} : \OKreftype
    \qquad
   C \vdashreftypematch {\mathit{rt}} \subreftypematch {\mathit{rt}'}
   }{
   C \vdashinstr \REFTEST~{\mathit{rt}} : {\mathit{rt}'} \rightarrow \I32
   }
   \qquad
   \end{array}

.. note::
   The liberty to pick a supertype :math:`{\mathit{rt}'}` allows typing the instruction with the least precise super type of :math:`{\mathit{rt}}` as input, that is, the top type in the corresponding heap subtyping hierarchy.


.. _valid-ref.cast:

:math:`\REFCAST~\X{rt}`
.......................




The :ref:`instruction <syntax-instr>` :math:`(\REFCAST~{\mathit{rt}})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{rt}'}~\rightarrow~{\mathit{rt}}` if:


   * The :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}}` is :ref:`valid <valid-reftype>`.

   * The :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}'}` is :ref:`valid <valid-reftype>`.

   * The :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}}` :ref:`matches <match-reftype>` the :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}'}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashreftype {\mathit{rt}} : \OKreftype
    \qquad
   C \vdashreftype {\mathit{rt}'} : \OKreftype
    \qquad
   C \vdashreftypematch {\mathit{rt}} \subreftypematch {\mathit{rt}'}
   }{
   C \vdashinstr \REFCAST~{\mathit{rt}} : {\mathit{rt}'} \rightarrow {\mathit{rt}}
   }
   \qquad
   \end{array}

.. note::
   The liberty to pick a supertype :math:`{\mathit{rt}'}` allows typing the instruction with the least precise super type of :math:`{\mathit{rt}}` as input, that is, the top type in the corresponding heap subtyping hierarchy.


.. index:: aggregate reference

Aggregate Reference Instructions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. _valid-struct.new:

:math:`\STRUCTNEW~x`
....................




The :ref:`instruction <syntax-instr>` :math:`(\STRUCTNEW~x)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t^\ast}~\rightarrow~(\REF~x)` if:


   * The :ref:`type <syntax-deftype>` :math:`C{.}\CTYPES{}[x]` exists.

   * The :ref:`expansion <aux-expand-deftype>` of :math:`C{.}\CTYPES{}[x]` is :math:`(\TSTRUCT~{({\TMUT^?}~{\mathit{zt}})^\ast})`.

   * The value type sequence :math:`{t^\ast}` is :math:`{{\unpack}({\mathit{zt}})^\ast}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTYPES{}[x] \approxexpanddt \TSTRUCT~{({\TMUT^?}~{\mathit{zt}})^\ast}
   }{
   C \vdashinstr \STRUCTNEW~x : {{\unpack}({\mathit{zt}})^\ast} \rightarrow (\REF~x)
   }
   \qquad
   \end{array}


.. _valid-struct.new_default:

:math:`\STRUCTNEWDEFAULT~x`
...........................




The :ref:`instruction <syntax-instr>` :math:`(\STRUCTNEWDEFAULT~x)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\epsilon~\rightarrow~(\REF~x)` if:


   * The :ref:`type <syntax-deftype>` :math:`C{.}\CTYPES{}[x]` exists.

   * The :ref:`expansion <aux-expand-deftype>` of :math:`C{.}\CTYPES{}[x]` is :math:`(\TSTRUCT~{({\TMUT^?}~{\mathit{zt}})^\ast})`.

   * For all :math:`{\mathit{zt}}` in :math:`{{\mathit{zt}}^\ast}`:

      * A :ref:`default value <aux-default>` for :math:`{\unpack}({\mathit{zt}})` is defined.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTYPES{}[x] \approxexpanddt \TSTRUCT~{({\TMUT^?}~{\mathit{zt}})^\ast}
    \qquad
   ({{\default}}_{{\unpack}({\mathit{zt}})} \neq \epsilon)^\ast
   }{
   C \vdashinstr \STRUCTNEWDEFAULT~x : \epsilon \rightarrow (\REF~x)
   }
   \qquad
   \end{array}


.. _valid-struct.get:
.. _valid-struct.get_u:
.. _valid-struct.get_s:

:math:`\STRUCTGET\K{\_}\sx^?~x~y`
.................................




The :ref:`instruction <syntax-instr>` :math:`({\STRUCTGET}{\mathsf{\_}}{{{\sx}^?}}~x~i)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`(\REF~\NULL~x)~\rightarrow~t` if:


   * The :ref:`type <syntax-deftype>` :math:`C{.}\CTYPES{}[x]` exists.

   * The :ref:`expansion <aux-expand-deftype>` of :math:`C{.}\CTYPES{}[x]` is :math:`(\TSTRUCT~{{\mathit{ft}}^\ast})`.

   * The length of :math:`{{\mathit{ft}}^\ast}` is greater than :math:`i`.

   * The :ref:`field type <syntax-fieldtype>` :math:`{{\mathit{ft}}^\ast}{}[i]` is of the form :math:`({\TMUT^?}~{\mathit{zt}})`.

   * The signedness :math:`{{\sx}^?}` is present if and only if :math:`{\mathit{zt}}` is a packed type.

   * The :ref:`value type <syntax-valtype>` :math:`t` is :math:`{\unpack}({\mathit{zt}})`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTYPES{}[x] \approxexpanddt \TSTRUCT~{{\mathit{ft}}^\ast}
    \qquad
   {{\mathit{ft}}^\ast}{}[i] = {\TMUT^?}~{\mathit{zt}}
    \qquad
   {{\sx}^?} \neq \epsilon \Leftrightarrow {\mathit{zt}} \neq {\unpack}({\mathit{zt}})
   }{
   C \vdashinstr {\STRUCTGET}{\mathsf{\_}}{{{\sx}^?}}~x~i : (\REF~\NULL~x) \rightarrow {\unpack}({\mathit{zt}})
   }
   \qquad
   \end{array}


.. _valid-struct.set:

:math:`\STRUCTSET~x~y`
......................




The :ref:`instruction <syntax-instr>` :math:`(\STRUCTSET~x~i)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`(\REF~\NULL~x)~t~\rightarrow~\epsilon` if:


   * The :ref:`type <syntax-deftype>` :math:`C{.}\CTYPES{}[x]` exists.

   * The :ref:`expansion <aux-expand-deftype>` of :math:`C{.}\CTYPES{}[x]` is :math:`(\TSTRUCT~{{\mathit{ft}}^\ast})`.

   * The length of :math:`{{\mathit{ft}}^\ast}` is greater than :math:`i`.

   * The :ref:`field type <syntax-fieldtype>` :math:`{{\mathit{ft}}^\ast}{}[i]` is of the form :math:`(\TMUT~{\mathit{zt}})`.

   * The :ref:`value type <syntax-valtype>` :math:`t` is :math:`{\unpack}({\mathit{zt}})`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTYPES{}[x] \approxexpanddt \TSTRUCT~{{\mathit{ft}}^\ast}
    \qquad
   {{\mathit{ft}}^\ast}{}[i] = \TMUT~{\mathit{zt}}
   }{
   C \vdashinstr \STRUCTSET~x~i : (\REF~\NULL~x)~{\unpack}({\mathit{zt}}) \rightarrow \epsilon
   }
   \qquad
   \end{array}


.. _valid-array.new:

:math:`\ARRAYNEW~x`
...................




The :ref:`instruction <syntax-instr>` :math:`(\ARRAYNEW~x)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`t~\I32~\rightarrow~(\REF~x)` if:


   * The :ref:`type <syntax-deftype>` :math:`C{.}\CTYPES{}[x]` exists.

   * The :ref:`expansion <aux-expand-deftype>` of :math:`C{.}\CTYPES{}[x]` is :math:`(\TARRAY~({\TMUT^?}~{\mathit{zt}}))`.

   * The :ref:`value type <syntax-valtype>` :math:`t` is :math:`{\unpack}({\mathit{zt}})`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTYPES{}[x] \approxexpanddt \TARRAY~({\TMUT^?}~{\mathit{zt}})
   }{
   C \vdashinstr \ARRAYNEW~x : {\unpack}({\mathit{zt}})~\I32 \rightarrow (\REF~x)
   }
   \qquad
   \end{array}


.. _valid-array.new_default:

:math:`\ARRAYNEWDEFAULT~x`
..........................




The :ref:`instruction <syntax-instr>` :math:`(\ARRAYNEWDEFAULT~x)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\I32~\rightarrow~(\REF~x)` if:


   * The :ref:`type <syntax-deftype>` :math:`C{.}\CTYPES{}[x]` exists.

   * The :ref:`expansion <aux-expand-deftype>` of :math:`C{.}\CTYPES{}[x]` is :math:`(\TARRAY~({\TMUT^?}~{\mathit{zt}}))`.

   * A :ref:`default value <aux-default>` for :math:`{\unpack}({\mathit{zt}})` is defined.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTYPES{}[x] \approxexpanddt \TARRAY~({\TMUT^?}~{\mathit{zt}})
    \qquad
   {{\default}}_{{\unpack}({\mathit{zt}})} \neq \epsilon
   }{
   C \vdashinstr \ARRAYNEWDEFAULT~x : \I32 \rightarrow (\REF~x)
   }
   \qquad
   \end{array}


.. _valid-array.new_fixed:

:math:`\ARRAYNEWFIXED~x~n`
..........................




The :ref:`instruction <syntax-instr>` :math:`(\ARRAYNEWFIXED~x~n)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t^{n}}~\rightarrow~(\REF~x)` if:


   * The :ref:`type <syntax-deftype>` :math:`C{.}\CTYPES{}[x]` exists.

   * The :ref:`expansion <aux-expand-deftype>` of :math:`C{.}\CTYPES{}[x]` is :math:`(\TARRAY~({\TMUT^?}~{\mathit{zt}}))`.

   * The :ref:`value type <syntax-valtype>` :math:`t` is :math:`{\unpack}({\mathit{zt}})`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTYPES{}[x] \approxexpanddt \TARRAY~({\TMUT^?}~{\mathit{zt}})
   }{
   C \vdashinstr \ARRAYNEWFIXED~x~n : {{\unpack}({\mathit{zt}})^{n}} \rightarrow (\REF~x)
   }
   \qquad
   \end{array}


.. _valid-array.new_elem:

:math:`\ARRAYNEWELEM~x~y`
.........................




The :ref:`instruction <syntax-instr>` :math:`(\ARRAYNEWELEM~x~y)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\I32~\I32~\rightarrow~(\REF~x)` if:


   * The :ref:`type <syntax-deftype>` :math:`C{.}\CTYPES{}[x]` exists.

   * The :ref:`expansion <aux-expand-deftype>` of :math:`C{.}\CTYPES{}[x]` is :math:`(\TARRAY~({\TMUT^?}~{\mathit{rt}}))`.

   * The :ref:`element segment <syntax-elemtype>` :math:`C{.}\CELEMS{}[y]` exists.

   * The :ref:`element segment <syntax-reftype>` :math:`C{.}\CELEMS{}[y]` :ref:`matches <match-reftype>` the :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTYPES{}[x] \approxexpanddt \TARRAY~({\TMUT^?}~{\mathit{rt}})
    \qquad
   C \vdashreftypematch C{.}\CELEMS{}[y] \subreftypematch {\mathit{rt}}
   }{
   C \vdashinstr \ARRAYNEWELEM~x~y : \I32~\I32 \rightarrow (\REF~x)
   }
   \qquad
   \end{array}


.. _valid-array.new_data:

:math:`\ARRAYNEWDATA~x~y`
.........................




The :ref:`instruction <syntax-instr>` :math:`(\ARRAYNEWDATA~x~y)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\I32~\I32~\rightarrow~(\REF~x)` if:


   * The :ref:`type <syntax-deftype>` :math:`C{.}\CTYPES{}[x]` exists.

   * The :ref:`expansion <aux-expand-deftype>` of :math:`C{.}\CTYPES{}[x]` is :math:`(\TARRAY~({\TMUT^?}~{\mathit{zt}}))`.

   * The :ref:`value type <syntax-valtype>` :math:`{\unpack}({\mathit{zt}})` is of the form :math:`{\numtype}` or :math:`{\unpack}({\mathit{zt}})` is of the form :math:`{\vectype}`.

   * The :ref:`data segment <syntax-datatype>` :math:`C{.}\CDATAS{}[y]` exists.

   * The :ref:`data segment <syntax-datatype>` :math:`C{.}\CDATAS{}[y]` is of the form :math:`\OKdata`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTYPES{}[x] \approxexpanddt \TARRAY~({\TMUT^?}~{\mathit{zt}})
    \qquad
   {\unpack}({\mathit{zt}}) = {\numtype} \lor {\unpack}({\mathit{zt}}) = {\vectype}
    \qquad
   C{.}\CDATAS{}[y] = \OKdata
   }{
   C \vdashinstr \ARRAYNEWDATA~x~y : \I32~\I32 \rightarrow (\REF~x)
   }
   \qquad
   \end{array}


.. _valid-array.get:
.. _valid-array.get_u:
.. _valid-array.get_s:

:math:`\ARRAYGET\K{\_}\sx^?~x`
..............................




The :ref:`instruction <syntax-instr>` :math:`({\ARRAYGET}{\mathsf{\_}}{{{\sx}^?}}~x)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`(\REF~\NULL~x)~\I32~\rightarrow~t` if:


   * The :ref:`type <syntax-deftype>` :math:`C{.}\CTYPES{}[x]` exists.

   * The :ref:`expansion <aux-expand-deftype>` of :math:`C{.}\CTYPES{}[x]` is :math:`(\TARRAY~({\TMUT^?}~{\mathit{zt}}))`.

   * The signedness :math:`{{\sx}^?}` is present if and only if :math:`{\mathit{zt}}` is a packed type.

   * The :ref:`value type <syntax-valtype>` :math:`t` is :math:`{\unpack}({\mathit{zt}})`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTYPES{}[x] \approxexpanddt \TARRAY~({\TMUT^?}~{\mathit{zt}})
    \qquad
   {{\sx}^?} \neq \epsilon \Leftrightarrow {\mathit{zt}} \neq {\unpack}({\mathit{zt}})
   }{
   C \vdashinstr {\ARRAYGET}{\mathsf{\_}}{{{\sx}^?}}~x : (\REF~\NULL~x)~\I32 \rightarrow {\unpack}({\mathit{zt}})
   }
   \qquad
   \end{array}


.. _valid-array.set:

:math:`\ARRAYSET~x`
...................




The :ref:`instruction <syntax-instr>` :math:`(\ARRAYSET~x)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`(\REF~\NULL~x)~\I32~t~\rightarrow~\epsilon` if:


   * The :ref:`type <syntax-deftype>` :math:`C{.}\CTYPES{}[x]` exists.

   * The :ref:`expansion <aux-expand-deftype>` of :math:`C{.}\CTYPES{}[x]` is :math:`(\TARRAY~(\TMUT~{\mathit{zt}}))`.

   * The :ref:`value type <syntax-valtype>` :math:`t` is :math:`{\unpack}({\mathit{zt}})`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTYPES{}[x] \approxexpanddt \TARRAY~(\TMUT~{\mathit{zt}})
   }{
   C \vdashinstr \ARRAYSET~x : (\REF~\NULL~x)~\I32~{\unpack}({\mathit{zt}}) \rightarrow \epsilon
   }
   \qquad
   \end{array}


.. _valid-array.len:

:math:`\ARRAYLEN`
.................




The :ref:`instruction <syntax-instr>` :math:`\ARRAYLEN` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`(\REF~\NULL~\ARRAY)~\rightarrow~\I32`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr \ARRAYLEN : (\REF~\NULL~\ARRAY) \rightarrow \I32
   }
   \qquad
   \end{array}


.. _valid-array.fill:

:math:`\ARRAYFILL~x`
....................




The :ref:`instruction <syntax-instr>` :math:`(\ARRAYFILL~x)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`(\REF~\NULL~x)~\I32~t~\I32~\rightarrow~\epsilon` if:


   * The :ref:`type <syntax-deftype>` :math:`C{.}\CTYPES{}[x]` exists.

   * The :ref:`expansion <aux-expand-deftype>` of :math:`C{.}\CTYPES{}[x]` is :math:`(\TARRAY~(\TMUT~{\mathit{zt}}))`.

   * The :ref:`value type <syntax-valtype>` :math:`t` is :math:`{\unpack}({\mathit{zt}})`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTYPES{}[x] \approxexpanddt \TARRAY~(\TMUT~{\mathit{zt}})
   }{
   C \vdashinstr \ARRAYFILL~x : (\REF~\NULL~x)~\I32~{\unpack}({\mathit{zt}})~\I32 \rightarrow \epsilon
   }
   \qquad
   \end{array}


.. _valid-array.copy:

:math:`\ARRAYCOPY~x~y`
......................




The :ref:`instruction <syntax-instr>` :math:`(\ARRAYCOPY~x_1~x_2)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`(\REF~\NULL~x_1)~\I32~(\REF~\NULL~x_2)~\I32~\I32~\rightarrow~\epsilon` if:


   * The :ref:`type <syntax-deftype>` :math:`C{.}\CTYPES{}[x_1]` exists.

   * The :ref:`expansion <aux-expand-deftype>` of :math:`C{.}\CTYPES{}[x_1]` is :math:`(\TARRAY~(\TMUT~{\mathit{zt}}_1))`.

   * The :ref:`type <syntax-deftype>` :math:`C{.}\CTYPES{}[x_2]` exists.

   * The :ref:`expansion <aux-expand-deftype>` of :math:`C{.}\CTYPES{}[x_2]` is :math:`(\TARRAY~({\TMUT^?}~{\mathit{zt}}_2))`.

   * The :ref:`storage type <syntax-storagetype>` :math:`{\mathit{zt}}_2` :ref:`matches <match-storagetype>` the :ref:`storage type <syntax-storagetype>` :math:`{\mathit{zt}}_1`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTYPES{}[x_1] \approxexpanddt \TARRAY~(\TMUT~{\mathit{zt}}_1)
    \qquad
   C{.}\CTYPES{}[x_2] \approxexpanddt \TARRAY~({\TMUT^?}~{\mathit{zt}}_2)
    \qquad
   C \vdashstoragetypematch {\mathit{zt}}_2 \substoragetypematch {\mathit{zt}}_1
   }{
   C \vdashinstr \ARRAYCOPY~x_1~x_2 : (\REF~\NULL~x_1)~\I32~(\REF~\NULL~x_2)~\I32~\I32 \rightarrow \epsilon
   }
   \qquad
   \end{array}


.. _valid-array.init_elem:

:math:`\ARRAYINITELEM~x~y`
..........................




The :ref:`instruction <syntax-instr>` :math:`(\ARRAYINITELEM~x~y)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`(\REF~\NULL~x)~\I32~\I32~\I32~\rightarrow~\epsilon` if:


   * The :ref:`type <syntax-deftype>` :math:`C{.}\CTYPES{}[x]` exists.

   * The :ref:`expansion <aux-expand-deftype>` of :math:`C{.}\CTYPES{}[x]` is :math:`(\TARRAY~(\TMUT~{\mathit{zt}}))`.

   * The :ref:`element segment <syntax-elemtype>` :math:`C{.}\CELEMS{}[y]` exists.

   * The :ref:`element segment <syntax-elemtype>` :math:`C{.}\CELEMS{}[y]` :ref:`matches <match>` the :ref:`storage type <syntax-storagetype>` :math:`{\mathit{zt}}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTYPES{}[x] \approxexpanddt \TARRAY~(\TMUT~{\mathit{zt}})
    \qquad
   C \vdashstoragetypematch C{.}\CELEMS{}[y] \substoragetypematch {\mathit{zt}}
   }{
   C \vdashinstr \ARRAYINITELEM~x~y : (\REF~\NULL~x)~\I32~\I32~\I32 \rightarrow \epsilon
   }
   \qquad
   \end{array}


.. _valid-array.init_data:

:math:`\ARRAYINITDATA~x~y`
..........................




The :ref:`instruction <syntax-instr>` :math:`(\ARRAYINITDATA~x~y)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`(\REF~\NULL~x)~\I32~\I32~\I32~\rightarrow~\epsilon` if:


   * The :ref:`type <syntax-deftype>` :math:`C{.}\CTYPES{}[x]` exists.

   * The :ref:`expansion <aux-expand-deftype>` of :math:`C{.}\CTYPES{}[x]` is :math:`(\TARRAY~(\TMUT~{\mathit{zt}}))`.

   * The :ref:`value type <syntax-valtype>` :math:`{\unpack}({\mathit{zt}})` is of the form :math:`{\numtype}` or :math:`{\unpack}({\mathit{zt}})` is of the form :math:`{\vectype}`.

   * The :ref:`data segment <syntax-datatype>` :math:`C{.}\CDATAS{}[y]` exists.

   * The :ref:`data segment <syntax-datatype>` :math:`C{.}\CDATAS{}[y]` is of the form :math:`\OKdata`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTYPES{}[x] \approxexpanddt \TARRAY~(\TMUT~{\mathit{zt}})
    \qquad
   {\unpack}({\mathit{zt}}) = {\numtype} \lor {\unpack}({\mathit{zt}}) = {\vectype}
    \qquad
   C{.}\CDATAS{}[y] = \OKdata
   }{
   C \vdashinstr \ARRAYINITDATA~x~y : (\REF~\NULL~x)~\I32~\I32~\I32 \rightarrow \epsilon
   }
   \qquad
   \end{array}


.. index:: scalar reference

Scalar Reference Instructions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. _valid-ref.i31:

:math:`\REFI31`
...............




The :ref:`instruction <syntax-instr>` :math:`\REFI31` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\I32~\rightarrow~(\REF~\I31)`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr \REFI31 : \I32 \rightarrow (\REF~\I31)
   }
   \qquad
   \end{array}


.. _valid-i31.get:

:math:`\I31GET\K{\_}\sx`
........................




The :ref:`instruction <syntax-instr>` :math:`({\I31GET}{\mathsf{\_}}{{\sx}})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`(\REF~\NULL~\I31)~\rightarrow~\I32`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr {\I31GET}{\mathsf{\_}}{{\sx}} : (\REF~\NULL~\I31) \rightarrow \I32
   }
   \qquad
   \end{array}



.. index:: external reference

External Reference Instructions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. _valid-any.convert_extern:

:math:`\ANYCONVERTEXTERN`
.........................




The :ref:`instruction <syntax-instr>` :math:`\ANYCONVERTEXTERN` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`(\REF~{{\NULL}_1^?}~\EXTERN)~\rightarrow~(\REF~{{\NULL}_2^?}~\ANY)` if:


   * :math:`{{\NULL}_1^?}` is of the form :math:`{{\NULL}_2^?}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   {{\NULL}_1^?} = {{\NULL}_2^?}
   }{
   C \vdashinstr \ANYCONVERTEXTERN : (\REF~{{\NULL}_1^?}~\EXTERN) \rightarrow (\REF~{{\NULL}_2^?}~\ANY)
   }
   \qquad
   \end{array}


.. _valid-extern.convert_any:

:math:`\EXTERNCONVERTANY`
.........................




The :ref:`instruction <syntax-instr>` :math:`\EXTERNCONVERTANY` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`(\REF~{{\NULL}_1^?}~\ANY)~\rightarrow~(\REF~{{\NULL}_2^?}~\EXTERN)` if:


   * :math:`{{\NULL}_1^?}` is of the form :math:`{{\NULL}_2^?}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   {{\NULL}_1^?} = {{\NULL}_2^?}
   }{
   C \vdashinstr \EXTERNCONVERTANY : (\REF~{{\NULL}_1^?}~\ANY) \rightarrow (\REF~{{\NULL}_2^?}~\EXTERN)
   }
   \qquad
   \end{array}


.. index:: numeric instruction
   pair: validation; instruction
   single: abstract syntax; instruction
.. _valid-instr-numeric:

Numeric Instructions
~~~~~~~~~~~~~~~~~~~~

.. _valid-const:

:math:`t\K{.}\CONST~c`
......................




The :ref:`instruction <syntax-instr>` :math:`({\mathit{nt}}{.}\CONST~c_{\mathit{nt}})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\epsilon~\rightarrow~{\mathit{nt}}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr {\mathit{nt}}{.}\CONST~c_{\mathit{nt}} : \epsilon \rightarrow {\mathit{nt}}
   }
   \qquad
   \end{array}


.. _valid-unop:

:math:`t\K{.}\unop`
...................




The :ref:`instruction <syntax-instr>` :math:`({\mathit{nt}} {.} {\unop}_{\mathit{nt}})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{nt}}~\rightarrow~{\mathit{nt}}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr {\mathit{nt}} {.} {\unop}_{\mathit{nt}} : {\mathit{nt}} \rightarrow {\mathit{nt}}
   }
   \qquad
   \end{array}


.. _valid-binop:

:math:`t\K{.}\binop`
....................




The :ref:`instruction <syntax-instr>` :math:`({\mathit{nt}} {.} {\binop}_{\mathit{nt}})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{nt}}~{\mathit{nt}}~\rightarrow~{\mathit{nt}}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr {\mathit{nt}} {.} {\binop}_{\mathit{nt}} : {\mathit{nt}}~{\mathit{nt}} \rightarrow {\mathit{nt}}
   }
   \qquad
   \end{array}


.. _valid-testop:

:math:`t\K{.}\testop`
.....................




The :ref:`instruction <syntax-instr>` :math:`({\mathit{nt}} {.} {\testop}_{\mathit{nt}})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{nt}}~\rightarrow~\I32`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr {\mathit{nt}} {.} {\testop}_{\mathit{nt}} : {\mathit{nt}} \rightarrow \I32
   }
   \qquad
   \end{array}


.. _valid-relop:

:math:`t\K{.}\relop`
....................




The :ref:`instruction <syntax-instr>` :math:`({\mathit{nt}} {.} {\relop}_{\mathit{nt}})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{nt}}~{\mathit{nt}}~\rightarrow~\I32`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr {\mathit{nt}} {.} {\relop}_{\mathit{nt}} : {\mathit{nt}}~{\mathit{nt}} \rightarrow \I32
   }
   \qquad
   \end{array}


.. _valid-cvtop:

:math:`t_1\K{.}\cvtop\K{\_}t_2\K{\_}\sx^?`
..........................................




The :ref:`instruction <syntax-instr>` :math:`({\mathit{nt}}_1 {.} {{\cvtop}}{\mathsf{\_}}{{\mathit{nt}}_2})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{nt}}_2~\rightarrow~{\mathit{nt}}_1`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr {\mathit{nt}}_1 {.} {{\cvtop}}{\mathsf{\_}}{{\mathit{nt}}_2} : {\mathit{nt}}_2 \rightarrow {\mathit{nt}}_1
   }
   \qquad
   \end{array}


.. index:: vector instruction
   pair: validation; instruction
   single: abstract syntax; instruction

.. _valid-instr-vec:
.. _aux-unpackshape:

Vector Instructions
~~~~~~~~~~~~~~~~~~~
Vector instructions can have a prefix to describe the :ref:`shape <syntax-shape>` of the operand. Packed numeric types, :math:`\I8` and :math:`\I16`, are not :ref:`value types <syntax-valtype>`. An auxiliary function maps such packed type shapes to value types:

.. math::
   \begin{array}[t]{@{}lcl@{}l@{}}
   {\unpack}({{\ntI}{{\ntN}}}{\Xshape}{M}) & = & {\unpack}({\ntI}{{\ntN}}) \\
   \end{array}


.. _valid-vconst:

:math:`\V128\K{.}\VCONST~c`
...........................




The :ref:`instruction <syntax-instr>` :math:`(\V128{.}\CONST~c)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\epsilon~\rightarrow~\V128`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr \V128{.}\CONST~c : \epsilon \rightarrow \V128
   }
   \qquad
   \end{array}


.. _valid-vvunop:

:math:`\V128\K{.}\vvunop`
.........................




The :ref:`instruction <syntax-instr>` :math:`(\V128 {.} {\vvunop})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\V128~\rightarrow~\V128`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr \V128 {.} {\vvunop} : \V128 \rightarrow \V128
   }
   \qquad
   \end{array}


.. _valid-vvbinop:

:math:`\V128\K{.}\vvbinop`
..........................




The :ref:`instruction <syntax-instr>` :math:`(\V128 {.} {\vvbinop})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\V128~\V128~\rightarrow~\V128`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr \V128 {.} {\vvbinop} : \V128~\V128 \rightarrow \V128
   }
   \qquad
   \end{array}


.. _valid-vvternop:

:math:`\V128\K{.}\vvternop`
...........................




The :ref:`instruction <syntax-instr>` :math:`(\V128 {.} {\vvternop})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\V128~\V128~\V128~\rightarrow~\V128`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr \V128 {.} {\vvternop} : \V128~\V128~\V128 \rightarrow \V128
   }
   \qquad
   \end{array}


.. _valid-vvtestop:

:math:`\V128\K{.}\vvtestop`
...........................




The :ref:`instruction <syntax-instr>` :math:`(\V128 {.} {\vvtestop})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\V128~\rightarrow~\I32`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr \V128 {.} {\vvtestop} : \V128 \rightarrow \I32
   }
   \qquad
   \end{array}


.. _valid-vunop:

:math:`\shape\K{.}\vunop`
.........................




The :ref:`instruction <syntax-instr>` :math:`({\mathit{sh}} {.} {\vunop})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\V128~\rightarrow~\V128`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr {\mathit{sh}} {.} {\vunop} : \V128 \rightarrow \V128
   }
   \qquad
   \end{array}


.. _valid-vbinop:

:math:`\shape\K{.}\vbinop`
..........................




The :ref:`instruction <syntax-instr>` :math:`({\mathit{sh}} {.} {\vbinop})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\V128~\V128~\rightarrow~\V128`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr {\mathit{sh}} {.} {\vbinop} : \V128~\V128 \rightarrow \V128
   }
   \qquad
   \end{array}


.. _valid-vternop:

:math:`\shape\K{.}\vternop`
...........................




The :ref:`instruction <syntax-instr>` :math:`({\mathit{sh}} {.} {\vternop})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\V128~\V128~\V128~\rightarrow~\V128`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr {\mathit{sh}} {.} {\vternop} : \V128~\V128~\V128 \rightarrow \V128
   }
   \qquad
   \end{array}


.. _valid-vtestop:

:math:`\shape\K{.}\vtestop`
...........................




The :ref:`instruction <syntax-instr>` :math:`({\mathit{sh}} {.} {\vtestop})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\V128~\rightarrow~\I32`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr {\mathit{sh}} {.} {\vtestop} : \V128 \rightarrow \I32
   }
   \qquad
   \end{array}


.. _valid-vrelop:

:math:`\shape\K{.}\vrelop`
..........................




The :ref:`instruction <syntax-instr>` :math:`({\mathit{sh}} {.} {\vrelop})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\V128~\V128~\rightarrow~\V128`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr {\mathit{sh}} {.} {\vrelop} : \V128~\V128 \rightarrow \V128
   }
   \qquad
   \end{array}


.. _valid-vshiftop:

:math:`\ishape\K{.}\vishiftop`
..............................




The :ref:`instruction <syntax-instr>` :math:`({\mathit{sh}} {.} {\vshiftop})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\V128~\I32~\rightarrow~\V128`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr {\mathit{sh}} {.} {\vshiftop} : \V128~\I32 \rightarrow \V128
   }
   \qquad
   \end{array}


.. _valid-vbitmask:

:math:`\ishape\K{.}\VBITMASK`
.............................




The :ref:`instruction <syntax-instr>` :math:`({\mathit{sh}}{.}\VBITMASK)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\V128~\rightarrow~\I32`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr {\mathit{sh}}{.}\VBITMASK : \V128 \rightarrow \I32
   }
   \qquad
   \end{array}


.. _valid-vswizzlop:

:math:`\K{i8x16.}\vswizzlop`
............................




The :ref:`instruction <syntax-instr>` :math:`({\mathit{sh}} {.} {\vswizzlop})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\V128~\V128~\rightarrow~\V128`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr {\mathit{sh}} {.} {\vswizzlop} : \V128~\V128 \rightarrow \V128
   }
   \qquad
   \end{array}


.. _valid-vshuffle:

:math:`\K{i8x16.}\VSHUFFLE~\laneidx^{16}`
.........................................




The :ref:`instruction <syntax-instr>` :math:`({\mathit{sh}}{.}\VSHUFFLE~{i^\ast})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\V128~\V128~\rightarrow~\V128` if:


   * For all :math:`i` in :math:`{i^\ast}`:

      * The :ref:`lane index <syntax-laneidx>` :math:`i` is less than :math:`2 \cdot {\shdim}({\mathit{sh}})`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   (i < 2 \cdot {\shdim}({\mathit{sh}}))^\ast
   }{
   C \vdashinstr {\mathit{sh}}{.}\VSHUFFLE~{i^\ast} : \V128~\V128 \rightarrow \V128
   }
   \qquad
   \end{array}


.. _valid-vsplat:

:math:`\shape\K{.}\VSPLAT`
..........................




The :ref:`instruction <syntax-instr>` :math:`({\mathit{sh}}{.}\VSPLAT)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\numtype}~\rightarrow~\V128` if:


   * The :ref:`number type <syntax-numtype>` :math:`{\numtype}` is :math:`{\unpack}({\mathit{sh}})`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr {\mathit{sh}}{.}\VSPLAT : {\unpack}({\mathit{sh}}) \rightarrow \V128
   }
   \qquad
   \end{array}


.. _valid-vextract_lane:

:math:`\shape\K{.}\VEXTRACTLANE\K{\_}\sx^?~\laneidx`
....................................................




The :ref:`instruction <syntax-instr>` :math:`({{\mathit{sh}}{.}\VEXTRACTLANE}{\mathsf{\_}}{{{\sx}^?}}~i)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\V128~\rightarrow~{\numtype}` if:


   * The :ref:`lane index <syntax-laneidx>` :math:`i` is less than :math:`{\shdim}({\mathit{sh}})`.

   * The :ref:`number type <syntax-numtype>` :math:`{\numtype}` is :math:`{\unpack}({\mathit{sh}})`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   i < {\shdim}({\mathit{sh}})
   }{
   C \vdashinstr {{\mathit{sh}}{.}\VEXTRACTLANE}{\mathsf{\_}}{{{\sx}^?}}~i : \V128 \rightarrow {\unpack}({\mathit{sh}})
   }
   \qquad
   \end{array}


.. _valid-vreplace_lane:

:math:`\shape\K{.}\VREPLACELANE~\laneidx`
.........................................




The :ref:`instruction <syntax-instr>` :math:`({\mathit{sh}}{.}\VREPLACELANE~i)` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\V128~{\numtype}~\rightarrow~\V128` if:


   * The :ref:`lane index <syntax-laneidx>` :math:`i` is less than :math:`{\shdim}({\mathit{sh}})`.

   * The :ref:`number type <syntax-numtype>` :math:`{\numtype}` is :math:`{\unpack}({\mathit{sh}})`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   i < {\shdim}({\mathit{sh}})
   }{
   C \vdashinstr {\mathit{sh}}{.}\VREPLACELANE~i : \V128~{\unpack}({\mathit{sh}}) \rightarrow \V128
   }
   \qquad
   \end{array}


.. _valid-vextunop:

:math:`\ishape_1\K{.}\vextunop\K{\_}\ishape_2`
..............................................




The :ref:`instruction <syntax-instr>` :math:`({\mathit{sh}}_1 {.} {{\vextunop}}{\mathsf{\_}}{{\mathit{sh}}_2})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\V128~\rightarrow~\V128`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr {\mathit{sh}}_1 {.} {{\vextunop}}{\mathsf{\_}}{{\mathit{sh}}_2} : \V128 \rightarrow \V128
   }
   \qquad
   \end{array}


.. _valid-vextbinop:

:math:`\ishape_1\K{.}\vextbinop\K{\_}\ishape_2`
...............................................




The :ref:`instruction <syntax-instr>` :math:`({\mathit{sh}}_1 {.} {{\vextbinop}}{\mathsf{\_}}{{\mathit{sh}}_2})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\V128~\V128~\rightarrow~\V128`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr {\mathit{sh}}_1 {.} {{\vextbinop}}{\mathsf{\_}}{{\mathit{sh}}_2} : \V128~\V128 \rightarrow \V128
   }
   \qquad
   \end{array}


.. _valid-vextternop:

:math:`\ishape_1\K{.}\vextternop\K{\_}\ishape_2`
................................................




The :ref:`instruction <syntax-instr>` :math:`({\mathit{sh}}_1 {.} {{\vextternop}}{\mathsf{\_}}{{\mathit{sh}}_2})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\V128~\V128~\V128~\rightarrow~\V128`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr {\mathit{sh}}_1 {.} {{\vextternop}}{\mathsf{\_}}{{\mathit{sh}}_2} : \V128~\V128~\V128 \rightarrow \V128
   }
   \qquad
   \end{array}


.. _valid-vnarrow:

:math:`\ishape_1\K{.}\VNARROW\K{\_}\ishape_2\K{\_}\sx`
......................................................




The :ref:`instruction <syntax-instr>` :math:`({{\mathit{sh}}_1{.}\VNARROW}{\mathsf{\_}}{{\mathit{sh}}_2}{\mathsf{\_}}{{\sx}})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\V128~\V128~\rightarrow~\V128`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr {{\mathit{sh}}_1{.}\VNARROW}{\mathsf{\_}}{{\mathit{sh}}_2}{\mathsf{\_}}{{\sx}} : \V128~\V128 \rightarrow \V128
   }
   \qquad
   \end{array}


.. _valid-vcvtop:

:math:`\shape\K{.}\vcvtop\K{\_}\half^?\K{\_}\shape\K{\_}\sx^?\K{\_zero}^?`
..........................................................................




The :ref:`instruction <syntax-instr>` :math:`({\mathit{sh}}_1 {.} {{\vcvtop}}{\mathsf{\_}}{{\mathit{sh}}_2})` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\V128~\rightarrow~\V128`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstr {\mathit{sh}}_1 {.} {{\vcvtop}}{\mathsf{\_}}{{\mathit{sh}}_2} : \V128 \rightarrow \V128
   }
   \qquad
   \end{array}


.. index:: instruction, instruction sequence, local type
.. _valid-instrs:

Instruction Sequences
~~~~~~~~~~~~~~~~~~~~~

Typing of instruction sequences is defined recursively.




The instruction sequence :math:`{{\instr}^\ast}` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{it}}` if:


   * Either:

      * The instruction sequence :math:`{{\instr}^\ast}` is empty.

      * The :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{it}}` is of the form :math:`\epsilon~\rightarrow~\epsilon`.

   * Or:

      * The instruction sequence :math:`{{\instr}^\ast}` is of the form :math:`{\instr'}`.

      * The :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{it}}` is of the form :math:`{t_1^\ast}~{\to}_{{x^\ast}}\,{t_2^\ast}`.

      * The :ref:`instruction <syntax-instr>` :math:`{\instr'}` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~{\to}_{{x^\ast}}\,{t_2^\ast}`.
   * Or:

      * The instruction sequence :math:`{{\instr}^\ast}` is of the form :math:`{{\instr}_1^\ast}~{{\instr}_2^\ast}`.

      * The :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{it}}` is of the form :math:`{t_1^\ast}~{\to}_{{x_1^\ast}~{x_2^\ast}}\,{t_3^\ast}`.

      * The instruction sequence :math:`{{\instr}_1^\ast}` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~{\to}_{{x_1^\ast}}\,{t_2^\ast}`.

      * For all :math:`x_1` in :math:`{x_1^\ast}`:

         * The :ref:`local <syntax-localtype>` :math:`C{.}\CLOCALS{}[x_1]` exists.

         * The :ref:`local <syntax-localtype>` :math:`C{.}\CLOCALS{}[x_1]` is of the form :math:`({\init}~t)`.

      * Under the context :math:`C` with the local types of :math:`{x_1^\ast}` updated to :math:`{(\SET~t)^\ast}`, the instruction sequence :math:`{{\instr}_2^\ast}` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t_2^\ast}~{\to}_{{x_2^\ast}}\,{t_3^\ast}`.
   * Or:

      * The instruction sequence :math:`{{\instr}^\ast}` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{it}''}`.

      * The :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{it}''}` :ref:`matches <match-instrtype>` the :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{it}}`.

      * The :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{it}}` is :ref:`valid <valid-instrtype>`.
   * Or:

      * The :ref:`instruction type <syntax-instrtype>` :math:`{\mathit{it}}` is of the form :math:`{t^\ast}~{t_1^\ast}~{\to}_{{x^\ast}}\,{t^\ast}~{t_2^\ast}`.

      * The instruction sequence :math:`{{\instr}^\ast}` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~{\to}_{{x^\ast}}\,{t_2^\ast}`.

      * The :ref:`result type <syntax-resulttype>` :math:`{t^\ast}` is :ref:`valid <valid-resulttype>`.




.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstrs \epsilon : \epsilon \rightarrow \epsilon
   }
   \qquad
   \end{array}

.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashinstr {\instr} : {t_1^\ast} \to_{{x^\ast}} {t_2^\ast}
   }{
   C \vdashinstrs {\instr} : {t_1^\ast} \to_{{x^\ast}} {t_2^\ast}
   }
   \qquad
   \end{array}

.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashinstrs {{\instr}_1^\ast} : {t_1^\ast} \to_{{x_1^\ast}} {t_2^\ast}
    \qquad
   (C{.}\CLOCALS{}[x_1] = {\init}~t)^\ast
    \qquad
   C{}[{.}\LOCAL{}[{x_1^\ast}] = {(\SET~t)^\ast}] \vdashinstrs {{\instr}_2^\ast} : {t_2^\ast} \to_{{x_2^\ast}} {t_3^\ast}
   }{
   C \vdashinstrs {{\instr}_1^\ast}~{{\instr}_2^\ast} : {t_1^\ast} \to_{{x_1^\ast}~{x_2^\ast}} {t_3^\ast}
   }
   \qquad
   \end{array}

.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashinstrs {{\instr}^\ast} : {\mathit{it}}
    \qquad
   C \vdashinstrtypematch {\mathit{it}} \subinstrtypematch {\mathit{it}'}
    \qquad
   C \vdashinstrtype {\mathit{it}'} : \OKinstrtype
   }{
   C \vdashinstrs {{\instr}^\ast} : {\mathit{it}'}
   }
   \qquad
   \end{array}

.. note::
   This *subsumption rule* allows to weaken the type of an instruction sequence to a supertype,
   which includes the ability to drop init variables :math:`{x^\ast}` from the instruction type in a context where they are not needed, for example, at the end of the body of a :ref:`block <valid-block>`.

.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashinstrs {{\instr}^\ast} : {t_1^\ast} \to_{{x^\ast}} {t_2^\ast}
    \qquad
   C \vdashresulttype {t^\ast} : \OKresulttype
   }{
   C \vdashinstrs {{\instr}^\ast} : ({t^\ast}~{t_1^\ast}) \to_{{x^\ast}} ({t^\ast}~{t_2^\ast})
   }
   \qquad
   \end{array}

.. note::
   In combination with the previous two rules,
   this *frame rule* allows to compose instructions whose types would not directly fit otherwise.
   For example, consider the instruction sequence

   .. math::
      (\I32{.}\CONST~1)~(\I32{.}\CONST~2)~(\I32 {.} \ADD)

   To type this sequence, its subsequence :math:`(\I32{.}\CONST~2)~(\I32 {.} \ADD)` needs to be valid with an intermediate type.
   But the direct type of :math:`(\I32{.}\CONST~2)` is :math:`\epsilon \rightarrow \I32`, not matching the two inputs expected by :math:`\I32 {.} \ADD`.
   The rule allows to weaken the type of :math:`(\I32{.}\CONST~2)` to the supertype :math:`\I32 \rightarrow \I32~\I32`, such that it can be composed with :math:`\I32 {.} \ADD` and yields the intermediate type :math:`\I32 \rightarrow \I32~\I32` for the subsequence. That can in turn be composed with the first constant.


.. index:: expression, result type
   pair: validation; expression
   single: abstract syntax; expression
   single: expression; constant
.. _valid-expr:

Expressions
~~~~~~~~~~~

Expressions :math:`{\expr}` are classified by :ref:`result types <syntax-resulttype>` :math:`{t^\ast}`.




The :ref:`expression <syntax-expr>` :math:`{{\instr}^\ast}` is :ref:`valid <valid-expr>` with the :ref:`result type <syntax-resulttype>` :math:`{t^\ast}` if:


   * The instruction sequence :math:`{{\instr}^\ast}` is :ref:`valid <valid-instr>` with the :ref:`instruction type <syntax-instrtype>` :math:`\epsilon~\rightarrow~{t^\ast}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashinstrs {{\instr}^\ast} : \epsilon \rightarrow {t^\ast}
   }{
   C \vdashexpr {{\instr}^\ast} : {t^\ast}
   }
   \qquad
   \end{array}


.. index:: ! constant
.. _valid-constant:

Constant Expressions
....................

In a *constant* expression, all instructions must be constant.




:math:`{{\instr}^\ast}` is constant if:


   * For all :math:`{\instr}` in :math:`{{\instr}^\ast}`:

      * :math:`{\instr}` is constant.






:math:`{\instr}` is constant if:


   * Either:

      * The :ref:`instruction <syntax-instr>` :math:`{\instr}` is of the form :math:`({\mathit{nt}}{.}\CONST~c_{\mathit{nt}})`.

   * Or:

      * The :ref:`instruction <syntax-instr>` :math:`{\instr}` is of the form :math:`({\mathit{vt}}{.}\CONST~c_{\mathit{vt}})`.
   * Or:

      * The :ref:`instruction <syntax-instr>` :math:`{\instr}` is of the form :math:`(\REFNULL~{\mathit{ht}})`.
   * Or:

      * The :ref:`instruction <syntax-instr>` :math:`{\instr}` is of the form :math:`\REFI31`.
   * Or:

      * The :ref:`instruction <syntax-instr>` :math:`{\instr}` is of the form :math:`(\REFFUNC~x)`.
   * Or:

      * The :ref:`instruction <syntax-instr>` :math:`{\instr}` is of the form :math:`(\STRUCTNEW~x)`.
   * Or:

      * The :ref:`instruction <syntax-instr>` :math:`{\instr}` is of the form :math:`(\STRUCTNEWDEFAULT~x)`.
   * Or:

      * The :ref:`instruction <syntax-instr>` :math:`{\instr}` is of the form :math:`(\ARRAYNEW~x)`.
   * Or:

      * The :ref:`instruction <syntax-instr>` :math:`{\instr}` is of the form :math:`(\ARRAYNEWDEFAULT~x)`.
   * Or:

      * The :ref:`instruction <syntax-instr>` :math:`{\instr}` is of the form :math:`(\ARRAYNEWFIXED~x~n)`.
   * Or:

      * The :ref:`instruction <syntax-instr>` :math:`{\instr}` is of the form :math:`\ANYCONVERTEXTERN`.
   * Or:

      * The :ref:`instruction <syntax-instr>` :math:`{\instr}` is of the form :math:`\EXTERNCONVERTANY`.
   * Or:

      * The :ref:`instruction <syntax-instr>` :math:`{\instr}` is of the form :math:`(\GLOBALGET~x)`.

      * The :ref:`global <syntax-globaltype>` :math:`C{.}\CGLOBALS{}[x]` exists.

      * The :ref:`global <syntax-globaltype>` :math:`C{.}\CGLOBALS{}[x]` is of the form :math:`(\epsilon~t)`.
   * Or:

      * The :ref:`instruction <syntax-instr>` :math:`{\instr}` is of the form :math:`({\ntI}{{\ntN}} {.} {\binop})`.

      * :math:`{\ntI}{{\ntN}}` is contained in [:math:`\I32`; :math:`\I64`].

      * :math:`{\binop}` is contained in [:math:`\ADD`; :math:`\SUB`; :math:`\MUL`].




.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   (C \vdashinstrconst {\instr}~\CONSTinstrconst)^\ast
   }{
   C \vdashexprconst {{\instr}^\ast}~\CONSTexprconst
   }
   \qquad
   \end{array}

.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashinstrconst ({\mathit{nt}}{.}\CONST~c_{\mathit{nt}})~\CONSTinstrconst
   }
   \qquad
   \frac{
   }{
   C \vdashinstrconst ({\mathit{vt}}{.}\CONST~c_{\mathit{vt}})~\CONSTinstrconst
   }
   \qquad
   \frac{
   {\ntI}{{\ntN}} \in \I32~\I64
    \qquad
   {\binop} \in \ADD~\SUB~\MUL
   }{
   C \vdashinstrconst ({\ntI}{{\ntN}} {.} {\binop})~\CONSTinstrconst
   }
   \\[3ex]\displaystyle
   \frac{
   }{
   C \vdashinstrconst (\REFNULL~{\mathit{ht}})~\CONSTinstrconst
   }
   \qquad
   \frac{
   }{
   C \vdashinstrconst (\REFI31)~\CONSTinstrconst
   }
   \qquad
   \frac{
   }{
   C \vdashinstrconst (\REFFUNC~x)~\CONSTinstrconst
   }
   \\[3ex]\displaystyle
   \frac{
   }{
   C \vdashinstrconst (\STRUCTNEW~x)~\CONSTinstrconst
   }
   \qquad
   \frac{
   }{
   C \vdashinstrconst (\STRUCTNEWDEFAULT~x)~\CONSTinstrconst
   }
   \\[3ex]\displaystyle
   \frac{
   }{
   C \vdashinstrconst (\ARRAYNEW~x)~\CONSTinstrconst
   }
   \qquad
   \frac{
   }{
   C \vdashinstrconst (\ARRAYNEWDEFAULT~x)~\CONSTinstrconst
   }
   \qquad
   \frac{
   }{
   C \vdashinstrconst (\ARRAYNEWFIXED~x~n)~\CONSTinstrconst
   }
   \\[3ex]\displaystyle
   \frac{
   }{
   C \vdashinstrconst (\ANYCONVERTEXTERN)~\CONSTinstrconst
   }
   \qquad
   \frac{
   }{
   C \vdashinstrconst (\EXTERNCONVERTANY)~\CONSTinstrconst
   }
   \\[3ex]\displaystyle
   \frac{
   C{.}\CGLOBALS{}[x] = t
   }{
   C \vdashinstrconst (\GLOBALGET~x)~\CONSTinstrconst
   }
   \qquad
   \end{array}

.. note::
   Currently, constant expressions occurring in :ref:`globals <syntax-global>` are further constrained in that contained :math:`\mathsf{global{.}get}` instructions are only allowed to refer to *imported* or *previously defined* globals. Constant expressions occurring in :ref:`tables <syntax-table>` may only have :math:`\mathsf{global{.}get}` instructions that refer to *imported* globals.
   This is enforced in the :ref:`validation rule for modules <valid-module>` by constraining the context :math:`C` accordingly.

   The definition of constant expression may be extended in future versions of WebAssembly.
