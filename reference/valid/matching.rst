.. index:: ! matching, ! subtyping
.. _subtyping:
.. _match:

Matching
--------

On most types, a notion of *subtyping* is defined that is applicable in :ref:`validation <valid>` rules, during :ref:`module instantiation <exec-instantiation>` when checking the types of imports, or during :ref:`execution <exec>`, when performing casts.


.. index:: number type
.. _match-numtype:

Number Types
~~~~~~~~~~~~




The :ref:`number type <syntax-numtype>` :math:`{\numtype}` :ref:`matches <match-numtype>` only itself.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashnumtypematch {\numtype} \subnumtypematch {\numtype}
   }
   \qquad
   \end{array}


.. index:: vector type
.. _match-vectype:

Vector Types
~~~~~~~~~~~~




The :ref:`vector type <syntax-vectype>` :math:`{\vectype}` :ref:`matches <match-vectype>` only itself.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashvectypematch {\vectype} \subvectypematch {\vectype}
   }
   \qquad
   \end{array}


.. index:: heap type, defined type, structure type, array type, function type, unboxed scalar type
.. _match-heaptype:

Heap Types
~~~~~~~~~~




The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_1` :ref:`matches <match-heaptype>` the :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_2` if:


   * Either:

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_2` is of the form :math:`{\heaptype}_1`.

   * Or:

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype'}` is :ref:`valid <valid-heaptype>`.

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_1` :ref:`matches <match-heaptype>` the :ref:`heap type <syntax-heaptype>` :math:`{\heaptype'}`.

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype'}` :ref:`matches <match-heaptype>` the :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_2`.
   * Or:

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_1` is of the form :math:`\EQT`.

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_2` is of the form :math:`\ANY`.
   * Or:

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_1` is of the form :math:`\I31`.

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_2` is of the form :math:`\EQT`.
   * Or:

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_1` is of the form :math:`\STRUCT`.

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_2` is of the form :math:`\EQT`.
   * Or:

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_1` is of the form :math:`\ARRAY`.

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_2` is of the form :math:`\EQT`.
   * Or:

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_1` is of the form :math:`{\deftype}`.

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_2` is of the form :math:`\STRUCT`.

      * The :ref:`expansion <aux-expand-deftype>` of :math:`{\deftype}` is :math:`(\TSTRUCT~{{\fieldtype}^\ast})`.
   * Or:

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_1` is of the form :math:`{\deftype}`.

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_2` is of the form :math:`\ARRAY`.

      * The :ref:`expansion <aux-expand-deftype>` of :math:`{\deftype}` is :math:`(\TARRAY~{\fieldtype})`.
   * Or:

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_1` is of the form :math:`{\deftype}`.

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_2` is of the form :math:`\FUNCT`.

      * The :ref:`expansion <aux-expand-deftype>` of :math:`{\deftype}` is :math:`(\TFUNC~{t_1^\ast}~\Tarrow~{t_2^\ast})`.
   * Or:

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_1` is of the form :math:`{\deftype}_1`.

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_2` is of the form :math:`{\deftype}_2`.

      * The :ref:`defined type <syntax-deftype>` :math:`{\deftype}_1` :ref:`matches <match-deftype>` the :ref:`defined type <syntax-deftype>` :math:`{\deftype}_2`.
   * Or:

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_1` is of the form :math:`{\typeidx}`.

      * The :ref:`type <syntax-deftype>` :math:`C{.}\CTYPES{}[{\typeidx}]` exists.

      * The :ref:`type <syntax-deftype>` :math:`C{.}\CTYPES{}[{\typeidx}]` :ref:`matches <match-deftype>` the :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_2`.
   * Or:

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_2` is of the form :math:`{\typeidx}`.

      * The :ref:`type <syntax-deftype>` :math:`C{.}\CTYPES{}[{\typeidx}]` exists.

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_1` :ref:`matches <match-heaptype>` the :ref:`type <syntax-deftype>` :math:`C{.}\CTYPES{}[{\typeidx}]`.
   * Or:

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_1` is of the form :math:`(\REC {.} i)`.

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_2` is of the form :math:`\STRUCT`.

      * The :ref:`recursive type <syntax-subtype>` :math:`C{.}\CRECS{}[i]` exists.

      * The :ref:`recursive type <syntax-subtype>` :math:`C{.}\CRECS{}[i]` is of the form :math:`(\TSUB~{\TFINAL^?}~(\TSTRUCT~{{\fieldtype}^\ast}))`.
   * Or:

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_1` is of the form :math:`(\REC {.} i)`.

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_2` is of the form :math:`\ARRAY`.

      * The :ref:`recursive type <syntax-subtype>` :math:`C{.}\CRECS{}[i]` exists.

      * The :ref:`recursive type <syntax-subtype>` :math:`C{.}\CRECS{}[i]` is of the form :math:`(\TSUB~{\TFINAL^?}~(\TARRAY~{\fieldtype}))`.
   * Or:

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_1` is of the form :math:`(\REC {.} i)`.

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_2` is of the form :math:`\FUNCT`.

      * The :ref:`recursive type <syntax-subtype>` :math:`C{.}\CRECS{}[i]` exists.

      * The :ref:`recursive type <syntax-subtype>` :math:`C{.}\CRECS{}[i]` is of the form :math:`(\TSUB~{\TFINAL^?}~(\TFUNC~{t_1^\ast}~\Tarrow~{t_2^\ast}))`.
   * Or:

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_1` is of the form :math:`(\REC {.} i)`.

      * The length of :math:`{{\typeuse}^\ast}` is greater than :math:`j`.

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_2` is of the form :math:`{{\typeuse}^\ast}{}[j]`.

      * The :ref:`recursive type <syntax-subtype>` :math:`C{.}\CRECS{}[i]` exists.

      * The :ref:`recursive type <syntax-subtype>` :math:`C{.}\CRECS{}[i]` is of the form :math:`(\TSUB~{\TFINAL^?}~{{\typeuse}^\ast}~{\mathit{ct}})`.
   * Or:

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_1` is of the form :math:`\NONE`.

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_2` :ref:`matches <match-heaptype>` the :ref:`heap type <syntax-heaptype>` :math:`\ANY`.

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_2` is not of the form :math:`\BOT`.
   * Or:

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_1` is of the form :math:`\NOFUNC`.

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_2` :ref:`matches <match-heaptype>` the :ref:`heap type <syntax-heaptype>` :math:`\FUNCT`.

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_2` is not of the form :math:`\BOT`.
   * Or:

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_1` is of the form :math:`\NOEXN`.

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_2` :ref:`matches <match-heaptype>` the :ref:`heap type <syntax-heaptype>` :math:`\EXN`.

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_2` is not of the form :math:`\BOT`.
   * Or:

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_1` is of the form :math:`\NOEXTERN`.

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_2` :ref:`matches <match-heaptype>` the :ref:`heap type <syntax-heaptype>` :math:`\EXTERN`.

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_2` is not of the form :math:`\BOT`.
   * Or:

      * The :ref:`heap type <syntax-heaptype>` :math:`{\heaptype}_1` is of the form :math:`\BOT`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashheaptypematch {\heaptype} \subheaptypematch {\heaptype}
   }
   \\[3ex]\displaystyle
   \frac{
   C \vdashheaptype {\heaptype'} : \OKheaptype
    \qquad
   C \vdashheaptypematch {\heaptype}_1 \subheaptypematch {\heaptype'}
    \qquad
   C \vdashheaptypematch {\heaptype'} \subheaptypematch {\heaptype}_2
   }{
   C \vdashheaptypematch {\heaptype}_1 \subheaptypematch {\heaptype}_2
   }
   \\[3ex]\displaystyle
   \frac{
   }{
   C \vdashheaptypematch \EQT \subheaptypematch \ANY
   }
   \qquad
   \frac{
   }{
   C \vdashheaptypematch \I31 \subheaptypematch \EQT
   }
   \qquad
   \frac{
   }{
   C \vdashheaptypematch \STRUCT \subheaptypematch \EQT
   }
   \qquad
   \frac{
   }{
   C \vdashheaptypematch \ARRAY \subheaptypematch \EQT
   }
   \\[3ex]\displaystyle
   \frac{
   {\deftype} \approxexpanddt \TSTRUCT~{{\fieldtype}^\ast}
   }{
   C \vdashheaptypematch {\deftype} \subheaptypematch \STRUCT
   }
   \qquad
   \frac{
   {\deftype} \approxexpanddt \TARRAY~{\fieldtype}
   }{
   C \vdashheaptypematch {\deftype} \subheaptypematch \ARRAY
   }
   \qquad
   \frac{
   {\deftype} \approxexpanddt \TFUNC~{t_1^\ast} \Tarrow {t_2^\ast}
   }{
   C \vdashheaptypematch {\deftype} \subheaptypematch \FUNCT
   }
   \\[3ex]\displaystyle
   \frac{
   C \vdashheaptypematch C{.}\CTYPES{}[{\typeidx}] \subheaptypematch {\heaptype}
   }{
   C \vdashheaptypematch {\typeidx} \subheaptypematch {\heaptype}
   }
   \qquad
   \frac{
   C \vdashheaptypematch {\heaptype} \subheaptypematch C{.}\CTYPES{}[{\typeidx}]
   }{
   C \vdashheaptypematch {\heaptype} \subheaptypematch {\typeidx}
   }
   \\[3ex]\displaystyle
   \frac{
   C \vdashheaptypematch {\heaptype} \subheaptypematch \ANY
    \qquad
   {\heaptype} \neq \BOT
   }{
   C \vdashheaptypematch \NONE \subheaptypematch {\heaptype}
   }
   \qquad
   \frac{
   C \vdashheaptypematch {\heaptype} \subheaptypematch \FUNCT
    \qquad
   {\heaptype} \neq \BOT
   }{
   C \vdashheaptypematch \NOFUNC \subheaptypematch {\heaptype}
   }
   \\[3ex]\displaystyle
   \frac{
   C \vdashheaptypematch {\heaptype} \subheaptypematch \EXN
    \qquad
   {\heaptype} \neq \BOT
   }{
   C \vdashheaptypematch \NOEXN \subheaptypematch {\heaptype}
   }
   \qquad
   \frac{
   C \vdashheaptypematch {\heaptype} \subheaptypematch \EXTERN
    \qquad
   {\heaptype} \neq \BOT
   }{
   C \vdashheaptypematch \NOEXTERN \subheaptypematch {\heaptype}
   }
   \\[3ex]\displaystyle
   \frac{
   }{
   C \vdashheaptypematch \BOT \subheaptypematch {\heaptype}
   }
   \qquad
   \end{array}



.. index:: reference type
.. _match-reftype:

Reference Types
~~~~~~~~~~~~~~~




The :ref:`reference type <syntax-reftype>` :math:`(\REF~{{\NULL}_1^?}~{\mathit{ht}}_1)` :ref:`matches <match-reftype>` the :ref:`reference type <syntax-reftype>` :math:`(\REF~{{\NULL}_2^?}~{\mathit{ht}}_2)` if:


   * The :ref:`heap type <syntax-heaptype>` :math:`{\mathit{ht}}_1` :ref:`matches <match-heaptype>` the :ref:`heap type <syntax-heaptype>` :math:`{\mathit{ht}}_2`.

   * Either:

      * :math:`{{\NULL}_1^?}` is absent.

      * :math:`{{\NULL}_2^?}` is absent.

   * Or:

      * :math:`{{\NULL}_1^?}` is of the form :math:`{\NULL^?}`.

      * :math:`{{\NULL}_2^?}` is of the form :math:`\NULL`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashheaptypematch {\mathit{ht}}_1 \subheaptypematch {\mathit{ht}}_2
   }{
   C \vdashreftypematch \REF~{\mathit{ht}}_1 \subreftypematch \REF~{\mathit{ht}}_2
   }
   \qquad
   \frac{
   C \vdashheaptypematch {\mathit{ht}}_1 \subheaptypematch {\mathit{ht}}_2
   }{
   C \vdashreftypematch \REF~{\NULL^?}~{\mathit{ht}}_1 \subreftypematch \REF~\NULL~{\mathit{ht}}_2
   }
   \qquad
   \end{array}


.. index:: value type, number type, reference type
.. _match-valtype:

Value Types
~~~~~~~~~~~




The :ref:`value type <syntax-valtype>` :math:`{\valtype}_1` :ref:`matches <match-valtype>` the :ref:`value type <syntax-valtype>` :math:`{\valtype}_2` if:


   * Either:

      * The :ref:`value type <syntax-valtype>` :math:`{\valtype}_1` is of the form :math:`{\numtype}_1`.

      * The :ref:`value type <syntax-valtype>` :math:`{\valtype}_2` is of the form :math:`{\numtype}_2`.

      * The :ref:`number type <syntax-numtype>` :math:`{\numtype}_1` :ref:`matches <match-numtype>` the :ref:`number type <syntax-numtype>` :math:`{\numtype}_2`.

   * Or:

      * The :ref:`value type <syntax-valtype>` :math:`{\valtype}_1` is of the form :math:`{\vectype}_1`.

      * The :ref:`value type <syntax-valtype>` :math:`{\valtype}_2` is of the form :math:`{\vectype}_2`.

      * The :ref:`vector type <syntax-vectype>` :math:`{\vectype}_1` :ref:`matches <match-vectype>` the :ref:`vector type <syntax-vectype>` :math:`{\vectype}_2`.
   * Or:

      * The :ref:`value type <syntax-valtype>` :math:`{\valtype}_1` is of the form :math:`{\reftype}_1`.

      * The :ref:`value type <syntax-valtype>` :math:`{\valtype}_2` is of the form :math:`{\reftype}_2`.

      * The :ref:`reference type <syntax-reftype>` :math:`{\reftype}_1` :ref:`matches <match-reftype>` the :ref:`reference type <syntax-reftype>` :math:`{\reftype}_2`.
   * Or:

      * The :ref:`value type <syntax-valtype>` :math:`{\valtype}_1` is of the form :math:`\BOT`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashvaltypematch \BOT \subvaltypematch {\valtype}
   }
   \qquad
   \end{array}



.. index:: result type, value type
.. _match-resulttype:

Result Types
~~~~~~~~~~~~

Subtyping is lifted to :ref:`result types <syntax-resulttype>` in a pointwise manner.




The :ref:`result type <syntax-resulttype>` :math:`{t_1^\ast}` :ref:`matches <match-resulttype>` the :ref:`result type <syntax-resulttype>` :math:`{t_2^\ast}` if:


   * For all :math:`t_1` in :math:`{t_1^\ast}`, and corresponding :math:`t_2` in :math:`{t_2^\ast}`:

      * The :ref:`value type <syntax-valtype>` :math:`t_1` :ref:`matches <match-valtype>` the :ref:`value type <syntax-valtype>` :math:`t_2`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   (C \vdashvaltypematch t_1 \subvaltypematch t_2)^\ast
   }{
   C \vdashresulttypematch {t_1^\ast} \subresulttypematch {t_2^\ast}
   }
   \qquad
   \end{array}


