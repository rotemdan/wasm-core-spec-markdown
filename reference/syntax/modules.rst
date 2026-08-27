.. index:: ! module, type definition, tag, global, memory, table, function, data, element, start function, import, export
   pair: abstract syntax; module
.. _syntax-module:

Modules
-------

WebAssembly programs are organized into *modules*,
which are the unit of deployment, loading, and compilation.
A module collects definitions for
:ref:`types <syntax-type>`,
:ref:`tags <syntax-tag>`, and
:ref:`globals <syntax-global>`,
:ref:`memories <syntax-mem>`,
:ref:`tables <syntax-table>`,
:ref:`functions <syntax-func>`.
In addition, it can declare
:ref:`imports <syntax-import>` and :ref:`exports <syntax-export>`
and provide initialization in the form of
:ref:`data <syntax-data>` and :ref:`element <syntax-elem>` segments,
or a :ref:`start function <syntax-start>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\module} & ::= & \begin{array}[t]{@{}l@{}} \MODULE \\
     {\list}({\type}) \\
     {\list}({\import}) \\
     {\list}({\tag}) \\
     {\list}({\global}) \\
     {\list}({\mem}) \\
     {\list}({\table}) \\
     {\list}({\func}) \\
     {\list}({\data}) \\
     {\list}({\elem}) \\
     {{\start}^?} \\
     {\list}({\export}) \end{array} \\
   \end{array}

Each of the lists --- and thus the entire module --- may be empty.


.. index:: ! index, ! index space, ! type index, ! tag index, ! global index, ! memory index, ! table index, ! function index, ! local index, ! label index, ! data index, ! element index, ! field index, tag, global, memory, table, function, data, element, local, parameter, import, field
   pair: abstract syntax; type index
   pair: abstract syntax; tag index
   pair: abstract syntax; global index
   pair: abstract syntax; memory index
   pair: abstract syntax; table index
   pair: abstract syntax; function index
   pair: abstract syntax; data index
   pair: abstract syntax; element index
   pair: abstract syntax; local index
   pair: abstract syntax; label index
   pair: abstract syntax; field index
   pair: type; index
   pair: tag; index
   pair: global; index
   pair: memory; index
   pair: table; index
   pair: function; index
   pair: data; index
   pair: element; index
   pair: local; index
   pair: label; index
   pair: field; index
.. _syntax-idx:
.. _syntax-typeidx:
.. _syntax-tagidx:
.. _syntax-globalidx:
.. _syntax-memidx:
.. _syntax-tableidx:
.. _syntax-funcidx:
.. _syntax-dataidx:
.. _syntax-elemidx:
.. _syntax-localidx:
.. _syntax-labelidx:
.. _syntax-fieldidx:
.. _syntax-index:

Indices
~~~~~~~

Definitions are referenced with zero-based *indices*.
Each class of definition has its own *index space*, as distinguished by the following classes.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\idx} & ::= & {\u32} \\
   & {\typeidx} & ::= & {\idx} \\
   & {\funcidx} & ::= & {\idx} \\
   & {\globalidx} & ::= & {\idx} \\
   & {\tableidx} & ::= & {\idx} \\
   & {\memidx} & ::= & {\idx} \\
   & {\tagidx} & ::= & {\idx} \\
   & {\elemidx} & ::= & {\idx} \\
   & {\dataidx} & ::= & {\idx} \\
   & {\labelidx} & ::= & {\idx} \\
   & {\localidx} & ::= & {\idx} \\
   & {\fieldidx} & ::= & {\idx} \\
   \end{array}

The index space for
:ref:`tags <syntax-tag>`,
:ref:`globals <syntax-global>`,
:ref:`memories <syntax-mem>`,
:ref:`tables <syntax-table>`, and
:ref:`functions <syntax-func>`
includes respective :ref:`imports <syntax-import>` declared in the same module.
The indices of these imports precede the indices of other definitions in the same index space.

Data indices reference :ref:`data segments <syntax-data>` and element indices reference :ref:`element segments <syntax-elem>`.

The index space for :ref:`locals <syntax-local>` is only accessible inside a :ref:`function <syntax-func>` and includes the parameters of that function, which precede the local variables.

Label indices reference :ref:`structured control instructions <syntax-instr-control>` inside an instruction sequence.

Each :ref:`aggregate type <syntax-aggrtype>` provides an index space for its :ref:`fields <syntax-fieldtype>`.


Conventions
...........

* The meta variable :math:`l` ranges over label indices.

* The meta variables :math:`x`, :math:`y` range over indices in any of the other index spaces.

.. _free-typeidx:
.. _free-tagidx:
.. _free-globalidx:
.. _free-memidx:
.. _free-tableidx:
.. _free-funcidx:
.. _free-dataidx:
.. _free-elemidx:
.. _free-localidx:
.. _free-labelidx:
.. _free-fieldidx:
.. _free-index:

* For every index space :math:`{\mathit{abcidx}}`, the notation :math:`{\mathrm{abcidx}}(A)` denotes the set of indices from that index space occurring free in :math:`A`. Sometimes this set is reinterpreted as the :ref:`list <syntax-list>` of its elements.

.. note::
   For example, if :math:`{{\instr}^\ast}` is :math:`(\DATADROP~1)~(\MEMORYINIT~2~3)`, then :math:`{\mathrm{dataidx}}_{\mathit{instrs}}({{\instr}^\ast}) = 1~3`, or equivalently, the set :math:`\{ 1, 3 \}`.


.. index:: ! type definition, type index, function type, aggregate type
   pair: abstract syntax; type definition
.. _syntax-type:
.. _syntax-typedef:

Types
~~~~~

The :math:`{\type}` section of a module defines a list of :ref:`recursive types <syntax-rectype>`, each consisting of a list of :ref:`sub types <syntax-subtype>` referenced by individual :ref:`type indices <syntax-typeidx>`.
All :ref:`function <syntax-functype>`, :ref:`structure <syntax-structtype>`, or :ref:`array <syntax-arraytype>` types used in a module must be defined in this section.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\type} & ::= & \TYPE~{\rectype} \\
   \end{array}


.. index:: ! tag, type index, tag type
   pair: abstract syntax; tag
.. _syntax-tag:

Tags
~~~~

The :math:`{\tag}` section of a module defines a list of *tags*:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\tag} & ::= & \TAG~{\tagtype} \\
   \end{array}

The :ref:`type index <syntax-typeidx>` of a tag must refer to a :ref:`function type <syntax-functype>` that declares its :ref:`tag type <syntax-tagtype>`.

Tags are referenced through :ref:`tag indices <syntax-tagidx>`,
starting with the smallest index not referencing a tag :ref:`import <syntax-import>`.


.. index:: ! global, global index, global type, mutability, expression, constant, value, import
   pair: abstract syntax; global
.. _syntax-global:

Globals
~~~~~~~

The :math:`{\global}` section of a module defines a list of *global variables* (or *globals* for short):

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\global} & ::= & \GLOBAL~{\globaltype}~{\expr} \\
   \end{array}

Each global stores a single value of the type specified in the :ref:`global type <syntax-globaltype>`.
It also specifies whether a global is immutable or mutable.
Moreover, each global is initialized with a value given by a :ref:`constant <valid-constant>` initializer :ref:`expression <syntax-expr>`.

Globals are referenced through :ref:`global indices <syntax-globalidx>`,
starting with the smallest index not referencing a global :ref:`import <syntax-import>`.


.. index:: ! memory, memory index, memory type, limits, page size, data, import
   pair: abstract syntax; memory
.. _syntax-mem:

Memories
~~~~~~~~

The :math:`{\mem}` section of a module defines a list of *linear memories* (or *memories* for short) as described by their :ref:`memory type <syntax-memtype>`:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\mem} & ::= & \MEMORY~{\memtype} \\
   \end{array}

A memory is a list of raw uninterpreted bytes.
The minimum size in the :ref:`limits <syntax-limits>` of its :ref:`memory type <syntax-memtype>` specifies the initial size of that memory, while its maximum, if present, restricts the size to which it can grow later.
Both are in units of :ref:`page size <page-size>`.

Memories can be initialized through :ref:`data segments <syntax-data>`.

Memories are referenced through :ref:`memory indices <syntax-memidx>`,
starting with the smallest index not referencing a memory :ref:`import <syntax-import>`.
Most constructs implicitly reference memory index :math:`0`.


.. index:: ! table, table index, table type, limits, element, import
   pair: abstract syntax; table
.. _syntax-table:

Tables
~~~~~~

The :math:`{\table}` section of a module defines a list of *tables* described by their :ref:`table type <syntax-tabletype>`:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\table} & ::= & \TABLE~{\tabletype}~{\expr} \\
   \end{array}

A table is an array of opaque values of a particular :ref:`reference type <syntax-reftype>` that is specified by the :ref:`table type <syntax-tabletype>`.
Each table slot is initialized with a value given by a :ref:`constant <valid-constant>` initializer :ref:`expression <syntax-expr>`.
Tables can further be initialized through :ref:`element segments <syntax-elem>`.

The minimum size in the :ref:`limits <syntax-limits>` of the table type specifies the initial size of that table, while its maximum restricts the size to which it can grow later.

Tables are referenced through :ref:`table indices <syntax-tableidx>`,
starting with the smallest index not referencing a table :ref:`import <syntax-import>`.
Most constructs implicitly reference table index :math:`0`.


.. index:: ! function, ! local, function index, local index, type index, value type, expression, import
   pair: abstract syntax; function
   pair: abstract syntax; local
.. _syntax-local:
.. _syntax-func:

Functions
~~~~~~~~~

The :math:`{\func}` section of a module defines a list of *functions* with the following structure:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\func} & ::= & \FUNC~{\typeidx}~{{\local}^\ast}~{\expr} \\
   & {\local} & ::= & \LOCAL~{\valtype} \\
   \end{array}

The :ref:`type index <syntax-typeidx>` of a function declares its signature by reference to a :ref:`function type <syntax-functype>` defined in the module.
The parameters of the function are referenced through 0-based :ref:`local indices <syntax-localidx>` in the function's body; they are mutable.

The locals declare a list of mutable local variables and their types.
These variables are referenced through :ref:`local indices <syntax-localidx>` in the function's body.
The index of the first local is the smallest index not referencing a parameter.

A function's :ref:`expression <syntax-expr>` is an :ref:`instruction <syntax-instr>` sequence that represents the body of the function.
Upon termination it must produce a stack matching the function type's :ref:`result type <syntax-resulttype>`.

Functions are referenced through :ref:`function indices <syntax-funcidx>`,
starting with the smallest index not referencing a function :ref:`import <syntax-import>`.


.. index:: ! data, active, passive, data index, memory, memory index, expression, constant, byte, list
   pair: abstract syntax; data
   single: memory; data
   single: data; segment
.. _syntax-data:
.. _syntax-datamode:

Data Segments
~~~~~~~~~~~~~

The :math:`{\data}` section of a module defines a list of *data segments*,
which can be used to initialize a range of memory from a static :ref:`list <syntax-list>` of :ref:`bytes <syntax-byte>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\data} & ::= & \DATA~{{\byte}^\ast}~{\datamode} \\
   & {\datamode} & ::= & \DACTIVE~{\memidx}~{\expr} ~~|~~ \DPASSIVE \\
   \end{array}

