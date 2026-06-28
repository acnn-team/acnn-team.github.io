---
title: "安装方法"
layout: single
classes: wide
lang: zh
lang-ref: installation
sidebar:
  nav: "docs"
---

ACNN 已在 x86_64 Linux 平台上完成测试。

### 依赖项

- LibTorch（建议使用 pre-cxx11 ABI 版本以获得更好的平台兼容性）
- OpenBLAS（用于 CPU 上的加速分子动力学模拟，可考虑 `make USE_OPENMP=1`）
- CUDA（可选，用于 GPU 加速；可根据需要加入 cuDNN）

### 从源码安装

解压 `torchdemo-x.x.x.tgz`，然后进入源码目录：

```console
$ tar -xvf torchdemo-v3.tar.gz

torchdemo-v3/
torchdemo-v3/CMakeLists.txt
torchdemo-v3/README.md
torchdemo-v3/al/
torchdemo-v3/al/al-py/
...
torchdemo-v3/src/utilities.h
torchdemo-v3/src/utilsnnp.cpp
torchdemo-v3/src/utilsnnp.h

$ cd torchdemo
```

修改 `prefix.cmake`，指定依赖包路径：

```cmake
# user modify
set(CUDA_TOOLKIT_ROOT_DIR /path/to/cuda)        # optional
set(CMAKE_CXX_COMPILER /path/to/c++compiler)    # required
set(Torch_DIR /path/to/libtorch)                # required
set(OpenBLAS_DIR /path/to/openblas)             # required
set(Python_EXECUTABLE /python/to/python3)       # optional  for python interface
set(pybind11_DIR /path/to/pybind11)             # optional  for python interface
```

执行以下命令进行默认构建：

```console
$ cmake -B build
$ cmake --build build --target acnn
```

可执行文件会生成在 `torchdemo/build` 中。可以考虑将该目录加入 `$PATH`：

```console
export PATH=$(pwd)/build:$PATH
export PATH=$(pwd)/interface/airss:$PATH
```

运行以下命令确认 ACNN 是否可以正常启动：

```console
$ acnn
```

输出菜单会给出 ACNN 模拟相关的使用提示，也可以进一步尝试创建输入控制文件。

> **注意：** 推荐使用 `g++` 9 或更高版本构建 ACNN 及相关工具。不支持 `ifort` 等其它编译器家族。

故障排查
---------------
