## Custom Sections and Annotations

This appendix defines dedicated [custom sections](binary-customsec) for WebAssembly's [binary format](binary) and [annotations](text-annot) for the text format.

Such sections or annotations do not contribute to, or otherwise affect, the WebAssembly semantics, and may be ignored by an implementation.

However, they provide useful meta data that implementations can make use of to improve user experience or take compilation hints.

### Name Section

The *name section* is a [custom section](binary-customsec) whose name string is itself `name`.

The name section should appear only once in a module, and only after the [data section](binary-datasec).

The purpose of this section is to attach printable names to definitions in a module, which e.g. can be used by a debugger or when parts of the module are to be rendered in [text form](text).

> **Note:** All [names](binary-name) are represented in Unicode encoded in UTF-8. Names need not be unique.

#### Subsections

The [data](binary-customsec) of a name section consists of a sequence of *subsections*.
Each subsection consists of a

* a one-byte subsection *id*,
* the U32 *size* of the contents, in bytes,
* the actual *contents*, whose structure is dependent on the subsection id.

```text
name section ::= section_0(namedata)

name data ::= n:name               (if n = name)
            | modulenamesubsec?
            | funcnamesubsec?
            | localnamesubsec?
            | typenamesubsec?
            | fieldnamesubsec?
            | tagnamesubsec?

name subsection ::= N:byte  size:u32  B   (if size = |B|)
```

The following subsection ids are used:

| Id | Subsection |
|----|------------|
| 0  | [module name](binary-modulenamesec) |
| 1  | [function names](binary-funcnamesec) |
| 2  | [local names](binary-localnamesec) |
| 4  | [type names](binary-typenamesec) |
| 10 | [field names](binary-fieldnamesec) |
| 11 | [tag names](binary-tagnamesec) |

Each subsection may occur at most once, and in order of increasing id.

#### Name Maps

A *name map* assigns [names](syntax-name) to [indices](syntax-index) in a given [index space](syntax-index).

It consists of a [list](binary-list) of index/name pairs in order of increasing index value.

Each index must be unique, but the assigned names need not be.

```text
name map ::= list(nameassoc)

name association ::= idx  name
```

An *indirect name map* assigns [names](syntax-name) to a two-dimensional [index space](syntax-index), where secondary indices are *grouped* by primary indices.

It consists of a list of primary index/name map pairs in order of increasing index value, where each name map in turn maps secondary indices to names.

Each primary index must be unique, and likewise each secondary index per individual name map.

```text
indirect name map ::= list(indirectnameassoc)

indirect name association ::= idx  namemap
```

#### Module Names

The *module name subsection* has the id 0.

It simply consists of a single [name](binary-name) that is assigned to the module itself.

```text
module name subsection ::= namesubsection_0(name)
```

#### Function Names

The *function name subsection* has the id 1.

It consists of a [name map](binary-namemap) assigning function names to [function indices](syntax-funcidx).

```text
function name subsection ::= namesubsection_1(namemap)
```

#### Local Names

The *local name subsection* has the id 2.

It consists of an [indirect name map](binary-indirectnamemap) assigning local names to [local indices](syntax-localidx) grouped by [function indices](syntax-funcidx).

```text
local name subsection ::= namesubsection_2(indirectnamemap)
```

#### Type Names

The *type name subsection* has the id 4.

It consists of a [name map](binary-namemap) assigning type names to [type indices](syntax-typeidx).

```text
type name subsection ::= namesubsection_4(namemap)
```

#### Field Names

The *field name subsection* has the id 10.

It consists of an [indirect name map](binary-indirectnamemap) assigning field names to [field indices](syntax-fieldidx) grouped by the [type indices](syntax-typeidx) of their respective [structure types](syntax-structtype).

```text
field name subsection ::= namesubsection_10(indirectnamemap)
```

#### Tag Names

The *tag name subsection* has the id 11.

It consists of a [name map](binary-namemap) assigning tag names to [tag indices](syntax-tagidx).

```text
tag name subsection ::= namesubsection_11(namemap)
```

### Name Annotations

*Name annotations* are the textual analogue to the [name section](binary-namesec) and provide a textual representation for it.

Consequently, their id is `@name`.

Analogous to the name section, name annotations are allowed on [modules](text-module), [functions](text-func), and [locals](text-local) (including [parameters](text-param)).

They can be placed where the text format allows binding occurrences of respective [identifiers](text-id).

If both an identifier and a name annotation are given, the annotation is expected *after* the identifier.

In that case, the annotation takes precedence over the identifier as a textual representation of the binding's name.

At most one name annotation may be given per binding.

All name annotations have the following format:

```text
name annotation ::= (@name string)
```

> **Note:** All name annotations can be arbitrary UTF-8 [strings](text-string). Names need not be unique.

#### Module Names

