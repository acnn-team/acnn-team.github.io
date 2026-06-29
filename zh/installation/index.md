---
title: "安装方法"
layout: single
classes: wide
lang: zh
lang-ref: installation
sidebar:
  nav: "docs"
---

# ACNN-CSP Installation Guide

ACNN-CSP 是通过 Conda 分发的晶体结构预测和基于神经网络势的结构优化工具包。

工具包包含：

- **ACNN** — 神经网络原子间势
- **ACNN-Relax** — 结构优化引擎
- **Workflow utilities** — 数据库和工作流管理脚本
- **Runtime dependencies** — LibTorch、OpenBLAS 以及其他必需运行库

---

## Conda 安装

### 系统要求

- Linux (x86_64)
- glibc >= 2.17
- Conda 或 Miniforge
- 可以从 **conda-forge** 下载包的网络环境（仅安装时需要）

---

### 安装

#### 1. 解压发布的 Conda channel

```bash
tar xzf acnn-conda-channel.tar.gz
```

该命令会生成本地 Conda channel：

```text
conda-bld/
├── channeldata.json
├── index.html
├── linux-64/
└── noarch/
```

---

#### 2. 创建新环境

```bash
conda create -n acnn \
    -c file://$PWD/conda-bld \
    -c conda-forge \
    acnn-suite
```

---

#### 3. 激活环境

```bash
conda activate acnn
```

---

### 验证安装

确认主要可执行程序可以被找到：

```bash
which acnn
which acnn_relax

which safe_sbatch
which acnn_deploy
which acnn_builddb
which rres
```

确认共享库链接正常：

```bash
ldd $(which acnn) | grep "not found"

ldd $(which acnn_relax) | grep "not found"
```

这两个命令都应该**没有任何输出**。

---

### 已安装组件

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

### 运行时依赖

以下运行库会在安装过程中由 Conda 自动安装：

- LibTorch (CPU)
- OpenBLAS

不需要手动安装这些依赖。

---

### 更新

如果提供了更新的 Conda channel，可以使用：

```bash
conda update \
    -c file://$PWD/conda-bld \
    -c conda-forge \
    acnn-suite
```

---

### 删除

删除 Conda 环境：

```bash
conda env remove -n acnn
```

---

### 故障排查

如果安装或运行失败，反馈问题时请附上以下命令的输出：

```bash
conda info

conda list

which acnn
which acnn_relax

ldd $(which acnn)
ldd $(which acnn_relax)
```

---

## 源码安装

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
```

运行以下命令确认 ACNN 是否可以正常启动：

```console
$ acnn
```

输出菜单会给出 ACNN 模拟相关的使用提示，也可以进一步尝试创建输入控制文件。

> **注意：** 推荐使用 `g++` 9 或更高版本构建 ACNN 及相关工具。不支持 `ifort` 等其它编译器家族。

**许可证。** 请参考该工具包中各个软件组件各自的许可证。
