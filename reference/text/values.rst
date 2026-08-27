.. index:: value
   pair: text format; value
.. _text-value:

Values
------

The grammar productions in this section define *lexical syntax*,
hence no :ref:`white space <text-space>` is allowed.


.. index:: integer, unsigned integer, signed integer, uninterpreted integer
   pair: text format; integer
   pair: text format; unsigned integer
   pair: text format; signed integer
   pair: text format; uninterpreted integer
.. _text-sign:
.. _text-digit:
.. _text-hexdigit:
.. _text-num:
.. _text-hexnum:
.. _text-sint:
.. _text-uint:
.. _text-int:

Integers
~~~~~~~~

All :ref:`integers <syntax-int>` can be written in either decimal or hexadecimal notation.
In both cases, digits can optionally be separated by underscores.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Tsign} & ::= & \epsilon ~\Rightarrow~ {+1} ~~|~~ \mbox{‘\texttt{+}’} ~\Rightarrow~ {+1} ~~|~~ \mbox{‘\texttt{{-}}’} ~\Rightarrow~ {-1} \\[0.8ex]
   & {\Tdigit} & ::= & \mbox{‘\texttt{0}’} ~\Rightarrow~ 0 ~~|~~ \ldots ~~|~~ \mbox{‘\texttt{9}’} ~\Rightarrow~ 9 \\
   & {\Thexdigit} & ::= & d{:}{\Tdigit} ~\Rightarrow~ d \\
   & & | & \mbox{‘\texttt{A}’} ~\Rightarrow~ 10 ~~|~~ \ldots ~~|~~ \mbox{‘\texttt{F}’} ~\Rightarrow~ 15 \\
   & & | & \mbox{‘\texttt{a}’} ~\Rightarrow~ 10 ~~|~~ \ldots ~~|~~ \mbox{‘\texttt{f}’} ~\Rightarrow~ 15 \\[0.8ex]
   & {\Tnum} & ::= & d{:}{\Tdigit} & \quad\Rightarrow\quad{} & d \\
   & & | & n{:}{\Tnum}~~{\mbox{‘\texttt{\_}’}^?}~~d{:}{\Tdigit} & \quad\Rightarrow\quad{} & 10 \, n + d \\
   & {\Thexnum} & ::= & h{:}{\Thexdigit} & \quad\Rightarrow\quad{} & h \\
   & & | & n{:}{\Thexnum}~~{\mbox{‘\texttt{\_}’}^?}~~h{:}{\Thexdigit} & \quad\Rightarrow\quad{} & 16 \, n + h \\
   \end{array}

The allowed syntax for integer literals depends on size and signedness.
Moreover, their value must lie within the range of the respective type.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\TuNX}}{N} & ::= & n{:}{\Tnum} & \quad\Rightarrow\quad{} & n & \quad \mbox{if}~ n < {2^{N}} \\
   & & | & \mbox{‘\texttt{0x}’}~~n{:}{\Thexnum} & \quad\Rightarrow\quad{} & n & \quad \mbox{if}~ n < {2^{N}} \\
   & {{\TsNX}}{N} & ::= & s{:}{\Tsign}~~n{:}{{\TuNX}}{N} & \quad\Rightarrow\quad{} & s \cdot n & \quad \mbox{if}~ {-{2^{N - 1}}} \leq s \cdot n < {2^{N - 1}} \\
   \end{array}

:ref:`Uninterpreted integers <syntax-int>` can be written as either signed or unsigned, and are normalized to unsigned in the abstract syntax.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\TiNX}}{N} & ::= & n{:}{{\TuNX}}{N} & \quad\Rightarrow\quad{} & n \\
   & & | & i{:}{{\TsNX}}{N} & \quad\Rightarrow\quad{} & {{{{\signed}}_{N}^{{-1}}}}{(i)} \\
   \end{array}


.. index:: floating-point number, mantissa
   pair: text format; floating-point number
.. _text-frac:
.. _text-hexfrac:
.. _text-mant:
.. _text-hexmant:
.. _text-float:
.. _text-hexfloat:

Floating-Point
~~~~~~~~~~~~~~

:ref:`Floating-point <syntax-float>` values can be represented in either decimal or hexadecimal notation.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Tfrac} & ::= & d{:}{\Tdigit} & \quad\Rightarrow\quad{} & d / 10 \\
   & & | & d{:}{\Tdigit}~~{\mbox{‘\texttt{\_}’}^?}~~p{:}{\Tfrac} & \quad\Rightarrow\quad{} & (d + p / 10) / 10 \\
   & {\Thexfrac} & ::= & h{:}{\Thexdigit} & \quad\Rightarrow\quad{} & h / 16 \\
   & & | & h{:}{\Thexdigit}~~{\mbox{‘\texttt{\_}’}^?}~~p{:}{\Thexfrac} & \quad\Rightarrow\quad{} & (h + p / 16) / 16 \\[0.8ex]
   & {\Tmant} & ::= & p{:}{\Tnum}~~{\mbox{‘\texttt{.}’}^?} & \quad\Rightarrow\quad{} & p \\
   & & | & p{:}{\Tnum}~~\mbox{‘\texttt{.}’}~~q{:}{\Tfrac} & \quad\Rightarrow\quad{} & p + q \\
   & {\Thexmant} & ::= & p{:}{\Thexnum}~~{\mbox{‘\texttt{.}’}^?} & \quad\Rightarrow\quad{} & p \\
   & & | & p{:}{\Thexnum}~~\mbox{‘\texttt{.}’}~~q{:}{\Thexfrac} & \quad\Rightarrow\quad{} & p + q \\[0.8ex]
   & {\Tfloat} & ::= & p{:}{\Tmant}~~(\mbox{‘\texttt{E}’} ~~|~~ \mbox{‘\texttt{e}’})~~s{:}{\Tsign}~~e{:}{\Tnum} & \quad\Rightarrow\quad{} & p \cdot {10^{s \cdot e}} \\
   & {\Thexfloat} & ::= & \mbox{‘\texttt{0x}’}~~p{:}{\Thexmant}~~(\mbox{‘\texttt{P}’} ~~|~~ \mbox{‘\texttt{p}’})~~s{:}{\Tsign}~~e{:}{\Tnum} & \quad\Rightarrow\quad{} & p \cdot {2^{s \cdot e}} \\
   \end{array}

The value of a literal must not lie outside the representable range of the corresponding |IEEE754|_ type
(that is, a numeric value must not overflow to :math:`{\pm\infty}`),
but it may be :ref:`rounded <aux-ieee>` to the nearest representable value.

.. note::
   Rounding can be prevented by using hexadecimal notation with no more significant bits than supported by the required type.

Floating-point values may also be written as constants for *infinity* or *canonical NaN* (*not a number*).
Furthermore, arbitrary NaN values may be expressed by providing an explicit payload value.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {{\TfNX}}{N} & ::= & ({+1}){:}{\Tsign}~~q{:}{\TfNmag} & \quad\Rightarrow\quad{} & {+q} \\
   & & | & ({-1}){:}{\Tsign}~~q{:}{\TfNmag} & \quad\Rightarrow\quad{} & {-q} \\
   & {\TfNmag} & ::= & q{:}{\Tfloat} & \quad\Rightarrow\quad{} & {{\ieee}}_{N}(q) & \quad \mbox{if}~ {{\ieee}}_{N}(q) \neq \infty \\
   & & | & q{:}{\Thexfloat} & \quad\Rightarrow\quad{} & {{\ieee}}_{N}(q) & \quad \mbox{if}~ {{\ieee}}_{N}(q) \neq \infty \\
   & & | & \mbox{‘\texttt{inf}’} & \quad\Rightarrow\quad{} & \infty \\
   & & | & \mbox{‘\texttt{nan}’} & \quad\Rightarrow\quad{} & {\NAN}{({{\canon}}_{N})} \\
   & & | & \mbox{‘\texttt{nan:0x}’}~~n{:}{\Thexnum} & \quad\Rightarrow\quad{} & {\NAN}{(n)} & \quad \mbox{if}~ 1 \leq n < {2^{{\signif}(N)}} \\
   \end{array}


