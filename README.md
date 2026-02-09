# 🤖 Setup Godot

[![GitHub release](https://img.shields.io/github/v/release/niranjanshk27/setup-godot?style=flat-square)](https://github.com/niranjanshk27/setup-godot/releases)
[![GitHub marketplace](https://img.shields.io/badge/marketplace-setup--godot-blue?style=flat-square&logo=github)](https://github.com/marketplace/actions/setup-godot-ci-cd)

A Github Action to setup Godot Engine for CI/CD workflows on Linux and macOS runners. Supports standard .NET (C#) versions with flexible caching strategies optiomized for both GitHub-hosted and self-hosted runners.

Features
- Multi-platform support.
- Support for Godot 4.x version
- Headless mode.
- .NET/Mono support.
- Export templages installation optional.
- Flexible caching (Gihub Action cache (default), local filesystem cache or no cache
- Self-hosted friendly.

---

## Quick start

For Github-hosted runners

```yaml
name: CI
on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Godot
        uses: niranjanshk27/setup-godot@v1
        with:
          version: "4.2.1"
      
      - name: Build project
        run: godot --headless --export-release "Linux/X11" ./build/game.x86_64
```

Self-hosted Runners (with local cache)

```yaml
name: CI
on: [push, pull_request]

jobs:
  build:
    runs-on: self-hosted
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Godot
        uses: niranjanshk27/setup-godot@v1
        with:
          version: "4.2.1"
          cache-strategy: "local"
          local-cache-dir: "/var/cache/godot"
      
      - name: Build project
        run: godot --headless --export-release "Linux/X11" ./build/game.x86_64
```
---

## Usage Examples

### C# / .NET Project

```yaml
- name: Setup Godot with .NET
  uses: niranjanshk27/setup-godot@v1
  with:
    version: "4.2.1"
    dotnet: true
    install-export-templates: true

- name: Build C# project
  run: |
    godot --headless --build-solutions
    godot --headless --export-release "Linux/X11" ./build/game.x86_64
```

### macOS Build

```yaml
jobs:
  build-macos:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Godot
        uses: niranjanshk27/setup-godot@v1
        with:
          version: "4.2.1"
          install-export-templates: true
      
      - name: Build for macOS
        run: godot --headless --export-release "macOS" ./build/game.app
```

### Matrix Testing

Test your project against multiple Godot versions:

```yaml
jobs:
  test:
    strategy:
      matrix:
        godot-version: ["4.2.0", "4.2.1", "4.3.0"]
        os: [ubuntu-latest, macos-latest]
    runs-on: ${{ matrix.os }}
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Godot
        uses: niranjanshk27/setup-godot@v1
        with:
          version: ${{ matrix.godot-version }}
      
      - name: Run tests
        run: godot --headless --script res://tests/run_tests.gd
```

### Export Multiple Platforms

```yaml
jobs:
  export:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Godot
        uses: niranjanshk27/setup-godot@v1
        with:
          version: "4.2.1"
          install-export-templates: true
      
      - name: Export Linux
        run: godot --headless --export-release "Linux/X11" ./builds/linux/game.x86_64
      
      - name: Export Windows
        run: godot --headless --export-release "Windows Desktop" ./builds/windows/game.exe
      
      - name: Export Web
        run: godot --headless --export-release "Web" ./builds/web/index.html
      
      - name: Upload artifacts
        uses: actions/upload-artifact@v4
        with:
          name: game-builds
          path: builds/
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `version` | Godot version to install (e.g., `4.2.1`, `4.3.0`). Must be a stable release. | **Yes** | - |
| `headless` | Install headless version (Linux only, ignored on macOS) | No | `true` |
| `dotnet` | Install .NET/Mono version for C# support | No | `false` |
| `install-export-templates` | Download and install export templates | No | `false` |
| `cache-strategy` | Caching strategy: `github`, `local`, or `none` | No | `github` |
| `local-cache-dir` | Local cache directory (used when `cache-strategy` is `local`) | No | `/tmp/godot-cache` |

---

## Cache Strategy

Choose how download files are cahced:
- `github`: Uses GitHub Actions cache (default, best for GitHub-hosted runners)
- `local`: Uses local filesystem cache (best for self-hosted runners)
- `none`: No caching, download every time

## Outputs

| Output | Description |
|---|---|
| `godot-path` | Absolute path to the installed `godot` binary (e.g. `/usr/local/bin/godot`). Useful when downstream steps need to reference the binary explicitly. |
| `templates-path` | Absolute path to the export-templates directory. Empty string when `install-export-templates` is `false`. |

---

## License

MIT
