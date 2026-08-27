.. index:: ! type, validation, instantiation, execution
   pair: abstract syntax; type

Types
-----

Various entities in WebAssembly are classified by types.
Types are checked during :ref:`validation <valid>`, :ref:`instantiation <exec-instantiation>`, and possibly :ref:`execution <syntax-call_indirect>`.


.. index:: ! number type, integer, floating-point, IEEE 754, bit width, memory
   pair: abstract syntax; number type
   pair: number; type
.. _syntax-numtype:

Number Types
~~~~~~~~~~~~

*Number types* classify numeric values.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\numtype} & ::= & \I32 ~~|~~ \I64 ~~|~~ \F32 ~~|~~ \F64 \\
   \end{array}

The types :math:`\mathsf{i{\scriptstyle 32}}` and :math:`\mathsf{i{\scriptstyle 64}}` classify 32 and 64 bit integers, respectively.
Integers are not inherently signed or unsigned, their interpretation is determined by individual operations.

The types :math:`\mathsf{f{\scriptstyle 32}}` and :math:`\mathsf{f{\scriptstyle 64}}` classify 32 and 64 bit floating-point data, respectively.
They correspond to the respective binary floating-point representations, also known as *single* and *double* precision, as defined by the |IEEE754|_ standard (Section 3.3).

Number types are *transparent*, meaning that their bit patterns can be observed.
Values of number type can be stored in :ref:`memories <syntax-mem>`.

.. _bitwidth-numtype:
.. _bitwidth-valtype:

Conventions
...........

