Modules
-------

The binary encoding of modules is organized into *sections*.
Most sections correspond to one component of a :ref:`module <syntax-module>` record,
except that :ref:`function definitions <syntax-func>` are split into two sections, separating their type declarations in the :ref:`function section <binary-funcsec>` from their bodies in the :ref:`code section <binary-codesec>`.

.. note::
   This separation enables *parallel* and *streaming* compilation of the functions in a module.


.. index:: index, type index, tag index, global index, memory index, table index, function index, data index, element index, local index, label index, field index
   pair: binary format; type index
   pair: binary format; tag index
   pair: binary format; global index
   pair: binary format; memory index
   pair: binary format; table index
   pair: binary format; function index
   pair: binary format; data index
   pair: binary format; element index
   pair: binary format; local index
   pair: binary format; label index
   pair: binary format; field index
.. _binary-typeidx:
.. _binary-tagidx:
.. _binary-globalidx:
.. _binary-memidx:
.. _binary-tableidx:
.. _binary-funcidx:
.. _binary-dataidx:
.. _binary-elemidx:
.. _binary-localidx:
.. _binary-labelidx:
.. _binary-fieldidx:
.. _binary-externidx:
.. _binary-index:

Indices
~~~~~~~

All basic :ref:`indices <syntax-index>` are encoded with their respective value.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Btypeidx} & ::= & x{:}{\Bu32} & \quad\Rightarrow\quad{} & x \\
   & {\Bfuncidx} & ::= & x{:}{\Bu32} & \quad\Rightarrow\quad{} & x \\
   & {\Btableidx} & ::= & x{:}{\Bu32} & \quad\Rightarrow\quad{} & x \\
   & {\Bmemidx} & ::= & x{:}{\Bu32} & \quad\Rightarrow\quad{} & x \\
   & {\Bglobalidx} & ::= & x{:}{\Bu32} & \quad\Rightarrow\quad{} & x \\
   & {\Btagidx} & ::= & x{:}{\Bu32} & \quad\Rightarrow\quad{} & x \\
   & {\Belemidx} & ::= & x{:}{\Bu32} & \quad\Rightarrow\quad{} & x \\
   & {\Bdataidx} & ::= & x{:}{\Bu32} & \quad\Rightarrow\quad{} & x \\
   & {\Blocalidx} & ::= & x{:}{\Bu32} & \quad\Rightarrow\quad{} & x \\
   & {\Bfieldidx} & ::= & x{:}{\Bu32} & \quad\Rightarrow\quad{} & x \\
   & {\Blabelidx} & ::= & l{:}{\Bu32} & \quad\Rightarrow\quad{} & l \\
   \end{array}

:ref:`External indices <syntax-externidx>` are encoded by a distiguishing byte followed by an encoding of their respective value.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Bexternidx} & ::= & \mathtt{0x00}~~x{:}{\Bfuncidx} & \quad\Rightarrow\quad{} & \XXFUNC~x \\
   & & | & \mathtt{0x01}~~x{:}{\Btableidx} & \quad\Rightarrow\quad{} & \XXTABLE~x \\
   & & | & \mathtt{0x02}~~x{:}{\Bmemidx} & \quad\Rightarrow\quad{} & \XXMEM~x \\
   & & | & \mathtt{0x03}~~x{:}{\Bglobalidx} & \quad\Rightarrow\quad{} & \XXGLOBAL~x \\
   & & | & \mathtt{0x04}~~x{:}{\Btagidx} & \quad\Rightarrow\quad{} & \XXTAG~x \\
   \end{array}


.. index:: ! section
   pair: binary format; section
.. _binary-section:

Sections
~~~~~~~~

Each section consists of

* a one-byte section *id*,
* the :math:`{\u32}` *length* of the contents, in bytes,
* the actual *contents*, whose structure is dependent on the section id.

Every section is optional; an omitted section is equivalent to the section being present with empty contents.

