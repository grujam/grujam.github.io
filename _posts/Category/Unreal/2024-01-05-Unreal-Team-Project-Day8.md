---
layout: single
title: "Unreal Team Project - Day 8"
categories: Unreal
classes: wide
---

# Day 8 : SkillComponent and SkillData

## 2024.01.05 Friday Scrum Log

- MH → 상점 UI 작업

- IS → 버그 코드 수정 완료, 스킬 구조 변경 및 클라이언트 별 색 변경

- SW → 간단한 UI 생성, Game State 변경 버그 해결

- HJ → 휴가

- 작성자 →SpawnSkill Replication 해결

## Daily 작업 과정

**SpawnSkill Replication 해결 방법**

가장 중요한 SpawnLocation은 SkillComponent가 보유, 이를 Replicate하기 위해 Skill Component에서 RPC 호출 → 실패, RPC는 Actor에서만 호출 가능

PushCharacter에 SpawnLocation을 동기화 해주는 RPC를 생성하여 SpawnLocation 동기화 해결 성공.

SpawnLocation 정하는 부분은 Projectile Skill의 경우 플레이어의 일정 거리 앞, 지점 스킬인 경우 Decal Location을 통해 스폰하기 때문에 이를 판별하는 부분도 넣어준다.

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

	//HasAuthority == false → 클라이언트이므로 무시한다.
	if (Owner == nullptr || Owner->HasAuthority() == false)
		return;

	USkillComponent* SkillComponent =  Helpers::GetComponent<USkillComponent>(Owner);

	if (SkillComponent == nullptr)
		return;

	FVector SpawnLocation, RelativeSpawnLocation;

	TSubclassOf<ASkill> SpawnSkill = SkillComponent->curSkillData->Skill;

	USkillData_Projectile* SkillData = Cast<USkillData_Projectile>(SkillComponent->curSkillData);
	if (SkillData != nullptr)
	{
		SpawnLocation = Owner->GetActorLocation() + Owner->GetActorForwardVector() * SkillComponent->curSkillData->RelativeDistance;
	}
	else
	{
		RelativeSpawnLocation = SkillComponent->SpawnLocation - Owner->GetActorLocation();
		SpawnLocation = Owner->GetActorLocation() + RelativeSpawnLocation + Owner->GetActorForwardVector() * SkillComponent->curSkillData->RelativeDistance;
	}

	FRotator SpawnRotation = Owner->GetActorForwardVector().Rotation();

	FActorSpawnParameters params;
	params.Owner = Owner;
	params.SpawnCollisionHandlingOverride = ESpawnActorCollisionHandlingMethod::AlwaysSpawn;

	Owner->GetWorld()->SpawnActor<ASkill>(SpawnSkill, SpawnLocation, SpawnRotation, params);
}
```

SpawnLocation 동기화 후 NF_SpawnKill에서 MeshComp의 Owner의 Authority 검사를 통해 서버에서만 스폰해서 양쪽 클라이언트에 동일하게 스폰하는 것을 확인.

![](/assets/images/Unreal/FireballReplication.PNG)
*양쪽 클라이언트에서 파이어볼이 나가는 모습*


**문제점**

Fireball은 내부 구현을 통해 Owner의 Controller를 따라 방향을 회전하게 하였지만, 서버에서 생성한 파이어볼의 Owner는 Controller를 갖지 않아 회전하지 않음.

Server에서 생성하지 않고 각 클라이언트에서 생성하도록 변경하면 시전한 Client의 Controller는 존재하지만 다른 클라이언트에서는 해당 Controller가 존재하지 않음.

마지막 해결 방법: 각 클라이언트에서 생성하고, ControlRotation이 아닌 Mesh의 ForwardVector 방향을 따르게 하여 해결.

```cpp
//Projectile.cpp의 Tick
void AProjectile::Tick(float DeltaSeconds)
{
	Super::Tick(DeltaSeconds);

	//회전하는 스킬만 따라 회전하게 변경
	if(Datas.CanRotate == true)
	{
		// 231229 이민학
		// 방향 조절 Projectile 핵심코드
		if (!!Owner)
		{
			velocity = Owner->GetActorForwardVector() * ProjectileComponent->InitialSpeed;

			ProjectileComponent->Velocity = UKismetMathLibrary::VInterpTo(ProjectileComponent->Velocity, velocity, DeltaSeconds, Datas.InterpSpeed);
			SetActorRotation(UKismetMathLibrary::RInterpTo(GetActorRotation(), rotation, DeltaSeconds, Datas.InterpSpeed));
		}
	}
}
```

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

	//HasAuthority 체크하는 부분을 true로 변경
	if (Owner == nullptr || Owner->HasAuthority() == true)
		return;

	USkillComponent* SkillComponent =  Helpers::GetComponent<USkillComponent>(Owner);

	if (SkillComponent == nullptr)
		return;

	FVector SpawnLocation, RelativeSpawnLocation;

	TSubclassOf<ASkill> SpawnSkill = SkillComponent->curSkillData->Skill;

	USkillData_Projectile* SkillData = Cast<USkillData_Projectile>(SkillComponent->curSkillData);
	if (SkillData != nullptr)
	{
		SpawnLocation = Owner->GetActorLocation() + Owner->GetActorForwardVector() * SkillComponent->curSkillData->RelativeDistance;
	}
	else
	{
		RelativeSpawnLocation = SkillComponent->SpawnLocation - Owner->GetActorLocation();
		SpawnLocation = Owner->GetActorLocation() + RelativeSpawnLocation + Owner->GetActorForwardVector() * SkillComponent->curSkillData->RelativeDistance;
	}

	FRotator SpawnRotation = Owner->GetActorForwardVector().Rotation();

	FActorSpawnParameters params;
	params.Owner = Owner;
	params.SpawnCollisionHandlingOverride = ESpawnActorCollisionHandlingMethod::AlwaysSpawn;

	Owner->GetWorld()->SpawnActor<ASkill>(SpawnSkill, SpawnLocation, SpawnRotation, params);
}
```
