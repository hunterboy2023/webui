# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

### GNU Make (Primary)
```sh
make          # Build release (static + dynamic)
make debug    # Build debug with symbols
make clean    # Clean build artifacts
```

### CMake
```sh
cmake -B build -S . && cmake --build build
```

### Zig Build
```sh
zig build
zig build examples
```

### Windows (MSVC)
```sh
nmake         # Release
nmake debug   # Debug
```

### Bridge Generation (TypeScript -> C Header)
```sh
cd bridge
build.bat     # Windows
./build.sh    # Linux/macOS
```

### Build Options
- `WEBUI_USE_TLS=1` - Enable TLS/SSL support (requires OpenSSL)
- `CC=clang` - Use Clang instead of GCC (Linux/macOS)

## Architecture Overview

WebUI is a cross-platform library that uses any web browser or WebView as GUI for applications written in C/C++ or other languages via bindings.

### Core Components

```
webui/
├── src/
│   ├── webui.c           # Main library implementation
│   ├── civetweb/         # Embedded HTTP/WebSocket server (vendored)
│   ├── bridge/           # Frontend<->Backend bridge (TypeScript -> C header)
│   └── webview/          # Native WebView implementations
├── include/
│   ├── webui.h           # Public C API
│   └── webui.hpp         # C++ wrapper
└── examples/
    ├── C/                # C examples
    └── C++/              # C++ examples
```

### Key Design Patterns

1. **Protocol-based Communication**: Binary protocol with 8-byte header:
   - Signature (1 byte) + Token (4 bytes) + ID (2 bytes) + Command (1 byte) + Data

2. **Command Types**: Commands like `WEBUI_CMD_JS`, `WEBUI_CMD_CLICK`, `WEBUI_CMD_CALL_FUNC` handle frontend-backend communication via WebSocket.

3. **Bridge Generation**: The TypeScript bridge (`bridge/webui.ts`) is transpiled to JavaScript via ESBuild, then converted to a C header (`bridge/webui_bridge.h`) via `js2c.py`.

### Platform-Specific Code

- **Windows**: WebView2, Winsock2, Ole32, User32
- **Linux**: GTK WebView, pthread, inotify
- **macOS**: WKWebView (Objective-C), Cocoa, WebKit frameworks

### Build System Notes

- GNU Makefile (`GNUmakefile`) is the primary build system
- CMake and Zig build systems are also maintained
- Bridge must be rebuilt when `webui.ts` changes: run `build.bat` (Windows) or `build.sh` (Linux/macOS) in `bridge/` directory