The following parameterized grammar rule defines the generic structure of a section with id :math:`N` and contents described by the grammar :math:`{\mathtt{X}}`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\Bsection}}_{N}({\mathtt{X}}) & ::= & N{:}{\Bbyte}~~{\mathit{len}}{:}{\Bu32}~~{{\mathit{en}}^\ast}{:}{\mathtt{X}} & \quad\Rightarrow\quad{} & {{\mathit{en}}^\ast} & \quad \mbox{if}~ {\mathit{len}} = ||{\mathtt{X}}|| \\
   & & | & \epsilon & \quad\Rightarrow\quad{} & \epsilon \\
   \end{array}

For most sections, the contents :math:`{\mathtt{X}}` encodes a :ref:`list <binary-list>`.
In these cases, the empty result :math:`\epsilon` is interpreted as the empty list.

.. note::
   Other than for unknown :ref:`custom sections <binary-customsec>`,
   the :math:`{\mathit{size}}` is not required for decoding, but can be used to skip sections when navigating through a binary.
   The module is malformed if the size does not match the length of the binary contents :math:`{\mathtt{X}}`.

The following section ids are used:

==  ===============================================
Id  Section                                        
==  ===============================================
 0  :ref:`custom section <binary-customsec>`       
 1  :ref:`type section <binary-typesec>`           
 2  :ref:`import section <binary-importsec>`       
 3  :ref:`function section <binary-funcsec>`       
 4  :ref:`table section <binary-tablesec>`         
 5  :ref:`memory section <binary-memsec>`          
 6  :ref:`global section <binary-globalsec>`       
 7  :ref:`export section <binary-exportsec>`       
 8  :ref:`start section <binary-startsec>`         
 9  :ref:`element section <binary-elemsec>`        
10  :ref:`code section <binary-codesec>`           
11  :ref:`data section <binary-datasec>`           
12  :ref:`data count section <binary-datacntsec>`
13  :ref:`tag section <binary-tagsec>`
==  ===============================================

.. note::
   Section ids do not always correspond to the :ref:`order of sections <binary-module>` in the encoding of a module.


.. index:: ! custom section
   pair: binary format; custom section
   single: section; custom
.. _binary-customsec:

Custom Section
~~~~~~~~~~~~~~

*Custom sections* have the id 0.
They are intended to be used for debugging information or third-party extensions, and are ignored by the WebAssembly semantics.
Their contents consist of a :ref:`name <syntax-name>` further identifying the custom section, followed by an uninterpreted sequence of bytes for custom use.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Bcustomsec} & ::= & {{\Bsection}}_{0}({\Bcustom}) \\
   & {\Bcustom} & ::= & {\Bname}~~{{\Bbyte}^\ast} \\
   \end{array}

.. note::
   If an implementation interprets the data of a custom section, then errors in that data, or the placement of the section, must not invalidate the module.


.. index:: ! type section, type definition, recursive type
   pair: binary format; type section
   pair: section; type
.. _binary-type:
.. _binary-typesec:

Type Section
~~~~~~~~~~~~

The *type section* has the id 1.
It decodes into the list of :ref:`recursive types <syntax-rectype>` of a :ref:`module <syntax-module>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Btypesec} & ::= & {{\mathit{ty}}^\ast}{:}{{\Bsection}}_{1}({\Blist}({\Btype})) & \quad\Rightarrow\quad{} & {{\mathit{ty}}^\ast} \\
   & {\Btype} & ::= & {\mathit{qt}}{:}{\Brectype} & \quad\Rightarrow\quad{} & \TYPE~{\mathit{qt}} \\
   \end{array}


.. index:: ! import section, import, name, function type, table type, memory type, global type, tag type
   pair: binary format; import
   pair: section; import
.. _binary-import:
.. _binary-importdesc:
.. _binary-importsec:

Import Section
~~~~~~~~~~~~~~

The *import section* has the id 2.
It decodes into the list of :ref:`imports <syntax-import>` of a :ref:`module <syntax-module>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Bimportsec} & ::= & {{\mathit{im}}^\ast}{:}{{\Bsection}}_{2}({\Blist}({\Bimport})) & \quad\Rightarrow\quad{} & {{\mathit{im}}^\ast} \\
   & {\Bimport} & ::= & {\mathit{nm}}_1{:}{\Bname}~~{\mathit{nm}}_2{:}{\Bname}~~{\mathit{xt}}{:}{\Bexterntype} & \quad\Rightarrow\quad{} & \IMPORT~{\mathit{nm}}_1~{\mathit{nm}}_2~{\mathit{xt}} \\
   \end{array}


