Modules
-------

:ref:`Modules <syntax-module>` are valid when all the components they contain are valid.
To verify this, most definitions are themselves classified with a suitable type.


.. index:: type, type index, defined type, recursive type
   pair: abstract syntax; type
   single: abstract syntax; type
.. _valid-type:
.. _valid-types:

Types
~~~~~

The sequence of :ref:`types <syntax-type>` defined in a module is validated incrementally, yielding a sequence of :ref:`defined types <syntax-deftype>` representing them individually.




The :ref:`type definition <syntax-type>` :math:`(\TYPE~{\rectype})` is :ref:`valid <valid-type>` with the defined type sequence :math:`{{\mathit{dt}}^\ast}` if:


   * The length of :math:`C{.}\CTYPES` is equal to :math:`x`.

   * The defined type sequence :math:`{{\mathit{dt}}^\ast}` is of the form :math:`{{{{\rolldt}}_{x}^\ast}}{({\rectype})}`.

   * Let :math:`{C'}` be the same context as :math:`C`, but with the defined type sequence :math:`{{\mathit{dt}}^\ast}` appended to the field :math:`\CTYPES`.

   * Under the context :math:`{C'}`, the :ref:`recursive type <syntax-rectype>` :math:`{\rectype}` is :ref:`valid <valid-rectype>` for the type index :math:`x`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   x = {|C{.}\CTYPES|}
    \qquad
   {{\mathit{dt}}^\ast} = {{{{\rolldt}}_{x}^\ast}}{({\rectype})}
    \qquad
   C \oplus \{ \CTYPES~{{\mathit{dt}}^\ast} \} \vdashrectype {\rectype} : {\OKsubtype}{(x)}
   }{
   C \vdashtype \TYPE~{\rectype} : {{\mathit{dt}}^\ast}
   }
   \qquad
   \end{array}





The type definition sequence :math:`{{\type}^\ast}` is :ref:`valid <valid-type>` with the defined type sequence :math:`{{\deftype}^\ast}` if:


   * Either:

      * The type definition sequence :math:`{{\type}^\ast}` is empty.

      * The defined type sequence :math:`{{\deftype}^\ast}` is empty.

   * Or:

      * The type definition sequence :math:`{{\type}^\ast}` is of the form :math:`{\type}_1~{{\type'}^\ast}`.

      * The defined type sequence :math:`{{\deftype}^\ast}` is of the form :math:`{{\mathit{dt}}_1^\ast}~{{\mathit{dt}}^\ast}`.

      * The :ref:`type definition <syntax-type>` :math:`{\type}_1` is :ref:`valid <valid-type>` with the defined type sequence :math:`{{\mathit{dt}}_1^\ast}`.

      * Let :math:`{C'}` be the same context as :math:`C`, but with the defined type sequence :math:`{{\mathit{dt}}_1^\ast}` appended to the field :math:`\CTYPES`.

      * Under the context :math:`{C'}`, the type definition sequence :math:`{{\type'}^\ast}` is :ref:`valid <valid-type>` with the defined type sequence :math:`{{\mathit{dt}}^\ast}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashtypes \epsilon : \epsilon
   }
   \qquad
   \frac{
   C \vdashtype {\type}_1 : {{\mathit{dt}}_1^\ast}
    \qquad
   C \oplus \{ \CTYPES~{{\mathit{dt}}_1^\ast} \} \vdashtypes {{\type}^\ast} : {{\mathit{dt}}^\ast}
   }{
   C \vdashtypes {\type}_1~{{\type}^\ast} : {{\mathit{dt}}_1^\ast}~{{\mathit{dt}}^\ast}
   }
   \qquad
   \end{array}


.. index:: tag, tag type, function type, exception tag
   pair: validation; tag
   single: abstract syntax; tag
.. _valid-tag:

Tags
~~~~

Tags :math:`\tag` are classified by their :ref:`tag types <syntax-tagtype>`,
which are :ref:`defined types <syntax-deftype>` expanding to :ref:`function types <syntax-functype>`.




The :ref:`tag <syntax-tag>` :math:`(\TAG~{\tagtype})` is :ref:`valid <valid-tag>` with the :ref:`tag type <syntax-tagtype>` :math:`{\tagtype'}` if:


   * The :ref:`tag type <syntax-tagtype>` :math:`{\tagtype}` is :ref:`valid <valid-tagtype>`.

   * The :ref:`tag type <syntax-tagtype>` :math:`{\tagtype'}` is :math:`{{\clostype}}_{C}({\tagtype})`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashtagtype {\tagtype} : \OKtagtype
   }{
   C \vdashtag \TAG~{\tagtype} : {{\clostype}}_{C}({\tagtype})
   }
   \qquad
   \end{array}


.. index:: global, global type, expression, constant
   pair: validation; global
   single: abstract syntax; global
.. _valid-global:
.. _valid-globalseq:

Globals
~~~~~~~

Globals :math:`{\global}` are classified by :ref:`global types <syntax-globaltype>`.




