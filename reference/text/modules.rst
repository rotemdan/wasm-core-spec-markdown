Modules
-------

Modules consist of a sequence of :ref:`declarations <text-decl>`.
The grammar rules for each declaration construct produce a pair,
consisting of not just the abstract syntax representing the respective declaration,
but also an :ref:`identifier context <text-context>` recording the new symbolic :ref:`identifiers <text-id>` bound by the construct,
for use in the remainder of the module.


.. index:: index, type index, tag index, global index, memory index, table index, function index, data index, element index, local index, label index
   pair: text format; type index
   pair: text format; tag index
   pair: text format; global index
   pair: text format; memory index
   pair: text format; table index
   pair: text format; function index
   pair: text format; data index
   pair: text format; element index
   pair: text format; local index
   pair: text format; label index
.. _text-idx:
.. _text-typeidx:
.. _text-tagidx:
.. _text-globalidx:
.. _text-memidx:
.. _text-tableidx:
.. _text-funcidx:
.. _text-dataidx:
.. _text-elemidx:
.. _text-localidx:
.. _text-labelidx:
.. _text-fieldidx:
.. _text-index:

Indices
~~~~~~~

:ref:`Indices <syntax-index>` can be given either in raw numeric form or as symbolic :ref:`identifiers <text-id>` when bound by a respective construct.
Such identifiers are looked up in the suitable space of the :ref:`identifier context <text-context>` :math:`{\I}`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tidx}}_{{\mathit{ids}}} & ::= & x{:}{\Tu32} & \quad\Rightarrow\quad{} & x \\
   & & | & {\mathit{id}}{:}{\Tid} & \quad\Rightarrow\quad{} & x & \quad \mbox{if}~ {\mathit{ids}}{}[x] = {\mathit{id}} \\[0.8ex]
   & {{\Ttypeidx}}_{{\I}} & ::= & {{\Tidx}}_{{\I}{.}\ITYPES} \\
   & {{\Ttagidx}}_{{\I}} & ::= & {{\Tidx}}_{{\I}{.}\ITAGS} \\
   & {{\Tglobalidx}}_{{\I}} & ::= & {{\Tidx}}_{{\I}{.}\IGLOBALS} \\
   & {{\Tmemidx}}_{{\I}} & ::= & {{\Tidx}}_{{\I}{.}\IMEMS} \\
   & {{\Ttableidx}}_{{\I}} & ::= & {{\Tidx}}_{{\I}{.}\ITABLES} \\
   & {{\Tfuncidx}}_{{\I}} & ::= & {{\Tidx}}_{{\I}{.}\IFUNCS} \\
   & {{\Tdataidx}}_{{\I}} & ::= & {{\Tidx}}_{{\I}{.}\IDATAS} \\
   & {{\Telemidx}}_{{\I}} & ::= & {{\Tidx}}_{{\I}{.}\IELEMS} \\
   & {{\Tlocalidx}}_{{\I}} & ::= & {{\Tidx}}_{{\I}{.}\ILOCALS} \\
   & {{\Tlabelidx}}_{{\I}} & ::= & {{\Tidx}}_{{\I}{.}\ILABELS} \\
   & {{\Tfieldidx}}_{{\I}, x} & ::= & {{\Tidx}}_{{\I}{.}\IFIELDS{}[x]} \\
   \end{array}


.. index:: type, recursive type, identifier
   pair: text format; type
.. _text-types:

Types
~~~~~

A type definition consists of a :ref:`recursive type <text-rectype>`.
The :ref:`identifier context <text-context>` produced for the local bindings is further extended with the respective sequence of :ref:`defined types <syntax-deftype>` that the recursive type generates.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Ttype}}_{{\I}} & ::= & ({\mathit{qt}}, {\I'}){:}{{\Trectype}}_{{\I}} & \quad\Rightarrow\quad{} & (\TYPE~{\mathit{qt}}, {\I'} \oplus {\I''}) & \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ {\mathit{qt}} = \TREC~{{\mathit{st}}^{n}} \\
   {\land}~ {\I''} = \{ \ITYPEDEFS~{({\mathit{qt}} {.} i)^{i<n}} \} \\
   \end{array} \\
   \end{array}


.. index:: tag, tag type, identifier, function type, exception tag
   pair: text format; tag
.. _text-tag:

Tags
~~~~

