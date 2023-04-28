---
title: C++ Debug in UE5
math: true
date: 2023-03-30 17:02:39
categories:
- UE
tags:
---

### Debug
该文档在 *BeginPlay()* 中演示了如何调用 *UE_LOG* 将Debug信息输出到日志并编写了若干自定义宏来实现Actor部分信息的可视化。

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
<span class="inline-left">![DebugShpere](C++%20Debug%20in%20UE5/DebugShpere.png)</span><span class="inline-left">![BP_DebugShpere](C++%20Debug%20in%20UE5/BP_DebugShpere.png)</span>

我们也可以通过下面的C++代码实现这个功能

```c++
UWorld* pWorld = GetWorld();
FVector Location = GetActorLocation();
if (pWorld) {
    DrawDebugSphere(pWorld, Location, 25.f, 12, Fcolor::Red, true);
}
```

除了Sphere，还有Line和Point。它们的调用方式与Sphere类似，在C++中，可以用宏封装上述功能，以便多次调用。

```c++
#define _DRAW_DEBUG_ENABLE_ false
// Sphere
#define DRAW_SPHERE(Location) if(GetWorld()) DrawDebugSphere(GetWorld(), Location, 25.f, 12, FColor::Red, _DRAW_DEBUG_ENABLE_)
// Line
#define DRAW_LINE(StartLocation, EndLocation) if (GetWorld()) DrawDebugLine(GetWorld(), StartLocation, EndLocation, FColor::Red, _DRAW_DEBUG_ENABLE_)
// Point
#define DRAW_POINT(location) if (GetWorld()) DrawDebugPoint(GetWorld(), location, 5.f, FColor::Red, _DRAW_DEBUG_ENABLE_)

// call
FVector Location = GetActorLocation();
FVector Forward = GetActorForwardVector();
DRAW_SHPERE(Location);
DRAW_LINE(Location, Location + Forward * 50);
DRAW_POINT(Location + Forward * 50);
```

如果把Point当作Vector的结束点，那么通过DRAW_LINE和DRAW_POINT可以绘制出Vector，我们也可以直接定义一个宏DRAW_VECTOR(StartLocation, EndLocation)

```c++
// Implementing multi-line macro definitions using a backslash
#define DRAW_VECTOR(StartLocation, EndLocation) if (GetWorld()) { \
	DrawDebugLine(GetWorld(), StartLocation, EndLocation, FColor::Red, _DRAW_DEBUG_ENABLE_); \
	DrawDebugPoint(GetWorld(), EndLocation, 5.f, FColor::Red, _DRAW_DEBUG_ENABLE_); \
}
```

最后，我们可以将自定义的宏放入一个专用的头文件中，我们还可以不断增加或者修改这宏来满足不同的需求。
```c++
#pragma once

#define DRAW_SPHERE(Location) if(GetWorld()) DrawDebugSphere(GetWorld(), Location, 25.f, 12, FColor::Red, true)
#define DRAW_SPHERE_SINGLEFRAME(Location) if(GetWorld()) DrawDebugSphere(GetWorld(), Location, 25.f, 12, FColor::Red, false, -1.f)

#define DRAW_LINE(StartLocation, EndLocation) if (GetWorld()) DrawDebugLine(GetWorld(), StartLocation, EndLocation, FColor::Red, true)
#define DRAW_LINE_SINGLEFRAME(StartLocation, EndLocation) if (GetWorld()) DrawDebugLine(GetWorld(), StartLocation, EndLocation, FColor::Red, false, -1.f, 0, 0.f)

#define DRAW_POINT(Location) if (GetWorld()) DrawDebugPoint(GetWorld(), Location, 5.f, FColor::Red, true)
#define DRAW_POINT_SINGLEFRAME(Location) if (GetWorld()) DrawDebugPoint(GetWorld(), Location, 5.f, FColor::Red, false, -1.f, 0)

#define DRAW_VECTOR(StartLocation, EndLocation) if (GetWorld()) { \
	DrawDebugLine(GetWorld(), StartLocation, EndLocation, FColor::Red, true); \
	DrawDebugPoint(GetWorld(), EndLocation, 5.f, FColor::Red, true); \
}
#define DRAW_VECTOR_SINGLEFRAME(StartLocation, EndLocation) if (GetWorld()) { \
	DrawDebugLine(GetWorld(), StartLocation, EndLocation, FColor::Red, false, -1.f, 0, 0.f); \
	DrawDebugPoint(GetWorld(), EndLocation, 5.f, FColor::Red, false, -1.f, 0); \
}

#define DRAW_CIRCLE_XY(Location, Radius) if (GetWorld()) { \
	DrawDebugCircle(GetWorld(), Location, Radius, 48, FColor::Green, true, -1.f, 0, 0.f, FVector(1.f, 0.f, 0.f), FVector(0.f, 1.f, 0.f), false); \
}
#define DRAW_CIRCLE_XZ(Location, Radius) if (GetWorld()) { \
	DrawDebugCircle(GetWorld(), Location, Radius, 48, FColor::Green, true, -1.f, 0, 0.f, FVector(1.f, 0.f, 0.f), FVector(0.f, 0.f, 1.f), false); \
}
#define DRAW_CIRCLE_YZ(Location, Radius) if (GetWorld()) { \
	DrawDebugCircle(GetWorld(), Location, Radius, 48, FColor::Green, true, -1.f, 0, 0.f, FVector(0.f, 1.f, 0.f), FVector(0.f, 0.f, 1.f), false); \
}
```