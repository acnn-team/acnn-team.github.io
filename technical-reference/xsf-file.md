---
title: "SHELX res"
layout: single
classes: wide
lang: en
lang-ref: Internal-utilities
sidebar:
  nav: "docs"
---

This section outlines the format and content of the .xsf files in SHLEX, which serve as the metadata of the dataset.

An example of a .xsf file is shown below

```console
# total energy = -21.42421364 eV

VIRIAL
         76.89614501        -63.24123555         -0.00002632        -63.24123555         72.93618155         -0.00002988         -0.00002632         -0.00002988         50.81682340
CRYSTAL
PRIMVEC
          4.99786000          0.00000000          0.00000000
         -0.15639807          4.99541232          0.00000000
         -2.42073105         -2.49770625          4.56496531
PRIMCOORD
11 1
Sr          0.67926000          4.18551000          0.00000000         -0.64865900          0.62866700         -0.00000800
B           1.28326000          3.60012000          1.60283000         -8.12401900          7.87363300          7.95771100
B          -1.13747000          1.10242000          2.96214000         -8.12400300          7.87365100         -7.95770700
B           2.19007000          2.72126000          1.45153000          2.22159000         -2.15313000          7.43212500
B          -0.23066000          0.22355000          3.11343000          2.22159000         -2.15312100         -7.43212300
B           3.02370000          1.91332000          0.85914000          0.82731300         -0.80183900          4.53257400
B           0.60297000         -0.58439000          3.70583000          0.82733500         -0.80181600         -4.53257200
B           0.61622000          0.85212000          0.00000000         -4.23388800         10.75718300          0.00000700
B           4.00904000          4.35283000          0.00000000        -10.88440200          3.89519100         -0.00000800
B           1.89215000          3.01000000          0.00000000          7.46689400         -7.23677900         -0.00000000
B           3.70091000          1.25698000          0.00000000         18.45024700        -17.88163900          0.00000200

```

The units of length in the file are all in Ångströms (Å).

The units of VIRIAL are in electronvolts (eV).

```console
VIRIAL
    xx  xy  xz  yx  yy  yz  zx  zy  zz           
```

> **Note:** The keyword VIRIAL is not commonly found in standard .xsf files. However, its unit being in electronvolts (eV) is helpful in reflecting the contribution of the PV term to the enthalpy in high pressure systems.