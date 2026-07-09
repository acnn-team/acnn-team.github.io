---
title: "关于 ACNN"
layout: single
classes: wide
lang: zh
lang-ref: about-notes
sidebar:
  nav: "docs"
---

Attention Coupled Neural Network (ACNN) 是一种面向原子模拟的机器学习势框架，可用于结构弛豫、分子动力学、声子相关计算、高通量筛选和晶体结构预测等任务。它最初作为 _ab initio_ Real-space Electronic Structure (ARES) 程序的人工智能模块开发，也可以作为相对独立的工具包使用。

ACNN 面向的是这样一类问题：第一性原理计算足够准确，但计算成本较高。它从第一性原理数据中学习，并提供可反复用于后续模拟的高效原子间势。因此，当用户希望把 DFT 质量的信息扩展到更大的体系、更长的时间尺度或更多候选结构时，ACNN 可以提供有效的加速。

## ACNN 提供什么

- 面向原子结构优化和模拟的神经网络原子间势。
- 基于 ACNN 模型的结构弛豫引擎 ACNN-Relax。
- 面向分子动力学、声子加速、结构弛豫和结构筛选的工作流。
- 用于 DFT 标注、数据库构建、模型训练和迭代改进的主动学习工具。
- 与结构生成程序结合时用于晶体结构预测的结构搜索工具。

## 程序接口

ACNN 主要使用 C++ 编写，并基于 LibTorch（PyTorch 的 C++ 发行版）实现。它可以与多种外部程序协同工作：

- ARES 和 ARES-PHONON：用于实空间第一性原理计算和声子计算工作流。
- LAMMPS：用于分子动力学和大规模原子模拟。
- CALYPSO 和 AIRSS：用于晶体结构生成和结构搜索工作流。
- Python 脚本：用于数据处理、工作流控制和结果分析。

## 文档导航

- **[Getting Started]({{ '/zh/installation/' | relative_url }})**：安装、示例、贡献者和项目概览。
- **[Tutorials]({{ '/zh/how-to-guides/high-pressure-binary-searches/' | relative_url }})**：实际 ACNN 工作流教程。
- **[Technical Reference]({{ '/zh/technical-reference/utilities/' | relative_url }})**：文件格式、工具脚本和外部程序接口说明。
- **[External]({{ '/zh/external-utilities/' | relative_url }})**：ACNN 与其他科学计算软件的协同方式。

## 许可和引用

本站文档随 ACNN 相关笔记一同发布，采用 [GPL 2.0 licence](https://www.gnu.org/licenses/gpl-2.0.html)。
如需了解更具体的许可信息，请参考项目中的 `LICENCE` 文件。

## 参考文献

1. Jiaxiang Li _et al._, _npj Computational Materials_ **12**, 101 (2026). [[Link](https://www.nature.com/articles/s41524-026-01971-9)]