.. index:: instruction type, result type
.. _match-instrtype:

Instruction Types
~~~~~~~~~~~~~~~~~

Subtyping is further lifted to :ref:`instruction types <syntax-instrtype>`.




The :ref:`instruction type <syntax-instrtype>` :math:`{t_{11}^\ast}~{\to}_{{x_1^\ast}}\,{t_{12}^\ast}` :ref:`matches <match-instrtype>` the :ref:`instruction type <syntax-instrtype>` :math:`{t_{21}^\ast}~{\to}_{{x_2^\ast}}\,{t_{22}^\ast}` if:


   * The :ref:`result type <syntax-resulttype>` :math:`{t_{21}^\ast}` :ref:`matches <match-resulttype>` the :ref:`result type <syntax-resulttype>` :math:`{t_{11}^\ast}`.

   * The :ref:`result type <syntax-resulttype>` :math:`{t_{12}^\ast}` :ref:`matches <match-resulttype>` the :ref:`result type <syntax-resulttype>` :math:`{t_{22}^\ast}`.

   * The local index sequence :math:`{x^\ast}` is of the form :math:`{x_2^\ast} \setminus {x_1^\ast}`.

   * For all :math:`x` in :math:`{x^\ast}`:

      * The :ref:`local <syntax-localtype>` :math:`C{.}\CLOCALS{}[x]` exists.

      * The :ref:`local <syntax-localtype>` :math:`C{.}\CLOCALS{}[x]` is of the form :math:`(\SET~t)`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashresulttypematch {t_{21}^\ast} \subresulttypematch {t_{11}^\ast}
    \qquad
   C \vdashresulttypematch {t_{12}^\ast} \subresulttypematch {t_{22}^\ast}
    \qquad
   {x^\ast} = {x_2^\ast} \setminus {x_1^\ast}
    \qquad
   (C{.}\CLOCALS{}[x] = \SET~t)^\ast
   }{
   C \vdashinstrtypematch {t_{11}^\ast} \to_{{x_1^\ast}} {t_{12}^\ast} \subinstrtypematch {t_{21}^\ast} \to_{{x_2^\ast}} {t_{22}^\ast}
   }
   \qquad
   \end{array}