Similar to element segments, data segments have a mode that identifies them as either *active* or *passive*.
A passive data segment's contents can be copied into a memory using the :math:`\mathsf{memory{.}init}` instruction.
An active data segment copies its contents into a memory during :ref:`instantiation <exec-instantiation>`, as specified by a :ref:`memory index <syntax-memidx>` and a :ref:`constant <valid-constant>` :ref:`expression <syntax-expr>` defining an offset into that memory.

Data segments are referenced through :ref:`data indices <syntax-dataidx>`.


.. index:: ! element, ! element mode, ! active, ! passive, ! declarative, element index, table, table index, expression, constant, function index, list
   pair: abstract syntax; element
   pair: abstract syntax; element mode
   single: table; element
   single: element; segment
   single: element; mode
.. _syntax-elem:
.. _syntax-elemmode:

Element Segments
~~~~~~~~~~~~~~~~

The :math:`{\elem}` section of a module defines a list of *element segments*,
which can be used to initialize a subrange of a table from a static :ref:`list <syntax-list>` of elements.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\elem} & ::= & \ELEM~{\reftype}~{{\expr}^\ast}~{\elemmode} \\
   & {\elemmode} & ::= & \EACTIVE~{\tableidx}~{\expr} ~~|~~ \EPASSIVE ~~|~~ \EDECLARE \\
   \end{array}