.. index:: ! function section, function, type index, function type
   pair: binary format; function
   pair: section; function
.. _binary-funcsec:

Function Section
~~~~~~~~~~~~~~~~

The *function section* has the id 3.
It decodes into a list of :ref:`type indices <syntax-typeidx>` that classify the :ref:`functions <syntax-func>` defined by a :ref:`module <syntax-module>`.
The bodies of the respective functions are encoded separately in the :ref:`code section <binary-codesec>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Bfuncsec} & ::= & {x^\ast}{:}{{\Bsection}}_{3}({\Blist}({\Btypeidx})) & \quad\Rightarrow\quad{} & {x^\ast} \\
   \end{array}


.. index:: ! table section, table, table type
   pair: binary format; table
   pair: section; table
.. _binary-table:
.. _binary-tablesec:

Table Section
~~~~~~~~~~~~~

The *table section* has the id 4.
It decodes into the list of :ref:`tables <syntax-table>` defined by a :ref:`module <syntax-module>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Btablesec} & ::= & {{\mathit{tab}}^\ast}{:}{{\Bsection}}_{4}({\Blist}({\Btable})) & \quad\Rightarrow\quad{} & {{\mathit{tab}}^\ast} \\
   & {\Btable} & ::= & {\mathit{tt}}{:}{\Btabletype} & \quad\Rightarrow\quad{} & \TABLE~{\mathit{tt}}~(\REFNULL~{\mathit{ht}}) & \quad \mbox{if}~ {\mathit{tt}} = {\mathit{at}}~{\mathit{lim}}~(\REF~{\NULL^?}~{\mathit{ht}}) \\
   & & | & \mathtt{0x40}~~\mathtt{0x00}~~{\mathit{tt}}{:}{\Btabletype}~~e{:}{\Bexpr} & \quad\Rightarrow\quad{} & \TABLE~{\mathit{tt}}~e \\
   \end{array}

.. note::
   The encoding of a table type cannot start with byte :math:`\mathtt{0x40}``,
   hence decoding is unambiguous.
   The zero byte following it is reserved for future extensions.


.. index:: ! memory section, memory, memory type
   pair: binary format; memory
   pair: section; memory
.. _binary-mem:
.. _binary-memsec:

Memory Section
~~~~~~~~~~~~~~

The *memory section* has the id 5.
It decodes into the list of :ref:`memories <syntax-mem>` defined by a :ref:`module <syntax-module>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Bmemsec} & ::= & {{\mem}^\ast}{:}{{\Bsection}}_{5}({\Blist}({\Bmem})) & \quad\Rightarrow\quad{} & {{\mem}^\ast} \\
   & {\Bmem} & ::= & {\mathit{mt}}{:}{\Bmemtype} & \quad\Rightarrow\quad{} & \MEMORY~{\mathit{mt}} \\
   \end{array}


.. index:: ! global section, global, global type, expression
   pair: binary format; global
   pair: section; global
.. _binary-global:
.. _binary-globalsec:

Global Section
~~~~~~~~~~~~~~

