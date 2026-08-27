Modules
-------

For modules, the execution semantics primarily defines :ref:`instantiation <exec-instantiation>`, which :ref:`allocates <alloc>` instances for a module and its contained definitions, initializes :ref:`memories <syntax-mem>` and :ref:`tables <syntax-table>` from contained :ref:`data <syntax-data>` and :ref:`element <syntax-elem>` segments, and invokes the :ref:`start function <syntax-start>` if present. It also includes :ref:`invocation <exec-invocation>` of exported functions.


.. index:: ! allocation, store, address
.. _alloc:

Allocation
~~~~~~~~~~

New instances of
:ref:`tags <syntax-taginst>`,
:ref:`globals <syntax-globalinst>`,
:ref:`memories <syntax-meminst>`,
:ref:`tables <syntax-tableinst>`,
:ref:`functions <syntax-funcinst>`,
:ref:`data segments <syntax-datainst>`, and
:ref:`element segments <syntax-eleminst>`
are *allocated* in a :ref:`store <syntax-store>` :math:`s`, as defined by the following auxiliary functions.


.. index:: tag, tag instance, tag address, tag type
.. _alloc-tag:

:ref:`Tags <syntax-taginst>`
............................


:math:`{\alloctag}(s, {\tagtype})`
..................................


1. Let :math:`{\taginst}` be the :ref:`tag instance <syntax-taginst>` :math:`\{ \HITYPE~{\tagtype} \}`.

#. Let :math:`a` be the length of :math:`s{.}\STAGS`.

#. Append :math:`{\taginst}` to :math:`s{.}\STAGS`.

#. Return :math:`a`.



.. math::
   \begin{array}[t]{@{}lcl@{}l@{}}
   {\alloctag}(s, {\tagtype}) & = & (s \oplus \{ \STAGS~{\taginst} \}, {|s{.}\STAGS|}) &  \\
    \multicolumn{4}{@{}l@{}}{\quad
   \quad \mbox{if}~ {\taginst} = \{ \HITYPE~{\tagtype} \}
   } \\
   \end{array}


.. index:: global, global instance, global address, global type, value type, mutability, value
.. _alloc-global:

:ref:`Globals <syntax-globalinst>`
..................................


:math:`{\allocglobal}(s, {\globaltype}, {\val})`
................................................


1. Let :math:`{\globalinst}` be the :ref:`global instance <syntax-globalinst>` :math:`\{ \GITYPE~{\globaltype},\;\allowbreak \GIVALUE~{\val} \}`.

#. Let :math:`a` be the length of :math:`s{.}\SGLOBALS`.

#. Append :math:`{\globalinst}` to :math:`s{.}\SGLOBALS`.

#. Return :math:`a`.



.. math::
   \begin{array}[t]{@{}lcl@{}l@{}}
   {\allocglobal}(s, {\globaltype}, {\val}) & = & (s \oplus \{ \SGLOBALS~{\globalinst} \}, {|s{.}\SGLOBALS|}) &  \\
    \multicolumn{4}{@{}l@{}}{\quad
   \quad \mbox{if}~ {\globalinst} = \{ \GITYPE~{\globaltype},\;\allowbreak \GIVALUE~{\val} \}
   } \\
   \end{array}


.. index:: memory, memory instance, memory address, memory type, limits, byte
.. _alloc-mem:

:ref:`Memories <syntax-meminst>`
................................


:math:`{\allocmem}(s, {\mathit{at}}~{}[ i \Ldotdot {j^?} ]~\PAGE)`
..................................................................


1. Let :math:`{\meminst}` be the :ref:`memory instance <syntax-meminst>` :math:`\{ \MITYPE~({\mathit{at}}~{}[ i \Ldotdot {j^?} ]~\PAGE),\;\allowbreak \MIBYTES~{\mathtt{0x00}^{i \cdot 64 \, {\mathrm{Ki}}}} \}`.

#. Let :math:`a` be the length of :math:`s{.}\SMEMS`.

#. Append :math:`{\meminst}` to :math:`s{.}\SMEMS`.

#. Return :math:`a`.



.. math::
   \begin{array}[t]{@{}lcl@{}l@{}}
   {\allocmem}(s, {\mathit{at}}~{}[ i \Ldotdot {j^?} ]~\PAGE) & = & (s \oplus \{ \SMEMS~{\meminst} \}, {|s{.}\SMEMS|}) &  \\
    \multicolumn{4}{@{}l@{}}{\quad
   \quad \mbox{if}~ {\meminst} = \{ \MITYPE~({\mathit{at}}~{}[ i \Ldotdot {j^?} ]~\PAGE),\;\allowbreak \MIBYTES~{(\mathtt{0x00})^{i \cdot 64 \, {\mathrm{Ki}}}} \}
   } \\
   \end{array}


.. index:: table, table instance, table address, table type, limits
.. _alloc-table:

:ref:`Tables <syntax-tableinst>`
................................


:math:`{\alloctable}(s, {\mathit{at}}~{}[ i \Ldotdot {j^?} ]~{\mathit{rt}}, {\reff})`
.....................................................................................


1. Let :math:`{\tableinst}` be the :ref:`table instance <syntax-tableinst>` :math:`\{ \TITYPE~({\mathit{at}}~{}[ i \Ldotdot {j^?} ]~{\mathit{rt}}),\;\allowbreak \TIREFS~{{\reff}^{i}} \}`.

#. Let :math:`a` be the length of :math:`s{.}\STABLES`.

#. Append :math:`{\tableinst}` to :math:`s{.}\STABLES`.

#. Return :math:`a`.



.. math::
   \begin{array}[t]{@{}lcl@{}l@{}}
   {\alloctable}(s, {\mathit{at}}~{}[ i \Ldotdot {j^?} ]~{\mathit{rt}}, {\reff}) & = & (s \oplus \{ \STABLES~{\tableinst} \}, {|s{.}\STABLES|}) &  \\
    \multicolumn{4}{@{}l@{}}{\quad
   \quad \mbox{if}~ {\tableinst} = \{ \TITYPE~({\mathit{at}}~{}[ i \Ldotdot {j^?} ]~{\mathit{rt}}),\;\allowbreak \TIREFS~{{\reff}^{i}} \}
   } \\
   \end{array}


.. index:: function, function instance, function address, module instance, function type
.. _alloc-func:

:ref:`Functions <syntax-funcinst>`
..................................


:math:`{\allocfunc}(s, {\deftype}, {{\funccode}}, {\moduleinst})`
.................................................................


1. Let :math:`{\funcinst}` be the :ref:`function instance <syntax-funcinst>` :math:`\{ \FITYPE~{\deftype},\;\allowbreak \FIMODULE~{\moduleinst},\;\allowbreak \FICODE~{{\funccode}} \}`.

#. Let :math:`a` be the length of :math:`s{.}\SFUNCS`.

#. Append :math:`{\funcinst}` to :math:`s{.}\SFUNCS`.

#. Return :math:`a`.



.. math::
   \begin{array}[t]{@{}lcl@{}l@{}}
   {\allocfunc}(s, {\deftype}, {{\funccode}}, {\moduleinst}) & = & (s \oplus \{ \SFUNCS~{\funcinst} \}, {|s{.}\SFUNCS|}) &  \\
    \multicolumn{4}{@{}l@{}}{\quad
   \quad \mbox{if}~ {\funcinst} = \{ \FITYPE~{\deftype},\;\allowbreak \FIMODULE~{\moduleinst},\;\allowbreak \FICODE~{{\funccode}} \}
   } \\
   \end{array}


.. index:: data, data instance, data address
.. _alloc-data:

:ref:`Data segments <syntax-datainst>`
......................................


:math:`{\allocdata}(s, \OKdata, {{\byte}^\ast})`
................................................


1. Let :math:`{\datainst}` be the :ref:`data instance <syntax-datainst>` :math:`\{ \DIBYTES~{{\byte}^\ast} \}`.

#. Let :math:`a` be the length of :math:`s{.}\SDATAS`.

#. Append :math:`{\datainst}` to :math:`s{.}\SDATAS`.

