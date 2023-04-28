---
title: Collision
math: true
date: 2023-04-16 16:44:27
categories: 
- UE
tags:
---

### Collision
Static Mesh 是 Actor 派生来的，其默认拥有 Static Mesh Component，修改 Collision Presets 组件满足不同的碰撞需求 
Collision Presets->Collision Enabled

- No Collision
- Query Only(No Physics Collision)
- Physics Only
- Collision Enabled

Collision Presets->Collision Responses
修改 Collision Responses 可以忽略对某一类对象的碰撞检测达到预期的碰撞效果。

<!--more-->
### Overlap
如果我们想让 Character 去获取场景中的道具，首先需要判断道具是否在 Character 的拾取范围中。我们可以对 Static Mesh 添加 SphereComponent，并检测 Character 与 SphereComponent 的 overlap 事件，将自定义函数绑定到回调函数上。

```c++
// Header
class USphereComponent
protected:
    // Queried in PrimitiveComponent.h
	UFUNCTION()
	void OnSphereBeginOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult);
	UFUNCTION()
	void OnSphereEndOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex);

private:
	UPROPERTY(VisibleAnywhere)
	UStaticMeshComponent* ItemMesh;
	UPROPERTY(VisibleAnywhere)
	USphereComponent* SphereComponent;

// Cpp

AItem::AItem() {
	PrimaryActorTick.bCanEverTick = true;

	SphereComponent = CreateDefaultSubobject<USphereComponent>(TEXT("SphereComponent"));
	SphereComponent->SetupAttachment(GetRootComponent());
	SphereComponent->SetSphereRadius(80.f, false);
}

void AItem::BeginPlay() {
	Super::BeginPlay();
	// Bind to recall
	SphereComponent->OnComponentBeginOverlap.AddDynamic(this, &AItem::OnSphereBeginOverlap);
	SphereComponent->OnComponentEndOverlap.AddDynamic(this, &AItem::OnSphereEndOverlap);
}

void AItem::OnSphereBeginOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult) {
	if (GEngine) {
		const FString ActorName = OtherActor->GetName();
		GEngine->AddOnScreenDebugMessage(1, 2.f, FColor::Cyan, ActorName);
	}
}

void AItem::OnSphereEndOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex) {

}
```

### Trace
在发生 Overlap 事件时，我们希望能够得到一个较为准确的向量来描述发生碰撞的点。我们能够对添加 Trace，当 Trace 命中物体时会返回一个结果。在 Collision Presets 需要设置对应的碰撞属性。
通过 Character 持有一把武器进行攻击作为例子，检测出武器攻击到的准确位置。

```c++
UPROPERTY(VisibleAnywhere, Category = "Weapon Properties")
UBoxComponent* WeaponBox;

// Add Secen for Trace
UPROPERTY(VisibleAnywhere)
USceneComponent* BoxTraceStart;
UPROPERTY(VisibleAnywhere)
USceneComponent* BoxTraceEnd;

AWeapon::AWeapon() {
	WeaponBox = CreateDefaultSubobject<UBoxComponent>(TEXT("Weapon Box"));
	WeaponBox->SetupAttachment(GetRootComponent());

	// Set Collision Presets
	WeaponBox->SetCollisionEnabled(ECollisionEnabled::NoCollision);
	WeaponBox->SetCollisionResponseToAllChannels(ECollisionResponse::ECR_Overlap);
	WeaponBox->SetCollisionResponseToChannel(ECollisionChannel::ECC_Pawn, ECollisionResponse::ECR_Ignore);

	BoxTraceStart = CreateDefaultSubobject<USceneComponent>(TEXT("Box Trace Start"));
	BoxTraceStart->SetupAttachment(GetRootComponent());
	BoxTraceEnd = CreateDefaultSubobject<USceneComponent>(TEXT("Box Trace End"));
	BoxTraceEnd->SetupAttachment(GetRootComponent());
}
```

 在 UBoxComponent 的 Overlap 事件的回调函数中来实现 Trace。为了 *UKismetSystemLibrary::BoxTraceSingle(）* 能够检测到碰撞的准确位置需要将 Collision Responses 下的 Visibility 选项设置为 Block。

```c++
// Set Visibility
GetMesh()->SetCollisionResponseToChannel(ECollisionChannel::ECC_Visibility, ECollisionResponse::ECR_Block);
```

```c++
void AWeapon::OnBoxOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult) {
	// Global location
	const FVector Start = BoxTraceStart->GetComponentLocation();
	const FVector End = BoxTraceEnd->GetComponentLocation();

	TArray<AActor*> ActorsToIgnore;
	ActorsToIgnore.Add(this);
	FHitResult BoxHit;

	UKismetSystemLibrary::BoxTraceSingle(
		this,
		Start,
		End,
		FVector(5.f, 5.f, 5.f),
		BoxTraceStart->GetComponentRotation(),
		ETraceTypeQuery::TraceTypeQuery1,
		false,
		ActorsToIgnore,
		EDrawDebugTrace::ForDuration,
		BoxHit,
		true
	);
}
 ```

 如果我们只想在需要的时候启用 Overlap 事件，首先可以在 Animation 中添加自定义 Notify，在 C++ 中设计相关的函数并公开给 BluePrint。在 Animation BluePrint 的 Event Graph 中调用公开的 C++ 函数。[Notify Event](https://hsaoming.github.io/2023/04/11/UE5/Character%20Class/#character-action-in-item)。

 ```c++
void AMyCharacter::SetWeaponCollisionEnabled(ECollisionEnabled::Type CollisionEnabled) {
	if (EquippedWeapon && EquippedWeapon->GetWeaponBox()) {
		EquippedWeapon->GetWeaponBox()->SetCollisionEnabled(CollisionEnabled);
	}
}
 ```