The *global section* has the id 6.
It decodes into the list of :ref:`globals <syntax-global>` defined by a :ref:`module <syntax-module>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Bglobalsec} & ::= & {{\mathit{glob}}^\ast}{:}{{\Bsection}}_{6}({\Blist}({\Bglobal})) & \quad\Rightarrow\quad{} & {{\mathit{glob}}^\ast} \\
   & {\Bglobal} & ::= & {\mathit{gt}}{:}{\Bglobaltype}~~e{:}{\Bexpr} & \quad\Rightarrow\quad{} & \GLOBAL~{\mathit{gt}}~e \\
   \end{array}


.. index:: ! export section, export, name, index, function index, table index, memory index, tag index, global index
   pair: binary format; export
   pair: section; export
.. _binary-export:
.. _binary-exportdesc:
.. _binary-exportsec:

Export Section
~~~~~~~~~~~~~~

The *export section* has the id 7.
It decodes into the list of :ref:`exports <syntax-export>` of a :ref:`module <syntax-module>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Bexportsec} & ::= & {{\mathit{ex}}^\ast}{:}{{\Bsection}}_{7}({\Blist}({\Bexport})) & \quad\Rightarrow\quad{} & {{\mathit{ex}}^\ast} \\
   & {\Bexport} & ::= & {\mathit{nm}}{:}{\Bname}~~{\mathit{xx}}{:}{\Bexternidx} & \quad\Rightarrow\quad{} & \EXPORT~{\mathit{nm}}~{\mathit{xx}} \\
   \end{array}


.. index:: ! start section, start function, function index
   pair: binary format; start function
   single: section; start
   single: start function; section
.. _binary-start:
.. _binary-startsec:

Start Section
~~~~~~~~~~~~~

The *start section* has the id 8.
It decodes into the optional :ref:`start function <syntax-start>` of a :ref:`module <syntax-module>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Bstartsec} & ::= & {{\start}^?}{:}{{\Bsection}}_{8}({\Bstart}) & \quad\Rightarrow\quad{} & {{\start}^?} \\
   & {\Bstart} & ::= & x{:}{\Bfuncidx} & \quad\Rightarrow\quad{} & (\START~x) \\
   \end{array}


.. index:: ! element section, element, table index, expression, function index
   pair: binary format; element
   pair: section; element
   single: table; element
   single: element; segment
.. _binary-elem:
.. _binary-elemsec:
.. _binary-elemkind:

Element Section
~~~~~~~~~~~~~~~

The *element section* has the id 9.
It decodes into the list of :ref:`element segments <syntax-elem>` defined by a :ref:`module <syntax-module>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Belemsec} & ::= & {{\elem}^\ast}{:}{{\Bsection}}_{9}({\Blist}({\Belem})) & \quad\Rightarrow\quad{} & {{\elem}^\ast} \\
   & {\Belemkind} & ::= & \mathtt{0x00} & \quad\Rightarrow\quad{} & \REF~\FUNCT \\
   & {\Belem} & ::= & 0{:}{\Bu32}~~e_o{:}{\Bexpr}~~{y^\ast}{:}{\Blist}({\Bfuncidx}) & \quad\Rightarrow\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \ELEM~(\REF~\FUNCT)~{(\REFFUNC~y)^\ast}~(\EACTIVE~0~e_o) \\
   \end{array}
   } \\
   & & | & 1{:}{\Bu32}~~{\mathit{rt}}{:}{\Belemkind}~~{y^\ast}{:}{\Blist}({\Bfuncidx}) & \quad\Rightarrow\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \ELEM~{\mathit{rt}}~{(\REFFUNC~y)^\ast}~\EPASSIVE \\
   \end{array}
   } \\
   & & | & 2{:}{\Bu32}~~x{:}{\Btableidx}~~e{:}{\Bexpr}~~{\mathit{rt}}{:}{\Belemkind}~~{y^\ast}{:}{\Blist}({\Bfuncidx}) & \quad\Rightarrow\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \ELEM~{\mathit{rt}}~{(\REFFUNC~y)^\ast}~(\EACTIVE~x~e) \\
   \end{array}
   } \\
   & & | & 3{:}{\Bu32}~~{\mathit{rt}}{:}{\Belemkind}~~{y^\ast}{:}{\Blist}({\Bfuncidx}) & \quad\Rightarrow\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \ELEM~{\mathit{rt}}~{(\REFFUNC~y)^\ast}~\EDECLARE \\
   \end{array}
   } \\
   & & | & 4{:}{\Bu32}~~e_{\mathsf{o}}{:}{\Bexpr}~~{e^\ast}{:}{\Blist}({\Bexpr}) & \quad\Rightarrow\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \ELEM~(\REF~\NULL~\FUNCT)~{e^\ast}~(\EACTIVE~0~e_{\mathsf{o}}) \\
   \end{array}
   } \\
   & & | & 5{:}{\Bu32}~~{\mathit{rt}}{:}{\Breftype}~~{e^\ast}{:}{\Blist}({\Bexpr}) & \quad\Rightarrow\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \ELEM~{\mathit{rt}}~{e^\ast}~\EPASSIVE \\
   \end{array}
   } \\
   & & | & 6{:}{\Bu32}~~x{:}{\Btableidx}~~e_{\mathsf{o}}{:}{\Bexpr}~~{\mathit{rt}}{:}{\Breftype}~~{e^\ast}{:}{\Blist}({\Bexpr}) & \quad\Rightarrow\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \ELEM~{\mathit{rt}}~{e^\ast}~(\EACTIVE~x~e_{\mathsf{o}}) \\
   \end{array}
   } \\
   & & | & 7{:}{\Bu32}~~{\mathit{rt}}{:}{\Breftype}~~{e^\ast}{:}{\Blist}({\Bexpr}) & \quad\Rightarrow\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \ELEM~{\mathit{rt}}~{e^\ast}~\EDECLARE \\
   \end{array}
   } \\
   \end{array}