A tag definition can bind a symbolic :ref:`tag identifier <text-id>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Ttag}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{tag}’}~~{{\mathit{id}}^?}{:}{{\Tid}^?}~~{\mathit{jt}}{:}{{\Ttagtype}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & (\TAG~{\mathit{jt}}, \{ \ITAGS~({{\mathit{id}}^?}) \}) \\
   \end{array}


.. index:: import, name
   pair: text format; import
.. index:: export, name, index, tag index
   pair: text format; export
.. index:: tag
.. _text-tag-abbrev:

Abbreviations
.............

Tags can be defined as :ref:`imports <text-import>` or :ref:`exports <text-export>` inline:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Timport}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{tag}’}~~{{\Tid}^?}~~\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{import}’}~~{{\Tname}^{2}}~~\mbox{‘\texttt{{)}}’}~~{{\Ttagtype}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\equiv\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{import}’}~~{{\Tname}^{2}}~~\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{tag}’}~~{{\Tid}^?}~~{{\Ttagtype}}_{{\I}}~~\mbox{‘\texttt{{)}}’}~~\mbox{‘\texttt{{)}}’} \\
   \end{array}
   } \\[0.8ex]
   & {{\Texport}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{tag}’}~~{{\mathit{id}}^?}{:}{{\Tid}^?}~~\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{export}’}~~{\Tname}~~\mbox{‘\texttt{{)}}’}~~\dots~~\mbox{‘\texttt{{)}}’} & \quad\equiv\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}l@{}l@{}}
   \begin{array}[t]{@{}l@{}} \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{tag}’}~~{\mathit{id}'}{:}{\Tid}~~\dots~~\mbox{‘\texttt{{)}}’} \\
     \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{export}’}~~{\Tname}~~\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{tag}’}~~{\Tid}~~\mbox{‘\texttt{{)}}’}~~\mbox{‘\texttt{{)}}’} \end{array} & & \\
    \multicolumn{3}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \quad \mbox{if}~ {{\mathit{id}}^?} = {\mathit{id}'} \lor {{\mathit{id}}^?} = \epsilon \land {\mathit{id}'} \notin {\I}{.}\ITAGS \\
   \end{array}
   } \\
   \end{array}
   } \\
   \end{array}

.. note::
   The latter abbreviation can be applied repeatedly, if ":math:`\dots`" contains additional export clauses.
   Consequently, a memory declaration can contain any number of exports, possibly followed by an import.


.. index:: global, global type, identifier, expression
   pair: text format; global
.. _text-global:

Globals
~~~~~~~

Global definitions can bind a symbolic :ref:`global identifier <text-id>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tglobal}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{global}’}~~{{\mathit{id}}^?}{:}{{\Tid}^?}~~{\mathit{gt}}{:}{{\Tglobaltype}}_{{\I}}~~e{:}{{\Texpr}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & (\GLOBAL~{\mathit{gt}}~e, \{ \IGLOBALS~({{\mathit{id}}^?}) \}) \\
   \end{array}


.. index:: import, name
   pair: text format; import
.. index:: export, name, index, global index
   pair: text format; export
.. _text-global-abbrev:

Abbreviations
.............

Globals can be defined as :ref:`imports <text-import>` or :ref:`exports <text-export>` inline:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Timport}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{global}’}~~{{\Tid}^?}~~\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{import}’}~~{{\Tname}^{2}}~~\mbox{‘\texttt{{)}}’}~~{{\Tglobaltype}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\equiv\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{import}’}~~{{\Tname}^{2}}~~\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{global}’}~~{{\Tid}^?}~~{{\Tglobaltype}}_{{\I}}~~\mbox{‘\texttt{{)}}’}~~\mbox{‘\texttt{{)}}’} \\
   \end{array}
   } \\[0.8ex]
   & {{\Texport}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{global}’}~~{{\mathit{id}}^?}{:}{{\Tid}^?}~~\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{export}’}~~{\Tname}~~\mbox{‘\texttt{{)}}’}~~\dots~~\mbox{‘\texttt{{)}}’} & \quad\equiv\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}l@{}l@{}}
   \begin{array}[t]{@{}l@{}} \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{global}’}~~{\mathit{id}'}{:}{\Tid}~~\dots~~\mbox{‘\texttt{{)}}’} \\
     \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{export}’}~~{\Tname}~~\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{global}’}~~{\Tid}~~\mbox{‘\texttt{{)}}’}~~\mbox{‘\texttt{{)}}’} \end{array} & & \\
    \multicolumn{3}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \quad \mbox{if}~ {{\mathit{id}}^?} = {\mathit{id}'} \lor {{\mathit{id}}^?} = \epsilon \land {\mathit{id}'} \notin {\I}{.}\IGLOBALS \\
   \end{array}
   } \\
   \end{array}
   } \\
   \end{array}

.. note::
   The latter abbreviation can be applied repeatedly, if ":math:`\dots`" contains additional export clauses.
   Consequently, a global declaration can contain any number of exports, possibly followed by an import.


.. index:: memory, memory type, identifier
   pair: text format; memory
.. _text-mem:

Memories
~~~~~~~~