* The notation :math:`{|t|}`` denotes the *bit width* of a number type :math:`t`.
  That is, :math:`{|\mathsf{i{\scriptstyle 32}}|} = {|\mathsf{f{\scriptstyle 32}}|} = 32` and :math:`{|\mathsf{i{\scriptstyle 64}}|} = {|\mathsf{f{\scriptstyle 64}}|} = 64`.




.. index:: ! vector type, integer, floating-point, IEEE 754, bit width, memory, SIMD
   pair: abstract syntax; number type
   pair: number; type
.. _syntax-vectype:

Vector Types
~~~~~~~~~~~~

*Vector types* classify vectors of :ref:`numeric <syntax-numtype>` values processed by vector instructions (also known as *SIMD* instructions, single instruction multiple data).

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\vectype} & ::= & \V128 \\
   \end{array}

The type :math:`\mathsf{v{\scriptstyle 128}}` corresponds to a 128 bit vector of packed integer or floating-point data. The packed data
can be interpreted as signed or unsigned integers, single or double precision floating-point
values, or a single 128 bit type. The interpretation is determined by individual operations.

Vector types, like :ref:`number types <syntax-numtype>` are *transparent*, meaning that their bit patterns can be observed.
Values of vector type can be stored in :ref:`memories <syntax-mem>`.

.. _bitwidth-vectype:

Conventions
...........

* The notation :math:`{|t|}` for :ref:`bit width <bitwidth-numtype>` extends to vector types as well, that is, :math:`{|\mathsf{v{\scriptstyle 128}}|} = 128`.




.. index:: ! type use, type index
   pair: abstract syntax; type use
.. _syntax-typeuse:

Type Uses
~~~~~~~~~

A *type use* is the use site of a :ref:`type index <syntax-typeidx>` referencing a :ref:`composite type <syntax-comptype>` :ref:`defined <syntax-type>` in a :ref:`module <syntax-module>`.
It classifies objects of the respective type.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\typeuse} & ::= & {\typeidx} ~~|~~ \dots \\
   \end{array}

The syntax of type uses is :ref:`extended <syntax-typeuse-ext>` with additional forms for the purpose of specifying :ref:`validation <valid>` and :ref:`execution <exec>`.


.. index:: ! heap type, store, type use, ! abstract type, ! concrete type, ! unboxed scalar
   pair: abstract syntax; heap type
.. _type-abstract:
.. _type-concrete:
.. _syntax-i31:
.. _syntax-heaptype:
.. _syntax-absheaptype:

Heap Types
~~~~~~~~~~

*Heap types* classify objects in the runtime :ref:`store <store>`.
There are three disjoint hierarchies of heap types:

- *function types* classify :ref:`functions <syntax-func>`,
- *aggregate types* classify dynamically allocated *managed* data, such as *structures*, *arrays*, or *unboxed scalars*,
- *external types* classify *external* references possibly owned by the :ref:`embedder <embedder>`.

The values from the latter two hierarchies are interconvertible by ways of the :math:`\EXTERNCONVERTANY` and :math:`\ANYCONVERTEXTERN` instructions.
That is, both type hierarchies are inhabited by an isomorphic set of values, but may have different, incompatible representations in practice.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\absheaptype} & ::= & \ANY ~~|~~ \EQT ~~|~~ \I31 ~~|~~ \STRUCT ~~|~~ \ARRAY ~~|~~ \NONE \\
   & & | & \FUNCT ~~|~~ \NOFUNC \\
   & & | & \EXN ~~|~~ \NOEXN \\
   & & | & \EXTERN ~~|~~ \NOEXTERN \\
   & & | & \dots \\
   & {\heaptype} & ::= & {\absheaptype} ~~|~~ {\typeuse} \\
   \end{array}

A heap type is either *abstract* or *concrete*.
A concrete heap type consists of a :ref:`type use <syntax-typeuse>` that classifies an object of the respective :ref:`type <syntax-type>` defined in a module.
Abstract types are denoted by individual keywords.

The type :math:`\mathsf{func}` denotes the common supertype of all :ref:`function types <syntax-functype>`, regardless of their concrete definition.
Dually, the type :math:`\mathsf{nofunc}` denotes the common subtype of all :ref:`function types <syntax-functype>`, regardless of their concrete definition.
This type has no values.

The type :math:`\mathsf{exn}` denotes the common supertype of all :ref:`exception references <syntax-ref.exn>`.
This type has no concrete subtypes.
Dually, the type :math:`\mathsf{noexn}` denotes the common subtype of all forms of exception references.
This type has no values.

The type :math:`\mathsf{extern}` denotes the common supertype of all external references received through the :ref:`embedder <embedder>`.
This type has no concrete subtypes.
Dually, the type :math:`\mathsf{noextern}` denotes the common subtype of all forms of external references.
This type has no values.

The type :math:`\mathsf{any}` denotes the common supertype of all aggregate types, as well as possibly abstract values produced by *internalizing* an external reference of type :math:`\mathsf{extern}`.
Dually, the type :math:`\mathsf{none}` denotes the common subtype of all forms of aggregate types.
This type has no values.

The type :math:`\mathsf{eq}` is a subtype of :math:`\mathsf{any}` that includes all types for which references can be compared, i.e., aggregate values and :math:`\mathsf{i{\scriptstyle 31}}`.

The types :math:`\mathsf{struct}` and :math:`\mathsf{array}` denote the common supertypes of all :ref:`structure <syntax-structtype>` and :ref:`array <syntax-arraytype>` aggregates, respectively.

The type :math:`\mathsf{i{\scriptstyle 31}}` denotes *unboxed scalars*, that is, integers injected into references.
Their observable value range is limited to 31 bits.

.. note::
   Values of type :math:`\mathsf{i{\scriptstyle 31}}` are not actually allocated in the store,
   but represented in a way that allows them to be mixed with actual references into the store without ambiguity.
   Engines need to perform some form of *pointer tagging* to achieve this,
   which is why one bit is reserved.
   Since this type is to be reliably unboxed on all hardware platforms supported by WebAssembly,
   it cannot be wider than 32 bits minus the tag bit.

   Although the types :math:`\mathsf{none}`, :math:`\mathsf{nofunc}`, :math:`\mathsf{noexn}`, and :math:`\mathsf{noextern}` are not inhabited by any values,
   they can be used to form the types of all null :ref:`references <syntax-reftype>` in their respective hierarchy.
   For example, :math:`(\mathsf{ref}~\mathsf{null}~\mathsf{nofunc})` is the generic type of a null reference compatible with all function reference types.

The syntax of abstract heap types is :ref:`extended <syntax-heaptype-ext>` with additional forms for the purpose of specifying :ref:`validation <valid>` and :ref:`execution <exec>`.


.. index:: ! reference type, heap type, reference, table, function, function type, null
   pair: abstract syntax; reference type
   pair: reference; type
.. _syntax-reftype:
.. _syntax-null:

Reference Types
~~~~~~~~~~~~~~~

*Reference types* classify :ref:`values <syntax-value>` that are first-class references to objects in the runtime :ref:`store <store>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\reftype} & ::= & \REF~{\NULL^?}~{\heaptype} \\
   \end{array}

