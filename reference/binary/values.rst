.. index:: value
   pair: binary format; value
.. _binary-value:

Values
------


.. index:: byte
   pair: binary format; byte
.. _binary-byte:

Bytes
~~~~~

:ref:`Bytes <syntax-byte>` encode themselves.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Bbyte} & ::= & \mathtt{0x00} ~~|~~ \ldots ~~|~~ \mathtt{0xFF} \\
   \end{array}


.. index:: integer, unsigned integer, signed integer, uninterpreted integer, LEB128, two's complement
   pair: binary format; integer
   pair: binary format; unsigned integer
   pair: binary format; signed integer
   pair: binary format; uninterpreted integer
.. _binary-sint:
.. _binary-uint:
.. _binary-int:

Integers
~~~~~~~~

All :ref:`integers <syntax-int>` are encoded using the |LEB128|_ variable-length integer encoding, in either unsigned or signed variant.

:ref:`Unsigned integers <syntax-uint>` are encoded in |UnsignedLEB128|_ format.
As an additional constraint, the total number of bytes encoding a :math:`{{\uNX}}{N}` value must not exceed :math:`{\mathrm{ceil}}(N / 7)` bytes.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\BuNX}}{N} & ::= & n{:}{\Bbyte} & \quad\Rightarrow\quad{} & n & \quad \mbox{if}~ n < {2^{7}} \land n < {2^{N}} \\
   & & | & n{:}{\Bbyte}~~m{:}{{\BuNX}}{(N - 7)} & \quad\Rightarrow\quad{} & {2^{7}} \cdot m + (n - {2^{7}}) & \quad \mbox{if}~ n \geq {2^{7}} \land N > 7 \\
   \end{array}

:ref:`Signed integers <syntax-sint>` are encoded in |SignedLEB128|_ format, which uses a two's complement representation.
As an additional constraint, the total number of bytes encoding an :math:`{{\sNX}}{N}` value must not exceed :math:`{\mathrm{ceil}}(N / 7)` bytes.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\BsNX}}{N} & ::= & n{:}{\Bbyte} & \quad\Rightarrow\quad{} & n & \quad \mbox{if}~ n < {2^{6}} \land n < {2^{N - 1}} \\
   & & | & n{:}{\Bbyte} & \quad\Rightarrow\quad{} & n - {2^{7}} & \quad \mbox{if}~ {2^{6}} \leq n < {2^{7}} \land n \geq {2^{7}} - {2^{N - 1}} \\
   & & | & n{:}{\Bbyte}~~i{:}{{\BsNX}}{(N - 7)} & \quad\Rightarrow\quad{} & {2^{7}} \cdot i + (n - {2^{7}}) & \quad \mbox{if}~ n \geq {2^{7}} \land N > 7 \\
   \end{array}

:ref:`Uninterpreted integers <syntax-int>` are encoded as signed integers.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\BiNX}}{N} & ::= & i{:}{{\BsNX}}{N} & \quad\Rightarrow\quad{} & {{{{\signed}}_{N}^{{-1}}}}{(i)} \\
   \end{array}

.. note::
   The side conditions :math:`N > 7` in the productions for non-terminal bytes of the :math:`{{\uNX}}{N}` and :math:`{{\sNX}}{N}` encodings restrict the encoding's length.
   However, "trailing zeros" are still allowed within these bounds.
   For example, :math:`\mathtt{0x03}` and :math:`\mathtt{0x83}~\mathtt{0x00}` are both well-formed encodings for the value :math:`3` as a :math:`{\u8}`.
   Similarly, either of :math:`\mathtt{0x7E}` and :math:`\mathtt{0xFE}~\mathtt{0x7F}` and :math:`\mathtt{0xFE}~\mathtt{0xFF}~\mathtt{0x7F}` are well-formed encodings of the value :math:`{-2}` as an :math:`{\mathit{s{\kern-0.1em\scriptstyle 16}}}`.

   The side conditions on the value :math:`n` of terminal bytes further enforce that
   any unused bits in these bytes must be :math:`0` for positive values and :math:`1` for negative ones.
   For example, :math:`\mathtt{0x83}~\mathtt{0x10}` is malformed as a :math:`{\u8}` encoding.
   Similarly, both :math:`\mathtt{0x83}~\mathtt{0x3E}` and :math:`\mathtt{0xFF}~\mathtt{0x7B}` are malformed as :math:`{\mathit{s{\kern-0.1em\scriptstyle 8}}}` encodings.


