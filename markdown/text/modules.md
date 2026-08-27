## Modules

Modules consist of a sequence of declarations.

The grammar rules for each declaration construct produce a pair, consisting of not just the abstract syntax representing the respective declaration, but also an identifier context recording the new symbolic identifiers bound by the construct, for use in the remainder of the module.

### Indices

Indices can be given either in raw numeric form or as symbolic identifiers when bound by a respective construct.

Such identifiers are looked up in the suitable space of the identifier context `I`.

```text
idx_ids ::=
  | x : u32  => x
  | id : id  => x   (if ids[x] = id)

typeidx_I ::= idx_{I.TYPES}

tagidx_I ::= idx_{I.TAGS}

globalidx_I ::= idx_{I.GLOBALS}

memidx_I ::= idx_{I.MEMS}

tableidx_I ::= idx_{I.TABLES}

funcidx_I ::= idx_{I.FUNCS}

dataidx_I ::= idx_{I.DATAS}

elemidx_I ::= idx_{I.ELEMS}

localidx_I ::= idx_{I.LOCALS}

labelidx_I ::= idx_{I.LABELS}

fieldidx_{I,x} ::= idx_{I.FIELDS[x]}
```

### Types

A type definition consists of a recursive type.

The identifier context produced for the local bindings is further extended with the respective sequence of defined types that the recursive type generates.

```text
type_I ::=
  | (qt, I') : rectype_I  =>  (type qt, I' ⊕ I'')
        (if qt = rec st^n
         && I'' = { TYPEDEFS (qt . i)^{i<n} })
```

### Tags

A tag definition can bind a symbolic tag identifier.

```text
tag_I ::=
  | '(' 'tag' id^? : id^? jt : tagtype_I ')'  =>  (tag jt, { TAGS (id^?) })
```

#### Abbreviations

Tags can be defined as imports or exports inline:

```text
import_I ::=
  | ...
  | '(' 'tag' id^? '(' 'import' name^2 ')' tagtype_I ')'
        ≡  '(' 'import' name^2 '(' 'tag' id^? tagtype_I ')' ')'

export_I ::=
  | '(' 'tag' id^? : id^? '(' 'export' name ')' ... ')'
        ≡  '(' 'tag' id' : id '...' ')'
            '(' 'export' name '(' 'tag' id ')' ')'
        (if id^? = id' || id^? = ε && id' ∉ I.TAGS)
```

> **Note:** The latter abbreviation can be applied repeatedly, if "`...`" contains additional export clauses. Consequently, a memory declaration can contain any number of exports, possibly followed by an import.

### Globals

Global definitions can bind a symbolic global identifier.

```text
global_I ::=
  | '(' 'global' id^? : id^? gt : globaltype_I e : expr_I ')'  =>  (global gt e, { GLOBALS (id^?) })
```

#### Abbreviations

Globals can be defined as imports or exports inline:

```text
import_I ::=
  | ...
  | '(' 'global' id^? '(' 'import' name^2 ')' globaltype_I ')'
        ≡  '(' 'import' name^2 '(' 'global' id^? globaltype_I ')' ')'

export_I ::=
  | '(' 'global' id^? : id^? '(' 'export' name ')' ... ')'
        ≡  '(' 'global' id' : id '...' ')'
            '(' 'export' name '(' 'global' id ')' ')'
        (if id^? = id' || id^? = ε && id' ∉ I.GLOBALS)
```

> **Note:** The latter abbreviation can be applied repeatedly, if "`...`" contains additional export clauses.Consequently, a global declaration can contain any number of exports, possibly followed by an import.

### Memories

Memory definitions can bind a symbolic memory identifier.

```text
mem_I ::=
  | '(' 'memory' id^? : id^? mt : memtype_I ')'  =>  (memory mt, { MEMS (id^?) })
```

#### Abbreviations

A data segment can be given inline with a memory definition, in which case its offset is `0` and the limits of the memory type are inferred from the length of the data, rounded up to page size:

