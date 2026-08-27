## Types

Various entities in WebAssembly are classified by types. Types are checked during validation, instantiation, and possibly execution.

### Number Types

*Number types* classify numeric values.

```text
numtype ::= i32 | i64 | f32 | f64
```

The types `i32` and `i64` classify 32 and 64 bit integers, respectively. Integers are not inherently signed or unsigned, their interpretation is determined by individual operations.

The types `f32` and `f64` classify 32 and 64 bit floating-point data, respectively. They correspond to the respective binary floating-point representations, also known as *single* and *double* precision, as defined by the IEEE 754 standard (Section 3.3).

Number types are *transparent*, meaning that their bit patterns can be observed. Values of number type can be stored in memories.

#### Conventions

* The notation `|t|` denotes the *bit width* of a number type `t`. That is, `|i32| = |f32| = 32` and `|i64| = |f64| = 64`.

### Vector Types

*Vector types* classify vectors of numeric values processed by vector instructions (also known as *SIMD* instructions, single instruction multiple data).

```text
vectype ::= v128
```

The type `v128` corresponds to a 128 bit vector of packed integer or floating-point data. The packed data can be interpreted as signed or unsigned integers, single or double precision floating-point values, or a single 128 bit type. The interpretation is determined by individual operations.

Vector types, like number types are *transparent*, meaning that their bit patterns can be observed. Values of vector type can be stored in memories.

#### Conventions

* The notation `|t|` for bit width extends to vector types as well, that is, `|v128| = 128`.

### Type Uses

A *type use* is the use site of a type index referencing a composite type defined in a module. It classifies objects of the respective type.

```text
typeuse ::= typeidx | ...
```

The syntax of type uses is extended with additional forms for the purpose of specifying validation and execution.

### Heap Types

*Heap types* classify objects in the runtime store. There are three disjoint hierarchies of heap types:

* *function types* classify functions,
* *aggregate types* classify dynamically allocated *managed* data, such as *structures*, *arrays*, or *unboxed scalars*,
* *external types* classify *external* references possibly owned by the embedder.

The values from the latter two hierarchies are interconvertible by ways of the `extern.convert_any` and `any.convert_extern` instructions. That is, both type hierarchies are inhabited by an isomorphic set of values, but may have different, incompatible representations in practice.

```text
absheaptype ::= any | eq | i31 | struct | array | none
              | func | nofunc
              | exn | noexn
              | extern | noextern
              | ...

heaptype ::= absheaptype | typeuse
```

A heap type is either *abstract* or *concrete*. A concrete heap type consists of a type use that classifies an object of the respective type defined in a module. Abstract types are denoted by individual keywords.

The type `func` denotes the common supertype of all function types, regardless of their concrete definition. Dually, the type `nofunc` denotes the common subtype of all function types, regardless of their concrete definition. This type has no values.

The type `exn` denotes the common supertype of all exception references. This type has no concrete subtypes. Dually, the type `noexn` denotes the common subtype of all forms of exception references. This type has no values.

The type `extern` denotes the common supertype of all external references received through the embedder. This type has no concrete subtypes. Dually, the type `noextern` denotes the common subtype of all forms of external references. This type has no values.

The type `any` denotes the common supertype of all aggregate types, as well as possibly abstract values produced by *internalizing* an external reference of type `extern`. Dually, the type `none` denotes the common subtype of all forms of aggregate types. This type has no values.

The type `eq` is a subtype of `any` that includes all types for which references can be compared, i.e., aggregate values and `i31`.

The types `struct` and `array` denote the common supertypes of all structure and array aggregates, respectively.

The type `i31` denotes *unboxed scalars*, that is, integers injected into references. Their observable value range is limited to 31 bits.

> **Note:** Values of type `i31` are not actually allocated in the store, but represented in a way that allows them to be mixed with actual references into the store without ambiguity. Engines need to perform some form of *pointer tagging* to achieve this, which is why one bit is reserved. Since this type is to be reliably unboxed on all hardware platforms supported by WebAssembly, it cannot be wider than 32 bits minus the tag bit.
>
> Although the types `none`, `nofunc`, `noexn`, and `noextern` are not inhabited by any values, they can be used to form the types of all null references in their respective hierarchy. For example, `(ref null nofunc)` is the generic type of a null reference compatible with all function reference types.

The syntax of abstract heap types is extended with additional forms for the purpose of specifying validation and execution.

### Reference Types

*Reference types* classify values that are first-class references to objects in the runtime store.

```text
reftype ::= ref null^? heaptype
```

A reference type is characterised by the heap type it points to.

In addition, a reference type of the form `ref null ht` is *nullable*, meaning that it can either be a proper reference to `ht` or null. Other references are *non-null*.

Reference types are *opaque*, meaning that neither their size nor their bit pattern can be observed. Values of reference type can be stored in tables but not in memories.

#### Conventions

* The reference type `anyref` is an abbreviation for `(ref null any)`.
* The reference type `eqref` is an abbreviation for `(ref null eq)`.
* The reference type `i31ref` is an abbreviation for `(ref null i31)`.
* The reference type `structref` is an abbreviation for `(ref null struct)`.
* The reference type `arrayref` is an abbreviation for `(ref null array)`.
* The reference type `funcref` is an abbreviation for `(ref null func)`.
* The reference type `exnref` is an abbreviation for `(ref null exn)`.
* The reference type `externref` is an abbreviation for `(ref null extern)`.
* The reference type `nullref` is an abbreviation for `(ref null none)`.
* The reference type `nullfuncref` is an abbreviation for `(ref null nofunc)`.
* The reference type `nullexnref` is an abbreviation for `(ref null noexn)`.
* The reference type `nullexternref` is an abbreviation for `(ref null noextern)`.

