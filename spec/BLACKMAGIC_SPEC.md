# BlackMagic Language Specification
## Version 0.1 — The Universal Product Maker

---

## 0. Preface

BlackMagic is the gold-standard systems programming language.
It synthesises the greatest ideas from C++, Rust, Zig, Koka, Pony,
Austral, Vale, Nim, D, Swift, Julia, Odin, Idris, and Mojo into a
single, coherent, maximally-powerful language.

**Core Tenets:**
1. **Modularity First** — every construct is a composable module
2. **Zero-Cost Abstractions** — you pay only for what you use
3. **Maximum Performance** — SIMD-native, comptime-everywhere, no GC
4. **Total Safety** — memory, resource, concurrency, and effect safety
5. **Radical Transparency** — no hidden allocations, no hidden control flow
6. **Expressive Power** — dependent types, algebraic effects, linear types
7. **Universal Target** — bare metal, WASM, GPU, embedded, cloud

---

## 1. Lexical Structure

### 1.1 Source Encoding
BlackMagic source files use UTF-8. The extension is `.bm`.

### 1.2 Whitespace
Spaces, tabs, and newlines are insignificant except for string literals.

### 1.3 Comments
```bm
// Single-line comment
/* Multi-line
   comment */
/// Doc comment (attached to next item)
//! Inner doc comment (documents enclosing module)
```

### 1.4 Keywords
```
fn       struct    enum      trait     impl      use
module   let       var       const     comptime  if
else     match     while     for       in        return
break    continue  defer     linear    actor     effect
handle   resume    spawn     async     await     move
owned    where     type      pub       priv      extern
unsafe   asm       volatile  and       or        not
true     false     null      self      super     dispatch
receive  never     as        is        mod       soa
region   cap       borrow    copy      drop      static
lazy     pure      total     atomic    packed    align
```

### 1.5 Primitive Types
```
i8   i16   i32   i64   i128   isize   int
u8   u16   u32   u64   u128   usize   uint
f16  f32   f64   f128
bool  char  str   void  never  byte
```

### 1.6 SIMD Types (first-class)
```
vec2<T>   vec4<T>   vec8<T>   vec16<T>   vec32<T>   vec64<T>
mat2<T>   mat3<T>   mat4<T>
tensor<T, [DIMS...]>
```

### 1.7 Literals
```bm
// Integer
42          // decimal
0xFF_AA_BB  // hexadecimal
0o755       // octal
0b1010_1010 // binary
42u32       // typed integer
42_000_000  // underscores for readability

// Float
3.14
3.14f32
1.0e-9
0x1.8p+1    // hex float

// String
"hello world"
"escaped \n \t \r \\ \""
"interpolated {expr} value"
r"raw \n no escapes"
b"byte string"

// Char
'a'  '\n'  '\u{1F600}'

// Array / Slice
[1, 2, 3]
[0; 256]   // 256 zeros

// Struct literal
Point { x: 1.0, y: 2.0 }

// Enum variant
.Some(42)
.None
```

---

## 2. Module System

BlackMagic's module system is the foundation. Everything is a module.

### 2.1 Module Declaration
```bm
// Declare module — creates a namespace
module geometry {
    pub struct Point { pub x: f64, pub y: f64 }
    pub struct Rect  { pub min: Point, pub max: Point }

    pub fn distance(a: &Point, b: &Point) -> f64 {
        let dx = a.x - b.x
        let dy = a.y - b.y
        (dx*dx + dy*dy).sqrt()
    }
}
```

### 2.2 Parametric Modules (Functors)
```bm
// Module parameterized by a type — compile-time, zero-cost
module Vec(T: type) {
    pub struct Vec {
        data: owned *T,
        len:  usize,
        cap:  usize,
    }

    pub fn new() -> Vec { Vec { data: null, len: 0, cap: 0 } }

    pub fn push(self: &mut Vec, val: T) { /* ... */ }
    pub fn pop(self: &mut Vec) -> ?T    { /* ... */ }
    pub fn get(self: &Vec, idx: usize) -> &T { /* ... */ }
}

// Instantiate
let IntVec = Vec(i32)
let StrVec = Vec(str)
var v: IntVec::Vec = IntVec::new()
v.push(42)
```

