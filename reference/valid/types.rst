Types
-----

Simple :ref:`types <syntax-type>`, such as :ref:`number types <syntax-numtype>` are universally valid.
However, restrictions apply to most other types, such as :ref:`reference types <syntax-reftype>`, :ref:`function types <syntax-functype>`, as well as the :ref:`limits <syntax-limits>` of :ref:`table types <syntax-tabletype>` and :ref:`memory types <syntax-memtype>`, which must be checked during validation.

Moreover, :ref:`block types <syntax-blocktype>` are converted to :ref:`instruction types <syntax-instrtype>` for ease of processing.


.. index:: number type
   pair: validation; number type
   single: abstract syntax; number type
.. _valid-numtype:

Number Types
~~~~~~~~~~~~




The :ref:`number type <syntax-numtype>` :math:`{\numtype}` is always :ref:`valid <valid-numtype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashnumtype {\numtype} : \OKnumtype
   }
   \qquad
   \end{array}


.. index:: vector type
   pair: validation; vector type
   single: abstract syntax; vector type
.. _valid-vectype:

Vector Types
~~~~~~~~~~~~




The :ref:`vector type <syntax-vectype>` :math:`{\vectype}` is always :ref:`valid <valid-vectype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashvectype {\vectype} : \OKvectype
   }
   \qquad
   \end{array}


.. index:: type index, type use
   pair: validation; type use
   single: abstract syntax; type use
.. _valid-typeuse:

Type Uses
~~~~~~~~~




The :ref:`type use <syntax-typeuse>` :math:`{\typeidx}` is :ref:`valid <valid-typeuse>` if:


   * The :ref:`type <syntax-deftype>` :math:`C{.}\CTYPES{}[{\typeidx}]` exists.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTYPES{}[{\typeidx}] = {\mathit{dt}}
   }{
   C \vdashtypeuse {\typeidx} : \OKtypeuse
   }
   \qquad
   \end{array}


.. index:: heap type, type use
   pair: validation; heap type
   single: abstract syntax; heap type
.. _valid-heaptype:

Heap Types
~~~~~~~~~~




The :ref:`heap type <syntax-heaptype>` :math:`{\absheaptype}` is always :ref:`valid <valid-heaptype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashheaptype {\absheaptype} : \OKheaptype
   }
   \qquad
   \end{array}


.. index:: reference type, heap type
   pair: validation; reference type
   single: abstract syntax; reference type
.. _valid-reftype:

Reference Types
~~~~~~~~~~~~~~~




The :ref:`reference type <syntax-reftype>` :math:`(\REF~{\NULL^?}~{\heaptype})` is :ref:`valid <valid-reftype>` if:


   * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}` is :ref:`valid <valid-heaptype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashheaptype {\heaptype} : \OKheaptype
   }{
   C \vdashreftype \REF~{\NULL^?}~{\heaptype} : \OKreftype
   }
   \qquad
   \end{array}


.. index:: value type, reference type, number type, vector type
   pair: validation; value type
   single: abstract syntax; value type
.. _valid-valtype:

Value Types
~~~~~~~~~~~




The :ref:`value type <syntax-valtype>` :math:`{\valtype}` is :ref:`valid <valid-valtype>` if:


   * Either:

      * The :ref:`value type <syntax-valtype>` :math:`{\valtype}` is of the form :math:`{\numtype}`.

      * The :ref:`number type <syntax-numtype>` :math:`{\numtype}` is :ref:`valid <valid-numtype>`.

   * Or:

      * The :ref:`value type <syntax-valtype>` :math:`{\valtype}` is of the form :math:`{\vectype}`.

      * The :ref:`vector type <syntax-vectype>` :math:`{\vectype}` is :ref:`valid <valid-vectype>`.
   * Or:

      * The :ref:`value type <syntax-valtype>` :math:`{\valtype}` is of the form :math:`{\reftype}`.

      * The :ref:`reference type <syntax-reftype>` :math:`{\reftype}` is :ref:`valid <valid-reftype>`.
   * Or:

      * The :ref:`value type <syntax-valtype>` :math:`{\valtype}` is of the form :math:`\BOT`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashvaltype \BOT : \OKvaltype
   }
   \qquad
   \end{array}


.. index:: result type, value type
   pair: validation; result type
   single: abstract syntax; result type
.. _valid-resulttype:

Result Types
~~~~~~~~~~~~




The :ref:`result type <syntax-resulttype>` :math:`{t^\ast}` is :ref:`valid <valid-resulttype>` if:


   * For all :math:`t` in :math:`{t^\ast}`:

      * The :ref:`value type <syntax-valtype>` :math:`t` is :ref:`valid <valid-valtype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   (C \vdashvaltype t : \OKvaltype)^\ast
   }{
   C \vdashresulttype {t^\ast} : \OKresulttype
   }
   \qquad
   \end{array}


