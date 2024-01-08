---
layout: single
title: "Unreal Team Project - Day 6"
categories: Unreal
classes: wide
---

# Day 6 : SkillComponent and SkillData

## 2024.01.03 Wednesday Scrum Log

- MH, 작성자 → SkillComponent 및 SkillData 설계 및 구현

- IS → 지점 스킬의 피해 위치 Decal Radius 조절하여 유동적으로 구현

- SW → 게임 서버와 클라이언트 데이터 분리 하여 브로드 캐스팅 하는 방법 찾기

- HJ → ItemComponent 및 Item 설계 & 구현 시작

## Daily 작업 과정

**SkillData**

SkillData는 Skill들의 내용을 관리하기 위한 UObject 형태의 데이터 구조.

하나의 스킬을 사용할 때 하는 동작 ActionMontage와 그 재생속도 PlayRate를 보유하고, 만들 스킬타입 Skill과 생성위치 RelativeDistance를 갖는다.

ActionMontage에서는 추후 Notify를 추가하여 Skill을 스폰하고 싶은 시점에 스폰할 수 있도록 구현.

```cpp
//SkillData.h
#pragma once

#include "CoreMinimal.h"
#include "UObject/NoExportTypes.h"
#include "SkillData.generated.h"

UCLASS(Blueprintable, BlueprintType)
class PUSH_API USkillData : public UObject
{
	GENERATED_BODY()

public:
	virtual void BeginPlay();

public:
	UPROPERTY(EditAnywhere)
		class UAnimMontage* ActionMontage;

	UPROPERTY(EditAnywhere)
		float PlayRate = 1.0f;

	UPROPERTY(EditAnywhere)
		TSubclassOf<class ASkill> Skill;

	UPROPERTY(EditAnywhere)
		float RelativeDistance = 0;

public:
	UFUNCTION()
		virtual void Play(ACharacter* InOwner);

};
```

**Skill Component**

Skill Component는 추후 단축키에 등록된 SkillData를 가져와 SkillData의 Play를 통해 스킬을 시전하는 것을 담당한다.

스킬의 실행은 UseSkill과 Execute 두 부분으로 나누어 단축키를 눌렀을 때와 클릭으로 진행하는 것을 구현하였다.

UseSkill은 단축키로 시전하고 싶은 스킬을 시전하고, 투사체의 경우 즉발로 발사하지만 지점 스킬의 경우 데칼 위치에 스폰하는 것을 구현해야하기 때문에 캐릭터의 클릭에 바인딩 될 Execute를 추가하였다.

UseSkill에서 curSkillData에 등록시켜주기 때문에 이후 Execute를 통해 현재 시전 준비된 스킬을 가져오게 하며, 이를 리셋해주는 것은 추후 추가 예정.

```cpp
//SkillComponent.h
#pragma once

#include "CoreMinimal.h"
#include "Components/ActorComponent.h"
#include "SkillComponent.generated.h"

UCLASS( ClassGroup=(Custom), meta=(BlueprintSpawnableComponent) )
class PUSH_API USkillComponent : public UActorComponent
{
	GENERATED_BODY()

public:
	USkillComponent();

protected:
	virtual void BeginPlay() override;

public:
	virtual void TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction) override;

public:
	void UseSkill(char InChar);

	UFUNCTION(BlueprintCallable)
		void Execute();

public:
	TMap<char, class USkillData*> SkillMap;
};

public:
	USkillData* curSkillData;
```

```cpp
//SkillComponent.cpp
#include "Components/SkillComponent.h"
#include "GameFramework/Character.h"
#include "Net/UnrealNetwork.h"
#include "Skill/SkillData.h"
#include "Skill/Skill.h"
#include "Skill/SkillDatas/SkillData_Projectile.h"
#include "Global.h"

USkillComponent::USkillComponent()
{
	PrimaryComponentTick.bCanEverTick = true;
}

void USkillComponent::BeginPlay()
{
	Super::BeginPlay();

	Owner = Cast<ACharacter>(GetOwner());
}

void USkillComponent::TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction)
{
	Super::TickComponent(DeltaTime, TickType, ThisTickFunction);

}

void USkillComponent::UseSkill(char InChar)
{
	if (!SkillMap.Contains(InChar))
		return;

	curSkillData = SkillMap[InChar];
	Execute();
}

void USkillComponent::Execute()
{
	curSkillData->Play(Owner);
}
```
