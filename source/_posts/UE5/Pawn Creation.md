---
title: Pawn Creation
math: true
date: 2023-04-02 14:03:53
categories: 
- UE
tags:
---

### Pawn
Pawn是可由玩家或AI控制的所有Actor的基类。Pawn不仅决定了玩家或AI实体的外观效果，还决定了它们如何与场景进行碰撞以及其他物理交互。
<!--more-->
### Capsule Component
游戏中的Mesh都是由多边形网格组成的，因为直接计算多边形网格之间的碰撞是一项十分昂贵的操作，所以通常会使用比较基本的形状来进行碰撞检测。

### Skeletal Mesh Component
与Static Mesh Component不同，Skeletal Mesh Component是一种可以存在动画的Mesh，下面是用C++在BluePrint中创建Skeletal Mesh Component并将其附加到Root Component的示例。

```c++
// Header
// Forward declared
class USkeletalMeshComponent;
UPROPERTY(VisibleAnywhere)
USkeletalMeshComponent* BridMesh;
// Cpp
#include "Components/SkeletalMeshComponent.h"

BridMesh = CreateDefaultSubobject<USkeletalMeshComponent>(TEXT("BridMesh"));
BridMesh->SetupAttachment(GetRootComponent());
```

### Controlling Pawns
Controller是一种可以控制Pawn（或Pawn的派生类，例如Character），从而控制其动作的非实体Actor。人类玩家使用PlayerController控制Pawn，而AIController则对它们控制的Pawn实加人工智能效果。控制器用Possess函数控制Pawn，用Unpossess函数放弃控制Pawn。

```c++
AutoPossessPlayer = EAutoReceiveInput::Player0;
```

控制器会接收其控制的Pawn所发生诸多事件的通知。因此控制器可借机实现响应此事件的行为，拦截事件并接替Pawn的默认行为。可以让控制器在给定的Pawn之前运行，从而从而最大限度减少输入处理与Pawn移动之间的延迟。

### Movement Component
通过添加Movement Component能够实现对Pawn的控制

### Enhanced Input


### Camera Component
当我们把AutoPossessPlayer绑定到Pawn上，虽然视野能够随者Pawn一起移动，但是视觉效果很差。这时，我们可以通过添加Camera Component来实现自定义相机的位置，Camera Component通常连接到Spring Arm上。

![Camera Component](Pawn%20Creation/Camera%20Component.png)

```c++
// Header
class USpringArmComponent;
class UCameraComponent;

UPROPERTY(VisibleAnywhere)
USpringArmComponent* SpringArm;
UPROPERTY(VisibleAnywhere)
UCameraComponent* ViewCamera;

// Cpp
SpringArm = CreateDefaultSubobject<USpringArmComponent>(TEXT("SpringArm"));
SpringArm->SetupAttachment(GetRootComponent());
// 
SpringArm->TargetArmLength = 1000.f;

ViewCamera = CreateDefaultSubobject<UCameraComponent>(TEXT("ViewCamera"));
// Bind Camera Component to Spring Arm
ViewCamera->SetupAttachment(SpringArm);
```

