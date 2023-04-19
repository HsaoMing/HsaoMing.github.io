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

AMyActor::AMyActor() {
	PrimaryActorTick.bCanEverTick = true;

	SphereComponent = CreateDefaultSubobject<USphereComponent>(TEXT("SphereComponent"));
	SphereComponent->SetupAttachment(GetRootComponent());
	SphereComponent->SetSphereRadius(80.f, false);
}

void AMyActor::BeginPlay() {
	Super::BeginPlay();
	// Bind to recall
	SphereComponent->OnComponentBeginOverlap.AddDynamic(this, &AMyActor::OnSphereBeginOverlap);
	SphereComponent->OnComponentEndOverlap.AddDynamic(this, &AMyActor::OnSphereEndOverlap);
}

void AMyActor::OnSphereBeginOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult) {
	if (GEngine) {
		const FString ActorName = OtherActor->GetName();
		GEngine->AddOnScreenDebugMessage(1, 2.f, FColor::Cyan, ActorName);
	}
}

void AMyActor::OnSphereEndOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex) {
	if (GEngine) {
		const FString ActorName = OtherActor->GetName();
		GEngine->AddOnScreenDebugMessage(1, 2.f, FColor::Red, ActorName);
	}
}
```