Each element segment defines a :ref:`reference type <syntax-reftype>` and a corresponding list of :ref:`constant <valid-constant>` element :ref:`expressions <syntax-expr>`.

Element segments have a mode that identifies them as either *active*, *passive*, or *declarative*.
A passive element segment's elements can be copied to a table using the :math:`\mathsf{table{.}init}` instruction.
An active element segment copies its elements into a table during :ref:`instantiation <exec-instantiation>`, as specified by a :ref:`table index <syntax-tableidx>` and a :ref:`constant <valid-constant>` :ref:`expression <syntax-expr>` defining an offset into that table.
A declarative element segment is not available at runtime but merely serves to forward-declare references that are formed in code with instructions like :math:`\mathsf{ref{.}func}`.
The offset is given by another :ref:`constant <valid-constant>` :ref:`expression <syntax-expr>`.

Element segments are referenced through :ref:`element indices <syntax-elemidx>`.


.. index:: ! start function, function, function index, table, memory, instantiation
   pair: abstract syntax; start function
.. _syntax-start:

Start Function
~~~~~~~~~~~~~~

The :math:`{\start}` section of a module declares the :ref:`function index <syntax-funcidx>` of a *start function* that is automatically invoked when the module is :ref:`instantiated <exec-instantiation>`, after :ref:`tables <syntax-table>` and :ref:`memories <syntax-mem>` have been initialized.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\start} & ::= & \START~{\funcidx} \\
   \end{array}

