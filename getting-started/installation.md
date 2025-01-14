---
title: "Installation"
layout: single
classes: wide
lang: en
lang-ref: installation
sidebar:
  nav: "docs"
---

ACNN has been successfully tested on the x86_64 Linux platform.

### Dependencies
- Libtorch (Pre-cxx11 ABI for broader platform support)
- OpenBLAS (Accelerated molecular dynamics simulations for CPU) [make USE_OPENMP=1]
- CUDA (Optional: for GPU accelerating. Consider adding cudnn)

### Installation from Source
Extract `torchdemo-x.x.x.tgz` and then navigate into the `torchdemo-x.x.x/` directory:

```console
$ tar -xvf torchdemo-v3.tar.gz

x torchdemo-v3/
x torchdemo-v3/CMakeLists.txt
x torchdemo-v3/README.md
x torchdemo-v3/al/
x torchdemo-v3/al/al-py/
...
x torchdemo-v3/src/utilities.h
x torchdemo-v3/src/utilsnnp.cpp
x torchdemo-v3/src/utilsnnp.h

$ cd torchdemo
```

Modify `prefix.cmake` for specifying the path of the dependency package.
```cmake
# user modify
set(CUDA_TOOLKIT_ROOT_DIR /path/to/cuda)        # optional
set(CMAKE_CXX_COMPILER /path/to/c++compiler)    # required
set(Torch_DIR /path/to/libtorch)                # required
set(OpenBLAS_DIR /path/to/openblas)             # required
set(Python_EXECUTABLE /python/to/python3)       # optional  for python interface
set(pybind11_DIR /path/to/pybind11)             # optional  for python interface(see Python Interface)
```

Execute the following compound command to perform a default installation:

```console
$ cmake -B build
$ cmake --build build --target acnn
```

The executables will be located in `torchdemo/build`. 
You may want to add this directory to your `$PATH`. 

```console
export PATH=$(pwd)/build:$PATH
export PATH=$(pwd)/interface/airss:$PATH
```

To confirm that the installation was successful, execute the following command:

```console
$ acnn
```

The output menu will tell you how to use acnn for simulations. Also attempt to create an input control file.

> **Note:** It is strongly recommended that `g++` version 9 and above are used to build the AIRSS
utilities. Other compiler families (such as `ifort`) are not supported.




Troubleshooting
---------------