A reference type is characterised by the :ref:`heap type <syntax-heaptype>` it points to.

In addition, a reference type of the form :math:`\mathsf{ref}~\mathsf{null}~{\mathit{ht}}` is *nullable*, meaning that it can either be a proper reference to :math:`{\mathit{ht}}` or :ref:`null <syntax-null>`.
Other references are *non-null*.

Reference types are *opaque*, meaning that neither their size nor their bit pattern can be observed.
Values of reference type can be stored in :ref:`tables <syntax-table>` but not in :ref:`memories <syntax-mem>`.

Conventions
...........

* The reference type :math:`\ANYREF` is an abbreviation for :math:`(\REF~\NULL~\ANY)`.

* The reference type :math:`\EQREF` is an abbreviation for :math:`(\REF~\NULL~\EQT)`.

* The reference type :math:`\I31REF` is an abbreviation for :math:`(\REF~\NULL~\I31)`.

* The reference type :math:`\STRUCTREF` is an abbreviation for :math:`(\REF~\NULL~\STRUCT)`.

* The reference type :math:`\ARRAYREF` is an abbreviation for :math:`(\REF~\NULL~\ARRAY)`.

* The reference type :math:`\FUNCREF` is an abbreviation for :math:`(\REF~\NULL~\FUNCT)`.

* The reference type :math:`\EXNREF` is an abbreviation for :math:`(\REF~\NULL~\EXN)`.

* The reference type :math:`\EXTERNREF` is an abbreviation for :math:`(\REF~\NULL~\EXTERN)`.

* The reference type :math:`\NULLREF` is an abbreviation for :math:`(\REF~\NULL~\NONE)`.

* The reference type :math:`\NULLFUNCREF` is an abbreviation for :math:`(\REF~\NULL~\NOFUNC)`.

* The reference type :math:`\NULLEXNREF` is an abbreviation for :math:`(\REF~\NULL~\NOEXN)`.

* The reference type :math:`\NULLEXTERNREF` is an abbreviation for :math:`(\REF~\NULL~\NOEXTERN)`.


.. index:: ! value type, number type, vector type, reference type
   pair: abstract syntax; value type
   pair: value; type
.. _syntax-valtype:
.. _syntax-consttype:

Value Types
~~~~~~~~~~~

