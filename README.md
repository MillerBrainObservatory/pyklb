# pyklb

Python wrapper of the [KLB file format](https://bitbucket.org/fernandoamat/keller-lab-block-filetype), a high-performance file format for up to 5-dimensional arrays. For more details, see https://bitbucket.org/fernandoamat/keller-lab-block-filetype

## Installation

### From PyPI (recommended)

```bash
pip install pyklb
```

### From source

```bash
pip install git+https://github.com/bhoeckendorf/pyklb.git
```

## Development

This project has been modernized to use modern Python packaging tools:

- **Build system**: [scikit-build-core](https://scikit-build-core.readthedocs.io/) with CMake
- **Package manager**: [uv](https://docs.astral.sh/uv/) (recommended) or pip
- **Python support**: Python 3.9+ (tested with 3.12.9)

### Requirements

- Python 3.9 or later
- C++ compiler (MSVC on Windows, GCC/Clang on Linux/Mac)
- CMake 3.15 or later (installed automatically by uv/pip)

The KLB C++ library and its dependencies (bzip2, zlib) are built automatically from source.

### Building from source

Using uv (recommended):

```bash
# Install uv if you haven't already
pip install uv

# Clone the repository
git clone https://github.com/bhoeckendorf/pyklb.git
cd pyklb

# Sync dependencies and build
uv sync

# Run tests
uv run pytest
```

Using pip:

```bash
# Clone the repository
git clone https://github.com/bhoeckendorf/pyklb.git
cd pyklb

# Install in development mode
pip install -e .

# Run tests
pytest
```

### Building wheels for distribution

```bash
# Using uv
uv build

# Using build
pip install build
python -m build
```

This creates both a source distribution (.tar.gz) and a wheel (.whl) in the `dist/` directory.