### Value Types

*Value types* classify the individual values that WebAssembly code can compute with and the values that a variable accepts. They are either number types, vector types, or reference types.

```text
consttype ::= numtype | vectype

valtype ::= numtype | vectype | reftype | ...
```

The syntax of value types is extended with additional forms for the purpose of specifying validation.

#### Conventions

* The meta variable `t` ranges over value types or subclasses thereof where clear from context.

### Result Types

*Result types* classify the result of executing instructions or functions, which is a sequence of values, written with brackets.

```text
resulttype ::= list(valtype)
```

### Block Types

*Block types* classify the *input* and *output* of structured control instructions delimiting blocks of instructions.

```text
blocktype ::= valtype?
            | typeidx
```

They are given either as a type index that refers to a suitable function type reinterpreted as an instruction type, or as an optional value type inline, which is a shorthand for the instruction type `ε -> valtype?`.

### Composite Types

*Composite types* are all types composed from simpler types, including *function types*, *structure types* and *array types*.

```text
comptype ::= struct list(fieldtype)
           | array fieldtype
           | func resulttype -> resulttype

fieldtype ::= mut^? storagetype

storagetype ::= valtype | packtype

packtype ::= i8 | i16
```

Function types classify the signature of functions, mapping a list of parameters to a list of results. They are also used to classify the inputs and outputs of instructions.

*Aggregate types* like structure or array types consist of a list of possibly mutable, possibly packed *field types* describing their components. Structures are heterogeneous, but require static indexing, while arrays need to be homogeneous, but allow dynamic indexing.

#### Conventions

* The notation `|t|` for the bit width of a value type `t` extends to packed types as well, that is, `|i8| = 8` and `|i16| = 16`.

* The auxiliary function `unpack` maps a storage type to the value type obtained when accessing a field:

```text
unpack(valtype) = valtype
unpack(packtype) = i32
```

### Recursive Types

*Recursive types* denote a group of mutually recursive composite types, each of which can optionally declare a list of type uses of supertypes that it matches. Each type can also be declared *final*, preventing further subtyping.

```text
rectype ::= rec list(subtype)

subtype ::= sub final^? typeuse* comptype
```

In a module, each member of a recursive type is assigned a separate type index.

### Address Types

*Address types* are a subset of number types that classify the values that can be used as offsets into memories and tables.

```text
addrtype ::= i32 | i64
```

#### Conventions

The *minimum* of two address types is defined as the address type whose bit width is the minimum of the two.

```text
addrtypemin(at1, at2) = at1   (if |at1| <= |at2|)
addrtypemin(at1, at2) = at2   (otherwise)
```

### Limits

*Limits* classify the size range of resizeable storage associated with memory types and table types.

```text
limits ::= [ u64 .. u64? ]
```

If no maximum is present, then the respective storage can grow to any valid size.

### Tag Types

*Tag types* classify the signature tags with a type use referring to the definition of a function type that declares the types of parameter and result values associated with the tag. The result type is empty for exception tags.

```text
tagtype ::= typeuse
```

### Global Types

*Global types* classify global variables, which hold a value and can either be mutable or immutable.

```text
globaltype ::= mut^? valtype
```

### Memory Types

*Memory types* classify linear memories and their size range.

```text
memtype ::= addrtype limits page
```

The limits constrain the minimum and optionally the maximum size of a memory. The limits are given in units of page size.

### Table Types

*Table types* classify tables over elements of reference type within a size range.

```text
tabletype ::= addrtype limits reftype
```

Like memories, tables are constrained by limits for their minimum and optionally maximum size. The limits are given in numbers of entries.

### Data Types

*Data types* classify data segments. Since the contents of a data segment requires no further classification, they merely consist of a universal marker `ok` indicating well-formedness.

```text
datatype ::= ok
```

### Element Types

*Element types* classify element segments by the reference type of its elements.

```text
elemtype ::= reftype
```

### External Types

*External types* classify imports and external addresses with their respective types.

```text
externtype ::= tag tagtype | global globaltype | mem memtype | table tabletype | func typeuse
```

For functions, the type use has to refer to the definition of a function type.

> **Note:** Future versions of WebAssembly may have additional uses for tags, and may allow non-empty result types in the function types of tags.

#### Conventions

The following auxiliary notation is defined for sequences of external types. It filters out entries of a specific kind in an order-preserving fashion:

```text
funcsxt(ε) = ε
funcsxt((func dt) xt*) = dt funcsxt(xt*)
funcsxt(externtype xt*) = funcsxt(xt*)   (otherwise)

tablesxt(ε) = ε
tablesxt((table tt) xt*) = tt tablesxt(xt*)
tablesxt(externtype xt*) = tablesxt(xt*)   (otherwise)

memsxt(ε) = ε
memsxt((mem mt) xt*) = mt memsxt(xt*)
memsxt(externtype xt*) = memsxt(xt*)   (otherwise)

globalsxt(ε) = ε
globalsxt((global gt) xt*) = gt globalsxt(xt*)
globalsxt(externtype xt*) = globalsxt(xt*)   (otherwise)

tagsxt(ε) = ε
tagsxt((tag jt) xt*) = jt tagsxt(xt*)
tagsxt(externtype xt*) = tagsxt(xt*)   (otherwise)
```
