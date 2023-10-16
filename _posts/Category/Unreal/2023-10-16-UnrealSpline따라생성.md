---
layout: single
title: "Unreal C++ - Spline Mesh"
categories: Unreal
classes: wide
---

### Unreal C++ - Spline을 따라 Mesh 생성

언리얼에서 Zipline과 같이 Spline을 따라 Mesh를 생성하는 방법을 기록하는 포스트입니다.
![UnrealSplineMesh](/assets/images/Unreal/SplineMesh.PNG)

Spline을 따라 Mesh를 생성할 ActorClass를 생성합니다.   
해당 포스트에서는 CZipline 클래스를 생성하였습니다.

C++로 생성한 액터는 DefaultSceneRoot가 생성되지 않기 때문에 Root 컴포넌트를 생성하고, Spline을 그리기 위해 USplineComponent* Spline을 생성합니다.

추가적으로 Spline을 따라 그리게 될 StaticMesh를 그때 그때 외부에서 지정해주기 위해 UStaticMesh* Mesh 변수를 만들고 EditAnywhere로 생성합니다.

```cpp
//CZipline.h 파일
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "CZipline.generated.h"

UCLASS()
class CPP_PORTFOLIO_API ACZipline : public AActor
{
	GENERATED_BODY()

public:
	ACZipline();

	virtual void Tick(float DeltaSeconds) override;

protected:
	virtual void OnConstruction(const FTransform& Transform) override;

	virtual void BeginPlay() override;

public:
	UPROPERTY(VisibleAnywhere)
		class USceneComponent* Root;

	UPROPERTY(VisibleAnywhere)
		class USplineComponent* Spline;

	UPROPERTY(EditAnywhere)
		class UStaticMesh* Mesh;

private:
	UFUNCTION(BlueprintCallable)
		void CreateLine();

	UFUNCTION()
		void FollowLine(class ACharacter* InCharacter);

private:
	int Distance;
};
```

Spline을 따라 Mesh를 생성하는 부분은 BeginPlay에 생성하여 게임이 시작된 후에 표시할 수도 있지만   
시작 전에도 잘 그려졌는지 확인할 수 있게 OnConstruction에 선을 생성하는 부분을 작성하여 줍니다.

```cpp
//CZipline.cpp 파일
#include "Obstacle/CZipline.h"
#include "Components/SplineComponent.h"
#include "Components/SplineMeshComponent.h"

ACZipline::ACZipline()
{
	Root = CreateDefaultSubobject<USceneComponent>("Root");
	SetRootComponent(Root);

	Spline = CreateDefaultSubobject<USplineComponent>("Spline");
	Spline->SetupAttachment(Root);
}

void ACZipline::OnConstruction(const FTransform& Transform)
{
	Super::OnConstruction(Transform);
	CreateLine();
}

void ACZipline::BeginPlay()
{
	Super::BeginPlay();

}

//Mesh 생성해주는 메서드
void ACZipline::CreateLine()
{
	int8 num = Spline->GetNumberOfSplinePoints() - 2;

	for (int8 i = 0; i < num + 1; ++i)
	{
		FVector StartLocation, EndLocation, StartTangent, EndTangent;

		Spline->GetLocationAndTangentAtSplinePoint(i, StartLocation, StartTangent, ESplineCoordinateSpace::Local);
		Spline->GetLocationAndTangentAtSplinePoint(i + 1, EndLocation, EndTangent, ESplineCoordinateSpace::Local);

		USplineMeshComponent* mesh = NewObject<USplineMeshComponent>(this, USplineMeshComponent::StaticClass());
		mesh->SetMobility(EComponentMobility::Movable);

		if (Mesh != nullptr)
			mesh->SetStaticMesh(Mesh);

		mesh->SetStartAndEnd(StartLocation, StartTangent, EndLocation, EndTangent);
		mesh->AttachToComponent(Root, FAttachmentTransformRules::KeepRelativeTransform);
	}
}

void ACZipline::FollowLine(ACharacter* InCharacter)
{

}

```

Mesh를 Spline을 따라 생성하는 방법은 Spline의 점마다 점의 시작점과 끝점, 시작 부분의 탄젠트 값과 끝 부분의 탄젠트 값을 GetLocationAndTangentAtSplinePoint를 통해 가져옵니다.

USplineMeshComponent를 통해 지정한 Mesh를 SetStartAndEnd를 통해 생성이 시작될 시작점과 끝부분, 시작 탄젠트와 끝 탄젠트를 넘겨주면 Mesh가 생성되고, 이를 Root에 붙여줍니다.

이 때 주의할 점은 Root의 Mobility와 USplineMeshComponent의 Mobility가 일치하지 않으면 Attach가 되지 않기 때문에 Mobility를 일치시켜 주어야 합니다.