A *module name annotation* must be placed on a [module](text-module) definition, directly after the `module` keyword, or if present, after the following module [identifier](text-id).

```text
module name annotation ::= nameannot
```

#### Function Names

A *function name annotation* must be placed on a [function](text-func) definition or function [import](text-import), directly after the `func` keyword, or if present, after the following function [identifier](text-id) or.

```text
function name annotation ::= nameannot
```

#### Parameter Names

A *parameter name annotation* must be placed on a [parameter](text-param) declaration, directly after the `param` keyword, or if present, after the following parameter [identifier](text-id).

It may only be placed on a declaration that declares exactly one parameter.

```text
parameter name annotation ::= nameannot
```

#### Local Names

A *local name annotation* must be placed on a [local](text-param) declaration, directly after the `local` keyword, or if present, after the following local [identifier](text-id).

It may only be placed on a declaration that declares exactly one local.

```text
local name annotation ::= nameannot
```

#### Type Names

A *type name annotation* must be placed on a [type](text-type) declaration, directly after the `type` keyword, or if present, after the following type [identifier](text-id).

```text
type name annotation ::= nameannot
```

#### Field Names

A *field name annotation* must be placed on the field of a [structure type](text-structtype), directly after the `field` keyword, or if present, after the following field [identifier](text-id).

It may only be placed on a declaration that declares exactly one field.

```text
field name annotation ::= nameannot
```

#### Tag Names

A *tag name annotation* must be placed on a [tag declaration](text-tag) or tag [import](text-import), directly after the `tag` keyword, or if present, after the following tag [identifier](text-id).

```text
tag name annotation ::= nameannot
```

### Custom Annotations

*Custom annotations* are a generic textual representation for any [custom section](binary-customsec).
Their id is `@custom`.

By generating custom annotations, tools converting between [binary format](binary) and [text format](text) can maintain and round-trip the content of custom sections even when they do not recognize them.

Custom annotations must be placed inside a [module](text-module) definition.

They must occur anywhere after the `module` keyword, or if present, after the following module [identifier](text-id).

They must not be nested into other constructs.

```text
custom annotation ::= (@custom string customplace? datastring)

custom placement ::= (before first)
                  | (before sec)
                  | (after sec)
                  | (after last)

section ::= type
          | import
          | func
          | table
          | memory
          | global
          | export
          | start
          | elem
          | code
          | data
          | datacount
```

The first [string](text-string) in a custom annotation denotes the name of the custom section it represents.

The remaining strings collectively represent the section's payload data, written as a [data string](text-datastring), which can be split up into a possibly empty sequence of individual string literals (similar to [data segments](text-data)).

An arbitrary number of custom annotations (even of the same name) may occur in a module, each defining a separate custom section when converting to [binary format](binary).

Placement of the sections in the binary can be customized via explicit *placement* directives, that position them either directly before or directly after a known section.

That section must exist and be non-empty in the binary encoding of the annotated module.

The placements `(before first)` and `(after last)` denote virtual sections before the first and after the last known section, respectively.

When the placement directive is omitted, it defaults to `(after last)`.

If multiple placement directives appear for the same position, then the sections are all placed there, in order of their appearance in the text.

For this purpose, the position `after` a section is considered different from the position `before` the consecutive section, and the former occurs before the latter.

> **Note:** Future versions of WebAssembly may introduce additional sections between others or at the beginning or end of a module. Using `first` and `last` guarantees that placement will still go before or after any future section, respectively.

If a custom section with a specific section id is given as well as annotations representing the same custom section (e.g., `@name` [annotations](text-nameannot) as well as a `@custom` annotation for a `name` [section](binary-namesec)), then two sections are assumed to be created.

Their relative placement will depend on the placement directive given for the `@custom` annotation as well as the implicit placement requirements of the custom section, which are applied to the other annotation.

> **Note:** For example, the following module,
>
> ```none
> (module
>   (@custom "A" "aaa")
>   (type $t (func))
>   (@custom "B" (after func) "bbb")
>   (@custom "C" (before func) "ccc")
>   (@custom "D" (after last) "ddd")
>   (table 10 funcref)
>   (func (type $t))
>   (@custom "E" (after import) "eee")
>   (@custom "F" (before type) "fff")
>   (@custom "G" (after data) "ggg")
>   (@custom "H" (after code) "hhh")
>   (@custom "I" (after func) "iii")
>   (@custom "J" (before func) "jjj")
>   (@custom "K" (before first) "kkk")
> )
> ```
>
> will result in the following section ordering:
>
> ```none
> custom section "K"
> custom section "F"
> type section
> custom section "E"
> custom section "C"
> custom section "J"
> function section
> custom section "B"
> custom section "I"
> table section
> code section
> custom section "H"
> custom section "G"
> custom section "A"
> custom section "D"
> ```