.. note::
   The initial integer can be interpreted as a bitfield.
   Bit 0 distinguishes a passive or declarative segment from an active segment,
   bit 1 indicates the presence of an explicit table index for an active segment and otherwise distinguishes passive from declarative segments,
   bit 2 indicates the use of element type and element :ref:`expressions <binary-expr>` instead of element kind and element indices.

   Additional element kinds may be added in future versions of WebAssembly.


.. index:: ! code section, function, local, type index, function type
   pair: binary format; function
   pair: binary format; local
   pair: section; code
.. _binary-code:
.. _binary-func:
.. _binary-local:
.. _binary-codesec:

Code Section
~~~~~~~~~~~~

The *code section* has the id 10.
It decodes into the list of *code* entries that are pairs of lists of :ref:`locals <syntax-list>` and :ref:`expressions <syntax-expr>`.
They represent the body of the :ref:`functions <syntax-func>` defined by a :ref:`module <syntax-module>`.
The types of the respective functions are encoded separately in the :ref:`function section <binary-funcsec>`.

The encoding of each code entry consists of

* the :math:`{\u32}` *length* of the function code in bytes,
* the actual *function code*, which in turn consists of

  * the declaration of *locals*,
  * the function *body* as an :ref:`expression <binary-expr>`.

Local declarations are compressed into a list whose entries consist of

* a :math:`{\u32}` *count*,
* a :ref:`value type <binary-valtype>`,

denoting *count* locals of the same value type.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Bcodesec} & ::= & {{\mathit{code}}^\ast}{:}{{\Bsection}}_{10}({\Blist}({\Bcode})) & \quad\Rightarrow\quad{} & {{\mathit{code}}^\ast} \\
   & {\Bcode} & ::= & {\mathit{len}}{:}{\Bu32}~~{\mathit{code}}{:}{\Bfunc} & \quad\Rightarrow\quad{} & {\mathit{code}} & \quad \mbox{if}~ {\mathit{len}} = ||{\Bfunc}|| \\
   & {\Bfunc} & ::= & {{{\mathit{loc}}^\ast}^\ast}{:}{\Blist}({\Blocals})~~e{:}{\Bexpr} & \quad\Rightarrow\quad{} & ({\bigcat}\, {{{\mathit{loc}}^\ast}^\ast}, e) & \quad \mbox{if}~ {|{\bigcat}\, {{{\mathit{loc}}^\ast}^\ast}|} < {2^{32}} \\
   & {\Blocals} & ::= & n{:}{\Bu32}~~t{:}{\Bvaltype} & \quad\Rightarrow\quad{} & {(\LOCAL~t)^{n}} \\
   \end{array}