.. index:: block type, instruction type
   pair: validation; block type
   single: abstract syntax; block type
.. _valid-blocktype:

Block Types
~~~~~~~~~~~

:ref:`Block types <syntax-blocktype>` may be expressed in one of two forms, both of which are converted to :ref:`instruction types <syntax-instrtype>` by the following rules.




The :ref:`block type <syntax-blocktype>` :math:`{\typeidx}` is :ref:`valid <valid-blocktype>` as the :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~\rightarrow~{t_2^\ast}` if:


   * The :ref:`type <syntax-deftype>` :math:`C{.}\CTYPES{}[{\typeidx}]` exists.

   * The :ref:`expansion <aux-expand-deftype>` of :math:`C{.}\CTYPES{}[{\typeidx}]` is :math:`(\TFUNC~{t_1^\ast}~\Tarrow~{t_2^\ast})`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTYPES{}[{\typeidx}] \approxexpanddt \TFUNC~{t_1^\ast} \Tarrow {t_2^\ast}
   }{
   C \vdashblocktype {\typeidx} : {t_1^\ast} \rightarrow {t_2^\ast}
   }
   \qquad
   \end{array}





The :ref:`block type <syntax-blocktype>` :math:`{{\valtype}^?}` is :ref:`valid <valid-blocktype>` as the :ref:`instruction type <syntax-instrtype>` :math:`\epsilon~\rightarrow~{{\valtype}^?}` if:


   * If :math:`{\valtype}` is defined, then:

      * The :ref:`value type <syntax-valtype>` :math:`{\valtype}` is :ref:`valid <valid-valtype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   (C \vdashvaltype {\valtype} : \OKvaltype)^?
   }{
   C \vdashblocktype {{\valtype}^?} : \epsilon \rightarrow {{\valtype}^?}
   }
   \qquad
   \end{array}


.. index:: instruction type
   pair: validation; instruction type
   single: abstract syntax; instruction type
.. _valid-instrtype:

Instruction Types
~~~~~~~~~~~~~~~~~




The :ref:`instruction type <syntax-instrtype>` :math:`{t_1^\ast}~{\to}_{{x^\ast}}\,{t_2^\ast}` is :ref:`valid <valid-instrtype>` if:


   * The :ref:`result type <syntax-resulttype>` :math:`{t_1^\ast}` is :ref:`valid <valid-resulttype>`.

   * The :ref:`result type <syntax-resulttype>` :math:`{t_2^\ast}` is :ref:`valid <valid-resulttype>`.

   * For all :math:`x` in :math:`{x^\ast}`:

      * The :ref:`local <syntax-localtype>` :math:`C{.}\CLOCALS{}[x]` exists.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashresulttype {t_1^\ast} : \OKresulttype
    \qquad
   C \vdashresulttype {t_2^\ast} : \OKresulttype
    \qquad
   (C{.}\CLOCALS{}[x] = {{\mathit{lt}}})^\ast
   }{
   C \vdashinstrtype {t_1^\ast} \to_{{x^\ast}} {t_2^\ast} : \OKinstrtype
   }
   \qquad
   \end{array}


.. index:: composite type, function type, aggregate type, structure type, array type, field type, storage type, packed type, value type, mutability
   pair: validation; composite type
   pair: validation; aggregate type
   pair: validation; structure type
   pair: validation; array type
   pair: validation; function type
   pair: validation; field type
   pair: validation; storage type
   pair: validation; packed type
   single: abstract syntax; composite type
   single: abstract syntax; function type
   single: abstract syntax; structure type
   single: abstract syntax; array type
   single: abstract syntax; field type
   single: abstract syntax; storage type
   single: abstract syntax; packed type
   single: abstract syntax; value type
.. _valid-comptype:
.. _valid-aggrtype:
.. _valid-structtype:
.. _valid-arraytype:
.. _valid-functype:
.. _valid-fieldtype:
.. _valid-storagetype:
.. _valid-packtype:

Composite Types
~~~~~~~~~~~~~~~




