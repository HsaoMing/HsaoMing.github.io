---
title: page
math: true
date: 2023-04-14 15:59:24
tags:
---

### Animation
我们可以使用 Animation BluePrint 实现 Character 的动作，在 AnimGraph ViewPort，将 Asset 链接到 Output Pose 以驱动角色动画。

![Animation](Animation/Animation.png)

为了实现较为复杂的动画组，可以采用 State Machine，在 AnimGraph ViewPort 为不同的动画状态添加转换的条件，使得 Character 的动画合理其逼真。与此同时，在 Event Graph ViewPort 中计算逻辑并获取 State Machine 需要的相关变量。

![Animation Control](Animation/Animation%20Control.png)

Animation BluePrint 有一个 C++ 父类 Anim Instance，那么可以通过 C++ 来实现上述逻辑，并让 Animation BluePrint 继承 Anim Instance。