.. note::
   Instruction types are contravariant in their input and covariant in their output.
   Moreover, the supertype may ignore variables from the init set :math:`{x_1^\ast}`.
   It may also *add* variables to the init set, provided these are already set in the context, i.e., are vacuously initialized.

.. scratch
   Subtyping also incorporates a sort of "frame" condition, which allows adding arbitrary invariant stack elements on both sides in the super type.


.. index:: composite types, aggregate type, structure type, array type, funciton type, result type, field type
.. _match-comptype:
.. _match-structtype:
.. _match-arraytype:
.. _match-functype:

Composite Types
~~~~~~~~~~~~~~~




The :ref:`composite type <syntax-comptype>` :math:`{\comptype}_1` :ref:`matches <match-comptype>` the :ref:`composite type <syntax-comptype>` :math:`{\comptype}_2` if:


   * Either:

      * The :ref:`composite type <syntax-comptype>` :math:`{\comptype}_1` is of the form :math:`(\TSTRUCT~{{\mathit{ft}}_1^\ast}~{{\mathit{ft}'}_1^\ast})`.

      * The :ref:`composite type <syntax-comptype>` :math:`{\comptype}_2` is of the form :math:`(\TSTRUCT~{{\mathit{ft}}_2^\ast})`.

      * For all :math:`{\mathit{ft}}_1` in :math:`{{\mathit{ft}}_1^\ast}`, and corresponding :math:`{\mathit{ft}}_2` in :math:`{{\mathit{ft}}_2^\ast}`:

         * The :ref:`field type <syntax-fieldtype>` :math:`{\mathit{ft}}_1` :ref:`matches <match-fieldtype>` the :ref:`field type <syntax-fieldtype>` :math:`{\mathit{ft}}_2`.

   * Or:

      * The :ref:`composite type <syntax-comptype>` :math:`{\comptype}_1` is of the form :math:`(\TARRAY~{\mathit{ft}}_1)`.

      * The :ref:`composite type <syntax-comptype>` :math:`{\comptype}_2` is of the form :math:`(\TARRAY~{\mathit{ft}}_2)`.

      * The :ref:`field type <syntax-fieldtype>` :math:`{\mathit{ft}}_1` :ref:`matches <match-fieldtype>` the :ref:`field type <syntax-fieldtype>` :math:`{\mathit{ft}}_2`.
   * Or:

      * The :ref:`composite type <syntax-comptype>` :math:`{\comptype}_1` is of the form :math:`(\TFUNC~{t_{11}^\ast}~\Tarrow~{t_{12}^\ast})`.

      * The :ref:`composite type <syntax-comptype>` :math:`{\comptype}_2` is of the form :math:`(\TFUNC~{t_{21}^\ast}~\Tarrow~{t_{22}^\ast})`.

      * The :ref:`result type <syntax-resulttype>` :math:`{t_{21}^\ast}` :ref:`matches <match-resulttype>` the :ref:`result type <syntax-resulttype>` :math:`{t_{11}^\ast}`.

      * The :ref:`result type <syntax-resulttype>` :math:`{t_{12}^\ast}` :ref:`matches <match-resulttype>` the :ref:`result type <syntax-resulttype>` :math:`{t_{22}^\ast}`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   (C \vdashfieldtypematch {\mathit{ft}}_1 \subfieldtypematch {\mathit{ft}}_2)^\ast
   }{
   C \vdashcomptypematch \TSTRUCT~({{\mathit{ft}}_1^\ast}~{{\mathit{ft}'}_1^\ast}) \subcomptypematch \TSTRUCT~{{\mathit{ft}}_2^\ast}
   }
   \qquad
   \frac{
   C \vdashfieldtypematch {\mathit{ft}}_1 \subfieldtypematch {\mathit{ft}}_2
   }{
   C \vdashcomptypematch \TARRAY~{\mathit{ft}}_1 \subcomptypematch \TARRAY~{\mathit{ft}}_2
   }
   \qquad
   \frac{
   C \vdashresulttypematch {t_{21}^\ast} \subresulttypematch {t_{11}^\ast}
    \qquad
   C \vdashresulttypematch {t_{12}^\ast} \subresulttypematch {t_{22}^\ast}
   }{
   C \vdashcomptypematch \TFUNC~{t_{11}^\ast} \Tarrow {t_{12}^\ast} \subcomptypematch \TFUNC~{t_{21}^\ast} \Tarrow {t_{22}^\ast}
   }
   \qquad
   \end{array}


.. index:: field type, storage type, value type, packed type, mutability
.. _match-fieldtype:
.. _match-storagetype:
.. _match-packtype:

Field Types
~~~~~~~~~~~




The :ref:`field type <syntax-fieldtype>` :math:`({{\TMUT}_1^?}~{\mathit{zt}}_1)` :ref:`matches <match-fieldtype>` the :ref:`field type <syntax-fieldtype>` :math:`({{\TMUT}_2^?}~{\mathit{zt}}_2)` if:


   * The :ref:`storage type <syntax-storagetype>` :math:`{\mathit{zt}}_1` :ref:`matches <match-storagetype>` the :ref:`storage type <syntax-storagetype>` :math:`{\mathit{zt}}_2`.

   * Either:

      * :math:`{{\TMUT}_1^?}` is absent.

      * :math:`{{\TMUT}_2^?}` is absent.

   * Or:

      * :math:`{{\TMUT}_1^?}` is of the form :math:`\TMUT`.

      * :math:`{{\TMUT}_2^?}` is of the form :math:`\TMUT`.

      * The :ref:`storage type <syntax-storagetype>` :math:`{\mathit{zt}}_2` :ref:`matches <match-storagetype>` the :ref:`storage type <syntax-storagetype>` :math:`{\mathit{zt}}_1`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashstoragetypematch {\mathit{zt}}_1 \substoragetypematch {\mathit{zt}}_2
   }{
   C \vdashfieldtypematch {\mathit{zt}}_1 \subfieldtypematch {\mathit{zt}}_2
   }
   \qquad
   \frac{
   C \vdashstoragetypematch {\mathit{zt}}_1 \substoragetypematch {\mathit{zt}}_2
    \qquad
   C \vdashstoragetypematch {\mathit{zt}}_2 \substoragetypematch {\mathit{zt}}_1
   }{
   C \vdashfieldtypematch \TMUT~{\mathit{zt}}_1 \subfieldtypematch \TMUT~{\mathit{zt}}_2
   }
   \qquad
   \end{array}





The :ref:`storage type <syntax-storagetype>` :math:`{\storagetype}_1` :ref:`matches <match-storagetype>` the :ref:`storage type <syntax-storagetype>` :math:`{\storagetype}_2` if:


   * Either:

      * The :ref:`storage type <syntax-storagetype>` :math:`{\storagetype}_1` is of the form :math:`{\valtype}_1`.

      * The :ref:`storage type <syntax-storagetype>` :math:`{\storagetype}_2` is of the form :math:`{\valtype}_2`.

      * The :ref:`value type <syntax-valtype>` :math:`{\valtype}_1` :ref:`matches <match-valtype>` the :ref:`value type <syntax-valtype>` :math:`{\valtype}_2`.

   * Or:

      * The :ref:`storage type <syntax-storagetype>` :math:`{\storagetype}_1` is of the form :math:`{\packtype}_1`.

      * The :ref:`storage type <syntax-storagetype>` :math:`{\storagetype}_2` is of the form :math:`{\packtype}_2`.

      * The :ref:`packed type <syntax-packtype>` :math:`{\packtype}_1` :ref:`matches <match-packtype>` the :ref:`packed type <syntax-packtype>` :math:`{\packtype}_2`.







The :ref:`packed type <syntax-packtype>` :math:`{\packtype}` :ref:`matches <match-packtype>` only itself.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   }{
   C \vdashpacktypematch {\packtype} \subpacktypematch {\packtype}
   }
   \qquad
   \end{array}


.. index:: defined type, recursive type, unroll, type equivalence
   pair: abstract syntax; defined type
.. _match-deftype:

Defined Types
~~~~~~~~~~~~~




The :ref:`defined type <syntax-deftype>` :math:`{\deftype}_1` :ref:`matches <match-deftype>` the :ref:`defined type <syntax-deftype>` :math:`{\deftype}_2` if:


   * Either:

      * The :ref:`defined type <syntax-deftype>` :math:`{{\clostype}}_{C}({\deftype}_1)` is of the form :math:`{{\clostype}}_{C}({\deftype}_2)`.

   * Or:

      * The :ref:`sub type <syntax-subtype>` :math:`{\unrolldt}({\deftype}_1)` is of the form :math:`(\TSUB~{\TFINAL^?}~{{\typeuse}^\ast}~{\mathit{ct}})`.

      * The length of :math:`{{\typeuse}^\ast}` is greater than :math:`i`.

      * The :ref:`type use <syntax-typeuse>` :math:`{{\typeuse}^\ast}{}[i]` :ref:`matches <match>` the :ref:`heap type <syntax-heaptype>` :math:`{\deftype}_2`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   {{\clostype}}_{C}({\deftype}_1) = {{\clostype}}_{C}({\deftype}_2)
   }{
   C \vdashdeftypematch {\deftype}_1 \subdeftypematch {\deftype}_2
   }
   \\[3ex]\displaystyle
   \frac{
   {\unrolldt}({\deftype}_1) = \TSUB~{\TFINAL^?}~{{\typeuse}^\ast}~{\mathit{ct}}
    \qquad
   C \vdashheaptypematch {{\typeuse}^\ast}{}[i] \subheaptypematch {\deftype}_2
   }{
   C \vdashdeftypematch {\deftype}_1 \subdeftypematch {\deftype}_2
   }
   \qquad
   \end{array}

