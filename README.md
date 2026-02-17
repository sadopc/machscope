# MachScope

A native macOS binary analysis tool providing Mach-O parsing, ARM64 disassembly, and process debugging. Built entirely in Swift with zero external dependencies.

[![Swift 6.0](https://img.shields.io/badge/Swift-6.0-orange.svg)](https://swift.org)
[![Platform](https://img.shields.io/badge/platform-macOS%2014%2B-lightgrey.svg)](https://developer.apple.com/macos/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

## Features

| Feature | Description |
|---------|-------------|
| **Mach-O Parsing** | Headers, segments, sections, symbols, dylibs, strings |
| **Code Signatures** | Entitlements, CDHash, signing info, team ID |
| **ARM64 Disassembly** | Full instruction decoder with PAC annotation |
| **Process Debugging** | Attach, breakpoints, memory, registers |
| **Swift Library** | Embed in your own projects |
| **JSON Output** | Script-friendly output format |

## Why MachScope?

- **Pure Swift** — No dependencies, easy to build and embed
- **ARM64 Native** — Built for Apple Silicon, understands PAC instructions
- **All-in-One** — Parse + Disassemble + Debug in one tool
- **Library + CLI** — Use standalone or integrate into your Swift projects
- **Well Tested** — 319+ tests with comprehensive error handling

## Quick Start

```bash
# Build
swift build

# Parse a binary
swift run machscope parse /bin/ls

# Parse a macOS app
swift run machscope parse /Applications/Calculator.app/Contents/MacOS/Calculator

# View entitlements
swift run machscope parse /Applications/Safari.app/Contents/MacOS/Safari --entitlements

# JSON output
swift run machscope parse /bin/ls --json
```

## Installation

### Homebrew

```bash
brew install sadopc/tap/machscope
```

### Build from Source

```bash
git clone https://github.com/sadopc/machscope.git
cd MachScope
swift build -c release
```

### Install Globally (Optional)

```bash
sudo cp .build/release/machscope /usr/local/bin/
```

## Usage

### Parse Command

Analyze Mach-O binary structure:

```bash
# Basic analysis
machscope parse /bin/ls

# Full analysis
machscope parse /bin/ls --all

# Specific sections
machscope parse /path/to/binary --symbols
machscope parse /path/to/binary --dylibs
machscope parse /path/to/binary --strings
machscope parse /path/to/binary --signatures
machscope parse /path/to/binary --entitlements

# JSON output for scripting
machscope parse /bin/ls --json --all > analysis.json
```

### Disassemble Command

Disassemble ARM64 code:

```bash
# List functions
machscope disasm /bin/ls --list-functions

# Disassemble from address
machscope disasm /bin/ls --address 0x100003f40 --length 50

# Show instruction bytes
machscope disasm /bin/ls --show-bytes
```

### Check Permissions

See what features are available:

```bash
machscope check-permissions
```

Output:
```
Feature               Status      Notes
------------------------------------------------------------
Static Analysis       ✓ Ready     No special permissions needed
Disassembly           ✓ Ready     No special permissions needed
Debugger              ✗ Denied    Missing debugger entitlement
```

### Debug Command

Attach to running processes (requires signing):

```bash
# First, sign with debugger entitlement
codesign --force --sign - --entitlements Resources/MachScope.entitlements .build/debug/machscope

# Enable Developer Tools in System Settings > Privacy & Security

# Attach to process
machscope debug <pid>
```

## Use as a Swift Library

Add MachScope to your `Package.swift`:

```swift
dependencies: [
    .package(url: "https://github.com/sadopc/machscope.git", from: "1.0.0")
]
```

Then use in your code:

```swift
import MachOKit
import Disassembler

// Parse a binary
let binary = try MachOBinary(path: "/bin/ls")
print("CPU: \(binary.header.cpuType)")
print("Segments: \(binary.segments.count)")

// Check entitlements
if let signature = try binary.parseCodeSignature(),
   let entitlements = signature.entitlements {
    for key in entitlements.keys {
        print("\(key): \(entitlements[key] ?? "nil")")
    }
}

// Disassemble
let disasm = ARM64Disassembler(binary: binary)
let result = try disasm.disassembleFunction("_main", from: binary)
for instruction in result.instructions {
    print(disasm.format(instruction))
}
```

## Requirements

- macOS 14.0 (Sonoma) or later
- Swift 6.0 or later
- ARM64 (Apple Silicon) — *x86_64 parsing supported, but tool runs on ARM64*

## Documentation

- [Installation Guide](docs/Installation.md)
- [Quick Start](docs/QuickStart.md)
- [Usage Guide](docs/Usage.md)
- [Architecture](docs/Architecture.md)
- [API Reference](docs/API.md)
- [Troubleshooting](docs/Troubleshooting.md)
- [Contributing](docs/Contributing.md)

## Project Structure

```
MachScope/
├── Sources/
│   ├── MachOKit/        # Core Mach-O parsing library
│   ├── Disassembler/    # ARM64 instruction decoder
│   ├── DebuggerCore/    # Process debugging
│   └── MachScope/       # CLI application
├── Tests/               # Test suites (319+ tests)
├── Resources/           # Entitlements for code signing
└── docs/                # Documentation
```

## Who Is This For?

- **iOS/macOS Developers** — Inspect binaries, check entitlements before App Store submission
- **Security Researchers** — Quick binary triage and analysis
- **Students** — Learn Mach-O format with readable Swift code
- **Tool Builders** — Embed MachOKit in your own Swift projects
- **CTF Players** — Fast binary analysis

## Comparison with Other Tools

| Tool | Language | Library? | ARM64 PAC | Debugger |
|------|----------|----------|-----------|----------|
| **MachScope** | Swift | ✅ Yes | ✅ Yes | ✅ Yes |
| otool | C | ❌ No | ❌ No | ❌ No |
| objdump | C | ❌ No | ❌ No | ❌ No |
| jtool2 | C | ❌ No | ✅ Yes | ❌ No |
| Hopper | — | ❌ No | ✅ Yes | ❌ No |

MachScope's main advantage: **Swift-native library** you can embed in your own tools.

## License

MIT License — See [LICENSE](LICENSE) for details.

## Contributing

Contributions welcome! Please read [Contributing Guide](docs/Contributing.md) first.

```bash
# Run tests before submitting
swift test

# Format code
xcrun swift-format -i -r Sources/ Tests/
```

## Acknowledgments

- Apple's Mach-O documentation
- ARM Architecture Reference Manual
- The Swift community

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=sadopc/machscope&type=date&legend=top-left)](https://www.star-history.com/#sadopc/machscope&type=date&legend=top-left)

---

**Built with ❤️ in Swift**