The :ref:`global <syntax-global>` :math:`(\GLOBAL~{\globaltype}~{\expr})` is :ref:`valid <valid-global>` with the :ref:`global type <syntax-globaltype>` :math:`{\globaltype}` if:


   * The :ref:`global type <syntax-globaltype>` :math:`{\globaltype}` is :ref:`valid <valid-globaltype>`.

   * The :ref:`global type <syntax-globaltype>` :math:`{\globaltype}` is of the form :math:`({\TMUT^?}~t)`.

   * The :ref:`expression <syntax-expr>` :math:`{\expr}` is :ref:`valid <valid-expr>` with the :ref:`value type <syntax-valtype>` :math:`t`.

   * :math:`{\expr}` is constant.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashglobaltype {\globaltype} : \OKglobaltype
    \qquad
   {\globaltype} = {\TMUT^?}~t
    \qquad
   C \vdashexprokconst {\expr} : t~\CONSTexprokconst
   }{
   C \vdashglobal \GLOBAL~{\globaltype}~{\expr} : {\globaltype}
   }
   \qquad
   \end{array}

Sequences of globals are handled incrementally, such that each definition has access to previous definitions.




The global sequence :math:`{{\global}^\ast}` is :ref:`valid <valid-global>` with the global type sequence :math:`{{\globaltype}^\ast}` if:


   * Either:

      * The global sequence :math:`{{\global}^\ast}` is empty.

      * The global type sequence :math:`{{\globaltype}^\ast}` is empty.

   * Or:

      * The global sequence :math:`{{\global}^\ast}` is of the form :math:`{\global}_1~{{\global'}^\ast}`.

      * The global type sequence :math:`{{\globaltype}^\ast}` is of the form :math:`{\mathit{gt}}_1~{{\mathit{gt}}^\ast}`.

      * The :ref:`global <syntax-global>` :math:`{\global}_1` is :ref:`valid <valid-global>` with the :ref:`global type <syntax-globaltype>` :math:`{\mathit{gt}}_1`.

      * Let :math:`{C'}` be the same context as :math:`C`, but with the global type sequence :math:`{\mathit{gt}}_1` appended to the field :math:`\CGLOBALS`.

      * Under the context :math:`{C'}`, the global sequence :math:`{{\global'}^\ast}` is :ref:`valid <valid-global>` with the global type sequence :math:`{{\mathit{gt}}^\ast}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashglobals \epsilon : \epsilon
   }
   \qquad
   \frac{
   C \vdashglobal {\global}_1 : {\mathit{gt}}_1
    \qquad
   C \oplus \{ \CGLOBALS~{\mathit{gt}}_1 \} \vdashglobals {{\global}^\ast} : {{\mathit{gt}}^\ast}
   }{
   C \vdashglobals {\global}_1~{{\global}^\ast} : {\mathit{gt}}_1~{{\mathit{gt}}^\ast}
   }
   \qquad
   \end{array}


.. index:: memory, memory type
   pair: validation; memory
   single: abstract syntax; memory
.. _valid-mem:

Memories
~~~~~~~~

Memories :math:`{\mem}` are classified by :ref:`memory types <syntax-memtype>`.




The :ref:`memory <syntax-mem>` :math:`(\MEMORY~{\memtype})` is :ref:`valid <valid-mem>` with the :ref:`memory type <syntax-memtype>` :math:`{\memtype}` if:


   * The :ref:`memory type <syntax-memtype>` :math:`{\memtype}` is :ref:`valid <valid-memtype>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashmemtype {\memtype} : \OKmemtype
   }{
   C \vdashmem \MEMORY~{\memtype} : {\memtype}
   }
   \qquad
   \end{array}


.. index:: table, table type, reference type, expression, constant, defaultable
   pair: validation; table
   single: abstract syntax; table
.. _valid-table:

Tables
~~~~~~

Tables :math:`{\table}` are classified by :ref:`table types <syntax-tabletype>`.




The :ref:`table <syntax-table>` :math:`(\TABLE~{\tabletype}~{\expr})` is :ref:`valid <valid-table>` with the :ref:`table type <syntax-tabletype>` :math:`{\tabletype}` if:


   * The :ref:`table type <syntax-tabletype>` :math:`{\tabletype}` is :ref:`valid <valid-tabletype>`.

   * The :ref:`table type <syntax-tabletype>` :math:`{\tabletype}` is of the form :math:`({\mathit{at}}~{\mathit{lim}}~{\mathit{rt}})`.

   * The :ref:`expression <syntax-expr>` :math:`{\expr}` is :ref:`valid <valid-expr>` with the :ref:`value type <syntax-valtype>` :math:`{\mathit{rt}}`.

   * :math:`{\expr}` is constant.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashtabletype {\tabletype} : \OKtabletype
    \qquad
   {\tabletype} = {\mathit{at}}~{\mathit{lim}}~{\mathit{rt}}
    \qquad
   C \vdashexprokconst {\expr} : {\mathit{rt}}~\CONSTexprokconst
   }{
   C \vdashtable \TABLE~{\tabletype}~{\expr} : {\tabletype}
   }
   \qquad
   \end{array}


