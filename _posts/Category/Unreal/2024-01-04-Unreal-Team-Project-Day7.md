---
layout: single
title: "Unreal Team Project - Day 7"
categories: Unreal
classes: wide
---

# Day 7 : SkillComponent and SkillData

## 2024.01.03 Wednesday Scrum Log

- MH, 작성자 → SkillComponent 및 SkillData 추가 작업

- IS → 버그 난 코드 복구 및 여러 스킬 구현

- SW → 기본 UI 게임모드에서 타이머 리플리케이션 정상 작동 확인

- HJ → ItemComponent 및 Item 설계 & 구현 시작

## Daily 작업 과정

**SkillComponent**

아직 상점 UI를 통해 TMap에 SkillData를 등록하는 부분을 구현하지 않아 임의로 클래스를 등록하여 파이어볼 시전을 구현 완료.

각 클라이언트 별 동일한 파이어볼 위치 스폰을 위해 SpawnLocation을 Replicate하여 스폰하려 했지만, Replication이 정상작동하지 않아 실패.

→ 다른 Replication 방법을 통해 해결 예정

스폰 지점을 Skill 시전 시 정하기 때문에 움직이며 시전하는 스킬은 이전의 위치에서 스폰되는 문제 발생

→ AnimNotify에서 스폰 지점을 정하고 스폰하도록 한다.

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
	ACharacter* Owner;

public:
	UPROPERTY(EditAnywhere)
		TSubclassOf<USkillData> ss;

public:
	UPROPERTY(EditAnywhere)
		class USkillData* curSkillData;

	UPROPERTY(EditAnywhere, Replicated)
		FVector SpawnLocation;

	void GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const override;
};
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

	if(ss != nullptr)
		curSkillData = NewObject<USkillData>(Owner, ss);
}

void USkillComponent::TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction)
{
	Super::TickComponent(DeltaTime, TickType, ThisTickFunction);

}

void USkillComponent::UseSkill(char InChar)
{
	if (!SkillMap.Contains(InChar))
		return;

	// curSkillData = SkillMap[InChar];
	Execute();
}

void USkillComponent::Execute()
{
	curSkillData->Play(Owner);
}

void USkillComponent::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
	Super::GetLifetimeReplicatedProps(OutLifetimeProps);

	DOREPLIFETIME(USkillComponent, SpawnLocation);
}
```

**SkillData**

상점에 등록할 이름과 Texture 변수 추가

```cpp
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
	UPROPERTY(EditAnywhere, BlueprintReadOnly)
		FString SkillName;

	UPROPERTY(EditAnywhere, BlueprintReadOnly)
		class UTexture2D* SkillTexture;

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

**NF_SpawnSkill**

처음 SpawnSkill을 통해 각 클라이언트에서 생성 시도 → SpawnLocation이 Replicate 되지 않아 맞지 않음

SpawnSkill을 서버에서만 실행 → 서버에서도 SpawnLocation이 Replicate 되지 않음

→ 추후 Spawn 위치 조정..

```cpp
//NF_SpawnSkill.cpp
#include "Notifies/Notify/NF_SpawnSkill.h"
#include "Global.h"
#include "GameFramework/Character.h"
#include "Components/SkillComponent.h"
#include "Skill/Skill.h"
#include "Skill/SkillData.h"
#include "Skill/SkillDatas/SkillData_Projectile.h"

FString UNF_SpawnSkill::GetNotifyName_Implementation() const
{
	return "Spawn Skill";
}

void UNF_SpawnSkill::Notify(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation)
{
	Super::Notify(MeshComp, Animation);

	ACharacter* Owner = Cast<ACharacter>(MeshComp->GetOwner());
	if (Owner == nullptr)
		return;

	USkillComponent* SkillComponent =  Helpers::GetComponent<USkillComponent>(Owner);

	if (SkillComponent == nullptr)
		return;

	FVector SpawnLocation, RelativeSpawnLocation;

	TSubclassOf<ASkill> SpawnSkill = SkillComponent->curSkillData->Skill;

	USkillData_Projectile* SkillData = Cast<USkillData_Projectile>(SkillComponent->curSkillData);

	RelativeSpawnLocation = SkillComponent->SpawnLocation - Owner->GetActorLocation();
	SpawnLocation = Owner->GetActorLocation() + RelativeSpawnLocation + Owner->GetActorForwardVector() * SkillComponent->curSkillData->RelativeDistance;

	FRotator SpawnRotation = Owner->GetActorForwardVector().Rotation();

	FActorSpawnParameters params;
	params.Owner = Owner;
	params.SpawnCollisionHandlingOverride = ESpawnActorCollisionHandlingMethod::AlwaysSpawn;

	Owner->GetWorld()->SpawnActor<ASkill>(SpawnSkill, SpawnLocation, SpawnRotation, params);

	//if(controller == Owner->GetWorld()->GetFirstPlayerController())
		//SkillComponent->SpawnCallOnServer_Implementation(controller, SpawnSkill, SpawnLocation, SpawnRotation);

  //SkillComponent->SpawnCallOnServer_Implementation(SpawnSkill, SpawnLocation, SpawnRotation);
}
```