```text
mem_I ::=
  | '(' 'memory' id^? : id^? at^? : addrtype^? '(' 'data' b* : datastring ')' ')'
        ≡  '(' 'memory' id' : id at^? : addrtype^? n : u64 n : u64 ')'
            '(' 'data' '(' 'memory' id' : id ')' '(' at' : addrtype '.const' '0' ')' datastring ')'
        (if id^? = id' || id^? = ε && id' ∉ I.MEMS
         && at^? = at' || at^? = ε && at' = I32
         && n = ceil(|b*| / 64 * Ki))
```

Memories can be defined as imports or exports inline:

```text
import_I ::=
  | ...
  | '(' 'memory' id^? '(' 'import' name^2 ')' memtype_I ')'
        ≡  '(' 'import' name^2 '(' 'memory' id^? memtype_I ')' ')'

export_I ::=
  | '(' 'memory' id^? : id^? '(' 'export' name ')' ... ')'
        ≡  '(' 'memory' id' : id '...' ')'
            '(' 'export' name '(' 'memory' id ')' ')'
        (if id^? = id' || id^? = ε && id' ∉ I.MEMS)
```

> **Note:** The latter abbreviation can be applied repeatedly, if "`...`" contains additional export clauses. Consequently, a memory declaration can contain any number of exports, possibly followed by an import.

### Tables

Table definitions can bind a symbolic table identifier.

```text
table_I ::=
  | '(' 'table' id^? : id^? tt : tabletype_I e : expr_I ')'  =>  (table tt e, { TABLES (id^?) })
```

#### Abbreviations

A table's initialization expression can be omitted, in which case it defaults to `ref.null`:

```text
table_I ::=
  | ...
  | '(' 'table' id^? tt : tabletype_I ')'
        ≡  '(' 'table' id^? tt : tabletype_I '(' 'ref.null' ht : heaptype_I ')' ')'
        (if tt = at lim (REF NULL^? ht))
```

An element segment can be given inline with a table definition, in which case its offset is `0` and the limits of the table type are inferred from the length of the given segment:

```text
table_I ::=
  | '(' 'table' id^? : id^? at^? : addrtype^? reftype_I '(' 'elem' (rt, e*) : elemlist_I ')' ')'
        ≡  '(' 'table' id' : id at^? : addrtype^? n : u64 n : u64 reftype_I ')'
            '(' 'elem' '(' 'table' id' : id ')' '(' at' : addrtype '.const' '0' ')' elemlist_I ')'
        (if id^? = id' || id^? = ε && id' ∉ I.TABLES
         && at^? = at' || at^? = ε && at' = I32
         && n = |e*|)
```

Tables can be defined as imports or exports inline:

```text
import_I ::=
  | ...
  | '(' 'table' id^? '(' 'import' name^2 ')' tabletype_I ')'
        ≡  '(' 'import' name^2 '(' 'table' id^? tabletype_I ')' ')'

export_I ::=
  | '(' 'table' id^? : id^? '(' 'export' name ')' ... ')'
        ≡  '(' 'table' id' : id '...' ')'
            '(' 'export' name '(' 'table' id ')' ')'
        (if id^? = id' || id^? = ε && id' ∉ I.TABLES)
```

> **Note:** The latter abbreviation can be applied repeatedly, if "`...`" contains additional export clauses. Consequently, a table declaration can contain any number of exports, possibly followed by an import.

### Functions

Function definitions can bind a symbolic function identifier, and local identifiers for its parameters and locals.

```text
func_I ::=
  | '(' 'func' id^? : id^? (x, I_1) : typeuse_I ((loc*, I_2) : local_I)* e : expr_I' ')'
        =>  (func x (++ (loc*)*) e, { FUNCS (id^?) })
        (if I' = I ⊕ I_1 ⊕ (++ I_2*)
         && |- I' : ok)

local_I ::=
  | '(' 'local' id^? : id^? t : valtype_I ')'  =>  (local t, { LOCALS (id^?) })
```

