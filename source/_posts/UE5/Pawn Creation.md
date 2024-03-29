---
title: Pawn Creation
math: true
date: 2023-04-02 14:03:53
categories: 
- UE
tags:
---

### Pawn
Pawn 是可由玩家或 AI 控制的所有 Actor 的基类。Pawn 不仅决定了玩家或 AI 实体的外观效果，还决定了它们如何与场景进行碰撞以及其他物理交互。
<!--more-->

### Capsule Component
游戏中的 Mesh 都是由多边形网格组成的，因为直接计算多边形网格之间的碰撞是一项十分昂贵的操作，所以通常会使用比较基本的形状来进行碰撞检测。

### Skeletal Mesh Component
与 Static Mesh Component 不同，Skeletal Mesh Component 是一种可以存在动画的 Mesh，下面是用 C++ 在 BluePrint 中创建 Skeletal Mesh Component 并将其附加到 Root Component的示例。

```c++
// Header
// Forward declared
class USkeletalMeshComponent;
UPROPERTY(VisibleAnywhere)
USkeletalMeshComponent* BridMesh;
// Cpp
#include "Components/SkeletalMeshComponent.h"

APawn::Apawn() {
    BridMesh = CreateDefaultSubobject<USkeletalMeshComponent>(TEXT("BridMesh"));
    BridMesh->SetupAttachment(GetRootComponent());
}
```

### Controlling Pawns
Controller 是一种可以控制 Pawn（或 Pawn 的派生类，例如 Character），从而控制其动作的非实体 Actor。人类玩家使用 PlayerController 控制 Pawn，而 AIController 则对它们控制的 Pawn 实加人工智能效果。控制器用 Possess 函数控制 Pawn，用 Unpossess 函数放弃控制 Pawn。

```c++
AutoPossessPlayer = EAutoReceiveInput::Player0;
```

控制器会接收其控制的 Pawn 所发生诸多事件的通知。因此控制器可借机实现响应此事件的行为，拦截事件并接替 Pawn 的默认行为。可以让控制器在给定的 Pawn 之前运行，从而从而最大限度减少输入处理与 Pawn 移动之间的延迟。

### Movement Component
通过添加 Movement Component 能够实现对 Pawn 的控制

### Enhanced Input
Enhanced Input 能够满足更加复杂的是输入功能，在运行时重新映射控件。
#### Input Action
Input Action 是 Enhanced Input System 与项目代码之间的通信链接。
#### Input Mapping Contexts
Input Mapping Contexts 是输入动作的集合，表示玩家可以处于特定的上下文，描述了关于什么会触发给定输入动作的规则。

以下是BluePrint实现控制器链接 Input Action 的例子
![BP_EnhancedInput](Pawn%20Creation/BP_EnhancedInput.png)

如果想使用 C++ EnhancedInput，需要添加 EnhancedInput Module，在 XX.Build.cs 中修改
```c#
PublicDependencyModuleNames.AddRange(new string[] { "EnhancedInput" });
```
下面是通过 C++ 实现 Pawn 水平移动的示例，值得注意的是，想要实现 Controller 对 Pawn 的控制，需要添加 FloatingPawnMovement，并且在 BluePrint 中添加 C++ 公开的变量。

![Input](Pawn%20Creation/Input.png)
```c++
// Header
#include "InputActionValue.h"

void Move(const FInputActionValue& Value);
UPROPERTY(VisibleAnywhere)
UFloatingPawnMovement* PawnMovement;
UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Input)
UInputMappingContext* BridMappingContext;
UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Input)
UInputAction* MoveAction;

// Cpp
#include "EnhancedInputSubsystems.h"
#include "EnhancedInputComponent.h"
#include "GameFramework/FloatingPawnMovement.h"

AMyPawn::AMyPawn() {
    // Set controller
	AutoPossessPlayer = EAutoReceiveInput::Player0;
    // Enable the controller rotation
    bUseControllerRotationPitch = true;
	bUseControllerRotationYaw = true;
	PawnMovement = CreateDefaultSubobject<UFloatingPawnMovement>(TEXT("PawnMove"));
}

void AMyPawn::BeginPlay() {
	Super::BeginPlay(); 
	
	if (APlayerController* PlayerController = Cast<APlayerController>(GetController())) {
		if (UEnhancedInputLocalPlayerSubsystem* Subsystem = ULocalPlayer::GetSubsystem<UEnhancedInputLocalPlayerSubsystem>(PlayerController->GetLocalPlayer())) {
			Subsystem->AddMappingContext(BridMappingContext, 0);
		 }
	}
}

void AMyPawn::Move(const FInputActionValue& Value) {
	const FVector2D MovementVector = Value.Get<FVector2D>();
	if (Controller) {
		const FRotator Rotation = Controller->GetControlRotation();
		const FRotator YawRotation(0, Rotation.Yaw, 0);
		const FVector ForwardDirection = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::X);
		const FVector RightDirection = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::Y);

		AddMovementInput(ForwardDirection, MovementVector.Y);
		AddMovementInput(RightDirection, MovementVector.X);
	}
}

void AMyPawn::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent) {
	Super::SetupPlayerInputComponent(PlayerInputComponent);
	if (UEnhancedInputComponent* EnhancedInputComponent = CastChecked<UEnhancedInputComponent>(PlayerInputComponent)) {
		EnhancedInputComponent->BindAction(MoveAction, ETriggerEvent::Triggered, this, &AMyPawn::Move);
	}
}
```
我们可以在 console 输入 showdebug enhancedinput 来查看相关信息。

### Camera Component
当我们把 AutoPossessPlayer 绑定到 Pawn 上，虽然视野能够随者 Pawn 一起移动，但是视觉效果很差。这时，我们可以通过添加 Camera Component 来实现自定义相机的位置，Camera Component 通常连接到 Spring Arm 上。

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
AMyPawn::Apawn() {
    SpringArm = CreateDefaultSubobject<USpringArmComponent>(TEXT("SpringArm"));
    SpringArm->SetupAttachment(GetRootComponent());
    // 
    SpringArm->TargetArmLength = 1000.f;

    ViewCamera = CreateDefaultSubobject<UCameraComponent>(TEXT("ViewCamera"));
    // Bind Camera Component to Spring Arm
    ViewCamera->SetupAttachment(SpringArm);
}
```

