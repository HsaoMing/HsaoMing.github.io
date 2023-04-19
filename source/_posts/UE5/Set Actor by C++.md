---
title: Set Actor by C++
math: true
date: 2023-03-31 21:24:58
categories:
- UE
tags:
---

### Actor

所有可以放入关卡的对象都是 Actor，比如 Camera, static mesh, player start location。Actor 支持三维变换，例如平移、旋转和缩放。你可以通过游戏逻辑代码（C++ 或 BluePrint）创建（生成）或销毁 Actor。在 C++ 中，AActor 是所有 Actor 的基类。
通过 *SetActorLocation()* 和 *SetActorRotation()* 可以分别设置 Actor的 Location 和 Rotation（Actor 不直接保存 Tranform）。

```c++
SetActorLocation(FVector(0.f, 0.f, 25.f));
SetActorRotation(FRotator(90.f, 0.f, 0.f));
```
这两个方法时直接修改 Actor 的 Location 和 Rotation，如果需要逐帧修改 Actor 的 Location 和 Rotation 以实现一个连续的运动，我们可以使用 *AddActorWorldOffset()* 和 *AddActorWorldRotation()* 来实现。
<!--more-->
```c++
void AMyActor::Tick(float DeltaTime) {
	Super::Tick(DeltaTime);
	FVector Location = GetActorLocation();
	FVector Forward = GetActorForwardVector();

	AddActorWorldOffset(FVector(1.f, 0.f, 0.f));
	DRAW_SPHERE_SINGLEFRAME(Location);

	AddActorWorldRotation(FRotator(1.f, 0.f, 0.f));
	DRAW_VECTOR_SINGLEFRAME(Location, Location + Forward * 50);
}
```

因为 *Tick()* 是依帧为单位调用的，如果希望对象的移动速度不受帧数的影响，可以用DetlaTime实现。

```c++
// Movement rate in units of cm/s
float MovementRate = 50.f;
float RotationRate = 45.f;

AddActorWorldOffset(FVector(MovementRate * DeltaTime, 0.f, 0.f));
AddActorWorldRotation(FRotator(MovementRate * DeltaTime, 0.f, 0.f));
```

### Reflection System
*UPROPERTY()* 能够将变量暴露给 BluePrint 以便调整，下面以正弦函数实现对象的周期运动为例，简单展示了如何将蓝图与 C++ 结合。

```c++
// Header file
protected:
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, Category = "Sine Parameters")
	float Amplitude = 0.25f;

	UPROPERTY(EditInstanceOnly, BlueprintReadWrite, Category = "Sine Parameters")
	float TimeConstant = 5.f;

private:
	UPROPERTY(VisibleInstanceOnly, BlueprintReadOnly, meta = (AllowPrivateAccess = "true"))
	float RunningTime;
	
// cpp
RunningTime += DeltaTime;
float DeltaZ = Amplitude * FMath::Sin(RunningTime * TimeConstant);
AddActorWorldOffset(FVector(0.f, 0.f, DeltaZ));
```

同样的，我们也可以将 C++ 中的函数公开给 BluePrint。不同的是，在 BluePrint 中，仅返回一个值的函数被称为BluePrint Pure Function（显示为绿色）。

```c++
// Header
protected:
	UFUNCTION(BlueprintPure)
	float TransformedSin();

// Cpp
float AMyActor::TransformedSin() {
	return Amplitude * FMath::Sin(RunningTime * TimeConstant);
}
```

### Components
每个 Actor 都至少有一个 Component，在 BluePrint 中会有 DefaultSceneRoot组件。

![BP_Componets](Set%20Actor%20by%20C++/BP_Componets.png)

在 C++ 中，它被称为 Root Component，类型为 USceneComponent，前面使用的*GetActorLocation()* 实际上就是返回 Root Component 的位置。

Static Mesh Component 是最常见的 Component，我们可以将 Component 附加到 Root Component上，能够保持两个 Component 之间的相对移动。

以下代码是用C++添加 Static Mesh Component 的示例。

```c++
//Header
class UStaticMeshComponent;

UPROPERTY(VisibleAnywhere)
UStaticMeshComponent* ItemMesh;
// Cpp
AMyActor::AMyActor() {
	UStaticMeshComponent* ItemMesh;
	ItemMesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("ItemMeshComponet"));
	RootComponent = ItemMesh;
}

```
