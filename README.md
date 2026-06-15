# IDO & DDS Compiler, Decompiler and Converter

A cross-platform desktop GUI tool for working with proprietary `.ido` game asset files and `.dds` textures. The application provides decompilation, compilation, and format conversion capabilities through an intuitive dark-themed Electron interface backed by Python processing scripts.

## Overview

This tool processes `.ido` files -- a custom container format used by games (likely Korean MMOs, given EUC-KR encoding). The format consists of a binary header followed by zlib-compressed content that may contain XML data, texture data (DDS, TGA, BMP, PNG), Gamebryo state blocks, or shop database records. The application can inspect, extract, repack, and convert these assets.

The original compiler/decompiler/converter logic was implemented in Rust by [ultimatuuuum](https://github.com/ultimatuuuum). This project reimplements the functionality in Python with an Electron GUI shell.

## Features

### Decompilation
- Decompile `.ido` files into human-readable XML with EUC-KR to UTF-8 decoding
- Extract embedded textures (DDS, TGA, BMP, PNG) with accompanying `.meta` header files for recompilation
- Extract Gamebryo state block files (binary passthrough)
- Parse and dump shop database records to CSV format (fixed 456-byte record structure)
- Embedded IDO headers preserved as XML comments for round-trip compilation

### Compilation
- Compile XML files back into `.ido` format with zlib compression
- Compile binary textures back into `.ido` format using stored `.meta` headers
- Support for both single-file and batch folder compilation
- Automatic header extraction from XML comments or external `.meta` files

### DDS/PNG Conversion
- Convert DDS textures to PNG (uncompressed RGBA8 extraction)
- Convert PNG images back to DDS (uncompressed RGBA8 format)
- Graceful fallback with manual DDS header construction when system DDS plugin is unavailable

## Technology Stack

| Layer | Technology |
|-------|-----------|
| Desktop Shell | Electron (Node.js) |
| UI | HTML5, CSS3, Vanilla JavaScript |
| Backend Processing | Python 3 |
| Image Processing | Pillow (PIL) |
| Compression | zlib (Python standard library) |
| Build System | electron-builder |

### Architecture

The application follows a two-tier architecture:

- **Electron Main Process** (`main.js`): Manages the application window (1200x800, dark theme), handles native file dialogs via IPC, spawns Python child processes, and relays real-time log output to the renderer.
- **Electron Renderer** (`renderer.js`): Provides a tabbed interface (Decompile, Compile, DDS Converter) with file selection, state management, color-coded console logging, and status updates.
- **Python Backend** (`ido_tool.py`, `dds_converter.py`): Performs all binary parsing, compression, encoding, and image conversion. Communicates with Electron via structured JSON on stdout with a final `RESULT:{...}` line.

## Prerequisites

- Node.js v16 or higher
- Python 3.x
- pip (Python package manager)

## Setup

```bash
# Install Node.js dependencies
npm install

# Install Python dependencies
pip install -r requirements.txt

# Verify Python is accessible
python --version
```

## Usage

### Running the Application

```bash
# Standard launch
npm start

# Development mode with logging enabled
npm run dev
```

### Decompiling IDO Files

1. Click the **Decompile** tab
2. Click **Browse** to select an `.ido` file
3. Choose or confirm the output location
4. Click **Start Decompilation**
5. Monitor progress in the console panel

Output files are determined by the detected content type:
- XML data: Saved as `.xml` with embedded header comment
- DDS/TGA/BMP/PNG textures: Saved with appropriate extension plus `.meta` header file
- Gamebryo state blocks: Saved as `.gb` (raw binary)
- Shop databases: Saved as `.csv`

### Compiling IDO Files

1. Click the **Compile** tab
2. Select input via **File** (single file) or **Folder** (batch mode)
3. Choose the output `.ido` path
4. Click **Start Compilation**

Compilation requires the original IDO header, sourced from:
- Embedded `<!-- IDO HEADER: ... -->` comment in XML files
- Accompanying `.meta` file for binary textures

### Converting DDS Images

1. Click the **DDS Converter** tab
2. Toggle between **DDS to PNG** or **PNG to DDS** mode
3. Browse for the input file
4. Choose the output location
5. Click **Convert**

Note: The converter handles uncompressed RGBA8 format. Compressed DDS formats (BC3/DXT5) require specialized tools.

## Building Standalone Executables

### All Platforms

```bash
npm install
pip install -r requirements.txt
```

### Windows

```bash
npm run build:win
```

Output: `dist/IDO Editor Setup.exe` (NSIS installer) and `dist/IDO Editor.exe` (portable)

### macOS

```bash
npm run build:mac
```

Output: `dist/IDO Editor.dmg` and `dist/IDO Editor.app.zip`

### Linux

```bash
npm run build:linux
```

Output: `dist/IDO-Editor.AppImage` and `dist/ido-editor.deb`

Builds are configured with `electron-builder` (app ID: `com.hanazono.ido-editor`). Python scripts are bundled as `extraResources`.

## Project Structure

```
.
+-- main.js                  Electron main process (window, IPC, process spawning)
+-- renderer.js              Electron renderer process (UI logic, tabs, state management)
+-- index.html               Application layout (tabbed interface, console, status bar)
+-- styles.css               Dark theme styling (CSS custom properties, animations)
+-- python-env.js            Python environment detection and script runner
+-- ido_tool.py              Core IDO decompilation/compilation engine (Python)
+-- dds_converter.py         DDS/PNG image converter (Python, Pillow)
+-- package.json             Node.js dependencies and build configuration
+-- requirements.txt         Python dependencies
+-- BUILD.md                 Build instructions (merged into this README)
+-- .gitignore               Ignored file patterns
```

## Application Interface

The GUI features a dark theme (`#0f0f23` background with `#4a90e2` blue accent and `#7b68ee` purple secondary) with:

- **Tabbed navigation**: Switch between Decompile, Compile, and DDS Converter modes
- **File dialog integration**: Native OS file/folder selection via Electron IPC
- **Real-time console**: Color-coded log output with auto-scroll (INFO in blue, SUCCESS in green, WARNING in orange, ERROR in red)
- **Status bar**: Current operation state and details
- **Responsive layout**: Adapts to window resizing (minimum 900x600)

## Console Log Levels

| Level | Color | Purpose |
|-------|-------|---------|
| INFO | Blue | General progress and information |
| SUCCESS | Green | Successful operation completion |
| WARNING | Orange | Non-critical issues |
| ERROR | Red | Critical failures |

## Credits

- **GUI Development**: HanazonoArchive
- **Original IDO Compiler/Decompiler/Converter**: ultimatuuuum (Rust implementation)

## License

ISC