.. index:: function, local, function index, local index, type index, function type, value type, local type, expression, import
   pair: abstract syntax; function
   single: abstract syntax; function
.. _valid-func:

Functions
~~~~~~~~~

Functions :math:`{\func}` are classified by :ref:`defined types <syntax-deftype>` that :ref:`expand <aux-expand-deftype>` to :ref:`function types <syntax-functype>` of the form :math:`\TFUNC~{t_1^\ast} \Tarrow {t_2^\ast}`.




The :ref:`function <syntax-func>` :math:`(\FUNC~x~{{\local}^\ast}~{\expr})` is :ref:`valid <valid-func>` with the :ref:`type <syntax-deftype>` :math:`C{.}\CTYPES{}[x]` if:


   * The :ref:`type <syntax-deftype>` :math:`C{.}\CTYPES{}[x]` exists.

   * The :ref:`expansion <aux-expand-deftype>` of :math:`C{.}\CTYPES{}[x]` is :math:`(\TFUNC~{t_1^\ast}~\Tarrow~{t_2^\ast})`.

   * For all :math:`{\local}` in :math:`{{\local}^\ast}`:

      * The :ref:`local <syntax-local>` :math:`{\local}` is :ref:`valid <valid-local>` with the :ref:`local type <syntax-localtype>` :math:`{{\mathit{lt}}}`.

   * :math:`{{{\mathit{lt}}}^\ast}` is the concatenation of all such :math:`{{\mathit{lt}}}`.

   * Under the context :math:`C` with the field :math:`\CLOCALS` appended by :math:`{(\SET~t_1)^\ast}~{{{\mathit{lt}}}^\ast}` and the field :math:`\CLABELS` appended by :math:`{t_2^\ast}` and the field :math:`\CRETURN` appended by :math:`{t_2^\ast}`, the :ref:`expression <syntax-expr>` :math:`{\expr}` is :ref:`valid <valid-expr>` with the :ref:`result type <syntax-resulttype>` :math:`{t_2^\ast}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   \begin{array}{@{}c@{}}
   C{.}\CTYPES{}[x] \approxexpanddt \TFUNC~{t_1^\ast} \Tarrow {t_2^\ast}
    \qquad
   (C \vdashlocal {\local} : {{\mathit{lt}}})^\ast
    \\
   C \oplus \{ \CLOCALS~{(\SET~t_1)^\ast}~{{{\mathit{lt}}}^\ast},\;\allowbreak \CLABELS~({t_2^\ast}),\;\allowbreak \CRETURN~({t_2^\ast}) \} \vdashexpr {\expr} : {t_2^\ast}
   \end{array}
   }{
   C \vdashfunc \FUNC~x~{{\local}^\ast}~{\expr} : C{.}\CTYPES{}[x]
   }
   \qquad
   \end{array}


.. index:: local, local type, value type
   pair: validation; local
   single: abstract syntax; local
.. _valid-local:

Locals
~~~~~~

Locals :math:`{\local}` are classified with :ref:`local types <syntax-localtype>`.




The :ref:`local <syntax-local>` :math:`(\LOCAL~t)` is :ref:`valid <valid-local>` with the :ref:`local type <syntax-localtype>` :math:`({\init}~t)` if:


   * The :ref:`value type <syntax-valtype>` :math:`t` is :ref:`valid <valid-valtype>`.

   * Either:

      * The :ref:`initialization status <syntax-init>` :math:`{\init}` is of the form :math:`\SET`.

      * A :ref:`default value <aux-default>` for :math:`t` is defined.

   * Or:

      * The :ref:`initialization status <syntax-init>` :math:`{\init}` is of the form :math:`\UNSET`.

      * A :ref:`default value <aux-default>` for :math:`t` is not defined.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashvaltype t : \OKvaltype
    \qquad
   {{\default}}_{t} \neq \epsilon
   }{
   C \vdashlocal \LOCAL~t : \SET~t
   }
   \qquad
   \frac{
   C \vdashvaltype t : \OKvaltype
    \qquad
   {{\default}}_{t} = \epsilon
   }{
   C \vdashlocal \LOCAL~t : \UNSET~t
   }
   \qquad
   \end{array}

.. note::
   For cases where both rules are applicable, the former yields the more permissable type.


.. index:: data, memory, memory index, expression, constant, byte
   pair: validation; data
   single: abstract syntax; data
   single: memory; data
   single: data; segment
.. _valid-data:

Data Segments
~~~~~~~~~~~~~

Data segments :math:`{\data}` are classified by the singleton :ref:`data type <syntax-datatype>`, which merely expresses well-formedness.




The :ref:`memory segment <syntax-data>` :math:`(\DATA~{b^\ast}~{\datamode})` is :ref:`valid <valid-data>` if:


   * The :ref:`data mode <syntax-datamode>` :math:`{\datamode}` is :ref:`valid <valid-datamode>`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashdatamode {\datamode} : \OKdata
   }{
   C \vdashdata \DATA~{b^\ast}~{\datamode} : \OKdata
   }
   \qquad
   \end{array}


.. _valid-datamode:




The :ref:`data mode <syntax-datamode>` :math:`{\datamode}` is :ref:`valid <valid-datamode>` if:


   * Either:

      * The :ref:`data mode <syntax-datamode>` :math:`{\datamode}` is of the form :math:`\DPASSIVE`.

   * Or:

      * The :ref:`data mode <syntax-datamode>` :math:`{\datamode}` is of the form :math:`(\DACTIVE~x~{\expr})`.

      * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` exists.

      * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` is of the form :math:`({\mathit{at}}~{\mathit{lim}}~\PAGE)`.

      * The :ref:`expression <syntax-expr>` :math:`{\expr}` is :ref:`valid <valid-expr>` with the :ref:`value type <syntax-valtype>` :math:`{\mathit{at}}`.

      * :math:`{\expr}` is constant.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashdatamode \DPASSIVE : \OKdata
   }
   \qquad
   \frac{
   C{.}\CMEMS{}[x] = {\mathit{at}}~{\mathit{lim}}~\PAGE
    \qquad
   C \vdashexprokconst {\expr} : {\mathit{at}}~\CONSTexprokconst
   }{
   C \vdashdatamode \DACTIVE~x~{\expr} : \OKdata
   }
   \qquad
   \end{array}


.. index:: element, table, table index, expression, constant, function index
   pair: validation; element
   single: abstract syntax; element
   single: table; element
   single: element; segment
.. _valid-elem:

Element Segments
~~~~~~~~~~~~~~~~

Element segments :math:`{\elem}` are classified by their :ref:`element type <syntax-elemtype>`.




The :ref:`table segment <syntax-elem>` :math:`(\ELEM~{\elemtype}~{{\expr}^\ast}~{\elemmode})` is :ref:`valid <valid-elem>` with the :ref:`element type <syntax-elemtype>` :math:`{\elemtype}` if:


   * The :ref:`reference type <syntax-reftype>` :math:`{\elemtype}` is :ref:`valid <valid-reftype>`.

   * For all :math:`{\expr}` in :math:`{{\expr}^\ast}`:

      * The :ref:`expression <syntax-expr>` :math:`{\expr}` is :ref:`valid <valid-expr>` with the :ref:`value type <syntax-valtype>` :math:`{\elemtype}`.

      * :math:`{\expr}` is constant.

   * The :ref:`element mode <syntax-elemmode>` :math:`{\elemmode}` is :ref:`valid <valid-elemmode>` with the :ref:`element type <syntax-elemtype>` :math:`{\elemtype}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashreftype {\elemtype} : \OKreftype
    \qquad
   (C \vdashexprokconst {\expr} : {\elemtype}~\CONSTexprokconst)^\ast
    \qquad
   C \vdashelemmode {\elemmode} : {\elemtype}
   }{
   C \vdashelem \ELEM~{\elemtype}~{{\expr}^\ast}~{\elemmode} : {\elemtype}
   }
   \qquad
   \end{array}


