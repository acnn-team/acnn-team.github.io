---
title: "Utilities"
layout: single
classes: wide
lang: en
lang-ref: utilities
sidebar:
  nav: "docs"
---

ACNN provides command-line utilities for deployment, DFT-output conversion, dataset checking, relaxation post-processing, phase-diagram analysis, structure databases, and scheduler control. This page summarizes the `acnn*` utilities shipped in `interface/airss`.

## Deployment and scheduler helpers

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

All five options are required and each option must appear only once. `acnn_deploy` writes the selected pressure, backend, partition, and bond limits into the generated DFT, phase-diagram, training, and relaxation scripts.

### `acnn_wait`

Wait until all Slurm jobs with a given job name have disappeared from the current user's queue.

```console
$ acnn_wait FPSrB
$ acnn_wait TRAINSrB
$ acnn_wait RELAXSrB
```

Internally it polls:

```console
$ squeue -u $(whoami) -n JOB_NAME
```

and sleeps for 20 seconds between checks.

### `acnn_limitjob`

Throttle Slurm submission by waiting until the number of jobs with a given name is below a limit.

```console
$ acnn_limitjob RELAXSrB 1000
```

Usage:

```console
$ acnn_limitjob <job_name> <max_count>
```

This is normally called inside generated batch scripts before submitting more jobs.

## DFT output to seed structures

### `acnn_outcar2seed`

Convert a VASP `OUTCAR` into a DFT-relaxed seed [`.res`]({{ '/technical-reference/res-file/' | relative_url }}) structure.

```console
$ acnn_outcar2seed OUTCAR SrB-end-Sr-1.res
```

Usage:

```console
$ acnn_outcar2seed <OUTCAR> <SEED_NAME>
```

The script reads the final VASP structure with ASE, converts it through `cabal poscar res`, extracts enthalpy or final `TOTEN`, converts pressure from kB to GPa, and rewrites the `.res` metadata with `rres`.

### `acnn_ares2seed`

Convert an ARES output file into a DFT-relaxed seed `.res` structure.

```console
$ acnn_ares2seed ares_output.dat SrB-end-Sr-1.res
```

Usage:

```console
$ acnn_ares2seed <ares_output.dat> <SEED_NAME>
```

The script extracts the final lattice, direct coordinates, element counts, enthalpy, and pressure from the ARES output, writes a temporary POSCAR-style structure, converts it with `cabal`, and updates `.res` metadata with `rres`.

## DFT output to ACNN training data

### `acnn_ares2xsf`

Convert an ARES SCF output file into ACNN [`.xsf`]({{ '/technical-reference/xsf-file/' | relative_url }}) training data.

```console
$ acnn_ares2xsf ares_output.dat > sample.xsf
```

Usage:

```console
$ acnn_ares2xsf <ARES output>
```

The output is written to standard output. The script extracts total energy, lattice vectors, Cartesian coordinates, forces, volume, and stress, then converts stress to the `VIRIAL` block used by ACNN.

### `acnn_recycling`

Convert one or more first-principles output files into an `.xsf` training set.

```console
$ acnn_recycling */OUTCAR
$ acnn_recycling --calc qe --outdir qe_xsf */scf.out
$ acnn_recycling --calc ares --outdir ares_xsf run*/scf_output
```

Usage:

```console
$ acnn_recycling [--calc vasp|qe|ares] [--outdir xsf_output] FILE [FILE ...]
```

Supported calculators:

| Calculator | Input |
| --- | --- |
| `vasp` | `OUTCAR`, `XDATCAR`, `goodStructures_POSCARS`, or `goodStructs_POSCAR` readable by ASE. |
| `qe` | Quantum ESPRESSO `espresso-out` files. |
| `ares` | ARES SCF output in the script's supported format. |

Output files are written to `--outdir` and named from the source path plus a zero-padded step number, for example `runs_Al_fcc_OUTCAR_step000003.xsf`.

### `acnn_checkdt`

Check an `.xsf` dataset for unreasonable force values.

```console
$ acnn_checkdt 50 XSF/IT0
```

Usage:

```console
$ acnn_checkdt [force_threshold] [xsf_directory]
```

Defaults are a force threshold of `100 eV/A` and the current directory. Any `.xsf` file containing an absolute force component above the threshold is renamed by removing the `.xsf` suffix, so it will no longer be picked up as training data.

## Relaxation post-processing and safety checks

### `acnn_bsafe`

Check all `.res` structures in the current directory against pairwise minimum bond distances.

```console
$ acnn_bsafe -b "Sr-Sr=2.075,Sr-B=1.671,B-B=1.267" -x 0.6
```

Options:

| Option | Meaning |
| --- | --- |
| `-b, --bond` | Comma-separated minimum bond distances. |
| `-x` | Scale factor applied to all minimum distances. Default: `1.0`. |

The script prints one row per `.res` file with the structure name, pressure, and `True` or `False`. For unsafe structures, it also reports the offending atom pair, indices, actual distance, and threshold.

### `acnn_ckrelax`

Check ACNN relaxation trajectories, convert safe relaxed structures to `.res`, and keep the last safe structure for unsafe trajectories.

