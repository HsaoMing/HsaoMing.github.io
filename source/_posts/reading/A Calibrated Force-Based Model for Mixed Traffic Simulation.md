---
title: A Calibrated Force-Based Model for Mixed Traffic Simulation
math: true
date: 2023-05-18 10:32:23
categroies:
- reading
tags:
---

### Introduction
高保真的交通模拟能够为虚拟城市、无人驾驶提供多样的交通情景。现有的交通仿真模型通常以反应的方式进行运动的决策而忽略了交通参与者之间的相互影响与真实互动。本文提出了一个基于力的框架、统一模拟交通参与者的行为与交互。
<!--more-->

### Related Work

#### Crowd Simulation Methods
基于智能体的人群模拟模型分为基于速度的与基于力的。

- velocity-based：智能体选择损失函数最小的速度
- force-based：智能体之间受到社会关系产生的虚拟力

#### Traffic Control Models
根据交通模拟的表达程度，可以将交通模拟分为基于连续体的宏观模型和基于主体的微观模型。

- continuum-based macroscopic models：用速度、流量、密度等方面的连续体来描述交通流，宏观的模拟不适合街道级的交通
- agent-based microscopic models：基于周围车辆的瞬时状态与道路信息控制车辆行为

#### Data-Driven Traffic Visualization and Simulation
随着传感器技术与计算机视觉的发展，能够从真实世界中获取车辆或行人的轨迹，借助元学习、强化学习等方法来合成车辆或行人的轨迹。

#### Traffic Model Calibration
高保真的交通仿真模型需要设置许多参数来达到与真实世界中的场景相似，通常将模型的参数校准过程作为一个优化问题来解决。

### Force-Based Framework
假设 $i$ 是交通混合流中的任一参与者，那么它受到的力可以由以下公式定义：
$$
\mathbf{F_i}(t) = \mathbf{F_i}^0(t) + \Sigma_{j(\not ={i})}\mathbf{F}_{ij}(t) + \Sigma_W\mathbf{F}_{iW}(t)+\Sigma_O\mathbf{F}_{iO}(t) + \xi_i
$$

$F_i(t)$ 是驱动力，意为 $i$ 以某一速度前往指定目的地的意图。
$$
\mathbf{F}_i(t) = m_i\frac{\mathbf{v_i}^0(t)-\mathbf{v_i}(t)}{\tau_i} = m_i\frac{\mathbf{v_i}^0(t)-\mathbf{v_i}(t)}{\mathbf{v_i}^0(t)}a_i
$$
其中，$a_i$ 是期望加速度，$v_i^0(t)$ 是期望速度，$m$ 是质量。

$$
\mathbf{F}_{iW}(t)=U_ie^\frac{|r_i^W+v_i^WT_i|}{R_i}mathbf{n}_i^W
$$
其中，$R_i$ 是 $i$ 对距离的敏感系数，$U_i$ 是环境力的比例因子，$r_i^W$ 是 $i$ 到 $W$（道路边界）的距离，$n_i^W$ 是 $W$ 距离 $i$ 最近的点指向 $i$ 的单位向量，$v_i^W$ 是方向$n_i^W$ 上的速度分量，$T_i$ 是反应时间。


### Force for Behaviors in Mixed Traffic

#### Force-Based Model for Vehicles
现有的微观交通模型对车辆的特定行为都是逐一建模和控制的，此外，这些方法通常侧重车辆前进方向的运动。本文提出了基于力方法，并对车辆的行驶做出如下约束：

- 驾驶员主要以跟车驾驶，运动状态会受到视野中所有车辆的影响
- 驾驶员必须保持在车道标线内行驶，并遵守交通规则
- 驾驶员在应对必要因素或是利用车道的允许速度会倾向于改变车道

##### Repulsive Forces Between Vehicles
根据交通中的车道保持规则，在不变道的情况下，相邻车道中的车辆 $q$ 对 $c$ 的作用力小于同车道中的车辆 $p$。同车道中的车辆 $p$ 对 $c$ 的作用力采用 IDM 的制动减速项来近似。
![Repulsive Force](A%20Calibrated%20Force-Based%20Model%20for%20Mixed%20Traffic%20Simulation/Repulsive%20Forces.png)

- Influence from vehicles in adjacent lanes:
$$
\mathbf{F}_{cq}^n(t)=U_ce^-\frac{r_{cq}}{R_c}\mathbf{n}_{cq}
$$

- Influence from the vehicle in the current lane:
$$
\mathbf{F}_{cp}^f(t)=-b_c(\frac{s^*}{s})^2\mathbf{n}_c
$$
$$
s^*=s_c^0+v_cT_c+\frac{v_c\Delta{v}}{2\sqrt{{a_cb_c}}}
$$

其中，$a_c$ 是 $c$ 的最大加速度，$b_c$ 是 $c$ 的舒适减速度，$s_c^0$ 是阻塞间隔，$T_c$ 是期望安全行车时间，$\mathbf{n}_c$ 是 $c$ 行驶方向的单位向量。

### Force For Lane Changing
