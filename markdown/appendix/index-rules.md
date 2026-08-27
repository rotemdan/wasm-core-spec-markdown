## Index of Semantic Rules

### Well-formedness of Types

| Construct | Judgement |
|-----------|-----------|
| [Numeric type](valid-numtype) | `C |-numtype numtype : OKnumtype` |
| [Vector type](valid-vectype) | `C |-vectype vectype : OKvectype` |
| [Heap type](valid-heaptype) | `C |-heaptype heaptype : OKheaptype` |
| [Reference type](valid-reftype) | `C |-reftype reftype : OKreftype` |
| [Value type](valid-valtype) | `C |-valtype valtype : OKvaltype` |
| [Packed type](valid-packtype) | `C |-packtype packtype : OKpacktype` |
| [Storage type](valid-storagetype) | `C |-storagetype storagetype : OKstoragetype` |
| [Field type](valid-fieldtype) | `C |-fieldtype fieldtype : OKfieldtype` |
| [Result type](valid-resulttype) | `C |-resulttype resulttype : OKresulttype` |
| [Instruction type](valid-instrtype) | `C |-instrtype instrtype : OKinstrtype` |
| [Composite type](valid-comptype) | `C |-comptype comptype : OKcomptype` |
| [Sub type](valid-subtype) | `C |-subtype subtype : OKsubtype` |
| [Recursive type](valid-rectype) | `C |-rectype rectype : OKrectype` |
| [Defined type](valid-deftype) | `C |-deftype deftype : OKdeftype` |
| [Block type](valid-blocktype) | `C |-blocktype blocktype : instrtype` |
| [Tag type](valid-tagtype) | `C |-tagtype tagtype : OKtagtype` |
| [Global type](valid-globaltype) | `C |-globaltype globaltype : OKglobaltype` |
| [Memory type](valid-memtype) | `C |-memtype memtype : OKmemtype` |
| [Table type](valid-tabletype) | `C |-tabletype tabletype : OKtabletype` |
| [External type](valid-externtype) | `C |-externtype externtype : OKexterntype` |
| [Type definitions](valid-type) | `C |-types type* : OKtypes` |

### Typing of Static Constructs

| Construct | Judgement |
|-----------|-----------|
| [Instruction](valid-instr) | `S;C |-instr instr : instrtype` |
| [Instruction sequence](valid-instrs) | `S;C |-instrs instr* : instrtype` |
| [Catch clause](valid-catch) | `C |-catch catch : OKcatch` |
| [Expression](valid-expr) | `C |-expr expr : resulttype` |
| [Limits](valid-limits) | `C |-limits limits : k` |
| [Tag](valid-tag) | `C |-tag tag : tagtype` |
| [Global](valid-global) | `C |-global global : globaltype` |
| [Memory](valid-mem) | `C |-mem mem : memtype` |
| [Table](valid-table) | `C |-table table : tabletype` |
| [Function](valid-func) | `C |-func func : deftype` |
| [Local](valid-local) | `C |-local local : localtype` |
| [Element segment](valid-elem) | `C |-elem elem : reftype` |
| [Element mode](valid-elemmode) | `C |-elemmode elemmode : reftype` |
| [Data segment](valid-data) | `C |-data data : OKdata` |
| [Data mode](valid-datamode) | `C |-datamode datamode : OKdatamode` |
| [Start function](valid-start) | `C |-start start : OKstart` |
| [Import](valid-import) | `C |-import import : externtype` |
| [Export](valid-export) | `C |-export export : externtype` |
| [Module](valid-module) | `|-module module : externtype* -> externtype*` |

### Typing of Runtime Constructs