### 2.3 Module Interfaces (Signatures)
```bm
// Define a module interface
interface Container {
    type Item
    fn new() -> Self
    fn insert(self: &mut Self, item: Self::Item) -> ()
    fn contains(self: &Self, item: &Self::Item) -> bool
    fn len(self: &Self) -> usize
}

// Implement interface
module HashSet(T: type) implements Container where T: Hash + Eq {
    pub type Item = T
    pub struct HashSet { /* ... */ }
    pub fn new() -> HashSet { /* ... */ }
    pub fn insert(self: &mut HashSet, item: T) { /* ... */ }
    pub fn contains(self: &HashSet, item: &T) -> bool { /* ... */ }
    pub fn len(self: &HashSet) -> usize { /* ... */ }
}
```

### 2.4 Module Import
```bm
use std::io
use std::io::{println, readln}
use std::collections::Vec as BmVec
use geometry::*          // glob import (use sparingly)
use super::utils         // parent module
```

### 2.5 Hot-Reloadable Modules
```bm
// Mark a module as hot-reloadable
#[hot_reload]
module game_logic {
    pub fn update(state: &mut GameState, dt: f64) { /* ... */ }
}
// The runtime can swap this module at runtime without restarting.
```

### 2.6 Module Visibility
```
pub      — visible everywhere
pub(mod) — visible within module and submodules
priv     — visible only within the declaring item (default)
pub(pkg) — visible within the package
```

---

## 3. Type System

### 3.1 Primitive Types
All primitive types have well-defined sizes and no padding.
Integer overflow panics in debug, wraps in release (configurable).

```bm
let a: i32  = -2_147_483_648
let b: u64  = 18_446_744_073_709_551_615u64
let c: f64  = 1.7976931348623157e+308f64
let d: bool = true
let e: char = '🔥'
```

### 3.2 Structs
```bm
struct Point {
    x: f64,
    y: f64,
}

// Packed struct (no padding)
#[packed]
struct Header {
    magic:   u32,
    version: u16,
    flags:   u8,
}

// Aligned struct
#[align(64)]
struct CacheLine {
    data: [u8; 64],
}

// Generic struct
struct Pair<A, B> {
    first:  A,
    second: B,
}

// SOA struct — fields stored as separate arrays (AoS → SoA)
#[soa]
struct Particle {
    x:    f32,
    y:    f32,
    z:    f32,
    mass: f32,
    vel:  vec3<f32>,
}
// Particle::soa([1000]) creates SoA layout for 1000 particles
```

### 3.3 Enums (Algebraic Data Types)
```bm
enum Option<T> {
    Some(T),
    None,
}

enum Result<T, E> {
    Ok(T),
    Err(E),
}

enum Shape {
    Circle { radius: f64 },
    Rect   { width: f64, height: f64 },
    Triangle { a: Point, b: Point, c: Point },
}

// C-style enum
enum Color: u8 {
    Red   = 0,
    Green = 1,
    Blue  = 2,
}

// Tagged union with no overhead
#[repr(C)]
enum CResult {
    Ok(i32),
    Err(u32),
}
```

### 3.4 Traits
```bm
trait Display {
    fn fmt(self: &Self) -> str
}

trait Add<Rhs = Self> {
    type Output
    fn add(self: Self, rhs: Rhs) -> Self::Output
}

// Trait with default implementations
trait Container {
    type Item
    fn len(self: &Self) -> usize
    fn is_empty(self: &Self) -> bool {  // default impl
        self.len() == 0
    }
}

// Trait bounds (where clause)
fn largest<T>(list: &[T]) -> &T
where T: Ord + Copy
{
    let mut max = &list[0]
    for item in list {
        if item > max { max = item }
    }
    max
}

// Impl trait (opaque types)
fn make_adder(x: i32) -> impl Fn(i32) -> i32 {
    fn(y) => x + y
}
```

