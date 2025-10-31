# Building pyklb Wheels for PyPI

This document describes how to build wheels for pyklb for multiple Python versions and platforms.

## Prerequisites

- Python 3.9 or later
- C++ compiler (MSVC on Windows, GCC/Clang on Linux/Mac)
- [uv](https://docs.astral.sh/uv/) package manager (recommended)
- [cibuildwheel](https://cibuildwheel.readthedocs.io/) for multi-version builds (optional)

## Quick Build (Current Python Version)

To build a wheel for your current Python version:

```bash
uv build
```

This creates:
- `dist/pyklb-0.3.0.tar.gz` - Source distribution
- `dist/pyklb-0.3.0-cpXXX-cpXXX-win_amd64.whl` - Wheel for current Python version

## Building for Multiple Python Versions

### Option 1: Using cibuildwheel (Recommended)

cibuildwheel automates building wheels for multiple Python versions:

```bash
pip install cibuildwheel
cibuildwheel --platform windows  # or linux, macos
```

Configuration is in `pyproject.toml` under `[tool.cibuildwheel]`.

Current configuration builds for:
- Python 3.9, 3.10, 3.11, 3.12, 3.13
- Windows x64, Linux x86_64, macOS x86_64/arm64

### Option 2: Using uv with multiple Python versions

Install multiple Python versions and build separately:

```bash
# Install Python versions
uv python install 3.9 3.10 3.11 3.12 3.13

# Build for each version
for version in 3.9 3.10 3.11 3.12 3.13; do
    uv python install $version
    uv build --python $version
done
```

### Option 3: Using Docker for Linux wheels

For manylinux wheels (Linux compatibility):

```bash
docker run -it --rm \
    -v $(pwd):/project \
    quay.io/pypa/manylinux2014_x86_64 \
    bash -c "cd /project && /opt/python/cp312-cp312/bin/pip install cibuildwheel && /opt/python/cp312-cp312/bin/cibuildwheel --platform linux"
```

## Testing Wheels

Before uploading to PyPI, test the wheels:

```bash
# Create a fresh virtual environment
uv venv test-env
source test-env/bin/activate  # or test-env\Scripts\activate on Windows

# Install the wheel
pip install dist/pyklb-0.3.0-cp312-cp312-win_amd64.whl

# Run tests
python -m pytest test/

# Test import
python -c "import pyklb; print(pyklb.__version__)"
```

## Uploading to PyPI

### Test PyPI (recommended for first upload)

```bash
pip install twine
twine upload --repository testpypi dist/*
```

### Production PyPI

```bash
twine upload dist/*
```

## Platform-Specific Notes

### Windows

- Requires MSVC (Visual Studio Build Tools)
- Builds 64-bit wheels only (32-bit is deprecated)
- Uses `delvewheel` to bundle required DLLs

### Linux

- Uses manylinux2014 for broad compatibility
- Requires Docker for building on non-Linux systems
- Static linking of libstdc++ for portability

### macOS

- Builds universal2 wheels (x86_64 + arm64)
- Minimum macOS version: 10.14 (Mojave)

## Continuous Integration

For automated builds on GitHub Actions, create `.github/workflows/build-wheels.yml`:

```yaml
name: Build wheels

on:
  push:
    branches: [main, master]
  release:
    types: [published]

jobs:
  build_wheels:
    name: Build wheels on ${{ matrix.os }}
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]

    steps:
      - uses: actions/checkout@v4
        with:
          submodules: recursive

      - name: Build wheels
        uses: pypa/cibuildwheel@v2.16
        env:
          CIBW_BUILD: cp39-* cp310-* cp311-* cp312-* cp313-*

      - uses: actions/upload-artifact@v4
        with:
          name: wheels-${{ matrix.os }}
          path: ./wheelhouse/*.whl
```

## Troubleshooting

### Build fails with "Cannot find CMake"

Install CMake:
```bash
pip install cmake
```

### Build fails with "C++ compiler not found"

- **Windows**: Install Visual Studio Build Tools
- **Linux**: Install `build-essential` or `gcc`
- **macOS**: Install Xcode Command Line Tools: `xcode-select --install`

### Tests fail after installing wheel

Make sure you're testing in a clean environment without the editable install:
```bash
pip uninstall pyklb
pip install dist/pyklb-*.whl
```

### Wheel size is too large

The source distribution includes the full KLB git repository. To reduce size:
- The wheel only includes compiled binaries (much smaller)
- Source distributions on PyPI will be larger but that's normal for C++ extensions

## Versioning

Version is defined in `pyproject.toml`:
```toml
[project]
version = "0.3.0"
```

Remember to:
1. Update version in `pyproject.toml`
2. Update version in `CMakeLists.txt` `__init__.py` generation
3. Create a git tag: `git tag v0.3.0`
4. Push tag: `git push origin v0.3.0`
