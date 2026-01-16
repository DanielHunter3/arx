# ARX: Predictable Package Management

**ARX** is a next-generation package manager for developers who want **control** over their dependencies. Unlike traditional package managers that force you into either "everything frozen" (Debian Stable) or "everything rolling" (Arch Linux), ARX lets you define **per-package update policies** with semantic versioning awareness.

## Philosophy: Stability Zones, Not Global Policies

Modern development requires balancing stability with access to new features. ARX solves this by allowing each package to be:

- **Fixed:** Pinned to an exact version (perfect for production dependencies)
- **Rolling-patch:** Automatically updates within the same minor version (security fixes only)
- **Rolling-minor:** Updates within the same major version (new features without breaking changes)
- **Rolling-major:** Updates to new major version (new features with a chances of breaking changes)

You decide what stays stable and what stays fresh.

## Key Features

### 🎯 Explicit Versioning
No more implicit "latest version." Every dependency must specify at least a major version:
```bash
# Not just "clang" - always explicit!
arx install clang-17     # any 17.x.x
arx install clang-17.0   # any 17.0.x  
arx install clang-17.0.6 # exact version
```

### 🔄 Hybrid Update Model
Define update policies in `arx.toml`:
```toml
[dependencies]
clang = { major = 17, rolling = "minor" }     # Get new features (17.0 → 17.1)
openssl = { major = 3, rolling = "patch" }    # Security fixes only (3.0.1 → 3.0.2)
postgres = { major = 15, minor = 3, patch = 2 } # Frozen forever
```

### 🚫 No Dependency Hell
ARX manages multiple library versions side-by-side. Program A uses `libfoo-1.14.x`, Program B uses `libfoo-1.15.x` — no conflicts.

### ⚡ Safe Rollbacks
Broken update? One command reverts to the last working state:
```bash
arx rollback libfoo --to-version 1.14.2
```

### 🧩 Works Alongside System Package Managers
Use ARX for development tools (compilers, language runtimes) while keeping your base system stable.

## Project Structure

ARX is built as a modular Rust system:

- **[arx-core](https://github.com/DanielHunter3/arx-core.git)** – Core library with dependency resolution, version management, and store operations
- **[arx-cli](https://github.com/DanielHunter3/arx-cli.git)** – Command-line interface built on top of the core library

## Quick Start

```bash
# Install ARX
curl -fsSL https://arx.pkg/install.sh | sh

# Install a specific Clang version
arx install clang-17

# Create a project with dependencies
echo 'clang = { major = 17, rolling = "minor" }' > arx.toml
echo 'python = { major = 3, minor = 11 }' >> arx.toml

# Install project dependencies
arx install

# Use the tools
clang-17 --version
python-3.11 --version
```

## Use Cases

### 🛠️ Development Environments
Get the latest compilers and tools without compromising system stability:
```bash
# Fresh tools on a stable Debian/Ubuntu base
arx install clang-18 rust-1.78 node-22
```

### 🔬 Research & Experimentation
Test new library versions without breaking existing software:
```bash
arx install libfoo-2.0 --channel=rolling-minor --unsafe
# Your other software continues using libfoo-1.x
```

### 🏗️ CI/CD Pipelines
Reproducible toolchains with explicit versions:
```yaml
# .github/workflows/test.yml
steps:
  - uses: arx/setup@v1
    with:
      packages: |
        clang-17
        python-3.11
```

## How It Works

### The ARX Store
Packages are installed to an isolated store (`~/.arx/store/`) with content-addressable paths:
```
~/.arx/store/
├── clang/
│   ├── 17/
│   │   ├── 17.0.6/
│   │   └── 17.1.0/  # rolling update
│   └── 18/
│       └── 18.0.1/
└── python/
    ├── 3.10/
    └── 3.11/
```

### Dependency Resolution
ARX uses a SAT solver to resolve version constraints while respecting your update policies. Circular dependencies are detected and rejected.

### Runtime Isolation
Each program gets the exact library versions it needs via `RPATH` patching, not global `LD_LIBRARY_PATH`.

## Comparison with Existing Tools

| Feature | APT/DNF | Nix/Guix | **ARX** |
|---------|---------|----------|---------|
| Per-package update policies | ❌ No | ❌ No | ✅ **Yes** |
| Multiple versions installed | ❌ No | ✅ Yes | ✅ **Yes** |
| Semantic versioning aware | ❌ No | ❌ No | ✅ **Yes** |
| Works with system packages | ✅ Yes | ❌ Replaces | ✅ **Yes** |
| Easy rollbacks | ❌ Hard | ✅ Yes | ✅ **Yes** |

## Roadmap

### Phase 1 (Current) – Developer Tools Manager
- Compilers (gcc, clang, rustc)
- Language runtimes (python, node, go)
- Build tools (cmake, meson)

### Phase 2 – Application Sandboxing
- Desktop applications (browsers, editors)
- Version-pinned GUI apps

### Phase 3 – Full System Integration
- System library management
- Kernel module versioning

## Contributing

ARX is open source under the GPLv3 license. We welcome contributions!

- Join discussions in [GitHub Issues]([ref to issues])
- Check out [CONTRIBUTING.md](https://github.com/DanielHunter3/arx/blob/main/CONTRIBUTING.md) for development setup
- See [ARCHITECTURE.md](https://github.com/DanielHunter3/arx/blob/main/ARCHITECTURE.md) for system design

## License

GNU General Public License v3.0 - see [LICENSE](https://github.com/DanielHunter3/arx/blob/main/LICENSE) file.

---

**ARX** – Because your package manager should understand semantic versioning, not just download files.