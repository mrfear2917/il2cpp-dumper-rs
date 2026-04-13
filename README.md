# 🦀 IL2CPP Dumper — Rust Edition

A blazing-fast, cross-platform IL2CPP binary dumper written in Rust. Full rewrite from the original C# [Il2CppDumper](https://github.com/Perfare/Il2CppDumper) with significant performance improvements and additional features.

---

## 🏆 Rust vs C# vs Python — Feature Comparison

| Feature | [Python](https://github.com/springmusk026/Il2CppDumper-Python) | [C#](https://github.com/Perfare/Il2CppDumper) | **Rust (This)** |
|---------|:---:|:---:|:---:|
| **dump.cs generation** | ⚠️ Basic (no properties) | ✅ Full | ✅ Full |
| **DummyDLL generation** | ❌ No | ✅ Yes | ✅ Yes (parallel) |
| **il2cpp.h struct gen** | ⚠️ Basic | ✅ Full | ✅ Full |
| **script.json** | ✅ Yes | ✅ Yes | ✅ Yes |
| **stringliteral.json** | ✅ Yes | ✅ Yes | ✅ Yes |
| **Split Dump Per Type (DiffableCS)** | ❌ No | ❌ No | ✅ **Yes (parallel)** |
| **Variable-width indices (v39/Unity 6)** | ❌ No | ❌ No | ✅ **Yes** |
| **Auto-XOR Metadata Decryption** | ❌ No | ❌ No | ✅ **Yes** |
| **Latest Unity Formats (v104, v106)** | ❌ No | ❌ No | ✅ **Yes** |
| **Assembly name in dump.cs** | ❌ No | ❌ No | ✅ **Yes** |
| **Unity version detection** | ❌ No | ❌ No | ✅ **Auto-detect** |
| **Inline Disassembly (ARM64/ARM32/x86/x64)** | ❌ No | ❌ No | ✅ **Yes** |
| **Control Flow Graph (CFG) Analysis** | ❌ No | ❌ No | ✅ **Yes** |
| **Metadata Annotations (strings, types, vtable)** | ❌ No | ❌ No | ✅ **Yes** |
| **Semantic Variable Tracking** | ❌ No | ❌ No | ✅ **Yes** |
| **Fat Mach-O (Universal)** | ❌ No | ✅ Yes | ✅ Yes |
| **WASM (WebGL)** | ✅ Yes | ✅ Yes | ✅ Yes |
| **Dump file support** | ❌ No | ✅ Yes | ✅ Yes (+ ELF reload) |
| **v27+ ImageBase fix** | ❌ No | ✅ Yes | ✅ Yes |
| **Parallel I/O (`rayon`)** | ❌ No | ❌ No | ✅ **Yes (DummyDLL & DiffableCS)** |
| **Auto-numbered output dirs** | ❌ No | ❌ No | ✅ **Dump0/, Dump1/...** |
| **Execution timer** | ❌ No | ❌ No | ✅ **Done! (8.35s)** |
| **Cross-platform binary** | ⚠️ Needs Python | ⚠️ Needs .NET | ✅ **Standalone** |
| **Web UI** | ✅ Flask | ❌ No | ❌ No |
| **GUI** | ❌ No | ✅ WinForms | ✅ **Jetpack Compose (Android)** |
| **Embeddable as library** | ❌ No | ❌ No | ✅ **Rust crate / JNI** |

### ⚡ Performance Comparison

| Phase | Python | C# | **Rust** |
|-------|:---:|:---:|:---:|
| Metadata loading | ~3.7s | ~2s | **~0.5s** |
| Binary loading | ~5.2s | ~3s | **~0.8s** |
| Search & Init | ~5.4s | ~2s | **~0.3s** |
| dump.cs | ~14s | ~5s | **~2s** |
| Struct generation | ~6.4s | ~5s | **~3.5s** |
| DummyDLL | ❌ N/A | ~3s | **~1.5s** |
| **Total** | **~35s** | **~20s** | **~8.35s** |

> **4x faster than Python, 2.4x faster than C#** — on the same binary.

---

## ✨ Features

### Core Dumping
- **dump.cs** — Full C# class/method/field/property decompilation with RVA/VA/Offset addresses and assembly (image) names
- **Inline Disassembly** — Optional per-method native assembly output embedded directly in dump.cs method bodies
- **DiffableCs (Split Dump Per Type)** — Automatically splits classes into beautifully formatted, individual `.cs` files sorted by Namespace. Powered by `rayon` parallel processing for instant SSD write speeds.
- **script.json** — Method addresses and signatures for IDA/Ghidra scripting
- **il2cpp.h** — C struct definitions for native analysis
- **stringliteral.json** — All string literal values and indices
- **DummyDLL** — Reconstructed .NET assemblies for dnSpy/ILSpy analysis (parallelized bounding)

### Disassembly Engine
- **Multi-Architecture** — ARM64 (`yaxpeax-arm`), ARM32, x86/x64 (`iced-x86`)
- **Control Flow Graph (CFG)** — Automatic basic block detection, branch targets, loop back-edges, and `if (condition)` reconstruction
- **Metadata Annotations** — String literals, type info, method references, field references, and vtable slot resolution resolved via `ADRP+LDR` patterns
- **Semantic Variable Tracking** — Maps registers to parameter names (e.g., `X0` → `this`, `X1` → `arg0`) using IL2CPP ABI and method metadata
- **Configurable** — Toggle hex bytes, field names, annotations, and CFG independently; cap max instructions per method

### Supported Formats

| Platform | Format | Status |
|----------|--------|:---:|
| Android | ELF32 / ELF64 | ✅ |
| iOS | Mach-O / Fat Mach-O | ✅ |
| macOS | Mach-O / Fat Mach-O | ✅ |
| Windows | PE32 / PE64 | ✅ |
| Nintendo Switch | NSO | ✅ |
| WebGL | WASM | ✅ |

### IL2CPP Version Support
- **v16 – v39** (Unity 5.3 through Unity 6 inclusive)
- **Variable-Width Indices** natively supported for v39/Unity 6 formats
- **Latest Unity Formats:** Native support for the newest undocumented metadata versions like `v104` and `v106`.
- **Automatic Metadata Decryption:** On-the-fly automated 1-byte and 4-byte XOR key sniffing and decryption against the `AF 1B B1 FA` magic header for quickly dumping protected `.dat` files.
- Manual version override via config

### Search Strategies
- **SectionHelper** — Format-aware section scanning with ELF-optimized search order
- **Symbol Search** — ELF/Mach-O symbol table lookup
- **ARM32 Search** — Dedicated ARM32 binary pattern matching
- **\_\_mod\_init\_func** — Mach-O initializer function analysis
- **Manual mode** — Enter addresses manually as fallback

---

## 📦 Installation

### From Source
```bash
git clone https://github.com/rodroidmods/il2cpp-dumper-rs.git
cd il2cpp-dumper-rs/il2cpp_dumper
cargo build --release
```

The binary will be at `target/release/il2cpp_dumper` (or `.exe` on Windows).

### Cross-Compilation
```bash
# Linux
cargo build --release --target x86_64-unknown-linux-gnu

# macOS
cargo build --release --target aarch64-apple-darwin

# Android (via cross)
cross build --release --target aarch64-linux-android
```

---

## 🔧 Usage

```bash
# Basic usage
il2cpp_dumper <il2cpp-binary> <global-metadata.dat> [output-dir]

# Examples
il2cpp_dumper libil2cpp.so global-metadata.dat
il2cpp_dumper GameAssembly.dll global-metadata.dat ./output
il2cpp_dumper UnityFramework global-metadata.dat
```

### Example Output
```
Initializing metadata...
Metadata Version: 31
Initializing IL2CPP binary...
Unity Version: 2022.3.62f2
Detected ELF64 format
IL2CPP Version: 31
Searching...
CodeRegistration : 0x44e5ff8
MetadataRegistration : 0x465a328
Dumping...
dump.cs generated
Generating struct...
script.json, il2cpp.h, stringliteral.json generated
Generating dummy dll...
Dummy dll files generated
Done! (8.35s)
```

---

## ⚙️ Configuration

Create a `config.json` in the working directory (or use `--config`):

```json
{
  "ForceIl2CppVersion": false,
  "ForceVersion": 29.0,
  "ForceDump": false,
  "NoRedirectedPointer": false,
  "GenerateStruct": true,
  "GenerateDummyDll": true,
  "DummyDllAddToken": true,
  "dumpDisassembly": false,
  "dumpDisassemblyHexBytes": true,
  "dumpDisassemblyFieldNames": true,
  "dumpDisassemblyAnnotations": true,
  "dumpDisassemblyCfg": true,
  "maxDisassemblyInstructions": 512
}
```

---

## 🏗️ Architecture

```
il2cpp_dumper/src/
├── main.rs              # CLI, format detection, orchestration
├── config.rs            # Configuration handling
├── formats/             # Binary format parsers
│   ├── elf.rs           # ELF32/64 (Android, Linux)
│   ├── pe.rs            # PE32/64 (Windows)
│   ├── macho.rs         # Mach-O + Fat Mach-O (iOS, macOS)
│   ├── nso.rs           # NSO (Nintendo Switch)
│   └── wasm.rs          # WebAssembly (WebGL)
├── il2cpp/              # IL2CPP structures and metadata
│   ├── base.rs          # Il2Cpp main struct
│   ├── metadata.rs      # Metadata parser
│   └── structures.rs    # IL2CPP type definitions
├── search/              # Registration search algorithms
│   └── section_helper.rs
├── executor/            # IL2CPP type resolution engine
├── disassembler/        # Multi-arch disassembly engine
│   ├── mod.rs           # CFG analysis, annotations, formatting
│   ├── arm.rs           # ARM64/ARM32 decoder (yaxpeax)
│   └── x86.rs           # x86/x64 decoder (iced-x86)
└── output/              # Output generators
    ├── decompiler.rs              # dump.cs + inline disassembly
    ├── struct_generator.rs        # script.json, il2cpp.h
    └── dummy_assembly_generator.rs # DummyDLL (parallel)
```

---

## � License

MIT

## 🙏 Credits

- [Perfare/Il2CppDumper](https://github.com/Perfare/Il2CppDumper) — Original C# implementation
- [springmusk026/Il2CppDumper-Python](https://github.com/springmusk026/Il2CppDumper-Python) — Python port
- [dotnetdll](https://crates.io/crates/dotnetdll) — .NET DLL generation crate
- [rayon](https://crates.io/crates/rayon) — Parallel processing

---

> **⚠️ Disclaimer**: This tool is for educational and research purposes. Respect game developers' rights and terms of service.
