---
title: Open World
math: true
date: 2023-03-29 09:46:53
categories:
- UE
tags:
---
一个大世界通常被分成多个小地图，当角色穿过某一小地图边界时，引擎将加载另一张地图，然而这些操作需要进行大量的管理工作。UE5引入了一个新的工作流程以更好的实现大世界，UE5的开放世界系统允许我们拥有一张大地图，引擎将大世界分为几个区域，只加载角色周围的区域。
<!--more-->
### Lighting and Atmosphere
- Sky Atmosphere：类似与大气层的散射光
- Directional Light：模拟无限远的光源（太阳光）
  - CTRL+L：可以调节第一个太阳的位置，当存在两个Directional Light时，第一个太阳会出现gimbal lock的现象（Bug）
  - CTRL+SHIFT+L：可以调节第二个太阳的位置
  - Mobility决定了灯光在游戏中的行为方式
     - Static：灯光的照明无法在游戏中更改，计算速度最快
     - Stationary：可以改变灯光的颜色和强度，但是不能改变灯光位置
     - Movable：灯光可以随意更改
  - Temperature：控制太阳的温度，在视觉效果上，温度越高越蓝、温度越低越红
- Sky Light：捕捉场景中的照明信息并用于场景
  - Real Time Capture：能够控制Sky Light是否实时捕获
- Exponential Height Fog：根据离地面的高度模拟雾
- Volumetric Clouds：材料驱动的三维的动态云
![Sky](Open%20World/Sky.png)

### Material
Material通常由base color、normal、texture构成。Quixel Bridge提供了大量具有真实感的Material。
如果想让Landscape拥有多种Material的混合
- 首先，在Material Editer中创建Landscape Layer Blend，查看Details Panel，Layer Name属性不能含有空格![Layer Blend Details](Open%20World/Layer%20Blend%20Details.png)
- 其次，将我们需要的base color拖入Material Editer
- 同样的，我们需要制作混合的normal与texture

### Landscape
Landscape是一个可塑造网格
- Number of Components：可以调节Landscape的大小
- Sculpt：按住鼠标左键并拖动可以对landscape雕。拥有多种雕刻模式
- Paint：当Lanscape拥有混合的Material时，可以使用不同的Material对Landscape作渲染

### Foliage
类似于Landscape上的Material Layers，从Quixel Bridge添加静态网格资产为Landscape添加细节。