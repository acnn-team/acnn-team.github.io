---
title: "SHELX res"
layout: single
classes: wide
lang: en
lang-ref: Internal-utilities
sidebar:
  nav: "docs"
---

This section outlines the format and content of the .res files in SHLEX, which serve as the foundation for most of the file analysis utilities in ACNN.

An example of a .res file is shown below

```console
TITL SrB-2286444-4824-10 0 115.382 0 0 0  12 (C2) n - 1
REM
REM in /public/home/projects/RSS/Binary
REM
REM
REM
REM
REM s
REM
REM
REM ./SrB.cell (4b4e2465a186bc3266ba68dc2f899007)
REM ## AIRSS Version 0.9.3 July 2022 commit b0220bf5cbf550a3b7
REM compiler GCC version 14.2.0
REM options -cpp -mtune=generic -march=x86-64 -g -Og -fpre-include=/usr/include/finclude/math-vector-fortran.h
REM cmdline: airss.pl -build -max 1000 -seed SrB 
REM seed 894416080 -1559583065 -1876802120 -1273630309 502417760 -2029562590 -623420922 -901089259
REM
CELL 1.54180    4.95734    4.95734    5.07324   84.16089   84.16089   68.83962
LATT -1
SFAC Sr B  
Sr     1  0.5654957000000  0.5112210000000  0.7154375000000 1.0
Sr     1  0.4887790000000  0.4345043000000  0.2845625000000 1.0
Sr     1  0.2504996000000  0.7495004000000  0.0000000000000 1.0
B      2  0.5408928000000  0.8437647000000  0.1371216000000 1.0
B      2  0.1562353000000  0.4591072000000  0.8628784000000 1.0
B      2  0.3631216000000  0.7739965000000  0.4235327000000 1.0
B      2  0.2260035000000  0.6368784000000  0.5764673000000 1.0
B      2  0.8537888000000  0.2910247000000  0.2785960000000 1.0
B      2  0.7089753000000  0.1462112000000  0.7214040000000 1.0
B      2  0.6900928000000  0.7915148000000  0.4502138000000 1.0
B      2  0.2084852000000  0.3099072000000  0.5497862000000 1.0
B      2  0.9508182000000  0.0491818000000  0.0000000000000 1.0
END
```

In the first line (TITL), each item represents specific information:

```console
TITL SrB-2286444-4824-10   0     115.382       0       0        0      12      (C2)       n - 1
[K]       [labal]       [press] [volume]  [enthalpy] [spin] [modspin] [na]   [pearson]   [n frame]
```

Lines beginning with `RES` typically contain comments or annotations related to specific data points or results within the file. These lines often serve to provide additional information or clarification about the structure, calculations, or parameters in the .res file.

For CELL

```console
CELL 1.54180    4.95734    4.95734    5.07324   84.16089   84.16089   68.83962
                  [a]        [b]        [c]      [alpha]    [beta]     [gamma]
```
