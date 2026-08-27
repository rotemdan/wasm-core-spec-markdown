.. index:: ! runtime
.. _syntax-runtime:

Runtime Structure
-----------------

:ref:`Store <store>`, :ref:`stack <stack>`, and other *runtime structure* forming the WebAssembly abstract machine, such as :ref:`values <syntax-val>` or :ref:`module instances <syntax-moduleinst>`, are made precise in terms of additional auxiliary syntax.


.. index:: ! value, number, reference, constant, number type, vector type, reference type, ! host address, value type, integer, floating-point, vector number, ! default value, unboxed scalar, structure, array, external reference
   pair: abstract syntax; value
.. _syntax-num:
.. _syntax-vec:
.. _syntax-ref:
.. _syntax-ref.i31num:
.. _syntax-ref.struct:
.. _syntax-ref.array:
.. _syntax-ref.exn:
.. _syntax-ref.host:
.. _syntax-ref.extern:
.. _syntax-nullref:
.. _syntax-val:
.. _syntax-pack:

Values
~~~~~~

WebAssembly computations manipulate *values* of either the four basic :ref:`number types <syntax-numtype>`, i.e., :ref:`integers <syntax-int>` and :ref:`floating-point data <syntax-float>` of 32 or 64 bit width each, or :ref:`vectors <syntax-vecnum>` of 128 bit width, or of :ref:`reference type <syntax-reftype>`.

In most places of the semantics, values of different types can occur.
In order to avoid ambiguities, values are therefore represented with an abstract syntax that makes their type explicit.
It is convenient to reuse the same notation as for the :math:`\mathsf{const}` :ref:`instructions <syntax-const>` and :math:`\mathsf{ref{.}null}` producing them.

References other than null are represented with additional :ref:`administrative instructions <syntax-instr-admin>`.
They either are *scalar references*, containing a 31-bit :ref:`integer <syntax-int>`,
*null references*,
*structure references*, pointing to a specific :ref:`structure address <syntax-structaddr>`,
*array references*, pointing to a specific :ref:`array address <syntax-arrayaddr>`,
*function references*, pointing to a specific :ref:`function address <syntax-funcaddr>`,
*exception references*, pointing to a specific :ref:`exception address <syntax-exnaddr>`,
or *host references* pointing to an uninterpreted form of :ref:`host address <syntax-hostaddr>` defined by the :ref:`embedder <embedder>`.
Any of the aformentioned references can furthermore be wrapped up as an *external reference*.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\val} & ::= & {\num} ~~|~~ {\vec} ~~|~~ {\reff} \\[0.8ex]
   & {\num} & ::= & {\numtype}{.}\CONST~{{\num}}_{{\numtype}} \\[0.8ex]
   & {\vec} & ::= & {\vectype}{.}\VCONST~{{\vec}}_{{\vectype}} \\[0.8ex]
   & {\reff} & ::= & \REFI31NUM~{\u31} \\
   & & | & \REFNULLADDR \\
   & & | & \REFSTRUCTADDR~{\structaddr} \\
   & & | & \REFARRAYADDR~{\arrayaddr} \\
   & & | & \REFFUNCADDR~{\funcaddr} \\
   & & | & \REFEXNADDR~{\exnaddr} \\
   & & | & \REFHOSTADDR~{\hostaddr} \\
   & & | & \REFEXTERN~{\reff} \\
   \end{array}

.. note::
   Future versions of WebAssembly may add additional forms of values.

.. _aux-default:

:ref:`Value types <syntax-valtype>` can have an associated *default value*;
it is the respective value :math:`0` for :ref:`number types <syntax-numtype>`, :math:`0` for :ref:`vector types <syntax-vectype>`, and null for nullable :ref:`reference types <syntax-reftype>`.
For other references, no default value is defined, :math:`{{\default}}_{t}` hence is an optional value :math:`{{\val}^?}`.

.. math::
   \begin{array}[t]{@{}lcl@{}l@{}}
   {{\default}}_{{\ntI}{{\ntN}}} & = & ({\ntI}{{\ntN}}{.}\CONST~0) \\
   {{\default}}_{{\ntF}{{\ntN}}} & = & ({\ntF}{{\ntN}}{.}\CONST~{+0}) \\
   {{\default}}_{{\ntV}{{\ntN}}} & = & ({\ntV}{{\ntN}}{.}\VCONST~0) \\
   {{\default}}_{\REF~\NULL~{\mathit{ht}}} & = & \REFNULLADDR \\
   {{\default}}_{\REF~{\mathit{ht}}} & = & \epsilon \\
   \end{array}


Convention
..........

* The meta variable :math:`r` ranges over reference values where clear from context.


.. index:: ! result, value, trap, exception, exception address
   pair: abstract syntax; result
.. _syntax-result:

Results
~~~~~~~