The :ref:`composite type <syntax-comptype>` :math:`(\TSTRUCT~{{\fieldtype}^\ast})` is :ref:`valid <valid-comptype>` if:


   * For all :math:`{\fieldtype}` in :math:`{{\fieldtype}^\ast}`:

      * The :ref:`field type <syntax-fieldtype>` :math:`{\fieldtype}` is :ref:`valid <valid-fieldtype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   (C \vdashfieldtype {\fieldtype} : \OKfieldtype)^\ast
   }{
   C \vdashcomptype \TSTRUCT~{{\fieldtype}^\ast} : \OKcomptype
   }
   \qquad
   \end{array}





The :ref:`composite type <syntax-comptype>` :math:`(\TARRAY~{\fieldtype})` is :ref:`valid <valid-comptype>` if:


   * The :ref:`field type <syntax-fieldtype>` :math:`{\fieldtype}` is :ref:`valid <valid-fieldtype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashfieldtype {\fieldtype} : \OKfieldtype
   }{
   C \vdashcomptype \TARRAY~{\fieldtype} : \OKcomptype
   }
   \qquad
   \end{array}





The :ref:`composite type <syntax-comptype>` :math:`(\TFUNC~{t_1^\ast}~\Tarrow~{t_2^\ast})` is :ref:`valid <valid-comptype>` if:


   * The :ref:`result type <syntax-resulttype>` :math:`{t_1^\ast}` is :ref:`valid <valid-resulttype>`.

   * The :ref:`result type <syntax-resulttype>` :math:`{t_2^\ast}` is :ref:`valid <valid-resulttype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashresulttype {t_1^\ast} : \OKresulttype
    \qquad
   C \vdashresulttype {t_2^\ast} : \OKresulttype
   }{
   C \vdashcomptype \TFUNC~{t_1^\ast} \Tarrow {t_2^\ast} : \OKcomptype
   }
   \qquad
   \end{array}





The :ref:`field type <syntax-fieldtype>` :math:`({\TMUT^?}~{\storagetype})` is :ref:`valid <valid-fieldtype>` if:


   * The :ref:`storage type <syntax-storagetype>` :math:`{\storagetype}` is :ref:`valid <valid-storagetype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashstoragetype {\storagetype} : \OKstoragetype
   }{
   C \vdashfieldtype {\TMUT^?}~{\storagetype} : \OKfieldtype
   }
   \qquad
   \end{array}





