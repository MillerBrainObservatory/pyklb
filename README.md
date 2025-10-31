# pyklb

Python wrapper of the [KLB file format](https://bitbucket.org/fernandoamat/keller-lab-block-filetype), a high-performance file format for up to 5-dimensional arrays. For more details, see https://bitbucket.org/fernandoamat/keller-lab-block-filetype

## Installation

```bash
uv pip install pyklb
```

### Requirements

- Python 3.9 or later
- C++ compiler (MSVC on Windows, GCC/Clang on Linux/Mac)
- CMake 3.15 or later (installed automatically by uv/pip)

The KLB C++ library and its dependencies (bzip2, zlib) are built automatically from source.