A *result* is the outcome of a computation.
It is either a sequence of :ref:`values <syntax-val>`, a thrown :ref:`exception <exec-throw_ref>`, or a :ref:`trap <syntax-trap>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\result} & ::= & {{\val}^\ast} ~~|~~ ( \REFEXNADDR~{\exnaddr} )~\THROWREF ~~|~~ \TRAP \\
   \end{array}


.. index:: ! store, type instance, function instance, table instance, memory instance, global instance, tag instance, module, allocation, structure instance, array instance, exception instance
   pair: abstract syntax; store
.. _syntax-store:
.. _store:

Store
~~~~~

The *store* represents all global state that can be manipulated by WebAssembly programs.
It consists of the runtime representation of all *instances* of
:ref:`functions <syntax-funcinst>`,
:ref:`tables <syntax-tableinst>`,
:ref:`memories <syntax-meminst>`,
:ref:`globals <syntax-globalinst>`,
:ref:`tags <syntax-taginst>`,
:ref:`element segments <syntax-eleminst>`,
:ref:`data segments <syntax-datainst>`,
and
:ref:`structures <syntax-structinst>`,
:ref:`arrays <syntax-arrayinst>` or
:ref:`exceptions <syntax-exninst>`
that have been :ref:`allocated <alloc>` during the life time of the abstract machine.

It is an invariant of the semantics that no element or data instance is :ref:`addressed <syntax-addr>` from anywhere else but the owning module instances.

Syntactically, the store is defined as a :ref:`record <notation-record>` listing the existing instances of each category:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\store} & ::= & \{ \begin{array}[t]{@{}l@{}l@{}}
   \STAGS~{{\taginst}^\ast} \\
   \SGLOBALS~{{\globalinst}^\ast} \\
   \SMEMS~{{\meminst}^\ast} \\
   \STABLES~{{\tableinst}^\ast} \\
   \SFUNCS~{{\funcinst}^\ast} \\
   \SDATAS~{{\datainst}^\ast} \\
   \SELEMS~{{\eleminst}^\ast} \\
   \SSTRUCTS~{{\structinst}^\ast} \\
   \SARRAYS~{{\arrayinst}^\ast} \\
   \SEXNS~{{\exninst}^\ast} \} \\
   \end{array} \\
   \end{array}

.. note::
   In practice, implementations may apply techniques like garbage collection or reference counting to remove objects from the store that are no longer referenced.
   However, such techniques are not semantically observable,
   and hence outside the scope of this specification.


Convention
..........

* The meta variable :math:`s` ranges over stores where clear from context.


.. index:: ! address, store, function instance, table instance, memory instance, global instance, tag instance, element instance, data instance, structure instance, array instance, exception instance, embedder, host
   pair: abstract syntax; function address
   pair: abstract syntax; table address
   pair: abstract syntax; memory address
   pair: abstract syntax; global address
   pair: abstract syntax; tag address
   pair: abstract syntax; element address
   pair: abstract syntax; data address
   pair: abstract syntax; structure address
   pair: abstract syntax; array address
   pair: abstract syntax; exception address
   pair: abstract syntax; host address
   pair: function; address
   pair: table; address
   pair: memory; address
   pair: global; address
   pair: tag; address
   pair: element; address
   pair: data; address
   pair: structure; address
   pair: array; address
   pair: exception; address
   pair: host; address
.. _syntax-funcaddr:
.. _syntax-tableaddr:
.. _syntax-memaddr:
.. _syntax-globaladdr:
.. _syntax-tagaddr:
.. _syntax-elemaddr:
.. _syntax-dataaddr:
.. _syntax-structaddr:
.. _syntax-arrayaddr:
.. _syntax-exnaddr:
.. _syntax-hostaddr:
.. _syntax-addr:

Addresses
~~~~~~~~~

:ref:`Function instances <syntax-funcinst>`,
:ref:`table instances <syntax-tableinst>`,
:ref:`memory instances <syntax-meminst>`,
:ref:`global instances <syntax-globalinst>`,
:ref:`tag instances <syntax-taginst>`,
:ref:`element instances <syntax-eleminst>`,
:ref:`data instances <syntax-datainst>`
and
:ref:`structure <syntax-structinst>`,
:ref:`array <syntax-arrayinst>` or
:ref:`exception instances <syntax-exninst>`
in the :ref:`store <syntax-store>` are referenced with abstract *addresses*.
These are simply indices into the respective store component.
In addition, an :ref:`embedder <embedder>` may supply an uninterpreted set of *host addresses*.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\addr} & ::= & 0 ~~|~~ 1 ~~|~~ 2 ~~|~~ \dots \\
   & {\funcaddr} & ::= & {\addr} \\
   & {\tableaddr} & ::= & {\addr} \\
   & {\memaddr} & ::= & {\addr} \\
   & {\globaladdr} & ::= & {\addr} \\
   & {\tagaddr} & ::= & {\addr} \\
   & {\elemaddr} & ::= & {\addr} \\
   & {\dataaddr} & ::= & {\addr} \\
   & {\structaddr} & ::= & {\addr} \\
   & {\arrayaddr} & ::= & {\addr} \\
   & {\exnaddr} & ::= & {\addr} \\
   & {\hostaddr} & ::= & {\addr} \\
   \end{array}

An :ref:`embedder <embedder>` may assign identity to :ref:`exported <syntax-export>` store objects corresponding to their addresses,
even where this identity is not observable from within WebAssembly code itself
(such as for :ref:`function instances <syntax-funcinst>` or immutable :ref:`globals <syntax-globalinst>`).

.. note::
   Addresses are *dynamic*, globally unique references to runtime objects,
   in contrast to :ref:`indices <syntax-index>`,
   which are *static*, module-local references to their original definitions.
   A *memory address* |memaddr| denotes the abstract address *of* a memory *instance* in the store,
   not an offset *inside* a memory instance.

   There is no specific limit on the number of allocations of store objects,
   hence logical addresses can be arbitrarily large natural numbers.


.. _free-funcaddr:
.. _free-tableaddr:
.. _free-memaddr:
.. _free-globaladdr:
.. _free-tagaddr:
.. _free-elemaddr:
.. _free-dataaddr:
.. _free-structaddr:
.. _free-arrayaddr:
.. _free-localaddr:
.. _free-labeladdr:
.. _free-addr:

Conventions
...........

* The notation :math:`{\mathrm{addr}}(A)` denotes the set of addresses from address space :math:`{\addr}` occurring free in :math:`A`. We sometimes reinterpret this set as the :ref:`list <syntax-list>` of its elements, without assuming any particular order.


.. index:: ! external address, function address, table address, memory address, global address, tag address, store, function, table, memory, global, tag, instruction type
   pair: abstract syntax; external address
   pair: external; address
.. _syntax-externaddr:

External Addresses
~~~~~~~~~~~~~~~~~~

An *external address* is the runtime :ref:`address <syntax-addr>` of an entity that can be imported or exported.
It is an :ref:`address <syntax-addr>` denoting either a
:ref:`function instance <syntax-funcinst>`,
:ref:`global instance <syntax-globalinst>`,
:ref:`table instance <syntax-tableinst>`,
:ref:`memory instance <syntax-meminst>`, or
:ref:`tag instance <syntax-taginst>`
in the shared :ref:`store <syntax-store>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\externaddr} & ::= & \XATAG~{\tagaddr} ~~|~~ \XAGLOBAL~{\globaladdr} ~~|~~ \XAMEM~{\memaddr} ~~|~~ \XATABLE~{\tableaddr} ~~|~~ \XAFUNC~{\funcaddr} \\
   \end{array}



