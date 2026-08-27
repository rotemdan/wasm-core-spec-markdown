.. index:: type
   pair: text format; type
.. _text-type:

Types
-----

.. index:: number type
   pair: text format; number type
.. _text-numtype:

Number Types
~~~~~~~~~~~~

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Tnumtype} & ::= & \mbox{‘\texttt{i32}’} & \quad\Rightarrow\quad{} & \I32 \\
   & & | & \mbox{‘\texttt{i64}’} & \quad\Rightarrow\quad{} & \I64 \\
   & & | & \mbox{‘\texttt{f32}’} & \quad\Rightarrow\quad{} & \F32 \\
   & & | & \mbox{‘\texttt{f64}’} & \quad\Rightarrow\quad{} & \F64 \\
   \end{array}


.. index:: vector type
   pair: text format; vector type
.. _text-vectype:

Vector Types
~~~~~~~~~~~~

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Tvectype} & ::= & \mbox{‘\texttt{v128}’} & \quad\Rightarrow\quad{} & \V128 \\
   \end{array}


.. index:: heap type
   pair: text format; heap type
.. _text-heaptype:
.. _text-absheaptype:

Heap Types
~~~~~~~~~~

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Tabsheaptype} & ::= & \mbox{‘\texttt{any}’} & \quad\Rightarrow\quad{} & \ANY \\
   & & | & \mbox{‘\texttt{eq}’} & \quad\Rightarrow\quad{} & \EQT \\
   & & | & \mbox{‘\texttt{i31}’} & \quad\Rightarrow\quad{} & \I31 \\
   & & | & \mbox{‘\texttt{struct}’} & \quad\Rightarrow\quad{} & \STRUCT \\
   & & | & \mbox{‘\texttt{array}’} & \quad\Rightarrow\quad{} & \ARRAY \\
   & & | & \mbox{‘\texttt{none}’} & \quad\Rightarrow\quad{} & \NONE \\
   & & | & \mbox{‘\texttt{func}’} & \quad\Rightarrow\quad{} & \FUNCT \\
   & & | & \mbox{‘\texttt{nofunc}’} & \quad\Rightarrow\quad{} & \NOFUNC \\
   & & | & \mbox{‘\texttt{exn}’} & \quad\Rightarrow\quad{} & \EXN \\
   & & | & \mbox{‘\texttt{noexn}’} & \quad\Rightarrow\quad{} & \NOEXN \\
   & & | & \mbox{‘\texttt{extern}’} & \quad\Rightarrow\quad{} & \EXTERN \\
   & & | & \mbox{‘\texttt{noextern}’} & \quad\Rightarrow\quad{} & \NOEXTERN \\[0.8ex]
   & {{\Theaptype}}_{{\I}} & ::= & {\mathit{ht}}{:}{\Tabsheaptype} & \quad\Rightarrow\quad{} & {\mathit{ht}} \\
   & & | & x{:}{{\Ttypeidx}}_{{\I}} & \quad\Rightarrow\quad{} & x \\
   \end{array}


.. index:: reference type
   pair: text format; reference type
.. _text-reftype:
.. _text-null:

Reference Types
~~~~~~~~~~~~~~~

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Tnull} & ::= & \mbox{‘\texttt{null}’} & \quad\Rightarrow\quad{} & \NULL \\[0.8ex]
   & {{\Treftype}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{ref}’}~~{\NULL^?}{:}{{\Tnull}^?}~~{\mathit{ht}}{:}{{\Theaptype}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & \REF~{\NULL^?}~{\mathit{ht}} \\
   \end{array}

Abbreviations
.............

There are shorthands for references to abstract heap types.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Treftype}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{anyref}’} & \quad\equiv\quad{} & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{ref}’}~~\mbox{‘\texttt{null}’}~~\mbox{‘\texttt{any}’}~~\mbox{‘\texttt{{)}}’} \\
   & & | & \mbox{‘\texttt{eqref}’} & \quad\equiv\quad{} & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{ref}’}~~\mbox{‘\texttt{null}’}~~\mbox{‘\texttt{eq}’}~~\mbox{‘\texttt{{)}}’} \\
   & & | & \mbox{‘\texttt{i31ref}’} & \quad\equiv\quad{} & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{ref}’}~~\mbox{‘\texttt{null}’}~~\mbox{‘\texttt{i31}’}~~\mbox{‘\texttt{{)}}’} \\
   & & | & \mbox{‘\texttt{structref}’} & \quad\equiv\quad{} & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{ref}’}~~\mbox{‘\texttt{null}’}~~\mbox{‘\texttt{struct}’}~~\mbox{‘\texttt{{)}}’} \\
   & & | & \mbox{‘\texttt{arrayref}’} & \quad\equiv\quad{} & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{ref}’}~~\mbox{‘\texttt{null}’}~~\mbox{‘\texttt{array}’}~~\mbox{‘\texttt{{)}}’} \\
   & & | & \mbox{‘\texttt{nullref}’} & \quad\equiv\quad{} & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{ref}’}~~\mbox{‘\texttt{null}’}~~\mbox{‘\texttt{none}’}~~\mbox{‘\texttt{{)}}’} \\
   & & | & \mbox{‘\texttt{funcref}’} & \quad\equiv\quad{} & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{ref}’}~~\mbox{‘\texttt{null}’}~~\mbox{‘\texttt{func}’}~~\mbox{‘\texttt{{)}}’} \\
   & & | & \mbox{‘\texttt{nullfuncref}’} & \quad\equiv\quad{} & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{ref}’}~~\mbox{‘\texttt{null}’}~~\mbox{‘\texttt{nofunc}’}~~\mbox{‘\texttt{{)}}’} \\
   & & | & \mbox{‘\texttt{exnref}’} & \quad\equiv\quad{} & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{ref}’}~~\mbox{‘\texttt{null}’}~~\mbox{‘\texttt{exn}’}~~\mbox{‘\texttt{{)}}’} \\
   & & | & \mbox{‘\texttt{nullexnref}’} & \quad\equiv\quad{} & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{ref}’}~~\mbox{‘\texttt{null}’}~~\mbox{‘\texttt{noexn}’}~~\mbox{‘\texttt{{)}}’} \\
   & & | & \mbox{‘\texttt{externref}’} & \quad\equiv\quad{} & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{ref}’}~~\mbox{‘\texttt{null}’}~~\mbox{‘\texttt{extern}’}~~\mbox{‘\texttt{{)}}’} \\
   & & | & \mbox{‘\texttt{nullexternref}’} & \quad\equiv\quad{} & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{ref}’}~~\mbox{‘\texttt{null}’}~~\mbox{‘\texttt{noextern}’}~~\mbox{‘\texttt{{)}}’} \\
   \end{array}


.. index:: value type, number type, vector type, reference type
   pair: text format; value type
.. _text-valtype:

Value Types
~~~~~~~~~~~

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tvaltype}}_{{\I}} & ::= & {\mathit{nt}}{:}{\Tnumtype} & \quad\Rightarrow\quad{} & {\mathit{nt}} \\
   & & | & {\mathit{vt}}{:}{\Tvectype} & \quad\Rightarrow\quad{} & {\mathit{vt}} \\
   & & | & {\mathit{rt}}{:}{{\Treftype}}_{{\I}} & \quad\Rightarrow\quad{} & {\mathit{rt}} \\
   \end{array}


.. index:: composite type, aggregate type, structure type, array type, function type, field type, storage type, value type, packed type, mutability, result type
   pair: text format; composite type
   pair: text format; aggregate type
   pair: text format; structure type
   pair: text format; array type
   pair: text format; function type
   pair: text format; field type
   pair: text format; storage type
   pair: text format; packed type
.. _text-comptype:
.. _text-aggrtype:
.. _text-structtype:
.. _text-arraytype:
.. _text-functype:
.. _text-param:
.. _text-result:
.. _text-fieldtype:
.. _text-storagetype:
.. _text-packtype:

Composite Types
~~~~~~~~~~~~~~~

Composite types are parsed into their respective abstract representation,
paired with the local :ref:`identifier context <text-context>` generated by their bound field or parameter identifiers:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tcomptype}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{struct}’}~~{({\mathit{ft}}, {{\mathit{id}}^?})^\ast}{:}{\Tlist}({{\Tfield}}_{{\I}})~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & (\TSTRUCT~{{\mathit{ft}}^\ast}, \{ \IFIELDS~({({{\mathit{id}}^?})^\ast}) \}) \\
   & & | & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{array}’}~~{\mathit{ft}}{:}{{\Tfieldtype}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & (\TARRAY~{\mathit{ft}}, \{ \IFIELDS~(\epsilon) \}) \\
   & & | & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{func}’}~~{(t_1, {{\mathit{id}}^?})^\ast}{:}{\Tlist}({{\Tparam}}_{{\I}})~~{t_2^\ast}{:}{\Tlist}({{\Tresult}}_{{\I}})~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & (\TFUNC~{t_1^\ast} \Tarrow {t_2^\ast}, \{ \IFIELDS~(\epsilon) \}) \\[0.8ex]
   & {{\Tfield}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{field}’}~~{{\mathit{id}}^?}{:}{{\Tid}^?}~~{\mathit{ft}}{:}{{\Tfieldtype}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & ({\mathit{ft}}, {{\mathit{id}}^?}) \\
   & {{\Tparam}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{param}’}~~{{\mathit{id}}^?}{:}{{\Tid}^?}~~t{:}{{\Tvaltype}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & (t, {{\mathit{id}}^?}) \\
   & {{\Tresult}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{result}’}~~t{:}{{\Tvaltype}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & t \\[0.8ex]
   & {{\Tfieldtype}}_{{\I}} & ::= & {\mathit{zt}}{:}{{\Tstoragetype}}_{{\I}} & \quad\Rightarrow\quad{} & {\mathit{zt}} \\
   & & | & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{mut}’}~~{\mathit{zt}}{:}{{\Tstoragetype}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & \TMUT~{\mathit{zt}} \\[0.8ex]
   & {{\Tstoragetype}}_{{\I}} & ::= & t{:}{{\Tvaltype}}_{{\I}} & \quad\Rightarrow\quad{} & t \\
   & & | & {\mathit{pt}}{:}{\Tpacktype} & \quad\Rightarrow\quad{} & {\mathit{pt}} \\[0.8ex]
   & {\Tpacktype} & ::= & \mbox{‘\texttt{i8}’} & \quad\Rightarrow\quad{} & \I8 \\
   & & | & \mbox{‘\texttt{i16}’} & \quad\Rightarrow\quad{} & \I16 \\
   \end{array}

.. note::
   The optional identifier names for parameters in a function type only have documentation purpose.
   They cannot be referenced from anywhere.

Abbreviations
.............

Multiple anonymous structure fields or parameters or multiple results may be combined into a single declaration:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tfield}}_{{\I}} & ::= & \dots ~~|~~ \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{field}’}~~{{{\Tfieldtype}}_{{\I}}^\ast}~~\mbox{‘\texttt{{)}}’} & \quad\equiv\quad{} & {(\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{field}’}~~{{\Tfieldtype}}_{{\I}}~~\mbox{‘\texttt{{)}}’})^\ast} \\
   & {{\Tparam}}_{{\I}} & ::= & \dots ~~|~~ \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{param}’}~~{{{\Tvaltype}}_{{\I}}^\ast}~~\mbox{‘\texttt{{)}}’} & \quad\equiv\quad{} & {(\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{param}’}~~{{\Tvaltype}}_{{\I}}~~\mbox{‘\texttt{{)}}’})^\ast} \\
   & {{\Tresult}}_{{\I}} & ::= & \dots ~~|~~ \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{result}’}~~{{{\Tvaltype}}_{{\I}}^\ast}~~\mbox{‘\texttt{{)}}’} & \quad\equiv\quad{} & {(\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{result}’}~~{{\Tvaltype}}_{{\I}}~~\mbox{‘\texttt{{)}}’})^\ast} \\
   \end{array}


