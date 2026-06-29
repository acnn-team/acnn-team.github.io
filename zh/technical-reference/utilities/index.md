---
title: "工具"
layout: single
classes: wide
lang: zh
lang-ref: utilities
sidebar:
  nav: "docs"
---

ACNN 提供了一组命令行工具，用于部署、DFT 输出转换、数据集检查、relaxation 后处理、相图分析、结构数据库和 Slurm 作业控制。本页整理 `interface/airss` 中所有 `acnn*` 工具。

## 部署和作业控制

### `acnn_deploy`

部署完整的主动学习运行目录。

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

五个参数都必须提供，并且每个参数只能出现一次。`acnn_deploy` 会把压强、后端、分区和键长限制写入生成的 DFT、相图、训练和优化脚本。

### `acnn_wait`

等待当前用户队列中指定作业名的 Slurm 作业全部结束。

```console
$ acnn_wait FPSrB
$ acnn_wait TRAINSrB
$ acnn_wait RELAXSrB
```

脚本内部会轮询：

```console
$ squeue -u $(whoami) -n JOB_NAME
```

每 20 秒检查一次。

### `acnn_limitjob`

限制 Slurm 作业数量；当指定作业名的数量低于阈值后才继续。

```console
$ acnn_limitjob RELAXSrB 1000
```

用法：

```console
$ acnn_limitjob <job_name> <max_count>
```

通常在批量提交脚本内部调用。

## DFT 输出转 seed 结构

### `acnn_outcar2seed`

把 VASP `OUTCAR` 转换成 DFT 优化后的 seed [`.res`](/zh/technical-reference/res-file/) 结构。

```console
$ acnn_outcar2seed OUTCAR SrB-end-Sr-1.res
```

用法：

```console
$ acnn_outcar2seed <OUTCAR> <SEED_NAME>
```

脚本使用 ASE 读取最终结构，经 `cabal poscar res` 转成 `.res`，提取 enthalpy 或最终 `TOTEN`，把压力从 kB 转换为 GPa，并用 `rres` 更新 `.res` 元数据。

### `acnn_ares2seed`

把 ARES 输出文件转换成 DFT 优化后的 seed `.res` 结构。

```console
$ acnn_ares2seed ares_output.dat SrB-end-Sr-1.res
```

用法：

```console
$ acnn_ares2seed <ares_output.dat> <SEED_NAME>
```

脚本从 ARES 输出中提取最终晶格、分数坐标、元素数目、enthalpy 和压力，写成 POSCAR 风格结构，再通过 `cabal` 和 `rres` 生成带元数据的 `.res`。

## DFT 输出转 ACNN 训练数据

### `acnn_ares2xsf`

把 ARES SCF 输出转换为 ACNN [`.xsf`](/zh/technical-reference/xsf-file/) 训练数据。

```console
$ acnn_ares2xsf ares_output.dat > sample.xsf
```

用法：

```console
$ acnn_ares2xsf <ARES output>
```

输出写到标准输出。脚本提取总能、晶格、笛卡尔坐标、力、体积和应力，并把应力转换为 ACNN 使用的 `VIRIAL` 块。

### `acnn_recycling`

把一个或多个第一性原理输出文件转换成 `.xsf` 训练集。

```console
$ acnn_recycling */OUTCAR
$ acnn_recycling --calc qe --outdir qe_xsf */scf.out
$ acnn_recycling --calc ares --outdir ares_xsf run*/scf_output
```

用法：

```console
$ acnn_recycling [--calc vasp|qe|ares] [--outdir xsf_output] FILE [FILE ...]
```

支持的计算后端：

| 后端 | 输入 |
| --- | --- |
| `vasp` | ASE 可读的 `OUTCAR`、`XDATCAR`、`goodStructures_POSCARS` 或 `goodStructs_POSCAR`。 |
| `qe` | Quantum ESPRESSO `espresso-out` 文件。 |
| `ares` | 脚本支持格式的 ARES SCF 输出。 |

输出文件写入 `--outdir`，文件名由源路径和 step 编号组成，例如 `runs_Al_fcc_OUTCAR_step000003.xsf`。

### `acnn_checkdt`

检查 `.xsf` 数据集中是否存在不合理的力。

```console
$ acnn_checkdt 50 XSF/IT0
```

用法：

```console
$ acnn_checkdt [force_threshold] [xsf_directory]
```

默认力阈值为 `100 eV/A`，默认检查当前目录。若某个 `.xsf` 文件中任意力分量绝对值超过阈值，脚本会把该文件重命名为去掉 `.xsf` 后缀的文件，使它不再作为训练数据被读取。

## Relaxation 后处理和安全检查

### `acnn_bsafe`

检查当前目录下所有 `.res` 结构是否满足成对最小键长限制。

```console
$ acnn_bsafe -b "Sr-Sr=2.075,Sr-B=1.671,B-B=1.267" -x 0.6
```

参数：

| 参数 | 含义 |
| --- | --- |
| `-b, --bond` | 逗号分隔的最小键长限制。 |
| `-x` | 所有最小键长的缩放因子，默认 `1.0`。 |

输出每个 `.res` 文件的结构名、压强和 `True`/`False`。对于不安全结构，还会报告违反限制的原子对、原子序号、实际距离和阈值。

### `acnn_ckrelax`

检查 ACNN relaxation 轨迹，把安全的最终结构转换成 `.res`；如果最终结构不安全，则寻找最后一个安全 step。

