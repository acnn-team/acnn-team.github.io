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

Attention Coupled Neural Network(ACNN) is a machine learning potential(MLP) approach for atomic simulations.
The original purpose was to serve as part of artificial intelligence module for the _ab initio_ Real-space Electronic Structure (ARES) code. 
However it can be relatively straightforward to be used independently. 

[//]: # (The main design concept is to couple a multi-head self-attention)

[//]: # (mechanism with traditional short-range machine learning potentials to improve)

[//]: # (the accuracy of machine learning potentials, allows users to train machine)

[//]: # (learning potential models with different levels of accuracy, including or excluding)

[//]: # (attention mechanisms, based on their specific accuracy requirements.)

ACNN is primarily written in C++ and based on LibTorch (C++ Distributions of PyTorch).
Some practical utilities are placed in `scripts/`,
the external program interfaces are placed in `interface/`.

Currently, ACNN can interface with:
- ARES-PW/MD/Phonon
- LAMMPS (Including LAMMPS based packages)
- Python
- CALYPSO
- AIRSS

Licence and Citation
--------------------

All the notes are released under the [GPL 2.0 licence](https://www.gnu.org/licenses/gpl-2.0.html). See the `LICENCE` file for more details. 

References
----------

[//]: # (&#40;1&#41; C.J. Pickard and R.J. Needs, Phys. Rev. Lett., **97**, 045504 &#40;2006&#41; [[Link][1]]  )

[//]: # (&#40;2&#41; C.J. Pickard and R.J. Needs, J. Phys.: Condens. Matter, **23**, 053201 &#40;2011&#41; [[Link][2]]  )

[//]: # (&#40;3&#41; C.J. Pickard and R.J. Needs, Nat. Phys., **3**, 473 &#40;2007&#41; [[Link][3]]  )

[//]: # (&#40;4&#41; C.J. Pickard and R.J. Needs, Nat. Mater., **9**, 624 &#40;2010&#41; [[Link][4]]  )

[//]: # (&#40;5&#41; C.J. Pickard and R.J. Needs, Nat. Mater., **7**, 775 &#40;2008&#41; [[Link][5]]  )

[//]: # (&#40;6&#41; A.J. Morris, C.J. Pickard and R.J. Needs, Phys. Rev. B, **78**, 184102 &#40;2008&#41; [[Link][6]]  )

[//]: # (&#40;7&#41; G. Schusteritsch and C.J. Pickard, Phys. Rev. B, **90**, 035424 &#40;2014&#41; [[Link][7]]  )

[//]: # ()
[//]: # ([1]: https://doi.org/10.1103/PhysRevLett.97.045504)

[//]: # ([2]: https://doi.org/10.1088/0953-8984/23/5/053201)

[//]: # ([3]: https://doi.org/10.1038/nphys625)

[//]: # ([4]: https://doi.org/10.1038/nmat2796)

[//]: # ([5]: https://doi.org/10.1038/nmat2261)

[//]: # ([6]: https://doi.org/10.1103/PhysRevB.78.184102)

[//]: # ([7]: https://doi.org/10.1103/PhysRevB.90.035424)