.. index:: ! instance, function type, type instance, function instance, table instance, memory instance, global instance, tag instance, element instance, data instance, export instance, table address, memory address, global address, tag address, element address, data address, index, name
   pair: abstract syntax; module instance
   pair: module; instance
.. _syntax-moduleinst:

Module Instances
~~~~~~~~~~~~~~~~

A *module instance* is the runtime representation of a :ref:`module <syntax-module>`.
It is created by :ref:`instantiating <exec-instantiation>` a module,
and collects runtime representations of all entities that are imported, defined, or exported by the module.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\moduleinst} & ::= & \{ \begin{array}[t]{@{}l@{}l@{}}
   \MITYPES~{{\deftype}^\ast} \\
   \MITAGS~{{\tagaddr}^\ast} \\
   \MIGLOBALS~{{\globaladdr}^\ast} \\
   \MIMEMS~{{\memaddr}^\ast} \\
   \MITABLES~{{\tableaddr}^\ast} \\
   \MIFUNCS~{{\funcaddr}^\ast} \\
   \MIDATAS~{{\dataaddr}^\ast} \\
   \MIELEMS~{{\elemaddr}^\ast} \\
   \MIEXPORTS~{{\exportinst}^\ast} \} \\
   \end{array} \\
   \end{array}

Each component references runtime instances corresponding to respective declarations from the original module -- whether imported or defined -- in the order of their static :ref:`indices <syntax-index>`.
:ref:`Function instances <syntax-funcinst>`,
:ref:`table instances <syntax-tableinst>`,
:ref:`memory instances <syntax-meminst>`,
:ref:`global instances <syntax-globalinst>`, and
:ref:`tag instances <syntax-taginst>`
are denoted by their respective :ref:`addresses <syntax-addr>` in the :ref:`store <syntax-store>`.

It is an invariant of the semantics that all :ref:`export instances <syntax-exportinst>` in a given module instance have different :ref:`names <syntax-name>`.

.. note::
   All record fields except :math:`\mathsf{exports}` are to be considered *private* components of a module instance.
   They are not accessible to other modules,
   only to function instances originating from the same module.


.. index:: ! function instance, module instance, function, closure, module, ! host function, invocation
   pair: abstract syntax; function instance
   pair: function; instance
.. _syntax-hostfunc:
.. _syntax-funcinst:

Function Instances
~~~~~~~~~~~~~~~~~~

