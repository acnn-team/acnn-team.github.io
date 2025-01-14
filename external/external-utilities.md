---
title: "External Utilities"
layout: single
classes: wide
lang: en
lang-ref: external-utilities
sidebar:
  nav: "docs"
---

ACNN calls a number of external packages to perform specific tasks related to calculation and analysis. This page provides an overview of these packages.

Dependence Graph
--------
```console
Project
├── acnn
│   ├── OpenBLAS
│   │   ├── Version: >= 0.3.0
│   │   └── Used for: Accelerates matrix operations
│   ├── LibTorch
│   │   ├── Version: >= 1.13.0
│   │   ├── Prebuilt binaries
│   │   │   ├── CPU version
│   │   │   └── GPU version (optional)
│   │   └── Used for: Machine learning computations
│   └── Functionality: Neural network predictions
│
├── acnn_relax
│   ├── OpenBLAS
│   │   ├── Version: >= 0.3.0
│   │   └── Used for: Accelerates matrix operations
│   ├── LibTorch
│   │   ├── Version: >= 1.13.0
│   │   ├── Prebuilt binaries
│   │   │   ├── CPU version
│   │   │   └── GPU version (optional)
│   │   └── Used for: Machine learning computations
│   └── Functionality: Structure relaxation with neural networks
│
└── lmp_mpi
    ├── OpenBLAS
    │   ├── Version: >= 0.3.0
    │   └── Used for: Accelerates matrix operations
    ├── LibTorch
    │   ├── Version: >= 1.13.0
    │   ├── Prebuilt binaries
    │   │   ├── CPU version
    │   │   └── GPU version (optional)
    │   └── Used for: Machine learning computations
    ├── LAMMPS
    │   ├── Version: lammps-2Aug2023
    │   └── Used for: Molecular dynamics simulations
    └── Functionality: Coupling molecular dynamics with machine learning
```

ACNN_RELAX (Recommend)
----------------------

This program is designed for large-scale batch structural optimization, employing the BFGS algorithm implemented in ARES.
```console
$ cd torchdemo/interface/bfgs
$ cmake -B build
$ cmake --build build --target acnn_relax
```


LAMMPS
------------------

LAMMPS stands for Large-scale Atomic/Molecular Massively Parallel Simulator. LAMMPS is a classical molecular dynamics simulation code focusing on materials modeling.
Please refer [here](https://www.lammps.org) for more information.

[//]: # (As a computational backend, ACNN can provide energy, atomic force, and cell pressure calculations for LAMMPS during atomic simulations. This capability enables material property simulations such as molecular dynamics learning and structural relaxation.)

An automic script `build_lammps_interface.sh` is designed for download lammps and build.

```console
$ cd torchdemo/interface/lammps
$ sh build_lammps_interface.sh build 8
```

qhull
-----

High-performance phase diagram solver.

http://www.qhull.org/

CALYPSO
-------

CALYPSO is a simple and powerful method for structure prediction.

http://calypso.cn

AIRSS
-----

AIRSS is a simple and powerful method for structure prediction.

https://www.mtg.msm.cam.ac.uk/Codes/AIRSS

ARES
----

[ARES](https://8.219.233.107) is used as a DFT computational backend to generate datasets for the ACNN model.


xmgrace
-------

http://plasma-gate.weizmann.ac.il/Grace/

Xmgrace/Grace is useful for visualising results. `.agr` scripts are generated to facilitate this.


R/Rscript (optional)
---------

The statistical package R is used to visualise ternary convex hulls. The `ternary.r` scripts can be executed using `Rscript`. The `ggtern` packaged is required.

https://cran.r-project.org/
http://www.ggtern.com/




