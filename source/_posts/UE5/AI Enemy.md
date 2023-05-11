---
title: AI Enemy
math: true
date: 2023-04-26 18:56:49
tags:
---

### Collision Setting
[Character](https://hsaoming.github.io/2023/04/11/UE5/Character%20Class/#more) 已经具备攻击的能力，那么我们需要设计对应的敌人，同样的，敌人也 作为 Character 类存在。为了响应 Character 的攻击事件，首先需要进行碰撞设置。

```c++
AEnemy::AEnemy() {
	PrimaryActorTick.bCanEverTick = true;
	GetMesh()->SetCollisionObjectType(ECollisionChannel::ECC_WorldDynamic);
	GetMesh()->SetCollisionResponseToChannel(ECollisionChannel::ECC_Visibility, ECollisionResponse::ECR_Block);
	GetMesh()->SetCollisionResponseToChannel(ECollisionChannel::ECC_Camera, ECollisionResponse::ECR_Ignore);
	GetMesh()->SetGenerateOverlapEvents(true);
	GetCapsuleComponent()->SetCollisionResponseToChannel(ECollisionChannel::ECC_Camera, ECollisionResponse::ECR_Ignore);
}
```

### Interfaces
当 Enemy 被击中时会有相应的动作，然而不同的 Enemy 的回应可能不同，这时需要引入 Interface 的概念。
- 创建 HitInterface 类与纯虚函数 *GetHit()*

```c++
#include "CoreMinimal.h"
#include "UObject/Interface.h"
#include "HitInterface.generated.h"

UINTERFACE(MinimalAPI)
class UHitInterface : public UInterface {
	GENERATED_BODY()
};
class LEARNING_API IHitInterface {
	GENERATED_BODY()
public:
	virtual void GetHit(const FVector& ImpactPoint) = 0;
};

```

- Enemy 继承 IHitInterface
```c++
UCLASS()
class LEARNING_API AEnemy : public ACharacter, public IHitInterface {
	GENERATED_BODY()
};
```

### Hit Only Once
当 Character 挥动武器攻击 Enemy 时，有时会击中 Enemy 多次。为了解决这一问题，我们需要从 *UKismetSystemLibrary::BoxTraceSingle()* 入手。

- 添加一个 ``TArray<AActer*>`` 公共变量用于存放 Character 攻击时需要忽略的对象，在触发一次 *GetHit()* 后，将 Enemy 放入公共变量中。再次触发 Overlap 事件时，不检测 Enemy 与武器的碰撞。

```c++
public:
    TArray<AActor*> IgnoreActors;

void AWeapon::OnBoxOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult) {
	// Global location
	const FVector Start = BoxTraceStart->GetComponentLocation();
	const FVector End = BoxTraceEnd->GetComponentLocation();

	TArray<AActor*> ActorsToIgnore;
	ActorsToIgnore.AddUnique(this);
	for (AActor* AActor : IgnoreActors) {
		ActorsToIgnore.AddUnique(AActor);
	}

	FHitResult BoxHit;
    /**
    * Call UKismetSystemLibrary::BoxTraceSingle()
    */

	if (BoxHit.GetActor()) {
		IHitInterface* HitInterface = Cast<IHitInterface>(BoxHit.GetActor());
		if (HitInterface) {
			HitInterface->GetHit(BoxHit.ImpactPoint);
		}
		IgnoreActors.AddUnique(BoxHit.GetActor());
	}
}
```

- 在关闭 BoxComponent 的 Overlap 事件时，清空``TArray<AActer*>`` 公共变量。

```c++
void AMyCharacter::SetWeaponCollisionEnabled(ECollisionEnabled::Type CollisionEnabled) {
	if (EquippedWeapon && EquippedWeapon->GetWeaponBox()) {
		EquippedWeapon->IgnoreActors.Empty();
	}
}
```

### Sounds And Particles
在创建 [Meta Sounds](https://hsaoming.github.io/2023/04/14/UE5/Animation/#Meta-Sounds) 后，我们能够使用 C++ 控制其播放的时间点。

```c++
UPROPERTY(EditDefaultsOnly, Category = Sounds)
USoundBase* HitSound;

void AEnemy::GetHit(const FVector& ImpactPoint) {
	DRAW_SPHERE_COLOR(ImpactPoint, FColor::Cyan);
	DirectionalHitReact(ImpactPoint);
	if (HitSound) {
		UGameplayStatics::PlaySoundAtLocation(
			this,
			HitSound,
			ImpactPoint
		);
	}
}
```
*UGameplayStatics::PlaySoundAtLocation()* 播放的声音会根据距离衰减，前提是需要为 Meta Sound 添加 Sound Attenuation。

添加粒子效果与声音的方式相同，调用 *UGameplayStatics::SpawnEmitterAtLocation()* 。

### Fracture
Character 不经能够攻击敌人，还能攻击场景中可以破坏的物品，例如陶罐被武器打碎并发出清脆的响声。
- 为 Static Mesh 添加 Geometry Collection, 在 Fracture Mode 中编辑。

- 对 Geometry Collection 施加力能够将其击碎，C++ 的函数可以被声明为事件，我们一个在 C++ 中调用该函数，在 BluePrint 中实现逻辑。以下是在 BluePrint 中使用 FieldSystem 的例子。

![FieldSystem](AI%20Enemy/FieldSystem.png)

创建一个 C++ 类作为这些易碎品的基类，该 C++ 类需要一个 Geometry Collection Component，用于存放 Geometry Collection。

```c++
#include "GeometryCollection/GeometryCollectionComponent.h"

ABreakableActor::ABreakableActor() {
	PrimaryActorTick.bCanEverTick = false;

	GeometryCollection = CreateDefaultSubobject<UGeometryCollectionComponent>(TEXT("GeometryCollection"));
	SetRootComponent(GeometryCollection);

	GeometryCollection->SetGenerateOverlapEvents(true);
	GeometryCollection->SetCollisionResponseToChannel(ECollisionChannel::ECC_Camera, ECollisionResponse::ECR_Ignore);
}
```
当物品被击碎后，我们要设置它的生命周期以减少资源的占用，以下 C++ 代码演示了如何添加 *OnChaosBreakEvent*。值得注意的是，需要在 X.Build.cs 文件中添加 GeometryCollectionEngine 与 ChaosSolverEngine 模块。

```c++
#include "GeometryCollection/GeometryCollectionComponent.h"
#include "Chaos/ChaosGameplayEventDispatcher.h"

void ABreakableActor::BeginPlay() {
	Super::BeginPlay();
	GeometryCollection->OnChaosBreakEvent.AddDynamic(this, &ABreakableActor::OnGeometryCollectionBreak);
}

void ABreakableActor::OnGeometryCollectionBreak(const FChaosBreakEvent& BreakEvent) {
	SetLifeSpan(3.f);
}

void ABreakableActor::GetHit_Implementation(const FVector& ImpactPoint) {
	if (bBroken) return;
	bBroken = true;
}

```