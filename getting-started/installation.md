---
title: "Installation"
layout: single
classes: wide
lang: en
lang-ref: installation
sidebar:
  nav: "docs"
---

# ACNN Conda Installation Guide

This page describes how to install ACNN with Conda. The Conda package installs the main ACNN executables, ACNN-Relax, workflow utilities, and the runtime libraries required by most ACNN tasks.

After installation, the environment includes

- **ACNN** — Neural-network interatomic potential
- **ACNN-Relax** — Structure relaxation engine
- **Workflow utilities** — Database and workflow management scripts
- **Runtime dependencies** — LibTorch, OpenBLAS, and other required libraries

---

## Conda Installation

### System Requirements

- Linux (x86_64)
- glibc >= 2.17
- Conda or Miniforge
- Internet access to download packages from **conda-forge** (required only during installation)

---

### Installation

#### 1. Extract the released Conda channel

```bash
tar xzf acnn-conda-channel.tar.gz
```

This creates the local Conda channel

```text
conda-bld/
├── channeldata.json
├── index.html
├── linux-64/
└── noarch/
```

---

#### 2. Create a new environment

```bash
conda create -n acnn \
    -c file://$PWD/conda-bld \
    -c conda-forge \
    acnn-suite
```

---

#### 3. Activate the environment

```bash
conda activate acnn
```

---

### Verify Installation

Verify that the major executables are available.

```bash
which acnn
which acnn_relax

which safe_sbatch
which acnn_deploy
which acnn_builddb
which rres
```

Verify that shared libraries are correctly linked.

```bash
ldd $(which acnn) | grep "not found"

ldd $(which acnn_relax) | grep "not found"
```

Both commands should produce **no output**.

---

### Installed Components

#### ACNN

```text
acnn
acnn_relax
```

---

#### Workflow Utilities

```text
safe_sbatch
rres
acnn_builddb
acnn_deploy
...
```

---

### Runtime Dependencies

The following runtime libraries are installed automatically by Conda during installation.

- LibTorch (CPU)
- OpenBLAS

No manual installation is required.

---

### Updating

If a newer Conda channel is provided, update the installed packages with

```bash
conda update \
    -c file://$PWD/conda-bld \
    -c conda-forge \
    acnn-suite
```

---

### Removing

Remove the Conda environment

```bash
conda env remove -n acnn
```

---

### Troubleshooting

If installation or execution fails, please include the outputs of the following commands when reporting an issue.

```bash
conda info

conda list

which acnn
which acnn_relax

ldd $(which acnn)
ldd $(which acnn_relax)
```

---

## Source Installation

ACNN has been successfully tested on the x86_64 Linux platform.

### Dependencies

- LibTorch (Pre-cxx11 ABI for broader platform support)
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
You may consider adding the directory of the executable programs to your `$PATH`. 

```console
export PATH=$(pwd)/build:$PATH
```

To confirm that the installation was successful, execute the following command:

```console
$ acnn
```

The output menu will provide instructions on how to use ACNN for simulations. Additionally, try creating an input control file.
> **Note:** It is strongly recommended to use `g++` version 9 or higher to build ACNN and utilities. Other compiler families (such as `ifort`) are not supported.

**License.** Please refer to the licenses of the individual software components included in this toolkit.