*Value types* classify the individual values that WebAssembly code can compute with and the values that a variable accepts.
They are either :ref:`number types <syntax-numtype>`, :ref:`vector types <syntax-vectype>`, or :ref:`reference types <syntax-reftype>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\consttype} & ::= & {\numtype} ~~|~~ {\vectype} \\
   & {\valtype} & ::= & {\numtype} ~~|~~ {\vectype} ~~|~~ {\reftype} ~~|~~ \dots \\
   \end{array}

The syntax of value types is :ref:`extended <syntax-valtype-ext>` with additional forms for the purpose of specifying :ref:`validation <valid>`.

Conventions
...........

* The meta variable :math:`t` ranges over value types or subclasses thereof where clear from context.


.. index:: ! result type, value type, list, instruction, execution, function
   pair: abstract syntax; result type
   pair: result; type
.. _syntax-resulttype:

Result Types
~~~~~~~~~~~~

*Result types* classify the result of :ref:`executing <exec-instr>` :ref:`instructions <syntax-instr>` or :ref:`functions <syntax-func>`,
which is a sequence of values, written with brackets.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\resulttype} & ::= & {\list}({\valtype}) \\
   \end{array}


.. index:: ! block type, block, type index, function type, value type, type index
   pair: abstract syntax; block type
   pair: block; type
   pair: type; block
.. _syntax-blocktype:

Block Types
~~~~~~~~~~~

*Block types* classify the *input* and *output* of structured :ref:`control instructions <syntax-instr-control>` delimiting :ref:`blocks <syntax-block>` of instructions.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\blocktype} & ::= & {{\valtype}^?} \\
   & & | & {\typeidx} \\
   \end{array}

They are given either as a :ref:`type index <syntax-funcidx>` that refers to a suitable :ref:`function type <syntax-functype>` reinterpreted as an :ref:`instruction type <syntax-instrtype>`,
or as an optional :ref:`value type <syntax-valtype>` inline,
which is a shorthand for the instruction type :math:`\epsilon \rightarrow {{\valtype}^?}`.


.. index:: ! composite type, ! function type, result type, ! aggregate type, ! structure type, ! array type, ! field type, ! storage type, ! packed type, bit width, function, structure, array, parameter, result, list
   pair: abstract syntax; composite type
   pair: abstract syntax; function type
   pair: abstract syntax; structure type
   pair: abstract syntax; array type
   pair: abstract syntax; field type
   pair: abstract syntax; storage type
   pair: abstract syntax; packed type
   pair: function; type
   pair: structure; type
   pair: array; type
.. _syntax-comptype:
.. _syntax-functype:
.. _syntax-aggrtype:
.. _syntax-structtype:
.. _syntax-arraytype:
.. _syntax-fieldtype:
.. _syntax-storagetype:
.. _syntax-packtype:

Composite Types
~~~~~~~~~~~~~~~

*Composite types* are all types composed from simpler types,
including *function types*, *structure types* and *array types*.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\comptype} & ::= & \TSTRUCT~{\list}({\fieldtype}) \\
   & & | & \TARRAY~{\fieldtype} \\
   & & | & \TFUNC~{\resulttype} \Tarrow {\resulttype} \\[0.8ex]
   & {\fieldtype} & ::= & {\TMUT^?}~{\storagetype} \\
   & {\storagetype} & ::= & {\valtype} ~~|~~ {\packtype} \\
   & {\packtype} & ::= & \I8 ~~|~~ \I16 \\
   \end{array}

Function types classify the signature of :ref:`functions <syntax-func>`,
mapping a list of parameters to a list of results.
They are also used to classify the inputs and outputs of :ref:`instructions <syntax-instr>`.

*Aggregate types* like structure or array types consist of a list of possibly mutable, possibly packed *field types* describing their components.
Structures are heterogeneous, but require static indexing, while arrays need to be homogeneous, but allow dynamic indexing.

.. _bitwidth-fieldtype:
.. _aux-unpack:

Conventions
...........

* The notation :math:`{|t|}` for the :ref:`bit width <bitwidth-valtype>` of a :ref:`value type <syntax-valtype>` :math:`t` extends to packed types as well, that is, :math:`{|\mathsf{i{\scriptstyle 8}}|} = 8` and :math:`{|\mathsf{i{\scriptstyle 16}}|} = 16`.



* The auxiliary function :math:`\unpack` maps a storage type to the :ref:`value type <syntax-valtype>` obtained when accessing a field:

.. math::
   \begin{array}[t]{@{}lcl@{}l@{}}
   {\unpack}({\valtype}) & = & {\valtype} \\
   {\unpack}({\packtype}) & = & \I32 \\
   \end{array}


.. index:: ! recursive type, ! sub type, composite type, ! final, subtyping, ! roll, ! unroll, recursive type index
   pair: abstract syntax; recursive type
   pair: abstract syntax; sub type
.. _syntax-rectype:
.. _syntax-subtype:
.. _syntax-final:

Recursive Types
~~~~~~~~~~~~~~~

*Recursive types* denote a group of mutually recursive :ref:`composite types <syntax-comptype>`, each of which can optionally declare a list of :ref:`type uses <syntax-typeuse>` of supertypes that it :ref:`matches <match-comptype>`.
Each type can also be declared *final*, preventing further subtyping.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\rectype} & ::= & \TREC~{\list}({\subtype}) \\
   & {\subtype} & ::= & \TSUB~{\TFINAL^?}~{{\typeuse}^\ast}~{\comptype} \\
   \end{array}

In a :ref:`module <syntax-module>`, each member of a recursive type is assigned a separate :ref:`type index <syntax-typeidx>`.


.. _index:: ! address type, number type, bit width
   pair: abstract syntax; address type
   single: memory; address type
   single: table; address type
.. _syntax-addrtype:

Address Types
~~~~~~~~~~~~~

*Address types* are a subset of :ref:`number types <syntax-numtype>` that classify the values that can be used as offsets into
:ref:`memories <syntax-mem>` and :ref:`tables <syntax-table>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\addrtype} & ::= & \I32 ~~|~~ \I64 \\
   \end{array}