Memory definitions can bind a symbolic :ref:`memory identifier <text-id>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tmem}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{memory}’}~~{{\mathit{id}}^?}{:}{{\Tid}^?}~~{\mathit{mt}}{:}{{\Tmemtype}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & (\MEMORY~{\mathit{mt}}, \{ \IMEMS~({{\mathit{id}}^?}) \}) \\
   \end{array}


.. index:: import, name
   pair: text format; import
.. index:: export, name, index, memory index
   pair: text format; export
.. index:: data, memory, memory index, expression, byte, page size
   pair: text format; data
   single: memory; data
   single: data; segment
.. _text-mem-abbrev:

Abbreviations
.............

A :ref:`data segment <text-data>` can be given inline with a memory definition, in which case its offset is :math:`0` and the :ref:`limits <text-limits>` of the :ref:`memory type <text-memtype>` are inferred from the length of the data, rounded up to :ref:`page size <page-size>`:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tmem}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{memory}’}~~{{\mathit{id}}^?}{:}{{\Tid}^?}~~{{\mathit{at}}^?}{:}{{\Taddrtype}^?}~~\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{data}’}~~{b^\ast}{:}{\Tdatastring}~~\mbox{‘\texttt{{)}}’}~~\mbox{‘\texttt{{)}}’} & \quad\equiv\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}l@{}l@{}}
   \begin{array}[t]{@{}l@{}} \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{memory}’}~~{\mathit{id}'}{:}{\Tid}~~{{\mathit{at}}^?}{:}{{\Taddrtype}^?}~~n{:}{\Tu64}~~n{:}{\Tu64}~~\mbox{‘\texttt{{)}}’} \\
     \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{data}’}~~\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{memory}’}~~{\mathit{id}'}{:}{\Tid}~~\mbox{‘\texttt{{)}}’}~~\mbox{‘\texttt{{(}}’}~~{\mathit{at}'}{:}{\Taddrtype}~~\mbox{‘\texttt{.const}’}~~\mbox{‘\texttt{0}’}~~\mbox{‘\texttt{{)}}’}~~{\Tdatastring}~~\mbox{‘\texttt{{)}}’} \end{array} & & \\
    \multicolumn{3}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ {{\mathit{id}}^?} = {\mathit{id}'} \lor {{\mathit{id}}^?} = \epsilon \land {\mathit{id}'} \notin {\I}{.}\IMEMS \\
   {\land}~ {{\mathit{at}}^?} = {\mathit{at}'} \lor {{\mathit{at}}^?} = \epsilon \land {\mathit{at}'} = \I32 \\
   {\land}~ n = {\mathrm{ceil}}({|{b^\ast}|} / 64 \cdot {\mathrm{Ki}}) \\
   \end{array} \\
   \end{array}
   } \\
   \end{array}
   } \\
   \end{array}

Memories can be defined as :ref:`imports <text-import>` or :ref:`exports <text-export>` inline:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Timport}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{memory}’}~~{{\Tid}^?}~~\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{import}’}~~{{\Tname}^{2}}~~\mbox{‘\texttt{{)}}’}~~{{\Tmemtype}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\equiv\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{import}’}~~{{\Tname}^{2}}~~\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{memory}’}~~{{\Tid}^?}~~{{\Tmemtype}}_{{\I}}~~\mbox{‘\texttt{{)}}’}~~\mbox{‘\texttt{{)}}’} \\
   \end{array}
   } \\[0.8ex]
   & {{\Texport}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{memory}’}~~{{\mathit{id}}^?}{:}{{\Tid}^?}~~\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{export}’}~~{\Tname}~~\mbox{‘\texttt{{)}}’}~~\dots~~\mbox{‘\texttt{{)}}’} & \quad\equiv\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}l@{}l@{}}
   \begin{array}[t]{@{}l@{}} \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{memory}’}~~{\mathit{id}'}{:}{\Tid}~~\dots~~\mbox{‘\texttt{{)}}’} \\
     \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{export}’}~~{\Tname}~~\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{memory}’}~~{\Tid}~~\mbox{‘\texttt{{)}}’}~~\mbox{‘\texttt{{)}}’} \end{array} & & \\
    \multicolumn{3}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \quad \mbox{if}~ {{\mathit{id}}^?} = {\mathit{id}'} \lor {{\mathit{id}}^?} = \epsilon \land {\mathit{id}'} \notin {\I}{.}\IMEMS \\
   \end{array}
   } \\
   \end{array}
   } \\
   \end{array}

.. note::
   The latter abbreviation can be applied repeatedly, if ":math:`\dots`" contains additional export clauses.
   Consequently, a memory declaration can contain any number of exports, possibly followed by an import.


.. index:: table, table type, identifier, expression
   pair: text format; table
.. _text-table:

Tables
~~~~~~

Table definitions can bind a symbolic :ref:`table identifier <text-id>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Ttable}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{table}’}~~{{\mathit{id}}^?}{:}{{\Tid}^?}~~{\mathit{tt}}{:}{{\Ttabletype}}_{{\I}}~~e{:}{{\Texpr}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & (\TABLE~{\mathit{tt}}~e, \{ \ITABLES~({{\mathit{id}}^?}) \}) \\
   \end{array}


.. index:: reference type, heap type
.. index:: import, name
   pair: text format; import
.. index:: export, name, index, table index
   pair: text format; export
.. index:: element, table index, function index
   pair: text format; element
   single: table; element
   single: element; segment
.. _text-table-abbrev:

Abbreviations
.............

A table's initialization :ref:`expression <text-expr>` can be omitted, in which case it defaults to :math:`\mathsf{ref{.}null}`:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Ttable}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{table}’}~~{{\Tid}^?}~~{\mathit{tt}}{:}{{\Ttabletype}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\equiv\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}l@{}l@{}}
   \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{table}’}~~{{\Tid}^?}~~{\mathit{tt}}{:}{{\Ttabletype}}_{{\I}}~~\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{ref.null}’}~~{\mathit{ht}}{:}{{\Theaptype}}_{{\I}}~~\mbox{‘\texttt{{)}}’}~~\mbox{‘\texttt{{)}}’} & & \\
    \multicolumn{3}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \quad \mbox{if}~ {\mathit{tt}} = {\mathit{at}}~{\mathit{lim}}~(\REF~{\NULL^?}~{\mathit{ht}}) \\
   \end{array}
   } \\
   \end{array}
   } \\
   \end{array}

