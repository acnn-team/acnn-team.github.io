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


> Several related software and programs need to be precompiled, see `External` for more details


Deploy the Pipeline
-------------------

We deploy a pipeline for concurrent learning on the binary Sr-B system at 40 GPa. Please refer to [here](https://arxiv.org/abs/2310.13945) for detailed information about the system.

```console
$ acnn_deploy -p 40 -s SrB -b Sr-Sr=2.075,Sr-B=1.671,B-B=1.267 -n amd9654

$ ls
auto  DFT  PD  POT  RELAX  RSS  SEED  XSF
```

Modify the Personalized File
----------------------------
Some files, such as pseudopotentials and input files for DFT calculations, need to be adjusted by the user according to meet their specific computational requirements.

For users hooked on VASP, you need to check the following files.

INCAR
```console
$ vi DFT/dyn_vasp_in

    ...
 46 SEGMENT_IN+="""
 47 PREC         =   Normal
 48 ISTART       =   0
 49 ISPIN        =   1
 50 LREAL        =   False
 51 ENCUT        =   650
 52 KSPACING     =   0.25
 53 LWAVE        =   False
 54 LCHARG       =   False
 55 ADDGRID      =   False
 56 ISYM         =   0
 57 KPAR         =   4
 58 NCORE        =   4
 59 ISMEAR       =   0
 60 SIGMA        =   0.05
 61 NELM         =   200
 62 NELMIN       =   6
 63 EDIFF        =   1e-07
 64 SYMPREC      =   1e-05
 65 PSTRESS      =   2000
 66 """
    ...
```

Slurm submits task script
```console
$ vi DFT/sub.sh

    ...
  9 source /public/env/compiler_oneapi2024 > /dev/null
 10 
    ...
 20 
 21 cmd="mpirun -np $total_cores vasp_std"
    ...
```

And POTCARS
```console
$ ls DFT/POTCAR*

DFT/POTCAR-Sr  DFT/POTCAR-B
```

Generate Random Structures
--------------------------
To be precise, we do not restrict the sources of structure generation but strictly regulate the carrier files of the structures.







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


```console
$ ls

batch_ares  dyn_ares_in  mkdft  O_ONCV_PBE-1.0.upf  scf.sh  Si_ONCV_PBE-1.0.upf  S_ONCV_PBE-1.0.upf

$ mkdft 0

  ...
```




