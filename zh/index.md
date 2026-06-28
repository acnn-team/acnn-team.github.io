---
title: "关于 ACNN"
layout: single
classes: wide
lang: zh
lang-ref: about-notes
sidebar:
  nav: "docs"
---

Attention Coupled Neural Network (ACNN) 是一种面向原子模拟的机器学习势（machine learning potential, MLP）方法。
它最初作为 _ab initio_ Real-space Electronic Structure (ARES) 程序人工智能模块的一部分开发，也可以相对独立地用于模型训练、结构弛豫和原子模拟工作流。

ACNN 主要使用 C++ 编写，并基于 LibTorch（PyTorch 的 C++ 发行版）实现。
项目中的 `scripts/` 目录提供了一些实用工具，`interface/` 目录则包含与外部程序协同工作的接口。

目前，ACNN 可以与以下程序或工作流配合使用：

- ARES-PW/MD/Phonon
- LAMMPS 及基于 LAMMPS 的程序包
- Python
- CALYPSO
- AIRSS

许可和引用
--------------------

本站文档随 ACNN 相关笔记一同发布，采用 [GPL 2.0 licence](https://www.gnu.org/licenses/gpl-2.0.html)。
如需了解更具体的许可信息，请参考项目中的 `LICENCE` 文件。
