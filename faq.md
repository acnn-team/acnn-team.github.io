---
title: "FAQ"
layout: single
classes: wide top-page
lang: en
lang-ref: faq
permalink: /faq/
---

<div class="faq-list" markdown="1">

<details class="faq-item" markdown="1">
<summary>Does the ACNN structure-prediction workflow support job systems other than Slurm?</summary>

Currently, the automated ACNN structure-prediction workflow only supports Slurm.

The workflow scripts submit and monitor first-principles calculations through Slurm job commands. Other schedulers, such as PBS, LSF, or local workstation queues, are not supported by the current automated workflow. Users who want to run ACNN with another job system would need to adapt the submission and monitoring scripts manually.

</details>

<details class="faq-item" markdown="1">
<summary>In crystal structure prediction, what are atomic volume and minimum bond length used for? How should I set them?</summary>

In the ACNN crystal-structure-prediction workflow, atomic volume and minimum bond length are input parameters used during candidate structure generation and relaxation. They help keep the generated and relaxed structures physically reasonable before the next round of first-principles labeling.

The atomic volume provides an initial estimate of the cell size for generated candidate structures. It should be chosen according to the target composition, pressure, and expected density. A practical starting point is to estimate it from known structures, previous calculations, experimental volumes, or a short set of trial first-principles relaxations under similar conditions.

If you already have many structures for related materials at known pressures, ACNN also provides a helper script, `acnn_estvol`, to estimate atomic volumes by a simple least-squares fit. The structures are read from standard input:

```bash
cat *.res | acnn_estvol
```

The minimum bond length defines the shortest allowed distance for each element pair, such as `A-A`, `A-B`, and `B-B`. In the ACNN workflow, these values are used as safety checks during structure relaxation. If atoms become unrealistically close, the structure can be flagged or stopped instead of being treated as a valid candidate.

For example, the [crystal structure prediction tutorial]({{ '/how-to-guides/high-pressure-binary-searches/' | relative_url }}) defines pairwise limits through `min_bonds`, and applies a scaling factor through `bonds_scale`:

```bash
min_bonds="Sr-Sr=2.075,Sr-B=1.671,B-B=1.267"
bonds_scale=".7"
```

A good starting strategy is:

1. Estimate reasonable pair distances from known compounds, covalent/ionic radii, or relaxed reference structures.
2. Use slightly conservative values at the beginning so that obviously unphysical structures are filtered out.
3. Check how many structures hit the bond limit during relaxation.
4. If too many structures are rejected, review `min_bonds` and `bonds_scale`; overly large minimum distances may stop valid candidates too early.
5. If many structures contain unrealistically short contacts, increase the relevant minimum bond distances or inspect the structure-generation settings.

In short, atomic volume controls the initial cell-size scale, while minimum bond length controls short-range structural safety during relaxation.

</details>

</div>
