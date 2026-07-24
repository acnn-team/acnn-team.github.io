---
title: "Crystal structure prediction with ACNN active learning"
layout: single
classes: wide
lang: en
lang-ref: crystal-structure-prediction
sidebar:
  nav: "docs"
---

This guide describes the current ACNN active-learning workflow for crystal
structure prediction. The examples use the Sr-B system at 40 GPa, but the same
workflow can be adapted to binary and multi-component systems.

The workflow supports two structure-generation modes:

1. **Online mode (recommended):** provide a composition list in
   `RSS/comp.txt`. CALYPSO generates structures while ACNN relaxations are
   distributed through Slurm.
2. **Offline mode:** generate structures in advance and place the `.res` files
   in `RSS/Base`.

Both modes use the same DFT, training, phase-diagram, and active-learning
stages. Online mode requires no user-prepared structures; it starts directly
from `RSS/comp.txt`.

> **Note:** ACNN-CSP, CALYPSO, a supported DFT backend, and the required
> scheduler commands must be available before starting. See the installation
> documentation for details.

Deploy the Pipeline
-------------------

The following example deploys a concurrent-learning workflow for the binary
Sr-B system at 40 GPa, using VASP for first-principles calculations and the
`amd9654` Slurm partition.

```console
$ mkdir SrB-40GPa
$ cd SrB-40GPa

$ acnn_deploy \
    -p 40 \
    -s SrB \
    -b Sr-Sr=2.075,Sr-B=1.671,B-B=1.267 \
    -n amd9654 \
    -e vasp
```

All five options are required:

- `-p` specifies the target pressure in GPa.
- `-s` specifies the chemical system. Element symbols must use their normal
  capitalization.
- `-b` specifies the minimum pair distances in angstrom.
- `-n` specifies the Slurm partition.
- `-e` selects the first-principles backend: `vasp` or `ares`.

After deployment, the working directory contains:

```console
$ ls

DFT  PD  POT  RELAX  RSS  SEED  XSF  auto  auto-online
```

The generated `auto-online` and `auto` scripts correspond to the online and
offline modes, respectively.

Directory roles:

| Directory | Purpose |
| --- | --- |
| `RSS/` | Stores composition input and structure-generation tools. |
| `RSS/Base/` | Online mode creates its small iteration-0 bootstrap set here automatically. In offline mode, this is the large user-prepared candidate-structure pool. |
| `DFT/` | Builds and submits VASP or ARES labeling jobs. |
| `XSF/` | Converts finished DFT output into [`.xsf`]({{ '/technical-reference/xsf-file/' | relative_url }}) training data. |
| `PD/` | Builds convex-hull and phase-diagram files. |
| `POT/` | Generates ACNN training input and submits training jobs. |
| `RELAX/` | Optimizes candidate structures in both modes: batch optimization in offline mode and CALYPSO + ACNN optimization in online mode. |
| `SEED/` | Stores known endpoint and reference structures for phase diagrams. |

The iterative loop is:

1. Prepare or generate structures for first-principles labeling.
2. Convert finished DFT calculations into ACNN training data.
3. Build a phase diagram from the labeled structures.
4. Train or restart the ACNN potential.
5. Generate and relax candidates with the selected online or offline mode.
6. Select low-energy and unsafe structures for the next DFT round.


Modify the Personalized Files
-----------------------------

The deployed scripts provide a working template, but cluster paths, DFT input
parameters, pseudopotentials, and Slurm resources must be checked before the
workflow is submitted.

For VASP, inspect the generated input template:

```console
$ vi DFT/dyn_vasp_in
```

Typical settings that require attention include `ENCUT`, `KSPACING`, `ISMEAR`,
`SIGMA`, `EDIFF`, and `PSTRESS`.

Inspect the VASP Slurm script and adapt the environment setup and executable
path to the local cluster:

```console
$ vi DFT/sub-vasp.sh
```

The generated DFT jobs are single-node MPI jobs. Keep the MPI transport
configured for shared memory within one node.

Prepare one VASP pseudopotential file for each element:

```console
$ ls DFT/POTCAR-*

DFT/POTCAR-B  DFT/POTCAR-Sr
```

If ARES is selected instead, inspect `DFT/dyn_ares_in` and
`DFT/sub-ares.sh`, and place the required `.upf` files in `DFT`.

The training and relaxation scripts also contain cluster-specific Slurm
settings:

```console
$ vi POT/sub.sh
$ vi RELAX/dyn_batch_relax_bfgs
```

> **Note:** Review the environment initialization in every generated Slurm
> script. Paths such as compiler setup scripts and DFT executables are
> installation-specific.


Make Seeds
----------

Seeds are known structures used to construct phase diagrams. They are not
added to the neural-network training set merely because they are present in
`SEED`.

Store seed structures as [`.res`
files]({{ '/technical-reference/res-file/' | relative_url }}):

```console
$ acnn_outcar2seed OUTCAR SrB-end-Sr-1.res
$ mv SrB-end-Sr-1.res SEED/
```

For a binary search, prepare suitable endpoint structures for both elements
and any other trusted reference phases needed for the phase diagram.


Choose a Structure-Generation Mode
----------------------------------

The two modes differ in how candidate structures enter the workflow.

| Mode | User-provided structure input | Run script |
| --- | --- | --- |
| Online (recommended) | `RSS/comp.txt` only | `auto-online` |
| Offline | `.res` files in `RSS/Base` | `auto` |

Do not run `auto` and `auto-online` simultaneously in the same deployed
directory.


Online Mode (Recommended)
-------------------------

Online mode generates CALYPSO structures and relaxes them with the current
ACNN model without first building a large static structure collection.
Compositions are dispatched through a shared Slurm queue, allowing several
worker jobs and several single-core ACNN relaxations per worker.

> **Requirement:** Online mode requires `calypso.x` to be available in
> `PATH`. Check it before starting:
>
> ```console
> $ command -v calypso.x
> ```


### Prepare `RSS/comp.txt`

Create a two-column composition file:

```console
$ vi RSS/comp.txt
```

For example:

```text
# formula  structures
SrB        10
SrB2       10
Sr2B       10
SrB3       10
Sr3B       10
```

The first column is a chemical formula. The second column is the total number
of structures requested for that formula during each online search stage.
Duplicate formulas are merged by summing their requested structure counts.

Start conservatively with about 10 structures per composition. As the
active-learning iterations proceed, increase this number gradually rather
than requesting thousands of structures at the beginning.

A practical indicator is the number of files in the current iteration's
`RELAX/IT*/LIM` directory:

```console
$ find RELAX/IT0/LIM -type f -name '*limit*.res' | wc -l
```

Replace `IT0` with the iteration being reviewed. If only a small number of
structures reach `LIM`, the current ACNN model is handling the sampled
configuration space reasonably well and the next iteration can use a larger
number per composition. If many structures enter `LIM`, keep the population
small until additional DFT data improve the model.

During the iteration-0 bootstrap only, `DFT/init_online` ignores the second
column. It distributes its own `total_structures` setting across the unique
formulas in `comp.txt` to create the initial DFT data set.


### Initial Online Structure Generation

No initial structures need to be prepared for online mode.
`DFT/init_online` automatically generates the small iteration-0 bootstrap set
from the formulas in `RSS/comp.txt`. The pressure-dependent crystal volumes
and pair-distance settings have already been written by `acnn_deploy` and
normally do not need to be edited.


### Configure Online Slurm Workers

Review the runtime settings near the top of
`RELAX/dyn_online_slurm_submit`:

```console
$ vi RELAX/dyn_online_slurm_submit

slots_per_task=48
max_tasks=4
keep_n=50
batch_popsize=50
partition="amd9654"
```

- `slots_per_task` is the number of concurrent single-core ACNN relaxation
  slots inside one Slurm worker.
- `max_tasks` is the maximum number of simultaneously running Slurm workers.
- `keep_n` is the number of low-energy structures retained from each
  composition.
- `batch_popsize` splits a large requested population into queue batches.
- `partition` is inherited from `acnn_deploy` and can still be edited.

