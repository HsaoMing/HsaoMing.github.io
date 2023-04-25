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


### Pick Up Item
为了实现 Character 与 Item 之间的交互，我们可以通过 *OnComponentBeginOverlap()* 实现 Overlap 事件的回调([example](https://hsaoming.github.io/2023/04/16/UE5/Collision/#overlap))。

修改 *OnSphereBeginOverlap()* 能够实现许多有趣的功能。此外，Character 还需要具有选择是否拾取 Item 的能力。

- 首先需要进行操作映射，参照 [EnhancedInput](https://hsaoming.github.io/2023/04/02/UE5/Pawn%20Creation/#enhanced-input)。
- 在 Character 的类中，创建 Item 的类来保存 Item

```c++
// Header

// Item class name
class AItem;

protected:
	// Execute when E key down
	void EKeyPressed();

private:
	UPROPERTY(VisibleInstanceOnly)
	AItem* OverlappingItem;

public:
    // Set value in overlap event
	FORCEINLINE void SetOverlappingItem(AItem* Item) { OverlappingItem = Item; }
```

- 在 *OnSphereBeginOverlap()* 与 *OnSphereEndOverlap()* 中，对 Item 赋值

```c++
void AItem::OnSphereBeginOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult) {
	if (AMyCharacter* MyCharacter = Cast<AMyCharacter>(OtherActor)) {
		MyCharacter->SetOverlappingItem(this);
	}
}

void AItem::OnSphereEndOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex) {
	if (AMyCharacter* MyCharacter = Cast<AMyCharacter>(OtherActor)) {
		MyCharacter->SetOverlappingItem(nullptr);
	}
}


```
- 实现按下 E 拾取 Item 的功能（即实现 *EKeyPressed()*）


```c++
void AMyCharacter::EKeyPressed() {
	if (AWeapon* OverlappingWeapon = Cast<AWeapon>(OverlappingItem)) {
		OverlappingWeapon->Equip(GetMesh(), FName("right_hand_socket"));
	}
}
```
AWeapon 继承 Item，直接调用已经实现的函数 *Equip()*，将 Item 关联到已创建的 Socket 上，以下是 *Equip()* 的实现。

```c++
void AWeapon::Equip(USceneComponent* InParent, FName InSocketName) {
	FAttachmentTransformRules TransformRules(EAttachmentRule::SnapToTarget, true);
	ItemMesh->AttachToComponent(InParent, TransformRules, InSocketName);
}
```

### Character State
Character 拥有不同的状态，Character 需要依据不同的状态做出相应的 Animation。这时我们需要一个 enum 类并将其作为一个独立的头文件，以下是 Character 装备武器状态的示例。

```c++
UENUM(BlueprintType)
enum class ECharacterState : uint8 {
	// ECS meams CharacterState
	ECS_Unequipped UMETA(DisplayName = "Unequipped"),
	ECS_EquippedOneHandedWeapon UMETA(DisplayName = "Equipped One-Hand Weapon"),
	ECS_EquippedTwoHandedWeapon UMETA(DisplayName = "Equipped Two-Hand Weapon")
};
```
为 Character 添加状态后，在 AnimGraph ViewPort 中可以根据不同的状态输出对应的 Pose。Blend Time 为 Animation 过度的时间。

![Animation State](Character%20Class/Animation%20State.png)

Character 在攻击时是禁止移动的，在对 Character 操作之前应该先对其状态进行判断
```c++
void AMyCharacter::Move(const FInputActionValue& Value) {
	if (ActionState == EActionState::EAS_Attacking) return;
	/*
	
	*/
}
```

### Character Action In Item
Character 对特定的 Item 可以执行多种不同的操作。例如武器，Character 可以将在场景拾取的武器背在背上，也可以将背上的武器取下并装备。

- 为特定的骨骼添加 Socket， 在执行将武器背起的操作时，将 Static Mesh 关联到 Socket。
- 我们需要在制作的 Montage 动画中添加合适的 Notify，设计装备武器与卸下武器的 C++ 函数并公开给蓝图。

![Notify](Character%20Class/Notify.png)
```c++
// Header
UFUNCTION(BlueprintCallable)
void Disarm();
UFUNCTION(BlueprintCallable)
void Arm();

// Cpp
void AMyCharacter::Disarm() {
	if (EquippedWeapon) {
		EquippedWeapon->AttachMeshToAocket(GetMesh(), FName("spine_socket"));
	}
}

void AMyCharacter::Arm() {
	if (EquippedWeapon) {
		EquippedWeapon->AttachMeshToAocket(GetMesh(), FName("right_hand_socket"));
	}
}
```
 我们还可以为这些 Animation 绑定输入，并且想用拾取 Item 相同的输入。那么需要在播放 Animation 之前需要判断当前的状态是否能够播放 Animation。

 ```c++
 void AMyCharacter::EKeyPressed() {
	AWeapon* OverlappingWeapon = Cast<AWeapon>(OverlappingItem);
	
	if (OverlappingWeapon) {
		OverlappingWeapon->Equip(GetMesh(), FName("right_hand_socket"));
		CharacterState = ECharacterState::ECS_EquippedOneHandedWeapon;
		EquippedWeapon = OverlappingWeapon;

		SetOverlappingItem(nullptr);
	} else {

		if (CanDisarm()) {
			PlayEquipMontage(FName("Unequip"));
			CharacterState = ECharacterState::ECS_Unequipped;
			ActionState = EActionState::EAS_EquippingWeapon;

		} else if (CanArm()) {
			PlayEquipMontage(FName("Equip"));
			CharacterState = ECharacterState::ECS_EquippedOneHandedWeapon;
			ActionState = EActionState::EAS_EquippingWeapon;
		}
	}
}

bool AMyCharacter::CanDisarm() {
	return ActionState == EActionState::EAS_Unoccupied && 
		CharacterState != ECharacterState::ECS_Unequipped && 
		EquipMontage;
}

// When Character pick up Weapon, EquippedWeapon will be assigned
bool AMyCharacter::CanArm() {
	return ActionState == EActionState::EAS_Unoccupied &&
		CharacterState == ECharacterState::ECS_Unequipped &&
		EquippedWeapon && EquipMontage;
}

 ```

 上述有关 Montage 的函数可以参考该链接 [How to play Montage](https://hsaoming.github.io/2023/04/14/UE5/Animation/#animation-montages)。