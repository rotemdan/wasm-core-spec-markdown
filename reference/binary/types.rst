.. index:: type
   pair: binary format; type

Types
-----

.. note::
   In some places, possible types include both type constructors or types denoted by :ref:`type indices <syntax-typeidx>`.
   Thus, the binary format for type constructors corresponds to the encodings of small negative :math:`{{\sNX}}{N}` values, such that they can unambiguously occur in the same place as (positive) type indices.


.. index:: number type
   pair: binary format; number type
.. _binary-numtype:

Number Types
~~~~~~~~~~~~

:ref:`Number types <syntax-numtype>` are encoded by a single byte.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Bnumtype} & ::= & \mathtt{0x7C} & \quad\Rightarrow\quad{} & \F64 \\
   & & | & \mathtt{0x7D} & \quad\Rightarrow\quad{} & \F32 \\
   & & | & \mathtt{0x7E} & \quad\Rightarrow\quad{} & \I64 \\
   & & | & \mathtt{0x7F} & \quad\Rightarrow\quad{} & \I32 \\
   \end{array}


.. index:: vector type
   pair: binary format; vector type
.. _binary-vectype:

Vector Types
~~~~~~~~~~~~

:ref:`Vector types <syntax-vectype>` are also encoded by a single byte.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Bvectype} & ::= & \mathtt{0x7B} & \quad\Rightarrow\quad{} & \V128 \\
   \end{array}


.. index:: heap type
   pair: binary format; heap type
.. _binary-heaptype:
.. _binary-absheaptype:

Heap Types
~~~~~~~~~~

:ref:`Heap types <syntax-reftype>` are encoded as either a single byte, or as a :ref:`type index <binary-typeidx>` encoded as a positive :ref:`signed integer <binary-sint>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Babsheaptype} & ::= & \mathtt{0x69} & \quad\Rightarrow\quad{} & \EXN \\
   & & | & \mathtt{0x6A} & \quad\Rightarrow\quad{} & \ARRAY \\
   & & | & \mathtt{0x6B} & \quad\Rightarrow\quad{} & \STRUCT \\
   & & | & \mathtt{0x6C} & \quad\Rightarrow\quad{} & \I31 \\
   & & | & \mathtt{0x6D} & \quad\Rightarrow\quad{} & \EQT \\
   & & | & \mathtt{0x6E} & \quad\Rightarrow\quad{} & \ANY \\
   & & | & \mathtt{0x6F} & \quad\Rightarrow\quad{} & \EXTERN \\
   & & | & \mathtt{0x70} & \quad\Rightarrow\quad{} & \FUNCT \\
   & & | & \mathtt{0x71} & \quad\Rightarrow\quad{} & \NONE \\
   & & | & \mathtt{0x72} & \quad\Rightarrow\quad{} & \NOEXTERN \\
   & & | & \mathtt{0x73} & \quad\Rightarrow\quad{} & \NOFUNC \\
   & & | & \mathtt{0x74} & \quad\Rightarrow\quad{} & \NOEXN \\[0.8ex]
   & {\Bheaptype} & ::= & {\mathit{ht}}{:}{\Babsheaptype} & \quad\Rightarrow\quad{} & {\mathit{ht}} \\
   & & | & x{:}{\Bs33} & \quad\Rightarrow\quad{} & x & \quad \mbox{if}~ x \geq 0 \\
   \end{array}

.. note::
   The heap type :math:`\BOT` cannot occur in a module.


.. index:: reference type
   pair: binary format; reference type
.. _binary-reftype:

Reference Types
~~~~~~~~~~~~~~~

:ref:`Reference types <syntax-reftype>` are either encoded by a single byte followed by a :ref:`heap type <binary-heaptype>`, or, as a short form, directly as an :ref:`abstract heap type <binary-absheaptype>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Breftype} & ::= & \mathtt{0x63}~~{\mathit{ht}}{:}{\Bheaptype} & \quad\Rightarrow\quad{} & \REF~\NULL~{\mathit{ht}} \\
   & & | & \mathtt{0x64}~~{\mathit{ht}}{:}{\Bheaptype} & \quad\Rightarrow\quad{} & \REF~{\mathit{ht}} \\
   & & | & {\mathit{ht}}{:}{\Babsheaptype} & \quad\Rightarrow\quad{} & \REF~\NULL~{\mathit{ht}} \\
   \end{array}


.. index:: value type, number type, reference type
   pair: binary format; value type
.. _binary-valtype:

Value Types
~~~~~~~~~~~