.. note::
   The start function is intended for initializing the state of a module.
   The module and its exports are not accessible externally before this initialization has completed.


.. index:: ! import, name, function type, table type, memory type, global type, tag type, index, index space, type index, function index, table index, memory index, global index, tag index, function, table, memory, tag, global, instantiation
   pair: abstract syntax; import
   single: tag; import
   single: global; import
   single: memory; import
   single: table; import
   single: function; import
.. _syntax-importdesc:
.. _syntax-import:

Imports
~~~~~~~

The :math:`{\import}` section of a module defines a set of *imports* that are required for :ref:`instantiation <exec-instantiation>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\import} & ::= & \IMPORT~{\name}~{\name}~{\externtype} \\
   \end{array}

Each import is labeled by a two-level :ref:`name <syntax-name>` space, consisting of a *module name* and an *item name* for an entity within that module.
Importable definitions are
:ref:`tags <syntax-tag>`,
:ref:`globals <syntax-global>`,
:ref:`memories <syntax-mem>`,
:ref:`tables <syntax-table>`, and
:ref:`functions <syntax-func>`.
Each import is specified by a respective :ref:`external type <syntax-externtype>` that a definition provided during instantiation is required to match.

Every import defines an index in the respective :ref:`index space <syntax-index>`.
In each index space, the indices of imports go before the first index of any definition contained in the module itself.

.. note::
   Unlike export names, import names are not necessarily unique.
   It is possible to import the same module/item name pair multiple times;
   such imports may even have different type descriptions, including different kinds of entities.
   A module with such imports can still be instantiated depending on the specifics of how an :ref:`embedder <embedder>` allows resolving and supplying imports.
   However, embedders are not required to support such overloading,
   and a WebAssembly module itself cannot implement an overloaded name.


.. index:: ! export, name, index, external index, function index, table index, memory index, global index, tag index, function, table, memory, global, tag, instantiation
   pair: abstract syntax; export
   pair: abstract syntax; external index
   single: tag; export
   single: global; export
   single: memory; export
   single: table; export
   single: function; export
.. _syntax-exportdesc:
.. _syntax-export:
.. _syntax-externidx:

Exports
~~~~~~~

The :math:`{\export}` section of a module defines a set of *exports* that become accessible to the host environment once the module has been :ref:`instantiated <exec-instantiation>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\export} & ::= & \EXPORT~{\name}~{\externidx} \\[0.8ex]
   & {\externidx} & ::= & \XXFUNC~{\funcidx} ~~|~~ \XXGLOBAL~{\globalidx} ~~|~~ \XXTABLE~{\tableidx} ~~|~~ \XXMEM~{\memidx} ~~|~~ \XXTAG~{\tagidx} \\
   \end{array}

