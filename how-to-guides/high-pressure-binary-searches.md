---
title: "High-pressure structure searches with ACNN active learning"
layout: single
classes: wide
lang: en
lang-ref: high-pressure-binary-searches
sidebar:
  nav: "docs"
---

This guide describes the current ACNN active-learning workflow for high-pressure crystal structure searches. The example scripts are written for the Sr-B system at 40 GPa, but the same workflow can be adapted to other binary or multi-component systems by changing the system tag, element list, pseudopotentials, bond limits, pressure, and scheduler settings.

The workflow is an iterative loop:

1. Select structures for first-principles labeling.
2. Convert finished DFT calculations into ACNN training data.
3. Build a phase diagram from the labeled structures.
4. Train or restart the ACNN potential.
5. Relax many candidate structures with `acnn_relax`.
6. Select low-energy and unsafe structures for the next DFT round.

The provided run directory is organized as:

```console
$ ls
auto  DFT  PD  POT  RELAX  RSS  SEED  XSF
```

Directory roles:

| Directory | Purpose |
| --- | --- |
| `RSS/` | Generates random structures with AIRSS or CALYPSO. |
| `RSS/Base/` | Pool of initial candidate `.res` structures. |
| `DFT/` | Builds and submits VASP or ARES labeling jobs. |
| `XSF/` | Converts finished DFT output into `.xsf` training data. |
| `PD/` | Builds convex-hull and phase-diagram files. |
| `POT/` | Generates ACNN training input and submits training jobs. |
| `RELAX/` | Relaxes candidate structures with the trained ACNN potential. |
| `SEED/` | Stores known endpoint or reference structures for phase diagrams. |

## Quick start workflow

For a new search, the shortest path is:

1. Deploy a run directory with `acnn_deploy`.
2. Edit the DFT, Slurm, seed, and structure-pool settings.
3. Start the active-learning loop in `tmux`.

### 1. Deploy the template

Use `acnn_deploy` to create the run directory for the target pressure, system, bond limits, Slurm partition, and first-principles backend:

```console
$ acnn_deploy -p 40 -s SrB -b Sr-Sr=2.075,Sr-B=1.671,B-B=1.267 -n amd9654 -e vasp
```

After deployment, the directory should contain:

```console
$ ls
auto  DFT  PD  POT  RELAX  RSS  SEED  XSF
```

The options mean:

| Option | Meaning |
| --- | --- |
| `-p 40` | Search pressure in GPa. |
| `-s SrB` | System tag used in filenames and job names. |
| `-b ...` | Pairwise minimum bond distances used for safety checks. |
| `-n amd9654` | Default Slurm partition or node group. |
| `-e vasp` | First-principles backend. Use `vasp` or `ares`. |

All five options are required, and each option should appear only once. For ARES, use:

```console
$ acnn_deploy -p 40 -s SrB -b Sr-Sr=2.075,Sr-B=1.671,B-B=1.267 -n amd9654 -e ares
```

### 2. Prepare the run directory

Before running `auto`, check these items.

DFT backend:

- Choose the backend at deployment time with `-e vasp` or `-e ares`.
- For VASP, prepare `DFT/POTCAR-*`, edit `DFT/dyn_vasp_in`, and check `DFT/sub-vasp.sh`.
- For ARES, prepare `DFT/*.upf`, edit `DFT/dyn_ares_in`, and check `DFT/sub-ares.sh`.
- Make sure the pressure settings are consistent in `DFT/dyn_vasp_in`, `DFT/dyn_ares_in`, `PD/mkpd`, and `RELAX/dyn_batch_relax_bfgs`.

Slurm and environment:

- Edit partitions, core counts, wall times, and module-loading commands in `DFT/sub-*.sh`, `POT/sub.sh`, `RELAX/dyn_batch_relax_bfgs`, and `RSS/ag`.
- If the system tag changes, also update job names such as `FPSrB`, `TRAINSrB`, `RELAXSrB`, and the matching `acnn_wait` calls in `auto`.

Initial structures:

- Put the initial candidate `.res` files in `RSS/Base/`.
- These structures can come from AIRSS, CALYPSO, previous searches, or any generator that writes valid `.res` files.
- In the first iteration, `DFT/mkdft` randomly selects structures from `RSS/Base/` for labeling.

Seed structures:

- Put known endpoint or reference structures in `SEED/`.
- These are used to build the phase diagram, not automatically injected into training.
- For VASP output, a typical conversion is:

```console
$ acnn_outcar2seed OUTCAR SrB-end-Sr-1.res
```

Iteration sizes:

- Change the number of active-learning iterations in `auto`, for example `seq 0 10`.
- Change the first-round DFT sampling count with `RANDMAX` in `DFT/mkdft`.
- Change the number of structures optimized per iteration with `frame` in `RELAX/dyn_batch_relax_bfgs`.
- Change relaxation grouping with `group`, `warp`, and `job_max` in `RELAX/dyn_batch_relax_bfgs`.
- Change random-structure generation counts in `RSS/ag`, `RSS/dyn_grand`, or `RSS/dyn_gcs`.

### 3. Run with tmux

The ACNN conda package includes `tmux`, so a typical long run can be started as:

```console
$ tmux new -s SrB
$ ./auto > log 2>&1
```

Detach from the session with:

```text
Ctrl-b d
```

Reattach later with:

```console
$ tmux attach -t SrB
```

Monitor the workflow from another terminal:

```console
$ tail -f log
$ squeue -u $USER
```

If you do not want to use `tmux`, `nohup` is also acceptable:

```console
$ nohup ./auto > log 2>&1 &
```

## Detailed workflow reference

The quick start above is enough to launch a prepared run directory. The sections below explain the main scripts, generated files, and system-specific parameters in more detail.

### Global search settings

The top-level `auto` script writes `in.seed` at the start of every iteration:

```bash
IT=0
seed="SrB"
min_bonds="Sr-Sr=2.075,Sr-B=1.671,B-B=1.267"
bonds_scale=".7"
```

`seed` is the system tag used in filenames and some cleanup commands. `min_bonds` defines the element-pair safety distances used during relaxation. `bonds_scale` is automatically adjusted during the loop: if too many structures hit the bond limit, the next iteration uses a slightly smaller scale; otherwise it uses a slightly larger one.

For a different system, update at least:

```bash
seed="AB"
min_bonds="A-A=...,A-B=...,B-B=..."
```

Then make the same element changes in the files listed below.

### Scheduler and environment

The scripts are written for Slurm and use `safe_sbatch`, `acnn_wait`, and `acnn_limitjob`. Check the job names, partition names, core counts, and environment modules in:

```text
RSS/ag
DFT/sub-vasp.sh
DFT/sub-ares.sh
POT/sub.sh
RELAX/dyn_batch_relax_bfgs
```

Typical fields to change:

```bash
#SBATCH --job-name=FPSrB
#SBATCH --partition=amd9654
#SBATCH --ntasks-per-node=48
source /public/env/compiler_intel2025 > /dev/null
```

The default job names are:

| Job name | Step |
| --- | --- |
| `FPSrB` | DFT labeling jobs |
| `TRAINSrB` | ACNN training jobs |
| `RELAXSrB` | ACNN relaxation jobs |
| `STGENSrB` | Random-structure generation jobs |

If you change `seed`, change these names as well, and update the corresponding `acnn_wait` calls in `auto`.

### DFT backend

DFT calculations are prepared by:

```console
$ cd DFT
$ ./mkdft vasp
```

or:

```console
$ cd DFT
$ ./mkdft ares
```

`mkdft` accepts one backend argument:

```text
mkdft [Backend]
supports dft backend: ares / vasp
```

For VASP, edit:

```text
DFT/dyn_vasp_in
DFT/sub-vasp.sh
DFT/POTCAR-Sr
DFT/POTCAR-B
```

The default VASP INCAR generator uses:

```text
ENCUT    = 800
KSPACING = 0.20
PSTRESS  = 400
```

VASP `PSTRESS` is in kB, so `PSTRESS = 400` corresponds to 40 GPa. If you change the search pressure, keep `RELAX/dyn_batch_relax_bfgs`, `PD/mkpd`, `DFT/dyn_vasp_in`, and `DFT/dyn_ares_in` consistent.

