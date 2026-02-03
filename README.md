# 🤖 Setup Godot

A GitHub Action that downloads and installs [Godot Engine](https://godotengine.org) directly from [GitHub Releases](https://github.com/godotengine/godot/releases) on **Linux** and **macOS** CI runners — no package managers involved.

Supports headless builds, .NET (mono / C#) variants, export templates, and caches all downloads so that repeated runs skip the network entirely.

---

## Quick start

```yaml
- uses: niranjanshk27/setup-godot@v1
  with:
    version: "4.2.1"
```

That single step downloads the headless Linux build, installs it to `/usr/local/bin/godot`, and verifies it. Everything else is optional.

---

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `version` | No | `4.2.1` | Godot version to install. Must match a tag on the [Godot releases page](https://github.com/godotengine/godot/releases) (the `-stable` suffix is added automatically). |
| `headless` | No | `true` | Install the headless build. **Linux only** — macOS has no separate headless binary; pass `--headless` at runtime instead. Ignored on macOS. |
| `dotnet` | No | `false` | Install the `.NET` (mono) variant, which adds full C# support. When combined with `install-export-templates`, the matching mono template archive is downloaded as well. |
| `install-export-templates` | No | `false` | Download and unpack export templates into the directory Godot reads by default, so that `godot --export` works without extra configuration. |

---

## Outputs

| Output | Description |
|---|---|
| `godot-path` | Absolute path to the installed `godot` binary (e.g. `/usr/local/bin/godot`). Useful when downstream steps need to reference the binary explicitly. |
| `templates-path` | Absolute path to the export-templates directory. Empty string when `install-export-templates` is `false`. |

---

## Usage examples

### Headless Linux build (default)

The simplest CI setup — runs tests or exports without a display server.

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: niranjanshk27/setup-godot@v1
        with:
          version: "4.2.1"
          headless: "true"

      - run: godot --headless --quit
```

### macOS build

On macOS the action installs the universal binary (Intel + Apple Silicon). The `headless` input is ignored; pass `--headless` to the binary at runtime when you need it.

```yaml
jobs:
  build-mac:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4

      - uses: niranjanshk27/setup-godot@v1
        with:
          version: "4.2.1"

      - run: godot --headless --quit
```

### .NET / C# project

Enables the mono variant of both the editor and the export templates in one go.

```yaml
jobs:
  build-dotnet:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: niranjanshk27/setup-godot@v1
        with:
          version: "4.2.1"
          dotnet: "true"
          install-export-templates: "true"

      - run: godot --headless --quit
```

### Export a game

Downloads the editor and the export templates, then runs a headless export.

```yaml
jobs:
  export:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: niranjanshk27/setup-godot@v1
        with:
          version: "4.2.1"
          install-export-templates: "true"

      - run: |
          godot --headless --export-release "Linux64" build/my-game.x86_64
```

### Referencing outputs in later steps

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - id: godot
        uses: niranjanshk27/setup-godot@v1
        with:
          version: "4.2.1"
          install-export-templates: "true"

      - run: |
          echo "Binary  : ${{ steps.godot.outputs.godot-path }}"
          echo "Templates: ${{ steps.godot.outputs.templates-path }}"
          ${{ steps.godot.outputs.godot-path }} --headless --quit
```

---

## How caching works

Downloads are cached with [actions/cache@v4](https://github.com/actions/cache) using two independent keys so that changing one dimension never invalidates the other:

| Cache | Key pattern | What is stored |
|---|---|---|
| Editor | `godot-editor-<OS>-<version>-dotnet-<bool>-headless-<bool>` | The raw `.zip` downloaded from GitHub |
| Templates | `godot-templates-<OS>-<version>-dotnet-<bool>` | The raw `.tpz` downloaded from GitHub |

On a cache hit both the download and the install steps are skipped entirely, bringing the total action time down to a few seconds.

---

## Platform notes

| Topic | Linux | macOS |
|---|---|---|
| Headless binary | Dedicated `.headless` asset | Not available; use `--headless` flag at runtime |
| Architecture | `x86_64` | Universal (Intel + Apple Silicon) |
| Binary location after install | `/usr/local/bin/godot` | `/usr/local/bin/godot` |
| Quarantine handling | — | `com.apple.quarantine` extended attribute is stripped automatically |
| Templates directory | `~/.local/share/godot/templates/<ver>.stable` | `~/Library/Application Support/godot/templates/<ver>.stable` |

---

## Requirements

* **Linux runners** — `curl` and `unzip` (pre-installed on all standard GitHub-hosted `ubuntu-*` images).
* **macOS runners** — `curl` and `unzip` (pre-installed on all standard GitHub-hosted `macos-*` images).
* A valid Godot version string that corresponds to an existing [GitHub Release](https://github.com/godotengine/godot/releases). The action appends `-stable` to whatever version you provide.

---

## License

MIT