Each export is labeled by a unique :ref:`name <syntax-name>`.
Exportable definitions are
:ref:`tags <syntax-tag>`,
:ref:`globals <syntax-global>`,
:ref:`memories <syntax-mem>`,
:ref:`tables <syntax-table>`, and
:ref:`functions <syntax-func>`,
which are referenced through a respective index.

Conventions
...........

The following auxiliary notation is defined for sequences of exports, filtering out indices of a specific kind in an order-preserving fashion:

.. math::
   \begin{array}[t]{@{}lcl@{}l@{}}
   {\funcsxx}(\epsilon) & = & \epsilon \\
   {\funcsxx}((\XXFUNC~x)~{{\mathit{xx}}^\ast}) & = & x~{\funcsxx}({{\mathit{xx}}^\ast}) \\
   {\funcsxx}({\externidx}~{{\mathit{xx}}^\ast}) & = & {\funcsxx}({{\mathit{xx}}^\ast}) & \quad \mbox{otherwise} \\[0.8ex]
   {\tablesxx}(\epsilon) & = & \epsilon \\
   {\tablesxx}((\XXTABLE~x)~{{\mathit{xx}}^\ast}) & = & x~{\tablesxx}({{\mathit{xx}}^\ast}) \\
   {\tablesxx}({\externidx}~{{\mathit{xx}}^\ast}) & = & {\tablesxx}({{\mathit{xx}}^\ast}) & \quad \mbox{otherwise} \\[0.8ex]
   {\memsxx}(\epsilon) & = & \epsilon \\
   {\memsxx}((\XXMEM~x)~{{\mathit{xx}}^\ast}) & = & x~{\memsxx}({{\mathit{xx}}^\ast}) \\
   {\memsxx}({\externidx}~{{\mathit{xx}}^\ast}) & = & {\memsxx}({{\mathit{xx}}^\ast}) & \quad \mbox{otherwise} \\[0.8ex]
   {\globalsxx}(\epsilon) & = & \epsilon \\
   {\globalsxx}((\XXGLOBAL~x)~{{\mathit{xx}}^\ast}) & = & x~{\globalsxx}({{\mathit{xx}}^\ast}) \\
   {\globalsxx}({\externidx}~{{\mathit{xx}}^\ast}) & = & {\globalsxx}({{\mathit{xx}}^\ast}) & \quad \mbox{otherwise} \\[0.8ex]
   {\tagsxx}(\epsilon) & = & \epsilon \\
   {\tagsxx}((\XXTAG~x)~{{\mathit{xx}}^\ast}) & = & x~{\tagsxx}({{\mathit{xx}}^\ast}) \\
   {\tagsxx}({\externidx}~{{\mathit{xx}}^\ast}) & = & {\tagsxx}({{\mathit{xx}}^\ast}) & \quad \mbox{otherwise} \\
   \end{array}
