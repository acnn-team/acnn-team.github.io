---
title: "External Interfaces"
layout: single
classes: wide
lang: en
lang-ref: external-utilities
sidebar:
  nav: "docs"
---

ACNN is designed to connect machine-learning potentials with first-principles calculation, structure search, molecular dynamics, and phonon workflows. This page summarizes the main external programs that can work together with ACNN and what each combination is useful for.

## Overview

| Program or workflow | What it does with ACNN |
| --- | --- |
| LAMMPS | Runs molecular dynamics and large-scale atomistic simulations with ACNN potentials. |
| CALYPSO | Generates candidate structures that can be evaluated and filtered through ACNN active learning. |
| AIRSS | Generates random structures and provides structure-format utilities used in ACNN workflows. |
| ARES | Provides first-principles data for ACNN model training and validation. |
| ARES-PHONON | Uses ACNN potentials to accelerate phonon calculations. |

## LAMMPS

[LAMMPS](https://www.lammps.org) is a large-scale molecular dynamics program developed and maintained by the LAMMPS community. It is widely used for classical molecular dynamics, materials modeling, and large atomistic simulations.

When combined with ACNN, LAMMPS can use a trained ACNN machine-learning potential to provide energies, atomic forces, and stress during molecular dynamics. This makes it possible to run simulations that are much larger or longer than direct first-principles molecular dynamics while retaining a data-driven potential trained from first-principles results.

Typical role in ACNN:

- Run molecular dynamics with trained ACNN potentials.
- Extend first-principles-trained models to larger systems and longer simulation times.
- Evaluate finite-temperature structural and dynamical behavior with an ACNN force field.

References:

1. S. Plimpton, _J. Comput. Phys._ **117**, 1 (1995). [[Link](https://doi.org/10.1006/jcph.1995.1039)]

## CALYPSO

[CALYPSO](http://calypso.cn) is a crystal structure prediction method and software package developed by the Ma group. It is designed to search for stable structures by generating and evolving candidate crystal structures across different compositions and pressures.

When combined with ACNN, CALYPSO can provide candidate structures for the active-learning loop. ACNN can then help evaluate, relax, filter, and select promising structures for further first-principles labeling. This combination is useful for high-pressure structure searches and complex composition spaces where many candidate structures must be screened efficiently.

Typical role in ACNN:

- Generate candidate structures for crystal structure prediction.
- Provide diverse structure pools for ACNN active learning.
- Help ACNN explore high-pressure and multi-composition structure spaces more efficiently.

References:

1. Y. Wang, J. Lv, L. Zhu, and Y. Ma, _Phys. Rev. B_ **82**, 094116 (2010). [[Link](https://doi.org/10.1103/PhysRevB.82.094116)]
2. Y. Wang, J. Lv, L. Zhu, and Y. Ma, _Comput. Phys. Commun._ **183**, 2063 (2012). [[Link](https://doi.org/10.1016/j.cpc.2012.05.008)]

## AIRSS

[AIRSS](https://www.mtg.msm.cam.ac.uk/Codes/AIRSS) is the Ab Initio Random Structure Searching method developed by Chris Pickard and collaborators. It generates random but chemically sensible structures and provides mature utilities for handling structure formats and analyzing generated structures.

ACNN can work with AIRSS-generated structures through the common `.res` structure format. AIRSS can supply diverse initial candidates, while ACNN active learning can iteratively train potentials, relax structures, and select important configurations for first-principles calculations. This is useful when a broad and unbiased initial structure pool is needed.

Typical role in ACNN:

- Generate random initial structures for active-learning searches.
- Provide `.res`-based structure handling and analysis utilities.
- Supply broad candidate pools when the stable structures are not known in advance.

References:

1. C. J. Pickard and R. J. Needs, _Phys. Rev. Lett._ **97**, 045504 (2006). [[Link](https://doi.org/10.1103/PhysRevLett.97.045504)]
2. C. J. Pickard and R. J. Needs, _J. Phys.: Condens. Matter_ **23**, 053201 (2011). [[Link](https://doi.org/10.1088/0953-8984/23/5/053201)]

## ARES

[ARES](https://8.219.233.107) is a real-space first-principles electronic-structure package led by Prof. Yu Xie's group at Jilin University. It is developed for density-functional-theory calculations and can compute energies, forces, stresses, and relaxed structures for atomistic systems.

In ACNN workflows, ARES can act as the first-principles backend that generates labeled data for ACNN training. ACNN can then learn from ARES-calculated structures and use the trained potential for faster relaxation, screening, molecular simulation, or downstream property workflows.

Typical role in ACNN:

- Generate first-principles labels for ACNN training data.
- Provide energies, forces, stresses, and relaxed structures for active learning.
- Serve as the DFT backend in ARES-ACNN structure-search and property workflows.

References:

1. L. Xue, Q. Xu, C. Ma, W. Mi, Y. Wang, Y. Xie, and Y. Ma, _Phys. Rev. B_ **110**, 155157 (2024). [[Link](https://doi.org/10.1103/PhysRevB.110.155157)]

## ARES-PHONON

ARES-PHONON is a phonon calculation package based on the nondiagonal supercell finite-displacement method. It is designed for phonon calculations and can be combined with machine-learning potentials to reduce the cost of force evaluation.

When combined with ACNN, ARES-PHONON can use an ACNN potential to accelerate phonon workflows, especially for systems where direct first-principles force calculations for many displaced supercells are expensive. This connects ACNN-trained force models with lattice-dynamics calculations.

Typical role in ACNN:

- Use trained ACNN potentials to accelerate force calculations for displaced supercells.
- Connect ACNN force models with phonon and lattice-dynamics workflows.
- Reduce the cost of phonon calculations for systems where direct first-principles force evaluation is expensive.

References:

1. Qian Wang, Jiaxiang Li, and Yu Xie, "ARES-Phonon: Phonon Calculation Package using Nondiagonal Supercell Finite Displacement Method with Machine Learning", arXiv:2503.11013 (2025). [[Link](https://arxiv.org/abs/2503.11013)]