### 3.5 Linear Types (Resource Safety)
```bm
// Linear type — must be used exactly once (no implicit copy, no implicit drop)
linear struct File {
    fd: i32,
}

impl File {
    pub fn open(path: str) -> Result<File, IOError> !IO { /* ... */ }
    pub fn close(self) -> void !IO { /* consumes, must call */ }
    pub fn read(self: &File, buf: &mut [u8]) -> usize !IO { /* ... */ }
    pub fn write(self: &mut File, data: &[u8]) -> usize !IO { /* ... */ }
}

fn use_file() !IO {
    let f = File::open("/tmp/x.txt")?
    defer f.close()          // guaranteed execution at scope exit
    let mut buf = [0u8; 512]
    let n = f.read(&mut buf)?
    // f.close() called here automatically by defer
}
```

### 3.6 Ownership & Borrowing
```bm
// Ownership rules (Rust-inspired, simplified syntax)
//   — Every value has a single owner
//   — When the owner goes out of scope, the value is dropped
//   — Ownership can be transferred (moved)

fn takes_ownership(s: owned String) { /* s dropped here */ }
fn borrows(s: &String) -> usize { s.len() }
fn mutably_borrows(s: &mut String) { s.push('!') }

// Borrow rules enforced by compiler:
//   — At most one &mut reference OR any number of & references
//   — References must not outlive the owned value

// Regions: extend borrow checker with arena lifetimes
fn process(data: &[u8]) -> &[u8] in 'arena {
    // all borrows within 'arena live at most as long as 'arena
    let result = transform(data)
    result
}
```

### 3.7 Reference Capabilities (Pony-inspired)
```bm
// Six reference capabilities for fine-grained aliasing control:
//
//   iso  — isolated: no other references exist anywhere (sendable, mutable)
//   val  — immutable value: all references see same immutable data
//   ref  — mutable reference: only one mutable ref, read from same thread
//   box  — read-only reference: readable from any context
//   trn  — transition: can become val or ref
//   tag  — opaque: only identity, no data access (for actors)

iso struct Unique { data: [u8; 4096] }
val struct Config { host: str, port: u16 }
```

### 3.8 Refinement Types
```bm
// Encode invariants directly in types
type PositiveInt = i32 where self > 0
type BoundedFloat = f64 where 0.0 <= self && self <= 1.0
type NonEmptyStr = str where self.len() > 0
type Index(len: usize) = usize where self < len

fn divide(a: i32, b: i32 where b != 0) -> i32 {
    a / b  // no division-by-zero possible
}

fn get_element<T>(arr: &[T], idx: Index(arr.len())) -> &T {
    &arr[idx]  // no bounds check needed — proven safe
}
```

### 3.9 Dependent Types (Light)
```bm
// Types depending on runtime values
type Matrix(rows: usize, cols: usize) = [[f64; cols]; rows]

fn mat_mul(
    a: &Matrix(M, K),
    b: &Matrix(K, N),
) -> Matrix(M, N) {
    // compiler verifies dimension compatibility
}

// Dimension errors are compile-time
// let bad = mat_mul(&m3x4, &m3x4)  // ERROR: inner dims must match
```

### 3.10 SIMD Types
```bm
// SIMD vectors are first-class
let a: vec4<f32> = vec4(1.0, 2.0, 3.0, 4.0)
let b: vec4<f32> = vec4(5.0, 6.0, 7.0, 8.0)
let c: vec4<f32> = a + b              // vectorised add
let d: f32 = (a * b).hadd()          // horizontal add → dot product
let e: vec4<f32> = a.shuffle(b, [0,2,1,3])

// SIMD operations
fn lerp_v8(a: vec8<f32>, b: vec8<f32>, t: vec8<f32>) -> vec8<f32> {
    a + t * (b - a)
}

// Tensor types for ML
fn softmax(x: tensor<f32, [N]>) -> tensor<f32, [N]> {
    let exp_x = x.map(f32::exp)
    exp_x / exp_x.sum()
}
```

