---
title: "Utilities"
layout: single
classes: wide
lang: en
lang-ref: utilities
sidebar:
  nav: "docs"
---

ACNN provides small command-line utilities for deployment, data conversion, dataset checking, phase-diagram analysis, and structure-format conversion. This page lists the utilities most commonly used in the active-learning structure-search workflow.

## ACNN workflow utilities

### `acnn_deploy`

Deploy a complete active-learning run directory.

```console
$ acnn_deploy -p 40 -s SrB -b Sr-Sr=2.075,Sr-B=1.671,B-B=1.267 -n amd9654 -e vasp
```

Required options:

| Option | Meaning |
| --- | --- |
| `-p, --pressure` | Target pressure in GPa. |
| `-s, --seed` | System tag used in filenames and job names. |
| `-b, --minbonds` | Pairwise minimum bond distances. |
| `-n, --partition` | Slurm partition or node group. |
| `-e, --backend` | First-principles backend: `vasp` or `ares`. |

`acnn_deploy` writes the selected pressure into the generated DFT, phase-diagram, and relaxation scripts.

### `acnn_wait`

Wait for a group of Slurm jobs to finish. In the standard loop it is used after DFT labeling, ACNN training, and ACNN relaxation.

```console
$ acnn_wait FPSrB
$ acnn_wait TRAINSrB
$ acnn_wait RELAXSrB
```

### `acnn_outcar2seed`

Convert a VASP `OUTCAR` into a DFT-relaxed seed structure.

```console
$ acnn_outcar2seed OUTCAR SrB-end-Sr-1.res
```

The output is a [`.res`](/technical-reference/res-file/) file suitable for `SEED/`.

### `acnn_checkdt`

Check the converted ACNN training dataset after DFT-to-XSF conversion.

```console
$ acnn_checkdt 50
```

Run this after `XSF/ry` generates [`.xsf`](/technical-reference/xsf-file/) files for the current iteration.

### `acnn_bsafe`

Check whether a structure satisfies pairwise minimum bond-distance constraints.

```console
$ acnn_bsafe -b "Sr-Sr=2.075,Sr-B=1.671,B-B=1.267" -x 0.6
```

This is useful for filtering unsafe structures before submitting expensive DFT or relaxation jobs.

### `acnn_pdhtml`

Analyze a batch of `.res` structures and generate phase-diagram HTML output.

```console
$ acnn_pdhtml
```

The phase-diagram workflow normally calls this through the generated `PD/mkpd` script.

### `acnn_ckrelax`

Check ACNN relaxation output and report relaxation status. This is typically used inside the relaxation post-processing workflow.

```console
$ acnn_ckrelax -b "Sr-Sr=2.075,Sr-B=1.671,B-B=1.267" -x 0.6 -p 40 -c 192
```

### `acnn_limitjob`

Limit or filter the structures selected from relaxation output before they are returned to the next DFT round.

```console
$ acnn_limitjob RELAXSrB 1000
```

## Structure conversion utilities

### `cabal`

`cabal` is an AIRSS utility included for structure-format conversion. It is commonly used to convert between `.res`, POSCAR, CIF, and other formats.

```console
$ cabal res poscar < input.res > POSCAR
$ cabal poscar res < POSCAR > output.res
```

General usage:

```console
$ cabal in out < seed.in > seed.out
```

### `ca`

`ca` is a wrapper around AIRSS structure-analysis tools. It is useful for inspecting `.res` files, checking composition and symmetry, and preparing phase-diagram inputs.

### `symm`

Find the space group of a structure:

```console
$ symm test
```

For a file named `test.res`, `symm test` reads the structure and reports the detected symmetry.