.. note::
   Note that there is no explicit definition of type *equivalence*,
   since it coincides with syntactic equality,
   as used in the premise of the former rule above.


.. index:: limits
.. _match-limits:

Limits
~~~~~~




The :ref:`limits range <syntax-limits>` :math:`{}[ n_1 \Ldotdot {{\u64}_1^?} ]` :ref:`matches <match-limits>` the :ref:`limits range <syntax-limits>` :math:`{}[ n_2 \Ldotdot {{\u64}_2^?} ]` if:


   * :math:`n_1` is greater than or equal to :math:`n_2`.

   * Either:

      * :math:`{{\u64}_1^?}` is of the form :math:`m_1`.

      * If :math:`{\u64}_2` is defined, then:

         * :math:`m_1` is less than or equal to :math:`{\u64}_2`.

   * Or:

      * :math:`{{\u64}_1^?}` is absent.

      * :math:`{{\u64}_2^?}` is absent.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   n_1 \geq n_2
    \qquad
   (m_1 \leq m_2)^?
   }{
   C \vdashlimitsmatch {}[ n_1 \Ldotdot m_1 ] \sublimitsmatch {}[ n_2 \Ldotdot {m_2^?} ]
   }
   \qquad
   \frac{
   n_1 \geq n_2
   }{
   C \vdashlimitsmatch {}[ n_1 \Ldotdot \epsilon ] \sublimitsmatch {}[ n_2 \Ldotdot \epsilon ]
   }
   \qquad
   \end{array}