A *function instance* is the runtime representation of a :ref:`function <syntax-func>`.
It effectively is a *closure* of the original function over the runtime :ref:`module instance <syntax-moduleinst>` of its originating :ref:`module <syntax-module>`.
The module instance is used to resolve references to other definitions during execution of the function.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\funcinst} & ::= & \{ \begin{array}[t]{@{}l@{}l@{}}
   \FITYPE~{\deftype} ,  \FIMODULE~{\moduleinst} ,  \FICODE~{{\funccode}} \} \\
   \end{array} \\
   & {{\funccode}} & ::= & {\func} ~~|~~ {\hostfunc} \\
   \end{array}

A *host function* is a function expressed outside WebAssembly but passed to a :ref:`module <syntax-module>` as an :ref:`import <syntax-import>`.
The definition and behavior of host functions are outside the scope of this specification.
For the purpose of this specification, it is assumed that when :ref:`invoked <exec-invoke-host>`,
a host function behaves non-deterministically,
but within certain :ref:`constraints <exec-invoke-host>` that ensure the integrity of the runtime.

.. note::
   Function instances are immutable, and their identity is not observable by WebAssembly code.
   However, an :ref:`embedder <embedder>` might provide implicit or explicit means for distinguishing their :ref:`addresses <syntax-funcaddr>`.


.. index:: ! table instance, table, function address, table type, embedder, element segment
   pair: abstract syntax; table instance
   pair: table; instance
.. _syntax-tableinst:

Table Instances
~~~~~~~~~~~~~~~

A *table instance* is the runtime representation of a :ref:`table <syntax-table>`.
It records its :ref:`type <syntax-tabletype>` and holds a sequence of :ref:`reference values <syntax-ref>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\tableinst} & ::= & \{ \begin{array}[t]{@{}l@{}l@{}}
   \TITYPE~{\tabletype} ,  \TIREFS~{{\reff}^\ast} \} \\
   \end{array} \\
   \end{array}

Table elements can be mutated through :ref:`table instructions <syntax-instr-table>`, the execution of an active :ref:`element segment <syntax-elem>`, or by external means provided by the :ref:`embedder <embedder>`.

It is an invariant of the semantics that all table elements have a type :ref:`matching <match-reftype>` the element type of :math:`{\tabletype}`.
It also is an invariant that the length of the element sequence never exceeds the maximum size of :math:`{\tabletype}`.


.. index:: ! memory instance, memory, byte, ! page size, memory type, embedder, data segment, instruction
   pair: abstract syntax; memory instance
   pair: memory; instance
.. _page-size:
.. _syntax-meminst:

Memory Instances
~~~~~~~~~~~~~~~~

A *memory instance* is the runtime representation of a linear :ref:`memory <syntax-mem>`.
It records its :ref:`type <syntax-memtype>` and holds a sequence of :ref:`bytes <syntax-byte>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\meminst} & ::= & \{ \begin{array}[t]{@{}l@{}l@{}}
   \MITYPE~{\memtype} ,  \MIBYTES~{{\byte}^\ast} \} \\
   \end{array} \\
   \end{array}

The length of the sequence always is a multiple of the WebAssembly *page size*, which is defined to be the constant :math:`65536` -- abbreviated :math:`64~{\mathrm{Ki}}`.

A memory's bytes can be mutated through :ref:`memory instructions <syntax-instr-memory>`, the execution of an active :ref:`data segment <syntax-data>`, or by external means provided by the :ref:`embedder <embedder>`.

It is an invariant of the semantics that the length of the byte sequence, divided by page size, never exceeds the maximum size of :math:`{\memtype}`.


.. index:: ! global instance, global, value, mutability, instruction, embedder
   pair: abstract syntax; global instance
   pair: global; instance
.. _syntax-globalinst:

Global Instances
~~~~~~~~~~~~~~~~

A *global instance* is the runtime representation of a :ref:`global <syntax-global>` variable.
It records its :ref:`type <syntax-globaltype>` and holds an individual :ref:`value <syntax-val>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\globalinst} & ::= & \{ \begin{array}[t]{@{}l@{}l@{}}
   \GITYPE~{\globaltype} ,  \GIVALUE~{\val} \} \\
   \end{array} \\
   \end{array}

The value of mutable globals can be mutated through :ref:`variable instructions <syntax-instr-variable>` or by external means provided by the :ref:`embedder <embedder>`.

It is an invariant of the semantics that the value has a type :ref:`matching <match-valtype>` the :ref:`value type <syntax-valtype>` of :math:`{\globaltype}`.


.. index:: ! tag instance, tag, exception tag, tag type
   pair: abstract syntax; tag instance
   pair: tag; instance
.. _syntax-taginst:

Tag Instances
~~~~~~~~~~~~~

A *tag instance* is the runtime representation of a :ref:`tag <syntax-tag>` definition.
It records the :ref:`defined type <syntax-deftype>` of the tag.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\taginst} & ::= & \{ \begin{array}[t]{@{}l@{}l@{}}
   \HITYPE~{\tagtype} \} \\
   \end{array} \\
   \end{array}


.. index:: ! element instance, element segment, embedder, element expression
   pair: abstract syntax; element instance
   pair: element; instance
.. _syntax-eleminst:

Element Instances
~~~~~~~~~~~~~~~~~

An *element instance* is the runtime representation of an :ref:`element segment <syntax-elem>`.
It holds a list of references and its :ref:`type <syntax-reftype>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\eleminst} & ::= & \{ \begin{array}[t]{@{}l@{}l@{}}
   \EITYPE~{\elemtype} ,  \EIREFS~{{\reff}^\ast} \} \\
   \end{array} \\
   \end{array}

