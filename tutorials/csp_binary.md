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


> **Note:** Several related software and programs need to be precompiled, see `External` for more details


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
To be precise, we do not restrict the sources of structure generation but only strictly regulate the carrier files of the structures.

Generally, the prepared structures are placed in the `DFT/Base` directory.

```console
$ ls

SrB-2457776-286-1.res
SrB-2457776-286-2.res
SrB-2457776-286-3.res
SrB-2457797-283-1.res
SrB-2457797-283-2.res
SrB-2457797-283-3.res
    ...
    
$ ca -s 

structure               P/GPa     V/A^3        H/eV  nfu formula      space group      #
    ...
Number of structures   :    642
Number of compositions :    158
```

In this case, 1~16 atoms are placed randomly into a unit cell of random shape, and a volume within 5% of 10 Å³ per atom. 
If a larger number of atoms were requested, then the volume of the cell would be scaled appropriately.


Make Seeds
----------
The seed here refers to the known structures in the current search system. Their purpose is limited to constructing phase diagrams, and their information will not be incorporated into any artificial intelligence process.

```console
$ acnn_outcar2seed OUTCAR SrB-end-Sr-1.res      # VASP OUTCAR to seed file
```


Run the Pipeline
----------------

An effective approach is to run the program in the background using either nohup or tmux, depending on your preference.

```console
$ tmux new -s CaH

$ ./auto > log 2>&1 &

$ ctrl+b d

$ tmux attach -t CaH
```

Or

```console
$ nohup ./auto > log 2>&1 &
```





[//]: # (`SiSO.cell` is the "seed" input file which describes how the random structures should be built &#40;by `buildcell`&#41;. In this case, 1~16 atoms are placed randomly into a unit cell of random shape, and a volume within 5% of 10 Å³ per atom. If a larger number of atoms were requested, then the volume of the cell would be scaled appropriately.)