.. index:: tag type
.. _match-tagtype:

Tag Types
~~~~~~~~~




The :ref:`tag type <syntax-tagtype>` :math:`{\deftype}_1` :ref:`matches <match-tagtype>` the :ref:`tag type <syntax-tagtype>` :math:`{\deftype}_2` if:


   * The :ref:`defined type <syntax-deftype>` :math:`{\deftype}_1` :ref:`matches <match-deftype>` the :ref:`defined type <syntax-deftype>` :math:`{\deftype}_2`.

   * The :ref:`defined type <syntax-deftype>` :math:`{\deftype}_2` :ref:`matches <match-deftype>` the :ref:`defined type <syntax-deftype>` :math:`{\deftype}_1`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashdeftypematch {\deftype}_1 \subdeftypematch {\deftype}_2
    \qquad
   C \vdashdeftypematch {\deftype}_2 \subdeftypematch {\deftype}_1
   }{
   C \vdashtagtypematch {\deftype}_1 \subtagtypematch {\deftype}_2
   }
   \qquad
   \end{array}

.. note::
   Although the conclusion of this rule looks identical to its premise,
   they in fact describe different relations:
   the premise invokes subtyping on defined types,
   while the conclusion defines it on tag types that happen to be expressed as defined types.


.. index:: global type, value type, mutability
.. _match-globaltype:

Global Types
~~~~~~~~~~~~




The :ref:`global type <syntax-globaltype>` :math:`({{\TMUT}_1^?}~{\valtype}_1)` :ref:`matches <match-globaltype>` the :ref:`global type <syntax-globaltype>` :math:`({{\TMUT}_2^?}~{\valtype}_2)` if:


   * The :ref:`value type <syntax-valtype>` :math:`{\valtype}_1` :ref:`matches <match-valtype>` the :ref:`value type <syntax-valtype>` :math:`{\valtype}_2`.

   * Either:

      * :math:`{{\TMUT}_1^?}` is absent.

      * :math:`{{\TMUT}_2^?}` is absent.

   * Or:

      * :math:`{{\TMUT}_1^?}` is of the form :math:`\TMUT`.

      * :math:`{{\TMUT}_2^?}` is of the form :math:`\TMUT`.

      * The :ref:`value type <syntax-valtype>` :math:`{\valtype}_2` :ref:`matches <match-valtype>` the :ref:`value type <syntax-valtype>` :math:`{\valtype}_1`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashvaltypematch {\valtype}_1 \subvaltypematch {\valtype}_2
   }{
   C \vdashglobaltypematch {\valtype}_1 \subglobaltypematch {\valtype}_2
   }
   \qquad
   \frac{
   C \vdashvaltypematch {\valtype}_1 \subvaltypematch {\valtype}_2
    \qquad
   C \vdashvaltypematch {\valtype}_2 \subvaltypematch {\valtype}_1
   }{
   C \vdashglobaltypematch \TMUT~{\valtype}_1 \subglobaltypematch \TMUT~{\valtype}_2
   }
   \qquad
   \end{array}


