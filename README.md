# RED4ext for macOS

A script extender for REDengine 4 ([Cyberpunk 2077](https://www.cyberpunk.net)) — **macOS Apple Silicon Port**.

> **This is a macOS port** of [RED4ext](https://github.com/WopsS/RED4ext) by WopsS.  
> For Windows, use the [original repository](https://github.com/WopsS/RED4ext).

---

## Status

| Feature | Status |
|---------|--------|
| Library injection | ✅ Working |
| Function hooks (9/9) | ✅ Working via Frida |
| Symbol resolution | ✅ 21,332 symbols |
| Plugin system | ✅ Ready |
| REDscript compilation | ✅ Working |

**Platform:** macOS 12+ on Apple Silicon (M1/M2/M3)

---

## Quick Start

### One-Command Installation

```bash
# Clone with submodules
git clone --recursive https://github.com/memaxo/RED4ext-macos.git
cd RED4ext-macos

# Build and install
./scripts/macos_install.sh --build
```

### Launch the Game

```bash
cd "~/Library/Application Support/Steam/steamapps/common/Cyberpunk 2077"
./launch_red4ext.sh
```

---

## Requirements

- **macOS 12+** (Monterey or later)
- **Apple Silicon Mac** (M1/M2/M3)
- **Cyberpunk 2077** (Steam, macOS native version)
- **Xcode Command Line Tools**: `xcode-select --install`

---

## How It Works

This port uses **Frida Gadget** for function hooking because Apple Silicon enforces W^X (Write XOR Execute) protection that blocks traditional code patching.

```
launch_red4ext.sh
├── DYLD_INSERT_LIBRARIES=RED4ext.dylib:FridaGadget.dylib
├── FridaGadget loads red4ext_hooks.js
├── Frida installs 9 hooks via JIT trampolines
└── RED4ext loads plugins from red4ext/plugins/
```

See [docs/FRIDA_INTEGRATION.md](docs/FRIDA_INTEGRATION.md) for technical details.

---

## Documentation

| Document | Description |
|----------|-------------|
| [docs/MACOS_PORT.md](docs/MACOS_PORT.md) | Full installation guide |
| [docs/FRIDA_INTEGRATION.md](docs/FRIDA_INTEGRATION.md) | Frida hooking details |
| [BUILDING.md](BUILDING.md) | Build from source |
| [scripts/README.md](scripts/README.md) | Script reference |

---

## Project Structure

```
RED4ext-macos/
├── src/dll/                # Main loader library
│   └── Platform/           # macOS-specific code
├── scripts/
│   ├── macos_install.sh    # One-command setup
│   └── frida/              # Frida hooks
├── docs/                   # Documentation
└── deps/
    └── red4ext.sdk/        # SDK submodule (memaxo/RED4ext.SDK-macos)
```

---

## Related Projects

- [RED4ext](https://github.com/WopsS/RED4ext) — Original Windows version
- [RED4ext.SDK](https://github.com/WopsS/RED4ext.SDK) — Original SDK (Windows)
- [RED4ext.SDK-macos](https://github.com/memaxo/RED4ext.SDK-macos) — macOS SDK fork (used by this repo)

---

## License

MIT License — see [LICENSE.md](LICENSE.md)

## Acknowledgments

- **WopsS** — Original RED4ext author
- **Frida** — Dynamic instrumentation toolkit
- **Cyberpunk 2077 modding community**