```console
$ export opt_dir="SrB-1 SrB-2 SrB-3"
$ acnn_ckrelax -b "Sr-Sr=2.075,Sr-B=1.671,B-B=1.267" -x 0.6 -p 40 -c 192
```

参数：

| 参数 | 含义 |
| --- | --- |
| `-b, --bond` | 逗号分隔的最小键长限制。 |
| `-x` | 所有最小键长的缩放因子。 |
| `-p, --press_in_gpa` | 写入 `.res` 元数据的压强，单位 GPa。 |
| `-c, --core` | 并行进程数。 |

`acnn_ckrelax` 从环境变量 `opt_dir` 读取需要检查的 relaxation 目录列表。每个目录应包含 `relax.vasp` 和 `relax.conv`。

输出：

| 情况 | 输出 |
| --- | --- |
| 最终结构安全 | `RELAX_DIR/RELAX_DIR-out.res` |
| 最终结构不安全但存在安全中间步 | `RELAX_DIR/RELAX_DIR-limitbonds.res`，并复制到 `LIM/` |

若系统中有 `cabal`，脚本会用它给生成的 `.res` 标注对称性。

## 相图工具

### `acnn_pdhtml`

读取当前目录所有 `.res` 文件并生成交互式相图 HTML。

```console
$ acnn_pdhtml
```

输出：

```text
pd.html
```

脚本使用 ASE 读取 `.res` 文件，并用 `pymatgen.analysis.phase_diagram` 构建相图。

### `acnn_pdb`

打印当前目录所有 `.res` 文件的相图稳定性信息。

```console
$ acnn_pdb
```

输出按 energy above hull 排序。稳定结构标记为 `+`，不稳定结构标记为 `-`。

### `acnn_shull`

打印当前目录所有 `.res` 文件的 hull 信息。

```console
$ acnn_shull
```

每行包含文件名、约化组分、元素种类数、energy above hull 和 formation energy per atom。

## 结构数据库工具

### `acnn_builddb`

把目录树中的 `.res` 文件导入 SQLite 数据库。

```console
$ acnn_builddb structures.db RSS/Base
```

用法：

```console
$ acnn_builddb <DB> <SRC>
```

数据库表为 `structures(id, filename, content, used)`。重复文件名会被忽略，也可以向已有数据库追加结构。

### `acnn_mergedb`

把一个结构数据库合并到另一个数据库。

```console
$ acnn_mergedb all.db new_batch.db
```

用法：

```console
$ acnn_mergedb <DST> <SRC>
```

源数据库以只读方式打开；目标数据库中已有的重复文件名会被忽略。

### `acnn_statdb`

打印结构数据库统计信息。

```console
$ acnn_statdb structures.db
```

输出 total、used 和 unused 的结构数量。

## 模型和模拟分析

### `acnn_emodel`

使用当前目录的 `in.acnn` 模板评估 ACNN 模型在数据集上的误差。

```console
$ acnn_emodel model-001 DT
```

用法：

```console
$ acnn_emodel <MODEL_PATH> <DT_PATH>
```

脚本修改 `in.acnn` 中的 `evalmodelpath` 和 `evaldatapath`，运行：

```console
$ acnn -eval in.acnn.tmp
```

然后用 `evalan.py` 分析输出。原始评估结果写入 `eval-<model_basename>-<dataset_basename>`。

### `acnn_estvol`

从 `.res` 结构中用最小二乘拟合估计每种元素的原子体积。

```console
$ cat *.res | acnn_estvol
```

脚本从标准输入读取 `.res`，调用 `cryan -r -l`，提取组分和体积，并求解元素体积的线性最小二乘问题。

### `acnn_lmp_thermodata`

解析 LAMMPS thermo block，并把每个 block 绘制成 PDF。

```console
$ acnn_lmp_thermodata log.lammps
```

输出文件：

```text
block-0.pdf
block-1.pdf
...
```

图中显示各 thermo 列随 `Step` 的变化，并带有采样均值和标准差标记。

## 结构操作

### `acnn_supercell`

从结构文件构建超胞，并把 POSCAR 文本写到标准输出。

```console
$ acnn_supercell POSCAR 2,2,1 > POSCAR.super
```

用法：

```console
$ acnn_supercell <file> <nx,ny,nz>
```

脚本使用 pymatgen 读取输入结构，按给定倍数扩胞，并输出 VASP POSCAR 格式。

### `acnn_symop`

打印空间群的对称操作。

```console
$ acnn_symop 166
$ acnn_symop R-3m
```

用法：

```console
$ acnn_symop <space_group_symbol_or_number>
```

输出包括空间群符号、国际编号、对称操作数量、xyz 表达式、旋转矩阵和平移向量。

## AIRSS 转换辅助工具

### `cabal`

`cabal` 是 AIRSS 提供的结构格式转换工具，常用于 `.res`、POSCAR、CIF 等格式之间的转换。

```console
$ cabal res poscar < input.res > POSCAR
$ cabal poscar res < POSCAR > output.res
```

### `ca`

`ca` 是 AIRSS 结构分析工具的封装，可用于检查 `.res` 文件、组分、对称性，以及准备相图输入。

### `symm`

查找结构的空间群：

```console
$ symm test
```

对于名为 `test.res` 的文件，`symm test` 会读取该结构并报告检测到的对称性。
