---
title: "Build Graph"
layout: single
classes: wide
lang: en
lang-ref: installation
sidebar:
  nav: "docs"
---

ACNN can be interfaced with a number of external packages and software to perform specific tasks related to simulation and analysis. This page provides an overview of these interfaces.

OVERVIEW
--------
```console
acnn
├── OpenBLAS
│   ├── Version: >= 0.3.0
│   └── Used for: Accelerates matrix operations
└── LibTorch
    ├── Version: >= 1.13.0
    ├── Prebuilt binaries
    │   ├── CPU version
    │   └── GPU version (optional)
    └── Machine learning computations
```







LAMMPS
------

LAMMPS stands for Large-scale Atomic/Molecular Massively Parallel Simulator. LAMMPS is a classical molecular dynamics simulation code focusing on materials modeling.
Please refer [here](https://www.lammps.org) for more information.

[//]: # (As a computational backend, ACNN can provide energy, atomic force, and cell pressure calculations for LAMMPS during atomic simulations. This capability enables material property simulations such as molecular dynamics learning and structural relaxation.)

An automic script `build_lammps_interface.sh` is designed for download lammps and build.

```console
cd torchdemo/interface/lammps && sh build_lammps_interface.sh build 8
```