> **Note:** The well-formedness condition on `I'` ensures that parameters and locals do not contain duplicate identifiers.

#### Abbreviations

Multiple anonymous locals may be combined into a single declaration:

```text
local_I ::=
  | ... | '(' 'local' valtype_I* ')'  ≡  ('(' 'local' valtype_I ')')^*
```

Functions can be defined as imports or exports inline:

```text
import_I ::=
  | ...
  | '(' 'func' id^? '(' 'import' name^2 ')' typeuse_I ')'
        ≡  '(' 'import' name^2 '(' 'func' id^? typeuse_I ')' ')'

export_I ::=
  | '(' 'func' id^? : id^? '(' 'export' name ')' ... ')'
        ≡  '(' 'func' id' : id '...' ')'
            '(' 'export' name '(' 'func' id ')' ')'
        (if id^? = id' || id^? = ε && id' ∉ I.FUNCS)
```

> **Note:** The latter abbreviation can be applied repeatedly, if "`...`" contains additional export clauses. Consequently, a function declaration can contain any number of exports, possibly followed by an import.

### Data Segments

Data segments allow for an optional memory index to identify the memory to initialize.
The data is written as a string, which may be split up into a possibly empty sequence of individual string literals.

```text
data_I ::=
  | '(' 'data' id^? : id^? b* : datastring ')'  =>  (data b* passive, { DATAS (id^?) })
  | '(' 'data' id^? : id^? x : memuse_I e : offsetexpr_I b* : datastring ')'
        =>  (data b* (active x e), { DATAS (id^?) })

datastring ::=
  | b** : string*  =>  ++ b**

memuse_I ::=
  | '(' 'memory' x : memidx_I ')'  =>  x

offsetexpr_I ::=
  | '(' 'offset' e : expr_I ')'  =>  e
```

#### Abbreviations

As an abbreviation, a single folded instruction may occur in place of the offset of an active segment:

```text
offsetexpr_I ::=
  | ... | foldedinstr_I  ≡  '(' 'offset' foldedinstr_I ')'
```

Also, a memory use can be omitted, defaulting to `0`.

```text
memuse_I ::=
  | ... | ε  ≡  '(' 'memory' '0' ')'
```

As another abbreviation, data segments may also be specified inline with memory definitions; see the respective section.

### Element Segments

Element segments allow for an optional table index to identify the table to initialize.

```text
elem_I ::=
  | '(' 'elem' id^? : id^? (rt, e*) : elemlist_I ')'  =>  (elem rt e* passive, { ELEMS (id^?) })
  | '(' 'elem' id^? : id^? x : tableuse_I e' : offsetexpr_I (rt, e*) : elemlist_I ')'
        =>  (elem rt e* (active x e'), { ELEMS (id^?) })
  | '(' 'elem' id^? : id^? 'declare' (rt, e*) : elemlist_I ')'  =>  (elem rt e* declarative, { ELEMS (id^?) })

elemlist_I ::=
  | rt : reftype_I e* : list(elemexpr_I)  =>  (rt, e*)

elemexpr_I ::=
  | '(' 'item' e : expr_I ')'  =>  e

tableuse_I ::=
  | '(' 'table' x : tableidx_I ')'  =>  x
```

#### Abbreviations

As an abbreviation, a single folded instruction may occur in place of the offset of an active element segment or as an element expression:

```text
elemexpr_I ::=
  | ... | foldedinstr_I  ≡  '(' 'item' foldedinstr_I ')'
```

Also, the element list may be written as just a sequence of function indices:

```text
elemlist_I ::=
  | ... | 'func' x* : funcidx_I*  ≡  '(' 'ref' 'func' ')' ('(' 'ref.func' funcidx_I ')')^*
```

A table use can be omitted, defaulting to `0`.

```text
tableuse_I ::=
  | ... | ε  ≡  '(' 'table' '0' ')'
```

