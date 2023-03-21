---
title: Social Interpretable Tree for Pedestrian Trajectory Prediction
date: 2023-03-15 09:46:23
tags:
---

### Introduction

由于行人移动的随机性，本文提出了名为SIT的方法。以树的形式推断行人未来可能的轨迹，然而树的复杂程度随着深度呈指数增长，因此又引入了CTT。该方法将高阶速度是做后续的移动方向，并增加了路径分割的时间。

### Related Work

手工制作的方法过度依赖先验信息，模型将仅限于特定场景，缺乏泛化能力。深度学习的发展推动了行人轨迹预测的研究，此类方法需要重复采样每个可能的未来轨迹。

### Our Method

![image-20230315213257300](C:\Users\512\AppData\Roaming\Typora\typora-user-images\image-20230315213257300.png)

#### Problem Formulation

基于已观测的轨迹$X$以及相互作用影响$S$预测未来的轨迹$Y$，选择最接近$\hat{Y}$预测结果

- $x^t_i$：时间步长$t$，行人$i$的坐标
- $\textbf{X} = \{X_i\}^N_{i=1},X_i=\{x^t_i\}^{T_{obs}}_{t=1}$：观察轨迹
- $\textbf{Y} = \{Y_j\}^K_{j=1}$：$K$个可接受的未来轨迹
- $\hat{Y} = \{y_i\}^N_{i=1},y_i = \{x^t_i\}^{T_{pred}}_{t=T_{obs}+1}$：真实轨迹（ground truth）
- $S = \{s_t\}^{T_{obs}}_{t=1}$：行人之间在每个时间步长$t$的相互作用影响
- $p(\hat{Y}|X,S)=\sum_{Y\in\textbf{Y}}{p(Y|X,S)p(|\hat{Y}|Y,X,S)}$

#### Trajectory Prediction with Tree

**Coarse Trajectory Tree**是为了平衡树的复杂性与覆盖范围。

**Trajectory Encoding** 一个MLP将$X$编码成$F_x$，另一个MLP将$Y_{coarse}$编码成$F_{tree}=\{f_i\}^M_{i=1}$，采用GCN交互编码成$F_s$

对$Y_{coarse}$中的数据进行评分并选择具有高置信度的对象优化

- $p = Softmax(\phi(F_S)\psi(F_{tree})^T)$
- $q$是最接近真实轨迹的标签
- $\mathcal{L}_{clf}=\mathcal{L}_{CE}(p,q)$，$\mathcal{L}_{CE}$为交叉熵损失

用MLP将$F_s$与$f*$融合得到$Y'$，依据$\mathcal{L}_{coarse}=\mathcal{L}_{reg}(Y',\hat{Y}_{coarse})$进行贪婪优化。

**Trajectory Refining**将$Y'$细化成 $Y_{fine}$，$\mathcal{L}_{ref}=\mathcal{L}_{reg}(Y_{coarse},Y)$

- $\mathcal{L}=\lambda_1\mathcal{L}_{coarse}+\lambda_2\mathcal{L}_{clf}+\lambda_3\mathcal{L}_{ref}$

### Experimental Analysis

用ADE（Average Displacement Error）与FDE（Final Displacement Error）来评估轨迹预测的性能。
