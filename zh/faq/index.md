---
title: "FAQ"
layout: single
classes: wide top-page
lang: zh
lang-ref: faq
---

## ACNN 可以和哪些外部程序一起使用？

ACNN 可以与 ARES、LAMMPS、AIRSS、CALYPSO 以及用于数据转换和工作流控制的脚本协同使用。

## 我应该从哪里开始？

建议先阅读[安装方法]({{ '/zh/installation/' | relative_url }})，如果要运行主动学习结构预测，可以继续阅读[晶体结构预测教程]({{ '/zh/tutorials/crystal-structure-prediction/' | relative_url }})。

## 文件格式在哪里说明？

技术参考中包含 [.res 格式]({{ '/zh/technical-reference/res-file/' | relative_url }}) 和 [.xsf 格式]({{ '/zh/technical-reference/xsf-file/' | relative_url }}) 的说明。

<div class="faq-list" markdown="1">

<details class="faq-item" markdown="1">
<summary>如何判断 ACNN 晶体结构预测搜索是否收敛？</summary>

**增加搜索量后，结果是否仍然稳定**

在 online 模式中，可以逐步增加 `RSS/comp.txt` 中每个组分的结构数，并继续主动学习迭代。如果扩大采样后，仍然反复找到已有的低焓结构，且新增候选经过 DFT 验证后没有进一步降低最低焓或改变凸包，说明搜索正在趋于收敛。

**将 `LIM` 作为辅助判断信号**

`RELAX/IT*/LIM` 中保存了触发键长或压力安全限制的结构。如果增加搜索量后，这类结构占实际搜索结构总数的比例仍然较低且稳定，可以作为搜索过程趋于稳定的辅助信号。但 `LIM` 较少并不能单独证明搜索收敛；比较不同迭代时，还应考虑 `bonds_scale` 等安全检查参数的变化。

</details>

</div>
