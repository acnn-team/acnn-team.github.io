---
title: "外部程序接口"
layout: single
classes: wide
lang: zh
lang-ref: external-utilities
sidebar:
  nav: "docs"
---

ACNN 的作用是把机器学习势与第一性原理计算、结构搜索、分子动力学和声子计算等工作流连接起来。本页概述 ACNN 可以结合的主要外部程序，以及这些组合可以完成什么任务。

## 总览

| 程序或工作流 | 与 ACNN 结合后可以做什么 |
| --- | --- |
| LAMMPS | 使用 ACNN 势进行分子动力学和大规模原子模拟。 |
| CALYPSO | 生成候选结构，并交给 ACNN 主动学习流程评估和筛选。 |
| AIRSS | 生成随机结构，并提供 ACNN 工作流常用的结构格式工具。 |
| ARES | 为 ACNN 模型训练和验证提供第一性原理数据。 |
| ARES-PHONON | 使用 ACNN 势加速声子计算。 |

## LAMMPS

[LAMMPS](https://www.lammps.org) 是由 LAMMPS 社区开发和维护的大规模分子动力学程序，广泛用于经典分子动力学、材料模拟和大规模原子体系计算。

与 ACNN 结合后，LAMMPS 可以使用训练好的 ACNN 机器学习势，在分子动力学过程中提供能量、原子力和应力。这使得用户可以进行比直接第一性原理分子动力学更大尺度、更长时间的模拟，同时保留来自第一性原理数据训练得到的势能模型。

Typical role in ACNN:

- 使用训练好的 ACNN 势进行分子动力学。
- 将第一性原理训练得到的模型扩展到更大体系和更长模拟时间。
- 用 ACNN 力场研究有限温度下的结构和动力学行为。

参考文献：

1. S. Plimpton, _J. Comput. Phys._ **117**, 1 (1995). [[Link](https://doi.org/10.1006/jcph.1995.1039)]

## CALYPSO

[CALYPSO](http://calypso.cn) 是由马琰铭课题组开发的晶体结构预测方法和软件，用于在不同组分和压力条件下生成并演化候选晶体结构，寻找稳定结构。

与 ACNN 结合后，CALYPSO 可以为主动学习循环提供候选结构。ACNN 可以进一步对这些结构进行评估、优化、筛选，并选择重要结构进入第一性原理标注。这种组合适合高压结构搜索和复杂组分空间中大量候选结构的高效筛选。

Typical role in ACNN:

- 为晶体结构预测生成候选结构。
- 为 ACNN 主动学习提供多样化结构池。
- 帮助 ACNN 更高效地探索高压和多组分结构空间。

参考文献：

1. Y. Wang, J. Lv, L. Zhu, and Y. Ma, _Phys. Rev. B_ **82**, 094116 (2010). [[Link](https://doi.org/10.1103/PhysRevB.82.094116)]
2. Y. Wang, J. Lv, L. Zhu, and Y. Ma, _Comput. Phys. Commun._ **183**, 2063 (2012). [[Link](https://doi.org/10.1016/j.cpc.2012.05.008)]

## AIRSS

[AIRSS](https://www.mtg.msm.cam.ac.uk/Codes/AIRSS) 是由 Chris Pickard 及合作者开发的 ab initio random structure searching 方法。它能够生成随机但化学上合理的结构，并提供成熟的结构格式处理和结构分析工具。

ACNN 可以通过通用的 `.res` 结构格式与 AIRSS 生成的结构结合使用。AIRSS 提供多样化的初始候选结构，ACNN 主动学习则可以迭代训练势函数、优化结构，并选择重要构型进行第一性原理计算。当需要宽泛且较少偏置的初始结构池时，这种组合很有用。

Typical role in ACNN:

- 为主动学习结构搜索生成随机初始结构。
- 提供基于 `.res` 的结构处理和分析工具。
- 在未知稳定结构时提供宽泛的候选结构池。

参考文献：

1. C. J. Pickard and R. J. Needs, _Phys. Rev. Lett._ **97**, 045504 (2006). [[Link](https://doi.org/10.1103/PhysRevLett.97.045504)]
2. C. J. Pickard and R. J. Needs, _J. Phys.: Condens. Matter_ **23**, 053201 (2011). [[Link](https://doi.org/10.1088/0953-8984/23/5/053201)]

## ARES

[ARES](https://8.219.233.107) 是由吉林大学谢禹教授团队领导开发的实空间第一性原理电子结构程序，可用于密度泛函理论计算，得到原子体系的能量、力、应力和优化结构。

在 ACNN 工作流中，ARES 可以作为第一性原理后端，为 ACNN 训练提供标注数据。ACNN 从 ARES 计算的结构中学习后，可以用训练好的势函数进行更快速的结构优化、筛选、分子模拟或后续性质计算。

Typical role in ACNN:

- 为 ACNN 训练数据生成第一性原理标注。
- 为主动学习提供能量、力、应力和优化结构。
- 作为 ARES-ACNN 结构搜索和性质计算工作流中的 DFT 后端。

参考文献：

1. L. Xue, Q. Xu, C. Ma, W. Mi, Y. Wang, Y. Xie, and Y. Ma, _Phys. Rev. B_ **110**, 155157 (2024). [[Link](https://doi.org/10.1103/PhysRevB.110.155157)]

## ARES-PHONON

ARES-PHONON 是基于非对角超胞有限位移法的声子计算程序，可与机器学习势结合以降低力计算成本。

与 ACNN 结合后，ARES-PHONON 可以使用 ACNN 势加速声子计算，尤其适用于需要对大量位移超胞进行力计算、而直接第一性原理计算成本较高的体系。这一组合把 ACNN 训练得到的力模型与晶格动力学计算连接起来。

Typical role in ACNN:

- 使用训练好的 ACNN 势加速位移超胞的力计算。
- 将 ACNN 力模型与声子和晶格动力学工作流连接起来。
- 降低直接第一性原理声子计算成本较高体系的计算开销。

参考文献：

1. Qian Wang, Jiaxiang Li, and Yu Xie, "ARES-Phonon: Phonon Calculation Package using Nondiagonal Supercell Finite Displacement Method with Machine Learning", arXiv:2503.11013 (2025). [[Link](https://arxiv.org/abs/2503.11013)]