The maximum number of concurrent ACNN relaxations is approximately:

```text
slots_per_task × max_tasks
```

For the settings above, at most 192 relaxation slots can run simultaneously.
Choose values that match the node size, memory limit, and user job limits of
the cluster.

The default worker pool contains `max_tasks` workers. A worker that exits
abnormally is requeued while unassigned batches remain.


### Configure Online Selection

Review the selection settings near the top of `RELAX/ppr_online`:

```console
$ vi RELAX/ppr_online

ncomp=120
nstruct=5
```

`ncomp` controls how many compositions are selected, and `nstruct` controls
how many structures are retained for each selected composition.


### Run the Online Pipeline

Using `tmux` is recommended because the top-level workflow runs through
multiple Slurm submissions and active-learning iterations.

```console
$ tmux new -s SrB

$ ./auto-online > log 2>&1
```

Detach from the session with `Ctrl-b d` and return later with:

```console
$ tmux attach -t SrB
```

At iteration 0, `auto-online` performs the following sequence:

1. Generate the bootstrap structures from `RSS/comp.txt`.
2. Submit first-principles calculations.
3. Build the phase diagram and training data.
4. Train the ACNN model.
5. Submit online CALYPSO + ACNN relaxation workers.
6. Select structures for the next DFT iteration.

Later iterations reuse the structures selected by the preceding online stage.


Offline Mode
------------

Offline mode is useful when structures are generated separately, collected
from several programs, or reviewed before entering the ACNN workflow.


### Prepare Random Structures

ACNN does not restrict how structures are generated, but each structure must
be stored in the [`.res`
format]({{ '/technical-reference/res-file/' | relative_url }}).

Place the prepared files in `RSS/Base`:

```console
$ find RSS/Base -maxdepth 1 -name '*.res' | head

RSS/Base/SrB-2457776-286-1.res
RSS/Base/SrB-2457776-286-2.res
RSS/Base/SrB2-2457797-283-1.res
```

Review the collection before starting:

```console
$ cd RSS/Base
$ ca -s
```

For iteration 0, the generated DFT workflow randomly selects up to 500
structures from `RSS/Base`. If a different initial DFT set size is required,
edit `RANDMAX` in `DFT/mkdft`.


### Run the Offline Pipeline

Return to the deployed project root and start `auto`:

```console
$ cd ../..
$ tmux new -s SrB

$ ./auto > log 2>&1
```

Alternatively:

```console
$ nohup ./auto > log 2>&1 &
```

The offline workflow performs DFT calculations, data conversion, phase
diagram construction, ACNN training, batch relaxation of the prepared
structures, and post-processing for the next iteration.


Check the Workflow
------------------

The top-level log records the current active-learning stage:

```console
$ tail -f log
```

Slurm jobs can be inspected with:

```console
$ squeue -u "$USER"
```

Each iteration produces matching `ITN` directories under `DFT`, `XSF`, `PD`,
`POT`, and `RELAX`. Check the corresponding local logs when a stage stops or
a Slurm job fails.

Before launching a long production search, run a small composition set and
confirm that:

- DFT jobs start correctly and produce valid output.
- DFT results are converted into training structures.
- ACNN training completes.
- ACNN relaxation can load the generated model.
- Phase-diagram and selection outputs are produced.


Detailed Workflow Reference
---------------------------

The preceding sections describe the shortest route to a running search. This
section explains the generated scripts and each active-learning stage in more
detail.


### Global Search Settings

At the start of every iteration, `auto-online` or `auto` writes `in.seed` in
the project root:

```bash
IT=0
seed="SrB"
min_bonds="Sr-Sr=2.075,Sr-B=1.671,B-B=1.267"
bonds_scale=".7"
```

`IT` is the active-learning iteration number. `seed` is the system tag used
in filenames and Slurm job names. `min_bonds` defines the element-pair safety
distances used during relaxation. `bonds_scale` scales those distances for
the active-learning safety check.

Online mode calls `RELAX/update_bonds_scale` before submitting its structure
queue. The script examines limit structures from the preceding iteration and
adjusts the scale within configured lower and upper bounds. The chosen value
is preserved when the next `in.seed` is written.