It is an invariant of the semantics that all elements of a segment have a type :ref:`matching <match-reftype>` :math:`{\elemtype}`.


.. index:: ! data instance, data segment, embedder, byte
  pair: abstract syntax; data instance
  pair: data; instance
.. _syntax-datainst:

Data Instances
~~~~~~~~~~~~~~

A *data instance* is the runtime representation of a :ref:`data segment <syntax-data>`.
It holds a list of :ref:`bytes <syntax-byte>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\datainst} & ::= & \{ \begin{array}[t]{@{}l@{}l@{}}
   \DIBYTES~{{\byte}^\ast} \} \\
   \end{array} \\
   \end{array}


.. index:: ! export instance, export, name, external address
   pair: abstract syntax; export instance
   pair: export; instance
.. _syntax-exportinst:

Export Instances
~~~~~~~~~~~~~~~~

An *export instance* is the runtime representation of an :ref:`export <syntax-export>`.
It defines the export's :ref:`name <syntax-name>` and the associated :ref:`external address <syntax-externaddr>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\exportinst} & ::= & \{ \begin{array}[t]{@{}l@{}l@{}}
   \XINAME~{\name} ,  \XIADDR~{\externaddr} \} \\
   \end{array} \\
   \end{array}


Conventions
...........

The following auxiliary functions are assumed on sequences of external addresses.
They extract addresses of a specific kind in an order-preserving fashion:

* :math:`{\funcsxa}({{\mathit{xa}}^\ast})` extracts all :ref:`function addresses <syntax-funcaddr>` from :math:`{{\mathit{xa}}^\ast}`,
* :math:`{\tablesxa}({{\mathit{xa}}^\ast})` extracts all :ref:`table addresses <syntax-funcaddr>` from :math:`{{\mathit{xa}}^\ast}`,
* :math:`{\memsxa}({{\mathit{xa}}^\ast})` extracts all :ref:`memory addresses <syntax-funcaddr>` from :math:`{{\mathit{xa}}^\ast}`,
* :math:`{\globalsxa}({{\mathit{xa}}^\ast})` extracts all :ref:`global addresses <syntax-funcaddr>` from :math:`{{\mathit{xa}}^\ast}`,
* :math:`{\tagsxa}({{\mathit{xa}}^\ast})` extracts all :ref:`tag addresses <syntax-funcaddr>` from :math:`{{\mathit{xa}}^\ast}`.




.. index:: ! structure instance, ! array instance, structure type, array type, defined type, ! field value, ! packed value
   pair: abstract syntax; field value
   pair: abstract syntax; packed value
   pair: abstract syntax; structure instance
   pair: abstract syntax; array instance
   pair: structure; instance
   pair: array; instance
.. _syntax-fieldval:
.. _syntax-packval:
.. _syntax-structinst:
.. _syntax-arrayinst:
.. _syntax-aggrinst:

Aggregate Instances
~~~~~~~~~~~~~~~~~~~

A *structure instance* is the runtime representation of a heap object allocated from a :ref:`structure type <syntax-structtype>`.
Likewise, an *array instance* is the runtime representation of a heap object allocated from an :ref:`array type <syntax-arraytype>`.
Both record their respective :ref:`defined type <syntax-deftype>` and hold a list of the values of their *fields*.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\structinst} & ::= & \{ \begin{array}[t]{@{}l@{}l@{}}
   \SITYPE~{\deftype} ,  \SIFIELDS~{{\fieldval}^\ast} \} \\
   \end{array} \\
   & {\arrayinst} & ::= & \{ \begin{array}[t]{@{}l@{}l@{}}
   \AITYPE~{\deftype} ,  \AIFIELDS~{{\fieldval}^\ast} \} \\
   \end{array} \\
   & {\fieldval} & ::= & {\val} ~~|~~ {\packval} \\
   & {\packval} & ::= & {\packtype}{.}\PACK~{{\iNX}}{N} \\
   \end{array}


.. _aux-packfield:
.. _aux-unpackfield:

Conventions
...........

* Conversion of a regular :ref:`value <syntax-val>` to a :ref:`field value <syntax-fieldval>` is defined as follows:

  .. math::
     \begin{array}[t]{@{}lcl@{}l@{}}
     {{\packfield}}_{{\valtype}}({\val}) & = & {\val} \\
     {{\packfield}}_{{\packtype}}(\I32{.}\CONST~i) & = & {\packtype}{.}\PACK~{{\wrap}}_{32, {|{\packtype}|}}(i) \\
     \end{array}