---

## 4. Functions

### 4.1 Function Declaration
```bm
// Basic function
fn add(a: i32, b: i32) -> i32 {
    a + b   // implicit return (last expression)
}

// Named return
fn divmod(a: i32, b: i32) -> (quot: i32, rem: i32) {
    (a / b, a % b)
}

// Functions are values
let f: fn(i32) -> i32 = add.curry(1)

// Closures
let multiplier = fn(x: i32) -> i32 { x * 3 }
let captured = {
    let factor = 10
    fn(x: i32) -> i32 { x * factor }  // captures factor
}
```

### 4.2 Effect Annotations
```bm
// ! introduces effects
fn read_file(path: str) -> Result<str, IOError> !IO { /* ... */ }
fn random_i32() -> i32 !Rand { /* ... */ }
fn might_fail() -> i32 !Fail<str> { /* ... */ }

// Multiple effects
fn complex() -> i32 !IO + Rand + Fail<str> { /* ... */ }

// Pure function — no effects allowed
pure fn compute(x: i32) -> i32 { x * x + 1 }

// Total function — always terminates
total pure fn fib(n: u64) -> u64 {
    match n {
        0 | 1 => n,
        n => fib(n-1) + fib(n-2),
    }
}
```

### 4.3 Comptime Functions
```bm
// Comptime: runs at compile time
comptime fn pow(base: u64, exp: u64) -> u64 {
    if exp == 0 { 1 }
    else { base * pow(base, exp - 1) }
}

const TWO_TO_16: u64 = pow(2, 16)  // = 65536

// Comptime type generation (Zig-style anytype)
comptime fn make_ring_buffer(T: type, CAP: usize) -> type {
    struct {
        data:  [T; CAP],
        read:  usize,
        write: usize,
        count: usize,

        fn new() -> This { /* ... */ }
        fn push(self: &mut This, val: T) -> bool { /* ... */ }
        fn pop(self: &mut This) -> ?T { /* ... */ }
    }
}

const AudioBuffer = make_ring_buffer(f32, 4096)
const EventQueue  = make_ring_buffer(Event, 256)

// Comptime conditional (zero-cost branching on type)
comptime fn simd_sum<T>(slice: &[T]) -> T {
    if comptime T == f32 && comptime std::target::has_avx2() {
        simd_sum_f32_avx2(slice)
    } else if comptime T == f64 {
        simd_sum_f64(slice)
    } else {
        scalar_sum(slice)
    }
}
```

### 4.4 UFCS (Uniform Function Call Syntax)
```bm
// Any function can be called as a method
fn double(x: i32) -> i32 { x * 2 }

42.double()          // = double(42)
[1,2,3].len()        // = len([1,2,3])

// Enables pipeline style
data
    .filter(fn(x) x > 0)
    .map(fn(x) x * x)
    .sum()
    .double()
```

### 4.5 Multiple Dispatch
```bm
// Method resolution on ALL argument types
dispatch fn collide(a: Shape, b: Shape) -> Collision {
    /* fallback */
}

dispatch fn collide(a: Circle, b: Circle) -> Collision {
    /* circle-circle */
}

dispatch fn collide(a: Circle, b: Rect) -> Collision {
    /* circle-rect */
}

dispatch fn collide(a: Rect, b: Rect) -> Collision {
    /* rect-rect */
}

// Compiler generates dispatch table — no virtual overhead
```

---

## 5. Effect System

BlackMagic uses algebraic effects — the cleanest solution to side effects.

### 5.1 Defining Effects
```bm
effect IO {
    fn stdin_read(n: usize) -> [u8]
    fn stdout_write(data: &[u8]) -> usize
    fn stderr_write(data: &[u8]) -> usize
}

effect Rand {
    fn next_u64() -> u64
    fn next_bytes(buf: &mut [u8]) -> ()
}

effect Async {
    fn yield_now() -> ()
    fn sleep(ms: u64) -> ()
}

effect Fail<E> {
    fn fail(err: E) -> never  // diverges: never returns
}

effect Log {
    fn log(level: LogLevel, msg: str) -> ()
}

effect State<S> {
    fn get() -> S
    fn put(s: S) -> ()
    fn modify(f: fn(S) -> S) -> ()
}
```

