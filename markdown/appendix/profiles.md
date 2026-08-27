## Profiles

To enable the use of WebAssembly in as many environments as possible, *profiles* specify coherent language subsets that fit constraints imposed by common classes of host environments.

A host platform can thereby decide to support the language only under a restricted profile, or even the intersection of multiple profiles.

### Conventions

A profile modification is specified by decorating selected rules in the main body of this specification with a *profile annotation* that defines them as conditional on the choice of profile.

For that purpose, every profile defines a *profile marker*, an alphanumeric short-hand like `ABC`.

A profile annotation of the form `exprofiles ABC XYZ` on a rule indicates that this rule is *excluded* for either of the profiles whose marker is `ABC` or `XYZ`.

There are two ways of subsetting the language in a profile:

* *Syntactic*, by *omitting* a feature, in which case certain constructs are removed from the syntax altogether.
* *Semantic*, by *restricting* a feature, in which case certain constructs are still present but some behaviours are ruled out.

### Syntax Annotations

To omit a construct from a profile syntactically, respective productions in the grammar of the [abstract syntax](syntax) are annotated with an associated profile marker.

This is defined to have the following implications:

1. Any production in the [binary](binary) or [textual](text) syntax that produces abstract syntax with a marked construct is omitted by extension.
2. Any [validation](valid) or [execution](exec) rule that handles a marked construct is omitted by extension.

The overall effect is that the respective construct is no longer part of the language under a respective profile.

> **Note:** For example, a "busy" profile marked `BUSY` could rule out the NOP instruction by marking the production for it in the abstract syntax as follows:
>
> ```text
> instr ::=
>   | ...
>   | nop    (exprofiles BUSY)
>   | unreachable
> ```
>
> A rule may be annotated by multiple markers, which could be the case if a construct is in the intersection of multiple features.

### Semantics Annotations

To restrict certain behaviours in a profile, individual [validation](valid) or [reduction](exec) rules or auxiliary definitions are annotated with an associated marker.

This has the consequence that the respective rule is no longer applicable under the given profile.

> **Note:** For example, an "infinite" profile marked `INF` could define that growing memory never fails:
>
> ```text
> S; F; (i32.const n) memory.grow x stepto S'; F; (i32.const sz)
>   (if F.MODULE.MIMEMS[x] = a
>    ∧ sz = |S.MEMS[a].MIDATAS|/64 Ki
>    ∧ S' = S with MEMS[a] = growmem(S.MEMS[a], n))
>
> exprofiles INF
> S; F; (i32.const n) memory.grow x stepto S; F; (i32.const signed_32^{-1}(-1))
> ```

### Properties

All profiles are defined such that the following properties are preserved:

* All profiles represent syntactic and semantic subsets of the [full profile](profile-full), i.e., they do not *add* syntax or *alter* behaviour.
* All profiles are mutually compatible, i.e., no two profiles subset semantic behaviour in inconsistent or ambiguous ways, and any intersection of profiles preserves the properties described here.
* Profiles do not violate [soundness](soundness), i.e., all [configurations](syntax-config) valid under that profile still have well-defined execution behaviour.

> **Note:** Tools are generally expected to handle and produce code for the full profile by default.
>
> In particular, producers should not generate code that *depends* on specific profiles. Instead, all code should preserve correctness when executed under the full profile.
>
> Moreover, profiles should be considered static and fixed for a given platform or ecosystem. Runtime conditioning on the "current" profile is not intended and should be avoided.

### Defined Profiles

> **Note:** The number of defined profiles is expected to remain small in the future. Profiles are intended for broad and permanent use cases only. In particular, profiles are not intended for language versioning.

### Full Profile (FUL)

The *full* profile contains the complete language and all possible behaviours.

It imposes no restrictions, i.e., all rules and definitions are active.

All other profiles define sub-languages of this profile.

### Deterministic Profile (DET)

The *deterministic* profile excludes all rules marked `exprofiles DET`.

It defines a sub-language that does not exhibit any incidental non-deterministic behaviour:

* All [NaN](syntax-nan) values [generated](aux-nans) by [floating-point instructions](syntax-instr-numeric) are canonical and positive.
* All [relaxed vector instructions](syntax-instr-vec-relaxed) have a fixed behaviour that does not depend on the implementation.

Even under this profile, the [MEMORYGROW](syntax-instr) and [TABLEGROW](syntax-instr) instructions technically remain [non-deterministic](exec-memory.grow), in order to be able to indicate resource exhaustion.

> **Note:** In future versions of WebAssembly, new non-deterministic behaviour may be added to the language, such that the deterministic profile will induce additional restrictions.
