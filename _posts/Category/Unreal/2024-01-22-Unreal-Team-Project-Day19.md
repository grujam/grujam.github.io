---
layout: single
title: "Unreal Team Project - Day 19"
categories: Unreal
classes: wide
---

# Day 19 : Ring 초기화

## 2024.01.22 Monday Scrum Log

### Daily Review

- UML 제작 및 Refactoring 완료


### Works to do

- MH → 상점 스킬 및 아이템 골드 판별, 스킬 설명 추가

- IS → 휴가

- SW → 게임 모드에 Ring 추가, 조절

- HJ → 버프 RPC 버그 수정

- 작성자 → 게임 모드 Ring 조절 용으로 수정


## Daily 작업 과정

기존 Ring은 맵에 올려놓고 조절 시 크기 반영과 Shrink 함수를 통해 몇초 동안 줄어드는 것만 적용.

추후 라운드 분할 적용 시 여러번에 걸쳐서 나눠지는 것은 Shrink를 통해 적용할 수 있지만, 하나의 라운드가 종료될 때 원래대로 돌아가는 것을 추가해야 함.

StartRadius라는 변수에 BeginPlay에서 초기 원의 반지름을 저장하고, Reset에서 이를 통해 조정한다.

```cpp
void ARing::BeginPlay()
{
	Super::BeginPlay();

	RingCapsule->OnComponentBeginOverlap.AddDynamic(this, &ARing::OnBeginOverlap);
	RingCapsule->OnComponentEndOverlap.AddDynamic(this, &ARing::OnEndOverlap);
	StartRadius = RingCapsule->GetUnscaledCapsuleRadius();
}

void ARing::Reset()
{
	RingCapsule->SetCapsuleRadius(StartRadius);

	float scale = StartRadius / Base;
	RingMesh->SetWorldScale3D(FVector(scale, scale, scale * 100));
}
```
