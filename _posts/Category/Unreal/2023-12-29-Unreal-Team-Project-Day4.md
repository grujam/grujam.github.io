---
layout: single
title: "Unreal Team Project - Day 4"
categories: Unreal
classes: wide
---

# Day 4 : IDamageable Interface

## 2023.12.28 Friday Scrum Log

- Projectile Skill Replication 완료, 스킬 애셋 탐색 후 여러 테스트 스킬 작성, Hit Interface 적용 → IS, MH

- Player에 Hit Interface 적용, 자기장 클래스 생성 → 작성자

- GamePlay 및 GameMode 추가 작업, CrossHair 및 HP바 생성 → SW

- 기본 조작 및 ResourceComponent 작성 → HJ

## Daily 작업 과정

**Player Hit Interface 적용**

```cpp
// Copyright Epic Games, Inc. All Rights Reserved.

#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Character.h"
#include "Interface/Damageable.h"
#include "PushCharacter.generated.h"

UCLASS(config = Game)
class APushCharacter : public ACharacter, public IDamageable
{
	GENERATED_BODY()

public:
	APushCharacter();

	virtual void BeginPlay() override;
	virtual void Tick(float DeltaSeconds) override;

protected:
	virtual void SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent) override;

public:
	FORCEINLINE class USpringArmComponent* GetCameraBoom() const { return CameraBoom; }
	FORCEINLINE class UCameraComponent* GetFollowCamera() const { return FollowCamera; }

public:
	UFUNCTION(BlueprintCallable)
		void NumberPressed();

	class ASkill* SkillActor;

	//2024_01_02 Hit Interface 적용
	//다른 Actor에서 피격 시 Hit_Implementation을 Call해서 해주세요
	virtual void Hit(const FHitData& InHitData) override {};
	virtual void Hit_Implementation(const FHitData& InHitData) override;

private:
	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = Camera, meta = (AllowPrivateAccess = "true"))
		class USpringArmComponent* CameraBoom;

	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = Camera, meta = (AllowPrivateAccess = "true"))
		class UCameraComponent* FollowCamera;

	UPROPERTY(VisibleAnywhere)
		class UResourceComponent* ResorceComponent;
	UPROPERTY(VisibleAnywhere)
		class UMoveComponent* MoveComponent;

	UPROPERTY(EditAnywhere)
		TSubclassOf<class ASkill> SkillClass;

};
```

Hit Interface의 내용을 재정의 하여 Player에 적용

*해소되지 않은 점:* Interface에서 NetMultiCast를 통해 선언한 함수는 원본과 _Implementation 둘 모두 재정의하지 않으면 빌드 오류가 발생한다 -> BlueprintNativeEvent 처럼 작동?

구현 내용은 추후 ResourceComponent 업데이트 후 추가 적용

**자기장 클래스 Ring**

```cpp
//Ring.h
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "Ring.generated.h"

UCLASS()
class PUSH_API ARing : public AActor
{
	GENERATED_BODY()

public:
	ARing();

public:
	virtual void OnConstruction(const FTransform& Transform) override;

protected:
	virtual void BeginPlay() override;
	virtual void EndPlay(const EEndPlayReason::Type EndPlayReason) override;

public:
	virtual void Tick(float DeltaTime) override;

public:
	//Collision을 담당하는 Capsule
	UPROPERTY(EditAnywhere)
		class UCapsuleComponent* RingCapsule;

	//자기장 생김새를 담당하는 Mesh
	UPROPERTY(EditAnywhere)
		class UStaticMeshComponent* RingMesh;

	//자기장 피해량
	UPROPERTY(EditAnywhere)
		float RingDamage = 5.0f;

public:
	//자기장 크기 줄이는 함수
	//InRadius: 줄어들 자기장 반지름 크기, InTime: 줄어들 시간 (초단위)
	UFUNCTION(BlueprintCallable)
		void Shrink(float InRadius, float InTime);

	//Timer를 통해 자기장 줄어드는 것을 적용하는 함수
	UFUNCTION(BlueprintCallable)
		void ChangeRadius();

	//OnConstruction 때 Mesh와 Capsule의 크기를 맞춰주기 위한 함수
	UFUNCTION()
		void Refresh();

	UFUNCTION()
		void OnBeginOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult);

	UFUNCTION()
		void OnEndOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex);

private:
	// Base로 나누어 비율 계산하면 Capsule과 Mesh의 크기가 딱 맞습니다
	float Base = 50.0f;

	TArray<class APushCharacter*> OverlappedCharacters;
	FTimerHandle TimerHandle;

	float DeltaRadius = 0.0f;
	float TargetRadius;
};

```

