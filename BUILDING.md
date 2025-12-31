# Building RED4ext for macOS

> **This is the macOS port.** For Windows, see [WopsS/RED4ext](https://github.com/WopsS/RED4ext).

---

## Prerequisites

- **macOS 12+** (Monterey or later)
- **Apple Silicon** (M1/M2/M3)
- **Xcode Command Line Tools**
- **CMake 3.23+**
- **Python 3.8+** (for address generation)

```bash
# Install prerequisites
xcode-select --install
brew install cmake

# Verify
cmake --version   # Need 3.23+
python3 --version # Need 3.8+
```

---

## Quick Build

```bash
# Clone with SDK submodule
git clone --recursive https://github.com/memaxo/RED4ext-macos.git
cd RED4ext-macos

# Build
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make -j$(sysctl -n hw.ncpu)

# Install to game
cd ..
./scripts/macos_install.sh
```

> **Important:** Use `--recursive` to fetch the macOS SDK from `memaxo/RED4ext.SDK-macos`.

---

## Build Outputs

```
build/
└── libs/
    └── RED4ext.dylib    # Main loader library
```

The install script copies this plus Frida Gadget to the game directory.

---

## Manual Installation

If not using `macos_install.sh`:

```bash
GAME_DIR="$HOME/Library/Application Support/Steam/steamapps/common/Cyberpunk 2077"

# Create directories
mkdir -p "$GAME_DIR/red4ext/bin/x64"
mkdir -p "$GAME_DIR/red4ext/plugins"
mkdir -p "$GAME_DIR/red4ext/logs"

# Copy library
cp build/libs/RED4ext.dylib "$GAME_DIR/red4ext/"

# Copy address files
cp scripts/cyberpunk2077_addresses.json "$GAME_DIR/red4ext/bin/x64/"

# Set up Frida Gadget
./scripts/setup_frida_gadget.sh
```

---

## Submodule Structure

| Submodule | URL | Purpose |
|-----------|-----|---------|
| `deps/red4ext.sdk` | `memaxo/RED4ext.SDK-macos` | macOS-compatible SDK |
| `deps/fishhook` | `facebook/fishhook` | Symbol rebinding |
| `deps/spdlog` | `gabime/spdlog` | Logging |
| `deps/fmt` | `fmtlib/fmt` | Formatting |

If submodules are missing:
```bash
git submodule update --init --recursive
```

---

## Regenerating Address Files

When the game updates, you may need to regenerate addresses:

```bash
cd scripts

# Generate symbol mappings (21,332 symbols)
python3 generate_symbol_mapping.py \
    "/path/to/Cyberpunk2077.app/Contents/MacOS/Cyberpunk2077" \
    --output cyberpunk2077_symbols.json

# Generate address database (9 hooks)
python3 generate_addresses.py \
    "/path/to/Cyberpunk2077.app/Contents/MacOS/Cyberpunk2077" \
    --manual manual_addresses_template.json \
    --output cyberpunk2077_addresses.json
```

---

## Debug Build

```bash
cmake -DCMAKE_BUILD_TYPE=Debug ..
make -j$(sysctl -n hw.ncpu)
```

Logs appear in `<game>/red4ext/logs/`.

---

## Troubleshooting

### Build fails with "SDK not found"
```bash
git submodule update --init --recursive
```

### Library not loading
- Check DYLD path: `echo $DYLD_INSERT_LIBRARIES`
- Sign the library: `codesign -s - build/libs/RED4ext.dylib`

### Hooks not working
- Verify Frida Gadget is installed: `ls <game>/red4ext/FridaGadget.dylib`
- Check Frida config exists: `ls <game>/red4ext/FridaGadget.config`

---

## See Also

- [docs/MACOS_PORT.md](docs/MACOS_PORT.md) — Installation guide
- [docs/FRIDA_INTEGRATION.md](docs/FRIDA_INTEGRATION.md) — How Frida hooks work
- [scripts/README.md](scripts/README.md) — Script documentation