.. _valid-elemmode:




The :ref:`element mode <syntax-elemmode>` :math:`{\elemmode}` is :ref:`valid <valid-elemmode>` with the :ref:`element type <syntax-elemtype>` :math:`{\mathit{rt}}` if:


   * Either:

      * The :ref:`element mode <syntax-elemmode>` :math:`{\elemmode}` is of the form :math:`\EPASSIVE`.

   * Or:

      * The :ref:`element mode <syntax-elemmode>` :math:`{\elemmode}` is of the form :math:`\EDECLARE`.
   * Or:

      * The :ref:`element mode <syntax-elemmode>` :math:`{\elemmode}` is of the form :math:`(\EACTIVE~x~{\expr})`.

      * The :ref:`table <syntax-tabletype>` :math:`C{.}\CTABLES{}[x]` exists.

      * The :ref:`table <syntax-tabletype>` :math:`C{.}\CTABLES{}[x]` is of the form :math:`({\mathit{at}}~{\mathit{lim}}~{\mathit{rt}'})`.

      * The :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}}` :ref:`matches <match-reftype>` the :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}'}`.

      * The :ref:`expression <syntax-expr>` :math:`{\expr}` is :ref:`valid <valid-expr>` with the :ref:`value type <syntax-valtype>` :math:`{\mathit{at}}`.

      * :math:`{\expr}` is constant.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashelemmode \EPASSIVE : {\mathit{rt}}
   }
   \qquad
   \frac{
   }{
   C \vdashelemmode \EDECLARE : {\mathit{rt}}
   }
   \\[3ex]\displaystyle
   \frac{
   C{.}\CTABLES{}[x] = {\mathit{at}}~{\mathit{lim}}~{\mathit{rt}'}
    \qquad
   C \vdashreftypematch {\mathit{rt}} \subreftypematch {\mathit{rt}'}
    \qquad
   C \vdashexprokconst {\expr} : {\mathit{at}}~\CONSTexprokconst
   }{
   C \vdashelemmode \EACTIVE~x~{\expr} : {\mathit{rt}}
   }
   \qquad
   \end{array}


.. index:: start function, function index
   pair: validation; start function
   single: abstract syntax; start function
.. _valid-start:

Start Function
~~~~~~~~~~~~~~




The :ref:`start function <syntax-start>` :math:`(\START~x)` is :ref:`valid <valid-start>` if:


   * The :ref:`function <syntax-deftype>` :math:`C{.}\CFUNCS{}[x]` exists.

   * The :ref:`expansion <aux-expand-deftype>` of :math:`C{.}\CFUNCS{}[x]` is :math:`(\TFUNC~\Tarrow)`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CFUNCS{}[x] \approxexpanddt \TFUNC~\epsilon \Tarrow \epsilon
   }{
   C \vdashstart \START~x : \OKstart
   }
   \qquad
   \end{array}


.. index:: import, name, tag type, global type, memory type, table type, function type
   pair: validation; import
   single: abstract syntax; import
.. _valid-importdesc:
.. _valid-import:

Imports
~~~~~~~

Imports :math:`{\import}` are classified by :ref:`external types <syntax-externtype>`.




The :ref:`import <syntax-import>` :math:`(\IMPORT~{\name}_1~{\name}_2~{\mathit{xt}})` is :ref:`valid <valid-import>` with the :ref:`external type <syntax-externtype>` :math:`{\externtype}` if:


   * The :ref:`external type <syntax-externtype>` :math:`{\mathit{xt}}` is :ref:`valid <valid-externtype>`.

   * The :ref:`external type <syntax-externtype>` :math:`{\externtype}` is :math:`{{\clostype}}_{C}({\mathit{xt}})`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashexterntype {\mathit{xt}} : \OKexterntype
   }{
   C \vdashimport \IMPORT~{\name}_1~{\name}_2~{\mathit{xt}} : {{\clostype}}_{C}({\mathit{xt}})
   }
   \qquad
   \end{array}


