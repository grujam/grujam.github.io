---
layout: single
title: "Unreal Team Project - Day 3"
categories: Unreal
classes: wide
---

# Day 3 : IDamageable Interface


## 2023.12.28 Thursday Scrum Log

기본적인 코드 작업 시작

- Skill은 기본적인 틀, Projectile, Area 등 작업 → IS, MH

- Enum과 Struct를 한군데에 모아 관리할 클래스 작성 → 작성자

- Gameplay에서 적용되는 대기 시간, 게임 시간 등을 GameMode를 통해 관리 및 클라이언트에게 Replicated 되는 과정 확인 → SW

- 모든 객체의 움직임을 담당하는 MovementComponent 작성 → HJ

- Helper 클래스 작성

## Daily 작업 과정

IDamageable Interface = 객체의 피해를 받을 수 있는 특징을 부여해주는 Interface.

```cpp
//현재 구성 요소
#pragma once

#include "CoreMinimal.h"
#include "UObject/Interface.h"
#include "Misc/Structures.h"
#include "Damageable.generated.h"

UINTERFACE(MinimalAPI)
class UDamageable : public UInterface
{
	GENERATED_BODY()
};

class PUSH_API IDamageable
{
	GENERATED_BODY()

public:
	UFUNCTION(NetMulticast, Reliable)
		virtual void Hit(const FHitData& InHitData) = 0;
};
```

Hit함수 재정의를 통해 다양한 객체에 대한 피격 내용을 구현할 수 있도록 작성.

NetMulticast와 Reliable을 통해 모든 클라이언트에 Replicated 되게 하였고, 데미지를 입는 과정은 신뢰성이 중요하므로 Reliable을 추가하였다.


**Structures와 Enums**

각 클래스에 선언하고 흩어져있는 구조체를 찾기보다 Structures에 선언 후 탐색 및 편집의 편리함을 위한 클래스.

현재 구성된 구조체: FHitData. 피격에 대한 정보를 보유하는 구조체

```cpp
//Structures.h
#pragma once

#include "CoreMinimal.h"
#include "Enums.h"
#include "Structures.generated.h"

USTRUCT()
struct FHitData
{
	GENERATED_BODY()

public:
	UPROPERTY(EditAnywhere, Category = "Damage")
		float Damage = 0.0f;

	UPROPERTY(EditAnywhere, Category = "Launch")
		float xLaunchPercent = 100.0f;

	UPROPERTY(EditAnywhere, Category = "Launch")
		float yLaunchPercent = 0.0f;

	UPROPERTY(EditAnywhere, Category = "Launch")
		float zLaunchPercent = 100.0f;

	UPROPERTY(EditAnywhere, Category = "Effect")
		class UFXSystemAsset* Effect;

	UPROPERTY(EditAnywhere, Category = "Effect")
		float EffectSize = 100.0f;

	UPROPERTY(EditAnywhere, Category = "Sound")
		class USoundWave* Sound;

	UPROPERTY(EditAnywhere, Category = "Debuff")
		TArray<EDebuff> Debuffs;

private:
	float xLaunch = 500.0f;
	float zLaunch = 500.0f;
	float yLaunch = 500.0f;
};

UCLASS()
class PUSH_API UStructures : public UObject
{
	GENERATED_BODY()

};
```

Enums도 동일한 이유로 열거체를 저장하는 클래스가

현재 작성된 열거체: EDebuff. 피격 후 적용되는 다양한 디버프에 대한 열거체.

```cpp
//Enums.h
#pragma once

#include "CoreMinimal.h"
#include "Enums.generated.h"

UENUM(BlueprintType)
enum class EDebuff : uint8
{
	ED_None, ED_Root, ED_Freeze, ED_Bleed
};

UCLASS()
class PUSH_API UEnums : public UObject
{
	GENERATED_BODY()

};

```
