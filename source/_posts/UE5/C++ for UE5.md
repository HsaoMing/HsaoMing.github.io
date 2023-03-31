---
title: C++ for UE5
math: true
date: 2023-03-30 17:02:39
tags:
---
### Actor
Actor是可以任意放置的Object，在C++中，AActor是所有Actor的基类。
<!--more-->
```c++
// When overwriting a virtual inherited function from actor, it's a good idea to call super just incase actor implements anything at the beginning of the game in the actor begin play function

// Called when the game starts or when spawned
virtual void BeginPlay() override;

//Called every frame
virtual void Tick(float DeltaTime) override;
```

### Debug
```c++
void MyActor::BeginPlay {
    super::BeginPlay();

    UE_LOG(LogTemp, Warning, TEXT("Begin Play called!"));

    if (GEngine) {
		GEngine->AddOnScreenDebugMessage(1, 60.f, FColor::Cyan, FString("MyActor OnScreen Message!"));
	}
}
```