.. index:: memory type, limits
.. _match-memtype:

Memory Types
~~~~~~~~~~~~




The :ref:`memory type <syntax-memtype>` :math:`({\addrtype}~{\limits}_1~\PAGE)` :ref:`matches <match-memtype>` the :ref:`memory type <syntax-memtype>` :math:`({\addrtype}~{\limits}_2~\PAGE)` if:


   * The :ref:`limits range <syntax-limits>` :math:`{\limits}_1` :ref:`matches <match-limits>` the :ref:`limits range <syntax-limits>` :math:`{\limits}_2`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashlimitsmatch {\limits}_1 \sublimitsmatch {\limits}_2
   }{
   C \vdashmemtypematch {\addrtype}~{\limits}_1~\PAGE \submemtypematch {\addrtype}~{\limits}_2~\PAGE
   }
   \qquad
   \end{array}


.. index:: table type, limits, element type
.. _match-tabletype:

Table Types
~~~~~~~~~~~




The :ref:`table type <syntax-tabletype>` :math:`({\addrtype}~{\limits}_1~{\reftype}_1)` :ref:`matches <match-tabletype>` the :ref:`table type <syntax-tabletype>` :math:`({\addrtype}~{\limits}_2~{\reftype}_2)` if:


   * The :ref:`limits range <syntax-limits>` :math:`{\limits}_1` :ref:`matches <match-limits>` the :ref:`limits range <syntax-limits>` :math:`{\limits}_2`.

   * The :ref:`reference type <syntax-reftype>` :math:`{\reftype}_1` :ref:`matches <match-reftype>` the :ref:`reference type <syntax-reftype>` :math:`{\reftype}_2`.

   * The :ref:`reference type <syntax-reftype>` :math:`{\reftype}_2` :ref:`matches <match-reftype>` the :ref:`reference type <syntax-reftype>` :math:`{\reftype}_1`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashlimitsmatch {\limits}_1 \sublimitsmatch {\limits}_2
    \qquad
   C \vdashreftypematch {\reftype}_1 \subreftypematch {\reftype}_2
    \qquad
   C \vdashreftypematch {\reftype}_2 \subreftypematch {\reftype}_1
   }{
   C \vdashtabletypematch {\addrtype}~{\limits}_1~{\reftype}_1 \subtabletypematch {\addrtype}~{\limits}_2~{\reftype}_2
   }
   \qquad
   \end{array}


