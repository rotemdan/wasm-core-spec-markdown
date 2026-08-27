.. index:: ! abstract syntax

Conventions
-----------

WebAssembly is a programming language that has multiple concrete representations
(its :ref:`binary format <binary>` and the :ref:`text format <text>`).
Both map to a common structure.
For conciseness, this structure is described in the form of an *abstract syntax*.
All parts of this specification are defined in terms of this abstract syntax.


.. index:: ! grammar notation, notation
   single: abstract syntax; grammar
   pair: abstract syntax; notation
.. _grammar:

Grammar Notation
~~~~~~~~~~~~~~~~

The following conventions are adopted in defining grammar rules for abstract syntax.

* Terminal symbols (atoms) are written in sans-serif font or in symbolic form: :math:`\mathsf{i{\scriptstyle 32}}`, :math:`\mathsf{nop}`, :math:`\rightarrow`, :math:`{}[ , ]`.

* Nonterminal symbols are written in italic font: :math:`{\valtype}`, :math:`{\instr}`.

* :math:`{A^{n}}` is a sequence of :math:`n \geq 0` iterations of :math:`A`.

* :math:`{A^\ast}` is a possibly empty sequence of iterations of :math:`A`.
  (This is a shorthand for :math:`{A^{n}}` used where :math:`n` is not relevant.)

* :math:`{A^{+}}`` is a non-empty sequence of iterations of :math:`A`.
  (This is a shorthand for :math:`{A^{n}}` where :math:`n \geq 1`.)

* :math:`{A^?}`` is an optional occurrence of :math:`A`.
  (This is a shorthand for :math:`{A^{n}}` where :math:`n \leq 1`.)

* Productions are written :math:`\begin{array}[t]{@{}l@{}r@{~}r@{~}l@{}l@{}} & {\mathit{sym}} & ::= & A_1 ~~|~~ \ldots ~~|~~ A_n \\ \end{array}`.

* Large productions may be split into multiple definitions, indicated by ending the first one with explicit ellipses, :math:`\begin{array}[t]{@{}l@{}r@{~}r@{~}l@{}l@{}} & {\mathit{sym}} & ::= & A_1 ~~|~~ \dots \\ \end{array}`, and starting continuations with ellipses, :math:`\begin{array}[t]{@{}l@{}r@{~}r@{~}l@{}l@{}} & {\mathit{sym}} & ::= & \dots ~~|~~ A_2 \\ \end{array}`.

* Some productions are augmented with side conditions, ":math:`\iff \X{condition}`", that provide a shorthand for a combinatorial expansion of the production into many separate cases.

* If the same meta variable or non-terminal symbol appears multiple times in a production, then all those occurrences must have the same instantiation.
  (This is a shorthand for a side condition requiring multiple different variables to be equal.)


.. _notation-epsilon:
.. _notation-length:
.. _notation-index:
.. _notation-slice:
.. _notation-replace:
.. _notation-record:
.. _notation-project:
.. _notation-concat:
.. _notation-compose:

Auxiliary Notation
~~~~~~~~~~~~~~~~~~



When dealing with syntactic constructs the following notation is also used:

* :math:`\epsilon` denotes the empty sequence.

* :math:`{|s|}` denotes the length of a sequence :math:`s`.