### 5.2 Effect Polymorphism
```bm
// Function polymorphic over effects
fn map_vec<T, U, E>(
    v: &[T],
    f: fn(&T) -> U !E
) -> Vec<U> !E {
    let mut result = Vec::new()
    for item in v {
        result.push(f(item))
    }
    result
}
// E is inferred from f — composes transparently
```

### 5.3 Effect Handlers
```bm
fn main() {
    // handle expression runs the computation and intercepts effects
    let pi = handle monte_carlo(1_000_000) with {
        // intercept Rand
        Rand::next_u64() => {
            let r = xorshift64()
            resume(r)          // resume computation with value
        },
        Rand::next_bytes(buf) => {
            fill_random(buf)
            resume(())
        },
        // intercept Log
        Log::log(level, msg) => {
            eprintln!("[{level}] {msg}")
            resume(())
        },
    }
    println("π ≈ {pi:.6}")
}
```

---

## 6. Concurrency

### 6.1 Actor Model
```bm
// Actor: isolated state, message-passing concurrency
actor Counter {
    var count: i64 = 0

    receive {
        msg::Increment(n: i64) => { count += n },
        msg::Decrement(n: i64) => { count -= n },
        msg::Reset              => { count = 0 },
        msg::Get(reply: Chan<i64>) => { reply.send(count) },
    }
}

fn main() !IO + Spawn {
    let c = spawn Counter::new()
    c.send(msg::Increment(10))
    c.send(msg::Decrement(3))

    let (tx, rx) = channel::<i64>()
    c.send(msg::Get(tx))
    println("count = {rx.recv()}")  // 7
}
```

### 6.2 Structured Concurrency
```bm
fn parallel_work() !Spawn {
    // All spawned tasks scoped to this block
    async scope {
        spawn fetch_data("https://api.example.com")
        spawn process_local_files()
        spawn update_ui()
        // All tasks joined here — no dangling tasks
    }
}
```

### 6.3 Channels & STM
```bm
// Typed channels
let (tx, rx) = channel::<Message>()
let (tx2, rx2) = bounded_channel::<Event>(256)

// Software Transactional Memory
fn transfer(from: &TVar<i64>, to: &TVar<i64>, amount: i64) {
    atomically {
        let src = from.read()
        if src < amount { retry() }  // block until condition met
        from.write(src - amount)
        to.write(to.read() + amount)
    }
}
```

---

## 7. Memory Management

### 7.1 Ownership (Default)
```bm
// Stack allocation by default
let x: i32 = 42

// Heap allocation — explicit
let boxed: owned i32 = Box::new(42)

// Array on heap
let arr: owned [f64] = Vec::with_len(1024)
```

### 7.2 Allocators (Explicit, No Hidden Allocations)
```bm
// Every allocating function takes an allocator
fn process(data: &[u8], alloc: Allocator) -> owned [u8] alloc {
    let result = alloc.alloc([u8], data.len())
    /* ... */
    result
}

// Common allocators
std::alloc::heap       // system malloc/free
std::alloc::arena(N)   // bump allocator, free all at once
std::alloc::stack(N)   // stack allocator
std::alloc::pool::<T>(N) // pool allocator for T
std::alloc::null       // fails all allocations (for testing)
```

### 7.3 Region-Based Memory
```bm
// Region: a named lifetime scope for grouped allocations
fn process_request(req: Request) -> Response {
    region tmp {
        // All allocations within this region freed at end of block
        let parsed = parse(req.body, tmp)
        let validated = validate(parsed, tmp)
        build_response(validated)  // result doesn't borrow tmp
    }
}
```