The generated top-level scripts run iterations 0 through 10:

```bash
for l in $(seq 0 10); do
    ...
done
```

Edit this range before starting if a different number of iterations is
required.


### Scheduler and Environment

The workflow is written for Slurm and uses the helper commands
`safe_sbatch`, `acnn_wait`, and `acnn_limitjob`. Confirm that they are
available:

```console
$ command -v safe_sbatch
$ command -v acnn_wait
$ command -v acnn_limitjob
```

Review job names, partitions, core counts, wall times, environment modules,
and executable paths in:

```text
DFT/sub-vasp.sh
DFT/sub-ares.sh
POT/sub.sh
RELAX/dyn_batch_relax_bfgs
RELAX/dyn_online_slurm_worker
```

Typical generated fields are:

```bash
#SBATCH --job-name=FPSrB
#SBATCH --partition=amd9654
#SBATCH --ntasks-per-node=48
source /public/env/compiler_intel2025 > /dev/null
```

For the Sr-B example, the job names are:

| Job name | Stage |
| --- | --- |
| `FPSrB` | First-principles labeling |
| `TRAINSrB` | ACNN training |
| `RELAXSrB` | Offline relaxation or online worker array |


### First-Principles Backend

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

`auto-online` and `auto` use the backend selected with `acnn_deploy -e`.

For VASP, review:

```text
DFT/dyn_vasp_in
DFT/sub-vasp.sh
DFT/POTCAR-Sr
DFT/POTCAR-B
```

For a 40 GPa deployment, `acnn_deploy` writes the pressure to VASP as:

```text
PSTRESS = 400
```

VASP `PSTRESS` is in kB, so 400 kB corresponds to 40 GPa. Check the remaining
INCAR settings, including:

```text
ENCUT
KSPACING
ISMEAR
SIGMA
EDIFF
NELM
```

For ARES, review:

```text
DFT/dyn_ares_in
DFT/sub-ares.sh
DFT/*.upf
```

The deployment pressure is also written into the generated ARES input,
phase-diagram script, and relaxation commands.


### Step 1: Build the Iteration-0 Data Set

The two modes prepare `RSS/Base` differently.

In online mode, the first loop iteration runs:

```console
$ cd DFT
$ ./init_online ../RSS/comp.txt
```

`init_online`:

1. reads the unique formulas from `RSS/comp.txt`;
2. shuffles the formula order;
3. distributes `total_structures` among those formulas;
4. writes a CALYPSO `input.dat`;
5. runs `calypso.x`; and
6. converts `POSCAR_*` into `.res` files in `RSS/Base`.

Its run-specific files are retained under:

```text
DFT/INIT/IT0-YYYYMMDD-HHMMSS/
```

The allocation used for the bootstrap can be inspected in:

```text
comp.init.txt
formulas.shuffled.txt
```

The second column of `RSS/comp.txt` is deliberately ignored during this one
bootstrap step. It becomes active later when the online relaxation queue is
created.

In offline mode, the user fills `RSS/Base` before starting `auto`. The
provided generators can be used manually. For AIRSS:

```console
$ cd RSS
$ mkdir -p gen
$ cd gen
$ ../dyn_gcs ../comp.txt 4
```

For CALYPSO:

```console
$ ../dyn_gcs_calypso ../comp.txt 4
```

Both generators read a composition and generation count from each
`comp.txt` line:

```text
SrB   200
SrB2  200
Sr2B  200
```

Check the element list, pair distances, volumes, and executable paths in the
generator before use. Move the generated `.res` files into the actual initial
pool:

```console
$ mkdir -p ../Base
$ find . -name '*.res' -type f -exec mv {} ../Base/ \;
$ cd ../..
```

In both modes, iteration 0 then samples up to `RANDMAX` structures from
`RSS/Base` for DFT:

```bash
RANDMAX=500
```

Edit `RANDMAX` in `DFT/mkdft` to change the initial labeling set size.


### Step 2: Submit DFT Labeling

For a VASP deployment, the automated command sequence is:

```console
$ cd DFT
$ ./mkdft vasp > log 2>&1
$ cd ..
$ acnn_wait FPSrB
```

For ARES, use:

```console
$ cd DFT
$ ./mkdft ares > log 2>&1
$ cd ..
$ acnn_wait FPSrB
```

In iteration 0, `mkdft` samples from `RSS/Base`. In later iterations, it uses
the structures selected by the previous post-processing stage:

```text
RELAX/IT*/FPC/SCF/
RELAX/IT*/FPC/OPT/
```

For VASP, `batch_vasp` creates one calculation directory per structure,
writes `POSCAR`, `INCAR`, and `POTCAR`, copies the Slurm script, and submits
the job. The active-learning labeling step uses the `scf` calculation type.


### Step 3: Convert DFT Results into ACNN Data

After all DFT jobs with the matching job name finish, the workflow runs:

```console
$ cd XSF
$ ./ry > log 2>&1
$ cd ..
```

The completed DFT results are converted into [`.xsf`
files]({{ '/technical-reference/xsf-file/' | relative_url }}) under:

```text
XSF/IT0/
XSF/IT1/
...
```

These files contain the structures, energies, forces, and stresses used for
ACNN training. `ry` also checks the generated data and should not be allowed
to silently continue with an empty iteration directory.

Check the result with:

```console
$ find XSF/IT0 -name '*.xsf' | wc -l
$ tail -n 50 XSF/log
```


### Step 4: Build the Phase Diagram

The automated command is:

```console
$ cd PD
$ ./mkpd > log 2>&1
$ cd ..
```

`mkpd` combines the current labeled structures with `SEED/*.res`, adds the
pressure-volume contribution to the energy, and builds convex-hull outputs.
For iteration 0, typical files include:

```text
PD/IT0/cam
PD/IT0/cam1
PD/IT0/cam2
PD/IT0/*.html
PD/IT0/*.agr
```

The enthalpy correction uses:

```bash
EVpG=0.0062415091
H = EVpG * PRESS * VOL + energy
```

`PRESS` is written in GPa by `acnn_deploy`. For binary and ternary systems,
plot files can also be generated through `gracebat` and `Rscript` when those
commands are installed.


### Step 5: Train or Restart the ACNN Potential

The automated command sequence is:

```console
$ cd POT
$ ./tr
$ cd ..
$ acnn_wait TRAINSrB
```

`POT/tr` creates:

```text
POT/IT0/DT/
POT/IT0/in.acnn
POT/IT0/sub.sh
```

It collects all available iteration training data, determines the data count,
and writes the ACNN input. Important generated defaults include:

```text
types         = Sr,B
rcut          = 6.5
acut          = 6.0
nbatch        = 1e5
batchsize     = 4
device        = cpu
float_type    = float64
hasattention  = false
```

Restart behavior:

| Iteration | Training behavior |
| --- | --- |
| `IT=0` | Train from scratch. |
| `IT=1` | Restart from `../IT0/model/model-last`. |
| `IT>=2` | Restart from the preceding `model-restart/model-last`. |

Review the cutoff, basis size, loss weights, training steps, batch size, and
device before the production run.


### Step 6A: Online Generation and Relaxation

After training, online mode runs:

```console
$ cd RELAX
$ ./update_bonds_scale >> log 2>&1
$ ./dyn_online_slurm_submit ../RSS/comp.txt >> log 2>&1
$ cd ..
$ acnn_wait RELAXSrB
```

The submit script first merges duplicate formulas:

```text
comp.txt
   |
   +--> comp.merged.txt
          |
          +--> comp.expanded.txt
                 |
                 +--> comp.queue.txt
```

For example, with `batch_popsize=50`:

```text
SrB 120
```

is divided into:

```text
SrB 50 0001
SrB 50 0002
SrB 20 0003
```

The expanded batches are shuffled before being written to
`RELAX/comp.queue.txt`. Queue state is protected with `flock`, so multiple
worker slots can claim different batches without claiming the same line.

Each Slurm worker receives:

```text
--ntasks=1
--cpus-per-task=slots_per_task
```