Furthermore, for backwards compatibility with earlier versions of WebAssembly, if the table use is omitted, the `'func'` keyword can be omitted as well.

```text
elem_I ::=
  | ...
  | '(' 'elem' offsetexpr_I list(funcidx_I) ')'
        ≡  '(' 'elem' offsetexpr_I 'func' list(funcidx_I) ')'
```

As yet another abbreviation, element segments may also be specified inline with table definitions; see the respective section.

### Start Function

A start function is defined in terms of its index.

```text
start_I ::=
  | '(' 'start' x : funcidx_I ')'  =>  (start x, { })
```

> **Note:** At most one start function may occur in a module, which is ensured by a suitable side condition on the module grammar.

### Imports

The external type in imports can bind a symbolic tag, global, memory, or function identifier.

```text
import_I ::=
  | '(' 'import' nm_1 : name nm_2 : name (xt, I') : externtype_I ')'  =>  (import nm_1 nm_2 xt, I')
```

#### Abbreviations

As an abbreviation, imports may also be specified inline with
tag,
global,
memory,
table, or
function
definitions; see the respective sections.

### Exports

The syntax for exports mirrors their abstract syntax directly.

```text
export_I ::=
  | '(' 'export' nm : name xx : externidx_I ')'  =>  (export nm xx, { })

externidx_I ::=
  | '(' 'tag' x : tagidx_I ')'  =>  tag x
  | '(' 'global' x : globalidx_I ')'  =>  global x
  | '(' 'memory' x : memidx_I ')'  =>  mem x
  | '(' 'table' x : tableidx_I ')'  =>  table x
  | '(' 'func' x : funcidx_I ')'  =>  func x
```

#### Abbreviations

As an abbreviation, exports may also be specified inline with
tag,
global,
memory,
table, or
function
definitions; see the respective sections.

### Modules

A module consists of a sequence of *declarations* that can occur in any order.

```text
decl ::= type | import | tag | global | mem | table | func | data | elem | start | export
```

All declarations and their respective bound identifiers scope over the entire module, including the text preceding them.

A module itself may optionally bind an identifier that names the module.

The name serves a documentary role only.

> **Note:** Tools may include the module name in the name section of the binary format.

```text
decl_I ::=
  | type_I | import_I | tag_I | global_I | mem_I | table_I
  | func_I | data_I | elem_I | start_I | export_I
```

```text
module ::=
  | '(' 'module' id^? (decl, I)* : decl_I'* ')'
        =>  module type* import* tag* global* mem* table* func* data* elem* start^? export*
        (if I' = ++ I*
         && |- I' : ok
         && type* = typesd(decl*)
         && import* = importsd(decl*)
         && tag* = tagsd(decl*)
         && global* = globalsd(decl*)
         && mem* = memsd(decl*)
         && table* = tablesd(decl*)
         && func* = funcsd(decl*)
         && data* = datasd(decl*)
         && elem* = elemsd(decl*)
         && start^? = startsd(decl*)
         && export* = exportsd(decl*)
         && ordered(decl*))
```

where `types(decl*)`, `imports(decl*)`, `tags(decl*)`, etc., extract the sequence of types, imports, tags, etc., contained in `decl*`, respectively.

The auxiliary predicate `ordered` checks that no imports occur after the first definition of a tag, global, memory, table, or function in a sequence of declarations:

```text
ordered(decl*) = true   (if importsd(decl*) = ε)
ordered(decl1* import decl2*) =
    importsd(decl1*) = ε && tagsd(decl1*) = ε && globalsd(decl1*) = ε
    && memsd(decl1*) = ε && tablesd(decl1*) = ε && funcsd(decl1*) = ε
```

#### Abbreviations

In a source file, the top-level `('module' ... ')'` surrounding the module body may be omitted.

```text
module ::=
  | ... | decl_I*  ≡  '(' 'module' decl_I* ')'
```
