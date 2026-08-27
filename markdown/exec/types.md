## Types

Execution has to check and compare types in a few places, such as executing `call_indirect` or instantiating modules.

It is an invariant of the semantics that all types occurring during execution are closed.

> **Note:** Runtime type checks generally involve types from multiple modules or types not defined by a module at all, such that any module-local type indices occurring inside them would not generally be meaningful.

### Instantiation

Any form of type can be *instantiated* into a closed type inside a module instance by substituting each type index `x` occurring in it with the corresponding defined type `moduleinst.ITYPES[x]`.

```text
insttype_moduleinst(t) = t [ assignsubst moduleinst.ITYPES ]
```

> **Note:** This is the runtime equivalent to type closure, which is applied at validation time.