.. index:: export, name, index, function index, table index, memory index, global index, tag index
   pair: validation; export
   single: abstract syntax; export
.. _valid-exportdesc:
.. _valid-export:
.. _valid-externidx:

Exports
~~~~~~~

Exports :math:`{\export}` are classified by their :ref:`external type <syntax-externtype>`.




The :ref:`export <syntax-export>` :math:`(\EXPORT~{\name}~{\externidx})` is :ref:`valid <valid-export>` with the :ref:`name <syntax-name>` :math:`{\name}` and the :ref:`external type <syntax-externtype>` :math:`{\mathit{xt}}` if:


   * The :ref:`external index <syntax-externidx>` :math:`{\externidx}` is :ref:`valid <valid-externidx>` with the :ref:`external type <syntax-externtype>` :math:`{\mathit{xt}}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashexternidx {\externidx} : {\mathit{xt}}
   }{
   C \vdashexport \EXPORT~{\name}~{\externidx} : {\name}~{\mathit{xt}}
   }
   \qquad
   \end{array}


:math:`\XXTAG~x`
................




The :ref:`external index <syntax-externidx>` :math:`(\XXTAG~x)` is :ref:`valid <valid-externidx>` with the :ref:`external type <syntax-externtype>` :math:`(\XTTAG~{\mathit{jt}})` if:


   * The :ref:`tag <syntax-tagtype>` :math:`C{.}\CTAGS{}[x]` exists.

   * The :ref:`tag <syntax-tagtype>` :math:`C{.}\CTAGS{}[x]` is of the form :math:`{\mathit{jt}}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTAGS{}[x] = {\mathit{jt}}
   }{
   C \vdashexternidx \XXTAG~x : \XTTAG~{\mathit{jt}}
   }
   \qquad
   \end{array}


:math:`\XXGLOBAL~x`
...................




The :ref:`external index <syntax-externidx>` :math:`(\XXGLOBAL~x)` is :ref:`valid <valid-externidx>` with the :ref:`external type <syntax-externtype>` :math:`(\XTGLOBAL~{\mathit{gt}})` if:


   * The :ref:`global <syntax-globaltype>` :math:`C{.}\CGLOBALS{}[x]` exists.

   * The :ref:`global <syntax-globaltype>` :math:`C{.}\CGLOBALS{}[x]` is of the form :math:`{\mathit{gt}}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CGLOBALS{}[x] = {\mathit{gt}}
   }{
   C \vdashexternidx \XXGLOBAL~x : \XTGLOBAL~{\mathit{gt}}
   }
   \qquad
   \end{array}


:math:`\XXMEM~x`
................




The :ref:`external index <syntax-externidx>` :math:`(\XXMEM~x)` is :ref:`valid <valid-externidx>` with the :ref:`external type <syntax-externtype>` :math:`(\XTMEM~{\mathit{mt}})` if:


   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` exists.

   * The :ref:`memory <syntax-memtype>` :math:`C{.}\CMEMS{}[x]` is of the form :math:`{\mathit{mt}}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CMEMS{}[x] = {\mathit{mt}}
   }{
   C \vdashexternidx \XXMEM~x : \XTMEM~{\mathit{mt}}
   }
   \qquad
   \end{array}


:math:`\XXTABLE~x`
..................




The :ref:`external index <syntax-externidx>` :math:`(\XXTABLE~x)` is :ref:`valid <valid-externidx>` with the :ref:`external type <syntax-externtype>` :math:`(\XTTABLE~{\mathit{tt}})` if:


   * The :ref:`table <syntax-tabletype>` :math:`C{.}\CTABLES{}[x]` exists.

   * The :ref:`table <syntax-tabletype>` :math:`C{.}\CTABLES{}[x]` is of the form :math:`{\mathit{tt}}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CTABLES{}[x] = {\mathit{tt}}
   }{
   C \vdashexternidx \XXTABLE~x : \XTTABLE~{\mathit{tt}}
   }
   \qquad
   \end{array}