### 7.4 RAII & defer
```bm
fn managed() !IO {
    let lock = Mutex::lock(&GLOBAL_MUTEX)
    defer lock.release()    // runs on any exit path

    let file = File::open("/tmp/x")?
    defer file.close()

    // both file and lock released in reverse declaration order
}
```

---

## 8. Pattern Matching

### 8.1 Match Expression
```bm
// Exhaustive — compiler errors on missing cases
match value {
    0         => println("zero"),
    1..=9     => println("single digit"),
    n if n < 0 => println("negative: {n}"),
    n          => println("large: {n}"),
}

// Destructuring
match point {
    Point { x: 0.0, y } => println("on y-axis at {y}"),
    Point { x, y: 0.0 } => println("on x-axis at {x}"),
    Point { x, y }      => println("({x}, {y})"),
}

// Enum matching
match shape {
    .Circle { radius }           => area = PI * radius * radius,
    .Rect   { width, height }    => area = width * height,
    .Triangle { a, b, c }        => area = triangle_area(a, b, c),
}

// Or patterns
match code {
    200 | 201 | 204 => Status::Success,
    400 | 422       => Status::ClientError,
    500..=599       => Status::ServerError,
    _               => Status::Unknown,
}
```

### 8.2 if-let / while-let
```bm
if let .Some(val) = option_value {
    println("got: {val}")
}

while let .Ok(event) = event_queue.pop() {
    handle_event(event)
}
```

### 8.3 Destructuring Assignment
```bm
let (x, y, z) = point_to_tuple(p)
let Point { x, y } = some_point
let [first, second, ..rest] = array_value
```

---

## 9. Error Handling

### 9.1 Result Type
```bm
fn parse_int(s: str) -> Result<i32, ParseError> {
    /* ... */
}

// ? operator — propagate errors
fn read_config() -> Result<Config, Error> {
    let text = File::open("config.bm")?.read_to_string()?
    let cfg  = parse_config(text)?
    Ok(cfg)
}
```

### 9.2 Effect-Based Errors
```bm
// More composable than Result: Fail effect
fn divide(a: f64, b: f64) -> f64 !Fail<DivisionByZero> {
    if b == 0.0 { Fail::fail(.DivisionByZero) }
    a / b
}

// Convert effect to Result
let result: Result<f64, DivisionByZero> = catch {
    divide(10.0, 0.0)
}
```

---

## 10. Macros & Metaprogramming

### 10.1 Declarative Macros
```bm
macro vec!($($expr:expr),*) {
    {
        let mut v = Vec::new()
        $(v.push($expr);)*
        v
    }
}

let nums = vec![1, 2, 3, 4, 5]
```

### 10.2 Procedural Macros (AST-level)
```bm
// Derive macro
#[derive(Debug, Clone, Eq, Hash)]
struct Point { x: i32, y: i32 }

// Custom derive
#[derive(Serialize, Deserialize)]
struct Config {
    host: str,
    port: u16,
}

// Attribute macro
#[memoize]
fn expensive(n: u64) -> u64 { /* ... */ }

// Custom DSL macro
let query = sql!{
    SELECT name, age FROM users
    WHERE age > 18
    ORDER BY name ASC
}
```

### 10.3 Comptime Reflection
```bm
// Inspect types at compile time
comptime fn print_fields(T: type) -> void {
    for field in comptime T::fields() {
        println("field: {field.name} : {field.type}")
    }
}

comptime fn generate_serializer(T: type) -> fn(&T) -> str {
    /* generate code based on T's structure */
}
```

---

## 11. Inline Assembly
```bm
fn cpuid(leaf: u32) -> (u32, u32, u32, u32) {
    asm volatile {
        "cpuid"
        in("eax")  leaf,
        in("ecx")  0u32,
        out("eax") eax: u32,
        out("ebx") ebx: u32,
        out("ecx") ecx: u32,
        out("edx") edx: u32,
        clobbers("memory"),
    }
    (eax, ebx, ecx, edx)
}

fn rdtsc() -> u64 {
    asm volatile {
        "rdtsc; shl rdx, 32; or rax, rdx"
        out("rax") ret: u64,
        clobbers("rdx"),
    }
    ret
}
```