.. index:: ! string, byte, character, ASCII, Unicode, UTF-8
   pair: text format; byte
   pair: text format; string
.. _text-byte:
.. _text-string:

Strings
~~~~~~~

*Strings* denote sequences of bytes that can represent both textual and binary data.
They are enclosed in quotation marks
and may contain any character other than |ASCII|_ control characters, quotation marks (:math:`\mbox{‘\texttt{\kern-0.02em{'}\kern-0.05em{'}\kern-0.02em}’}`), or backslash (:math:`\mbox{‘\texttt{\(\mathtt{\backslash}\)}’}`),
except when expressed with an *escape sequence*.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Tstring} & ::= & \mbox{‘\texttt{\kern-0.02em{'}\kern-0.05em{'}\kern-0.02em}’}~~{({b^\ast}{:}{\Tstringelem})^\ast}~~\mbox{‘\texttt{\kern-0.02em{'}\kern-0.05em{'}\kern-0.02em}’} & \quad\Rightarrow\quad{} & {\bigcat}\, {{b^\ast}^\ast} & \quad \mbox{if}~ {|{\bigcat}\, {{b^\ast}^\ast}|} < {2^{32}} \\[0.8ex]
   & {\Tstringelem} & ::= & c{:}{\Tstringchar} & \quad\Rightarrow\quad{} & {\utf8}(c) \\
   & & | & \mbox{‘\texttt{\(\mathtt{\backslash}\)}’}~~h_1{:}{\Thexdigit}~~h_2{:}{\Thexdigit} & \quad\Rightarrow\quad{} & 16 \, h_1 + h_2 \\
   \end{array}

