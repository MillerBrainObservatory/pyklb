cmake_minimum_required(VERSION 3.15...3.27)
project(${SKBUILD_PROJECT_NAME} LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

# Find Python and NumPy
find_package(Python REQUIRED COMPONENTS Interpreter Development.Module NumPy)

# Build KLB library first
# Set option to not install KLB separately
set(CMAKE_PROJECT_NAME ${PROJECT_NAME} CACHE STRING "" FORCE)
add_subdirectory(external/klb EXCLUDE_FROM_ALL)

# Get the KLB include directories
set(KLB_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/external/klb/src)

# Cythonize the pyx file
find_package(Python REQUIRED COMPONENTS Interpreter)
add_custom_command(
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/pyklb.cpp
    COMMAND Python::Interpreter -m cython
        ${CMAKE_CURRENT_SOURCE_DIR}/src/pyklb.pyx
        --cplus
        -3
        -o ${CMAKE_CURRENT_BINARY_DIR}/pyklb.cpp
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/src/pyklb.pyx
    COMMENT "Cythonizing pyklb.pyx"
    VERBATIM
)

# Create the Python extension module
python_add_library(pyklb MODULE
    ${CMAKE_CURRENT_BINARY_DIR}/pyklb.cpp
    WITH_SOABI
)

# Link against KLB and set include directories
target_link_libraries(pyklb PRIVATE klb_static)
target_include_directories(pyklb PRIVATE
    ${KLB_INCLUDE_DIR}
    ${Python_NumPy_INCLUDE_DIRS}
)

# Install the extension module
install(TARGETS pyklb DESTINATION pyklb)

# Create __init__.py
file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/__init__.py "# pyklb - Python wrapper for KLB file format
from .pyklb import *

__version__ = '0.3.0'
__all__ = [
    'readheader',
    'readfull',
    'readroi',
    'readfull_inplace',
    'readroi_inplace',
    'writefull',
]
")

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/__init__.py DESTINATION pyklb)
