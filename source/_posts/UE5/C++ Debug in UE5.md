---
title: C++ Debug in UE5
math: true
date: 2023-03-30 17:02:39
tags:
---
### Actor
Actor是可以任意放置的Object，在C++中，AActor是所有Actor的基类。

```c++
/*When overwriting a virtual inherited function from actor, it's a good idea to call 
super just incase actor implements anything at the beginning of the game in the actor
begin play function*/

// Called when the game starts or when spawned
virtual void BeginPlay() override;

//Called every frame
virtual void Tick(float DeltaTime) override;
```
<!--more-->

### Debug
```c++
void MyActor::BeginPlay {
    Super::BeginPlay();

    UE_LOG(LogTemp, Warning, TEXT("Begin Play called!"));

    if (GEngine) {
		GEngine->AddOnScreenDebugMessage(1, 60.f, FColor::Cyan, FString("MyActor OnScreen Message!"));
	}
}

void MyActor::Tick(float DeltaTime) {
    Super::Tick();

    UE_LOG(LogTemp, Warning, TEXT("DeltaTime: %f"), DeltaTime);

    if (GEngine) {
        FString Name = GetName();
        FString Message = FString::Printf(TEXT("Actor Name: %s"), *Name);
	    GEngine->AddOnScreenDebugMessage(1, 60.f, FColor::Cyan, Message);
	}
}
```

### Drawing Debug

创建一个DebugShpere，引擎会在指定位置绘制线框,使用UE5的BluePrint功能我们可以轻易的实现上述功能
<span class="inline-left">![DebugShpere](C++%20for%20UE5/DebugShpere.png)</span><span class="inline-left">![Alt text](C++%20for%20UE5/BP_DebugShpere.png)</span>

我们也可以通过下面的C++代码实现这个功能

```c++
UWorld* pworld = GetWorld();
FVector location = GetActorLocation();
if (pworld) {
    DrawDebugSphere(pworld, location, 25.f, 12, Fcolor::Red, true);
}
```

除了Sphere，还有Line和Point。它们的调用方式与Sphere类似，在C++中，可以用宏封装上述功能，以便多次调用。

```c++
#define _DRAW_DEBUG_ENABLE_ false
// Sphere
#define DRAW_SPHERE(location) if(GetWorld()) DrawDebugSphere(GetWorld(), location, 25.f, 12, FColor::Red, _DRAW_DEBUG_ENABLE_)
// Line
#define DRAW_LINE(startLocation, endLocation) if (GetWorld()) DrawDebugLine(GetWorld(), startLocation, endLocation, FColor::Red, _DRAW_DEBUG_ENABLE_)
// Point
#define DRAW_POINT(location) if (GetWorld()) DrawDebugPoint(GetWorld(), location, 5.f, FColor::Red, _DRAW_DEBUG_ENABLE_)

// call
FVector location = GetActorLocation();
FVector forward = GetActorForwardVector();
DRAW_SHPERE(location);
DRAW_LINE(location, location + forward * 50);
DRAW_POINT(location + forward * 50);
```

如果把Point当作Vector的结束点，那么通过DRAW_LINE和DRAW_POINT可以绘制出Vector，我们也可以直接定义一个宏DRAW_VECTOR(startLocation, endLocation)

```c++
// Implementing multi-line macro definitions using a backslash
#define DRAW_VECTOR(startLocation, endLocation) if (GetWorld()) { \
	DrawDebugLine(GetWorld(), startLocation, endLocation, FColor::Red, _DRAW_DEBUG_ENABLE_); \
	DrawDebugPoint(GetWorld(), endLocation, 5.f, FColor::Red, _DRAW_DEBUG_ENABLE_); \
}
```