:ref:`Value types <syntax-valtype>` are encoded with their respective encoding as a :ref:`number type <binary-numtype>`, :ref:`vector type <binary-vectype>`, or :ref:`reference type <binary-reftype>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Bvaltype} & ::= & {\mathit{nt}}{:}{\Bnumtype} & \quad\Rightarrow\quad{} & {\mathit{nt}} \\
   & & | & {\mathit{vt}}{:}{\Bvectype} & \quad\Rightarrow\quad{} & {\mathit{vt}} \\
   & & | & {\mathit{rt}}{:}{\Breftype} & \quad\Rightarrow\quad{} & {\mathit{rt}} \\
   \end{array}

.. note::
   The value type :math:`\BOT` cannot occur in a module.

   Value types can occur in contexts where :ref:`type indices <syntax-typeidx>` are also allowed, such as in the case of :ref:`block types <binary-blocktype>`.
   Thus, the binary format for types corresponds to the |SignedLEB128|_ :ref:`encoding <binary-sint>` of small negative :math:`{{\sNX}}{N}` values, so that they can coexist with (positive) type indices in the future.


.. index:: result type, value type
   pair: binary format; result type
.. _binary-resulttype:

Result Types
~~~~~~~~~~~~

:ref:`Result types <syntax-resulttype>` are encoded by the respective :ref:`lists <binary-list>` of :ref:`value types <binary-valtype>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Bresulttype} & ::= & {t^\ast}{:}{\Blist}({\Bvaltype}) & \quad\Rightarrow\quad{} & {t^\ast} \\
   \end{array}


.. index:: composite type, aggregate type, structure type, array type, function type, result type, value type, field type, storage type, packed type, mutability
   pair: binary format; composite type
   pair: binary format; aggregate type
   pair: binary format; function type
   pair: binary format; structure type
   pair: binary format; array type
   pair: binary format; field type
   pair: binary format; storage type
   pair: binary format; packed type
.. _binary-mut:
.. _binary-comptype:
.. _binary-aggrtype:
.. _binary-functype:
.. _binary-structtype:
.. _binary-arraytype:
.. _binary-fieldtype:
.. _binary-storagetype:
.. _binary-packtype:

Composite Types
~~~~~~~~~~~~~~~

:ref:`Composite types <syntax-comptype>` are encoded by a distinct byte followed by a type encoding of the respective form.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Bmut} & ::= & \mathtt{0x00} & \quad\Rightarrow\quad{} & \epsilon \\
   & & | & \mathtt{0x01} & \quad\Rightarrow\quad{} & \TMUT \\[0.8ex]
   & {\Bcomptype} & ::= & \mathtt{0x5E}~~{\mathit{ft}}{:}{\Bfieldtype} & \quad\Rightarrow\quad{} & \TARRAY~{\mathit{ft}} \\
   & & | & \mathtt{0x5F}~~{{\mathit{ft}}^\ast}{:}{\Blist}({\Bfieldtype}) & \quad\Rightarrow\quad{} & \TSTRUCT~{{\mathit{ft}}^\ast} \\
   & & | & \mathtt{0x60}~~{t_1^\ast}{:}{\Bresulttype}~~{t_2^\ast}{:}{\Bresulttype} & \quad\Rightarrow\quad{} & \TFUNC~{t_1^\ast} \Tarrow {t_2^\ast} \\[0.8ex]
   & {\Bfieldtype} & ::= & {\mathit{zt}}{:}{\Bstoragetype}~~{\TMUT^?}{:}{\Bmut} & \quad\Rightarrow\quad{} & {\TMUT^?}~{\mathit{zt}} \\[0.8ex]
   & {\Bstoragetype} & ::= & t{:}{\Bvaltype} & \quad\Rightarrow\quad{} & t \\
   & & | & {\mathit{pt}}{:}{\Bpacktype} & \quad\Rightarrow\quad{} & {\mathit{pt}} \\[0.8ex]
   & {\Bpacktype} & ::= & \mathtt{0x77} & \quad\Rightarrow\quad{} & \I16 \\
   & & | & \mathtt{0x78} & \quad\Rightarrow\quad{} & \I8 \\
   \end{array}


.. index:: recursive type, sub type, composite type
   pair: binary format; recursive type
   pair: binary format; sub type
.. _binary-rectype:
.. _binary-subtype:

Recursive Types
~~~~~~~~~~~~~~~

:ref:`Recursive types <syntax-rectype>` are encoded by the byte :math:`\mathtt{0x4E}` followed by a :ref:`list <binary-list>` of :ref:`sub types <syntax-subtype>`.
Additional shorthands are recognized for unary recursions and sub types without super types.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Brectype} & ::= & \mathtt{0x4E}~~{{\mathit{st}}^\ast}{:}{\Blist}({\Bsubtype}) & \quad\Rightarrow\quad{} & \TREC~{{\mathit{st}}^\ast} \\
   & & | & {\mathit{st}}{:}{\Bsubtype} & \quad\Rightarrow\quad{} & \TREC~{\mathit{st}} \\[0.8ex]
   & {\Bsubtype} & ::= & \mathtt{0x4F}~~{x^\ast}{:}{\Blist}({\Btypeidx})~~{\mathit{ct}}{:}{\Bcomptype} & \quad\Rightarrow\quad{} & \TSUB~\TFINAL~{x^\ast}~{\mathit{ct}} \\
   & & | & \mathtt{0x50}~~{x^\ast}{:}{\Blist}({\Btypeidx})~~{\mathit{ct}}{:}{\Bcomptype} & \quad\Rightarrow\quad{} & \TSUB~{x^\ast}~{\mathit{ct}} \\
   & & | & {\mathit{ct}}{:}{\Bcomptype} & \quad\Rightarrow\quad{} & \TSUB~\TFINAL~\epsilon~{\mathit{ct}} \\
   \end{array}