Each character in a string literal represents the byte sequence corresponding to its UTF-8 |Unicode|_ (Section 2.5) encoding,
except for hexadecimal escape sequences :math:`\mbox{‘\texttt{\(\mathtt{\backslash}\)hh}’}`, which represent raw bytes of the respective value.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Tstringchar} & ::= & c{:}{\Tchar} & \quad\Rightarrow\quad{} & c & \quad \mbox{if}~ c \geq \mathrm{U{+}20} \land c \neq \mathrm{U{+}7F} \land c \neq \mbox{‘\texttt{\kern-0.02em{'}\kern-0.05em{'}\kern-0.02em}’} \land c \neq \mbox{‘\texttt{\(\mathtt{\backslash}\)}’} \\
   & & | & \mbox{‘\texttt{\(\mathtt{\backslash}\)t}’} & \quad\Rightarrow\quad{} & \mathrm{U{+}09} \\
   & & | & \mbox{‘\texttt{\(\mathtt{\backslash}\)n}’} & \quad\Rightarrow\quad{} & \mathrm{U{+}0A} \\
   & & | & \mbox{‘\texttt{\(\mathtt{\backslash}\)r}’} & \quad\Rightarrow\quad{} & \mathrm{U{+}0D} \\
   & & | & \mbox{‘\texttt{\(\mathtt{\backslash}\)\kern-0.02em{'}\kern-0.05em{'}\kern-0.02em}’} & \quad\Rightarrow\quad{} & \mathrm{U{+}22} \\
   & & | & \mbox{‘\texttt{\(\mathtt{\backslash}\)\kern0.03em{'}\kern0.03em}’} & \quad\Rightarrow\quad{} & \mathrm{U{+}27} \\
   & & | & \mbox{‘\texttt{\(\mathtt{\backslash}\)\(\mathtt{\backslash}\)}’} & \quad\Rightarrow\quad{} & \mathrm{U{+}5C} \\
   & & | & \mbox{‘\texttt{\(\mathtt{\backslash}\)u{\{}}’}~~n{:}{\Thexnum}~~\mbox{‘\texttt{{\}}}’} & \quad\Rightarrow\quad{} & n & \quad \mbox{if}~ n < \mathtt{0xD800} \lor \mathtt{0xE800} \leq n < \mathtt{0x110000} \\
   \end{array}


.. index:: name, byte, character, character
   pair: text format; name
.. _text-name:

Names
~~~~~

:ref:`Names <syntax-name>` are strings denoting a literal character sequence. 
A name string must form a valid UTF-8 encoding as defined by |Unicode|_ (Section 2.5) and is interpreted as a string of Unicode scalar values.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Tname} & ::= & {b^\ast}{:}{\Tstring} & \quad\Rightarrow\quad{} & {c^\ast} & \quad \mbox{if}~ {b^\ast} = {\utf8}({c^\ast}) \\
   \end{array}

.. note::
   Presuming the source text is itself encoded correctly,
   strings that do not contain any uses of hexadecimal byte escapes are always valid names.


.. index:: ! identifiers
   pair: text format; identifiers
.. _text-idchar:
.. _text-id:

Identifiers
~~~~~~~~~~~

:ref:`Indices <syntax-index>` can be given in both numeric and symbolic form.
Symbolic *identifiers* that stand in lieu of indices start with :math:`\mbox{‘\texttt{\$}’}`, followed by either a sequence of printable |ASCII|_ characters that does not contain a space, quotation mark, comma, semicolon, or bracket, or by a quoted :ref:`name <text-name>`.

.. math::
   \begin{array}[t]{@{}l@{}rrl@{}l@{}l@{}l@{}}
   & {\Tid} & ::= & \mbox{‘\texttt{\$}’}~~{c^\ast}{:}{{\Tidchar}^{+}} & \quad\Rightarrow\quad{} & {c^\ast} \\
   & & | & \mbox{‘\texttt{\$}’}~~{c^\ast}{:}{\Tname} & \quad\Rightarrow\quad{} & {c^\ast} & \quad \mbox{if}~ {|{c^\ast}|} > 0 \\
   & {\Tidchar} & ::= & \mbox{‘\texttt{0}’} ~~|~~ \ldots ~~|~~ \mbox{‘\texttt{9}’} \\
   & & | & \mbox{‘\texttt{A}’} ~~|~~ \ldots ~~|~~ \mbox{‘\texttt{Z}’} \\
   & & | & \mbox{‘\texttt{a}’} ~~|~~ \ldots ~~|~~ \mbox{‘\texttt{z}’} \\
   & & | & \mbox{‘\texttt{!}’} ~~|~~ \mbox{‘\texttt{\#}’} ~~|~~ \mbox{‘\texttt{\$}’} ~~|~~ \mbox{‘\texttt{\%}’} ~~|~~ \mbox{‘\texttt{\&}’} ~~|~~ \mbox{‘\texttt{\kern0.03em{'}\kern0.03em}’} ~~|~~ \mbox{‘\texttt{*}’} ~~|~~ \mbox{‘\texttt{+}’} ~~|~~ \mbox{‘\texttt{{-}}’} ~~|~~ \mbox{‘\texttt{.}’} ~~|~~ \mbox{‘\texttt{/}’} \\
   & & | & \mbox{‘\texttt{:}’} ~~|~~ \mbox{‘\texttt{{<}}’} ~~|~~ \mbox{‘\texttt{{=}}’} ~~|~~ \mbox{‘\texttt{{>}}’} ~~|~~ \mbox{‘\texttt{?}’} ~~|~~ \mbox{‘\texttt{@}’} ~~|~~ \mbox{‘\texttt{\(\mathtt{\backslash}\)}’} ~~|~~ \mbox{‘\texttt{\(\mathtt{\hat{~~}}\)}’} ~~|~~ \mbox{‘\texttt{\_}’} ~~|~~ \mbox{‘\texttt{\(\mathtt{\grave{~~}}\)}’} ~~|~~ \mbox{‘\texttt{|}’} ~~|~~ \mbox{‘\texttt{\(\mathtt{\tilde{~~}}\)}’} \\
   \end{array}

.. note::
   The value of an identifier character is the Unicode codepoint denoting it.

.. _text-id-fresh:

Conventions
...........

The expansion rules of some abbreviations require insertion of a *fresh* identifier.
That may be any syntactically valid identifier that does not already occur in the given source text.