An :ref:`element segment <text-elem>` can be given inline with a table definition, in which case its offset is :math:`0` and the :ref:`limits <text-limits>` of the :ref:`table type <text-tabletype>` are inferred from the length of the given segment:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Ttable}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{table}’}~~{{\mathit{id}}^?}{:}{{\Tid}^?}~~{{\mathit{at}}^?}{:}{{\Taddrtype}^?}~~{{\Treftype}}_{{\I}}~~\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{elem}’}~~({\mathit{rt}}, {e^\ast}){:}{{\Telemlist}}_{{\I}}~~\mbox{‘\texttt{{)}}’}~~\mbox{‘\texttt{{)}}’} & \quad\equiv\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}l@{}l@{}}
   \begin{array}[t]{@{}l@{}} \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{table}’}~~{\mathit{id}'}{:}{\Tid}~~{{\mathit{at}}^?}{:}{{\Taddrtype}^?}~~n{:}{\Tu64}~~n{:}{\Tu64}~~{{\Treftype}}_{{\I}}~~\mbox{‘\texttt{{)}}’} \\
     \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{elem}’}~~\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{table}’}~~{\mathit{id}'}{:}{\Tid}~~\mbox{‘\texttt{{)}}’}~~\mbox{‘\texttt{{(}}’}~~{\mathit{at}'}{:}{\Taddrtype}~~\mbox{‘\texttt{.const}’}~~\mbox{‘\texttt{0}’}~~\mbox{‘\texttt{{)}}’}~~{{\Telemlist}}_{{\I}}~~\mbox{‘\texttt{{)}}’} \end{array} & & \\
    \multicolumn{3}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ {{\mathit{id}}^?} = {\mathit{id}'} \lor {{\mathit{id}}^?} = \epsilon \land {\mathit{id}'} \notin {\I}{.}\ITABLES \\
   {\land}~ {{\mathit{at}}^?} = {\mathit{at}'} \lor {{\mathit{at}}^?} = \epsilon \land {\mathit{at}'} = \I32 \\
   {\land}~ n = {|{e^\ast}|} \\
   \end{array} \\
   \end{array}
   } \\
   \end{array}
   } \\
   \end{array}

Tables can be defined as :ref:`imports <text-import>` or :ref:`exports <text-export>` inline:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Timport}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{table}’}~~{{\Tid}^?}~~\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{import}’}~~{{\Tname}^{2}}~~\mbox{‘\texttt{{)}}’}~~{{\Ttabletype}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\equiv\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{import}’}~~{{\Tname}^{2}}~~\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{table}’}~~{{\Tid}^?}~~{{\Ttabletype}}_{{\I}}~~\mbox{‘\texttt{{)}}’}~~\mbox{‘\texttt{{)}}’} \\
   \end{array}
   } \\[0.8ex]
   & {{\Texport}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{table}’}~~{{\mathit{id}}^?}{:}{{\Tid}^?}~~\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{export}’}~~{\Tname}~~\mbox{‘\texttt{{)}}’}~~\dots~~\mbox{‘\texttt{{)}}’} & \quad\equiv\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}l@{}l@{}}
   \begin{array}[t]{@{}l@{}} \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{table}’}~~{\mathit{id}'}{:}{\Tid}~~\dots~~\mbox{‘\texttt{{)}}’} \\
     \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{export}’}~~{\Tname}~~\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{table}’}~~{\Tid}~~\mbox{‘\texttt{{)}}’}~~\mbox{‘\texttt{{)}}’} \end{array} & & \\
    \multicolumn{3}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \quad \mbox{if}~ {{\mathit{id}}^?} = {\mathit{id}'} \lor {{\mathit{id}}^?} = \epsilon \land {\mathit{id}'} \notin {\I}{.}\ITABLES \\
   \end{array}
   } \\
   \end{array}
   } \\
   \end{array}

.. note::
   The latter abbreviation can be applied repeatedly, if ":math:`\dots`" contains additional export clauses.
   Consequently, a table declaration can contain any number of exports, possibly followed by an import.


.. index:: function, type index, function type, identifier, local
   pair: text format; function
   pair: text format; local
.. _text-local:
.. _text-func:

Functions
~~~~~~~~~

Function definitions can bind a symbolic :ref:`function identifier <text-id>`, and :ref:`local identifiers <text-id>` for its :ref:`parameters <text-typeuse>` and locals.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tfunc}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{func}’}~~{{\mathit{id}}^?}{:}{{\Tid}^?}~~(x, {\I}_1){:}{{\Ttypeuse}}_{{\I}}~~{(({{\mathit{loc}}^\ast}, {\I}_2){:}{{\Tlocal}}_{{\I}})^\ast}~~e{:}{{\Texpr}}_{{\I'}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}l@{}l@{}}
   (\FUNC~x~({\bigcat}\, {{{\mathit{loc}}^\ast}^\ast})~e, \{ \IFUNCS~({{\mathit{id}}^?}) \}) & & \\
    \multicolumn{3}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ {\I'} = {\I} \oplus {\I}_1 \oplus {\bigcat}\, {{\I}_2^\ast} \\
   {\land}~ {\vdash}\, {\I'} : \mathsf{ok} \\
   \end{array} \\
   \end{array}
   } \\
   \end{array}
   } \\
   \end{array}

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tlocal}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{local}’}~~{{\mathit{id}}^?}{:}{{\Tid}^?}~~t{:}{{\Tvaltype}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   (\LOCAL~t, \{ \ILOCALS~({{\mathit{id}}^?}) \}) \\
   \end{array}
   } \\
   \end{array}

.. note::
   The :ref:`well-formedness <text-context-wf>` condition on :math:`{\I'}` ensures that parameters and locals do not contain duplicate identifiers.


.. index:: import, name
   pair: text format; import
.. index:: export, name, index, function index
   pair: text format; export
.. _text-func-abbrev:

Abbreviations
.............

Multiple anonymous locals may be combined into a single declaration:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tlocal}}_{{\I}} & ::= & \dots ~~|~~ \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{local}’}~~{{{\Tvaltype}}_{{\I}}^\ast}~~\mbox{‘\texttt{{)}}’} & \quad\equiv\quad{} & {(\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{local}’}~~{{\Tvaltype}}_{{\I}}~~\mbox{‘\texttt{{)}}’})^\ast} \\
   \end{array}

Functions can be defined as :ref:`imports <text-import>` or :ref:`exports <text-export>` inline:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Timport}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{func}’}~~{{\Tid}^?}~~\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{import}’}~~{{\Tname}^{2}}~~\mbox{‘\texttt{{)}}’}~~{{\Ttypeuse}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\equiv\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{import}’}~~{{\Tname}^{2}}~~\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{func}’}~~{{\Tid}^?}~~{{\Ttypeuse}}_{{\I}}~~\mbox{‘\texttt{{)}}’}~~\mbox{‘\texttt{{)}}’} \\
   \end{array}
   } \\[0.8ex]
   & {{\Texport}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{func}’}~~{{\mathit{id}}^?}{:}{{\Tid}^?}~~\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{export}’}~~{\Tname}~~\mbox{‘\texttt{{)}}’}~~\dots~~\mbox{‘\texttt{{)}}’} & \quad\equiv\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}l@{}l@{}}
   \begin{array}[t]{@{}l@{}} \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{func}’}~~{\mathit{id}'}{:}{\Tid}~~\dots~~\mbox{‘\texttt{{)}}’} \\
     \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{export}’}~~{\Tname}~~\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{func}’}~~{\Tid}~~\mbox{‘\texttt{{)}}’}~~\mbox{‘\texttt{{)}}’} \end{array} & & \\
    \multicolumn{3}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \quad \mbox{if}~ {{\mathit{id}}^?} = {\mathit{id}'} \lor {{\mathit{id}}^?} = \epsilon \land {\mathit{id}'} \notin {\I}{.}\IFUNCS \\
   \end{array}
   } \\
   \end{array}
   } \\
   \end{array}

.. note::
   The latter abbreviation can be applied repeatedly, if ":math:`\dots`" contains additional export clauses.
   Consequently, a function declaration can contain any number of exports, possibly followed by an import.


.. index:: data, memory, memory index, expression, byte
   pair: text format; data
   single: memory; data
   single: data; segment
.. _text-datastring:
.. _text-data:
.. _text-memuse:
.. _text-offsetexpr:

Data Segments
~~~~~~~~~~~~~

