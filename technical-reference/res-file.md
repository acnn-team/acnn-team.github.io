---
title: "SHELX .res format"
layout: single
classes: wide
lang: en
lang-ref: res-file
sidebar:
  nav: "docs"
---

ACNN uses SHELX-style `.res` files as a compact structure format. In a high-pressure active-learning run, `.res` files are used for initial candidate structures, DFT-relaxed seed structures, phase-diagram structures, and relaxed structures selected for the next iteration.

Use `.res` when you need to store or exchange a crystal structure. Use [`.xsf`]({{ '/technical-reference/xsf-file/' | relative_url }}) when you need ACNN training data with energy, virial, and forces.

## Where `.res` files appear

| Location | Meaning |
| --- | --- |
| `RSS/Base/` | Initial candidate structures sampled in the first DFT iteration. These structures can be unrelaxed. |
| `SEED/` | DFT-relaxed endpoint or reference structures with computed energies. These are used for phase-diagram construction. |
| `PD/IT*/` | Structures collected for the convex hull at each iteration. |
| `RELAX/IT*/FPC/` | Structures selected after ACNN relaxation for the next DFT labeling round. |

## Example `.res` file

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

A typical `.res` file contains:

| Line | Purpose |
| --- | --- |
| `TITL` | Structure name and metadata used by ACNN/AIRSS-style tools. |
| `REM` | Comments. ACNN ignores these for most structure parsing. |
| `CELL` | Unit-cell parameters. |
| `LATT` | Lattice-centering and inversion flag in SHELX notation. |
| `SFAC` | Element list. Atom lines refer to this element order by index. |
| Atom lines | Element label, species index, fractional coordinates, and occupancy. |
| `END` | End of structure. |

## `TITL` metadata

ACNN tools follow the common AIRSS/SHELX-style `TITL` convention:

```console
TITL SrB-2286444-4824-10   0     115.382       0       0        0      12      (C2)       n - 1
TITL <name> <pressure> <volume> <enthalpy> <spin> <modspin> <#ions> <(symmetry)> n - <#copies>
```

Important fields:

| Field | Meaning |
| --- | --- |
| `<name>` | Structure identifier. This is usually also used as the filename stem. |
| `<pressure>` | Pressure in GPa when available. Generated candidates may contain `0` before DFT labeling. |
| `<volume>` | Cell volume in Å^3. |
| `<enthalpy>` | Enthalpy or energy-derived value used by analysis tools. Generated candidates may contain `0`. |
| `<#ions>` | Number of atoms in the structure. |
| `<(symmetry)>` | Space-group or symmetry label when known. |
| `<#copies>` | Number of equivalent or duplicate copies reported by analysis tools. |

For unrelaxed random structures in `RSS/Base/`, energy-like fields can be placeholders. For seed structures in `SEED/`, the energy or enthalpy fields should come from DFT-relaxed calculations so the phase diagram is meaningful.

## `CELL`

```console
CELL 1.54180    4.95734    4.95734    5.07324   84.16089   84.16089   68.83962
```

The first value is the SHELX wavelength field and is normally left as `1.54180`. The remaining six values are the cell parameters:

```text
CELL <wavelength> <a> <b> <c> <alpha> <beta> <gamma>
```

Lengths are in Å, and angles are in degrees.

## `SFAC` and atom lines

```console
SFAC Sr B
Sr     1  0.5654957000000  0.5112210000000  0.7154375000000 1.0
B      2  0.5408928000000  0.8437647000000  0.1371216000000 1.0
```

`SFAC` defines the element order. In the atom lines, the integer after the atom label is the index into `SFAC`:

| Value | Meaning |
| --- | --- |
| Atom label | Usually the chemical symbol, such as `Sr` or `B`. |
| Species index | `1` means the first `SFAC` element, `2` means the second, and so on. |
| `x y z` | Fractional coordinates. |
| Occupancy | Usually `1.0` for ACNN structure-search workflows. |

## Common conversions and checks

Use `cabal` to convert between `.res` and other structure formats when needed:

```console
$ cabal res poscar < input.res > POSCAR
$ cabal poscar res < POSCAR > output.res
```

Use `ca` or related analysis utilities to inspect structure identity, composition, volume, symmetry, and enthalpy metadata. When a `.res` file is used as a seed structure, always confirm that it represents a DFT-relaxed structure with the correct composition and pressure.