```cpp
//Ring.cpp
#include "Objects/Ring.h"
#include "Global.h"
#include "Character/PushCharacter.h"

ARing::ARing()
{
	PrimaryActorTick.bCanEverTick = true;

	Helpers::CreateComponent<UCapsuleComponent>(this, &RingCapsule, "RingCapsule");
	Helpers::CreateComponent<UStaticMeshComponent>(this, &RingMesh, "RingMesh", RingCapsule);

	FString path = "StaticMesh'/Game/StarterContent/Shapes/Shape_Cylinder.Shape_Cylinder'";
	RingMesh->SetStaticMesh(ConstructorHelpers::FObjectFinder<UStaticMesh>(*path).Object);

	RingMesh->SetCollisionProfileName("OverlapAllDynamic");
}

void ARing::OnConstruction(const FTransform& Transform)
{
	Super::OnConstruction(Transform);
	TargetRadius = RingCapsule->GetUnscaledCapsuleRadius();
	Refresh();
}

void ARing::BeginPlay()
{
	Super::BeginPlay();


	RingCapsule->OnComponentBeginOverlap.AddDynamic(this, &ARing::OnBeginOverlap);
	RingCapsule->OnComponentEndOverlap.AddDynamic(this, &ARing::OnEndOverlap);
}

void ARing::EndPlay(const EEndPlayReason::Type EndPlayReason)
{
	Super::EndPlay(EndPlayReason);

	GetWorldTimerManager().ClearTimer(TimerHandle);
}

void ARing::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	FHitData hitData;
	hitData.Damage = RingDamage;

	for(APushCharacter* character : OverlappedCharacters)
	{
		character->Hit_Implementation(hitData);
	}
}

void ARing::Shrink(float InRadius, float InTime)
{
	DeltaRadius = (RingCapsule->GetUnscaledCapsuleRadius() - InRadius) / (InTime * 100);
	TargetRadius = InRadius;
	GetWorldTimerManager().SetTimer(TimerHandle, this, &ARing::ChangeRadius, 0.01f, true, 0.0f);
}

void ARing::ChangeRadius()
{
	float radius = RingCapsule->GetUnscaledCapsuleRadius() - DeltaRadius;
	RingCapsule->SetCapsuleRadius(radius);
	CLog::Print(radius);
	if (FMath::IsNearlyEqual(radius, TargetRadius))
		GetWorldTimerManager().ClearTimer(TimerHandle);

	float scale = radius / Base;
	RingMesh->SetWorldScale3D(FVector(scale, scale, scale * 100));
}

void ARing::Refresh()
{
	float radius = RingCapsule->GetUnscaledCapsuleRadius();
	RingCapsule->SetCapsuleRadius(radius);

	float scale = radius / Base;
	RingMesh->SetWorldScale3D(FVector(scale, scale, scale * 100));
}

void ARing::OnBeginOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	APushCharacter* character = Cast<APushCharacter>(OtherActor);

	if (!ensure(character != nullptr))
		return;

	if(OverlappedCharacters.Contains(character))
		OverlappedCharacters.Remove(character);
}

void ARing::OnEndOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex)
{
	APushCharacter* character = Cast<APushCharacter>(OtherActor);

	if (!ensure(character != nullptr))
		return;

	OverlappedCharacters.AddUnique(character);
}
```

Ring 클래스는 Capsule Collision과 Static Mesh의 조합.

Capsule Collsion은 Mesh를 가질 수 없기 때문에 Static Mesh를 통해 자기장과 같은 형태를 구현, 실제 Overlap은 Capsule Collision에서 적용.


**데미지 구현 방법**

Ring은 ACharacter의 배열을 보유, 정해진 Tick 시간 초 마다 배열에 있는 캐릭터들에게 데미지 적용.

배열은 EndOverlap의 경우 Overlap한 캐릭터를 배열에 넣고, BeginOverlap의 경우 자기장 안으로 들어온 것 이므로 캐릭터를 배열에서 제거.


**자기장 축소 구현 방법**

Shrink 함수를 통해 줄어들 크기와 줄어드는데 걸리는 시간 초를 받는다.

0.01초마다 줄어들 크기를 (현재 크기 - 줄어들 크기) / (걸리는 시간 * 100)를 통해 구하고, Timer를 통해 0.01초마다 줄어들게 하며 TargetRadius를 줄어들 크기로 정한다.

줄어드는 것을 적용하는 ChangeRadius 함수에서 현재 크기가 TargetRadius에 가까워 지면 Timer를 Clear하여 줄어드는 것을 멈춘다.