Data segments allow for an optional :ref:`memory index <text-memidx>` to identify the memory to initialize.
The data is written as a :ref:`string <text-string>`, which may be split up into a possibly empty sequence of individual string literals.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tdata}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{data}’}~~{{\mathit{id}}^?}{:}{{\Tid}^?}~~{b^\ast}{:}{\Tdatastring}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   (\DATA~{b^\ast}~\DPASSIVE, \{ \IDATAS~({{\mathit{id}}^?}) \}) \\
   \end{array}
   } \\
   & & | & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{data}’}~~{{\mathit{id}}^?}{:}{{\Tid}^?}~~x{:}{{\Tmemuse}}_{{\I}}~~e{:}{{\Toffsetexpr}}_{{\I}}~~{b^\ast}{:}{\Tdatastring}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   (\DATA~{b^\ast}~(\DACTIVE~x~e), \{ \IDATAS~({{\mathit{id}}^?}) \}) \\
   \end{array}
   } \\[0.8ex]
   & {\Tdatastring} & ::= & {{b^\ast}^\ast}{:}{{\Tstring}^\ast} & \quad\Rightarrow\quad{} & {\bigcat}\, {{b^\ast}^\ast} \\[0.8ex]
   & {{\Tmemuse}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{memory}’}~~x{:}{{\Tmemidx}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & x \\
   & {{\Toffsetexpr}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{offset}’}~~e{:}{{\Texpr}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & e \\
   \end{array}


Abbreviations
.............

As an abbreviation, a single :ref:`folded instruction <text-foldedinstr>` may occur in place of the offset of an active segment:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Toffsetexpr}}_{{\I}} & ::= & \dots ~~|~~ {{\Tfoldedinstr}}_{{\I}} & \quad\equiv\quad{} & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{offset}’}~~{{\Tfoldedinstr}}_{{\I}}~~\mbox{‘\texttt{{)}}’} \\
   \end{array}

Also, a memory use can be omitted, defaulting to :math:`0`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tmemuse}}_{{\I}} & ::= & \dots ~~|~~ \epsilon & \quad\equiv\quad{} & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{memory}’}~~\mbox{‘\texttt{0}’}~~\mbox{‘\texttt{{)}}’} \\
   \end{array}

As another abbreviation, data segments may also be specified inline with :ref:`memory <text-mem>` definitions; see the respective section.


.. index:: element, table index, expression, function index
   pair: text format; element
   single: table; element
   single: element; segment
.. _text-elem:
.. _text-elemlist:
.. _text-elemexpr:
.. _text-tableuse:

Element Segments
~~~~~~~~~~~~~~~~

Element segments allow for an optional :ref:`table index <text-tableidx>` to identify the table to initialize.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Telem}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{elem}’}~~{{\mathit{id}}^?}{:}{{\Tid}^?}~~({\mathit{rt}}, {e^\ast}){:}{{\Telemlist}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   (\ELEM~{\mathit{rt}}~{e^\ast}~\EPASSIVE, \{ \IELEMS~({{\mathit{id}}^?}) \}) \\
   \end{array}
   } \\
   & & | & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{elem}’}~~{{\mathit{id}}^?}{:}{{\Tid}^?}~~x{:}{{\Ttableuse}}_{{\I}}~~{e'}{:}{{\Toffsetexpr}}_{{\I}}~~({\mathit{rt}}, {e^\ast}){:}{{\Telemlist}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   (\ELEM~{\mathit{rt}}~{e^\ast}~(\EACTIVE~x~{e'}), \{ \IELEMS~({{\mathit{id}}^?}) \}) \\
   \end{array}
   } \\
   & & | & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{elem}’}~~{{\mathit{id}}^?}{:}{{\Tid}^?}~~\mbox{‘\texttt{declare}’}~~({\mathit{rt}}, {e^\ast}){:}{{\Telemlist}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   (\ELEM~{\mathit{rt}}~{e^\ast}~\EDECLARE, \{ \IELEMS~({{\mathit{id}}^?}) \}) \\
   \end{array}
   } \\[0.8ex]
   & {{\Telemlist}}_{{\I}} & ::= & {\mathit{rt}}{:}{{\Treftype}}_{{\I}}~~{e^\ast}{:}{\Tlist}({{\Telemexpr}}_{{\I}}) & \quad\Rightarrow\quad{} & ({\mathit{rt}}, {e^\ast}) \\
   & {{\Telemexpr}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{item}’}~~e{:}{{\Texpr}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & e \\[0.8ex]
   & {{\Ttableuse}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{table}’}~~x{:}{{\Ttableidx}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & x \\
   \end{array}


Abbreviations
.............

As an abbreviation, a single :ref:`folded instruction <text-foldedinstr>` may occur in place of the offset of an active element segment or as an element expression:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Telemexpr}}_{{\I}} & ::= & \dots ~~|~~ {{\Tfoldedinstr}}_{{\I}} & \quad\equiv\quad{} & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{item}’}~~{{\Tfoldedinstr}}_{{\I}}~~\mbox{‘\texttt{{)}}’} \\
   \end{array}

Also, the element list may be written as just a sequence of :ref:`function indices <text-funcidx>`:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Telemlist}}_{{\I}} & ::= & \dots ~~|~~ \mbox{‘\texttt{func}’}~~{x^\ast}{:}{{{\Tfuncidx}}_{{\I}}^\ast} & \quad\equiv\quad{} & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{ref}’}~~\mbox{‘\texttt{func}’}~~\mbox{‘\texttt{{)}}’}~~{(\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{ref.func}’}~~{{\Tfuncidx}}_{{\I}}~~\mbox{‘\texttt{{)}}’})^\ast} \\
   \end{array}

