## Modules

WebAssembly programs are organized into *modules*, which are the unit of deployment, loading, and compilation. A module collects definitions for types, tags, and globals, memories, tables, functions. In addition, it can declare imports and exports and provide initialization in the form of data and element segments, or a start function.

```text
module ::= module
  list(type)
  list(import)
  list(tag)
  list(global)
  list(mem)
  list(table)
  list(func)
  list(data)
  list(elem)
  start?
  list(export)
```

Each of the lists --- and thus the entire module --- may be empty.

### Indices

Definitions are referenced with zero-based *indices*. Each class of definition has its own *index space*, as distinguished by the following classes.

```text
idx ::= u32

typeidx ::= idx

funcidx ::= idx

globalidx ::= idx

tableidx ::= idx

memidx ::= idx

tagidx ::= idx

elemidx ::= idx

dataidx ::= idx

labelidx ::= idx

localidx ::= idx

fieldidx ::= idx
```

The index space for tags, globals, memories, tables, and functions includes respective imports declared in the same module. The indices of these imports precede the indices of other definitions in the same index space.

Data indices reference data segments and element indices reference element segments.

The index space for locals is only accessible inside a function and includes the parameters of that function, which precede the local variables.

Label indices reference structured control instructions inside an instruction sequence.

Each aggregate type provides an index space for its fields.

#### Conventions

* The meta variable `l` ranges over label indices.
* The meta variables `x`, `y` range over indices in any of the other index spaces.
* For every index space `abcidx`, the notation `abcidx(A)` denotes the set of indices from that index space occurring free in `A`. Sometimes this set is reinterpreted as the list of its elements.

> **Note:** For example, if `instr*` is `(data.drop 1) (memory.init 2 3)`, then `dataidx_instrs(instr*) = 1 3`, or equivalently, the set `{ 1, 3 }`.

### Types

The `type` section of a module defines a list of recursive types, each consisting of a list of sub types referenced by individual type indices. All function, structure, or array types used in a module must be defined in this section.

```text
type ::= type rectype
```

### Tags

The `tag` section of a module defines a list of *tags*:

```text
tag ::= tag tagtype
```

The type index of a tag must refer to a function type that declares its tag type.

Tags are referenced through tag indices, starting with the smallest index not referencing a tag import.

### Globals

The `global` section of a module defines a list of *global variables* (or *globals* for short):

```text
global ::= global globaltype expr
```

Each global stores a single value of the type specified in the global type. It also specifies whether a global is immutable or mutable. Moreover, each global is initialized with a value given by a constant initializer expression.

Globals are referenced through global indices, starting with the smallest index not referencing a global import.

### Memories

The `mem` section of a module defines a list of *linear memories* (or *memories* for short) as described by their memory type:

```text
mem ::= memory memtype
```

A memory is a list of raw uninterpreted bytes. The minimum size in the limits of its memory type specifies the initial size of that memory, while its maximum, if present, restricts the size to which it can grow later. Both are in units of page size.

Memories can be initialized through data segments.

Memories are referenced through memory indices, starting with the smallest index not referencing a memory import. Most constructs implicitly reference memory index `0`.

### Tables

The `table` section of a module defines a list of *tables* described by their table type:

```text
table ::= table tabletype expr
```

A table is an array of opaque values of a particular reference type that is specified by the table type. Each table slot is initialized with a value given by a constant initializer expression. Tables can further be initialized through element segments.

The minimum size in the limits of the table type specifies the initial size of that table, while its maximum restricts the size to which it can grow later.

Tables are referenced through table indices, starting with the smallest index not referencing a table import. Most constructs implicitly reference table index `0`.

### Functions

The `func` section of a module defines a list of *functions* with the following structure:

```text
func ::= func typeidx local* expr

local ::= local valtype
```

The type index of a function declares its signature by reference to a function type defined in the module. The parameters of the function are referenced through 0-based local indices in the function's body; they are mutable.

The locals declare a list of mutable local variables and their types. These variables are referenced through local indices in the function's body. The index of the first local is the smallest index not referencing a parameter.

