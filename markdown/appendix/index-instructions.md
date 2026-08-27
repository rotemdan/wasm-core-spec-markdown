## Index of Instructions

| Instruction | Binary Opcode | Type | Validation | Execution |
|-------------|---------------|------|------------|-----------|
| `unreachable` | `0x00` | `[t1*] -> [t2*]` | [validation](valid-unreachable) | [execution](exec-unreachable) |
| `nop` | `0x01` | `[] -> []` | [validation](valid-nop) | [execution](exec-nop) |
| `block bt` | `0x02` | `[t1*] -> [t2*]` | [validation](valid-block) | [execution](exec-block) |
| `loop bt` | `0x03` | `[t1*] -> [t2*]` | [validation](valid-loop) | [execution](exec-loop) |
| `if bt` | `0x04` | `[t1* i32] -> [t2*]` | [validation](valid-if) | [execution](exec-if) |
| `else` | `0x05` | | | |
| (reserved) | `0x06` | | | |
| (reserved) | `0x07` | | | |
| `throw x` | `0x08` | `[t1* t_x*] -> [t2*]` | [validation](valid-throw) | [execution](exec-throw) |
| (reserved) | `0x09` | | | |
| `throw_ref` | `0x0A` | `[t1* exnref] -> [t2*]` | [validation](valid-throw_ref) | [execution](exec-throw_ref) |
| `end` | `0x0B` | | | |
| `br l` | `0x0C` | `[t1* t*] -> [t2*]` | [validation](valid-br) | [execution](exec-br) |
| `br_if l` | `0x0D` | `[t* i32] -> [t*]` | [validation](valid-br_if) | [execution](exec-br_if) |
| `br_table l* l` | `0x0E` | `[t1* t* i32] -> [t2*]` | [validation](valid-br_table) | [execution](exec-br_table) |
| `return` | `0x0F` | `[t1* t*] -> [t2*]` | [validation](valid-return) | [execution](exec-return) |
| `call x` | `0x10` | `[t1*] -> [t2*]` | [validation](valid-call) | [execution](exec-call) |
| `call_indirect x y` | `0x11` | `[t1* i32] -> [t2*]` | [validation](valid-call_indirect) | [execution](exec-call_indirect) |
| `return_call x` | `0x12` | `[t1*] -> [t2*]` | [validation](valid-return_call) | [execution](exec-return_call) |
| `return_call_indirect x y` | `0x13` | `[t1* i32] -> [t2*]` | [validation](valid-return_call_indirect) | [execution](exec-return_call_indirect) |
| `call_ref x` | `0x14` | `[t1* (ref null x)] -> [t2*]` | [validation](valid-call_ref) | [execution](exec-call_ref) |
| `return_call_ref x` | `0x15` | `[t1* (ref null x)] -> [t2*]` | [validation](valid-return_call_ref) | [execution](exec-return_call_ref) |
| (reserved) | `0x16` | | | |
| (reserved) | `0x17` | | | |
| (reserved) | `0x18` | | | |
| (reserved) | `0x19` | | | |
| `drop` | `0x1A` | `[t] -> []` | [validation](valid-drop) | [execution](exec-drop) |
| `select` | `0x1B` | `[t t i32] -> [t]` | [validation](valid-select) | [execution](exec-select) |
| `select t` | `0x1C` | `[t t i32] -> [t]` | [validation](valid-select) | [execution](exec-select) |
| (reserved) | `0x1D` | | | |
| (reserved) | `0x1E` | | | |
| `try_table bt` | `0x1F` | `[t1*] -> [t2*]` | [validation](valid-try_table) | [execution](exec-try_table) |
| `local.get x` | `0x20` | `[] -> [t]` | [validation](valid-local.get) | [execution](exec-local.get) |
| `local.set x` | `0x21` | `[t] -> []` | [validation](valid-local.set) | [execution](exec-local.set) |
| `local.tee x` | `0x22` | `[t] -> [t]` | [validation](valid-local.tee) | [execution](exec-local.tee) |
| `global.get x` | `0x23` | `[] -> [t]` | [validation](valid-global.get) | [execution](exec-global.get) |
| `global.set x` | `0x24` | `[t] -> []` | [validation](valid-global.set) | [execution](exec-global.set) |
| `table.get x` | `0x25` | `[at] -> [t]` | [validation](valid-table.get) | [execution](exec-table.get) |
| `table.set x` | `0x26` | `[at t] -> []` | [validation](valid-table.set) | [execution](exec-table.set) |
| (reserved) | `0x27` | | | |
| `i32.load x memarg` | `0x28` | `[at] -> [i32]` | [validation](valid-load-val) | [execution](exec-load-val) |
| `i64.load x memarg` | `0x29` | `[at] -> [i64]` | [validation](valid-load-val) | [execution](exec-load-val) |
| `f32.load x memarg` | `0x2A` | `[at] -> [f32]` | [validation](valid-load-val) | [execution](exec-load-val) |
| `f64.load x memarg` | `0x2B` | `[at] -> [f64]` | [validation](valid-load-val) | [execution](exec-load-val) |
| `i32.load8_s x memarg` | `0x2C` | `[at] -> [i32]` | [validation](valid-load-pack) | [execution](exec-load-pack) |
| `i32.load8_u x memarg` | `0x2D` | `[at] -> [i32]` | [validation](valid-load-pack) | [execution](exec-load-pack) |
| `i32.load16_s x memarg` | `0x2E` | `[at] -> [i32]` | [validation](valid-load-pack) | [execution](exec-load-pack) |
| `i32.load16_u x memarg` | `0x2F` | `[at] -> [i32]` | [validation](valid-load-pack) | [execution](exec-load-pack) |
| `i64.load8_s x memarg` | `0x30` | `[at] -> [i64]` | [validation](valid-load-pack) | [execution](exec-load-pack) |
| `i64.load8_u x memarg` | `0x31` | `[at] -> [i64]` | [validation](valid-load-pack) | [execution](exec-load-pack) |
| `i64.load16_s x memarg` | `0x32` | `[at] -> [i64]` | [validation](valid-load-pack) | [execution](exec-load-pack) |
| `i64.load16_u x memarg` | `0x33` | `[at] -> [i64]` | [validation](valid-load-pack) | [execution](exec-load-pack) |
| `i64.load32_s x memarg` | `0x34` | `[at] -> [i64]` | [validation](valid-load-pack) | [execution](exec-load-pack) |
| `i64.load32_u x memarg` | `0x35` | `[at] -> [i64]` | [validation](valid-load-pack) | [execution](exec-load-pack) |
| `i32.store x memarg` | `0x36` | `[at i32] -> []` | [validation](valid-store-val) | [execution](exec-store-val) |
| `i64.store x memarg` | `0x37` | `[at i64] -> []` | [validation](valid-store-val) | [execution](exec-store-val) |
| `f32.store x memarg` | `0x38` | `[at f32] -> []` | [validation](valid-store-val) | [execution](exec-store-val) |
| `f64.store x memarg` | `0x39` | `[at f64] -> []` | [validation](valid-store-val) | [execution](exec-store-val) |
| `i32.store8 x memarg` | `0x3A` | `[at i32] -> []` | [validation](valid-store-pack) | [execution](exec-store-pack) |
| `i32.store16 x memarg` | `0x3B` | `[at i32] -> []` | [validation](valid-store-pack) | [execution](exec-store-pack) |
| `i64.store8 x memarg` | `0x3C` | `[at i64] -> []` | [validation](valid-store-pack) | [execution](exec-store-pack) |
| `i64.store16 x memarg` | `0x3D` | `[at i64] -> []` | [validation](valid-store-pack) | [execution](exec-store-pack) |
| `i64.store32 x memarg` | `0x3E` | `[at i64] -> []` | [validation](valid-store-pack) | [execution](exec-store-pack) |
| `memory.size x` | `0x3F` | `[] -> [at]` | [validation](valid-memory.size) | [execution](exec-memory.size) |
| `memory.grow x` | `0x40` | `[at] -> [at]` | [validation](valid-memory.grow) | [execution](exec-memory.grow) |
| `i32.const i32` | `0x41` | `[] -> [i32]` | [validation](valid-const) | [execution](exec-const) |
| `i64.const i64` | `0x42` | `[] -> [i64]` | [validation](valid-const) | [execution](exec-const) |
| `f32.const f32` | `0x43` | `[] -> [f32]` | [validation](valid-const) | [execution](exec-const) |
| `f64.const f64` | `0x44` | `[] -> [f64]` | [validation](valid-const) | [execution](exec-const) |
| `i32.eqz` | `0x45` | `[i32] -> [i32]` | [validation](valid-testop) | [execution](exec-testop) ([operator](op-ieqz)) |
| `i32.eq` | `0x46` | `[i32 i32] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-ieq)) |
| `i32.ne` | `0x47` | `[i32 i32] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-ine)) |
| `i32.lt_s` | `0x48` | `[i32 i32] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-ilt)) |
| `i32.lt_u` | `0x49` | `[i32 i32] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-ilt)) |
| `i32.gt_s` | `0x4A` | `[i32 i32] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-igt)) |
| `i32.gt_u` | `0x4B` | `[i32 i32] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-igt)) |
| `i32.le_s` | `0x4C` | `[i32 i32] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-ile)) |
| `i32.le_u` | `0x4D` | `[i32 i32] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-ile)) |
| `i32.ge_s` | `0x4E` | `[i32 i32] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-ige)) |
| `i32.ge_u` | `0x4F` | `[i32 i32] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-ige)) |
| `i64.eqz` | `0x50` | `[i64] -> [i32]` | [validation](valid-testop) | [execution](exec-testop) ([operator](op-ieqz)) |
| `i64.eq` | `0x51` | `[i64 i64] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-ieq)) |
| `i64.ne` | `0x52` | `[i64 i64] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-ine)) |
| `i64.lt_s` | `0x53` | `[i64 i64] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-ilt)) |
| `i64.lt_u` | `0x54` | `[i64 i64] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-ilt)) |
| `i64.gt_s` | `0x55` | `[i64 i64] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-igt)) |
| `i64.gt_u` | `0x56` | `[i64 i64] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-igt)) |
| `i64.le_s` | `0x57` | `[i64 i64] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-ile)) |
| `i64.le_u` | `0x58` | `[i64 i64] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-ile)) |
| `i64.ge_s` | `0x59` | `[i64 i64] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-ige)) |
| `i64.ge_u` | `0x5A` | `[i64 i64] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-ige)) |
| `f32.eq` | `0x5B` | `[f32 f32] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-feq)) |
| `f32.ne` | `0x5C` | `[f32 f32] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-fne)) |
| `f32.lt` | `0x5D` | `[f32 f32] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-flt)) |
| `f32.gt` | `0x5E` | `[f32 f32] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-fgt)) |
| `f32.le` | `0x5F` | `[f32 f32] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-fle)) |
| `f32.ge` | `0x60` | `[f32 f32] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-fge)) |
| `f64.eq` | `0x61` | `[f64 f64] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-feq)) |
| `f64.ne` | `0x62` | `[f64 f64] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-fne)) |
| `f64.lt` | `0x63` | `[f64 f64] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-flt)) |
| `f64.gt` | `0x64` | `[f64 f64] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-fgt)) |
| `f64.le` | `0x65` | `[f64 f64] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-fle)) |
| `f64.ge` | `0x66` | `[f64 f64] -> [i32]` | [validation](valid-relop) | [execution](exec-relop) ([operator](op-fge)) |
| `i32.clz` | `0x67` | `[i32] -> [i32]` | [validation](valid-unop) | [execution](exec-unop) ([operator](op-iclz)) |
| `i32.ctz` | `0x68` | `[i32] -> [i32]` | [validation](valid-unop) | [execution](exec-unop) ([operator](op-ictz)) |
| `i32.popcnt` | `0x69` | `[i32] -> [i32]` | [validation](valid-unop) | [execution](exec-unop) ([operator](op-ipopcnt)) |
| `i32.add` | `0x6A` | `[i32 i32] -> [i32]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-iadd)) |
| `i32.sub` | `0x6B` | `[i32 i32] -> [i32]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-isub)) |
| `i32.mul` | `0x6C` | `[i32 i32] -> [i32]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-imul)) |
| `i32.div_s` | `0x6D` | `[i32 i32] -> [i32]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-idiv)) |
| `i32.div_u` | `0x6E` | `[i32 i32] -> [i32]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-idiv)) |
| `i32.rem_s` | `0x6F` | `[i32 i32] -> [i32]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-irem)) |
| `i32.rem_u` | `0x70` | `[i32 i32] -> [i32]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-irem)) |
| `i32.and` | `0x71` | `[i32 i32] -> [i32]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-iand)) |
| `i32.or` | `0x72` | `[i32 i32] -> [i32]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-ior)) |
| `i32.xor` | `0x73` | `[i32 i32] -> [i32]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-ixor)) |
| `i32.shl` | `0x74` | `[i32 i32] -> [i32]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-ishl)) |
| `i32.shr_s` | `0x75` | `[i32 i32] -> [i32]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-ishr)) |
| `i32.shr_u` | `0x76` | `[i32 i32] -> [i32]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-ishr)) |
| `i32.rotl` | `0x77` | `[i32 i32] -> [i32]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-irotl)) |
| `i32.rotr` | `0x78` | `[i32 i32] -> [i32]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-irotr)) |
| `i64.clz` | `0x79` | `[i64] -> [i64]` | [validation](valid-unop) | [execution](exec-unop) ([operator](op-iclz)) |
| `i64.ctz` | `0x7A` | `[i64] -> [i64]` | [validation](valid-unop) | [execution](exec-unop) ([operator](op-ictz)) |
| `i64.popcnt` | `0x7B` | `[i64] -> [i64]` | [validation](valid-unop) | [execution](exec-unop) ([operator](op-ipopcnt)) |
| `i64.add` | `0x7C` | `[i64 i64] -> [i64]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-iadd)) |
| `i64.sub` | `0x7D` | `[i64 i64] -> [i64]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-isub)) |
| `i64.mul` | `0x7E` | `[i64 i64] -> [i64]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-imul)) |
| `i64.div_s` | `0x7F` | `[i64 i64] -> [i64]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-idiv)) |
| `i64.div_u` | `0x80` | `[i64 i64] -> [i64]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-idiv)) |
| `i64.rem_s` | `0x81` | `[i64 i64] -> [i64]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-irem)) |
| `i64.rem_u` | `0x82` | `[i64 i64] -> [i64]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-irem)) |
| `i64.and` | `0x83` | `[i64 i64] -> [i64]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-iand)) |
| `i64.or` | `0x84` | `[i64 i64] -> [i64]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-ior)) |
| `i64.xor` | `0x85` | `[i64 i64] -> [i64]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-ixor)) |
| `i64.shl` | `0x86` | `[i64 i64] -> [i64]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-ishl)) |
| `i64.shr_s` | `0x87` | `[i64 i64] -> [i64]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-ishr)) |
| `i64.shr_u` | `0x88` | `[i64 i64] -> [i64]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-ishr)) |
| `i64.rotl` | `0x89` | `[i64 i64] -> [i64]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-irotl)) |
| `i64.rotr` | `0x8A` | `[i64 i64] -> [i64]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-irotr)) |
| `f32.abs` | `0x8B` | `[f32] -> [f32]` | [validation](valid-unop) | [execution](exec-unop) ([operator](op-fabs)) |
| `f32.neg` | `0x8C` | `[f32] -> [f32]` | [validation](valid-unop) | [execution](exec-unop) ([operator](op-fneg)) |
| `f32.ceil` | `0x8D` | `[f32] -> [f32]` | [validation](valid-unop) | [execution](exec-unop) ([operator](op-fceil)) |
| `f32.floor` | `0x8E` | `[f32] -> [f32]` | [validation](valid-unop) | [execution](exec-unop) ([operator](op-ffloor)) |
| `f32.trunc` | `0x8F` | `[f32] -> [f32]` | [validation](valid-unop) | [execution](exec-unop) ([operator](op-ftrunc)) |
| `f32.nearest` | `0x90` | `[f32] -> [f32]` | [validation](valid-unop) | [execution](exec-unop) ([operator](op-fnearest)) |
| `f32.sqrt` | `0x91` | `[f32] -> [f32]` | [validation](valid-unop) | [execution](exec-unop) ([operator](op-fsqrt)) |
| `f32.add` | `0x92` | `[f32 f32] -> [f32]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-fadd)) |
| `f32.sub` | `0x93` | `[f32 f32] -> [f32]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-fsub)) |
| `f32.mul` | `0x94` | `[f32 f32] -> [f32]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-fmul)) |
| `f32.div` | `0x95` | `[f32 f32] -> [f32]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-fdiv)) |
| `f32.min` | `0x96` | `[f32 f32] -> [f32]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-fmin)) |
| `f32.max` | `0x97` | `[f32 f32] -> [f32]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-fmax)) |
| `f32.copysign` | `0x98` | `[f32 f32] -> [f32]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-fcopysign)) |
| `f64.abs` | `0x99` | `[f64] -> [f64]` | [validation](valid-unop) | [execution](exec-unop) ([operator](op-fabs)) |
| `f64.neg` | `0x9A` | `[f64] -> [f64]` | [validation](valid-unop) | [execution](exec-unop) ([operator](op-fneg)) |
| `f64.ceil` | `0x9B` | `[f64] -> [f64]` | [validation](valid-unop) | [execution](exec-unop) ([operator](op-fceil)) |
| `f64.floor` | `0x9C` | `[f64] -> [f64]` | [validation](valid-unop) | [execution](exec-unop) ([operator](op-ffloor)) |
| `f64.trunc` | `0x9D` | `[f64] -> [f64]` | [validation](valid-unop) | [execution](exec-unop) ([operator](op-ftrunc)) |
| `f64.nearest` | `0x9E` | `[f64] -> [f64]` | [validation](valid-unop) | [execution](exec-unop) ([operator](op-fnearest)) |
| `f64.sqrt` | `0x9F` | `[f64] -> [f64]` | [validation](valid-unop) | [execution](exec-unop) ([operator](op-fsqrt)) |
| `f64.add` | `0xA0` | `[f64 f64] -> [f64]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-fadd)) |
| `f64.sub` | `0xA1` | `[f64 f64] -> [f64]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-fsub)) |
| `f64.mul` | `0xA2` | `[f64 f64] -> [f64]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-fmul)) |
| `f64.div` | `0xA3` | `[f64 f64] -> [f64]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-fdiv)) |
| `f64.min` | `0xA4` | `[f64 f64] -> [f64]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-fmin)) |
| `f64.max` | `0xA5` | `[f64 f64] -> [f64]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-fmax)) |
| `f64.copysign` | `0xA6` | `[f64 f64] -> [f64]` | [validation](valid-binop) | [execution](exec-binop) ([operator](op-fcopysign)) |
| `i32.wrap_i64` | `0xA7` | `[i64] -> [i32]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-wrap)) |
| `i32.trunc_f32_s` | `0xA8` | `[f32] -> [i32]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-trunc)) |
| `i32.trunc_f32_u` | `0xA9` | `[f32] -> [i32]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-trunc)) |
| `i32.trunc_f64_s` | `0xAA` | `[f64] -> [i32]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-trunc)) |
| `i32.trunc_f64_u` | `0xAB` | `[f64] -> [i32]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-trunc)) |
| `i64.extend_i32_s` | `0xAC` | `[i32] -> [i64]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-extend)) |
| `i64.extend_i32_u` | `0xAD` | `[i32] -> [i64]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-extend)) |
| `i64.trunc_f32_s` | `0xAE` | `[f32] -> [i64]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-trunc)) |
| `i64.trunc_f32_u` | `0xAF` | `[f32] -> [i64]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-trunc)) |
| `i64.trunc_f64_s` | `0xB0` | `[f64] -> [i64]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-trunc)) |
| `i64.trunc_f64_u` | `0xB1` | `[f64] -> [i64]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-trunc)) |
| `f32.convert_i32_s` | `0xB2` | `[i32] -> [f32]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-convert)) |
| `f32.convert_i32_u` | `0xB3` | `[i32] -> [f32]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-convert)) |
| `f32.convert_i64_s` | `0xB4` | `[i64] -> [f32]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-convert)) |
| `f32.convert_i64_u` | `0xB5` | `[i64] -> [f32]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-convert)) |
| `f32.demote_f64` | `0xB6` | `[f64] -> [f32]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-demote)) |
| `f64.convert_i32_s` | `0xB7` | `[i32] -> [f64]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-convert)) |
| `f64.convert_i32_u` | `0xB8` | `[i32] -> [f64]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-convert)) |
| `f64.convert_i64_s` | `0xB9` | `[i64] -> [f64]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-convert)) |
| `f64.convert_i64_u` | `0xBA` | `[i64] -> [f64]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-convert)) |
| `f64.promote_f32` | `0xBB` | `[f32] -> [f64]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-promote)) |
| `i32.reinterpret_f32` | `0xBC` | `[f32] -> [i32]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-reinterpret)) |
| `i64.reinterpret_f64` | `0xBD` | `[f64] -> [i64]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-reinterpret)) |
| `f32.reinterpret_i32` | `0xBE` | `[i32] -> [f32]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-reinterpret)) |
| `f64.reinterpret_i64` | `0xBF` | `[i64] -> [f64]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-reinterpret)) |
| `i32.extend8_s` | `0xC0` | `[i32] -> [i32]` | [validation](valid-unop) | [execution](exec-unop) ([operator](op-iextendn)) |
| `i32.extend16_s` | `0xC1` | `[i32] -> [i32]` | [validation](valid-unop) | [execution](exec-unop) ([operator](op-iextendn)) |
| `i64.extend8_s` | `0xC2` | `[i64] -> [i64]` | [validation](valid-unop) | [execution](exec-unop) ([operator](op-iextendn)) |
| `i64.extend16_s` | `0xC3` | `[i64] -> [i64]` | [validation](valid-unop) | [execution](exec-unop) ([operator](op-iextendn)) |
| `i64.extend32_s` | `0xC4` | `[i64] -> [i64]` | [validation](valid-unop) | [execution](exec-unop) ([operator](op-iextendn)) |
| (reserved) | `0xC5` | | | |
| (reserved) | `0xC6` | | | |
| (reserved) | `0xC7` | | | |
| (reserved) | `0xC8` | | | |
| (reserved) | `0xC9` | | | |
| (reserved) | `0xCA` | | | |
| (reserved) | `0xCB` | | | |
| (reserved) | `0xCC` | | | |
| (reserved) | `0xCD` | | | |
| (reserved) | `0xCE` | | | |
| (reserved) | `0xCF` | | | |
| `ref.null ht` | `0xD0` | `[] -> [(ref null ht)]` | [validation](valid-ref.null) | [execution](exec-ref.null) |
| `ref.is_null` | `0xD1` | `[(ref null ht)] -> [i32]` | [validation](valid-ref.is_null) | [execution](exec-ref.is_null) |
| `ref.func x` | `0xD2` | `[] -> [ref ht]` | [validation](valid-ref.func) | [execution](exec-ref.func) |
| `ref.eq` | `0xD3` | `[eqref eqref] -> [i32]` | [validation](valid-ref.eq) | [execution](exec-ref.eq) |
| `ref.as_non_null` | `0xD4` | `[(ref null ht)] -> [(ref ht)]` | [validation](valid-ref.as_non_null) | [execution](exec-ref.as_non_null) |
| `br_on_null l` | `0xD5` | `[t* (ref null ht)] -> [t* (ref ht)]` | [validation](valid-br_on_null) | [execution](exec-br_on_null) |
| `br_on_non_null l` | `0xD6` | `[t* (ref null ht)] -> [t*]` | [validation](valid-br_on_non_null) | [execution](exec-br_on_non_null) |
| (reserved) | `0xD7` | | | |
| (reserved) | `0xD8` | | | |
| (reserved) | `0xD9` | | | |
| (reserved) | `0xDA` | | | |
| (reserved) | `0xDB` | | | |
| (reserved) | `0xDC` | | | |
| (reserved) | `0xDD` | | | |
| (reserved) | `0xDE` | | | |
| (reserved) | `0xDF` | | | |
| (reserved) | `0xE0` | | | |
| (reserved) | `0xE1` | | | |
| (reserved) | `0xE2` | | | |
| (reserved) | `0xE3` | | | |
| (reserved) | `0xE4` | | | |
| (reserved) | `0xE5` | | | |
| (reserved) | `0xE6` | | | |
| (reserved) | `0xE7` | | | |
| (reserved) | `0xE8` | | | |
| (reserved) | `0xE9` | | | |
| (reserved) | `0xEA` | | | |
| (reserved) | `0xEB` | | | |
| (reserved) | `0xEC` | | | |
| (reserved) | `0xED` | | | |
| (reserved) | `0xEE` | | | |
| (reserved) | `0xEF` | | | |
| (reserved) | `0xF0` | | | |
| (reserved) | `0xF1` | | | |
| (reserved) | `0xF2` | | | |
| (reserved) | `0xF3` | | | |
| (reserved) | `0xF4` | | | |
| (reserved) | `0xF5` | | | |
| (reserved) | `0xF6` | | | |
| (reserved) | `0xF7` | | | |
| (reserved) | `0xF8` | | | |
| (reserved) | `0xF9` | | | |
| (reserved) | `0xFA` | | | |
| `struct.new x` | `0xFB 0x00` | `[t*] -> [(ref x)]` | [validation](valid-struct.new) | [execution](exec-struct.new) |
| `struct.new_default x` | `0xFB 0x01` | `[] -> [(ref x)]` | [validation](valid-struct.new_default) | [execution](exec-struct.new_default) |
| `struct.get x y` | `0xFB 0x02` | `[(ref null x)] -> [t]` | [validation](valid-struct.get) | [execution](exec-struct.get) |
| `struct.get_s x y` | `0xFB 0x03` | `[(ref null x)] -> [i32]` | [validation](valid-struct.get) | [execution](exec-struct.get) |
| `struct.get_u x y` | `0xFB 0x04` | `[(ref null x)] -> [i32]` | [validation](valid-struct.get) | [execution](exec-struct.get) |
| `struct.set x y` | `0xFB 0x05` | `[(ref null x) t] -> []` | [validation](valid-struct.set) | [execution](exec-struct.set) |
| `array.new x` | `0xFB 0x06` | `[t i32] -> [(ref x)]` | [validation](valid-array.new) | [execution](exec-array.new) |
| `array.new_default x` | `0xFB 0x07` | `[i32] -> [(ref x)]` | [validation](valid-array.new) | [execution](exec-array.new) |
| `array.new_fixed x n` | `0xFB 0x08` | `[t^n] -> [(ref x)]` | [validation](valid-array.new_fixed) | [execution](exec-array.new_fixed) |
| `array.new_data x y` | `0xFB 0x09` | `[i32 i32] -> [(ref x)]` | [validation](valid-array.new_data) | [execution](exec-array.new_data) |
| `array.new_elem x y` | `0xFB 0x0A` | `[i32 i32] -> [(ref x)]` | [validation](valid-array.new_elem) | [execution](exec-array.new_elem) |
| `array.get x` | `0xFB 0x0B` | `[(ref null x) i32] -> [t]` | [validation](valid-array.get) | [execution](exec-array.get) |
| `array.get_s x` | `0xFB 0x0C` | `[(ref null x) i32] -> [i32]` | [validation](valid-array.get) | [execution](exec-array.get) |
| `array.get_u x` | `0xFB 0x0D` | `[(ref null x) i32] -> [i32]` | [validation](valid-array.get) | [execution](exec-array.get) |
| `array.set x` | `0xFB 0x0E` | `[(ref null x) i32 t] -> []` | [validation](valid-array.set) | [execution](exec-array.set) |
| `array.len` | `0xFB 0x0F` | `[(ref null array)] -> [i32]` | [validation](valid-array.len) | [execution](exec-array.len) |
| `array.fill x` | `0xFB 0x10` | `[(ref null x) i32 t i32] -> []` | [validation](valid-array.fill) | [execution](exec-array.fill) |
| `array.copy x y` | `0xFB 0x11` | `[(ref null x) i32 (ref null y) i32 i32] -> []` | [validation](valid-array.copy) | [execution](exec-array.copy) |
| `array.init_data x y` | `0xFB 0x12` | `[(ref null x) i32 i32 i32] -> []` | [validation](valid-array.init_data) | [execution](exec-array.init_data) |
| `array.init_elem x y` | `0xFB 0x13` | `[(ref null x) i32 i32 i32] -> []` | [validation](valid-array.init_elem) | [execution](exec-array.init_elem) |
| `ref.test (ref t)` | `0xFB 0x14` | `[(ref t')] -> [i32]` | [validation](valid-ref.test) | [execution](exec-ref.test) |
| `ref.test (ref null t)` | `0xFB 0x15` | `[(ref null t')] -> [i32]` | [validation](valid-ref.test) | [execution](exec-ref.test) |
| `ref.cast (ref t)` | `0xFB 0x16` | `[(ref t')] -> [(ref t)]` | [validation](valid-ref.cast) | [execution](exec-ref.cast) |
| `ref.cast (ref null t)` | `0xFB 0x17` | `[(ref null t')] -> [(ref null t)]` | [validation](valid-ref.cast) | [execution](exec-ref.cast) |
| `br_on_cast t1 t2` | `0xFB 0x18` | `[t1] -> [t1 reftypediff t2]` | [validation](valid-br_on_cast) | [execution](exec-br_on_cast) |
| `br_on_cast_fail t1 t2` | `0xFB 0x19` | `[t1] -> [t2]` | [validation](valid-br_on_cast_fail) | [execution](exec-br_on_cast_fail) |
| `any.convert_extern` | `0xFB 0x1A` | `[(ref null extern)] -> [(ref null any)]` | [validation](valid-any.convert_extern) | [execution](exec-any.convert_extern) |
| `extern.convert_any` | `0xFB 0x1B` | `[(ref null any)] -> [(ref null extern)]` | [validation](valid-extern.convert_any) | [execution](exec-extern.convert_any) |
| `ref.i31` | `0xFB 0x1C` | `[i32] -> [(ref i31)]` | [validation](valid-ref.i31) | [execution](exec-ref.i31) |
| `i31.get_s` | `0xFB 0x1D` | `[i31ref] -> [i32]` | [validation](valid-i31.get) | [execution](exec-i31.get) |
| `i31.get_u` | `0xFB 0x1E` | `[i31ref] -> [i32]` | [validation](valid-i31.get) | [execution](exec-i31.get) |
| (reserved) | `0xFB 0x1F ...` | | | |
| `i32.trunc_sat_f32_s` | `0xFC 0x00` | `[f32] -> [i32]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-trunc_sat)) |
| `i32.trunc_sat_f32_u` | `0xFC 0x01` | `[f32] -> [i32]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-trunc_sat)) |
| `i32.trunc_sat_f64_s` | `0xFC 0x02` | `[f64] -> [i32]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-trunc_sat)) |
| `i32.trunc_sat_f64_u` | `0xFC 0x03` | `[f64] -> [i32]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-trunc_sat)) |
| `i64.trunc_sat_f32_s` | `0xFC 0x04` | `[f32] -> [i64]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-trunc_sat)) |
| `i64.trunc_sat_f32_u` | `0xFC 0x05` | `[f32] -> [i64]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-trunc_sat)) |
| `i64.trunc_sat_f64_s` | `0xFC 0x06` | `[f64] -> [i64]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-trunc_sat)) |
| `i64.trunc_sat_f64_u` | `0xFC 0x07` | `[f64] -> [i64]` | [validation](valid-cvtop) | [execution](exec-cvtop) ([operator](op-trunc_sat)) |
| `memory.init x y` | `0xFC 0x08` | `[at i32 i32] -> []` | [validation](valid-memory.init) | [execution](exec-memory.init) |
| `data.drop x` | `0xFC 0x09` | `[] -> []` | [validation](valid-data.drop) | [execution](exec-data.drop) |
| `memory.copy x y` | `0xFC 0x0A` | `[at1 at2 at] -> []` | [validation](valid-memory.copy) | [execution](exec-memory.copy) |
| `memory.fill y` | `0xFC 0x0B` | `[at i32 at] -> []` | [validation](valid-memory.fill) | [execution](exec-memory.fill) |
| `table.init x y` | `0xFC 0x0C` | `[at i32 i32] -> []` | [validation](valid-table.init) | [execution](exec-table.init) |
| `elem.drop x` | `0xFC 0x0D` | `[] -> []` | [validation](valid-elem.drop) | [execution](exec-elem.drop) |
| `table.copy x y` | `0xFC 0x0E` | `[at1 at2 at] -> []` | [validation](valid-table.copy) | [execution](exec-table.copy) |
| `table.grow x` | `0xFC 0x0F` | `[t at] -> [at]` | [validation](valid-table.grow) | [execution](exec-table.grow) |
| `table.size x` | `0xFC 0x10` | `[] -> [at]` | [validation](valid-table.size) | [execution](exec-table.size) |
| `table.fill x` | `0xFC 0x11` | `[at t at] -> []` | [validation](valid-table.fill) | [execution](exec-table.fill) |
| (reserved) | `0xFC 0x12 ...` | | | |
| `v128.load x memarg` | `0xFD 0x00` | `[at] -> [v128]` | [validation](valid-vload-val) | [execution](exec-vload-val) |
| `v128.load8x8_s x memarg` | `0xFD 0x01` | `[at] -> [v128]` | [validation](valid-vload-pack) | [execution](exec-vload-pack) |
| `v128.load8x8_u x memarg` | `0xFD 0x02` | `[at] -> [v128]` | [validation](valid-vload-pack) | [execution](exec-vload-pack) |
| `v128.load16x4_s x memarg` | `0xFD 0x03` | `[at] -> [v128]` | [validation](valid-vload-pack) | [execution](exec-vload-pack) |
| `v128.load16x4_u x memarg` | `0xFD 0x04` | `[at] -> [v128]` | [validation](valid-vload-pack) | [execution](exec-vload-pack) |
| `v128.load32x2_s x memarg` | `0xFD 0x05` | `[at] -> [v128]` | [validation](valid-vload-pack) | [execution](exec-vload-pack) |
| `v128.load32x2_u x memarg` | `0xFD 0x06` | `[at] -> [v128]` | [validation](valid-vload-pack) | [execution](exec-vload-pack) |
| `v128.load8_splat x memarg` | `0xFD 0x07` | `[at] -> [v128]` | [validation](valid-vload-splat) | [execution](exec-vload-splat) |
| `v128.load16_splat x memarg` | `0xFD 0x08` | `[at] -> [v128]` | [validation](valid-vload-splat) | [execution](exec-vload-splat) |
| `v128.load32_splat x memarg` | `0xFD 0x09` | `[at] -> [v128]` | [validation](valid-vload-splat) | [execution](exec-vload-splat) |
| `v128.load64_splat x memarg` | `0xFD 0x0A` | `[at] -> [v128]` | [validation](valid-vload-splat) | [execution](exec-vload-splat) |
| `v128.store x memarg` | `0xFD 0x0B` | `[at v128] -> []` | [validation](valid-vstore) | [execution](exec-vstore) |
| `v128.const i128` | `0xFD 0x0C` | `[] -> [v128]` | [validation](valid-vconst) | [execution](exec-vconst) |
| `i8x16.shuffle laneidx^16` | `0xFD 0x0D` | `[v128 v128] -> [v128]` | [validation](valid-vshuffle) | [execution](exec-vshuffle) ([operator](op-ivshuffle)) |
| `i8x16.swizzle` | `0xFD 0x0E` | `[v128 v128] -> [v128]` | [validation](valid-vswizzlop) | [execution](exec-vswizzlop) ([operator](op-ivswizzle)) |
| `i8x16.splat` | `0xFD 0x0F` | `[i32] -> [v128]` | [validation](valid-vsplat) | [execution](exec-vsplat) |
| `i16x8.splat` | `0xFD 0x10` | `[i32] -> [v128]` | [validation](valid-vsplat) | [execution](exec-vsplat) |
| `i32x4.splat` | `0xFD 0x11` | `[i32] -> [v128]` | [validation](valid-vsplat) | [execution](exec-vsplat) |
| `i64x2.splat` | `0xFD 0x12` | `[i64] -> [v128]` | [validation](valid-vsplat) | [execution](exec-vsplat) |
| `f32x4.splat` | `0xFD 0x13` | `[f32] -> [v128]` | [validation](valid-vsplat) | [execution](exec-vsplat) |
| `f64x2.splat` | `0xFD 0x14` | `[f64] -> [v128]` | [validation](valid-vsplat) | [execution](exec-vsplat) |
| `i8x16.extract_lane_s laneidx` | `0xFD 0x15` | `[v128] -> [i32]` | [validation](valid-vextract_lane) | [execution](exec-vextract_lane) |
| `i8x16.extract_lane_u laneidx` | `0xFD 0x16` | `[v128] -> [i32]` | [validation](valid-vextract_lane) | [execution](exec-vextract_lane) |
| `i8x16.replace_lane laneidx` | `0xFD 0x17` | `[v128 i32] -> [v128]` | [validation](valid-vreplace_lane) | [execution](exec-vreplace_lane) |
| `i16x8.extract_lane_s laneidx` | `0xFD 0x18` | `[v128] -> [i32]` | [validation](valid-vextract_lane) | [execution](exec-vextract_lane) |
| `i16x8.extract_lane_u laneidx` | `0xFD 0x19` | `[v128] -> [i32]` | [validation](valid-vextract_lane) | [execution](exec-vextract_lane) |
| `i16x8.replace_lane laneidx` | `0xFD 0x1A` | `[v128 i32] -> [v128]` | [validation](valid-vreplace_lane) | [execution](exec-vreplace_lane) |
| `i32x4.extract_lane laneidx` | `0xFD 0x1B` | `[v128] -> [i32]` | [validation](valid-vextract_lane) | [execution](exec-vextract_lane) |
| `i32x4.replace_lane laneidx` | `0xFD 0x1C` | `[v128 i32] -> [v128]` | [validation](valid-vreplace_lane) | [execution](exec-vreplace_lane) |
| `i64x2.extract_lane laneidx` | `0xFD 0x1D` | `[v128] -> [i64]` | [validation](valid-vextract_lane) | [execution](exec-vextract_lane) |
| `i64x2.replace_lane laneidx` | `0xFD 0x1E` | `[v128 i64] -> [v128]` | [validation](valid-vreplace_lane) | [execution](exec-vreplace_lane) |
| `f32x4.extract_lane laneidx` | `0xFD 0x1F` | `[v128] -> [f32]` | [validation](valid-vextract_lane) | [execution](exec-vextract_lane) |
| `f32x4.replace_lane laneidx` | `0xFD 0x20` | `[v128 f32] -> [v128]` | [validation](valid-vreplace_lane) | [execution](exec-vreplace_lane) |
| `f64x2.extract_lane laneidx` | `0xFD 0x21` | `[v128] -> [f64]` | [validation](valid-vextract_lane) | [execution](exec-vextract_lane) |
| `f64x2.replace_lane laneidx` | `0xFD 0x22` | `[v128 f64] -> [v128]` | [validation](valid-vreplace_lane) | [execution](exec-vreplace_lane) |
| `i8x16.eq` | `0xFD 0x23` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-ieq)) |
| `i8x16.ne` | `0xFD 0x24` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-ine)) |
| `i8x16.lt_s` | `0xFD 0x25` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-ilt)) |
| `i8x16.lt_u` | `0xFD 0x26` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-ilt)) |
| `i8x16.gt_s` | `0xFD 0x27` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-igt)) |
| `i8x16.gt_u` | `0xFD 0x28` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-igt)) |
| `i8x16.le_s` | `0xFD 0x29` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-ile)) |
| `i8x16.le_u` | `0xFD 0x2A` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-ile)) |
| `i8x16.ge_s` | `0xFD 0x2B` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-ige)) |
| `i8x16.ge_u` | `0xFD 0x2C` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-ige)) |
| `i16x8.eq` | `0xFD 0x2D` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-ieq)) |
| `i16x8.ne` | `0xFD 0x2E` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-ine)) |
| `i16x8.lt_s` | `0xFD 0x2F` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-ilt)) |
| `i16x8.lt_u` | `0xFD 0x30` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-ilt)) |
| `i16x8.gt_s` | `0xFD 0x31` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-igt)) |
| `i16x8.gt_u` | `0xFD 0x32` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-igt)) |
| `i16x8.le_s` | `0xFD 0x33` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-ile)) |
| `i16x8.le_u` | `0xFD 0x34` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-ile)) |
| `i16x8.ge_s` | `0xFD 0x35` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-ige)) |
| `i16x8.ge_u` | `0xFD 0x36` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-ige)) |
| `i32x4.eq` | `0xFD 0x37` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-ieq)) |
| `i32x4.ne` | `0xFD 0x38` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-ine)) |
| `i32x4.lt_s` | `0xFD 0x39` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-ilt)) |
| `i32x4.lt_u` | `0xFD 0x3A` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-ilt)) |
| `i32x4.gt_s` | `0xFD 0x3B` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-igt)) |
| `i32x4.gt_u` | `0xFD 0x3C` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-igt)) |
| `i32x4.le_s` | `0xFD 0x3D` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-ile)) |
| `i32x4.le_u` | `0xFD 0x3E` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-ile)) |
| `i32x4.ge_s` | `0xFD 0x3F` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-ige)) |
| `i32x4.ge_u` | `0xFD 0x40` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-ige)) |
| `f32x4.eq` | `0xFD 0x41` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-feq)) |
| `f32x4.ne` | `0xFD 0x42` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-fne)) |
| `f32x4.lt` | `0xFD 0x43` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-flt)) |
| `f32x4.gt` | `0xFD 0x44` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-fgt)) |
| `f32x4.le` | `0xFD 0x45` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-fle)) |
| `f32x4.ge` | `0xFD 0x46` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-fge)) |
| `f64x2.eq` | `0xFD 0x47` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-feq)) |
| `f64x2.ne` | `0xFD 0x48` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-fne)) |
| `f64x2.lt` | `0xFD 0x49` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-flt)) |
| `f64x2.gt` | `0xFD 0x4A` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-fgt)) |
| `f64x2.le` | `0xFD 0x4B` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-fle)) |
| `f64x2.ge` | `0xFD 0x4C` | `[v128 v128] -> [v128]` | [validation](valid-vrelop) | [execution](exec-vrelop) ([operator](op-fge)) |
| `v128.not` | `0xFD 0x4D` | `[v128] -> [v128]` | [validation](valid-vvunop) | [execution](exec-vvunop) ([operator](op-inot)) |
| `v128.and` | `0xFD 0x4E` | `[v128 v128] -> [v128]` | [validation](valid-vvbinop) | [execution](exec-vvbinop) ([operator](op-iand)) |
| `v128.andnot` | `0xFD 0x4F` | `[v128 v128] -> [v128]` | [validation](valid-vvbinop) | [execution](exec-vvbinop) ([operator](op-iandnot)) |
| `v128.or` | `0xFD 0x50` | `[v128 v128] -> [v128]` | [validation](valid-vvbinop) | [execution](exec-vvbinop) ([operator](op-ior)) |
| `v128.xor` | `0xFD 0x51` | `[v128 v128] -> [v128]` | [validation](valid-vvbinop) | [execution](exec-vvbinop) ([operator](op-ixor)) |
| `v128.bitselect` | `0xFD 0x52` | `[v128 v128 v128] -> [v128]` | [validation](valid-vvternop) | [execution](exec-vvternop) ([operator](op-ibitselect)) |
| `v128.any_true` | `0xFD 0x53` | `[v128] -> [i32]` | [validation](valid-vvtestop) | [execution](exec-vvtestop) |
| `v128.load8_lane memarg laneidx` | `0xFD 0x54` | `[at v128] -> [v128]` | [validation](valid-vload_lane) | [execution](exec-vload_lane) |
| `v128.load16_lane memarg laneidx` | `0xFD 0x55` | `[at v128] -> [v128]` | [validation](valid-vload_lane) | [execution](exec-vload_lane) |
| `v128.load32_lane memarg laneidx` | `0xFD 0x56` | `[at v128] -> [v128]` | [validation](valid-vload_lane) | [execution](exec-vload_lane) |
| `v128.load64_lane memarg laneidx` | `0xFD 0x57` | `[at v128] -> [v128]` | [validation](valid-vload_lane) | [execution](exec-vload_lane) |
| `v128.store8_lane memarg laneidx` | `0xFD 0x58` | `[at v128] -> []` | [validation](valid-vstore_lane) | [execution](exec-vstore_lane) |
| `v128.store16_lane memarg laneidx` | `0xFD 0x59` | `[at v128] -> []` | [validation](valid-vstore_lane) | [execution](exec-vstore_lane) |
| `v128.store32_lane memarg laneidx` | `0xFD 0x5A` | `[at v128] -> []` | [validation](valid-vstore_lane) | [execution](exec-vstore_lane) |
| `v128.store64_lane memarg laneidx` | `0xFD 0x5B` | `[at v128] -> []` | [validation](valid-vstore_lane) | [execution](exec-vstore_lane) |
| `v128.load32_zero memarg` | `0xFD 0x5C` | `[at] -> [v128]` | [validation](valid-vload-zero) | [execution](exec-vload-zero) |
| `v128.load64_zero memarg` | `0xFD 0x5D` | `[at] -> [v128]` | [validation](valid-vload-zero) | [execution](exec-vload-zero) |
| `f32x4.demote_f64x2_zero` | `0xFD 0x5E` | `[v128] -> [v128]` | [validation](valid-vcvtop) | [execution](exec-vcvtop) ([operator](op-demote)) |
| `f64x2.promote_low_f32x4` | `0xFD 0x5F` | `[v128] -> [v128]` | [validation](valid-vcvtop) | [execution](exec-vcvtop) ([operator](op-promote)) |
| `i8x16.abs` | `0xFD 0x60` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vunop) ([operator](op-iabs)) |
| `i8x16.neg` | `0xFD 0x61` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vunop) ([operator](op-ineg)) |
| `i8x16.popcnt` | `0xFD 0x62` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vunop) ([operator](op-ipopcnt)) |
| `i8x16.all_true` | `0xFD 0x63` | `[v128] -> [i32]` | [validation](valid-vtestop) | [execution](exec-vtestop) |
| `i8x16.bitmask` | `0xFD 0x64` | `[v128] -> [i32]` | [validation](valid-vbitmask) | [execution](exec-vbitmask) ([operator](op-ivbitmask)) |
| `i8x16.narrow_i16x8_s` | `0xFD 0x65` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vnarrow) ([operator](op-vnarrow)) |
| `i8x16.narrow_i16x8_u` | `0xFD 0x66` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vnarrow) ([operator](op-vnarrow)) |
| `f32x4.ceil` | `0xFD 0x67` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vunop) ([operator](op-fceil)) |
| `f32x4.floor` | `0xFD 0x68` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vunop) ([operator](op-ffloor)) |
| `f32x4.trunc` | `0xFD 0x69` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vunop) ([operator](op-ftrunc)) |
| `f32x4.nearest` | `0xFD 0x6A` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vunop) ([operator](op-fnearest)) |
| `i8x16.shl` | `0xFD 0x6B` | `[v128 i32] -> [v128]` | [validation](valid-vshiftop) | [execution](exec-vshiftop) ([operator](op-ishl)) |
| `i8x16.shr_s` | `0xFD 0x6C` | `[v128 i32] -> [v128]` | [validation](valid-vshiftop) | [execution](exec-vshiftop) ([operator](op-ishr)) |
| `i8x16.shr_u` | `0xFD 0x6D` | `[v128 i32] -> [v128]` | [validation](valid-vshiftop) | [execution](exec-vshiftop) ([operator](op-ishr)) |
| `i8x16.add` | `0xFD 0x6E` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-iadd)) |
| `i8x16.add_sat_s` | `0xFD 0x6F` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-iadd_sat)) |
| `i8x16.add_sat_u` | `0xFD 0x70` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-iadd_sat)) |
| `i8x16.sub` | `0xFD 0x71` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-isub)) |
| `i8x16.sub_sat_s` | `0xFD 0x72` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-isub_sat)) |
| `i8x16.sub_sat_u` | `0xFD 0x73` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-isub_sat)) |
| `f64x2.ceil` | `0xFD 0x74` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vunop) ([operator](op-fceil)) |
| `f64x2.floor` | `0xFD 0x75` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vunop) ([operator](op-ffloor)) |
| `i8x16.min_s` | `0xFD 0x76` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-imin)) |
| `i8x16.min_u` | `0xFD 0x77` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-imin)) |
| `i8x16.max_s` | `0xFD 0x78` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-imax)) |
| `i8x16.max_u` | `0xFD 0x79` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-imax)) |
| `f64x2.trunc` | `0xFD 0x7A` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vunop) ([operator](op-ftrunc)) |
| `i8x16.avgr_u` | `0xFD 0x7B` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-iavgr)) |
| `i16x8.extadd_pairwise_i8x16_s` | `0xFD 0x7C` | `[v128] -> [v128]` | [validation](valid-vextunop) | [execution](exec-vextunop) ([operator](op-vextunop)) |
| `i16x8.extadd_pairwise_i8x16_u` | `0xFD 0x7D` | `[v128] -> [v128]` | [validation](valid-vextunop) | [execution](exec-vextunop) ([operator](op-vextunop)) |
| `i32x4.extadd_pairwise_i16x8_s` | `0xFD 0x7E` | `[v128] -> [v128]` | [validation](valid-vextunop) | [execution](exec-vextunop) ([operator](op-vextunop)) |
| `i32x4.extadd_pairwise_i16x8_u` | `0xFD 0x7F` | `[v128] -> [v128]` | [validation](valid-vextunop) | [execution](exec-vextunop) ([operator](op-vextunop)) |
| `i16x8.abs` | `0xFD 0x80 0x01` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vunop) ([operator](op-iabs)) |
| `i16x8.neg` | `0xFD 0x81 0x01` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vunop) ([operator](op-ineg)) |
| `i16x8.q15mulr_sat_s` | `0xFD 0x82 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-iq15mulrsat)) |
| `i16x8.all_true` | `0xFD 0x83 0x01` | `[v128] -> [i32]` | [validation](valid-vtestop) | [execution](exec-vtestop) |
| `i16x8.bitmask` | `0xFD 0x84 0x01` | `[v128] -> [i32]` | [validation](valid-vbitmask) | [execution](exec-vbitmask) ([operator](op-ivbitmask)) |
| `i16x8.narrow_i32x4_s` | `0xFD 0x85 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vnarrow) ([operator](op-vnarrow)) |
| `i16x8.narrow_i32x4_u` | `0xFD 0x86 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vnarrow) ([operator](op-vnarrow)) |
| `i16x8.extend_low_i8x16_s` | `0xFD 0x87 0x01` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vcvtop) |
| `i16x8.extend_high_i8x16_s` | `0xFD 0x88 0x01` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vcvtop) |
| `i16x8.extend_low_i8x16_u` | `0xFD 0x89 0x01` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vcvtop) |
| `i16x8.extend_high_i8x16_u` | `0xFD 0x8A 0x01` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vcvtop) |
| `i16x8.shl` | `0xFD 0x8B 0x01` | `[v128 i32] -> [v128]` | [validation](valid-vshiftop) | [execution](exec-vshiftop) ([operator](op-ishl)) |
| `i16x8.shr_s` | `0xFD 0x8C 0x01` | `[v128 i32] -> [v128]` | [validation](valid-vshiftop) | [execution](exec-vshiftop) ([operator](op-ishr)) |
| `i16x8.shr_u` | `0xFD 0x8D 0x01` | `[v128 i32] -> [v128]` | [validation](valid-vshiftop) | [execution](exec-vshiftop) ([operator](op-ishr)) |
| `i16x8.add` | `0xFD 0x8E 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-iadd)) |
| `i16x8.add_sat_s` | `0xFD 0x8F 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-iadd_sat)) |
| `i16x8.add_sat_u` | `0xFD 0x90 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-iadd_sat)) |
| `i16x8.sub` | `0xFD 0x91 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-isub)) |
| `i16x8.sub_sat_s` | `0xFD 0x92 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-isub_sat)) |
| `i16x8.sub_sat_u` | `0xFD 0x93 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-isub_sat)) |
| `f64x2.nearest` | `0xFD 0x94 0x01` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vunop) ([operator](op-fnearest)) |
| `i16x8.mul` | `0xFD 0x95 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-imul)) |
| `i16x8.min_s` | `0xFD 0x96 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-imin)) |
| `i16x8.min_u` | `0xFD 0x97 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-imin)) |
| `i16x8.max_s` | `0xFD 0x98 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-imax)) |
| `i16x8.max_u` | `0xFD 0x99 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-imax)) |
| (reserved) | `0xFD 0x9A 0x01` | | | |
| `i16x8.avgr_u` | `0xFD 0x9B 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-iavgr)) |
| `i16x8.extmul_low_i8x16_s` | `0xFD 0x9C 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vextbinop) | [execution](exec-vextbinop) ([operator](op-vextbinop)) |
| `i16x8.extmul_high_i8x16_s` | `0xFD 0x9D 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vextbinop) | [execution](exec-vextbinop) ([operator](op-vextbinop)) |
| `i16x8.extmul_low_i8x16_u` | `0xFD 0x9E 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vextbinop) | [execution](exec-vextbinop) ([operator](op-vextbinop)) |
| `i16x8.extmul_high_i8x16_u` | `0xFD 0x9F 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vextbinop) | [execution](exec-vextbinop) ([operator](op-vextbinop)) |
| `i32x4.abs` | `0xFD 0xA0 0x01` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vunop) ([operator](op-iabs)) |
| `i32x4.neg` | `0xFD 0xA1 0x01` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vunop) ([operator](op-ineg)) |
| (reserved) | `0xFD 0xA2 0x01` | | | |
| `i32x4.all_true` | `0xFD 0xA3 0x01` | `[v128] -> [i32]` | [validation](valid-vtestop) | [execution](exec-vtestop) |
| `i32x4.bitmask` | `0xFD 0xA4 0x01` | `[v128] -> [i32]` | [validation](valid-vbitmask) | [execution](exec-vbitmask) ([operator](op-ivbitmask)) |
| (reserved) | `0xFD 0xA5 0x01` | | | |
| (reserved) | `0xFD 0xA6 0x01` | | | |
| `i32x4.extend_low_i16x8_s` | `0xFD 0xA7 0x01` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vcvtop) |
| `i32x4.extend_high_i16x8_s` | `0xFD 0xA8 0x01` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vcvtop) |
| `i32x4.extend_low_i16x8_u` | `0xFD 0xA9 0x01` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vcvtop) |
| `i32x4.extend_high_i16x8_u` | `0xFD 0xAA 0x01` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vcvtop) |
| `i32x4.shl` | `0xFD 0xAB 0x01` | `[v128 i32] -> [v128]` | [validation](valid-vshiftop) | [execution](exec-vshiftop) ([operator](op-ishl)) |
| `i32x4.shr_s` | `0xFD 0xAC 0x01` | `[v128 i32] -> [v128]` | [validation](valid-vshiftop) | [execution](exec-vshiftop) ([operator](op-ishr)) |
| `i32x4.shr_u` | `0xFD 0xAD 0x01` | `[v128 i32] -> [v128]` | [validation](valid-vshiftop) | [execution](exec-vshiftop) ([operator](op-ishr)) |
| `i32x4.add` | `0xFD 0xAE 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-iadd)) |
| (reserved) | `0xFD 0xAF 0x01` | | | |
| (reserved) | `0xFD 0xB0 0x01` | | | |
| `i32x4.sub` | `0xFD 0xB1 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-isub)) |
| (reserved) | `0xFD 0xB2 0x01` | | | |
| (reserved) | `0xFD 0xB3 0x01` | | | |
| (reserved) | `0xFD 0xB4 0x01` | | | |
| `i32x4.mul` | `0xFD 0xB5 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-imul)) |
| `i32x4.min_s` | `0xFD 0xB6 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-imin)) |
| `i32x4.min_u` | `0xFD 0xB7 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-imin)) |
| `i32x4.max_s` | `0xFD 0xB8 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-imax)) |
| `i32x4.max_u` | `0xFD 0xB9 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-imax)) |
| `i32x4.dot_i16x8_s` | `0xFD 0xBA 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vextbinop) | [execution](exec-vextbinop) ([operator](op-vextbinop)) |
| `i32x4.extmul_low_i16x8_s` | `0xFD 0xBC 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vextbinop) | [execution](exec-vextbinop) ([operator](op-vextbinop)) |
| `i32x4.extmul_high_i16x8_s` | `0xFD 0xBD 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vextbinop) | [execution](exec-vextbinop) ([operator](op-vextbinop)) |
| `i32x4.extmul_low_i16x8_u` | `0xFD 0xBE 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vextbinop) | [execution](exec-vextbinop) ([operator](op-vextbinop)) |
| `i32x4.extmul_high_i16x8_u` | `0xFD 0xBF 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vextbinop) | [execution](exec-vextbinop) ([operator](op-vextbinop)) |
| `i64x2.abs` | `0xFD 0xC0 0x01` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vunop) ([operator](op-iabs)) |
| `i64x2.neg` | `0xFD 0xC1 0x01` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vunop) ([operator](op-ineg)) |
| (reserved) | `0xFD 0xC2 0x01` | | | |
| `i64x2.all_true` | `0xFD 0xC3 0x01` | `[v128] -> [i32]` | [validation](valid-vtestop) | [execution](exec-vtestop) |
| `i64x2.bitmask` | `0xFD 0xC4 0x01` | `[v128] -> [i32]` | [validation](valid-vbitmask) | [execution](exec-vbitmask) ([operator](op-ivbitmask)) |
| (reserved) | `0xFD 0xC5 0x01` | | | |
| (reserved) | `0xFD 0xC6 0x01` | | | |
| `i64x2.extend_low_i32x4_s` | `0xFD 0xC7 0x01` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vcvtop) |
| `i64x2.extend_high_i32x4_s` | `0xFD 0xC8 0x01` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vcvtop) |
| `i64x2.extend_low_i32x4_u` | `0xFD 0xC9 0x01` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vcvtop) |
| `i64x2.extend_high_i32x4_u` | `0xFD 0xCA 0x01` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vcvtop) |
| `i64x2.shl` | `0xFD 0xCB 0x01` | `[v128 i32] -> [v128]` | [validation](valid-vshiftop) | [execution](exec-vshiftop) ([operator](op-ishl)) |
| `i64x2.shr_s` | `0xFD 0xCC 0x01` | `[v128 i32] -> [v128]` | [validation](valid-vshiftop) | [execution](exec-vshiftop) ([operator](op-ishr)) |
| `i64x2.shr_u` | `0xFD 0xCD 0x01` | `[v128 i32] -> [v128]` | [validation](valid-vshiftop) | [execution](exec-vshiftop) ([operator](op-ishr)) |
| `i64x2.add` | `0xFD 0xCE 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-iadd)) |
| (reserved) | `0xFD 0xCF 0x01` | | | |
| (reserved) | `0xFD 0xD0 0x01` | | | |
| `i64x2.sub` | `0xFD 0xD1 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-isub)) |
| (reserved) | `0xFD 0xD2 0x01` | | | |
| (reserved) | `0xFD 0xD3 0x01` | | | |
| (reserved) | `0xFD 0xD4 0x01` | | | |
| `i64x2.mul` | `0xFD 0xD5 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-imul)) |
| `i64x2.eq` | `0xFD 0xD6 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-ieq)) |
| `i64x2.ne` | `0xFD 0xD7 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-ine)) |
| `i64x2.lt_s` | `0xFD 0xD8 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-ilt)) |
| `i64x2.gt_s` | `0xFD 0xD9 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-igt)) |
| `i64x2.le_s` | `0xFD 0xDA 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-ile)) |
| `i64x2.ge_s` | `0xFD 0xDB 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-ige)) |
| `i64x2.extmul_low_i32x4_s` | `0xFD 0xDC 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vextbinop) | [execution](exec-vextbinop) ([operator](op-vextbinop)) |
| `i64x2.extmul_high_i32x4_s` | `0xFD 0xDD 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vextbinop) | [execution](exec-vextbinop) ([operator](op-vextbinop)) |
| `i64x2.extmul_low_i32x4_u` | `0xFD 0xDE 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vextbinop) | [execution](exec-vextbinop) ([operator](op-vextbinop)) |
| `i64x2.extmul_high_i32x4_u` | `0xFD 0xDF 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vextbinop) | [execution](exec-vextbinop) ([operator](op-vextbinop)) |
| `f32x4.abs` | `0xFD 0xE0 0x01` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vunop) ([operator](op-fabs)) |
| `f32x4.neg` | `0xFD 0xE1 0x01` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vunop) ([operator](op-fneg)) |
| (reserved) | `0xFD 0xE2 0x01` | | | |
| `f32x4.sqrt` | `0xFD 0xE3 0x01` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vunop) ([operator](op-fsqrt)) |
| `f32x4.add` | `0xFD 0xE4 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-fadd)) |
| `f32x4.sub` | `0xFD 0xE5 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-fsub)) |
| `f32x4.mul` | `0xFD 0xE6 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-fmul)) |
| `f32x4.div` | `0xFD 0xE7 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-fdiv)) |
| `f32x4.min` | `0xFD 0xE8 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-fmin)) |
| `f32x4.max` | `0xFD 0xE9 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-fmax)) |
| `f32x4.pmin` | `0xFD 0xEA 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-fpmin)) |
| `f32x4.pmax` | `0xFD 0xEB 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-fpmax)) |
| `f64x2.abs` | `0xFD 0xEC 0x01` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vunop) ([operator](op-fabs)) |
| `f64x2.neg` | `0xFD 0xED 0x01` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vunop) ([operator](op-fneg)) |
| (reserved) | `0xFD 0xEE 0x01` | | | |
| `f64x2.sqrt` | `0xFD 0xEF 0x01` | `[v128] -> [v128]` | [validation](valid-vunop) | [execution](exec-vunop) ([operator](op-fsqrt)) |
| `f64x2.add` | `0xFD 0xF0 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-fadd)) |
| `f64x2.sub` | `0xFD 0xF1 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-fsub)) |
| `f64x2.mul` | `0xFD 0xF2 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-fmul)) |
| `f64x2.div` | `0xFD 0xF3 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-fdiv)) |
| `f64x2.min` | `0xFD 0xF4 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-fmin)) |
| `f64x2.max` | `0xFD 0xF5 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-fmax)) |
| `f64x2.pmin` | `0xFD 0xF6 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-fpmin)) |
| `f64x2.pmax` | `0xFD 0xF7 0x01` | `[v128 v128] -> [v128]` | [validation](valid-vbinop) | [execution](exec-vbinop) ([operator](op-fpmax)) |
