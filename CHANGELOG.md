# BlackMagic Language — CHANGELOG

## v0.2.0 — Juicer Edition

### New: Full Compiler Pipeline

**Type Checker** (`src/typecheck.rs`)
- Hindley-Milner Algorithm W with constraint solving
- Rank-1 polymorphism (let-generalisation)
- Effect inference — propagates through higher-order functions
- Overload resolution for multiple dispatch functions
- Dependent type checking (array bounds, refinement predicates)
- `check_file()` → `check_function()` → `infer_expr()` pipeline

**Borrow Checker** (`src/borrow.rs`)
- Region-based borrow tracking
- Aliasing XOR mutation enforcement (`&T` ⊕ `&var T`)
- Linear type consumption verification (must use exactly once)
- Loan expiry at end of region (scope-based NLL approximation)
- Move tracking: use-after-move detected and reported

**Effect Checker** (`src/effects.rs`)
- Verifies all performed effects are declared in function signature
- `handle ... with` block coverage verification
- `resume` outside-handler detection
- Effect propagation through closures and HOFs
- Unused-declared-effect warning

**MIR** (`src/mir.rs`)
- SSA-form Control Flow Graph (Static Single Assignment)
- Phi nodes at join points
- Three-address instructions (`dest = op(a, b)`)
- SIMD rvalues: `SimdSplat`, `SimdBinOp`
- Effect perform instructions
- Terminators: Goto, Branch, Switch, Return, Call, Drop, Panic, Unreachable
- `MirBuilder` for lowering typed AST → MIR
- Pretty-printer for `--emit=mir`

**Optimiser** (`src/opt.rs`)
- `PassManager`: `standard()` and `sicko()` modes
- Fixpoint iteration (runs until no pass changes the MIR)
- Passes:
  - Constant Propagation: folds `1 + 2` → `3` at compile time
  - Dead Code Elimination: removes unused SSA values
  - Common Subexpression Elimination: value numbering
  - Strength Reduction: `x * 1` → `x`, `x / 1` → `x`
  - Tail Call Elimination: self-recursive tail calls → `goto entry`
  - Copy Propagation: eliminates redundant copies
  - Block Merging: fuses linear block chains
  - Loop Invariant Code Motion (skeleton, requires dominator tree)
  - Inliner (skeleton, threshold configurable)
  - Vectorisation Hints (skeleton, annotates vectorisable loops)

**Compile-Time Evaluator** (`src/comptime.rs`)
- Evaluates `comptime fn` at compile time with caching
- Evaluates `comptime { expr }` blocks inline
- Supported: integer/float/bool/string arithmetic, array construction, conditionals, recursion
- Recursion depth limit: 256
- `CValue` type for all comptime-representable values

**Code Generation** (`src/codegen/`)
- `mod.rs`: target dispatch (C | LLVM | WASM)
- `c.rs`: C99 transpiler
  - Goto-based CFG lowering (one label per basic block)
  - Effect performs → vtable callbacks
  - SIMD → GCC vector extensions
  - Aggregate constructors → compound literals
- `llvm.rs`: LLVM IR emitter
  - SSA maps directly to LLVM IR
  - Phi nodes, alloca entry block
  - Lifetime intrinsics for linear types
  - FMA, vector types for SIMD rvalues
- `wasm.rs`: WASM binary encoder
  - Direct binary format (no intermediate text)
  - LEB128 encoding
  - Structured control flow (block/loop/if)
  - SIMD via wasm128 proposal

**Rich Diagnostics** (`src/diag.rs`)
- Ariadne-style source excerpts with squiggles
- Severity levels: Hint, Warning, Error, Fatal
- ANSI colour output for terminals
- JSON output for IDE/LSP integration
- Error codes (E0001–E9999)
- Notes and help suggestions per error
- Multi-annotation support (primary + secondary spans)

### New: Standard Library Packages

**`stdlib/juicer/mod.bm`**
- `#[juicer]` / `juicer::optimise(f)` — apply full pass pipeline
- `juicer::vectorise(f)` — auto-vectorise loop body
- `juicer::no_bounds_check(f)` — SMT-verified bounds elimination
- `juicer::unroll<N>(f)` — compile-time loop unrolling
- `juicer::force_inline(f)` / `no_inline(f)`
- `juicer::pack_struct(T)` — struct field reordering for minimal padding
- `juicer::choose_layout(T, hint)` — AOS vs SOA selection
- `juicer::rdtsc()` — RDTSC nanosecond timer
- `juicer::bench(f, iters)` — compile-time benchmarking

**`stdlib/sicko/mod.bm`**
- `#[sicko]` / `sicko::activate()` — maximum aggression mode
- `sicko::stream_store(ptr, val)` — non-temporal cache-bypass writes
- `sicko::stream_load(ptr)` — non-temporal cache-bypass reads
- `sicko::prefetch(ptr, write, locality)` — software prefetch
- `sicko::avx2_load/store` — raw AVX2 256-bit I/O
- `sicko::fma(a, b, c)` — forced fused multiply-add
- `sicko::hsum(v)` — horizontal SIMD sum
- `sicko::fast_inv_sqrt(x)` — Quake III 1/√x method
- `sicko::min_branchless/max_branchless` — CMOV-friendly min/max
- `sicko::clz/ctz/popcount` — hardware bit manipulation
- `sicko::cas/fetch_add` — lock-free atomics
- `sicko::make_ring_buffer(T, CAP)` — comptime stack ring buffer
- `sicko::small_vec(T, N)` — inline storage small vector
- `sicko::benchmark(name, f, iters)` — median + p99 harness

### Improvements

- `src/error.rs`: added `ComptimeError { span, msg }` variant
- `src/error.rs`: added `Span::default()` (zero-position span)
- `src/lib.rs`: exports all new compiler modules

### Tests

- 63 tests passing (27 lexer + 36 parser)
- Type checker, borrow checker, effect checker, MIR, opt, comptime — integration tests planned for v0.3.0

---

## v0.1.0 — Genesis

- Full lexer (80+ keywords, SIMD types, string interpolation)
- Complete AST for all language constructs
- Recursive descent parser with error recovery
- Type system (HM base + linear + refinement + effects + SIMD)
- Module system (parametric modules, hot-reload, topo sort)
- CLI: `build`, `check`, `run`, `lex`, `parse`, `fmt`
- 7 example files (hello, ownership, comptime, effects, modules, SIMD, benchmarks)
- 63/63 tests passing