.. _aux-addrtype-min:

Conventions
...........

The *minimum* of two address types is defined as the address type whose :ref:`bit width <bitwidth-numtype>` is the minimum of the two.

.. math::
   \begin{array}[t]{@{}lcl@{}l@{}}
   {\addrtypemin}({\mathit{at}}_1, {\mathit{at}}_2) & = & {\mathit{at}}_1 & \quad \mbox{if}~ {|{\mathit{at}}_1|} \leq {|{\mathit{at}}_2|} \\
   {\addrtypemin}({\mathit{at}}_1, {\mathit{at}}_2) & = & {\mathit{at}}_2 & \quad \mbox{otherwise} \\
   \end{array}


.. index:: ! limits, memory type, table type
   pair: abstract syntax; limits
   single: memory; limits
   single: table; limits
.. _syntax-limits:

Limits
~~~~~~

*Limits* classify the size range of resizeable storage associated with :ref:`memory types <syntax-memtype>` and :ref:`table types <syntax-tabletype>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\limits} & ::= & {}[ {\u64} \Ldotdot {{\u64}^?} ] \\
   \end{array}

If no maximum is present,
then the respective storage can grow to any valid size.


.. index:: ! tag type, type use, tag, function type, exception tag
   pair: abstract syntax; tag type
   pair: tag; type
.. _syntax-tagtype:

Tag Types
~~~~~~~~~

*Tag types* classify the signature :ref:`tags <syntax-tag>`
with a :ref:`type use <syntax-typeuse>` referring to the definition of a :ref:`function type <syntax-functype>` that declares the types of parameter and result values associated with the tag.
The result type is empty for exception tags.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\tagtype} & ::= & {\typeuse} \\
   \end{array}


.. index:: ! global type, ! mutability, value type, global, mutability
   pair: abstract syntax; global type
   pair: abstract syntax; mutability
   pair: global; type
   pair: global; mutability
.. _syntax-mut:
.. _syntax-globaltype:

Global Types
~~~~~~~~~~~~

*Global types* classify :ref:`global <syntax-global>` variables, which hold a value and can either be mutable or immutable.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\globaltype} & ::= & {\TMUT^?}~{\valtype} \\
   \end{array}


.. index:: ! memory type, limits, page size, memory
   pair: abstract syntax; memory type
   pair: memory; type
   pair: memory; limits
.. _syntax-memtype:

Memory Types
~~~~~~~~~~~~

*Memory types* classify linear :ref:`memories <syntax-mem>` and their size range.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\memtype} & ::= & {\addrtype}~{\limits}~\PAGE \\
   \end{array}

The limits constrain the minimum and optionally the maximum size of a memory.
The limits are given in units of :ref:`page size <page-size>`.


.. index:: ! table type, reference type, limits, table, element
   pair: abstract syntax; table type
   pair: table; type
   pair: table; limits
.. _syntax-tabletype:

Table Types
~~~~~~~~~~~

*Table types* classify :ref:`tables <syntax-table>` over elements of :ref:`reference type <syntax-reftype>` within a size range.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\tabletype} & ::= & {\addrtype}~{\limits}~{\reftype} \\
   \end{array}

Like memories, tables are constrained by limits for their minimum and optionally maximum size.
The limits are given in numbers of entries.


.. index:: ! data type, memory
   pair: abstract syntax; data type
   pair: data; type
.. _syntax-datatype:

Data Types
~~~~~~~~~~

*Data types* classify :ref:`data segments <syntax-elem>`.
Since the contents of a data segment requires no further classification, they merely consist of a universal marker :math:`\mathsf{ok}` indicating well-formedness.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\datatype} & ::= & \OKdata \\
   \end{array}


.. index:: ! element type, reference type, table, element
   pair: abstract syntax; element type
   pair: element; type
.. _syntax-elemtype:

Element Types
~~~~~~~~~~~~~

*Element types* classify :ref:`element segments <syntax-elem>` by the :ref:`reference type <syntax-reftype>` of its elements.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\elemtype} & ::= & {\reftype} \\
   \end{array}


.. index:: ! external type, defined type, function type, table type, memory type, global type, tag type, import, external address
   pair: abstract syntax; external type
   pair: external; type
.. _syntax-externtype:

External Types
~~~~~~~~~~~~~~

*External types* classify :ref:`imports <syntax-import>` and :ref:`external addresses <syntax-externaddr>` with their respective types.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\externtype} & ::= & \XTTAG~{\tagtype} ~~|~~ \XTGLOBAL~{\globaltype} ~~|~~ \XTMEM~{\memtype} ~~|~~ \XTTABLE~{\tabletype} ~~|~~ \XTFUNC~{\typeuse} \\
   \end{array}

For functions, the :ref:`type use <syntax-typeuse>` has to refer to the definition of a :ref:`function type <syntax-functype>`.

.. note::
   Future versions of WebAssembly may have additional uses for tags, and may allow non-empty result types in the function types of tags.


Conventions
...........

The following auxiliary notation is defined for sequences of external types.
It filters out entries of a specific kind in an order-preserving fashion:

.. math::
   \begin{array}[t]{@{}lcl@{}l@{}}
   {\funcsxt}(\epsilon) & = & \epsilon \\
   {\funcsxt}((\XTFUNC~{\mathit{dt}})~{{\mathit{xt}}^\ast}) & = & {\mathit{dt}}~{\funcsxt}({{\mathit{xt}}^\ast}) \\
   {\funcsxt}({\externtype}~{{\mathit{xt}}^\ast}) & = & {\funcsxt}({{\mathit{xt}}^\ast}) & \quad \mbox{otherwise} \\[0.8ex]
   {\tablesxt}(\epsilon) & = & \epsilon \\
   {\tablesxt}((\XTTABLE~{\mathit{tt}})~{{\mathit{xt}}^\ast}) & = & {\mathit{tt}}~{\tablesxt}({{\mathit{xt}}^\ast}) \\
   {\tablesxt}({\externtype}~{{\mathit{xt}}^\ast}) & = & {\tablesxt}({{\mathit{xt}}^\ast}) & \quad \mbox{otherwise} \\[0.8ex]
   {\memsxt}(\epsilon) & = & \epsilon \\
   {\memsxt}((\XTMEM~{\mathit{mt}})~{{\mathit{xt}}^\ast}) & = & {\mathit{mt}}~{\memsxt}({{\mathit{xt}}^\ast}) \\
   {\memsxt}({\externtype}~{{\mathit{xt}}^\ast}) & = & {\memsxt}({{\mathit{xt}}^\ast}) & \quad \mbox{otherwise} \\[0.8ex]
   {\globalsxt}(\epsilon) & = & \epsilon \\
   {\globalsxt}((\XTGLOBAL~{\mathit{gt}})~{{\mathit{xt}}^\ast}) & = & {\mathit{gt}}~{\globalsxt}({{\mathit{xt}}^\ast}) \\
   {\globalsxt}({\externtype}~{{\mathit{xt}}^\ast}) & = & {\globalsxt}({{\mathit{xt}}^\ast}) & \quad \mbox{otherwise} \\[0.8ex]
   {\tagsxt}(\epsilon) & = & \epsilon \\
   {\tagsxt}((\XTTAG~{\mathit{jt}})~{{\mathit{xt}}^\ast}) & = & {\mathit{jt}}~{\tagsxt}({{\mathit{xt}}^\ast}) \\
   {\tagsxt}({\externtype}~{{\mathit{xt}}^\ast}) & = & {\tagsxt}({{\mathit{xt}}^\ast}) & \quad \mbox{otherwise} \\
   \end{array}
