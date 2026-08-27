.. index:: value
.. exec-val:

Values
------

.. index:: value, value type, validation, structure, structure type, structure instance, array, array type, array instance, function, function type, function instance, null reference, scalar reference, store
.. _valid-val:

Value Typing
~~~~~~~~~~~~

For the purpose of checking argument :ref:`values <syntax-val>` against the parameter types of exported :ref:`functions <syntax-func>`,
values are classified by :ref:`value types <syntax-valtype>`.
The following auxiliary typing rules specify this typing relation relative to a :ref:`store <syntax-store>` :math:`S` in which possibly referenced :ref:`addresses <syntax-addr>` live.




.. _valid-num:

Numeric Values
..............




The :ref:`number value <syntax-num>` :math:`({\mathit{nt}}{.}\CONST~c)` is :ref:`valid <valid-num>` with the :ref:`number type <syntax-numtype>` :math:`{\mathit{nt}}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   s \vdashnum {\mathit{nt}}{.}\CONST~c : {\mathit{nt}}
   }
   \qquad
   \end{array}


.. _valid-vec:

Vector Values
.............




The :ref:`vector value <syntax-vec>` :math:`({\mathit{vt}}{.}\VCONST~c)` is :ref:`valid <valid-vec>` with the :ref:`vector type <syntax-vectype>` :math:`{\mathit{vt}}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   s \vdashvec {\mathit{vt}}{.}\VCONST~c : {\mathit{vt}}
   }
   \qquad
   \end{array}


.. _valid-ref:

Null References
...............




The :ref:`reference value <syntax-ref>` :math:`\REFNULLADDR` is :ref:`valid <valid-ref>` with the :ref:`reference type <syntax-reftype>` :math:`(\REF~\NULL~\BOT)`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   s \vdashref \REFNULLADDR : \REF~\NULL~\BOT
   }
   \qquad
   \end{array}


.. _valid-ref.i31num:

Scalar References
.................




The :ref:`reference value <syntax-ref>` :math:`(\REFI31NUM~i)` is :ref:`valid <valid-ref>` with the :ref:`reference type <syntax-reftype>` :math:`(\REF~\I31)`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   s \vdashref \REFI31NUM~i : \REF~\I31
   }
   \qquad
   \end{array}


.. _valid-ref.struct:

Structure References
....................




The :ref:`reference value <syntax-ref>` :math:`(\REFSTRUCTADDR~a)` is :ref:`valid <valid-ref>` with the :ref:`reference type <syntax-reftype>` :math:`(\REF~{\mathit{dt}})` if:


   * The :ref:`structure instance <syntax-structinst>` :math:`s{.}\SSTRUCTS{}[a]` exists.

   * The :ref:`defined type <syntax-deftype>` :math:`s{.}\SSTRUCTS{}[a]{.}\SITYPE` is of the form :math:`{\mathit{dt}}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   s{.}\SSTRUCTS{}[a]{.}\SITYPE = {\mathit{dt}}
   }{
   s \vdashref \REFSTRUCTADDR~a : \REF~{\mathit{dt}}
   }
   \qquad
   \end{array}


.. _valid-ref.array:

Array References
................




The :ref:`reference value <syntax-ref>` :math:`(\REFARRAYADDR~a)` is :ref:`valid <valid-ref>` with the :ref:`reference type <syntax-reftype>` :math:`(\REF~{\mathit{dt}})` if:


   * The :ref:`array instance <syntax-arrayinst>` :math:`s{.}\SARRAYS{}[a]` exists.

   * The :ref:`defined type <syntax-deftype>` :math:`s{.}\SARRAYS{}[a]{.}\AITYPE` is of the form :math:`{\mathit{dt}}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   s{.}\SARRAYS{}[a]{.}\AITYPE = {\mathit{dt}}
   }{
   s \vdashref \REFARRAYADDR~a : \REF~{\mathit{dt}}
   }
   \qquad
   \end{array}


.. _valid-ref.exn:

Exception References
....................




The :ref:`reference value <syntax-ref>` :math:`(\REFEXNADDR~a)` is :ref:`valid <valid-ref>` with the :ref:`reference type <syntax-reftype>` :math:`(\REF~\EXN)` if:


   * The :ref:`exception instance <syntax-exninst>` :math:`s{.}\SEXNS{}[a]` exists.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   s{.}\SEXNS{}[a] = {\mathit{exn}}
   }{
   s \vdashref \REFEXNADDR~a : \REF~\EXN
   }
   \qquad
   \end{array}