Here, :math:`{\mathit{code}}` ranges over pairs :math:`({{\local}^\ast}, {\expr})`.
Any code for which the length of the resulting sequence is out of bounds of the maximum size of a :ref:`list <syntax-list>` is malformed.

.. note::
   Like with :ref:`sections <binary-section>`, the code :math:`{\mathit{size}}` is not needed for decoding, but can be used to skip functions when navigating through a binary.
   The module is malformed if a size does not match the length of the respective function code.


.. index:: ! data section, data, memory, memory index, expression, byte
   pair: binary format; data
   pair: section; data
   single: memory; data
   single: data; segment
.. _binary-data:
.. _binary-datasec:

Data Section
~~~~~~~~~~~~

The *data section* has the id 11.
It decodes into the list of :ref:`data segments <syntax-data>` defined by a :ref:`module <syntax-module>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Bdatasec} & ::= & {{\data}^\ast}{:}{{\Bsection}}_{11}({\Blist}({\Bdata})) & \quad\Rightarrow\quad{} & {{\data}^\ast} \\
   & {\Bdata} & ::= & 0{:}{\Bu32}~~e{:}{\Bexpr}~~{b^\ast}{:}{\Blist}({\Bbyte}) & \quad\Rightarrow\quad{} & \DATA~{b^\ast}~(\DACTIVE~0~e) \\
   & & | & 1{:}{\Bu32}~~{b^\ast}{:}{\Blist}({\Bbyte}) & \quad\Rightarrow\quad{} & \DATA~{b^\ast}~\DPASSIVE \\
   & & | & 2{:}{\Bu32}~~x{:}{\Bmemidx}~~e{:}{\Bexpr}~~{b^\ast}{:}{\Blist}({\Bbyte}) & \quad\Rightarrow\quad{} & \DATA~{b^\ast}~(\DACTIVE~x~e) \\
   \end{array}

.. note::
   The initial integer can be interpreted as a bitfield.
   Bit 0 indicates a passive segment,
   bit 1 indicates the presence of an explicit memory index for an active segment.


.. index:: ! data count section, data count, data segment
   pair: binary format; data count
   pair: section; data count
.. _binary-datacntsec:
.. _binary-datacnt:

Data Count Section
~~~~~~~~~~~~~~~~~~

The *data count section* has the id 12.
It decodes into an optional :math:`{\u32}` count that represents the number of :ref:`data segments <syntax-data>` in the :ref:`data section <binary-datasec>`.
If this count does not match the length of the data segment list, the module is malformed.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Bdatacntsec} & ::= & {n^?}{:}{{\Bsection}}_{12}({\Bdatacnt}) & \quad\Rightarrow\quad{} & {n^?} \\
   & {\Bdatacnt} & ::= & n{:}{\Bu32} & \quad\Rightarrow\quad{} & n \\
   \end{array}

.. note::
   The data count section is used to simplify single-pass validation. Since the
   data section occurs after the code section, the :math:`\mathsf{memory{.}init}` and
   :math:`\mathsf{data{.}drop}` instructions would not be able to check whether the data
   segment index is valid until the data section is read. The data count section
   occurs before the code section, so a single-pass validator can use this count
   instead of deferring validation.


.. index:: ! tag section, tag, tag type, function type index, exception tag
   pair: binary format; tag
   pair: section; tag
.. _binary-tag:
.. _binary-tagsec:

Tag Section
~~~~~~~~~~~

The *tag section* has the id 13.
It decodes into the list of :ref:`tags <syntax-tag>` defined by a :ref:`module <syntax-module>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Btagsec} & ::= & {{\tag}^\ast}{:}{{\Bsection}}_{13}({\Blist}({\Btag})) & \quad\Rightarrow\quad{} & {{\tag}^\ast} \\
   & {\Btag} & ::= & {\mathit{jt}}{:}{\Btagtype} & \quad\Rightarrow\quad{} & \TAG~{\mathit{jt}} \\
   \end{array}


.. index:: module, section, type definition, function type, function, table, memory, tag, global, element, data, start function, import, export, context, version
   pair: binary format; module