#. Return :math:`a`.



.. math::
   \begin{array}[t]{@{}lcl@{}l@{}}
   {\allocdata}(s, \OKdata, {{\byte}^\ast}) & = & (s \oplus \{ \SDATAS~{\datainst} \}, {|s{.}\SDATAS|}) &  \\
    \multicolumn{4}{@{}l@{}}{\quad
   \quad \mbox{if}~ {\datainst} = \{ \DIBYTES~{{\byte}^\ast} \}
   } \\
   \end{array}


.. index:: element, element instance, element address
.. _alloc-elem:

:ref:`Element segments <syntax-eleminst>`
.........................................


:math:`{\allocelem}(s, {\elemtype}, {{\reff}^\ast})`
....................................................


1. Let :math:`{\eleminst}` be the :ref:`element instance <syntax-eleminst>` :math:`\{ \EITYPE~{\elemtype},\;\allowbreak \EIREFS~{{\reff}^\ast} \}`.

#. Let :math:`a` be the length of :math:`s{.}\SELEMS`.

#. Append :math:`{\eleminst}` to :math:`s{.}\SELEMS`.

#. Return :math:`a`.



.. math::
   \begin{array}[t]{@{}lcl@{}l@{}}
   {\allocelem}(s, {\elemtype}, {{\reff}^\ast}) & = & (s \oplus \{ \SELEMS~{\eleminst} \}, {|s{.}\SELEMS|}) &  \\
    \multicolumn{4}{@{}l@{}}{\quad
   \quad \mbox{if}~ {\eleminst} = \{ \EITYPE~{\elemtype},\;\allowbreak \EIREFS~{{\reff}^\ast} \}
   } \\
   \end{array}


.. index:: memory, memory instance, memory address, grow, limits
.. _grow-mem:

Growing :ref:`memories <syntax-meminst>`
........................................


:math:`{\growmem}({\meminst}, n)`
.................................


1. Let :math:`\{ \MITYPE~({\mathit{at}}~{}[ i \Ldotdot {j^?} ]~\PAGE),\;\allowbreak \MIBYTES~{b^\ast} \}` be the destructuring of :math:`{\meminst}`.

#. Let :math:`{i'}` be :math:`{|{b^\ast}|} / (64 \, {\mathrm{Ki}}) + n`.

#. If not :math:`{({i'} \leq j)^?}`, then:

   a. Fail.

#. If :math:`{i'} \leq {2^{{|{\mathit{at}}|} - 16}}`, then:

   a. Let :math:`{\meminst'}` be the :ref:`memory instance <syntax-meminst>` :math:`\{ \MITYPE~({\mathit{at}}~{}[ {i'} \Ldotdot {j^?} ]~\PAGE),\;\allowbreak \MIBYTES~{b^\ast}~{\mathtt{0x00}^{n \cdot 64 \, {\mathrm{Ki}}}} \}`.

   #. Return :math:`{\meminst'}`.

#. Fail.