.. index:: floating-point number, little endian
   pair: binary format; floating-point number
.. _binary-float:

Floating-Point
~~~~~~~~~~~~~~

:ref:`Floating-point <syntax-float>` values are encoded directly by their |IEEE754|_ (Section 3.4) bit pattern in |LittleEndian|_ byte order:

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\BfNX}}{N} & ::= & {b^\ast}{:}{{\Bbyte}^{N / 8}} & \quad\Rightarrow\quad{} & {{{{\bytes}}_{{\FNX}{N}}^{{-1}}}}{({b^\ast})} \\
   \end{array}


.. index:: name, byte, Unicode, ! UTF-8
   pair: binary format; name
.. _binary-utf8:
.. _binary-name:

Names
~~~~~

:ref:`Names <syntax-name>` are encoded as a :ref:`list <binary-list>` of bytes containing the |Unicode|_ (Section 3.9) UTF-8 encoding of the name's character sequence.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Bname} & ::= & {b^\ast}{:}{\Blist}({\Bbyte}) & \quad\Rightarrow\quad{} & {\name} & \quad \mbox{if}~ {\utf8}({\name}) = {b^\ast} \\
   \end{array}

The auxiliary :math:`{\utf8}` function expressing this encoding is defined as follows:

.. math::
   \begin{array}[t]{@{}lcl@{}l@{}}
   {\utf8}({{\mathit{ch}}^\ast}) & = & {\bigcat}\, {{\utf8}({\mathit{ch}})^\ast} \\
   {\utf8}({\mathit{ch}}) & = & b & \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ {\mathit{ch}} < \mathrm{U{+}80} \\
   {\land}~ {\mathit{ch}} = b \\
   \end{array} \\
   {\utf8}({\mathit{ch}}) & = & b_1~b_2 & \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ \mathrm{U{+}80} \leq {\mathit{ch}} < \mathrm{U{+}0800} \\
   {\land}~ {\mathit{ch}} = {2^{6}} \cdot (b_1 - \mathtt{0xC0}) + {\mathrm{cont}}(b_2) \\
   \end{array} \\
   {\utf8}({\mathit{ch}}) & = & b_1~b_2~b_3 & \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ \mathrm{U{+}0800} \leq {\mathit{ch}} < \mathrm{U{+}D800} \lor \mathrm{U{+}E000} \leq {\mathit{ch}} < \mathrm{U{+}10000} \\
   {\land}~ {\mathit{ch}} = {2^{12}} \cdot (b_1 - \mathtt{0xE0}) + {2^{6}} \cdot {\mathrm{cont}}(b_2) + {\mathrm{cont}}(b_3) \\
   \end{array} \\
   {\utf8}({\mathit{ch}}) & = & b_1~b_2~b_3~b_4 & \quad
   \begin{array}[t]{@{}l@{}}
   \mbox{if}~ \mathrm{U{+}10000} \leq {\mathit{ch}} < \mathrm{U{+}11000} \\
   {\land}~ {\mathit{ch}} = {2^{18}} \cdot (b_1 - \mathtt{0xF0}) + {2^{12}} \cdot {\mathrm{cont}}(b_2) + {2^{6}} \cdot {\mathrm{cont}}(b_3) + {\mathrm{cont}}(b_4) \\
   \end{array} \\
   \end{array}

where :math:`\begin{array}[t]{@{}l@{~}c@{~}l@{}l@{}} {\mathrm{cont}}(b) & = & b - \mathtt{0x80} & \quad \mbox{if}~ (\mathtt{0x80} < b < \mathtt{0xC0}) \\ \end{array}`

.. note::
   Unlike in some other formats, name strings are not 0-terminated.