A table use can be omitted, defaulting to :math:`0`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Ttableuse}}_{{\I}} & ::= & \dots ~~|~~ \epsilon & \quad\equiv\quad{} & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{table}’}~~\mbox{‘\texttt{0}’}~~\mbox{‘\texttt{{)}}’} \\
   \end{array}

Furthermore, for backwards compatibility with earlier versions of WebAssembly, if the table use is omitted, the :math:`\mbox{‘\texttt{func}’}` keyword can be omitted as well.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Telem}}_{{\I}} & ::= & \dots \\
   & & | & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{elem}’}~~{{\Toffsetexpr}}_{{\I}}~~{\Tlist}({{\Tfuncidx}}_{{\I}})~~\mbox{‘\texttt{{)}}’} & \quad\equiv\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{elem}’}~~{{\Toffsetexpr}}_{{\I}}~~\mbox{‘\texttt{func}’}~~{\Tlist}({{\Tfuncidx}}_{{\I}})~~\mbox{‘\texttt{{)}}’} \\
   \end{array}
   } \\
   \end{array}

As yet another abbreviation, element segments may also be specified inline with :ref:`table <text-table>` definitions; see the respective section.


.. index:: start function, function index
   pair: text format; start function
.. _text-start:

Start Function
~~~~~~~~~~~~~~

