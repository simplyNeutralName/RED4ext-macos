# Building RED4ext

## Platforms

| Platform | Architecture | Status |
|----------|--------------|--------|
| Windows | x86-64 | ✅ Supported |
| macOS | ARM64 (Apple Silicon) | ✅ Supported |

---

## macOS Build Instructions

### Prerequisites

- **macOS 12+** (Monterey or later)
- **Xcode Command Line Tools**
- **CMake 3.23+**
- **Python 3.8+** (for address generation scripts)

```bash
# Install prerequisites
xcode-select --install
brew install cmake

# Verify installations
cmake --version
python3 --version
```

### Clone Repository

**Important:** The macOS port uses a modified SDK. Use the `--recursive` flag to fetch all submodules:

```bash
git clone --recursive https://github.com/memaxo/RED4ext-macos.git
cd RED4ext-macos

# If you forgot --recursive, or submodules are out of date:
git submodule update --init --recursive
```

> **Note:** The `deps/red4ext.sdk` submodule should point to `memaxo/RED4ext.SDK-macos` which contains macOS compatibility changes.

### 2. Configure and Build

```bash
# Create build directory
mkdir -p build && cd build

# Configure
cmake ..

# Build (use all CPU cores)
make -j$(sysctl -n hw.ncpu)
```

### 3. Build Outputs

After successful build:
```
build/
├── libs/
│   └── libred4ext.dylib          # Main loader library
├── cyberpunk2077_symbols.json     # Symbol mappings (if generated)
└── cyberpunk2077_addresses.json   # Address database (if generated)
```

## Generating Address Files

The address files are required for RED4ext to hook game functions.

### Generate Symbol Mapping

```bash
cd scripts

python3 generate_symbol_mapping.py \
    "/path/to/Cyberpunk2077.app/Contents/MacOS/Cyberpunk2077" \
    --output ../build/cyberpunk2077_symbols.json \
    --demangle-cache ../build/demangle_cache.sqlite3
```

### Generate Address Database

```bash
python3 generate_addresses.py \
    "/path/to/Cyberpunk2077.app/Contents/MacOS/Cyberpunk2077" \
    --manual manual_addresses_template.json \
    --output ../build/cyberpunk2077_addresses.json
```

See [scripts/README.md](scripts/README.md) for detailed documentation.

## Installation

### Method 1: Manual Installation

1. Copy files to game directory:
```bash
GAME_DIR="$HOME/Library/Application Support/Steam/steamapps/common/Cyberpunk 2077"

# Create directories
mkdir -p "$GAME_DIR/red4ext/bin/x64"
mkdir -p "$GAME_DIR/red4ext/plugins"

# Copy library
cp build/libs/libred4ext.dylib "$GAME_DIR/red4ext/"

# Copy address files
cp build/cyberpunk2077_symbols.json "$GAME_DIR/red4ext/bin/x64/"
cp build/cyberpunk2077_addresses.json "$GAME_DIR/red4ext/bin/x64/"
```

2. Launch with library injection:
```bash
DYLD_INSERT_LIBRARIES="$GAME_DIR/red4ext/libred4ext.dylib" \
    "$GAME_DIR/Cyberpunk2077.app/Contents/MacOS/Cyberpunk2077"
```

### Method 2: Using Launcher Script

```bash
cp red4ext_launcher.sh "$GAME_DIR/"
chmod +x "$GAME_DIR/red4ext_launcher.sh"
"$GAME_DIR/red4ext_launcher.sh"
```

## Troubleshooting

### Library not loading
- Verify `DYLD_INSERT_LIBRARIES` path is correct
- Check System Integrity Protection (SIP) status
- Ensure library is code-signed: `codesign -s - libred4ext.dylib`

### Symbols not resolving
- Verify `cyberpunk2077_symbols.json` is in `red4ext/bin/x64/`
- Check logs for symbol resolution errors
- Regenerate symbol mapping if game was updated

### Build errors
- Ensure all submodules are initialized
- Check CMake version: `cmake --version` (need 3.23+)
- Try clean build: `rm -rf build && mkdir build && cd build && cmake ..`

## Development

### Debug Build

```bash
cmake -DCMAKE_BUILD_TYPE=Debug ..
make -j$(sysctl -n hw.ncpu)
```

### Running Tests

```bash
# From build directory
ctest --output-on-failure
```

### Code Signing (for distribution)

```bash
codesign -s "Developer ID Application: Your Name" \
    --options runtime \
    build/libs/libred4ext.dylib
```

## Platform Differences from Windows

| Feature | Windows | macOS |
|---------|---------|-------|
| Binary format | PE (DLL) | Mach-O (dylib) |
| Architecture | x86-64 | ARM64 |
| Function hooking | Detours | fishhook |
| Library loading | LoadLibrary | dlopen |
| Symbol resolution | GetProcAddress | dlsym |
| Thread-local storage | `__declspec(thread)` | `thread_local` |

---

## Windows Build Instructions

### Prerequisites

- **Windows 10/11**
- **Visual Studio 2022** with C++ workload
- **CMake 3.23+**

### Clone and Build

```bash
git clone --recursive https://github.com/WopsS/RED4ext.git
cd RED4ext
mkdir build && cd build
cmake .. -G "Visual Studio 17 2022"
cmake --build . --config Release
```

---

## Submodule Structure

The project uses git submodules for dependencies:

| Submodule | Windows URL | macOS URL |
|-----------|-------------|-----------|
| `deps/red4ext.sdk` | `WopsS/RED4ext.SDK` | `memaxo/RED4ext.SDK-macos` |
| `deps/fishhook` | (not used) | `facebook/fishhook` |
| `deps/spdlog` | `gabime/spdlog` | same |
| `deps/fmt` | `fmtlib/fmt` | same |

**For macOS users:** Ensure your `deps/red4ext.sdk` points to the macOS-compatible fork:

```bash
# Check current submodule URL
git config --file=.gitmodules submodule.deps/red4ext.sdk.url

# Should show: https://github.com/memaxo/RED4ext.SDK-macos
```

---

## See Also

- [docs/MACOS_PORT.md](docs/MACOS_PORT.md) - macOS installation guide
- [scripts/README.md](scripts/README.md) - Address generation tools
- [scripts/REVERSE_ENGINEERING_GUIDE.md](scripts/REVERSE_ENGINEERING_GUIDE.md) - Finding function addresses
- [docs/porting/](docs/porting/) - Port development history