For ARES, edit:

```text
DFT/dyn_ares_in
DFT/sub-ares.sh
DFT/*.upf
```

The default ARES input uses:

```text
ecutwfc  = 1200
kspacing = 0.106
pressure = 400
```

## Step 1: Prepare initial structures

The active-learning loop needs an initial pool of candidate structures in:

```text
RSS/Base/
```

The first DFT iteration samples up to 500 structures from this directory:

```bash
RANDMAX=500
res_file=$(find "$BASE" -name "*.res" | shuf | head -n $RANDMAX || true)
```

You can fill `RSS/Base/` using AIRSS, CALYPSO, previous searches, or any other generator that produces valid `.res` files.

The provided AIRSS generators are:

```console
$ cd RSS
$ ./dyn_grand 20000 32
```

and, for composition-directed generation:

```console
$ ./gcom 150 200 > comp.txt
$ ./dyn_gcs comp.txt 32
```

`dyn_grand` creates random structures in many `RAND*` folders. `dyn_gcs` reads compositions from `comp.txt`, writes one AIRSS input per composition, and generates `.res` files. The template currently contains Sr-B-specific values such as:

```text
#SPECIES=Sr,B
#NATOM=1-16
#MINSEP=0.8 Sr-Sr=2.075 Sr-B=1.671 B-B=1.267
```

Change these before using the scripts for another system.

There is also a CALYPSO generator:

```console
$ ./dyn_gcs_calypso comp.txt 32
```

It creates `input.dat` files from the compositions, runs `calypso.x`, then converts `POSCAR_*` files into `.res` files. Check `SYS`, `ATOMIC_VOL`, and `DIST` in `dyn_gcs_calypso` before use.

## Step 2: Add reference seed structures

Known structures should be placed in:

```text
SEED/
```

Seed structures are used for phase-diagram construction. They are not automatically added to the ACNN training set unless they are also selected for DFT labeling.

For VASP results, a typical conversion is:

```console
$ acnn_outcar2seed OUTCAR SrB-end-Sr-1.res
```

The phase-diagram step copies `SEED/*.res` into the current `PD/IT*/` directory before building the hull.

## Step 3: Run one DFT labeling round

From the run root, the automated loop runs:

```console
$ cd DFT
$ ./mkdft vasp > log 2>&1
$ cd ..
$ acnn_wait FPSrB
```

In iteration 0, `mkdft` samples from `RSS/Base/`. In later iterations, it uses structures selected by the previous relaxation/post-processing step:

```text
RELAX/IT*/FPC/SCF/
RELAX/IT*/FPC/OPT/
```

For VASP, `batch_vasp` creates one directory per structure, writes `POSCAR`, `INCAR`, `POTCAR`, copies the Slurm script, then submits the job:

```console
$ echo "y" | ./batch_vasp -p POTCAR-Sr -p POTCAR-B -ts sub.sh -ct scf *.res
```

Supported calculation types are:

```text
scf
cell-relax
```

The standard active-learning loop uses `scf`.

## Step 4: Convert DFT results into ACNN data

After the DFT jobs finish, convert the finished calculations to XSF:

```console
$ cd XSF
$ ./ry > log 2>&1
$ cd ..
```

Internally this calls:

```console
$ vasp2xsf DFT XSF $IT $main_seed
$ acnn_checkdt 50
```

The output is placed in:

```text
XSF/IT0/
XSF/IT1/
...
```

These `.xsf` files are the training data used by `POT/tr`.

## Step 5: Build the phase diagram

Run:

```console
$ cd PD
$ ./mkpd > log 2>&1
$ cd ..
```

`mkpd` collects `SEED/*.res` and the current iteration's `XSF/IT*/` files, converts them to `.res`, adds the pressure-volume term, and builds convex-hull outputs:

```text
PD/IT0/cam
PD/IT0/cam1
PD/IT0/cam2
PD/IT0/*.html
PD/IT0/*.agr
```

The pressure correction uses:

```bash
EVpG=0.0062415091
H = EVpG * PRESS * VOL + energy
```

The default pressure is:

```bash
PRESS=40
```

For binary and ternary systems, `mkpd` can also generate plotting files through `gracebat` and `Rscript` if those commands are available.

## Step 6: Train or restart the ACNN potential

Run:

```console
$ cd POT
$ ./tr
$ cd ..
$ acnn_wait STGENSrB
$ acnn_wait TRAINSrB
```

`POT/tr` creates:

```text
POT/IT0/DT/
POT/IT0/in.acnn
POT/IT0/sub.sh
```

It copies all current `.xsf` data into `DT/`, counts the number of structures, and writes the ACNN input file. Important defaults are:

```text
types       = Sr,B
rcut        = 6.5
acut        = 6.0
nbatch      = 1e5
batchsize   = 4
device      = cpu
float_type  = float64
hasattention = false
```

Restart behavior:

| Iteration | Restart setting |
| --- | --- |
| `IT=0` | Train from scratch. |
| `IT=1` | Restart from `../IT0/model/model-last`. |
| `IT>=2` | Restart from `../IT*/model-restart/model-last`. |

The training job is submitted by `POT/sub.sh`:

```bash
acnn -debug in.acnn
```

For another system, change at least:

```text
types = A,B
```

and check cutoff, basis, loss weights, and device settings.

## Step 7: Relax candidate structures with ACNN

Run:

```console
$ cd RELAX
$ echo "y" | nohup ./dyn_batch_relax_bfgs > log 2>&1
$ cd ..
$ acnn_wait RELAXSrB
```

The relaxation script selects the newest trained model:

```bash
pot=$(ls "$pot_dir"/model*/model-* | sort -V | tail -n 1)
```

It then samples structures from `RSS/Base/`, splits them into groups, and submits multiple Slurm tasks that run:

```console
$ acnn_relax -p 40 -m MODEL -t Sr,B
```

Important defaults:

```text
press   = 40
frame   = 500
group   = 50
warp    = 4
job_max = 4
```

Meaning:

| Parameter | Meaning |
| --- | --- |
| `frame` | Number of candidate structures sampled for relaxation. |
| `group` | Number of structures per group file. |
| `warp` | Number of group relaxations packed into one Slurm job. |
| `job_max` | Maximum number of running relaxation jobs. |

Before submitting, the script prints the dynamic parameters and asks:

```text
CHECK THE PARAMETERS: [y/n]
```

The automated loop answers this with `echo "y"`.

Unsafe trajectories are detected with:

```console
$ acnn_ckrelax -b "$min_bonds" -x $bonds_scale -p "$press" -c $NCORE
```

Structures that violate the safety bond criterion are copied to:

```text
RELAX/IT*/LIM/
```

If more than 100 limit structures appear, the script stops the relaxation batch.

## Step 8: Post-process relaxed structures

Run:

```console
$ cd RELAX
$ ./ppr >> log 2>&1
$ cd ..
```

`ppr` collects all relaxed `*out.res` files into:

```text
RELAX/IT*/RES/
```

Then it runs:

```console
$ cas.py
$ cau.py
$ ca -m -l
```

and selects structures for the next DFT round.

The next-round structures are written into:

```text
RELAX/IT*/FPC/SCF/
RELAX/IT*/FPC/OPT/
```

`FPC/SCF` usually contains low-energy or hull-relevant candidates. `FPC/OPT` contains limit/unsafe structures that should be checked more carefully.

For systems with multiple chemical orders, `ppr` can focus on an n-component hull:

```console
$ ./ppr --focus 2
```

If no focus is provided, it uses all available compositions.

After post-processing, clean relaxation trajectory directories:

```console
$ cd RELAX
$ ./clean_traj >> log 2>&1 &
$ cd ..
```

`clean_traj` removes first-level directories matching the current `seed` inside `RELAX/IT*/`.

## Step 9: Run the full loop

The top-level `auto` script runs the whole loop for iterations 0 through 10:

```bash
for l in $(seq 0 10); do
    ...
done
```

Recommended ways to run it:

```console
$ tmux new -s SrB
$ ./auto > log 2>&1
```

Detach with `Ctrl-b d`, and reattach with:

```console
$ tmux attach -t SrB
```

Or run it with `nohup`:

```console
$ nohup ./auto > log 2>&1 &
```

Watch progress with:

```console
$ tail -f log
$ squeue -u $USER
```

Typical iteration outputs:

```text
DFT/IT0/
XSF/IT0/
PD/IT0/
POT/IT0/
RELAX/IT0/
```

## Adapting the template to another system

The current template is Sr-B-specific. Use this checklist before launching a new search.

| Item | Files |
| --- | --- |
| System tag | `auto`, `RELAX/clean_traj`, job names in submit scripts |
| Element list | `POT/tr`, `RELAX/dyn_batch_relax_bfgs`, `RSS/dyn_grand`, `RSS/dyn_gcs`, `RSS/dyn_gcs_calypso` |
| Bond limits | `auto`, `RSS/dyn_grand`, `RSS/dyn_gcs`, `RSS/dyn_gcs_calypso` |
| Pressure | `PD/mkpd`, `RELAX/dyn_batch_relax_bfgs`, `DFT/dyn_vasp_in`, `DFT/dyn_ares_in` |
| Pseudopotentials | `DFT/POTCAR-*` or `DFT/*.upf` |
| Slurm partition and modules | `DFT/sub-*.sh`, `POT/sub.sh`, `RELAX/dyn_batch_relax_bfgs`, `RSS/ag` |
| Project path | `RELAX/dyn_batch_relax_bfgs` |

The relaxation script currently contains an absolute project path:

```bash
prj="/public/home/lizhao/lijx/run"
```

Change it to the absolute path of your run directory before submitting relaxations. If the path is wrong, `RELAX/dyn_batch_relax_bfgs` will not find `RSS/Base`, `POT/IT*/`, or `XSF/IT*/`.

## Common problems

### `mkdft` finds no structures

Check that the initial pool exists:

```console
$ find RSS/Base -name "*.res" | head
```

For `IT > 0`, check:

```console
$ find RELAX/IT$((IT-1))/FPC -name "*.res"
```

### DFT jobs are not submitted

Check `safe_sbatch`, Slurm partition names, and job limits:

```console
$ which safe_sbatch
$ squeue -u $USER
```

For VASP, make sure the pseudopotential names match the element names in `POSCAR`:

```text
POTCAR-Sr
POTCAR-B
```

### No training data is generated

Check whether DFT calculations finished successfully, then inspect:

```console
$ find XSF/IT0 -name "*.xsf" | wc -l
$ cat XSF/log
```

`POT/tr` copies `XSF/*/*.xsf` into `POT/IT*/DT`, so an empty `XSF/IT*` directory will produce an invalid training round.

### Too many structures hit the bond limit

Inspect:

```console
$ find RELAX/IT*/LIM -name "*limit*.res" | wc -l
```

If this number is large, review `min_bonds` and `bonds_scale` in `in.seed`. The automatic loop adjusts `bonds_scale`, but unreasonable pair distances can still make the relaxation step stop early.

### Phase diagram is empty

Check:

```console
$ cat PD/IT0/cam
$ find SEED -name "*.res"
```

The phase diagram uses both labeled structures and seed structures. Missing endpoints can make the hull hard to interpret.

## Minimal command summary

For an already configured run directory:

```console
$ ./auto > log 2>&1
```

For a manual single iteration:

```console
$ cd DFT && ./mkdft vasp > log 2>&1 && cd ..
$ acnn_wait FPSrB

$ cd XSF && ./ry > log 2>&1 && cd ..

$ cd PD && ./mkpd > log 2>&1 && cd ..

$ cd POT && ./tr && cd ..
$ acnn_wait TRAINSrB

$ cd RELAX && echo "y" | ./dyn_batch_relax_bfgs > log 2>&1 && cd ..
$ acnn_wait RELAXSrB

$ cd RELAX && ./ppr >> log 2>&1 && cd ..
$ cd RELAX && ./clean_traj >> log 2>&1 & cd ..
```