.. math::
   \begin{array}[t]{@{}lcl@{}l@{}}
   {\growmem}({\meminst}, n) & = & {\meminst'} & \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ {\meminst} = \{ \MITYPE~({\mathit{at}}~{}[ i \Ldotdot {j^?} ]~\PAGE),\; \MIBYTES~{b^\ast} \} \\
   {\land}~ {\meminst'} = \{ \MITYPE~({\mathit{at}}~{}[ {i'} \Ldotdot {j^?} ]~\PAGE),\; \MIBYTES~{b^\ast}~{(\mathtt{0x00})^{n \cdot 64 \, {\mathrm{Ki}}}} \} \\
   {\land}~ {i'} = {|{b^\ast}|} / (64 \, {\mathrm{Ki}}) + n \\
   {\land}~ ({i'} \leq j)^? \\
   {\land}~ {i'} \leq {2^{{|{\mathit{at}}|} - 16}} \\
   \end{array} \\
   \end{array}


.. index:: table, table instance, table address, grow, limits
.. _grow-table:

Growing :ref:`tables <syntax-tableinst>`
........................................


:math:`{\growtable}({\tableinst}, n, r)`
........................................


1. Let :math:`\{ \TITYPE~({\mathit{at}}~{}[ i \Ldotdot {j^?} ]~{\mathit{rt}}),\;\allowbreak \TIREFS~{{r'}^\ast} \}` be the destructuring of :math:`{\tableinst}`.

#. Let :math:`{i'}` be :math:`{|{{r'}^\ast}|} + n`.

#. If not :math:`{({i'} \leq j)^?}`, then:

   a. Fail.

#. If :math:`{i'} \leq {2^{{|{\mathit{at}}|}}} - 1`, then:

   a. Let :math:`{\tableinst'}` be the :ref:`table instance <syntax-tableinst>` :math:`\{ \TITYPE~({\mathit{at}}~{}[ {i'} \Ldotdot {j^?} ]~{\mathit{rt}}),\;\allowbreak \TIREFS~{{r'}^\ast}~{r^{n}} \}`.

   #. Return :math:`{\tableinst'}`.

#. Fail.



.. math::
   \begin{array}[t]{@{}lcl@{}l@{}}
   {\growtable}({\tableinst}, n, r) & = & {\tableinst'} & \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ {\tableinst} = \{ \TITYPE~({\mathit{at}}~{}[ i \Ldotdot {j^?} ]~{\mathit{rt}}),\; \TIREFS~{{r'}^\ast} \} \\
   {\land}~ {\tableinst'} = \{ \TITYPE~({\mathit{at}}~{}[ {i'} \Ldotdot {j^?} ]~{\mathit{rt}}),\; \TIREFS~{{r'}^\ast}~{r^{n}} \} \\
   {\land}~ {i'} = {|{{r'}^\ast}|} + n \\
   {\land}~ ({i'} \leq j)^? \\
   {\land}~ {i'} \leq {2^{{|{\mathit{at}}|}}} - 1 \\
   \end{array} \\
   \end{array}


.. index:: module, module instance, tag instance, global instance, memory instance, table instance, function instance, data instance, element instance, export instance, tag address, global address, memory address, table address, function address, data address, element address, tag index, global index, memory index, table index, function index, type, tag, global, memory, table, function, data segment, element segment, import, export, external address, external type, matching
.. _alloc-module:

:ref:`Modules <syntax-moduleinst>`
..................................


:math:`{\allocmodule}(s, {\module}, {{\externaddr}^\ast}, {{\val}_{\mathsf{g}}^\ast}, {{\reff}_{\mathsf{t}}^\ast}, {{{\reff}_{\mathsf{e}}^\ast}^\ast})`
.......................................................................................................................................................


1. Let :math:`(\MODULE~{{\type}^\ast}~{{\import}^\ast}~{{\tag}^\ast}~{{\global}^\ast}~{{\mem}^\ast}~{{\table}^\ast}~{{\func}^\ast}~{{\data}^\ast}~{{\elem}^\ast}~{{\start}^?}~{{\export}^\ast})` be the destructuring of :math:`{\module}`.

#. Let :math:`{{\mathit{aa}}_{\mathsf{i}}^\ast}` be :math:`{\tagsxa}({{\externaddr}^\ast})`.

#. Let :math:`{{\mathit{ga}}_{\mathsf{i}}^\ast}` be :math:`{\globalsxa}({{\externaddr}^\ast})`.

#. Let :math:`{{\mathit{fa}}_{\mathsf{i}}^\ast}` be :math:`{\funcsxa}({{\externaddr}^\ast})`.

#. Let :math:`{{\mathit{ma}}_{\mathsf{i}}^\ast}` be :math:`{\memsxa}({{\externaddr}^\ast})`.

#. Let :math:`{{\mathit{ta}}_{\mathsf{i}}^\ast}` be :math:`{\tablesxa}({{\externaddr}^\ast})`.

#. Let :math:`{{\mathit{fa}}^\ast}` be :math:`{|s{.}\SFUNCS|} + i_{\mathsf{f}}` for all :math:`i_{\mathsf{f}}` from :math:`0` to :math:`{|{{\func}^\ast}|} - 1`.

#. Let :math:`{{\tagtype}^\ast}` be the tag type sequence :math:`\epsilon`.

#. For each :math:`{\tag}` in :math:`{{\tag}^\ast}`, do:

   a. Let :math:`(\TAG~{\tagtype})` be the destructuring of :math:`{\tag}`.

   #. Append :math:`{\tagtype}` to :math:`{{\tagtype}^\ast}`.

#. Let :math:`{{{\byte}^\ast}^\ast}` be the byte sequence sequence :math:`\epsilon`.

#. For each :math:`{\data}` in :math:`{{\data}^\ast}`, do:

   a. Let :math:`(\DATA~{{\byte}^\ast}~{\datamode})` be the destructuring of :math:`{\data}`.

   #. Append :math:`{{\byte}^\ast}` to :math:`{{{\byte}^\ast}^\ast}`.

#. Let :math:`{{\globaltype}^\ast}` be the global type sequence :math:`\epsilon`.

#. For each :math:`{\global}` in :math:`{{\global}^\ast}`, do:

   a. Let :math:`(\GLOBAL~{\globaltype}~{\expr}_{\mathsf{g}})` be the destructuring of :math:`{\global}`.

   #. Append :math:`{\globaltype}` to :math:`{{\globaltype}^\ast}`.

#. Let :math:`{{\tabletype}^\ast}` be the table type sequence :math:`\epsilon`.

#. For each :math:`{\table}` in :math:`{{\table}^\ast}`, do:

   a. Let :math:`(\TABLE~{\tabletype}~{\expr}_{\mathsf{t}})` be the destructuring of :math:`{\table}`.

   #. Append :math:`{\tabletype}` to :math:`{{\tabletype}^\ast}`.

#. Let :math:`{{\memtype}^\ast}` be the memory type sequence :math:`\epsilon`.

#. For each :math:`{\mem}` in :math:`{{\mem}^\ast}`, do:

   a. Let :math:`(\MEMORY~{\memtype})` be the destructuring of :math:`{\mem}`.

   #. Append :math:`{\memtype}` to :math:`{{\memtype}^\ast}`.

#. Let :math:`{{\mathit{dt}}^\ast}` be :math:`{{{\alloctype}^\ast}}{({{\type}^\ast})}`.

#. Let :math:`{{\elemtype}^\ast}` be the reference type sequence :math:`\epsilon`.

#. For each :math:`{\elem}` in :math:`{{\elem}^\ast}`, do:

   a. Let :math:`(\ELEM~{\elemtype}~{{\expr}_{\mathsf{e}}^\ast}~{\elemmode})` be the destructuring of :math:`{\elem}`.

   #. Append :math:`{\elemtype}` to :math:`{{\elemtype}^\ast}`.

#. Let :math:`{{\expr}_{\mathsf{f}}^\ast}` be the expression sequence :math:`\epsilon`.

#. Let :math:`{{{\local}^\ast}^\ast}` be the local sequence sequence :math:`\epsilon`.

#. Let :math:`{x^\ast}` be the type index sequence :math:`\epsilon`.

#. For each :math:`{\func}` in :math:`{{\func}^\ast}`, do:

   a. Let :math:`(\FUNC~x~{{\local}^\ast}~{\expr}_{\mathsf{f}})` be the destructuring of :math:`{\func}`.

   #. Append :math:`{\expr}_{\mathsf{f}}` to :math:`{{\expr}_{\mathsf{f}}^\ast}`.

   #. Append :math:`{{\local}^\ast}` to :math:`{{{\local}^\ast}^\ast}`.

   #. Append :math:`x` to :math:`{x^\ast}`.

#. Let :math:`{{\mathit{aa}}^\ast}` be :math:`\epsilon`.

#. For each :math:`{\tagtype}` in :math:`{{\tagtype}^\ast}`, do:

   a. Let :math:`{\mathit{aa}}` be the :ref:`tag address <syntax-tagaddr>` :math:`{\alloctag}(s, {{\tagtype}}{{}[ {\assignsubst}\, {{\mathit{dt}}^\ast} ]})`.

   #. Append :math:`{\mathit{aa}}` to :math:`{{\mathit{aa}}^\ast}`.

#. Let :math:`{{\mathit{ga}}^\ast}` be :math:`\epsilon`.

#. For each :math:`{\globaltype}` in :math:`{{\globaltype}^\ast}` and :math:`{\val}_{\mathsf{g}}` in :math:`{{\val}_{\mathsf{g}}^\ast}`, do:

   a. Let :math:`{\mathit{ga}}` be the :ref:`global address <syntax-globaladdr>` :math:`{\allocglobal}(s, {{\globaltype}}{{}[ {\assignsubst}\, {{\mathit{dt}}^\ast} ]}, {\val}_{\mathsf{g}})`.

   #. Append :math:`{\mathit{ga}}` to :math:`{{\mathit{ga}}^\ast}`.

#. Let :math:`{{\mathit{ma}}^\ast}` be :math:`\epsilon`.

#. For each :math:`{\memtype}` in :math:`{{\memtype}^\ast}`, do:

   a. Let :math:`{\mathit{ma}}` be the :ref:`memory address <syntax-memaddr>` :math:`{\allocmem}(s, {{\memtype}}{{}[ {\assignsubst}\, {{\mathit{dt}}^\ast} ]})`.

   #. Append :math:`{\mathit{ma}}` to :math:`{{\mathit{ma}}^\ast}`.

#. Let :math:`{{\mathit{ta}}^\ast}` be :math:`\epsilon`.

#. For each :math:`{\tabletype}` in :math:`{{\tabletype}^\ast}` and :math:`{\reff}_{\mathsf{t}}` in :math:`{{\reff}_{\mathsf{t}}^\ast}`, do:

   a. Let :math:`{\mathit{ta}}` be the :ref:`table address <syntax-tableaddr>` :math:`{\alloctable}(s, {{\tabletype}}{{}[ {\assignsubst}\, {{\mathit{dt}}^\ast} ]}, {\reff}_{\mathsf{t}})`.

   #. Append :math:`{\mathit{ta}}` to :math:`{{\mathit{ta}}^\ast}`.

#. Let :math:`{{\mathit{xi}}^\ast}` be :math:`\epsilon`.

#. For each :math:`{\export}` in :math:`{{\export}^\ast}`, do:

   a. Let :math:`{\mathit{xi}}` be the :ref:`export instance <syntax-exportinst>` :math:`{\allocexport}({\moduleinst}, {\export})`.

   #. Append :math:`{\mathit{xi}}` to :math:`{{\mathit{xi}}^\ast}`.

#. Let :math:`{{\mathit{da}}^\ast}` be :math:`\epsilon`.

#. For each :math:`{{\byte}^\ast}` in :math:`{{{\byte}^\ast}^\ast}`, do:

   a. Let :math:`{\mathit{da}}` be the :ref:`data address <syntax-dataaddr>` :math:`{\allocdata}(s, \OKdata, {{\byte}^\ast})`.

   #. Append :math:`{\mathit{da}}` to :math:`{{\mathit{da}}^\ast}`.

#. Let :math:`{{\mathit{ea}}^\ast}` be :math:`\epsilon`.

#. For each :math:`{\elemtype}` in :math:`{{\elemtype}^\ast}` and :math:`{{\reff}_{\mathsf{e}}^\ast}` in :math:`{{{\reff}_{\mathsf{e}}^\ast}^\ast}`, do:

   a. Let :math:`{\mathit{ea}}` be the :ref:`elem address <syntax-elemaddr>` :math:`{\allocelem}(s, {{\elemtype}}{{}[ {\assignsubst}\, {{\mathit{dt}}^\ast} ]}, {{\reff}_{\mathsf{e}}^\ast})`.

   #. Append :math:`{\mathit{ea}}` to :math:`{{\mathit{ea}}^\ast}`.

#. Let :math:`{\moduleinst}` be the :ref:`module instance <syntax-moduleinst>` :math:`\{ \MITYPES~{{\mathit{dt}}^\ast},\;\allowbreak \MITAGS~{{\mathit{aa}}_{\mathsf{i}}^\ast}~{{\mathit{aa}}^\ast},\;\allowbreak \MIGLOBALS~{{\mathit{ga}}_{\mathsf{i}}^\ast}~{{\mathit{ga}}^\ast},\;\allowbreak \MIMEMS~{{\mathit{ma}}_{\mathsf{i}}^\ast}~{{\mathit{ma}}^\ast},\;\allowbreak \MITABLES~{{\mathit{ta}}_{\mathsf{i}}^\ast}~{{\mathit{ta}}^\ast},\;\allowbreak \MIFUNCS~{{\mathit{fa}}_{\mathsf{i}}^\ast}~{{\mathit{fa}}^\ast},\;\allowbreak \MIDATAS~{{\mathit{da}}^\ast},\;\allowbreak \MIELEMS~{{\mathit{ea}}^\ast},\;\allowbreak \MIEXPORTS~{{\mathit{xi}}^\ast} \}`.

#. Let :math:`{{\funcaddr}_0^\ast}` be :math:`\epsilon`.

#. For each :math:`{\expr}_{\mathsf{f}}` in :math:`{{\expr}_{\mathsf{f}}^\ast}` and :math:`{{\local}^\ast}` in :math:`{{{\local}^\ast}^\ast}` and :math:`x` in :math:`{x^\ast}`, do:

   a. Let :math:`{\funcaddr}_0` be the :ref:`function address <syntax-funcaddr>` :math:`{\allocfunc}(s, {{\mathit{dt}}^\ast}{}[x], \FUNC~x~{{\local}^\ast}~{\expr}_{\mathsf{f}}, {\moduleinst})`.

   #. Append :math:`{\funcaddr}_0` to :math:`{{\funcaddr}_0^\ast}`.

#. Assert: Due to validation, :math:`{{\funcaddr}_0^\ast} = {{\mathit{fa}}^\ast}`.

#. Return :math:`{\moduleinst}`.



.. math::
   \begin{array}[t]{@{}lcl@{}l@{}}
   {\allocmodule}(s, {\module}, {{\externaddr}^\ast}, {{\val}_{\mathsf{g}}^\ast}, {{\reff}_{\mathsf{t}}^\ast}, {({{\reff}_{\mathsf{e}}^\ast})^\ast}) & = & (s_7, {\moduleinst}) &  \\
    \multicolumn{4}{@{}l@{}}{\quad
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ {\module} = \MODULE~{{\type}^\ast}~{{\import}^\ast}~{{\tag}^\ast}~{{\global}^\ast}~{{\mem}^\ast}~{{\table}^\ast}~{{\func}^\ast}~{{\data}^\ast}~{{\elem}^\ast}~{{\start}^?}~{{\export}^\ast} \\
   {\land}~ {{\tag}^\ast} = {(\TAG~{\tagtype})^\ast} \\
   {\land}~ {{\global}^\ast} = {(\GLOBAL~{\globaltype}~{\expr}_{\mathsf{g}})^\ast} \\
   {\land}~ {{\mem}^\ast} = {(\MEMORY~{\memtype})^\ast} \\
   {\land}~ {{\table}^\ast} = {(\TABLE~{\tabletype}~{\expr}_{\mathsf{t}})^\ast} \\
   {\land}~ {{\func}^\ast} = {(\FUNC~x~{{\local}^\ast}~{\expr}_{\mathsf{f}})^\ast} \\
   {\land}~ {{\data}^\ast} = {(\DATA~{{\byte}^\ast}~{\datamode})^\ast} \\
   {\land}~ {{\elem}^\ast} = {(\ELEM~{\elemtype}~{{\expr}_{\mathsf{e}}^\ast}~{\elemmode})^\ast} \\
   {\land}~ {{\mathit{aa}}_{\mathsf{i}}^\ast} = {\tagsxa}({{\externaddr}^\ast}) \\
   {\land}~ {{\mathit{ga}}_{\mathsf{i}}^\ast} = {\globalsxa}({{\externaddr}^\ast}) \\
   {\land}~ {{\mathit{ma}}_{\mathsf{i}}^\ast} = {\memsxa}({{\externaddr}^\ast}) \\
   {\land}~ {{\mathit{ta}}_{\mathsf{i}}^\ast} = {\tablesxa}({{\externaddr}^\ast}) \\
   {\land}~ {{\mathit{fa}}_{\mathsf{i}}^\ast} = {\funcsxa}({{\externaddr}^\ast}) \\
   {\land}~ {{\mathit{dt}}^\ast} = {{{\alloctype}^\ast}}{({{\type}^\ast})} \\
   {\land}~ {{\mathit{fa}}^\ast} = {({|s{.}\SFUNCS|} + i_{\mathsf{f}})^{i_{\mathsf{f}}<{|{{\func}^\ast}|}}} \\
   {\land}~ (s_1, {{\mathit{aa}}^\ast}) = {{{\alloctag}^\ast}}{(s, {{{\tagtype}}{{}[ {\assignsubst}\, {{\mathit{dt}}^\ast} ]}^\ast})} \\
   {\land}~ (s_2, {{\mathit{ga}}^\ast}) = {{{\allocglobal}^\ast}}{(s_1, {{{\globaltype}}{{}[ {\assignsubst}\, {{\mathit{dt}}^\ast} ]}^\ast}, {{\val}_{\mathsf{g}}^\ast})} \\
   {\land}~ (s_3, {{\mathit{ma}}^\ast}) = {{{\allocmem}^\ast}}{(s_2, {{{\memtype}}{{}[ {\assignsubst}\, {{\mathit{dt}}^\ast} ]}^\ast})} \\
   {\land}~ (s_4, {{\mathit{ta}}^\ast}) = {{{\alloctable}^\ast}}{(s_3, {{{\tabletype}}{{}[ {\assignsubst}\, {{\mathit{dt}}^\ast} ]}^\ast}, {{\reff}_{\mathsf{t}}^\ast})} \\
   {\land}~ (s_5, {{\mathit{da}}^\ast}) = {{{\allocdata}^\ast}}{(s_4, {\OKdata^{{|{{\data}^\ast}|}}}, {({{\byte}^\ast})^\ast})} \\
   {\land}~ (s_6, {{\mathit{ea}}^\ast}) = {{{\allocelem}^\ast}}{(s_5, {{{\elemtype}}{{}[ {\assignsubst}\, {{\mathit{dt}}^\ast} ]}^\ast}, {({{\reff}_{\mathsf{e}}^\ast})^\ast})} \\
   {\land}~ (s_7, {{\mathit{fa}}^\ast}) = {{{\allocfunc}^\ast}}{(s_6, {{{\mathit{dt}}^\ast}{}[x]^\ast}, {(\FUNC~x~{{\local}^\ast}~{\expr}_{\mathsf{f}})^\ast}, {{\moduleinst}^{{|{{\func}^\ast}|}}})} \\
   {\land}~ {{\mathit{xi}}^\ast} = {{{\allocexport}^\ast}}{(\{ \MITAGS~{{\mathit{aa}}_{\mathsf{i}}^\ast}~{{\mathit{aa}}^\ast},\; \MIGLOBALS~{{\mathit{ga}}_{\mathsf{i}}^\ast}~{{\mathit{ga}}^\ast},\; \MIMEMS~{{\mathit{ma}}_{\mathsf{i}}^\ast}~{{\mathit{ma}}^\ast},\; \MITABLES~{{\mathit{ta}}_{\mathsf{i}}^\ast}~{{\mathit{ta}}^\ast},\; \MIFUNCS~{{\mathit{fa}}_{\mathsf{i}}^\ast}~{{\mathit{fa}}^\ast} \}, {{\export}^\ast})} \\
   {\land}~ {\moduleinst} = \{ \begin{array}[t]{@{}l@{}}
   \MITYPES~{{\mathit{dt}}^\ast},\; \\
     \MITAGS~{{\mathit{aa}}_{\mathsf{i}}^\ast}~{{\mathit{aa}}^\ast},\; \MIGLOBALS~{{\mathit{ga}}_{\mathsf{i}}^\ast}~{{\mathit{ga}}^\ast},\; \\
     \MIMEMS~{{\mathit{ma}}_{\mathsf{i}}^\ast}~{{\mathit{ma}}^\ast},\; \\
     \MITABLES~{{\mathit{ta}}_{\mathsf{i}}^\ast}~{{\mathit{ta}}^\ast},\; \MIFUNCS~{{\mathit{fa}}_{\mathsf{i}}^\ast}~{{\mathit{fa}}^\ast},\; \MIDATAS~{{\mathit{da}}^\ast},\; \\
     \MIELEMS~{{\mathit{ea}}^\ast},\; \MIEXPORTS~{{\mathit{xi}}^\ast} \}\end{array} \\
   \end{array}
   } \\
   \end{array}

Here, the notation :math:`\F{allocx}^\ast` is shorthand for multiple :ref:`allocations <alloc>` of object kind :math:`X`, defined as follows:

.. math::
   \begin{array}[t]{@{}lcl@{}l@{}}
   {{{\mathrm{allocX}}^\ast}}{(s, \epsilon, \epsilon)} & = & (s, \epsilon) \\
   {{{\mathrm{allocX}}^\ast}}{(s, X~{{X'}^\ast}, Y~{{Y'}^\ast})} & = & (s_2, a~{{a'}^\ast}) & \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ (s_1, a) = {\mathrm{allocX}}(X, Y, s, X, Y) \\
   {\land}~ (s_2, {{a'}^\ast}) = {{{\mathrm{allocX}}^\ast}}{(s_1, {{X'}^\ast}, {{Y'}^\ast})} \\
   \end{array} \\
   \end{array}


For types, however, allocation is defined in terms of :ref:`rolling <aux-roll-rectype>` and :ref:`substitution <notation-subst>` of all preceding types to produce a list of :ref:`closed <type-closed>` :ref:`defined types <syntax-deftype>`:

.. _alloc-type:


:math:`{{{\alloctype}^\ast}}{({{\type''}^\ast})}`
.................................................


1. If :math:`{{\type''}^\ast} = \epsilon`, then:

   a. Return :math:`\epsilon`.

#. Let :math:`{{\type'}^\ast}~{\type}` be :math:`{{\type''}^\ast}`.

#. Let :math:`(\TYPE~{\rectype})` be the destructuring of :math:`{\type}`.

#. Let :math:`{{\deftype'}^\ast}` be :math:`{{{\alloctype}^\ast}}{({{\type'}^\ast})}`.

#. Let :math:`x` be the length of :math:`{{\deftype'}^\ast}`.

#. Let :math:`{{\deftype}^\ast}` be :math:`{{{{{\rolldt}}_{x}^\ast}}{({\rectype})}}{{}[ {\assignsubst}\, {{\deftype'}^\ast} ]}`.

#. Return :math:`{{\deftype'}^\ast}~{{\deftype}^\ast}`.



.. math::
   \begin{array}[t]{@{}lcl@{}l@{}}
   {{{\alloctype}^\ast}}{(\epsilon)} & = & \epsilon \\
   {{{\alloctype}^\ast}}{({{\type'}^\ast}~{\type})} & = & {{\deftype'}^\ast}~{{\deftype}^\ast} & \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ {{\deftype'}^\ast} = {{{\alloctype}^\ast}}{({{\type'}^\ast})} \\
   {\land}~ {\type} = \TYPE~{\rectype} \\
   {\land}~ {{\deftype}^\ast} = {{{{{\rolldt}}_{x}^\ast}}{({\rectype})}}{{}[ {\assignsubst}\, {{\deftype'}^\ast} ]} \\
   {\land}~ x = {|{{\deftype'}^\ast}|} \\
   \end{array} \\
   \end{array}

Finally, export instances are produced with the help of the following definition:

.. _alloc-export:





:math:`{\allocexport}({\moduleinst}, \EXPORT~{\name}~{\externidx})`
...................................................................


1. If :math:`{\externidx}` is some :math:`\XXTAG~{\tagidx}`, then:

   a. Let :math:`(\XXTAG~x)` be the destructuring of :math:`{\externidx}`.

   #. Return :math:`\{ \XINAME~{\name},\;\allowbreak \XIADDR~(\XATAG~{\moduleinst}{.}\MITAGS{}[x]) \}`.

#. If :math:`{\externidx}` is some :math:`\XXGLOBAL~{\globalidx}`, then:

   a. Let :math:`(\XXGLOBAL~x)` be the destructuring of :math:`{\externidx}`.

   #. Return :math:`\{ \XINAME~{\name},\;\allowbreak \XIADDR~(\XAGLOBAL~{\moduleinst}{.}\MIGLOBALS{}[x]) \}`.

#. If :math:`{\externidx}` is some :math:`\XXMEM~{\memidx}`, then:

   a. Let :math:`(\XXMEM~x)` be the destructuring of :math:`{\externidx}`.

   #. Return :math:`\{ \XINAME~{\name},\;\allowbreak \XIADDR~(\XAMEM~{\moduleinst}{.}\MIMEMS{}[x]) \}`.

#. If :math:`{\externidx}` is some :math:`\XXTABLE~{\tableidx}`, then:

   a. Let :math:`(\XXTABLE~x)` be the destructuring of :math:`{\externidx}`.

   #. Return :math:`\{ \XINAME~{\name},\;\allowbreak \XIADDR~(\XATABLE~{\moduleinst}{.}\MITABLES{}[x]) \}`.

#. Assert: Due to validation, :math:`{\externidx}` is some :math:`\XXFUNC~{\funcidx}`.

#. Let :math:`(\XXFUNC~x)` be the destructuring of :math:`{\externidx}`.

#. Return :math:`\{ \XINAME~{\name},\;\allowbreak \XIADDR~(\XAFUNC~{\moduleinst}{.}\MIFUNCS{}[x]) \}`.



.. math::
   \begin{array}[t]{@{}lcl@{}l@{}}
   {\allocexport}({\moduleinst}, \EXPORT~{\name}~(\XXTAG~x)) & = & \{ \XINAME~{\name},\;\allowbreak \XIADDR~(\XATAG~{\moduleinst}{.}\MITAGS{}[x]) \} \\
   {\allocexport}({\moduleinst}, \EXPORT~{\name}~(\XXGLOBAL~x)) & = & \{ \XINAME~{\name},\;\allowbreak \XIADDR~(\XAGLOBAL~{\moduleinst}{.}\MIGLOBALS{}[x]) \} \\
   {\allocexport}({\moduleinst}, \EXPORT~{\name}~(\XXMEM~x)) & = & \{ \XINAME~{\name},\;\allowbreak \XIADDR~(\XAMEM~{\moduleinst}{.}\MIMEMS{}[x]) \} \\
   {\allocexport}({\moduleinst}, \EXPORT~{\name}~(\XXTABLE~x)) & = & \{ \XINAME~{\name},\;\allowbreak \XIADDR~(\XATABLE~{\moduleinst}{.}\MITABLES{}[x]) \} \\
   {\allocexport}({\moduleinst}, \EXPORT~{\name}~(\XXFUNC~x)) & = & \{ \XINAME~{\name},\;\allowbreak \XIADDR~(\XAFUNC~{\moduleinst}{.}\MIFUNCS{}[x]) \} \\
   \end{array}

.. note::
   The definition of module allocation is mutually recursive with the allocation of its associated functions, because the resulting module instance is passed to the allocators as an argument, in order to form the necessary closures.
   In an implementation, this recursion is easily unraveled by mutating one or the other in a secondary step.



.. index:: ! instantiation, module, instance, store, trap, exception
.. _exec-module:
.. _exec-instantiation:

Instantiation
~~~~~~~~~~~~~

Given a :ref:`store <syntax-store>` :math:`s`, a :math:`{\module}` is instantiated with a list of :ref:`external addresses <syntax-externaddr>` :math:`{{\externaddr}^\ast}` supplying the required imports as follows.

Instantiation checks that the module is :ref:`valid <valid>` and the provided imports :ref:`match <match-externtype>` the declared types,
and may *fail* with an error otherwise.
Instantiation can also result in an :ref:`exception <exception>` or :ref:`trap <trap>` when initializing a :ref:`table <syntax-table>` or :ref:`memory <syntax-mem>` from an :ref:`active segment <syntax-data>` or when executing the :ref:`start <syntax-start>` function.
It is up to the :ref:`embedder <embedder>` to define how such conditions are reported.

.. _exec-initvals:


:math:`{\instantiate}(s, {\module}, {{\externaddr}^\ast})`
..........................................................


1. If :math:`{\module}` is not :ref:`valid <valid-module>`, then:

   a. Fail.

#. Let :math:`{{\mathit{xt}}_{\mathsf{i}}^\ast}~\toM~{{\mathit{xt}}_{\mathsf{e}}^\ast}` be the destructuring of the type of :math:`{\module}`.

#. Let :math:`(\MODULE~{{\type}^\ast}~{{\import}^\ast}~{{\tag}^\ast}~{{\global}^\ast}~{{\mem}^\ast}~{{\table}^\ast}~{{\func}^\ast}~{{\data}^\ast}~{{\elem}^\ast}~{{\start}^?}~{{\export}^\ast})` be the destructuring of :math:`{\module}`.

#. If :math:`{|{{\externaddr}^\ast}|} \neq {|{{\mathit{xt}}_{\mathsf{i}}^\ast}|}`, then:

   a. Fail.

#. For all :math:`{\externaddr}` in :math:`{{\externaddr}^\ast}`, and corresponding :math:`{\mathit{xt}}_{\mathsf{i}}` in :math:`{{\mathit{xt}}_{\mathsf{i}}^\ast}`:

   a. If :math:`{\externaddr}` is not :ref:`valid <valid-val>` with type :math:`{\mathit{xt}}_{\mathsf{i}}`, then:

      1) Fail.

#. Let :math:`{{\instr}_{\mathsf{d}}^\ast}` be the :ref:`concatenation <notation-concat>` of :math:`{{{\rundata}}_{i_{\mathsf{d}}}({{\data}^\ast}{}[i_{\mathsf{d}}])^{i_{\mathsf{d}}<{|{{\data}^\ast}|}}}`.

#. Let :math:`{{\instr}_{\mathsf{e}}^\ast}` be the :ref:`concatenation <notation-concat>` of :math:`{{{\runelem}}_{i_{\mathsf{e}}}({{\elem}^\ast}{}[i_{\mathsf{e}}])^{i_{\mathsf{e}}<{|{{\elem}^\ast}|}}}`.

#. Let :math:`{\moduleinst}_0` be the :ref:`module instance <syntax-moduleinst>` :math:`\{ \MITYPES~{{{\alloctype}^\ast}}{({{\type}^\ast})},\;\allowbreak \MIGLOBALS~{\globalsxa}({{\externaddr}^\ast}),\;\allowbreak \MIFUNCS~{\funcsxa}({{\externaddr}^\ast})~{({|s{.}\SFUNCS|} + i_{\mathsf{f}})^{i_{\mathsf{f}}<{|{{\func}^\ast}|}}} \}`.

#. Let :math:`{{\expr}_{\mathsf{t}}^\ast}` be the expression sequence :math:`\epsilon`.

#. For each :math:`{\table}` in :math:`{{\table}^\ast}`, do:

   a. Let :math:`(\TABLE~{\tabletype}~{\expr}_{\mathsf{t}})` be the destructuring of :math:`{\table}`.

   #. Append :math:`{\expr}_{\mathsf{t}}` to :math:`{{\expr}_{\mathsf{t}}^\ast}`.

#. Let :math:`{{\expr}_{\mathsf{g}}^\ast}` be the expression sequence :math:`\epsilon`.

#. Let :math:`{{\globaltype}^\ast}` be the global type sequence :math:`\epsilon`.

#. For each :math:`{\global}` in :math:`{{\global}^\ast}`, do:

   a. Let :math:`(\GLOBAL~{\globaltype}~{\expr}_{\mathsf{g}})` be the destructuring of :math:`{\global}`.

   #. Append :math:`{\expr}_{\mathsf{g}}` to :math:`{{\expr}_{\mathsf{g}}^\ast}`.

   #. Append :math:`{\globaltype}` to :math:`{{\globaltype}^\ast}`.

#. Let :math:`{{{\expr}_{\mathsf{e}}^\ast}^\ast}` be the expression sequence sequence :math:`\epsilon`.

#. For each :math:`{\elem}` in :math:`{{\elem}^\ast}`, do:

   a. Let :math:`(\ELEM~{\reftype}~{{\expr}_{\mathsf{e}}^\ast}~{\elemmode})` be the destructuring of :math:`{\elem}`.

   #. Append :math:`{{\expr}_{\mathsf{e}}^\ast}` to :math:`{{{\expr}_{\mathsf{e}}^\ast}^\ast}`.

#. Let :math:`z` be the :ref:`state <syntax-state>` :math:`(s, \{ \AMODULE~{\moduleinst}_0 \})`.

#. Let :math:`F` be the :math:`\mathsf{frame}` :math:`z{.}\ZFRAME`.

#. Push the :math:`\mathsf{frame}` :math:`F`.

#. Let :math:`{{\val}_{\mathsf{g}}^\ast}` be :math:`{{{\mathrm{evalglobal}}^\ast}}{(z, {{\globaltype}^\ast}, {{\expr}_{\mathsf{g}}^\ast})}`.

#. Let :math:`{{\reff}_{\mathsf{t}}^\ast}` be :math:`{{{\mathrm{evalexpr}}^\ast}}{(z, {{\expr}_{\mathsf{t}}^\ast})}`.

#. Let :math:`{{{\reff}_{\mathsf{e}}^\ast}^\ast}` be :math:`{{{{\mathrm{evalexpr}}^\ast}^\ast}}{(z, {{{\expr}_{\mathsf{e}}^\ast}^\ast})}`.

#. Pop the :math:`\mathsf{frame}` from the stack.

#. Let :math:`(s, f)` be the destructuring of :math:`z`.

#. Let :math:`{\moduleinst}` be :math:`{\allocmodule}(s, {\module}, {{\externaddr}^\ast}, {{\val}_{\mathsf{g}}^\ast}, {{\reff}_{\mathsf{t}}^\ast}, {{{\reff}_{\mathsf{e}}^\ast}^\ast})`.

#. Let :math:`{F'}` be the :math:`\mathsf{frame}` :math:`\{ \AMODULE~{\moduleinst} \}`.

#. Push the :math:`\mathsf{frame}` :math:`{F'}`.

#. Execute the sequence :math:`{{\instr}_{\mathsf{e}}^\ast}`.

#. Execute the sequence :math:`{{\instr}_{\mathsf{d}}^\ast}`.

#. If :math:`{{\start}^?}` is defined, then:

   a. Let :math:`(\START~x)` be :math:`{{\start}^?}`.

   #. Let :math:`{\instr}_{\mathsf{s}}` be the :ref:`instruction <syntax-instr>` :math:`(\CALL~x)`.

   #. Execute the instruction :math:`{\instr}_{\mathsf{s}}`.

#. Pop the :math:`\mathsf{frame}` from the stack.

#. Return :math:`{\moduleinst}`.



.. math::
   \begin{array}[t]{@{}lcl@{}l@{}}
   {\instantiate}(s, {\module}, {{\externaddr}^\ast}) & = & {s''''} ; \{ \AMODULE~{\moduleinst} \} ; {{\instr}_{\mathsf{e}}^\ast}~{{\instr}_{\mathsf{d}}^\ast}~{{\instr}_{\mathsf{s}}^?} &  \\
    \multicolumn{4}{@{}l@{}}{\quad
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ {\vdashmodule}\, {\module} : {{\mathit{xt}}_{\mathsf{i}}^\ast} \toM {{\mathit{xt}}_{\mathsf{e}}^\ast} \\
   {\land}~ (s \vdashexternaddr {\externaddr} : {\mathit{xt}}_{\mathsf{i}})^\ast \\[0.8ex]
   {\land}~ {\module} = \MODULE~{{\type}^\ast}~{{\import}^\ast}~{{\tag}^\ast}~{{\global}^\ast}~{{\mem}^\ast}~{{\table}^\ast}~{{\func}^\ast}~{{\data}^\ast}~{{\elem}^\ast}~{{\start}^?}~{{\export}^\ast} \\
   {\land}~ {{\global}^\ast} = {(\GLOBAL~{\globaltype}~{\expr}_{\mathsf{g}})^\ast} \\
   {\land}~ {{\table}^\ast} = {(\TABLE~{\tabletype}~{\expr}_{\mathsf{t}})^\ast} \\
   {\land}~ {{\data}^\ast} = {(\DATA~{{\byte}^\ast}~{\datamode})^\ast} \\
   {\land}~ {{\elem}^\ast} = {(\ELEM~{\reftype}~{{\expr}_{\mathsf{e}}^\ast}~{\elemmode})^\ast} \\
   {\land}~ {{\start}^?} = {(\START~x)^?} \\
   {\land}~ {\moduleinst}_0 = \{ \begin{array}[t]{@{}l@{}}
   \MITYPES~{{{\alloctype}^\ast}}{({{\type}^\ast})},\; \\
     \MIGLOBALS~{\globalsxa}({{\externaddr}^\ast}),\; \\
     \MIFUNCS~{\funcsxa}({{\externaddr}^\ast})~{({|s{.}\SFUNCS|} + i_{\mathsf{f}})^{i_{\mathsf{f}}<{|{{\func}^\ast}|}}} \}\end{array} \\
   {\land}~ z = s ; \{ \AMODULE~{\moduleinst}_0 \} \\
   {\land}~ ({z'}, {{\val}_{\mathsf{g}}^\ast}) = {{{\mathrm{evalglobal}}^\ast}}{(z, {{\globaltype}^\ast}, {{\expr}_{\mathsf{g}}^\ast})} \\
   {\land}~ ({z''}, {{\reff}_{\mathsf{t}}^\ast}) = {{{\mathrm{evalexpr}}^\ast}}{({z'}, {{\expr}_{\mathsf{t}}^\ast})} \\
   {\land}~ ({z'''}, {{{\reff}_{\mathsf{e}}^\ast}^\ast}) = {{{{\mathrm{evalexpr}}^\ast}^\ast}}{({z''}, {{{\expr}_{\mathsf{e}}^\ast}^\ast})} \\
   {\land}~ {z'''} = {s'''} ; f \\
   {\land}~ ({s''''}, {\moduleinst}) = {\allocmodule}({s'''}, {\module}, {{\externaddr}^\ast}, {{\val}_{\mathsf{g}}^\ast}, {{\reff}_{\mathsf{t}}^\ast}, {({{\reff}_{\mathsf{e}}^\ast})^\ast}) \\
   {\land}~ {{\instr}_{\mathsf{d}}^\ast} = {\bigcat}\, {{{\rundata}}_{i_{\mathsf{d}}}({{\data}^\ast}{}[i_{\mathsf{d}}])^{i_{\mathsf{d}}<{|{{\data}^\ast}|}}} \\
   {\land}~ {{\instr}_{\mathsf{e}}^\ast} = {\bigcat}\, {{{\runelem}}_{i_{\mathsf{e}}}({{\elem}^\ast}{}[i_{\mathsf{e}}])^{i_{\mathsf{e}}<{|{{\elem}^\ast}|}}} \\
   {\land}~ {{\instr}_{\mathsf{s}}^?} = {(\CALL~x)^?} \\
   \end{array}
   } \\
   \end{array}

where:

.. _eval-exprs:


:math:`{{{\mathrm{evalexpr}}^\ast}}{(z, {{\expr''}^\ast})}`
...........................................................


1. If :math:`{{\expr''}^\ast} = \epsilon`, then:

   a. Return :math:`\epsilon`.

#. Let :math:`{\expr}~{{\expr'}^\ast}` be :math:`{{\expr''}^\ast}`.

#. Let :math:`{\reff}` be the result of :ref:`evaluating <exec-expr>` :math:`{\expr}` with state :math:`z`.

#. Let :math:`{{\reff'}^\ast}` be :math:`{{{\mathrm{evalexpr}}^\ast}}{(z, {{\expr'}^\ast})}`.

#. Return :math:`{\reff}~{{\reff'}^\ast}`.



.. math::
   \begin{array}[t]{@{}lcl@{}l@{}}
   {{{\mathrm{evalexpr}}^\ast}}{(z, \epsilon)} & = & (z, \epsilon) \\
   {{{\mathrm{evalexpr}}^\ast}}{(z, {\expr}~{{\expr'}^\ast})} & = & ({z''}, {\reff}~{{\reff'}^\ast}) &  \\
   && \multicolumn{2}{@{}l@{}}{\quad
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ z ; {\expr} \steptostar {z'} ; {\reff} \\
   {\land}~ ({z''}, {{\reff'}^\ast}) = {{{\mathrm{evalexpr}}^\ast}}{({z'}, {{\expr'}^\ast})} \\
   \end{array}
   } \\
   \end{array}

and:

.. _eval-globals:


:math:`{{{\mathrm{evalglobal}}^\ast}}{(z, {{\globaltype}^\ast}, {{\expr''}^\ast})}`
...................................................................................


1. If :math:`{{\expr''}^\ast} = \epsilon`, then:

   a. Assert: Due to validation, :math:`{{\globaltype}^\ast} = \epsilon`.

   #. Return :math:`\epsilon`.

#. Else:

   a. Let :math:`{\expr}~{{\expr'}^\ast}` be :math:`{{\expr''}^\ast}`.

   #. Assert: Due to validation, :math:`{|{{\globaltype}^\ast}|} \geq 1`.

   #. Let :math:`{\mathit{gt}}~{{\mathit{gt}'}^\ast}` be :math:`{{\globaltype}^\ast}`.

   #. Let :math:`{\val}` be the result of :ref:`evaluating <exec-expr>` :math:`{\expr}` with state :math:`z`.

   #. Let :math:`(s, f)` be the destructuring of :math:`z`.

   #. Let :math:`a` be :math:`{\allocglobal}(s, {\mathit{gt}}, {\val})`.

   #. Append :math:`a` to :math:`f{.}\AMODULE{.}\MIGLOBALS`.

   #. Let :math:`{{\val'}^\ast}` be :math:`{{{\mathrm{evalglobal}}^\ast}}{((s, f), {{\mathit{gt}'}^\ast}, {{\expr'}^\ast})}`.

   #. Return :math:`{\val}~{{\val'}^\ast}`.



.. math::
   \begin{array}[t]{@{}lcl@{}l@{}}
   {{{\mathrm{evalglobal}}^\ast}}{(z, \epsilon, \epsilon)} & = & (z, \epsilon) \\
   {{{\mathrm{evalglobal}}^\ast}}{(z, {\mathit{gt}}~{{\mathit{gt}'}^\ast}, {\expr}~{{\expr'}^\ast})} & = & ({z''}, {\val}~{{\val'}^\ast}) &  \\
   && \multicolumn{2}{@{}l@{}}{\quad
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ z ; {\expr} \steptostar {z'} ; {\val} \\
   {\land}~ {z'} = s ; f \\
   {\land}~ ({s'}, a) = {\allocglobal}(s, {\mathit{gt}}, {\val}) \\
   {\land}~ ({z''}, {{\val'}^\ast}) = {{{\mathrm{evalglobal}}^\ast}}{(({s'} ; f{}[{.}\AMODULE{.}\MIGLOBALS \mathrel{{=}{\oplus}} a]), {{\mathit{gt}'}^\ast}, {{\expr'}^\ast})} \\
   \end{array}
   } \\
   \end{array}

.. _aux-rundata:
.. _aux-runelem:


:math:`{{\rundata}}_{x}(\DATA~{b^{n}}~{\datamode})`
...................................................


1. If :math:`{\datamode} = \DPASSIVE`, then:

   a. Return :math:`\epsilon`.

#. Assert: Due to validation, :math:`{\datamode}` is some :math:`\DACTIVE~{\memidx}~{\expr}`.

#. Let :math:`(\DACTIVE~y~{{\instr}^\ast})` be the destructuring of :math:`{\datamode}`.

#. Return :math:`{{\instr}^\ast}~(\I32{.}\CONST~0)~(\I32{.}\CONST~n)~(\MEMORYINIT~y~x)~(\DATADROP~x)`.




:math:`{{\runelem}}_{x}(\ELEM~{\mathit{rt}}~{e^{n}}~{\elemmode})`
.................................................................


1. If :math:`{\elemmode} = \EPASSIVE`, then:

   a. Return :math:`\epsilon`.

#. If :math:`{\elemmode} = \EDECLARE`, then:

   a. Return :math:`(\ELEMDROP~x)`.

#. Assert: Due to validation, :math:`{\elemmode}` is some :math:`\EACTIVE~{\tableidx}~{\expr}`.

#. Let :math:`(\EACTIVE~y~{{\instr}^\ast})` be the destructuring of :math:`{\elemmode}`.

#. Return :math:`{{\instr}^\ast}~(\I32{.}\CONST~0)~(\I32{.}\CONST~n)~(\TABLEINIT~y~x)~(\ELEMDROP~x)`.



.. math::
   \begin{array}[t]{@{}lcl@{}l@{}}
   {{\rundata}}_{x}(\DATA~{b^{n}}~(\DPASSIVE)) & = & \epsilon \\
   {{\rundata}}_{x}(\DATA~{b^{n}}~(\DACTIVE~y~{{\instr}^\ast})) & = & & \\
    \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   {{\instr}^\ast}~(\I32{.}\CONST~0)~(\I32{.}\CONST~n)~(\MEMORYINIT~y~x)~(\DATADROP~x) \\
   \end{array}
   } \\[0.8ex]
   {{\runelem}}_{x}(\ELEM~{\mathit{rt}}~{e^{n}}~(\EPASSIVE)) & = & \epsilon \\
   {{\runelem}}_{x}(\ELEM~{\mathit{rt}}~{e^{n}}~(\EDECLARE)) & = & (\ELEMDROP~x) \\
   {{\runelem}}_{x}(\ELEM~{\mathit{rt}}~{e^{n}}~(\EACTIVE~y~{{\instr}^\ast})) & = & & \\
    \multicolumn{4}{@{}l@{}}{\quad
   \begin{array}[t]{@{}l@{}}
   {{\instr}^\ast}~(\I32{.}\CONST~0)~(\I32{.}\CONST~n)~(\TABLEINIT~y~x)~(\ELEMDROP~x) \\
   \end{array}
   } \\
   \end{array}

.. note::
   Checking import types assumes that the :ref:`module instance <syntax-moduleinst>` has already been :ref:`allocated <alloc-module>` to compute the respective :ref:`closed <type-closed>` :ref:`defined types <syntax-deftype>`.
   However, this forward reference merely is a way to simplify the specification.
   In practice, implementations will likely allocate or canonicalize types beforehand, when *compiling* a module, in a stage before instantiation and before imports are checked.

   Similarly, module :ref:`allocation <alloc-module>` and the :ref:`evaluation <exec-expr>` of :ref:`global <syntax-global>` and :ref:`table <syntax-table>` initializers as well as :ref:`element segments <syntax-elem>` are mutually recursive because the global initialization :ref:`values <syntax-val>` :math:`{{\val}_{\mathsf{g}}^\ast}`, :math:`{\reff}_{\mathsf{t}}`, and element segment contents :math:`{{{\reff}_{\mathsf{e}}^\ast}^\ast}` are passed to the module allocator while depending on the module instance :math:`{\moduleinst}` and store :math:`{s'}` returned by allocation.
   Again, this recursion is just a specification device.
   In practice, the initialization values can :ref:`be determined <exec-initvals>` beforehand by staging module allocation such that first, the module's own :ref:`function instances <syntax-funcinst>` are pre-allocated in the store, then the initializer expressions are evaluated in order, allocating globals on the way, then the rest of the module instance is allocated, and finally the new function instances' :math:`\mathsf{module}` fields are set to that module instance.
   This is possible because :ref:`validation <valid-module>` ensures that initialization expressions cannot actually call a function, only take their reference.

   All failure conditions are checked before any observable mutation of the store takes place.
   Store mutation is not atomic;
   it happens in individual steps that may be interleaved with other threads.

   :ref:`Evaluation <exec-expr>` of :ref:`constant expressions <valid-constant>` does not affect the store.


.. index:: ! invocation, module, module instance, function, export, function address, function instance, function type, value, stack, trap, exception, store
.. _exec-invocation:

Invocation
~~~~~~~~~~

Once a :ref:`module <syntax-module>` has been :ref:`instantiated <exec-instantiation>`, any exported function can be *invoked* externally via its :ref:`function address <syntax-funcaddr>` :math:`{\funcaddr}` in the :ref:`store <syntax-store>` :math:`s` and an appropriate list :math:`{{\val}^\ast}` of argument :ref:`values <syntax-val>`.

Invocation may *fail* with an error if the arguments do not fit the :ref:`function type <syntax-functype>`.
Invocation can also result in an :ref:`exception <exception>` or :ref:`trap <trap>`.
It is up to the :ref:`embedder <embedder>` to define how such conditions are reported.

.. note::
   If the :ref:`embedder <embedder>` API performs type checks itself, either statically or dynamically, before performing an invocation, then no failure other than traps or exceptions can occur.


:math:`{\invoke}(s, {\funcaddr}, {{\val}^\ast})`
................................................


1. Assert: Due to validation, the :ref:`expansion <aux-expand-deftype>` of :math:`s{.}\SFUNCS{}[{\funcaddr}]{.}\FITYPE` is some :math:`\TFUNC~{\resulttype} \Tarrow {\resulttype}`.

#. Let :math:`(\TFUNC~{t_1^\ast}~\Tarrow~{t_2^\ast})` be the destructuring of the :ref:`expansion <aux-expand-deftype>` of :math:`s{.}\SFUNCS{}[{\funcaddr}]{.}\FITYPE`.

#. If :math:`{|{t_1^\ast}|} \neq {|{{\val}^\ast}|}`, then:

   a. Fail.

#. For all :math:`t_1` in :math:`{t_1^\ast}`, and corresponding :math:`{\val}` in :math:`{{\val}^\ast}`:

   a. If :math:`{\val}` is not :ref:`valid <valid-val>` with type :math:`t_1`, then:

      1) Fail.

#. Let :math:`k` be the length of :math:`{t_2^\ast}`.

#. Let :math:`F` be the :math:`\mathsf{frame}` :math:`\{ \AMODULE~\{  \} \}` whose arity is :math:`k`.

#. Push the :math:`\mathsf{frame}` :math:`F`.

#. Push the values :math:`{{\val}^\ast}` to the stack.

#. Push the value :math:`(\REFFUNCADDR~{\funcaddr})` to the stack.

#. Execute the instruction :math:`(\CALLREF~s{.}\SFUNCS{}[{\funcaddr}]{.}\FITYPE)`.

#. Pop the values :math:`{{\val'}^{k}}` from the stack.

#. Pop the :math:`\mathsf{frame}` from the stack.

#. Return :math:`{{\val'}^{k}}`.



.. math::
   \begin{array}[t]{@{}lcl@{}l@{}}
   {\invoke}(s, {\funcaddr}, {{\val}^\ast}) & = & s ; \{ \AMODULE~\{  \} \} ; {{\val}^\ast}~(\REFFUNCADDR~{\funcaddr})~(\CALLREF~s{.}\SFUNCS{}[{\funcaddr}]{.}\FITYPE) &  \\
   && \multicolumn{2}{@{}l@{}}{\quad
   \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ s{.}\SFUNCS{}[{\funcaddr}]{.}\FITYPE \approxexpanddt \TFUNC~{t_1^\ast} \Tarrow {t_2^\ast} \\
   {\land}~ (s \vdashval {\val} : t_1)^\ast \\
   \end{array}
   } \\
   \end{array}