---

## 12. GPU Kernels
```bm
// GPU kernels as first-class functions
#[kernel(threads = [16, 16, 1])]
fn matrix_multiply(
    a: &tensor<f32, [M, K]>  @global,
    b: &tensor<f32, [K, N]>  @global,
    c: &mut tensor<f32, [M, N]> @global,
) {
    let row = @thread.x + @block.x * @blockdim.x
    let col = @thread.y + @block.y * @blockdim.y
    if row >= M || col >= N { return }

    var sum: f32 = 0.0
    for k in 0..K {
        sum += a[row, k] * b[k, col]
    }
    c[row, col] = sum
}

// Launch kernel
let result = matrix_multiply.launch([M, N], a, b, &mut c)
```

---

## 13. Foreign Function Interface
```bm
// C interop
extern "C" {
    fn malloc(size: usize) -> *mut void
    fn free(ptr: *mut void) -> void
    fn printf(fmt: *const u8, ...) -> i32
}

// C++ interop
extern "C++" {
    type std::string
    fn std::to_string(n: i32) -> std::string
}

// Export for C
#[export("bm_add")]
pub extern "C" fn add(a: i32, b: i32) -> i32 { a + b }

// WASM export
#[wasm_export]
pub fn run(data: &[u8]) -> Vec<u8> { /* ... */ }
```

---

## 14. Standard Library (Overview)

```
std::io        — I/O (files, stdin/stdout, network)
std::fs        — filesystem
std::net       — networking (TCP, UDP, HTTP)
std::alloc     — allocators (heap, arena, pool, stack)
std::collections — Vec, HashMap, HashSet, BTreeMap, VecDeque, etc.
std::sync      — Mutex, RwLock, AtomicXxx, channels, STM
std::thread    — threading primitives
std::process   — subprocess management
std::fmt       — formatting and printing
std::math      — math functions (all SIMD-optimised)
std::simd      — explicit SIMD intrinsics
std::crypto    — cryptographic primitives
std::time      — clocks, timers, durations
std::env       — environment variables, args
std::path      — path manipulation
std::regex     — regular expressions
std::json      — JSON parsing/serialisation
std::proto     — protobuf-style serialisation
std::gpu       — GPU kernel dispatch
std::test      — testing framework
std::bench     — benchmarking
std::profile   — built-in profiling hooks
std::log       — structured logging via effects
std::rand      — random number generation via effects
std::comptime  — compile-time utilities
std::target    — target architecture queries (comptime)
std::meta      — reflection and metaprogramming
```

---

## 15. Build System

BlackMagic's build system is defined in `build.bm`:
```bm
// build.bm — project manifest
project {
    name:    "myapp",
    version: "1.0.0",
    target:  .native,      // or .wasm, .arm64, .riscv64, .gpu
}

dependency("std",        version: "0.1")
dependency("serde",      version: "1.0")
dependency("networking", version: "0.5")

build_step("generate") {
    comptime run_script("generate.bm")
}

test_suite("unit") {
    files: ["tests/unit/**/*.bm"]
    filter: .debug
}
```

---

## 16. Performance Philosophy

- **Comptime everything possible**: generics, specialisation, constant folding
- **Monomorphisation**: all generic code specialised per type, no boxing
- **SOA by default for bulk data**: use `#[soa]` structs for data-parallel code
- **SIMD-native**: `vec<T,N>` types compiled to hardware SIMD
- **No GC, no runtime**: deterministic performance, predictable latency
- **LTO by default in release**: whole-program optimisation
- **Profile-guided optimisation**: `bmc build --pgo=collect / --pgo=use`
- **Zero hidden allocations**: every heap allocation is explicit
- **Link-time dead-code elimination**: unused functions never emitted

---

*BlackMagic Language Specification v0.1 — The Universal Product Maker*
*All rights reserved — Abe's personal copy — DO NOT DISTRIBUTE*