```console
$ export opt_dir="SrB-1 SrB-2 SrB-3"
$ acnn_ckrelax -b "Sr-Sr=2.075,Sr-B=1.671,B-B=1.267" -x 0.6 -p 40 -c 192
```

Options:

| Option | Meaning |
| --- | --- |
| `-b, --bond` | Comma-separated minimum bond distances. |
| `-x` | Scale factor applied to all minimum distances. |
| `-p, --press_in_gpa` | Pressure in GPa written into generated `.res` metadata. |
| `-c, --core` | Number of parallel worker processes. |

`acnn_ckrelax` reads the list of relaxation directories from the environment variable `opt_dir`. Each directory is expected to contain `relax.vasp` and `relax.conv`.

Outputs:

| Case | Output |
| --- | --- |
| Safe final structure | `RELAX_DIR/RELAX_DIR-out.res` |
| Unsafe final structure with a safe earlier step | `RELAX_DIR/RELAX_DIR-limitbonds.res` and a copy in `LIM/` |

The script uses `cabal` when available to annotate symmetry in the generated `.res` file.

## Phase-diagram utilities

### `acnn_pdhtml`

Build a phase diagram from all `.res` files in the current directory and write an interactive HTML plot.

```console
$ acnn_pdhtml
```

Output:

```text
pd.html
```

It uses ASE to read `.res` files and `pymatgen.analysis.phase_diagram` to construct the phase diagram.

### `acnn_pdb`

Print phase-diagram stability information for all `.res` files in the current directory.

```console
$ acnn_pdb
```

The output lists structures sorted by energy above hull. Stable entries are marked with `+`; unstable entries are marked with `-`.

### `acnn_shull`

Print hull information for all `.res` files in the current directory.

```console
$ acnn_shull
```

For each structure, the output includes filename, reduced composition, number of element types, energy above hull, and formation energy per atom.

## Structure database utilities

### `acnn_builddb`

Import `.res` files from a directory tree into a SQLite database.

```console
$ acnn_builddb structures.db RSS/Base
```

Usage:

```console
$ acnn_builddb <DB> <SRC>
```

The database table is `structures(id, filename, content, used)`. Duplicate filenames are ignored, and the command can append to an existing database.

### `acnn_mergedb`

Merge one structure database into another.

```console
$ acnn_mergedb all.db new_batch.db
```

Usage:

```console
$ acnn_mergedb <DST> <SRC>
```

The source database is opened read-only. Duplicate filenames in the destination are ignored.

### `acnn_statdb`

Print summary statistics for a structure database.

```console
$ acnn_statdb structures.db
```

The output reports total, used, and unused structure counts.

## Model and simulation analysis

### `acnn_emodel`

Evaluate an ACNN model on a dataset using the local `in.acnn` template.

```console
$ acnn_emodel model-001 DT
```

Usage:

```console
$ acnn_emodel <MODEL_PATH> <DT_PATH>
```

The script edits `evalmodelpath` and `evaldatapath` in `in.acnn`, runs:

```console
$ acnn -eval in.acnn.tmp
```

then analyzes the output with `evalan.py`. The raw evaluation output is written to `eval-<model_basename>-<dataset_basename>`.

### `acnn_estvol`

Estimate per-element atomic volumes from `.res` structures using a least-squares fit.

```console
$ cat *.res | acnn_estvol
```

The command reads `.res` structures from standard input, calls `cryan -r -l`, extracts formula and volume information, and solves a linear least-squares problem for element volumes.

### `acnn_lmp_thermodata`

Parse LAMMPS thermo blocks and plot each block to PDF.

```console
$ acnn_lmp_thermodata log.lammps
```

Output files are named:

```text
block-0.pdf
block-1.pdf
...
```

Each plot shows thermo columns versus `Step`, with sampled mean and standard-deviation markers.

## Structure manipulation

### `acnn_supercell`

Build a supercell from a structure file and write POSCAR text to standard output.

```console
$ acnn_supercell POSCAR 2,2,1 > POSCAR.super
```

Usage:

```console
$ acnn_supercell <file> <nx,ny,nz>
```

The script uses pymatgen to read the input structure, multiply it by the requested supercell, and print VASP POSCAR format.

### `acnn_symop`

Print symmetry operations for a space group.

```console
$ acnn_symop 166
$ acnn_symop R-3m
```

Usage:

```console
$ acnn_symop <space_group_symbol_or_number>
```

The output includes the space-group symbol, international number, number of symmetry operations, xyz notation, rotation matrices, and translation vectors.

## AIRSS conversion helpers

### `cabal`

`cabal` is an AIRSS utility included for structure-format conversion. It is commonly used to convert between `.res`, POSCAR, CIF, and other formats.

```console
$ cabal res poscar < input.res > POSCAR
$ cabal poscar res < POSCAR > output.res
```

### `ca`

`ca` is a wrapper around AIRSS structure-analysis tools. It is useful for inspecting `.res` files, checking composition and symmetry, and preparing phase-diagram inputs.

### `symm`

Find the space group of a structure:

```console
$ symm test
```

For a file named `test.res`, `symm test` reads the structure and reports the detected symmetry.