.. index:: external type, tag type, global type, memory type, table type, function type
.. _match-externtype:

External Types
~~~~~~~~~~~~~~




The :ref:`external type <syntax-externtype>` :math:`(\XTTAG~{\tagtype}_1)` :ref:`matches <match-externtype>` the :ref:`external type <syntax-externtype>` :math:`(\XTTAG~{\tagtype}_2)` if:


   * The :ref:`tag type <syntax-tagtype>` :math:`{\tagtype}_1` :ref:`matches <match-tagtype>` the :ref:`tag type <syntax-tagtype>` :math:`{\tagtype}_2`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashtagtypematch {\tagtype}_1 \subtagtypematch {\tagtype}_2
   }{
   C \vdashexterntypematch \XTTAG~{\tagtype}_1 \subexterntypematch \XTTAG~{\tagtype}_2
   }
   \qquad
   \end{array}





The :ref:`external type <syntax-externtype>` :math:`(\XTGLOBAL~{\globaltype}_1)` :ref:`matches <match-externtype>` the :ref:`external type <syntax-externtype>` :math:`(\XTGLOBAL~{\globaltype}_2)` if:


   * The :ref:`global type <syntax-globaltype>` :math:`{\globaltype}_1` :ref:`matches <match-globaltype>` the :ref:`global type <syntax-globaltype>` :math:`{\globaltype}_2`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashglobaltypematch {\globaltype}_1 \subglobaltypematch {\globaltype}_2
   }{
   C \vdashexterntypematch \XTGLOBAL~{\globaltype}_1 \subexterntypematch \XTGLOBAL~{\globaltype}_2
   }
   \qquad
   \end{array}





The :ref:`external type <syntax-externtype>` :math:`(\XTMEM~{\memtype}_1)` :ref:`matches <match-externtype>` the :ref:`external type <syntax-externtype>` :math:`(\XTMEM~{\memtype}_2)` if:


   * The :ref:`memory type <syntax-memtype>` :math:`{\memtype}_1` :ref:`matches <match-memtype>` the :ref:`memory type <syntax-memtype>` :math:`{\memtype}_2`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashmemtypematch {\memtype}_1 \submemtypematch {\memtype}_2
   }{
   C \vdashexterntypematch \XTMEM~{\memtype}_1 \subexterntypematch \XTMEM~{\memtype}_2
   }
   \qquad
   \end{array}





The :ref:`external type <syntax-externtype>` :math:`(\XTTABLE~{\tabletype}_1)` :ref:`matches <match-externtype>` the :ref:`external type <syntax-externtype>` :math:`(\XTTABLE~{\tabletype}_2)` if:


   * The :ref:`table type <syntax-tabletype>` :math:`{\tabletype}_1` :ref:`matches <match-tabletype>` the :ref:`table type <syntax-tabletype>` :math:`{\tabletype}_2`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashtabletypematch {\tabletype}_1 \subtabletypematch {\tabletype}_2
   }{
   C \vdashexterntypematch \XTTABLE~{\tabletype}_1 \subexterntypematch \XTTABLE~{\tabletype}_2
   }
   \qquad
   \end{array}





The :ref:`external type <syntax-externtype>` :math:`(\XTFUNC~{\deftype}_1)` :ref:`matches <match-externtype>` the :ref:`external type <syntax-externtype>` :math:`(\XTFUNC~{\deftype}_2)` if:


   * The :ref:`defined type <syntax-deftype>` :math:`{\deftype}_1` :ref:`matches <match-deftype>` the :ref:`defined type <syntax-deftype>` :math:`{\deftype}_2`.



.. math::
   \begin{array}{@{}c@{}}\displaystyle
   \frac{
   C \vdashdeftypematch {\deftype}_1 \subdeftypematch {\deftype}_2
   }{
   C \vdashexterntypematch \XTFUNC~{\deftype}_1 \subexterntypematch \XTFUNC~{\deftype}_2
   }
   \qquad
   \end{array}
