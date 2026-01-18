# Nuwa Build Action

A GitHub Action to automatically build Python wheels for **[Nuwa Build](https://github.com/martineastwood/nuwa-build)** projects using `cibuildwheel`.

This action handles the complexity of installing the Nim compiler across different platforms (Linux Docker containers, macOS, and Windows) and orchestrating the wheel build process.

## Features

- **Zero Configuration**: Automatically sets up Nim on the runner or inside build containers.
- **Multi-Platform Support**: Builds wheels for Linux (`manylinux`), macOS (`x86_64` & `arm64`), and Windows.
- **Cibuildwheel Integration**: leverages the industry-standard `cibuildwheel` for reliable Python wheel generation.
- **Customizable**: Supports specific Nim versions and standard `cibuildwheel` environment variables.

## Usage

Add this action to your workflow file (e.g., `.github/workflows/publish.yml`).

### Basic Example

```yaml
name: Build Wheels

on: [push, pull_request]

jobs:
  build_wheels:
    name: Build wheels on ${{ matrix.os }}
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]

    steps:
      - uses: actions/checkout@v4

      - name: Build wheels
        uses: martineastwood/nuwa-build-action@v1
        with:
          nim-version: "2.0.0"

      - uses: actions/upload-artifact@v4
        with:
          name: cibw-wheels-${{ matrix.os }}-${{ strategy.job-index }}
          path: ./wheelhouse/*.whl
```

### Advanced Configuration

You can control which Python versions to build or architectures to target using standard `cibuildwheel` environment variables.

```yaml
steps:
  - name: Build wheels
    uses: martineastwood/nuwa-build-action@v1
    with:
      nim-version: "2.0.0"
      cibw-version: "2.16.5"
    env:
      # Build Python 3.10 through 3.13
      CIBW_BUILD: "cp310-* cp311-* cp312-* cp313-*"
      # Skip PyPy and musllinux
      CIBW_SKIP: "pp* *-musllinux_*"
      # Build universal wheels for macOS
      CIBW_ARCHS_MACOS: "x86_64 arm64"
```

## Inputs

| Input          | Description                                 | Default  |
| -------------- | ------------------------------------------- | -------- |
| `nim-version`  | The version of the Nim compiler to install. | `2.0.0`  |
| `cibw-version` | The version of `cibuildwheel` to use.       | `2.22.0` |

## How It Works

This action performs platform-specific setup to ensure the Nim compiler is available during the build process:

### 🐧 Linux

On Linux, wheels are built inside Docker containers (usually `manylinux`) to ensure compatibility.

- The action injects a script into `CIBW_BEFORE_ALL_LINUX`.
- This script installs `gcc` and `curl`, downloads the official Nim binaries, and adds them to the PATH inside the Docker container before the build starts.

### 🍎 macOS

- Installs Nim using the official `choosenim` installer.
- Adds Nim to the system PATH.

### 🪟 Windows

- Installs Nim using `chocolatey`.
- Adds Nim to the system PATH.

## Frequently Asked Questions

### Can I run custom commands before the build?

Yes, but be careful on Linux. On Linux, this action sets the `CIBW_BEFORE_ALL_LINUX` environment variable to install Nim. If you overwrite this variable in your workflow configuration, the Nim installation will be skipped, and the build will likely fail.

On macOS and Windows, you can run standard `run` steps before this action to set up other dependencies.

### Where are the wheels stored?

By default, `cibuildwheel` outputs built wheels to the `./wheelhouse/` directory. You should use `actions/upload-artifact` to save them, as shown in the usage example.

## License

MIT
