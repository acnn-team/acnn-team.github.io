---
title: "About ACNN"
layout: single
classes: wide
lang: en
lang-ref: about-notes
sidebar:
  nav: "docs"
permalink: /
---

Attention Coupled Neural Network (ACNN) is a machine-learning potential framework for atomistic simulations. It can be used for structure relaxation, molecular dynamics, phonon-related workflows, high-throughput screening, and crystal structure prediction. ACNN was originally developed as the artificial-intelligence module of the _ab initio_ Real-space Electronic Structure (ARES) code, and can also be used as an independent toolkit.

ACNN is designed for problems where direct first-principles calculations are accurate but expensive. It learns from first-principles data and provides an efficient interatomic potential that can be used repeatedly in downstream simulations. This makes ACNN useful whenever users need to extend DFT-quality information to larger systems, longer time scales, or many candidate structures.

## What ACNN Provides

- A neural-network interatomic potential for atomic structure optimization and simulation.
- ACNN-Relax, a structure relaxation engine based on trained ACNN models.
- Workflows for molecular dynamics, phonon acceleration, structure relaxation, and structure screening.
- Active-learning utilities for DFT labeling, database construction, model training, and iterative improvement.
- Structure-search utilities for crystal structure prediction when coupled with structure-generation programs.

## Program Interfaces

ACNN is primarily written in C++ and based on LibTorch, the C++ distribution of PyTorch. The toolkit can work together with several external programs:

- ARES and ARES-PHONON for real-space first-principles calculations and phonon workflows.
- LAMMPS for molecular dynamics and large-scale atomistic simulations.
- CALYPSO and AIRSS for crystal structure generation and structure-search workflows.
- Python-based scripts for data processing, workflow control, and analysis.

## Documentation Map

- **[Getting Started]({{ '/getting-started/installation/' | relative_url }})** introduces installation, examples, contributors, and this project overview.
- **[Tutorials]({{ '/tutorials/crystal-structure-prediction/' | relative_url }})** describe practical ACNN workflows.
- **[Technical Reference]({{ '/technical-reference/utilities/' | relative_url }})** records file formats, utilities, and external-program interfaces.
- **[External]({{ '/external/external-utilities/' | relative_url }})** explains how ACNN connects with other scientific software.

## Licence and Citation

All the notes are released under the [GPL 2.0 licence](https://www.gnu.org/licenses/gpl-2.0.html). See the `LICENCE` file for more details.

## References

1. Jiaxiang Li _et al._, _npj Computational Materials_ **12**, 101 (2026). [[Link](https://www.nature.com/articles/s41524-026-01971-9)]