:math:`\XXFUNC~x`
.................




The :ref:`external index <syntax-externidx>` :math:`(\XXFUNC~x)` is :ref:`valid <valid-externidx>` with the :ref:`external type <syntax-externtype>` :math:`(\XTFUNC~{\mathit{dt}})` if:


   * The :ref:`function <syntax-deftype>` :math:`C{.}\CFUNCS{}[x]` exists.

   * The :ref:`function <syntax-deftype>` :math:`C{.}\CFUNCS{}[x]` is of the form :math:`{\mathit{dt}}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C{.}\CFUNCS{}[x] = {\mathit{dt}}
   }{
   C \vdashexternidx \XXFUNC~x : \XTFUNC~{\mathit{dt}}
   }
   \qquad
   \end{array}


.. index:: module, type definition, recursive type, tag, global, memory, table, function, data segment, element segment, start function, import, export, context
   pair: validation; module
   single: abstract syntax; module
.. _valid-module:
.. _syntax-moduletype:

Modules
~~~~~~~

Modules are classified by their mapping from the :ref:`external types <syntax-externtype>` of their :ref:`imports <syntax-import>` to those of their :ref:`exports <syntax-export>`.

A module is entirely *closed*,
that is, its components can only refer to definitions that appear in the module itself.
Consequently, no initial :ref:`context <context>` is required.
Instead, the :ref:`context <context>` :math:`C` for validation of the module's content is constructed from the definitions in the module.




The :ref:`module <syntax-module>` :math:`(\MODULE~{{\type}^\ast}~{{\import}^\ast}~{{\tag}^\ast}~{{\global}^\ast}~{{\mem}^\ast}~{{\table}^\ast}~{{\func}^\ast}~{{\data}^\ast}~{{\elem}^\ast}~{{\start}^?}~{{\export}^\ast})` is :ref:`valid <valid-module>` with the :ref:`module type <syntax-moduletype>` :math:`{\moduletype}` if:


   * Under the context :math:`\{ \CRETURN~\epsilon \}`, the type definition sequence :math:`{{\type}^\ast}` is :ref:`valid <valid-type>` with the defined type sequence :math:`{{\mathit{dt}'}^\ast}`.

   * For all :math:`{\import}` in :math:`{{\import}^\ast}`:

      * Under the context :math:`\{ \CTYPES~{{\mathit{dt}'}^\ast},\;\allowbreak \CRETURN~\epsilon \}`, the :ref:`import <syntax-import>` :math:`{\import}` is :ref:`valid <valid-import>` with the :ref:`external type <syntax-externtype>` :math:`{\mathit{xt}}_{\mathsf{i}}`.

   * :math:`{{\mathit{xt}}_{\mathsf{i}}^\ast}` is the concatenation of all such :math:`{\mathit{xt}}_{\mathsf{i}}`.

   * For all :math:`{\tag}` in :math:`{{\tag}^\ast}`:

      * Under the context :math:`{C'}`, the :ref:`tag <syntax-tag>` :math:`{\tag}` is :ref:`valid <valid-tag>` with the :ref:`tag type <syntax-tagtype>` :math:`{\mathit{jt}}`.

   * :math:`{{\mathit{jt}}^\ast}` is the concatenation of all such :math:`{\mathit{jt}}`.

   * Under the context :math:`{C'}`, the global sequence :math:`{{\global}^\ast}` is :ref:`valid <valid-global>` with the global type sequence :math:`{{\mathit{gt}}^\ast}`.

   * For all :math:`{\mem}` in :math:`{{\mem}^\ast}`:

      * Under the context :math:`{C'}`, the :ref:`memory <syntax-mem>` :math:`{\mem}` is :ref:`valid <valid-mem>` with the :ref:`memory type <syntax-memtype>` :math:`{\mathit{mt}}`.

   * :math:`{{\mathit{mt}}^\ast}` is the concatenation of all such :math:`{\mathit{mt}}`.

   * For all :math:`{\table}` in :math:`{{\table}^\ast}`:

      * Under the context :math:`{C'}`, the :ref:`table <syntax-table>` :math:`{\table}` is :ref:`valid <valid-table>` with the :ref:`table type <syntax-tabletype>` :math:`{\mathit{tt}}`.

   * :math:`{{\mathit{tt}}^\ast}` is the concatenation of all such :math:`{\mathit{tt}}`.

   * For all :math:`{\func}` in :math:`{{\func}^\ast}`:

      * The :ref:`function <syntax-func>` :math:`{\func}` is :ref:`valid <valid-func>` with the :ref:`defined type <syntax-deftype>` :math:`{\mathit{dt}}`.

   * :math:`{{\mathit{dt}}^\ast}` is the concatenation of all such :math:`{\mathit{dt}}`.

   * For all :math:`{\data}` in :math:`{{\data}^\ast}`:

      * The :ref:`memory segment <syntax-data>` :math:`{\data}` is :ref:`valid <valid-data>`.

   * :math:`{{\mathit{ok}}^\ast}` is the concatenation of all such :math:`{\mathit{ok}}`.

   * For all :math:`{\elem}` in :math:`{{\elem}^\ast}`:

      * The :ref:`table segment <syntax-elem>` :math:`{\elem}` is :ref:`valid <valid-elem>` with the :ref:`element type <syntax-elemtype>` :math:`{\mathit{rt}}`.

   * :math:`{{\mathit{rt}}^\ast}` is the concatenation of all such :math:`{\mathit{rt}}`.

   * If :math:`{\start}` is defined, then:

      * The :ref:`start function <syntax-start>` :math:`{\start}` is :ref:`valid <valid-start>`.

   * For all :math:`{\export}` in :math:`{{\export}^\ast}`:

      * The :ref:`export <syntax-export>` :math:`{\export}` is :ref:`valid <valid-export>` with the :ref:`name <syntax-name>` :math:`{\mathit{nm}}` and the :ref:`external type <syntax-externtype>` :math:`{\mathit{xt}}_{\mathsf{e}}`.

   * :math:`{{\mathit{nm}}^\ast}` is the concatenation of all such :math:`{\mathit{nm}}`.

   * :math:`{{\mathit{xt}}_{\mathsf{e}}^\ast}` is the concatenation of all such :math:`{\mathit{xt}}_{\mathsf{e}}`.

   * :math:`{{\mathit{nm}}^\ast}~{\mathrm{disjoint}}` is true.

   * The :ref:`context <context>` :math:`C` is of the form :math:`{C'}` with the field :math:`\CTAGS` appended by :math:`{{\mathit{jt}}_{\mathsf{i}}^\ast}~{{\mathit{jt}}^\ast}` and the field :math:`\CGLOBALS` appended by :math:`{{\mathit{gt}}^\ast}` and the field :math:`\CMEMS` appended by :math:`{{\mathit{mt}}_{\mathsf{i}}^\ast}~{{\mathit{mt}}^\ast}` and the field :math:`\CTABLES` appended by :math:`{{\mathit{tt}}_{\mathsf{i}}^\ast}~{{\mathit{tt}}^\ast}` and the field :math:`\CDATAS` appended by :math:`{{\mathit{ok}}^\ast}` and the field :math:`\CELEMS` appended by :math:`{{\mathit{rt}}^\ast}`.

   * The :ref:`context <context>` :math:`{C'}` is of the form :math:`\{ \CTYPES~{{\mathit{dt}'}^\ast},\;\allowbreak \CGLOBALS~{{\mathit{gt}}_{\mathsf{i}}^\ast},\;\allowbreak \CFUNCS~{{\mathit{dt}}_{\mathsf{i}}^\ast}~{{\mathit{dt}}^\ast},\;\allowbreak \CRETURN~\epsilon,\;\allowbreak \CREFS~{x^\ast} \}`.

   * The function index sequence :math:`{x^\ast}` is of the form :math:`{\freefuncidx}({{\global}^\ast}~{{\mem}^\ast}~{{\table}^\ast}~{{\elem}^\ast}~{{\export}^\ast})`.

   * The tag type sequence :math:`{{\mathit{jt}}_{\mathsf{i}}^\ast}` is of the form :math:`{\tagsxt}({{\mathit{xt}}_{\mathsf{i}}^\ast})`.

   * The global type sequence :math:`{{\mathit{gt}}_{\mathsf{i}}^\ast}` is of the form :math:`{\globalsxt}({{\mathit{xt}}_{\mathsf{i}}^\ast})`.

   * The memory type sequence :math:`{{\mathit{mt}}_{\mathsf{i}}^\ast}` is of the form :math:`{\memsxt}({{\mathit{xt}}_{\mathsf{i}}^\ast})`.

   * The table type sequence :math:`{{\mathit{tt}}_{\mathsf{i}}^\ast}` is of the form :math:`{\tablesxt}({{\mathit{xt}}_{\mathsf{i}}^\ast})`.

   * The defined type sequence :math:`{{\mathit{dt}}_{\mathsf{i}}^\ast}` is of the form :math:`{\funcsxt}({{\mathit{xt}}_{\mathsf{i}}^\ast})`.

   * The :ref:`module type <syntax-moduletype>` :math:`{\moduletype}` is :math:`{{\clostype}}_{C}({{\mathit{xt}}_{\mathsf{i}}^\ast}~\toM~{{\mathit{xt}}_{\mathsf{e}}^\ast})`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   \begin{array}{@{}c@{}}
   \{  \} \vdashtypes {{\type}^\ast} : {{\mathit{dt}'}^\ast}
    \qquad
   (\{ \CTYPES~{{\mathit{dt}'}^\ast} \} \vdashimport {\import} : {\mathit{xt}}_{\mathsf{i}})^\ast
    \\
   ({C'} \vdashtag {\tag} : {\mathit{jt}})^\ast
    \qquad
   {C'} \vdashglobals {{\global}^\ast} : {{\mathit{gt}}^\ast}
    \qquad
   ({C'} \vdashmem {\mem} : {\mathit{mt}})^\ast
    \qquad
   ({C'} \vdashtable {\table} : {\mathit{tt}})^\ast
    \qquad
   (C \vdashfunc {\func} : {\mathit{dt}})^\ast
    \\
   (C \vdashdata {\data} : {\mathit{ok}})^\ast
    \qquad
   (C \vdashelem {\elem} : {\mathit{rt}})^\ast
    \qquad
   (C \vdashstart {\start} : \OKstart)^?
    \qquad
   (C \vdashexport {\export} : {\mathit{nm}}~{\mathit{xt}}_{\mathsf{e}})^\ast
    \qquad
   {{\mathit{nm}}^\ast}~{\mathrm{disjoint}}
    \\
   C = {C'} \oplus \{ \CTAGS~{{\mathit{jt}}_{\mathsf{i}}^\ast}~{{\mathit{jt}}^\ast},\;\allowbreak \CGLOBALS~{{\mathit{gt}}^\ast},\;\allowbreak \CMEMS~{{\mathit{mt}}_{\mathsf{i}}^\ast}~{{\mathit{mt}}^\ast},\;\allowbreak \CTABLES~{{\mathit{tt}}_{\mathsf{i}}^\ast}~{{\mathit{tt}}^\ast},\;\allowbreak \CDATAS~{{\mathit{ok}}^\ast},\;\allowbreak \CELEMS~{{\mathit{rt}}^\ast} \}
    \\
   {C'} = \{ \CTYPES~{{\mathit{dt}'}^\ast},\;\allowbreak \CGLOBALS~{{\mathit{gt}}_{\mathsf{i}}^\ast},\;\allowbreak \CFUNCS~{{\mathit{dt}}_{\mathsf{i}}^\ast}~{{\mathit{dt}}^\ast},\;\allowbreak \CREFS~{x^\ast} \}
    \qquad
   {x^\ast} = {\freefuncidx}({{\global}^\ast}~{{\mem}^\ast}~{{\table}^\ast}~{{\elem}^\ast}~{{\export}^\ast})
    \\
   {{\mathit{jt}}_{\mathsf{i}}^\ast} = {\tagsxt}({{\mathit{xt}}_{\mathsf{i}}^\ast})
    \qquad
   {{\mathit{gt}}_{\mathsf{i}}^\ast} = {\globalsxt}({{\mathit{xt}}_{\mathsf{i}}^\ast})
    \qquad
   {{\mathit{mt}}_{\mathsf{i}}^\ast} = {\memsxt}({{\mathit{xt}}_{\mathsf{i}}^\ast})
    \qquad
   {{\mathit{tt}}_{\mathsf{i}}^\ast} = {\tablesxt}({{\mathit{xt}}_{\mathsf{i}}^\ast})
    \qquad
   {{\mathit{dt}}_{\mathsf{i}}^\ast} = {\funcsxt}({{\mathit{xt}}_{\mathsf{i}}^\ast})
   \end{array}
   }{
   {\vdashmodule}\, \MODULE~{{\type}^\ast}~{{\import}^\ast}~{{\tag}^\ast}~{{\global}^\ast}~{{\mem}^\ast}~{{\table}^\ast}~{{\func}^\ast}~{{\data}^\ast}~{{\elem}^\ast}~{{\start}^?}~{{\export}^\ast} : {{\clostype}}_{C}({{\mathit{xt}}_{\mathsf{i}}^\ast} \toM {{\mathit{xt}}_{\mathsf{e}}^\ast})
   }
   \qquad
   \end{array}

.. note::
   All functions in a module are mutually recursive.
   Consequently, the definition of the :ref:`context <context>` :math:`C` in this rule is recursive:
   it depends on the outcome of validation of the function, table, memory, and global definitions contained in the module,
   which itself depends on :math:`C`.
   However, this recursion is just a specification device.
   All types needed to construct :math:`C` can easily be determined from a simple pre-pass over the module that does not perform any actual validation.

   Globals, however, are not recursive but evaluated sequentially, such that each :ref:`constant expressions <valid-constant>` only has access to imported or previously defined globals.