A :ref:`start function <syntax-start>` is defined in terms of its index.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tstart}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{start}’}~~x{:}{{\Tfuncidx}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & (\START~x, \{  \}) \\
   \end{array}

.. note::
   At most one start function may occur in a module,
   which is ensured by a suitable side condition on the :math:`{\Tmodule}` grammar.



.. index:: import, name, tag type, global type, memory type, table type, function type
   pair: text format; import
.. _text-import:

Imports
~~~~~~~

The :ref:`external type <syntax-externtype>` in imports can bind a symbolic tag, global, memory, or function :ref:`identifier <text-id>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Timport}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{import}’}~~{\mathit{nm}}_1{:}{\Tname}~~{\mathit{nm}}_2{:}{\Tname}~~({\mathit{xt}}, {\I'}){:}{{\Texterntype}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & (\IMPORT~{\mathit{nm}}_1~{\mathit{nm}}_2~{\mathit{xt}}, {\I'}) \\
   \end{array}


Abbreviations
.............

As an abbreviation, imports may also be specified inline with
:ref:`tag <text-tag>`,
:ref:`global <text-global>`,
:ref:`memory <text-mem>`,
:ref:`table <text-table>`, or
:ref:`function <text-func>`
definitions; see the respective sections.


.. index:: export, name, index, external index, tag index, global index, memory index, table index, function index
   pair: text format; export
.. _text-externidx:
.. _text-export:

Exports
~~~~~~~

The syntax for exports mirrors their :ref:`abstract syntax <syntax-export>` directly.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Texport}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{export}’}~~{\mathit{nm}}{:}{\Tname}~~{\mathit{xx}}{:}{{\Texternidx}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & (\EXPORT~{\mathit{nm}}~{\mathit{xx}}, \{  \}) \\[0.8ex]
   & {{\Texternidx}}_{{\I}} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{tag}’}~~x{:}{{\Ttagidx}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & \XXTAG~x \\
   & & | & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{global}’}~~x{:}{{\Tglobalidx}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & \XXGLOBAL~x \\
   & & | & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{memory}’}~~x{:}{{\Tmemidx}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & \XXMEM~x \\
   & & | & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{table}’}~~x{:}{{\Ttableidx}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & \XXTABLE~x \\
   & & | & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{func}’}~~x{:}{{\Tfuncidx}}_{{\I}}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & \XXFUNC~x \\
   \end{array}


Abbreviations
.............

As an abbreviation, exports may also be specified inline with
:ref:`tag <text-tag>`,
:ref:`global <text-global>`,
:ref:`memory <text-mem>`,
:ref:`table <text-table>`, or
:ref:`function <text-func>`
definitions; see the respective sections.


.. index:: module, type definition, recursive type, tag, global, memory, table, function, data segment, element segment, start function, import, export, identifier context, identifier, name section, ! declaration
   pair: text format; module
   single: section; name
.. _syntax-decl:
.. _text-decl:
.. _text-module:

Modules
~~~~~~~

A module consists of a sequence of *declarations* that can occur in any order.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}}
   & {\decl} & ::= & {\type} ~~|~~ {\import} ~~|~~ {\tag} ~~|~~ {\global} ~~|~~ {\mem} ~~|~~ {\table} ~~|~~ {\func} ~~|~~ {\data} ~~|~~ {\elem} ~~|~~ {\start} ~~|~~ {\export} \\
   \end{array}

All declarations and their respective bound :ref:`identifiers <text-id>` scope over the entire module, including the text preceding them.
A module itself may optionally bind an :ref:`identifier <text-id>` that names the module.
The name serves a documentary role only.

.. note::
   Tools may include the module name in the :ref:`name section <binary-namesec>` of the :ref:`binary format <binary>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Tdecl}}_{{\I}} & ::= & {{\Ttype}}_{{\I}} ~~|~~ {{\Timport}}_{{\I}} ~~|~~ {{\Ttag}}_{{\I}} ~~|~~ {{\Tglobal}}_{{\I}} ~~|~~ {{\Tmem}}_{{\I}} ~~|~~ {{\Ttable}}_{{\I}} \\
   & & | & {{\Tfunc}}_{{\I}} ~~|~~ {{\Tdata}}_{{\I}} ~~|~~ {{\Telem}}_{{\I}} ~~|~~ {{\Tstart}}_{{\I}} ~~|~~ {{\Texport}}_{{\I}} \\
   \end{array}

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Tmodule} & ::= & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{module}’}~~{{\Tid}^?}~~{({\decl}, {\I})^\ast}{:}{{{\Tdecl}}_{{\I'}}^\ast}~~\mbox{‘\texttt{{)}}’} & \quad\Rightarrow\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}l@{}l@{}}
   \MODULE~{{\type}^\ast}~{{\import}^\ast}~{{\tag}^\ast}~{{\global}^\ast}~{{\mem}^\ast}~{{\table}^\ast}~{{\func}^\ast}~{{\data}^\ast}~{{\elem}^\ast}~{{\start}^?}~{{\export}^\ast} & & \\
    \multicolumn{3}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ {\I'} = {\bigcat}\, {{\I}^\ast} \\
   {\land}~ {\vdash}\, {\I'} : \mathsf{ok} \\
   {\land}~ {{\type}^\ast} = {\typesd}({{\decl}^\ast}) \\
   {\land}~ {{\import}^\ast} = {\importsd}({{\decl}^\ast}) \\
   {\land}~ {{\tag}^\ast} = {\tagsd}({{\decl}^\ast}) \\
   {\land}~ {{\global}^\ast} = {\globalsd}({{\decl}^\ast}) \\
   {\land}~ {{\mem}^\ast} = {\memsd}({{\decl}^\ast}) \\
   {\land}~ {{\table}^\ast} = {\tablesd}({{\decl}^\ast}) \\
   {\land}~ {{\func}^\ast} = {\funcsd}({{\decl}^\ast}) \\
   {\land}~ {{\data}^\ast} = {\datasd}({{\decl}^\ast}) \\
   {\land}~ {{\elem}^\ast} = {\elemsd}({{\decl}^\ast}) \\
   {\land}~ {{\start}^?} = {\startsd}({{\decl}^\ast}) \\
   {\land}~ {{\export}^\ast} = {\exportsd}({{\decl}^\ast}) \\
   {\land}~ {\ordered}({{\decl}^\ast}) \\
   \end{array} \\
   \end{array}
   } \\
   \end{array}
   } \\
   \end{array}

where :math:`{\mathrm{types}}({{\decl}^\ast})`, :math:`{\mathrm{imports}}({{\decl}^\ast})`, :math:`{\mathrm{tags}}({{\decl}^\ast})`, etc., extract the sequence of :ref:`types <syntax-type>`, :ref:`imports <syntax-import>`, :ref:`tags <syntax-tag>`, etc., contained in :math:`{{\decl}^\ast}`, respectively.
The auxiliary predicate :math:`{\ordered}` checks that no imports occur after the first definition of a
:ref:`tag <syntax-tag>`,
:ref:`global <syntax-global>`,
:ref:`memory <syntax-mem>`,
:ref:`table <syntax-table>`, or
:ref:`function <syntax-func>`
in a sequence of declarations:

.. _aux-ordered:

.. math::
   \begin{array}[t]{@{}lcl@{}l@{}}
   {\ordered}({{\decl}^\ast}) & = & \mathsf{true} & \quad \mbox{if}~ {\importsd}({{\decl}^\ast}) = \epsilon \\
   {\ordered}({{\decl}_1^\ast}~{\import}~{{\decl}_2^\ast}) & = & & \\
    \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   {\importsd}({{\decl}_1^\ast}) = \epsilon \land {\tagsd}({{\decl}_1^\ast}) = \epsilon \land {\globalsd}({{\decl}_1^\ast}) = \epsilon \land {\memsd}({{\decl}_1^\ast}) = \epsilon \land {\tablesd}({{\decl}_1^\ast}) = \epsilon \land {\funcsd}({{\decl}_1^\ast}) = \epsilon \\
   \end{array}
   } \\
   \end{array}




Abbreviations
.............

In a source file, the toplevel :math:`\mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{module}’}~~\dots~~\mbox{‘\texttt{{)}}’}` surrounding the module body may be omitted.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Tmodule} & ::= & \dots ~~|~~ {{{\Tdecl}}_{{\I}}^\ast} & \quad\equiv\quad{} & \mbox{‘\texttt{{(}}’}~~\mbox{‘\texttt{module}’}~~{{{\Tdecl}}_{{\I}}^\ast}~~\mbox{‘\texttt{{)}}’} \\
   \end{array}