.. index:: limits
   pair: binary format; limits
.. _binary-limits:

Limits
~~~~~~

:ref:`Limits <syntax-limits>` are encoded with a preceding flag indicating whether a maximum is present, and a flag for the :ref:`address type <syntax-addrtype>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Blimits} & ::= & \mathtt{0x00}~~n{:}{\Bu64} & \quad\Rightarrow\quad{} & (\I32, {}[ n \Ldotdot \epsilon ]) \\
   & & | & \mathtt{0x01}~~n{:}{\Bu64}~~m{:}{\Bu64} & \quad\Rightarrow\quad{} & (\I32, {}[ n \Ldotdot m ]) \\
   & & | & \mathtt{0x04}~~n{:}{\Bu64} & \quad\Rightarrow\quad{} & (\I64, {}[ n \Ldotdot \epsilon ]) \\
   & & | & \mathtt{0x05}~~n{:}{\Bu64}~~m{:}{\Bu64} & \quad\Rightarrow\quad{} & (\I64, {}[ n \Ldotdot m ]) \\
   \end{array}


.. index:: tag type, function type, exception tag
   pair: binary format; tag type
.. _binary-tagtype:

Tag Types
~~~~~~~~~

:ref:`Tag types <syntax-tagtype>` are encoded by a :ref:`type index <syntax-typeidx>` denoting a :ref:`function type <syntax-functype>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Btagtype} & ::= & \mathtt{0x00}~~x{:}{\Btypeidx} & \quad\Rightarrow\quad{} & x \\
   \end{array}

.. note::
   In future versions of WebAssembly,
   the preceding zero byte may encode additional attributes.


.. index:: global type, mutability, value type
   pair: binary format; global type
   pair: binary format; mutability
.. _binary-globaltype:

Global Types
~~~~~~~~~~~~

:ref:`Global types <syntax-globaltype>` are encoded by their :ref:`value type <binary-valtype>` and a flag for their :ref:`mutability <syntax-mut>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Bglobaltype} & ::= & t{:}{\Bvaltype}~~{\TMUT^?}{:}{\Bmut} & \quad\Rightarrow\quad{} & {\TMUT^?}~t \\
   \end{array}


.. index:: memory type, limits, page size
   pair: binary format; memory type
.. _binary-memtype:

Memory Types
~~~~~~~~~~~~

:ref:`Memory types <syntax-memtype>` are encoded with their :ref:`limits <binary-limits>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Bmemtype} & ::= & ({\mathit{at}}, {\mathit{lim}}){:}{\Blimits} & \quad\Rightarrow\quad{} & {\mathit{at}}~{\mathit{lim}}~\PAGE \\
   \end{array}


.. index:: table type, reference type, limits
   pair: binary format; table type
.. _binary-tabletype:

Table Types
~~~~~~~~~~~

:ref:`Table types <syntax-tabletype>` are encoded with their :ref:`limits <binary-limits>` and the encoding of their element :ref:`reference type <syntax-reftype>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Btabletype} & ::= & {\mathit{rt}}{:}{\Breftype}~~({\mathit{at}}, {\mathit{lim}}){:}{\Blimits} & \quad\Rightarrow\quad{} & {\mathit{at}}~{\mathit{lim}}~{\mathit{rt}} \\
   \end{array}


.. index:: external type
   pair: binary format; external type
.. _binary-externtype:

External Types
~~~~~~~~~~~~~~

:ref:`External types <syntax-externtype>` are encoded by a distiguishing byte followed by an encoding of the respective form of type.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Bexterntype} & ::= & \mathtt{0x00}~~x{:}{\Btypeidx} & \quad\Rightarrow\quad{} & \XTFUNC~x \\
   & & | & \mathtt{0x01}~~{\mathit{tt}}{:}{\Btabletype} & \quad\Rightarrow\quad{} & \XTTABLE~{\mathit{tt}} \\
   & & | & \mathtt{0x02}~~{\mathit{mt}}{:}{\Bmemtype} & \quad\Rightarrow\quad{} & \XTMEM~{\mathit{mt}} \\
   & & | & \mathtt{0x03}~~{\mathit{gt}}{:}{\Bglobaltype} & \quad\Rightarrow\quad{} & \XTGLOBAL~{\mathit{gt}} \\
   & & | & \mathtt{0x04}~~{\mathit{jt}}{:}{\Btagtype} & \quad\Rightarrow\quad{} & \XTTAG~{\mathit{jt}} \\
   \end{array}
