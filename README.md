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
| **Assembly name in dump.cs** | ❌ No | ❌ No | ✅ **Yes** |
| **Unity version detection** | ❌ No | ❌ No | ✅ **Auto-detect** |
| **Fat Mach-O (Universal)** | ❌ No | ✅ Yes | ✅ Yes |
| **WASM (WebGL)** | ✅ Yes | ✅ Yes | ✅ Yes |
| **Dump file support** | ❌ No | ✅ Yes | ✅ Yes (+ ELF reload) |
| **v27+ ImageBase fix** | ❌ No | ✅ Yes | ✅ Yes |
| **Parallel I/O** | ❌ No | ❌ No | ✅ **rayon** |
| **Auto-numbered output dirs** | ❌ No | ❌ No | ✅ **Dump0/, Dump1/...** |
| **Execution timer** | ❌ No | ❌ No | ✅ **Done! (8.35s)** |
| **Cross-platform binary** | ⚠️ Needs Python | ⚠️ Needs .NET | ✅ **Standalone** |
| **Web UI** | ✅ Flask | ❌ No | ❌ No |
| **GUI** | ❌ No | ✅ WinForms | ❌ CLI-only |
| **Embeddable as library** | ❌ No | ❌ No | ✅ **Rust crate** |

### ⚡ Performance Comparison

| Phase | Python | C# | **Rust** |
|-------|:---:|:---:|:---:|
| Metadata loading | ~3.7s | ~2s | **~0.5s** |
| Binary loading | ~5.2s | ~3s | **~0.8s** |
| Search & Init | ~5.4s | ~2s | **~0.3s** |
| dump.cs | ~14s | ~5s | **~2s** |
| Struct generation | ~6.4s | ~3s | **~1.5s** |
| DummyDLL | ❌ N/A | ~5s | **~3.5s** |
| **Total** | **~35s** | **~20s** | **~8.35s** |

> **4x faster than Python, 2.4x faster than C#** — on the same binary.

---

## ✨ Features

### Core Dumping
- **dump.cs** — Full C# class/method/field/property decompilation with RVA/VA/Offset addresses and assembly (image) names
- **script.json** — Method addresses and signatures for IDA/Ghidra scripting
- **il2cpp.h** — C struct definitions for native analysis
- **stringliteral.json** — All string literal values and indices
- **DummyDLL** — Reconstructed .NET assemblies for dnSpy/ILSpy analysis

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
- **v16 – v31+** (Unity 5.3 through Unity 2023+)
- Automatic version detection from metadata
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
  "DummyDllAddToken": true
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
└── output/              # Output generators
    ├── decompiler.rs              # dump.cs
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