* :math:`s{}[i]` denotes the :math:`i`-th element of a sequence :math:`s``, starting from :math:`0`.

* :math:`s{}[i : n]` denotes the sub-sequence :math:`s{}[i] \ldots s{}[i + n - 1]` of a sequence :math:`s`.

* :math:`s{}[{}[i] = A]` denotes the same sequence as :math:`s`,
  except that the :math:`i`-th element is replaced with :math:`A`.

* :math:`s{}[{}[i : n] = {A^{n}}]` denotes the same sequence as :math:`s`,
  except that the sub-sequence :math:`s{}[i : n]` is replaced with :math:`{A^{n}}`.

* :math:`s_1 \oplus s_2` denotes the sequence :math:`s_1` concatenated with :math:`s_2`;
  this is equivalent to :math:`s_1~s_2`, but used for clarity.

* :math:`{\bigoplus}\, {s^\ast}` denotes the flattened sequence, formed by concatenating all sequences :math:`s_i` in :math:`{s^\ast}`.

* :math:`A \in s` denotes that :math:`A` is a member of the sequence :math:`s`, that is, :math:`s` is of the form :math:`s_1~A~s_2` for some sequences :math:`s_1`, :math:`s_2`.

Moreover, the following conventions are employed:

* The notation :math:`{x^{n}}`, where :math:`x` is a non-terminal symbol, is treated as a meta variable ranging over respective sequences of :math:`x` (similarly for :math:`{x^\ast}`, :math:`{x^{+}}`, :math:`{x^?}`).

* When given a sequence :math:`{x^{n}}`,
  then the occurrences of :math:`x` in an iterated sequence :math:`{( \ldots x \ldots )^{n}}` are assumed to denote the individual elements of :math:`{x^{n}}`, respectively
  (similarly for :math:`{x^\ast}`, :math:`{x^{+}}`, :math:`{x^?}`).
  This implicitly expresses a form of mapping syntactic constructions over a sequence.

* :math:`{e^{i<n}}` denotes the same sequence as :math:`{e^{n}}`,
  but implicitly also defines :math:`{i^{n}}` to be the sequence of values :math:`0` to :math:`(n - 1)`.

.. note::
   For example, if :math:`{x^{n}}` is the sequence :math:`a~b~c`, then :math:`{({\mathrm{f}}(x) + 1)^{n}}` denotes the sequence :math:`({\mathrm{f}}(a) + 1)~({\mathrm{f}}(b) + 1)~({\mathrm{f}}(c) + 1)`.

   The form :math:`{e^{i<n}}` additionally gives access to an index variable inside the iteration.
   For example, :math:`{({\mathrm{f}}(x) + i)^{i<n}}` denotes the sequence :math:`({\mathrm{f}}(a) + 0)~({\mathrm{f}}(b) + 1)~({\mathrm{f}}(c) + 2)`.

Productions of the following form are interpreted as *records* that map a fixed set of fields :math:`{\mathsf{field}}_{i}` to "values" :math:`A_i`, respectively:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & r & ::= & \{ \begin{array}[t]{@{}l@{}l@{}}
   {\mathsf{field}}_{1}~A_1 ,  {\mathsf{field}}_{2}~A_2 ,  \ldots~ \} \\
   \end{array} \\
   \end{array}


The following notation is adopted for manipulating such records:

* Where the type of a record is clear from context, empty fields with value :math:`\epsilon` are often omitted.

* :math:`r{.}\mathsf{field}` denotes the contents of the :math:`\mathsf{field}` component of :math:`r`.

* :math:`r{}[{.}\mathsf{field} = A]` denotes the same record as :math:`r`,
  except that the value of the :math:`\mathsf{field}` component is replaced with :math:`A`.

* :math:`r{}[{.}\mathsf{field} \mathrel{{=}{\oplus}} {A^\ast}]` denotes the same record as :math:`r`,
  except that :math:`{A^\ast}` is appended to the sequence value of the :math:`\mathsf{field}` component,
  that is, it is short for :math:`r{}[{.}\mathsf{field} = r{.}\mathsf{field} \oplus {A^\ast}]`.

* :math:`r_1 \oplus r_2` denotes the composition of two identically shaped records by concatenating each field of sequences point-wise:

  .. math::
     \{ {\mathsf{field}}_{1}\,{A_1^\ast}, {\mathsf{field}}_{2}\,{A_2^\ast}, \ldots \} \oplus \{ {\mathsf{field}}_{1}\,{B_1^\ast}, {\mathsf{field}}_{2}\,{B_2^\ast}, \ldots \} = \{ {\mathsf{field}}_{1}\,({A_1^\ast} \oplus {B_1^\ast}), {\mathsf{field}}_{2}\,({A_2^\ast} \oplus {B_2^\ast}), \ldots \}

* :math:`{\bigoplus}\, {r^\ast}` denotes the composition of a sequence of records, respectively; if the sequence is empty, then all fields of the resulting record are empty.

The update notation for sequences and records generalizes recursively to nested components accessed by "paths" :math:`\begin{array}[t]{@{}l@{}r@{~}r@{~}l@{}l@{}} & {\mathit{pth}} & ::= & {({}[ i ]~\mid~{.}\mathsf{field})^{+}} \\ \end{array}`:

* :math:`s{}[{{}[ i ]}{{\mathit{pth}}} = A]` is short for :math:`s{}[{}[i] = s{}[i]{}[{\mathit{pth}} = A]]`,

* :math:`r{}[{.}\mathsf{field}~{\mathit{pth}} = A]` is short for :math:`r{}[{.}\mathsf{field} = r{.}\mathsf{field}{}[{\mathit{pth}} = A]]`.



.. index:: ! list
   pair: abstract syntax; list
.. _syntax-list:

Lists
~~~~~

*Lists* are bounded sequences of the form :math:`{A^{n}}` (or :math:`{A^\ast}`),
where the :math:`A` can either be values or complex constructions.
A list can have at most :math:`{2^{32}} - 1` elements.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\list}(X) & ::= & {X^\ast} & \quad \mbox{if}~ {|{X^\ast}|} < {2^{32}} \\
   \end{array}
