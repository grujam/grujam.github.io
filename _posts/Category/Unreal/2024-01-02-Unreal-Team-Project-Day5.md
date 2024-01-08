---
layout: single
title: "Unreal Team Project - Day 5"
categories: Unreal
classes: wide
---

# Day 5 : Ring

## 2024.01.02 Tuesday Scrum Log

- MH → 가족여행으로 인한 휴가

- IS → FProjectile의 데미지는 FHitData에서 적용할 것이므로 제거

- SW → UI 카운트 다운, PushGameMode & PushGameState 리팩토링, Widget 체력 추가

- HJ → ItemClass 틀 작성

- 작성자 → Ring의 피격 시 효과와 Hit Interface 세부 구현

## Daily 작업 과정

**Ring 피격 시 피격 효과**

피격 시 효과를 보여주는 Widget을 BeginPlay때 생성하고, 피격 시 이 Widget의 Widget Animation을 플레이하는 방식으로 설계

Widget Animation의 접근 용이를 위해 UUserWidget 파생 클래스 WDG_EffectBase를 생성하고, BlueprintImplementableEvent로 블루프린트에서 호출하게 설계.

Widget Blueprint에서 Animation을 생성하고 Play Effect에 연결만 하면 C++에서 PlayEffect를 호출하여 Widget Animation을 재생할 수 있다.

```cpp
//Ring.cpp에서 Tick마다 데미지를 주는 부분

void ARing::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	FHitData hitData;
	hitData.Damage = RingDamage;

	for(APushCharacter* character : OverlappedCharacters)
	{
		character->Hit_Implementation(hitData);

		APlayerController* controller = Cast<APlayerController>(character->GetController());

		if (controller == nullptr)
			return;

		AMainHUD* hud = Cast<AMainHUD>(controller->GetHUD());

		if (hud == nullptr)
			return;

		hud->EffectWidget->PlayEffect();
	}
}

```

```cpp
//WDG_EffectBase.h
#pragma once

#include "CoreMinimal.h"
#include "Blueprint/UserWidget.h"
#include "WDG_EffectBase.generated.h"

UCLASS(Abstract)
class PUSH_API UWDG_EffectBase : public UUserWidget
{
	GENERATED_BODY()

public:
	UFUNCTION(BlueprintImplementableEvent)
		void PlayEffect();
};
```


**Hit Interface 세부 구현**

```cpp
//H
void APushCharacter::Hit_Implementation(const FHitData& InHitData)
{
    if(ResourceComponent != nullptr)
    {
		ResourceComponent->AdjustHP(-InHitData.Damage);
    }
    if(InHitData.xLaunch + InHitData.yLaunch + InHitData.zLaunch > 0.0f)
    {
        LaunchCharacter(FVector(InHitData.xLaunch, InHitData.yLaunch, InHitData.zLaunch), false, false);
    }

    if(InHitData.Effect != nullptr)
    {
        UParticleSystem* particle = Cast<UParticleSystem>(InHitData.Effect);
        UNiagaraSystem* niagara = Cast<UNiagaraSystem>(InHitData.Effect);

        if(particle != nullptr)
        {
            UGameplayStatics::SpawnEmitterAtLocation(GetWorld(), particle, InHitData.Location, FRotator::ZeroRotator, InHitData.EffectScale);
        }

        if(niagara != nullptr)
        {
            UNiagaraFunctionLibrary::SpawnSystemAtLocation(GetWorld(), niagara, InHitData.Location, FRotator::ZeroRotator, InHitData.EffectScale);
        }
    }

    if(InHitData.Sound != nullptr)
    {
        UGameplayStatics::SpawnSoundAtLocation(GetWorld(), InHitData.Sound, InHitData.Location);
    }
}

```

Interface를 통해 상속받 Hit는 ResourceComponent의 체력을 조정, Launch, Effect, Sound 등이 존재하면 실행한다.

**현재 문제점**
LaunchCharacter가 Replication되지 않아 한 쪽 클라이언트에서만 날라가는 현상이 발생. Hit_Implementation을 NetMulticast를 통해 모든 클라이언트에 동기화해야하며, 맞은 쪽에서 날라가게 해야한다.