Function References
...................




The :ref:`reference value <syntax-ref>` :math:`(\REFFUNCADDR~a)` is :ref:`valid <valid-ref>` with the :ref:`reference type <syntax-reftype>` :math:`(\REF~{\mathit{dt}})` if:


   * The :ref:`function instance <syntax-funcinst>` :math:`s{.}\SFUNCS{}[a]` exists.

   * The :ref:`defined type <syntax-deftype>` :math:`s{.}\SFUNCS{}[a]{.}\FITYPE` is of the form :math:`{\mathit{dt}}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   s{.}\SFUNCS{}[a]{.}\FITYPE = {\mathit{dt}}
   }{
   s \vdashref \REFFUNCADDR~a : \REF~{\mathit{dt}}
   }
   \qquad
   \end{array}


Host References
...............




The :ref:`reference value <syntax-ref>` :math:`(\REFHOSTADDR~a)` is :ref:`valid <valid-ref>` with the :ref:`reference type <syntax-reftype>` :math:`(\REF~\ANY)`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   s \vdashref \REFHOSTADDR~a : \REF~\ANY
   }
   \qquad
   \end{array}

.. note::
   A bare host reference is considered internalized.


External References
...................




The :ref:`reference value <syntax-ref>` :math:`(\REFEXTERN~{\reff})` is :ref:`valid <valid-ref>` with the :ref:`reference type <syntax-reftype>` :math:`(\REF~\EXTERN)` if:


   * The :ref:`reference value <syntax-ref>` :math:`{\reff}` is :ref:`valid <valid-ref>` with the :ref:`reference type <syntax-reftype>` :math:`(\REF~\ANY)`.

   * The :ref:`reference value <syntax-ref>` :math:`{\reff}` is not of the form :math:`\REFNULLADDR`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   s \vdashref {\reff} : \REF~\ANY
    \qquad
   {\reff} \neq \REFNULLADDR
   }{
   s \vdashref \REFEXTERN~{\reff} : \REF~\EXTERN
   }
   \qquad
   \end{array}


Subsumption
...........




The :ref:`reference value <syntax-ref>` :math:`{\reff}` is :ref:`valid <valid-ref>` with the :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}}` if:


   * The :ref:`reference value <syntax-ref>` :math:`{\reff}` is :ref:`valid <valid-ref>` with the :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}'}`.

   * Under the context :math:`\{ \CRETURN~\epsilon \}`, the :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}}` is :ref:`valid <valid-reftype>`.

   * The :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}'}` :ref:`matches <match-reftype>` the :ref:`reference type <syntax-reftype>` :math:`{\mathit{rt}}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   s \vdashref {\reff} : {\mathit{rt}'}
    \qquad
   \{  \} \vdashreftype {\mathit{rt}} : \OKreftype
    \qquad
   \{  \} \vdashreftypematch {\mathit{rt}'} \subreftypematch {\mathit{rt}}
   }{
   s \vdashref {\reff} : {\mathit{rt}}
   }
   \qquad
   \end{array}


.. index:: external address, external type, validation, import, store
.. _valid-externaddr:

External Typing
~~~~~~~~~~~~~~~

For the purpose of checking :ref:`external address <syntax-externaddr>` against :ref:`imports <syntax-import>`,
such values are classified by :ref:`external types <syntax-externtype>`.
The following auxiliary typing rules specify this typing relation relative to a :ref:`store <syntax-store>` :math:`S` in which the referenced instances live.


.. index:: function type, function address
.. _valid-externaddr-func:

Functions
.........




The :ref:`external address <syntax-externaddr>` :math:`(\XAFUNC~a)` is :ref:`valid <valid-externaddr>` with the :ref:`external type <syntax-externtype>` :math:`(\XTFUNC~{\funcinst}{.}\FITYPE)` if:


   * The :ref:`function instance <syntax-funcinst>` :math:`s{.}\SFUNCS{}[a]` exists.

   * The :ref:`function instance <syntax-funcinst>` :math:`s{.}\SFUNCS{}[a]` is of the form :math:`{\funcinst}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   s{.}\SFUNCS{}[a] = {\funcinst}
   }{
   s \vdashexternaddr \XAFUNC~a : \XTFUNC~{\funcinst}{.}\FITYPE
   }
   \qquad
   \end{array}


.. index:: table type, table address
.. _valid-externaddr-table:

Tables
......




