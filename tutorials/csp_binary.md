---
title: "Binary Structures Search with ACNN"
layout: single
classes: wide
lang: en
lang-ref: examples
sidebar:
  nav: "docs"
---

This page offers a binary crystal structure search, complete with the associated scripts, to document the complex process and serve as an example for other similar tasks.

AIRSS
-----

We use [AIRSS](https://www.mtg.msm.cam.ac.uk/Codes/AIRSS) to generate 500,000 random structures.

```console
$ ls

SiSO.cell

$ cat SiSO.cell

#VARVOL=10
#MAXTIME=0.1

#### FORMULA ####
#SPECIES=Si,S,O
#NATOM=1-16
#MINSEP=0.8 Si-Si=1.440 Si-S=1.260 Si-O=1.200 S-S=1.080 S-O=1.020 O-O=0.960

#### SYMMETRY ####
#NFORM=1
#SYMMOPS=1-48
##SYMMNO=2-230
```

`SiSO.cell` is the "seed" input file which describes how the random structures should be built (by `buildcell`). In this case, 1~16 atoms are placed randomly into a unit cell of random shape, and a volume within 5% of 10 Å³ per atom. If a larger number of atoms were requested, then the volume of the cell would be scaled appropriately.

```console
$ airss.pl -build -max 500000 -seed SiSO &

$ ca -s

structure               P/GPa     V/A^3        H/eV  nfu formula      space group      #
  ...
Number of structures   :   5000
Number of compositions :    663
```

It is evident that the extensive compositional space in the structure prediction of this system presents a significant challenge to a comprehensive exploration.


ARES
----

[ARES](https://8.219.233.107) is used as a DFT computational backend to generate datasets for the ACNN model.

```console
$ ls

batch_ares  dyn_ares_in  mkdft  O_ONCV_PBE-1.0.upf  scf.sh  Si_ONCV_PBE-1.0.upf  S_ONCV_PBE-1.0.upf

$ mkdft 0

  ...
```




