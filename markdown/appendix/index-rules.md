## Index of Semantic Rules

### Well-formedness of Types

| Construct | Judgement |
| - | - |
| [Numeric type](valid-numtype) | `C \|- numtype : OK` |
| [Vector type](valid-vectype) | `C \|- vectype : OK` |
| [Heap type](valid-heaptype) | `C \|- heaptype : OK` |
| [Reference type](valid-reftype) | `C \|- reftype : OK` |
| [Value type](valid-valtype) | `C \|- valtype : OK` |
| [Packed type](valid-packtype) | `C \|- packtype : OK` |
| [Storage type](valid-storagetype) | `C \|- storagetype : OK` |
| [Field type](valid-fieldtype) | `C \|- fieldtype : OK` |
| [Result type](valid-resulttype) | `C \|- resulttype : OK` |
| [Instruction type](valid-instrtype) | `C \|- instrtype : OK` |
| [Composite type](valid-comptype) | `C \|- comptype : OK` |
| [Sub type](valid-subtype) | `C \|- subtype : OK` |
| [Recursive type](valid-rectype) | `C \|- rectype : OK` |
| [Defined type](valid-deftype) | `C \|- deftype : OK` |
| [Block type](valid-blocktype) | `C \|- blocktype : instrtype` |
| [Tag type](valid-tagtype) | `C \|- tagtype : OK` |
| [Global type](valid-globaltype) | `C \|- globaltype : OK` |
| [Memory type](valid-memtype) | `C \|- memtype : OK` |
| [Table type](valid-tabletype) | `C \|- tabletype : OK` |
| [External type](valid-externtype) | `C \|- externtype : OK` |
| [Type definitions](valid-type) | `C \|- type* : OK` |

### Typing of Static Constructs

| Construct | Judgement |
| - | - |
| [Instruction](valid-instr) | `S;C \|- instr : instrtype` |
| [Instruction sequence](valid-instrs) | `S;C \|- instr* : instrtype` |
| [Catch clause](valid-catch) | `C \|- catch : OK` |
| [Expression](valid-expr) | `C \|- expr : resulttype` |
| [Limits](valid-limits) | `C \|- limits : k` |
| [Tag](valid-tag) | `C \|- tag : tagtype` |
| [Global](valid-global) | `C \|- global : globaltype` |
| [Memory](valid-mem) | `C \|- mem : memtype` |
| [Table](valid-table) | `C \|- table : tabletype` |
| [Function](valid-func) | `C \|- func : deftype` |
| [Local](valid-local) | `C \|- local : localtype` |
| [Element segment](valid-elem) | `C \|- elem : reftype` |
| [Element mode](valid-elemmode) | `C \|- elemmode : reftype` |
| [Data segment](valid-data) | `C \|- data : OK` |
| [Data mode](valid-datamode) | `C \|- datamode : OK` |
| [Start function](valid-start) | `C \|- start : OK` |
| [Import](valid-import) | `C \|- import : externtype` |
| [Export](valid-export) | `C \|- export : externtype` |
| [Module](valid-module) | `\|- module : externtype* -> externtype*` |

### Typing of Runtime Constructs

| Construct | Judgement |
| - | - |
| [Value](valid-val) | `S \|- val : valtype` |
| [Result](valid-result) | `S \|- result : resulttype` |
| [Packed value](valid-packval) | `S \|- packval : packtype` |
| [Field value](valid-fieldval) | `S \|- fieldval : storagetype` |
| [External address](valid-externaddr) | `S \|- externaddr : externtype` |
| [Tag instance](valid-taginst) | `S \|- taginst : tagtype` |
| [Global instance](valid-globalinst) | `S \|- globalinst : globaltype` |
| [Memory instance](valid-meminst) | `S \|- meminst : memtype` |
| [Table instance](valid-tableinst) | `S \|- tableinst : tabletype` |
| [Function instance](valid-funcinst) | `S \|- funcinst : deftype` |
| [Data instance](valid-datainst) | `S \|- datainst : OK` |
| [Element instance](valid-eleminst) | `S \|- eleminst : t` |
| [Structure instance](valid-structinst) | `S \|- structinst : OK` |
| [Array instance](valid-arrayinst) | `S \|- arrayinst : OK` |
| [Export instance](valid-exportinst) | `S \|- exportinst : OK` |
| [Module instance](valid-moduleinst) | `S \|- moduleinst : C` |
| [Store](valid-store) | `\|- store : OK` |
| [Configuration](valid-config) | `\|- config : [t*]` |
| [Thread](valid-thread) | `S;resulttype? \|- thread : resulttype` |
| [Frame](valid-frame) | `S \|- frame : C` |

### Constantness

| Construct | Judgement |
| - | - |
| [Constant expression](valid-constant) | `C \|- expr const` |
| [Constant instruction](valid-constant) | `C \|- instr const` |

### Matching

| Construct | Judgement |
| - | - |
| [Number type](match-numtype) | `C \|- numtype1 <: numtype2` |
| [Vector type](match-vectype) | `C \|- vectype1 <: vectype2` |
| [Heap type](match-heaptype) | `C \|- heaptype1 <: heaptype2` |
| [Reference type](match-reftype) | `C \|- reftype1 <: reftype2` |
| [Value type](match-valtype) | `C \|- valtype1 <: valtype2` |
| [Packed type](match-packtype) | `C \|- packtype1 <: packtype2` |
| [Storage type](match-storagetype) | `C \|- storagetype1 <: storagetype2` |
| [Field type](match-fieldtype) | `C \|- fieldtype1 <: fieldtype2` |
| [Result type](match-resulttype) | `C \|- resulttype1 <: resulttype2` |
| [Instruction type](match-instrtype) | `C \|- instrtype1 <: instrtype2` |
| [Composite type](match-comptype) | `C \|- comptype1 <: comptype2` |
| [Defined type](match-deftype) | `C \|- deftype1 <: deftype2` |
| [Limits](match-limits) | `C \|- limits1 <: limits2` |
| [Tag type](match-tagtype) | `C \|- tagtype1 <: tagtype2` |
| [Global type](match-globaltype) | `C \|- globaltype1 <: globaltype2` |
| [Memory type](match-memtype) | `C \|- memtype1 <: memtype2` |
| [Table type](match-tabletype) | `C \|- tabletype1 <: tabletype2` |
| [External type](match-externtype) | `C \|- externtype1 <: externtype2` |

### Store Extension

| Construct | Judgement |
| - | - |
| [Tag instance](extend-taginst) | `\|- taginst1 extends taginst2` |
| [Global instance](extend-globalinst) | `\|- globalinst1 extends globalinst2` |
| [Memory instance](extend-meminst) | `\|- meminst1 extends meminst2` |
| [Table instance](extend-tableinst) | `\|- tableinst1 extends tableinst2` |
| [Function instance](extend-funcinst) | `\|- funcinst1 extends funcinst2` |
| [Data instance](extend-datainst) | `\|- datainst1 extends datainst2` |
| [Element instance](extend-eleminst) | `\|- eleminst1 extends eleminst2` |
| [Structure instance](extend-structinst) | `\|- structinst1 extends structinst2` |
| [Array instance](extend-arrayinst) | `\|- arrayinst1 extends arrayinst2` |
| [Store](extend-store) | `\|- store1 extends store2` |

### Execution

| Construct | Judgement |
| - | - |
| [Instruction](exec-instr) | `S;F;instr* -> S';F';instr'*` |
| [Expression](exec-expr) | `S;F;expr -> S';F';expr'` |