| Construct | Judgement |
|-----------|-----------|
| [Value](valid-val) | `S |-val val : valtype` |
| [Result](valid-result) | `S |-result result : resulttype` |
| [Packed value](valid-packval) | `S |-packval packval : packtype` |
| [Field value](valid-fieldval) | `S |-fieldval fieldval : storagetype` |
| [External address](valid-externaddr) | `S |-externaddr externaddr : externtype` |
| [Tag instance](valid-taginst) | `S |-taginst taginst : tagtype` |
| [Global instance](valid-globalinst) | `S |-globalinst globalinst : globaltype` |
| [Memory instance](valid-meminst) | `S |-meminst meminst : memtype` |
| [Table instance](valid-tableinst) | `S |-tableinst tableinst : tabletype` |
| [Function instance](valid-funcinst) | `S |-funcinst funcinst : deftype` |
| [Data instance](valid-datainst) | `S |-datainst datainst : OKdatainst` |
| [Element instance](valid-eleminst) | `S |-eleminst eleminst : t` |
| [Structure instance](valid-structinst) | `S |-structinst structinst : OKstructinst` |
| [Array instance](valid-arrayinst) | `S |-arrayinst arrayinst : OKarrayinst` |
| [Export instance](valid-exportinst) | `S |-exportinst exportinst : OKexportinst` |
| [Module instance](valid-moduleinst) | `S |-moduleinst moduleinst : C` |
| [Store](valid-store) | `|-store store : OKstore` |
| [Configuration](valid-config) | `|-config config : [t*]` |
| [Thread](valid-thread) | `S;resulttype? |-thread thread : resulttype` |
| [Frame](valid-frame) | `S |-frame frame : C` |

### Constantness

| Construct | Judgement |
|-----------|-----------|
| [Constant expression](valid-constant) | `C |-exprconst expr CONSTexprconst` |
| [Constant instruction](valid-constant) | `C |-instrconst instr CONSTinstrconst` |

### Matching

| Construct | Judgement |
|-----------|-----------|
| [Number type](match-numtype) | `C |-numtypematch numtype1 <: numtype2` |
| [Vector type](match-vectype) | `C |-vectypematch vectype1 <: vectype2` |
| [Heap type](match-heaptype) | `C |-heaptypematch heaptype1 <: heaptype2` |
| [Reference type](match-reftype) | `C |-reftypematch reftype1 <: reftype2` |
| [Value type](match-valtype) | `C |-valtypematch valtype1 <: valtype2` |
| [Packed type](match-packtype) | `C |-packtypematch packtype1 <: packtype2` |
| [Storage type](match-storagetype) | `C |-storagetypematch storagetype1 <: storagetype2` |
| [Field type](match-fieldtype) | `C |-fieldtypematch fieldtype1 <: fieldtype2` |
| [Result type](match-resulttype) | `C |-resulttypematch resulttype1 <: resulttype2` |
| [Instruction type](match-instrtype) | `C |-instrtypematch instrtype1 <: instrtype2` |
| [Composite type](match-comptype) | `C |-comptypematch comptype1 <: comptype2` |
| [Defined type](match-deftype) | `C |-deftypematch deftype1 <: deftype2` |
| [Limits](match-limits) | `C |-limitsmatch limits1 <: limits2` |
| [Tag type](match-tagtype) | `C |-tagtypematch tagtype1 <: tagtype2` |
| [Global type](match-globaltype) | `C |-globaltypematch globaltype1 <: globaltype2` |
| [Memory type](match-memtype) | `C |-memtypematch memtype1 <: memtype2` |
| [Table type](match-tabletype) | `C |-tabletypematch tabletype1 <: tabletype2` |
| [External type](match-externtype) | `C |-externtypematch externtype1 <: externtype2` |

### Store Extension

| Construct | Judgement |
|-----------|-----------|
| [Tag instance](extend-taginst) | `|-taginstextends taginst1 extends taginst2` |
| [Global instance](extend-globalinst) | `|-globalinstextends globalinst1 extends globalinst2` |
| [Memory instance](extend-meminst) | `|-meminstextends meminst1 extends meminst2` |
| [Table instance](extend-tableinst) | `|-tableinstextends tableinst1 extends tableinst2` |
| [Function instance](extend-funcinst) | `|-funcinstextends funcinst1 extends funcinst2` |
| [Data instance](extend-datainst) | `|-datainstextends datainst1 extends datainst2` |
| [Element instance](extend-eleminst) | `|-eleminstextends eleminst1 extends eleminst2` |
| [Structure instance](extend-structinst) | `|-structinstextends structinst1 extends structinst2` |
| [Array instance](extend-arrayinst) | `|-arrayinstextends arrayinst1 extends arrayinst2` |
| [Store](extend-store) | `|-storeextends store1 extends store2` |

### Execution

| Construct | Judgement |
|-----------|-----------|
| [Instruction](exec-instr) | `S;F;instr* stepto S';F';instr'*` |
| [Expression](exec-expr) | `S;F;expr stepto S';F';expr'` |