.. _binary-magic:
.. _binary-version:
.. _binary-module:

Modules
~~~~~~~

The encoding of a :ref:`module <syntax-module>` starts with a preamble containing a 4-byte magic number (the string :math:`\text{\backslash0asm}`) and a version field.
The current version of the WebAssembly binary format is 1.

The preamble is followed by a sequence of :ref:`sections <binary-section>`.
:ref:`Custom sections <binary-customsec>` may be inserted at any place in this sequence,
while other sections must occur at most once and in the prescribed order.
All sections can be empty.

The lengths of lists produced by the (possibly empty) :ref:`function <binary-funcsec>` and :ref:`code <binary-codesec>` section must match up.

Similarly, the optional data count must match the length of the :ref:`data segment <binary-datasec>` list.
Furthermore, it must be present if any :ref:`data index <syntax-dataidx>` occurs in the code section.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Bmagic} & ::= & \mathtt{0x00}~~\mathtt{0x61}~~\mathtt{0x73}~~\mathtt{0x6D} \\
   & {\Bversion} & ::= & \mathtt{0x01}~~\mathtt{0x00}~~\mathtt{0x00}~~\mathtt{0x00} \\
   & {\Bmodule} & ::= & \begin{array}[t]{@{}l@{}} {\Bmagic}~~{\Bversion} \\
     {{\Bcustomsec}^\ast}~~{{\type}^\ast}{:}{\Btypesec} \\
     {{\Bcustomsec}^\ast}~~{{\import}^\ast}{:}{\Bimportsec} \\
     {{\Bcustomsec}^\ast}~~{{\typeidx}^\ast}{:}{\Bfuncsec} \\
     {{\Bcustomsec}^\ast}~~{{\table}^\ast}{:}{\Btablesec} \\
     {{\Bcustomsec}^\ast}~~{{\mem}^\ast}{:}{\Bmemsec} \\
     {{\Bcustomsec}^\ast}~~{{\tag}^\ast}{:}{\Btagsec} \\
     {{\Bcustomsec}^\ast}~~{{\global}^\ast}{:}{\Bglobalsec} \\
     {{\Bcustomsec}^\ast}~~{{\export}^\ast}{:}{\Bexportsec} \\
     {{\Bcustomsec}^\ast}~~{{\start}^?}{:}{\Bstartsec} \\
     {{\Bcustomsec}^\ast}~~{{\elem}^\ast}{:}{\Belemsec} \\
     {{\Bcustomsec}^\ast}~~{n^?}{:}{\Bdatacntsec} \\
     {{\Bcustomsec}^\ast}~~{({{\local}^\ast}, {\expr})^\ast}{:}{\Bcodesec} \\
     {{\Bcustomsec}^\ast}~~{{\data}^\ast}{:}{\Bdatasec} \\
     {{\Bcustomsec}^\ast} \end{array} & \quad\Rightarrow\quad{} & & \\
   &&& \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}l@{}l@{}}
   \MODULE~{{\type}^\ast}~{{\import}^\ast}~{{\tag}^\ast}~{{\global}^\ast}~{{\mem}^\ast}~{{\table}^\ast}~{{\func}^\ast}~{{\data}^\ast}~{{\elem}^\ast}~{{\start}^?}~{{\export}^\ast} & & \\
    \multicolumn{3}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ (n = {|{{\data}^\ast}|})^? \\
   {\land}~ ({n^?} \neq \epsilon \lor {\freedataidx}({{\func}^\ast}) = \epsilon) \\
   {\land}~ ({\func} = \FUNC~{\typeidx}~{{\local}^\ast}~{\expr})^\ast \\
   \end{array} \\
   \end{array}
   } \\
   \end{array}
   } \\
   \end{array}

.. note::
   The version of the WebAssembly binary format may increase in the future
   if backward-incompatible changes have to be made to the format.
   However, such changes are expected to occur very infrequently, if ever.
   The binary format is intended to be extensible,
   such that future features can be added without incrementing its version.