A function's expression is an instruction sequence that represents the body of the function. Upon termination it must produce a stack matching the function type's result type.

Functions are referenced through function indices, starting with the smallest index not referencing a function import.

### Data Segments

The `data` section of a module defines a list of *data segments*, which can be used to initialize a range of memory from a static list of bytes.

```text
data ::= data byte* datamode

datamode ::= active memidx expr | passive
```

Similar to element segments, data segments have a mode that identifies them as either *active* or *passive*. A passive data segment's contents can be copied into a memory using the `memory.init` instruction. An active data segment copies its contents into a memory during instantiation, as specified by a memory index and a constant expression defining an offset into that memory.

Data segments are referenced through data indices.

### Element Segments

The `elem` section of a module defines a list of *element segments*, which can be used to initialize a subrange of a table from a static list of elements.

```text
elem ::= elem reftype expr* elemmode

elemmode ::= active tableidx expr | passive | declarative
```

Each element segment defines a reference type and a corresponding list of constant element expressions.

Element segments have a mode that identifies them as either *active*, *passive*, or *declarative*. A passive element segment's elements can be copied to a table using the `table.init` instruction. An active element segment copies its elements into a table during instantiation, as specified by a table index and a constant expression defining an offset into that table. A declarative element segment is not available at runtime but merely serves to forward-declare references that are formed in code with instructions like `ref.func`. The offset is given by another constant expression.

Element segments are referenced through element indices.

### Start Function

The `start` section of a module declares the function index of a *start function* that is automatically invoked when the module is instantiated, after tables and memories have been initialized.

```text
start ::= start funcidx
```

> **Note:** The start function is intended for initializing the state of a module. The module and its exports are not accessible externally before this initialization has completed.

### Imports

The `import` section of a module defines a set of *imports* that are required for instantiation.

```text
import ::= import name name externtype
```

Each import is labeled by a two-level name space, consisting of a *module name* and an *item name* for an entity within that module. Importable definitions are tags, globals, memories, tables, and functions. Each import is specified by a respective external type that a definition provided during instantiation is required to match.

Every import defines an index in the respective index space. In each index space, the indices of imports go before the first index of any definition contained in the module itself.

> **Note:** Unlike export names, import names are not necessarily unique. It is possible to import the same module/item name pair multiple times; such imports may even have different type descriptions, including different kinds of entities. A module with such imports can still be instantiated depending on the specifics of how an embedder allows resolving and supplying imports. However, embedders are not required to support such overloading, and a WebAssembly module itself cannot implement an overloaded name.

### Exports

The `export` section of a module defines a set of *exports* that become accessible to the host environment once the module has been instantiated.

```text
export ::= export name externidx

externidx ::= func funcidx | global globalidx | table tableidx | mem memidx | tag tagidx
```

Each export is labeled by a unique name. Exportable definitions are tags, globals, memories, tables, and functions, which are referenced through a respective index.

#### Conventions

The following auxiliary notation is defined for sequences of exports, filtering out indices of a specific kind in an order-preserving fashion:

```text
funcsxx(ε) = ε
funcsxx((func x) xx*) = x funcsxx(xx*)
funcsxx(externidx xx*) = funcsxx(xx*)   (otherwise)

tablesxx(ε) = ε
tablesxx((table x) xx*) = x tablesxx(xx*)
tablesxx(externidx xx*) = tablesxx(xx*)   (otherwise)

memsxx(ε) = ε
memsxx((mem x) xx*) = x memsxx(xx*)
memsxx(externidx xx*) = memsxx(xx*)   (otherwise)

globalsxx(ε) = ε
globalsxx((global x) xx*) = x globalsxx(xx*)
globalsxx(externidx xx*) = globalsxx(xx*)   (otherwise)

tagsxx(ε) = ε
tagsxx((tag x) xx*) = x tagsxx(xx*)
tagsxx(externidx xx*) = tagsxx(xx*)   (otherwise)
```
