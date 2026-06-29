---
title: "工具"
layout: single
classes: wide
lang: zh
lang-ref: utilities
sidebar:
  nav: "docs"
---

ACNN 提供了一组小型命令行工具，用于部署、数据转换、数据集检查、相图分析和结构格式转换。本页列出主动学习结构搜索流程中最常用的工具。

## ACNN 工作流工具

### `acnn_deploy`

部署一个完整的主动学习运行目录。

```console
$ acnn_deploy -p 40 -s SrB -b Sr-Sr=2.075,Sr-B=1.671,B-B=1.267 -n amd9654 -e vasp
```

必需参数：

| 参数 | 含义 |
| --- | --- |
| `-p, --pressure` | 目标压强，单位 GPa。 |
| `-s, --seed` | 用于文件名和作业名的体系标签。 |
| `-b, --minbonds` | 成对最小键长限制。 |
| `-n, --partition` | Slurm 分区或节点组。 |
| `-e, --backend` | 第一性原理计算后端：`vasp` 或 `ares`。 |

`acnn_deploy` 会把指定压强写入生成的 DFT、相图和优化脚本中。

### `acnn_wait`

等待一组 Slurm 作业完成。在标准循环中，它用于等待 DFT 标注、ACNN 训练和 ACNN 优化。

```console
$ acnn_wait FPSrB
$ acnn_wait TRAINSrB
$ acnn_wait RELAXSrB
```

### `acnn_outcar2seed`

把 VASP `OUTCAR` 转换成 DFT 优化后的 seed 结构。

```console
$ acnn_outcar2seed OUTCAR SrB-end-Sr-1.res
```

输出是适合放入 `SEED/` 的 [`.res`](/zh/technical-reference/res-file/) 文件。

### `acnn_checkdt`

在 DFT-to-XSF 转换后检查 ACNN 训练数据集。

```console
$ acnn_checkdt 50
```

通常在 `XSF/ry` 为当前轮次生成 [`.xsf`](/zh/technical-reference/xsf-file/) 文件后运行。

### `acnn_bsafe`

检查结构是否满足成对最小键长限制。

```console
$ acnn_bsafe -b "Sr-Sr=2.075,Sr-B=1.671,B-B=1.267" -x 0.6
```

在提交昂贵的 DFT 或优化作业前，可以用它筛掉不安全结构。

### `acnn_pdhtml`

分析一批 `.res` 结构并生成相图 HTML 输出。

```console
$ acnn_pdhtml
```

相图流程通常通过生成的 `PD/mkpd` 脚本调用它。

### `acnn_ckrelax`

检查 ACNN 优化输出并报告优化状态，通常用于优化后的后处理流程。

```console
$ acnn_ckrelax -b "Sr-Sr=2.075,Sr-B=1.671,B-B=1.267" -x 0.6 -p 40 -c 192
```

### `acnn_limitjob`

限制或筛选从优化结果中选出的结构，再送回下一轮 DFT。

```console
$ acnn_limitjob RELAXSrB 1000
```

## 结构转换工具

### `cabal`

`cabal` 是用于结构格式转换的 AIRSS 工具。它常用于 `.res`、POSCAR、CIF 等格式之间的转换。

```console
$ cabal res poscar < input.res > POSCAR
$ cabal poscar res < POSCAR > output.res
```

通用用法：

```console
$ cabal in out < seed.in > seed.out
```

### `ca`

`ca` 是 AIRSS 结构分析工具的封装，可用于检查 `.res` 文件、组分、对称性，以及准备相图输入。

### `symm`

查找结构的空间群：

```console
$ symm test
```

对于名为 `test.res` 的文件，`symm test` 会读取该结构并报告检测到的对称性。
