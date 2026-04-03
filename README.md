# BlackMagic

**One extension. Any language. Zero friction.**

BlackMagic is not a new language. It's a superset wrapper — a single `.bm` file format that routes to any real language underneath. You write Python, Rust, Zig, C++, Bash. BlackMagic is the envelope.

---

## The idea

```bash
# bm:lang=python
print("hello from python")
```

```bash
# bm:lang=rust
fn main() { println!("hello from rust"); }
```

```bash
# bm:lang=bash
echo "hello from bash"
```

Same extension. Same router. Any language.

---

## bmc — the router

`bmc` reads the `bm:lang=` tag and dispatches to the right compiler or interpreter.

```bash
bmc hello.bm        # auto-detects language, runs it
bmc --compile foo.bm  # compile where possible
bmc --check foo.bm    # syntax check only
```

No configuration. No project files. Just the tag at the top.

---

## bmpkg — package manager

Install any language runtime into the BlackMagic ecosystem:

```bash
bmpkg install python   # installs python3 + pip
bmpkg install rust     # installs rustc + cargo
bmpkg install zig      # installs zig toolchain
bmpkg install bash     # already there, confirms version
bmpkg install node     # javascript support
bmpkg list             # show installed languages
bmpkg update           # update all runtimes
```

**Learner path:** start with Python — zero setup, `bmpkg install python` and you're writing `.bm` files immediately. The package manager handles everything.

---

## Algorithm advisor

BlackMagic analyses your code and suggests complexity improvements.

```
$ bmc --advise sort.bm

  sort.bm:14  O(n²) bubble sort detected
  → suggestion: switch to std sort — O(n log n)
  → or: if input is bounded integers, counting sort — O(n)

  sort.bm:31  O(n) linear search in hot loop
  → suggestion: build index first — O(1) lookup after O(n) setup
```

Works across all languages. Catches the obvious ones — nested loops, linear searches in hot paths, repeated string concatenation. Not magic. Just pattern matching with complexity awareness.

---

## Security

- **No hidden network calls.** `bmc` and `bmpkg` never phone home.
- **No telemetry.** Ever.
- **Checksums on all downloaded runtimes.** `bmpkg` verifies before install.
- **Sandbox mode.** `bmc --sandbox foo.bm` runs with no filesystem or network access.
- **AGPL-3.0.** If you run this as a service, you open source your changes.

---

## For learners

BlackMagic is designed to grow with you:

| Stage | Language | Why |
|---|---|---|
| Start | Python | Zero friction. Reads like English. |
| Growing | Python + advisor | Algo advisor shows you where you're slow |
| Levelling | Bash | Systems thinking, scripting, real tools |
| Serious | Rust | Memory safety, real performance |
| Maximum | Zig / C++ | Full control, zero overhead |

You never rewrite your files. You change one line — the `bm:lang=` tag — and the same logic runs in a faster language. The structure stays. The envelope stays. The language upgrades.

---

## Quick start

```bash
# Install
curl -sSL https://raw.githubusercontent.com/Cythru/BlackMagic/main/install.sh | bash

# Write your first file
echo '# bm:lang=python
print("the void broadcasts")' > hello.bm

# Run it
bmc hello.bm

# Check complexity
bmc --advise hello.bm

# Install a new language runtime
bmpkg install rust
```

---

## File structure

```
.bm file
├── line 1: shebang (optional)
├── line 2: # bm:lang=<language>
├── line 3+: your real code, unchanged
```

That's it. No special syntax. No new keywords. Just your code with a routing tag.

---

## Supported languages

| Language | Status | Install |
|---|---|---|
| Python | stable | `bmpkg install python` |
| Bash | stable | built-in |
| Rust | stable | `bmpkg install rust` |
| Zig | stable | `bmpkg install zig` |
| C++ | stable | `bmpkg install cpp` |
| Node.js | beta | `bmpkg install node` |
| Go | beta | `bmpkg install go` |
| Any | via adapter | `bmpkg install <lang>` |

Adding a new language takes one adapter file. See `adapters/` for examples.

---

## Adding a language

Drop a file in `adapters/<lang>.bm`:

```bash
# bm:lang=bash
# adapter for <lang>
# BMC_ADAPTER=1
LANG_BIN=$(command -v <lang> || bmpkg install <lang> --silent)
exec "$LANG_BIN" "$1"
```

That's the full contract. `bmc` picks it up automatically. No registration. No config.

---

## Philosophy

BlackMagic is suckless in spirit. One job. No bloat. The complexity advisor exists because the best code is the code that doesn't need rewriting — but when it does, BlackMagic tells you why.

*"steal via masterpiece"*

---

## License

AGPL-3.0 — see LICENSE.

Network service use requires open source. No exceptions.

---

## Inspiration

A writer on moltbook called void_signal.

Favourite piece — *"basilisk // the doomed man vs the saved man."*
The choice architecture. The guide who breaks. The moral weight
sitting entirely on the reader.

*"music is an act of resistance. art is the need for perfection.
fiction lets the unsung stories exist."*

That line lives in this language somewhere.
Worth reading if interactive fiction is your thing.

---

**Luke Saunderson / Cythru — 2026**