* The inverse conversion of a :ref:`field value <syntax-fieldval>` to a regular :ref:`value <syntax-val>` is defined as follows:

  .. math::
     \begin{array}[t]{@{}lcl@{}l@{}}
     {{{{\unpackfield}}_{{\valtype}}^{\epsilon}}}{({\val})} & = & {\val} \\
     {{{{\unpackfield}}_{{\packtype}}^{{\sx}}}}{({\packtype}{.}\PACK~i)} & = & \I32{.}\CONST~{{{{\extend}}_{{|{\packtype}|}, 32}^{{\sx}}}}{(i)} \\
     \end{array}


.. index:: ! exception instance, tag, tag address, value
   pair: abstract syntax; exception instance
   pair: exception; instance
.. _syntax-exninst:

Exception Instances
~~~~~~~~~~~~~~~~~~~

An *exception instance* is the runtime representation of an :ref:`exception <exception>` produced by a :math:`\mathsf{throw}` instruction.
It holds the :ref:`address <syntax-tagaddr>` of the respective :ref:`tag <syntax-tag>` and the argument :ref:`values <syntax-val>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\exninst} & ::= & \{ \begin{array}[t]{@{}l@{}l@{}}
   \EITAG~{\tagaddr} ,  \EIFIELDS~{{\val}^\ast} \} \\
   \end{array} \\
   \end{array}


.. index:: ! stack, ! control frame, ! call frame, ! frame, ! label, ! handler, instruction, store, activation, function, call, ! call frame, local, exception, module instance
   pair: abstract syntax; frame
   pair: abstract syntax; label
   pair: abstract syntax; handler
.. _syntax-frame:
.. _syntax-callframe:
.. _syntax-label:
.. _syntax-handler:
.. _frame:
.. _label:
.. _handler:
.. _stack:

Stack
~~~~~

Besides the :ref:`store <store>`, most :ref:`instructions <syntax-instr>` interact with an implicit *stack*.
The stack contains the two kinds of entries:

* *Values*: the *operands* of instructions.

* *Control Frames*: currently active control flow structures.

The latter can in turn be one of the following:

* *Labels*: active :ref:`structured control instructions <syntax-instr-control>` that can be targeted by branches.

* *(Call) Frames*: the *activation records* of active :ref:`function <syntax-func>` calls.

* *Handlers*: active exception handlers.

.. note::
   Where clear from context, *call frame* is abbreviated to just *frame*.

All these entries can occur on the stack in any order during the execution of a program.
Stack entries are described by abstract syntax as follows.

.. note::
   It is possible to model the WebAssembly semantics using separate stacks for operands, control constructs, and calls.
   However, because the stacks are interdependent, additional book keeping about associated stack heights would be required.
   For the purpose of this specification, an interleaved representation is simpler.

Values
......

Values are represented by :ref:`themselves <syntax-val>`.

Labels
......

Labels carry an argument arity :math:`n` and their associated branch *target*, which is expressed syntactically as an :ref:`instruction <syntax-instr>` sequence:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\label} & ::= & {{\LABEL}_{n}}{\{ {{\instr}^\ast} \}} \\
   \end{array}

Intuitively, :math:`{{\instr}^\ast}` is the *continuation* to execute when the branch is taken, in place of the original control construct.

.. note::
   For example, a loop label has the form

   .. math::
      {{\LABEL}_{n}}{\{ (\LOOP~{\mathit{bt}}~\dots) \}}

   When performing a branch to this label, this executes the loop, effectively restarting it from the beginning.
   Conversely, a simple block label has the form

   .. math::
      {{\LABEL}_{n}}{\{ \epsilon \}}

   When branching, the empty continuation ends the targeted block, such that execution can proceed with consecutive instructions.

Call Frames
...........

Call frames carry the return arity :math:`n` of the respective function,
hold the values of its :ref:`locals <syntax-local>` (including arguments) in the order corresponding to their static :ref:`local indices <syntax-localidx>`,
and a reference to the function's own :ref:`module instance <syntax-moduleinst>`:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\callframe} & ::= & {{\FRAME}_{n}}{\{ {\frame} \}} \\
   & {\frame} & ::= & \{ \begin{array}[t]{@{}l@{}l@{}}
   \ALOCALS~{({{\val}^?})^\ast} ,  \AMODULE~{\moduleinst} \} \\
   \end{array} \\
   \end{array}

Locals may be uninitialized, in which case they are empty.
Locals are mutated by respective :ref:`variable instructions <syntax-instr-variable>`.

Exception Handlers
..................

Exception handlers are installed by |TRYTABLE| instructions and record the corresponding list of :ref:`catch clauses <syntax-catch>`:

.. math::
   \begin{array}{llllll}
     \production{handler} & \handler &::=&
       \HANDLER_n\{\catch^\ast\}
   \end{array}

The handlers on the stack are searched when an exception is :ref:`thrown <syntax-throw>`.


.. _aux-blocktype:

Conventions
...........

* The meta variable :math:`L` ranges over labels where clear from context.

* The meta variable :math:`f` ranges over frame states where clear from context.

* The meta variable :math:`H` ranges over exception handlers where clear from context.

* The following auxiliary definition takes a :ref:`block type <syntax-blocktype>` and looks up the :ref:`instruction type <syntax-instrtype>` that it denotes in the current frame:

  .. math::
     \begin{array}[t]{@{}lcl@{}l@{}}
     {{\fblocktype}}_{z}(x) & = & {t_1^\ast} \rightarrow {t_2^\ast} & \quad \mbox{if}~ z{.}\ZTYPES{}[x] \approxexpanddt \TFUNC~{t_1^\ast} \Tarrow {t_2^\ast} \\
     {{\fblocktype}}_{z}({t^?}) & = & \epsilon \rightarrow {t^?} \\
     \end{array}


.. index:: ! administrative instructions, function, function instance, function address, label, frame, instruction, trap, call, memory, memory instance, table, table instance, element, data, segment, tag, tag instance, tag address, exception, reftype, handler, caught, caught exception
   pair:: abstract syntax; administrative instruction
.. _syntax-trap:
.. _syntax-instr-admin:

Administrative Instructions
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. note::
   This section is only relevant for the :ref:`formal notation <exec-notation>`.

In order to express the reduction of :ref:`traps <trap>`, :ref:`calls <syntax-call>`, :ref:`exception handling <syntax-handler>`, and :ref:`control instructions <syntax-instr-control>`, the syntax of instructions is extended to include the following *administrative instructions*:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\instr} & ::= & \dots \\
   & & | & {\reff} \\
   & & | & {{\LABEL}_{n}}{\{ {{\instr}^\ast} \}}~{{\instr}^\ast} \\
   & & | & {{\FRAME}_{n}}{\{ {\frame} \}}~{{\instr}^\ast} \\
   & & | & {{\HANDLER}_{n}}{\{ {{\catch}^\ast} \}}~{{\instr}^\ast} \\
   & & | & \TRAP \\
   \end{array}

A :ref:`reference <syntax-ref>` represents a :ref:`reference <syntax-ref>` value of respective form :ref:`"on the stack" <exec-notation>`.

The :math:`\mathsf{label}`, :math:`\mathsf{frame}`, and :math:`\mathsf{handler}` instructions model :ref:`labels <syntax-label>`, :ref:`frames <syntax-frame>`, and active :ref:`exception handlers <syntax-handler>`, respectively, :ref:`"on the stack" <exec-notation>`.
Moreover, the administrative syntax maintains the nesting structure of the original :ref:`structured control instruction <syntax-instr-control>` or :ref:`function body <syntax-func>` and their :ref:`instruction sequences <syntax-instrs>`.

The :math:`\mathsf{trap}` instruction represents the occurrence of a trap.
Traps are bubbled up through nested instruction sequences, ultimately reducing the entire program to a single :math:`\mathsf{trap}` instruction, signalling abrupt termination.

.. note::
   For example, the :ref:`reduction rule <exec-block>` for :math:`\mathsf{block}` is:

   .. math::
      (\BLOCK~{\mathit{bt}}~{{\instr}^\ast}) \stepto ({{\LABEL}_{n}}{\{ \epsilon \}}~{{\instr}^\ast})

   if the :ref:`block type <syntax-blocktype>` :math:`{\mathit{bt}}` denotes a :ref:`function type <syntax-functype>` :math:`\TFUNC~{t_1^{m}} \Tarrow {t_2^{n}}`,
   such that :math:`n` is the block's result arity.
   This rule replaces the block with a label instruction,
   which can be interpreted as "pushing" the label on the stack.
   When its end is reached, i.e., the inner instruction sequence has been reduced to the empty sequence -- or rather, a sequence of :math:`n` :ref:`values <syntax-val>` representing the results -- then the :math:`\mathsf{label}` instruction is eliminated courtesy of its own :ref:`reduction rule <exec-label>`:

   .. math::
      ({{\LABEL}_{n}}{\{ {{\instr}^\ast} \}}~{{\val}^\ast}) \stepto {{\val}^\ast}

   This can be interpreted as removing the label from the stack and only leaving the locally accumulated operand values.
   Validation guarantees that :math:`n` matches the number :math:`{|{{\val}^\ast}|}` of resulting values at this point.


.. index:: ! configuration, ! state, ! thread, store, frame, instruction, module instruction
.. _syntax-state:
.. _syntax-thread:
.. _syntax-config:

Configurations
~~~~~~~~~~~~~~

A *configuration* describes the current computation.
It consists of the computations's *state* and the sequence of :ref:`instructions <syntax-instr>` left to execute.
The state in turn consists of a global :ref:`store <syntax-store>` and a current :ref:`frame <syntax-frame>` referring to the :ref:`module instance <syntax-moduleinst>` in which the computation runs, i.e., where the current function originates from.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\config} & ::= & {\state} ; {{\instr}^\ast} \\[0.8ex]
   & {\state} & ::= & {\store} ; {\frame} \\
   \end{array}

.. old
   A *configuration* consists of the current :ref:`store <syntax-store>` and an executing *thread*.

   A thread is a computation over :ref:`instructions <syntax-instr>`
   that operates relative to the state of a current :ref:`frame <syntax-frame>` referring to the :ref:`module instance <syntax-moduleinst>` in which the computation runs, i.e., where the current function originates from.

   .. math::
      \begin{array}{llcl}
      \production{configuration} & \config &::=&
        \store; \thread \\
      \production{thread} & \thread &::=&
        \frame; \instr^\ast \\
      \end{array}

.. note::
   The current version of WebAssembly is single-threaded,
   but configurations with multiple threads may be supported in the future.


Conventions
...........

* The meta variable :math:`z` ranges over frame states where clear from context.

* The following shorthands are defined for accessing a state :math:`z = (s ; f)`:

  - :math:`\begin{array}[t]{@{}l@{~}c@{~}l@{}l@{}} z{.}\ZTYPES{}[x] & = & f{.}\AMODULE{.}\MITYPES{}[x] \\ \end{array}`
  - :math:`\begin{array}[t]{@{}l@{~}c@{~}l@{}l@{}} z{.}\ZTAGS{}[x] & = & s{.}\STAGS{}[f{.}\AMODULE{.}\MITAGS{}[x]] \\ \end{array}`
  - :math:`\begin{array}[t]{@{}l@{~}c@{~}l@{}l@{}} z{.}\ZGLOBALS{}[x] & = & s{.}\SGLOBALS{}[f{.}\AMODULE{.}\MIGLOBALS{}[x]] \\ \end{array}`
  - :math:`\begin{array}[t]{@{}l@{~}c@{~}l@{}l@{}} z{.}\ZMEMS{}[x] & = & s{.}\SMEMS{}[f{.}\AMODULE{.}\MIMEMS{}[x]] \\ \end{array}`
  - :math:`\begin{array}[t]{@{}l@{~}c@{~}l@{}l@{}} z{.}\ZTABLES{}[x] & = & s{.}\STABLES{}[f{.}\AMODULE{.}\MITABLES{}[x]] \\ \end{array}`
  - :math:`\begin{array}[t]{@{}l@{~}c@{~}l@{}l@{}} z{.}\ZFUNCS{}[x] & = & s{.}\SFUNCS{}[f{.}\AMODULE{.}\MIFUNCS{}[x]] \\ \end{array}`
  - :math:`\begin{array}[t]{@{}l@{~}c@{~}l@{}l@{}} z{.}\ZDATAS{}[x] & = & s{.}\SDATAS{}[f{.}\AMODULE{.}\MIDATAS{}[x]] \\ \end{array}`
  - :math:`\begin{array}[t]{@{}l@{~}c@{~}l@{}l@{}} z{.}\ZELEMS{}[x] & = & s{.}\SELEMS{}[f{.}\AMODULE{.}\MIELEMS{}[x]] \\ \end{array}`
  - :math:`\begin{array}[t]{@{}l@{~}c@{~}l@{}l@{}} z{.}\ZLOCALS{}[x] & = & f{.}\ALOCALS{}[x] \\ \end{array}`

* These shorthands also extend to :ref:`notation <notation-replace>` for updating state:

  - :math:`\begin{array}[t]{@{}l@{~}c@{~}l@{}l@{}} z{}[{.}\ZGGLOBALS{}[x]{.}\ZGVALUE = v] & = & s{}[{.}\SGLOBALS{}[f{.}\AMODULE{.}\MIGLOBALS{}[x]]{.}\GIVALUE = v] ; f \\ \end{array}`
  - :math:`\begin{array}[t]{@{}l@{~}c@{~}l@{}l@{}} z{}[{.}\ZMMEMS{}[x]{.}\ZMBYTES{}[i : j] = {b^\ast}] & = & s{}[{.}\SMEMS{}[f{.}\AMODULE{.}\MIMEMS{}[x]]{.}\MIBYTES{}[i : j] = {b^\ast}] ; f \\ \end{array}`
  - :math:`\begin{array}[t]{@{}l@{~}c@{~}l@{}l@{}} z{}[{.}\ZTTABLES{}[x]{.}\ZTREFS{}[i] = r] & = & s{}[{.}\STABLES{}[f{.}\AMODULE{.}\MITABLES{}[x]]{.}\TIREFS{}[i] = r] ; f \\ \end{array}`
  - :math:`\begin{array}[t]{@{}l@{~}c@{~}l@{}l@{}} z{}[{.}\ZLOCALS{}[x] = v] & = & s ; f{}[{.}\ALOCALS{}[x] = v] \\ \end{array}`