The worker then starts `slots_per_task` independent shell slots. Each slot
repeatedly:

1. locks the shared queue;
2. claims the next unassigned batch;
3. runs CALYPSO for that formula and batch;
4. converts the generated structures;
5. relaxes them with the newest ACNN model;
6. checks bond and pressure limits; and
7. retains the low-energy structures.

The ACNN command has the form:

```console
$ acnn_relax -p 40 -m MODEL -t Sr,B
```

Generated and relaxed files are organized under `RELAX/IT*/` by composition
and batch.

Online assignment stops when the queue is empty or when the number of
`*limit*.res` files exceeds `LIM_MAX`, whose default is 100. The number of
limit structures is also the main practical signal for deciding whether to
increase the population in `RSS/comp.txt` for the next iteration.


### Step 6B: Offline Batch Relaxation

Offline mode runs:

```console
$ cd RELAX
$ echo "y" | nohup ./dyn_batch_relax_bfgs > log 2>&1
$ cd ..
$ acnn_wait RELAXSrB
```

The script selects the newest trained model:

```bash
pot=$(ls "$pot_dir"/model*/model-* | sort -V | tail -n 1)
```

It samples candidate structures from `RSS/Base`, splits them into groups, and
submits Slurm tasks. Important generated parameters are:

```text
press   = 40
frame   = 500
group   = 50
warp    = 4
job_max = 4
```

| Parameter | Meaning |
| --- | --- |
| `frame` | Number of candidate structures sampled for relaxation. |
| `group` | Number of structures in one group file. |
| `warp` | Number of group relaxations packed into one Slurm job. |
| `job_max` | Maximum number of running relaxation jobs. |

Before submission, the script prints its dynamic parameters and asks:

```text
CHECK THE PARAMETERS: [y/n]
```

The top-level `auto` script supplies `y` automatically.

Unsafe trajectories are detected with:

```console
$ acnn_ckrelax -b "$min_bonds" -x "$bonds_scale" -p "$press" -c "$NCORE"
```

Structures that violate the safety criterion are copied to:

```text
RELAX/IT*/LIM/
```

The offline relaxation batch stops if the configured maximum number of limit
structures is exceeded.


### Step 7A: Online Post-Processing

Online mode runs:

```console
$ cd RELAX
$ ./ppr_online >> log 2>&1
$ cd ..
```

`ppr_online` gathers the relaxed structures, merges equivalent formula-unit
compositions, builds composition-resolved hull data, and selects candidates
for the next DFT round. It uses:

```text
ncomp=120
nstruct=5
```

The selected compositions are recorded in:

```text
RELAX/IT*/PPR/selected_compositions.tsv
```

The per-composition selections are written below:

```text
RELAX/IT*/PPR/selected/
```

The combined selection is written to:

```text
RELAX/IT*/PPR/final_selection.tsv
```

and the chosen `.res` structures are copied into the next-round FPC
directories. Selection can continue briefly after every per-composition file
exists because the final deduplication and merged selection are separate
steps.


### Step 7B: Offline Post-Processing

Offline mode runs:

```console
$ cd RELAX
$ ./ppr >> log 2>&1
$ cd ..
```

`ppr` collects relaxed `*out.res` files into:

```text
RELAX/IT*/RES/
```

It then runs structure statistics, energy processing, convex-hull analysis,
and duplicate removal. The candidates for the next DFT iteration are written
to:

```text
RELAX/IT*/FPC/SCF/
RELAX/IT*/FPC/OPT/
```

`FPC/SCF` normally contains low-energy or hull-relevant candidates.
`FPC/OPT` contains limit or unsafe structures that need more careful
first-principles treatment.

For systems with multiple chemical orders, `ppr` can focus on a particular
n-component hull:

```console
$ ./ppr --focus 2
```

After offline post-processing, trajectory directories can be cleaned with:

```console
$ cd RELAX
$ ./clean_traj >> log 2>&1 &
$ cd ..
```


Adapting the Template to Another System
---------------------------------------

Most chemical and workflow settings are generated directly from the
`acnn_deploy` command. Use the following checklist before launching a new
system:

| Item | Where to check |
| --- | --- |
| System tag | `auto-online`, `auto`, generated Slurm job names |
| Element list | `POT/tr`, relaxation commands, structure generators |
| Minimum bond distances | `in.seed`, online CALYPSO input, relaxation safety checks |
| Pressure | `DFT/dyn_*_in`, `PD/mkpd`, ACNN relaxation commands |
| Pseudopotentials | `DFT/POTCAR-*` or `DFT/*.upf` |
| Slurm resources | `DFT/sub-*.sh`, `POT/sub.sh`, offline and online relaxation scripts |
| Online composition population | `RSS/comp.txt` |
| Online concurrency | `slots_per_task`, `max_tasks`, `batch_popsize` |
| Offline initial pool | `RSS/Base/*.res`, `RANDMAX`, `frame` |

The generated relaxation scripts contain the absolute project path at deploy
time. If the deployed run directory is moved afterward, inspect and update
that path before submitting relaxation jobs.


Common Problems
---------------

### Online Bootstrap Produces No Structures

Check:

```console
$ command -v calypso.x
$ cat RSS/comp.txt
$ find DFT/INIT -name calypso.log -print
$ find RSS/Base -name '*.res' | wc -l
```

The formula must contain valid element symbols, and all pair distances used by
that formula should be present in the deployed bond table.


### Offline `mkdft` Finds No Structures

For iteration 0:

```console
$ find RSS/Base -name '*.res' | head
```

For later iterations:

```console
$ find RELAX/IT$((IT-1))/FPC -name '*.res'
```


### DFT Jobs Are Not Submitted

Check Slurm, the partition name, and user job limits:

```console
$ command -v safe_sbatch
$ squeue -u "$USER"
```

For VASP, verify that the pseudopotential filenames match the element names:

```text
POTCAR-Sr
POTCAR-B
```


### No Training Data Is Generated

Check whether the DFT calculations finished successfully:

```console
$ find XSF/IT0 -name '*.xsf' | wc -l
$ tail -n 100 XSF/log
```

`POT/tr` depends on the XSF data. An empty `XSF/IT*` directory cannot produce
a valid training round.


### Too Many Structures Hit the Bond Limit

Inspect:

```console
$ find RELAX/IT*/LIM -name '*limit*.res' | wc -l
```

Review `min_bonds` and `bonds_scale` in `in.seed`. For online mode, also
inspect the latest output from `RELAX/update_bonds_scale`. Unrealistic pair
distances can cause either relaxation mode to stop early.


### The Phase Diagram Is Empty

Check:

```console
$ cat PD/IT0/cam
$ find SEED -name '*.res'
```

The phase diagram uses both labeled structures and seed structures. Missing
elemental endpoints can make the hull empty or difficult to interpret.


Minimal Command Summary
-----------------------

For a configured online run:

```console
$ ./auto-online > log 2>&1
```

For a configured offline run:

```console
$ ./auto > log 2>&1
```

For one manual shared DFT/training sequence using VASP:

```console
$ cd DFT && ./mkdft vasp > log 2>&1 && cd ..
$ acnn_wait FPSrB

$ cd XSF && ./ry > log 2>&1 && cd ..

$ cd PD && ./mkpd > log 2>&1 && cd ..

$ cd POT && ./tr && cd ..
$ acnn_wait TRAINSrB
```

Continue online with:

```console
$ cd RELAX
$ ./update_bonds_scale >> log 2>&1
$ ./dyn_online_slurm_submit ../RSS/comp.txt >> log 2>&1
$ cd ..
$ acnn_wait RELAXSrB
$ cd RELAX && ./ppr_online >> log 2>&1 && cd ..
```

Or continue offline with:

```console
$ cd RELAX && echo "y" | ./dyn_batch_relax_bfgs > log 2>&1 && cd ..
$ acnn_wait RELAXSrB
$ cd RELAX && ./ppr >> log 2>&1 && cd ..
```

> ACNN has been tested across multiple systems and demonstrates high
> efficiency, reliability, and success rate (except for those who don't know
> how to use it).
