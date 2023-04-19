---
title: page
math: true
date: 2023-04-14 15:59:24
tags:
---

### Animation
我们可以使用 Animation BluePrint 实现 Character 的动作，在 AnimGraph ViewPort，将 Asset 链接到 Output Pose 以驱动角色动画。

![Animation](Animation/Animation.png)

为了实现较为复杂的动画组，可以采用 State Machine，在 AnimGraph ViewPort 为不同的动画状态添加转换的条件，用多个 State 与多个状态转变的条件使得 Character 的动画合理其逼真。与此同时，在 Event Graph ViewPort 中计算逻辑并获取 State Machine 需要的相关变量。

![Animation Control](Animation/Animation%20Control.png)

Animation BluePrint 有一个 C++ 父类 Anim Instance，那么可以通过 C++ 来实现上述逻辑，并让 Animation BluePrint 继承 Anim Instance (在Class Settings中)。

```c++ 
// Header
virtual void NativeInitializeAnimation() override;
virtual void NativeUpdateAnimation(float DeltaTime) override;

UPROPERTY(BlueprintReadOnly)
AMyCharacter* MyCharacter;
UPROPERTY(BlueprintReadOnly, Category = Movement)
UCharacterMovementComponent* MyCharacterMovement;
UPROPERTY(BlueprintReadOnly, Category = Movement)
float GroundSpeed;

// Cpp
#include "Characters/MyCharacter.h"
#include "GameFramework/CharacterMovementComponent.h"
#include "Kismet/KismetMathLibrary.h"

void UMyAnimInstance::NativeInitializeAnimation() {
	Super::NativeInitializeAnimation();
	
	MyCharacter = Cast<AMyCharacter>(TryGetPawnOwner());
	if (MyCharacter) {
		MyCharacterMovement = MyCharacter->GetCharacterMovement();
	}
}

void UMyAnimInstance::NativeUpdateAnimation(float DeltaTime) {
	Super::NativeUpdateAnimation(DeltaTime);
	if (MyCharacterMovement) {
		GroundSpeed = UKismetMathLibrary::VSizeXY(MyCharacterMovement->Velocity);
	}
}
```

### Inverse Kinematics
Inverse Kinematics 通过解方程的方法来移动特定的骨骼。
- Sphere trace from feet to ground

![1](Animation/1.png)
- Interpolate smoothly toward offset targets

![2](Animation/2.png)
- Use the lowest foot offset to prevent overextension

![3](Animation/3.png)
- Add the interpolated offsets to the IK foot bones. This will move them up or down

![4](Animation/4.png)
- Use full body IK node to solve IK, and use the IK foot bones as the effector targets for each foot

![5](Animation/5.png)

通过上述步骤，我们实现了关于脚的 IK 动画，接下来需要在 AnimGraph ViewPort 控制动画输出的状态与逻辑。

![Animation Control with Control Rig](Animation/Animation%20Control%20with%20Control%20Rig.png)

### IK Rig
[Mixamo](https://www.mixamo.com/) 中有许多基于 X_Bot 骨骼的动画，我们可以通过 IK Rig 实现 Animation 迁移。让不同骨骼的 Character 拥有相同的 Animation。
- 下载 Animation 与 X_Bot
- 创建 IK Rig 选择 X_Bot Skeletal Mesh 并重定位 Skeleton

![IK Rig](Animation/IK%20Rig.png)

- 创建 IK Rig 选择目标 Skeletal Mesh 并重定位 Skeleton（与上图相同）
- 创建 IK Retargeter 并调整 Pose 使得 Animation 较为流畅
- 导出 Animation
