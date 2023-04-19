---
title: Character Class
math: true
date: 2023-04-11 13:31:09
categories: 
- UE
tags:
---
class Character 继承 class Pawn, 我已经在 [Pawn Creation](https://hsaoming.github.io/2023/04/02/UE5/Pawn%20Creation/) 中已经实现对 Pawn 简单操作。

<!--more-->
### Character Class
将 Character 的 UseControllerRotationPitch、UseControllerRotationYaw、UseControllerRotationRoll 属性设置为true，那么 Character 就会随着视野一起转动。

``` c++
AMyCharacter::AMyCharacter() {
	bUseControllerRotationPitch = true;
    bUseControllerRotationYaw = true;
	bUseControllerRotationRoll = true;
}
```

### Camera And Spring Arm
为了实现视角的自由转动且保持 Character 相对静止，需要通过对Spring Arm 的 UsePawnControlRotation 属性进行修改。 此时的 Character 只是简单的平移，需要通过修改 Character Movement 的 OrientRotationToMovement 为 true，同时 RotationRate 属性可以修改 Character 转向的速度。值得注意的是，要将 Controller 的相关 Rotation 属性改为 false， 否则 Character 的 Forward Vector 会产生冲突。

``` c++
AMyCharacter::AMyCharacter() {
	bUseControllerRotationPitch = false;
	bUseControllerRotationRoll = false;
	bUseControllerRotationYaw = false;

	GetCharacterMovement()->bOrientRotationToMovement = true;
	GetCharacterMovement()->RotationRate = FRotator(0.f, 400.f, 0.f);

	SpringArm = CreateDefaultSubobject<USpringArmComponent>(TEXT("SpringArm"));
	SpringArm->SetupAttachment(GetRootComponent());
	SpringArm->TargetArmLength = 300.f;
	SpringArm->bUsePawnControlRotation = true;

	ViewCamera = CreateDefaultSubobject<UCameraComponent>(TEXT("ViewCamera"));
	ViewCamera->SetupAttachment(SpringArm);
	ViewCamera->bUsePawnControlRotation = false;
}
```

### GroomComponent
GroomComponent 允许 Character 拥有头发或眉毛，能够使 Character 更加逼真。GroomComponent 是特定的 C++ Component，需要在 XX.Build.cs 中添加 HairStrandsCore Module 和 Niagara Module 并重新构建项目，除此之外，还需要在 Unreal Editer 中启用插件。

```c++
class UGroomComponent;

UPROPERTY(VisibleAnywhere, Category = Hair)
UGroomComponent* Hair;
UPROPERTY(VisibleAnywhere, Category = Hair)
UGroomComponent* Eyebrows;

#include "GroomComponent.h"
AMyCharacter::AMyCharacter() {
	Hair = CreateDefaultSubobject<UGroomComponent>(TEXT("Hair"));
	Hair->SetupAttachment(GetMesh());
	Hair->AttachmentName = FString("head");

	Eyebrows = CreateDefaultSubobject<UGroomComponent>(TEXT("Eyebrows"));
	Eyebrows->SetupAttachment(GetMesh());
	Eyebrows->AttachmentName = FString("head");
}
```
我们还可以根据自己的喜好修改 GroomComponent 的 Materials。

### Socket
我们的 Character 拥有与环境交互的能力，比如捡起场景中的武器，我们能够通过 Socket 来实现。编辑 Character 的 Skeletal Mesh Asset，可以在任意骨骼中添加 Socket，Socket可以添加 Static Mesh Asset 并且会随着关联的骨骼一起变换。