The :ref:`external address <syntax-externaddr>` :math:`(\XATABLE~a)` is :ref:`valid <valid-externaddr>` with the :ref:`external type <syntax-externtype>` :math:`(\XTTABLE~{\tableinst}{.}\TITYPE)` if:


   * The :ref:`table instance <syntax-tableinst>` :math:`s{.}\STABLES{}[a]` exists.

   * The :ref:`table instance <syntax-tableinst>` :math:`s{.}\STABLES{}[a]` is of the form :math:`{\tableinst}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   s{.}\STABLES{}[a] = {\tableinst}
   }{
   s \vdashexternaddr \XATABLE~a : \XTTABLE~{\tableinst}{.}\TITYPE
   }
   \qquad
   \end{array}


.. index:: memory type, memory address
.. _valid-externaddr-mem:

Memories
........




The :ref:`external address <syntax-externaddr>` :math:`(\XAMEM~a)` is :ref:`valid <valid-externaddr>` with the :ref:`external type <syntax-externtype>` :math:`(\XTMEM~{\meminst}{.}\MITYPE)` if:


   * The :ref:`memory instance <syntax-meminst>` :math:`s{.}\SMEMS{}[a]` exists.

   * The :ref:`memory instance <syntax-meminst>` :math:`s{.}\SMEMS{}[a]` is of the form :math:`{\meminst}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   s{.}\SMEMS{}[a] = {\meminst}
   }{
   s \vdashexternaddr \XAMEM~a : \XTMEM~{\meminst}{.}\MITYPE
   }
   \qquad
   \end{array}


.. index:: global type, global address, value type, mutability
.. _valid-externaddr-global:

Globals
.......




The :ref:`external address <syntax-externaddr>` :math:`(\XAGLOBAL~a)` is :ref:`valid <valid-externaddr>` with the :ref:`external type <syntax-externtype>` :math:`(\XTGLOBAL~{\globalinst}{.}\GITYPE)` if:


   * The :ref:`global instance <syntax-globalinst>` :math:`s{.}\SGLOBALS{}[a]` exists.

   * The :ref:`global instance <syntax-globalinst>` :math:`s{.}\SGLOBALS{}[a]` is of the form :math:`{\globalinst}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   s{.}\SGLOBALS{}[a] = {\globalinst}
   }{
   s \vdashexternaddr \XAGLOBAL~a : \XTGLOBAL~{\globalinst}{.}\GITYPE
   }
   \qquad
   \end{array}


.. index:: tag type, tag address, exception tag, function type
.. _valid-externaddr-tag:

Tags
....




The :ref:`external address <syntax-externaddr>` :math:`(\XATAG~a)` is :ref:`valid <valid-externaddr>` with the :ref:`external type <syntax-externtype>` :math:`(\XTTAG~{\taginst}{.}\HITYPE)` if:


   * The :ref:`tag instance <syntax-taginst>` :math:`s{.}\STAGS{}[a]` exists.

   * The :ref:`tag instance <syntax-taginst>` :math:`s{.}\STAGS{}[a]` is of the form :math:`{\taginst}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   s{.}\STAGS{}[a] = {\taginst}
   }{
   s \vdashexternaddr \XATAG~a : \XTTAG~{\taginst}{.}\HITYPE
   }
   \qquad
   \end{array}


Subsumption
...........




The :ref:`external address <syntax-externaddr>` :math:`{\externaddr}` is :ref:`valid <valid-externaddr>` with the :ref:`external type <syntax-externtype>` :math:`{\mathit{xt}}` if:


   * The :ref:`external address <syntax-externaddr>` :math:`{\externaddr}` is :ref:`valid <valid-externaddr>` with the :ref:`external type <syntax-externtype>` :math:`{\mathit{xt}'}`.

   * Under the context :math:`\{ \CRETURN~\epsilon \}`, the :ref:`external type <syntax-externtype>` :math:`{\mathit{xt}}` is :ref:`valid <valid-externtype>`.

   * The :ref:`external type <syntax-externtype>` :math:`{\mathit{xt}'}` :ref:`matches <match-externtype>` the :ref:`external type <syntax-externtype>` :math:`{\mathit{xt}}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   s \vdashexternaddr {\externaddr} : {\mathit{xt}'}
    \qquad
   \{  \} \vdashexterntype {\mathit{xt}} : \OKexterntype
    \qquad
   \{  \} \vdashexterntypematch {\mathit{xt}'} \subexterntypematch {\mathit{xt}}
   }{
   s \vdashexternaddr {\externaddr} : {\mathit{xt}}
   }
   \qquad
   \end{array}