.. index:: recursive type, sub type, composite type
   pair: text format; recursive type
   pair: text format; sub type
.. _text-rectype:
.. _text-subtype:
.. _text-typedef:
.. _text-final:

Recursive Types
~~~~~~~~~~~~~~~

Recursive types are parsed into their respective abstract representation,
paired with the :ref:`identifier context <text-context>` generated by their bound identifiers:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Tfinal} & ::= & \mbox{‘\texttt{final}’} & \quad\Rightarrow\quad{} & \TFINAL \\[0.8ex]
   & {{\Tsubtype}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{sub}’}~~{{\mathit{fin}}^?}{:}{{\Tfinal}^?}~~{x^\ast}{:}{\Tlist}({{\Ttypeidx}}_{{\I}})~~({\mathit{ct}}, {\I'}){:}{{\Tcomptype}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & (\TSUB~{{\mathit{fin}}^?}~{x^\ast}~{\mathit{ct}}, {\I'}) \\[0.8ex]
   & {{\Ttypedef}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{type}’}~~{{\mathit{id}}^?}{:}{{\Tid}^?}~~({\mathit{st}}, {\I'}){:}{{\Tsubtype}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & ({\mathit{st}}, {\I'} \oplus \{ \ITYPES~({{\mathit{id}}^?}) \}) \\[0.8ex]
   & {{\Trectype}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{rec}’}~~{({\mathit{st}}, {\I'})^\ast}{:}{\Tlist}({{\Ttypedef}}_{{\I}})~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & (\TREC~{{\mathit{st}}^\ast}, {\bigcat}\, {{\I'}^\ast}) \\
   \end{array}


Abbreviations
.............

Final sub types with no super-types can omit the :math:`\mbox{‘\texttt{sub}’}` keyword and its arguments:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tsubtype}}_{{\I}} & ::= & \dots ~~|~~ {{\Tcomptype}}_{{\I}} & \quad\equiv\quad{} & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{sub}’}~~\mbox{‘\texttt{final}’}~~{{\Tcomptype}}_{{\I}}~~\mbox{‘\texttt{{)}}’} \\
   \end{array}

Similarly, singular recursive types can omit the :math:`\mbox{‘\texttt{rec}’}` keyword:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Trectype}}_{{\I}} & ::= & \dots ~~|~~ {{\Ttypedef}}_{{\I}} & \quad\equiv\quad{} & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{rec}’}~~{{\Ttypedef}}_{{\I}}~~\mbox{‘\texttt{{)}}’} \\
   \end{array}


.. index:: address type
   pair: text format; address type
.. _text-addrtype:

Address Types
~~~~~~~~~~~~~

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Taddrtype} & ::= & \mbox{‘\texttt{i32}’} & \quad\Rightarrow\quad{} & \I32 \\
   & & | & \mbox{‘\texttt{i64}’} & \quad\Rightarrow\quad{} & \I64 \\
   \end{array}

Abbreviations
.............

The address type can be omitted, in which case it defaults :math:`\I32`:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Taddrtype} & ::= & \dots ~~|~~ \epsilon & \quad\equiv\quad{} & \mbox{‘\texttt{i32}’} \\
   \end{array}


.. index:: limits
   pair: text format; limits
.. _text-limits:

Limits
~~~~~~

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Tlimits} & ::= & n{:}{\Tu64} & \quad\Rightarrow\quad{} & {}[ n \Ldotdot \epsilon ] \\
   & & | & n{:}{\Tu64}~~m{:}{\Tu64} & \quad\Rightarrow\quad{} & {}[ n \Ldotdot m ] \\
   \end{array}


.. index:: tag type, type use
   pair: text format; tag type
.. _text-tagtype:

Tag Types
~~~~~~~~~

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Ttagtype}}_{{\I}} & ::= & (x, {\I'}){:}{{\Ttypeuse}}_{{\I}} & \quad\Rightarrow\quad{} & x \\
   \end{array}


.. index:: global type, mutability, value type
   pair: text format; global type
   pair: text format; mutability
.. _text-globaltype:

Global Types
~~~~~~~~~~~~

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tglobaltype}}_{{\I}} & ::= & t{:}{{\Tvaltype}}_{{\I}} & \quad\Rightarrow\quad{} & t \\
   & & | & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{mut}’}~~t{:}{{\Tvaltype}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & \TMUT~t \\
   \end{array}


.. index:: memory type, limits, page size
   pair: text format; memory type
.. _text-memtype:

Memory Types
~~~~~~~~~~~~

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tmemtype}}_{{\I}} & ::= & {\mathit{at}}{:}{\Taddrtype}~~{\mathit{lim}}{:}{\Tlimits} & \quad\Rightarrow\quad{} & {\mathit{at}}~{\mathit{lim}}~\PAGE \\
   \end{array}


.. index:: table type, reference type, limits
   pair: text format; table type
.. _text-tabletype:

Table Types
~~~~~~~~~~~

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Ttabletype}}_{{\I}} & ::= & {\mathit{at}}{:}{\Taddrtype}~~{\mathit{lim}}{:}{\Tlimits}~~{\mathit{rt}}{:}{{\Treftype}}_{{\I}} & \quad\Rightarrow\quad{} & {\mathit{at}}~{\mathit{lim}}~{\mathit{rt}} \\
   \end{array}


.. index:: external type, tag type, global type, memory type, table type, function type
   pair: text format; external type
.. _text-externtype:

External Types
~~~~~~~~~~~~~~

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Texterntype}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{tag}’}~~{{\mathit{id}}^?}{:}{{\Tid}^?}~~{\mathit{jt}}{:}{{\Ttagtype}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & (\XTTAG~{\mathit{jt}}, \{ \ITAGS~({{\mathit{id}}^?}) \}) \\
   & & | & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{global}’}~~{{\mathit{id}}^?}{:}{{\Tid}^?}~~{\mathit{gt}}{:}{{\Tglobaltype}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & (\XTGLOBAL~{\mathit{gt}}, \{ \IGLOBALS~({{\mathit{id}}^?}) \}) \\
   & & | & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{memory}’}~~{{\mathit{id}}^?}{:}{{\Tid}^?}~~{\mathit{mt}}{:}{{\Tmemtype}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & (\XTMEM~{\mathit{mt}}, \{ \IMEMS~({{\mathit{id}}^?}) \}) \\
   & & | & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{table}’}~~{{\mathit{id}}^?}{:}{{\Tid}^?}~~{\mathit{tt}}{:}{{\Ttabletype}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & (\XTTABLE~{\mathit{tt}}, \{ \ITABLES~({{\mathit{id}}^?}) \}) \\
   & & | & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{func}’}~~{{\mathit{id}}^?}{:}{{\Tid}^?}~~(x, {\I'}){:}{{\Ttypeuse}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & (\XTFUNC~x, \{ \IFUNCS~({{\mathit{id}}^?}) \}) \\
   \end{array}


.. index:: type use
   pair: text format; type use
.. _text-typeuse:

Type Uses
~~~~~~~~~

A *type use* is a reference to a :ref:`type definition <text-type>`.
Where it is required to reference a :ref:`function type <text-functype>`,
it may optionally be augmented by explicit inlined :ref:`parameter <text-param>` and :ref:`result <text-result>` declarations.
That allows binding symbolic :ref:`identifiers <text-id>` to name the :ref:`local indices <text-localidx>` of parameters.
If inline declarations are given, then their types must match the referenced :ref:`function type <text-type>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Ttypeuse}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{type}’}~~x{:}{{\Ttypeidx}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & (x, {\I'}) &  \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ {\I}{.}\ITYPEDEFS{}[x] = (\TREC~{{\mathit{st}}^\ast}) {.} i \\
   {\land}~ {{\mathit{st}}^\ast}{}[i] = \TSUB~\TFINAL~(\TFUNC~{t_1^\ast} \Tarrow {t_2^\ast}) \\
   {\land}~ {\I'} = \{ \ILOCALS~{(\epsilon)^{{|{t_1^\ast}|}}} \} \\
   \end{array}
   } \\
   & & | & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{type}’}~~x{:}{{\Ttypeidx}}_{{\I}}~~\mbox{‘\texttt{{)}}’}~~{(t_1, {{\mathit{id}}^?})^\ast}{:}{{{\Tparam}}_{{\I}}^\ast}~~{t_2^\ast}{:}{{{\Tresult}}_{{\I}}^\ast} & \quad\Rightarrow\quad{} & (x, {\I'}) &  \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ {\I}{.}\ITYPEDEFS{}[x] = (\TREC~{{\mathit{st}}^\ast}) {.} i \\
   {\land}~ {{\mathit{st}}^\ast}{}[i] = \TSUB~\TFINAL~(\TFUNC~{t_1^\ast} \Tarrow {t_2^\ast}) \\
   {\land}~ {\I'} = \{ \ILOCALS~{({{\mathit{id}}^?})^\ast} \} \\
   {\land}~ {\vdash}\, {\I'} : \mathsf{ok} \\
   \end{array}
   } \\
   \end{array}

.. note::
   If inline declarations are given, their types must be *syntactically* equal to the types from the indexed definition;
   possible type :ref:`substitutions <notation-subst>` from other definitions that might make them equal are not taken into account.
   This is to simplify syntactic pre-processing.

The synthesized attribute of a :math:`{\mathtt{typeuse}}` is a pair consisting of both the used :ref:`type index <syntax-typeidx>` and the local :ref:`identifier context <text-context>` containing possible parameter identifiers.

.. note::
   Both productions overlap for the case that the function type is :math:`\TFUNC~\epsilon \Tarrow \epsilon`.
   However, in that case, they also produce the same results, so that the choice is immaterial.

   The :ref:`well-formedness <text-context-wf>` condition on :math:`{\I'}` ensures that the parameters do not contain duplicate identifiers.


.. _text-typeuse-abbrev:

Abbreviations
.............

A type use may also be replaced entirely by inline :ref:`parameter <text-param>` and :ref:`result <text-result>` declarations.
In that case, a :ref:`type index <syntax-typeidx>` is automatically inserted:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Ttypeuse}}_{{\I}} & ::= & \dots ~~|~~ {(t_1, {{\mathit{id}}^?})^\ast}{:}{{{\Tparam}}_{{\I}}^\ast}~~{t_2^\ast}{:}{{{\Tresult}}_{{\I}}^\ast} & \quad\equiv\quad{} & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{type}’}~~x{:}{{\Ttypeidx}}_{{\I}}~~\mbox{‘\texttt{{)}}’}~~{{{\Tparam}}_{{\I}}^\ast}~~{{{\Tresult}}_{{\I}}^\ast} &  \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ {\I}{.}\ITYPEDEFS{}[x] = (\TREC~(\TSUB~\TFINAL~(\TFUNC~{t_1^\ast} \Tarrow {t_2^\ast}))) {.} 0 \\
   {\land}~ ({\I}{.}\ITYPEDEFS{}[i] \neq (\TREC~(\TSUB~\TFINAL~(\TFUNC~{t_1^\ast} \Tarrow {t_2^\ast}))) {.} 0)^{i<x} \\
   \end{array}
   } \\
   \end{array}

where :math:`x` is the smallest existing :ref:`type index <syntax-typeidx>` whose :ref:`recursive type <syntax-rectype>` definition parses into a singular, final :ref:`function type <syntax-functype>` with the same parameters and results.
If no such index exists, then a new :ref:`recursive type <text-rectype>` of the same form is inserted at the end of the module.

Abbreviations are expanded in the order they appear, such that previously inserted type definitions are reused by consecutive expansions.