The :ref:`packed type <syntax-packtype>` :math:`{\packtype}` is always :ref:`valid <valid-packtype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashpacktype {\packtype} : \OKpacktype
   }
   \qquad
   \end{array}


.. index:: recursive type, sub type, composite type, final, subtyping
   pair: abstract syntax; recursive type
   pair: abstract syntax; sub type
.. _valid-rectype:
.. _valid-subtype:

Recursive Types
~~~~~~~~~~~~~~~

:ref:`Recursive types <syntax-rectype>` are validated with respect to the first :ref:`type index <syntax-typeidx>` defined by the recursive group.




The :ref:`recursive type <syntax-rectype>` :math:`(\TREC~{{\subtype}^\ast})` is :ref:`valid <valid-rectype>` for the type index :math:`x` if:


   * Either:

      * The sub type sequence :math:`{{\subtype}^\ast}` is empty.

   * Or:

      * The sub type sequence :math:`{{\subtype}^\ast}` is of the form :math:`{\subtype}_1~{{\subtype'}^\ast}`.

      * The :ref:`sub type <syntax-subtype>` :math:`{\subtype}_1` is :ref:`valid <valid-subtype>` for the type index :math:`x`.

      * The :ref:`recursive type <syntax-rectype>` :math:`(\TREC~{{\subtype'}^\ast})` is :ref:`valid <valid-rectype>` for the type index :math:`x + 1`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashrectype \TREC~\epsilon : {\OKsubtype}{(x)}
   }
   \qquad
   \frac{
   C \vdashsubtype {\subtype}_1 : {\OKsubtype}{(x)}
    \qquad
   C \vdashrectype \TREC~{{\subtype}^\ast} : {\OKsubtype}{(x + 1)}
   }{
   C \vdashrectype \TREC~({\subtype}_1~{{\subtype}^\ast}) : {\OKsubtype}{(x)}
   }
   \qquad
   \end{array}





The :ref:`sub type <syntax-subtype>` :math:`(\TSUB~{\TFINAL^?}~{x^\ast}~{\comptype})` is :ref:`valid <valid-subtype>` for the type index :math:`x_0` if:


   * The length of :math:`{x^\ast}` is less than or equal to :math:`1`.

   * For all :math:`x` in :math:`{x^\ast}`:

      * The :ref:`index <syntax-idx>` :math:`x` is less than :math:`x_0`.

      * The :ref:`type <syntax-deftype>` :math:`C{.}\CTYPES{}[x]` exists.

      * The :ref:`sub type <syntax-subtype>` :math:`{\unrolldt}(C{.}\CTYPES{}[x])` is of the form :math:`(\TSUB~{y^\ast}~{\comptype'})`.

   * :math:`{{\comptype'}^\ast}` is the concatenation of all such :math:`{\comptype'}`.

   * The :ref:`composite type <syntax-comptype>` :math:`{\comptype}` is :ref:`valid <valid-comptype>`.

   * For all :math:`{\comptype'}` in :math:`{{\comptype'}^\ast}`:

      * The :ref:`composite type <syntax-comptype>` :math:`{\comptype}` :ref:`matches <match-comptype>` the :ref:`composite type <syntax-comptype>` :math:`{\comptype'}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   \begin{array}{@{}c@{}}
   {|{x^\ast}|} \leq 1
    \qquad
   (x < x_0)^\ast
    \qquad
   ({\unrolldt}(C{.}\CTYPES{}[x]) = \TSUB~{y^\ast}~{\comptype'})^\ast
    \\
   C \vdashcomptype {\comptype} : \OKcomptype
    \qquad
   (C \vdashcomptypematch {\comptype} \subcomptypematch {\comptype'})^\ast
   \end{array}
   }{
   C \vdashsubtype \TSUB~{\TFINAL^?}~{x^\ast}~{\comptype} : {\OKsubtype}{(x_0)}
   }
   \qquad
   \end{array}

.. note::
   The side condition on the index ensures that a declared supertype is a previously defined types,
   preventing cyclic subtype hierarchies.

   Future versions of WebAssembly may allow more than one supertype.


.. index:: limits
   pair: validation; limits
   single: abstract syntax; limits
.. _valid-limits:

Limits
~~~~~~

:ref:`Limits <syntax-limits>` must have meaningful bounds that are within a given range.




The :ref:`limits range <syntax-limits>` :math:`{}[ n \Ldotdot {m^?} ]` is :ref:`valid <valid-limits>` within :math:`k` if:


   * :math:`n` is less than or equal to :math:`k`.

   * If :math:`m` is defined, then:

      * :math:`n` is less than or equal to :math:`m`.

      * :math:`m` is less than or equal to :math:`k`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   n \leq k
    \qquad
   (n \leq m \leq k)^?
   }{
   C \vdashlimits {}[ n \Ldotdot {m^?} ] : k
   }
   \qquad
   \end{array}


.. index:: tag type, function type, exception tag
   pair: validation; tag type
   single: abstract syntax; tag type
.. _valid-tagtype:

Tag Types
~~~~~~~~~




The :ref:`tag type <syntax-tagtype>` :math:`{\typeuse}` is :ref:`valid <valid-tagtype>` if:


   * The :ref:`type use <syntax-typeuse>` :math:`{\typeuse}` is :ref:`valid <valid-typeuse>`.

   * The :ref:`expansion <aux-expand-typeuse>` of :math:`C` is :math:`(\TFUNC~{t_1^\ast}~\Tarrow)`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashtypeuse {\typeuse} : \OKtypeuse
    \qquad
   {\typeuse} \approxexpandyy_{C} \TFUNC~{t_1^\ast} \Tarrow {}[]
   }{
   C \vdashtagtype {\typeuse} : \OKtagtype
   }
   \qquad
   \end{array}


.. index:: global type, value type, mutability
   pair: validation; global type
   single: abstract syntax; global type
.. _valid-globaltype:

Global Types
~~~~~~~~~~~~




The :ref:`global type <syntax-globaltype>` :math:`({\TMUT^?}~t)` is :ref:`valid <valid-globaltype>` if:


   * The :ref:`value type <syntax-valtype>` :math:`t` is :ref:`valid <valid-valtype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashvaltype t : \OKvaltype
   }{
   C \vdashglobaltype {\TMUT^?}~t : \OKglobaltype
   }
   \qquad
   \end{array}


.. index:: memory type, limits
   pair: validation; memory type
   single: abstract syntax; memory type
