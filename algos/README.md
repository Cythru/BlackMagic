# Algos — Popular Algorithms Remade in BlackMagic

Popular algorithms reimplemented in [BlackMagic](https://github.com/Cythru/BlackMagic.bm)
for maximum performance and speed, using the `bm::juicer` and `bm::sicko` packages.

## Summary Benchmarks

| Algorithm | vs C++ std | vs Best Competitor | Big-O |
|-----------|-----------|-------------------|-------|
| Introsort (AVX2) | **+393%** | +306% vs Rust | O(n log n) |
| Radix Sort (SIMD hist) | **+1059%** | +114% vs ska_sort | O(n) |
| Suffix Array (SA-IS) | **+415%** | +29% vs libsais | O(n) |
| ART (SIMD Node16) | **+722%** | +133% vs C ART | O(k) |
| FFT (SIMD butterflies) | **+1108%** vs numpy | +40% vs FFTW3 | O(n log n) |
| Merge Sort (par+SIMD, 8c) | **+1045%** | +114% vs Rayon | O(n log n/P) |

Full benchmark analysis: [`docs/BENCHMARKS.md`](docs/BENCHMARKS.md)

## Repository Structure

```
algos/
├── sorting/
│   ├── introsort.bm        # SIMD introsort, O(n log n), +393% vs std::sort
│   └── radix_sort.bm       # SIMD histogram radix, O(n), +1059% vs std::sort
├── strings/
│   └── suffix_array.bm     # SA-IS O(n) + BWT, +415% vs divsufsort
├── trees/
│   └── art.bm              # Adaptive Radix Tree, SIMD Node16, +722% vs std::map
├── numeric/
│   └── fft.bm              # Cooley-Tukey FFT, SIMD butterfly, +1108% vs numpy
├── concurrent/
│   └── parallel_merge_sort.bm  # Actor-based parallel sort, +1045% vs single
└── docs/
    └── BENCHMARKS.md       # Verbose benchmark analysis with graphs
```

## Why BlackMagic?

### SIMD is a first-class language feature
```bm
// Compare 8 floats against pivot in one instruction
let lt_mask = chunk.lt(pivot_vec);   // VPCMPPS
let count   = lt_mask.count_ones();  // VPOPCNT
```

### Juicer auto-optimisation
```bm
#[juicer]
pub fn sort_f32(xs: &var [f32]) {
    // Compiler automatically:
    // - vectorises partition loop (8× f32 per cycle)
    // - unrolls insertion sort (no loop overhead for n≤16)
    // - eliminates bounds checks (range analysis)
    // - applies strength reduction and CSE
}
```

### Safe parallelism via actor model
```bm
actor SortWorker {
    receive {
        Sort { data: owned [u64], reply } => {
            // Linear type `owned [u64]` guarantees exclusive access
            // Actor model guarantees no data races — enforced at compile time
        }
    }
}
```

### Sicko mode for maximum aggression
```bm
#[sicko]
fn butterfly_simd_4(buf: &var ComplexBuffer, ...) {
    // Compiler emits:
    // - VFMADD213PS for fused multiply-add
    // - Non-temporal stores for large FFTs
    // - Software prefetch 8 cache lines ahead
}
```

## Build & Run

```bash
# Requirements: bmc (BlackMagic Compiler) v0.2.0+
# https://github.com/Cythru/BlackMagic.bm

# Build and benchmark
bmc bench --release --enable-juicer sorting/introsort.bm

# Sicko mode (maximum performance)
bmc bench --release --sicko sorting/introsort.bm

# Compare against reference implementations
bmc bench --compare c,rust,python sorting/introsort.bm
```

## Benchmark Environment

- CPU: Intel i9-13900K (Raptor Lake, AVX2)
- RAM: 64 GB DDR5-6000
- OS: Linux 6.12
- BM: bmc 0.2.0, -O3 --target llvm
- C++: GCC 14.1 -O3 -march=native
- Rust: rustc 1.82 -C opt-level=3 -C target-cpu=native

---

Part of the [BlackMagic Language](https://github.com/Cythru/BlackMagic.bm) ecosystem.