.. _valid-memtype:

Memory Types
~~~~~~~~~~~~




The :ref:`memory type <syntax-memtype>` :math:`({\addrtype}~{\limits}~\PAGE)` is :ref:`valid <valid-memtype>` if:


   * The :ref:`limits range <syntax-limits>` :math:`{\limits}` is :ref:`valid <valid-limits>` within :math:`{2^{{|{\addrtype}|} - 16}}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashlimits {\limits} : {2^{{|{\addrtype}|} - 16}}
   }{
   C \vdashmemtype {\addrtype}~{\limits}~\PAGE : \OKmemtype
   }
   \qquad
   \end{array}


.. index:: table type, reference type, limits
   pair: validation; table type
   single: abstract syntax; table type
.. _valid-tabletype:

Table Types
~~~~~~~~~~~




The :ref:`table type <syntax-tabletype>` :math:`({\addrtype}~{\limits}~{\reftype})` is :ref:`valid <valid-tabletype>` if:


   * The :ref:`limits range <syntax-limits>` :math:`{\limits}` is :ref:`valid <valid-limits>` within :math:`{2^{{|{\addrtype}|}}} - 1`.

   * The :ref:`reference type <syntax-reftype>` :math:`{\reftype}` is :ref:`valid <valid-reftype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashlimits {\limits} : {2^{{|{\addrtype}|}}} - 1
    \qquad
   C \vdashreftype {\reftype} : \OKreftype
   }{
   C \vdashtabletype {\addrtype}~{\limits}~{\reftype} : \OKtabletype
   }
   \qquad
   \end{array}


.. index:: external type, function type, table type, memory type, global type
   pair: validation; external type
   single: abstract syntax; external type
.. _valid-externtype:

External Types
~~~~~~~~~~~~~~




The :ref:`external type <syntax-externtype>` :math:`(\XTTAG~{\tagtype})` is :ref:`valid <valid-externtype>` if:


   * The :ref:`tag type <syntax-tagtype>` :math:`{\tagtype}` is :ref:`valid <valid-tagtype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashtagtype {\tagtype} : \OKtagtype
   }{
   C \vdashexterntype \XTTAG~{\tagtype} : \OKexterntype
   }
   \qquad
   \end{array}





The :ref:`external type <syntax-externtype>` :math:`(\XTGLOBAL~{\globaltype})` is :ref:`valid <valid-externtype>` if:


   * The :ref:`global type <syntax-globaltype>` :math:`{\globaltype}` is :ref:`valid <valid-globaltype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashglobaltype {\globaltype} : \OKglobaltype
   }{
   C \vdashexterntype \XTGLOBAL~{\globaltype} : \OKexterntype
   }
   \qquad
   \end{array}





The :ref:`external type <syntax-externtype>` :math:`(\XTMEM~{\memtype})` is :ref:`valid <valid-externtype>` if:


   * The :ref:`memory type <syntax-memtype>` :math:`{\memtype}` is :ref:`valid <valid-memtype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashmemtype {\memtype} : \OKmemtype
   }{
   C \vdashexterntype \XTMEM~{\memtype} : \OKexterntype
   }
   \qquad
   \end{array}





The :ref:`external type <syntax-externtype>` :math:`(\XTTABLE~{\tabletype})` is :ref:`valid <valid-externtype>` if:


   * The :ref:`table type <syntax-tabletype>` :math:`{\tabletype}` is :ref:`valid <valid-tabletype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashtabletype {\tabletype} : \OKtabletype
   }{
   C \vdashexterntype \XTTABLE~{\tabletype} : \OKexterntype
   }
   \qquad
   \end{array}





The :ref:`external type <syntax-externtype>` :math:`(\XTFUNC~{\typeuse})` is :ref:`valid <valid-externtype>` if:


   * The :ref:`type use <syntax-typeuse>` :math:`{\typeuse}` is :ref:`valid <valid-typeuse>`.

   * The :ref:`expansion <aux-expand-typeuse>` of :math:`C` is :math:`(\TFUNC~{t_1^\ast}~\Tarrow~{t_2^\ast})`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashtypeuse {\typeuse} : \OKtypeuse
    \qquad
   {\typeuse} \approxexpandyy_{C} \TFUNC~{t_1^\ast} \Tarrow {t_2^\ast}
   }{
   C \vdashexterntype \XTFUNC~{\typeuse} : \OKexterntype
   }
   \qquad
   \end{array}
