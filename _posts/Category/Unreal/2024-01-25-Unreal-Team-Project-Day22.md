---
layout: single
title: "Unreal Team Project - Day 22"
categories: Unreal
classes: wide
---

# Day 22 : 라운드 종료 시 순위에 따른 골드 및 기본 골드 지급

## 2024.01.25 Thursday Scrum Log


### Daily Review

- MH → 트랜스볼 완료, 스킬 쿨타임 버그 해결

- IS → 킬로그 제작

- SW → 상점 페이즈 동안 행동 불가

- HJ → 지뢰 스킬 제작, 포션 아이템 제작

- 작성자 → 라운드 분할 및 킬 뎃 골드 동기화


### Works to do

- MH → 카메라 셰이크 적용해보기

- IS → 채팅 시스템, 및 에임 오프셋 시도

- SW, 작성자 → 라운드 시스템 완성

- HJ → 아이템 효과 이펙트 추가


## Daily 작업 과정

**라운드 종료 시 순위 판별 및 골드 지급**

라운드 종료 시 순위를 판별하기 위해서는 사망한 플레이어를 배열에 집어넣고 역순으로 판별해야 한다.

사망할 때 마다 플레이어를 배열에 넣고 진행하였지만 간과한 점이 한가지 있다 → 배틀로얄은 모두 죽고 끝나는 것이 아닌 한명이 남으면 끝남.

즉, 마지막 1등 플레이어는 배열에 들어가지 않기 때문에 다른 방법으로 넣어주어야 한다.

죽은 플레이어를 배열에 넣고 배열의 크기가 현재 플레이어들의 수보다 1만큼 적을 때, 마지막 남은 플레이어를 배열에 넣어 순위를 판별한다.

```cpp
//PushCharacter에서 사망 시 서버의 GameMode에게 사망을 알리는 함수
void APushCharacter::Dead_Server_Implementation()
{
    APushGameMode* GameMode = Cast<APushGameMode>(UGameplayStatics::GetGameMode(GetWorld()));

    if (GameMode == nullptr)
        return;

    APushPlayerController* controller = Cast<APushPlayerController>(GetController());

    if (controller == nullptr)
        return;

    GameMode->PlayerDead(controller);
}
```

```cpp
//PushGameMode에서 죽은 플레이어의 컨트롤러를 배열에 넣으라고 PushGameState에게 알리며, 마지막 플레이러를 확인하는 함수
void APushGameMode::PlayerDead(APushPlayerController* InController)
{
	PushGameState->AddToRank(InController);

	if (++NumofDeadPlayers >= (PushGameState->PlayerArray.Num() - 1))
	{
		for (APlayerState* player : PushGameState->PlayerArray)
		{
			APushCharacter* character = Cast<APushCharacter>(player->GetPawn());

			if (character == nullptr)
				continue;

			UStateComponent* state = Helpers::GetComponent<UStateComponent>(character);

			if (state == nullptr)
				continue;

			if (!state->IsDeadMode())
			{
				APushPlayerController* controller = Cast<APushPlayerController>(character->GetController());

				if (controller == nullptr)
					continue;

				PushGameState->AddToRank(controller);
				break;
			}
		}
		tempTime = GetWorld()->GetTimeSeconds();
		CountdownTime = ResultTime;
		Round = 0;
		SetMatchState(MatchState::Result); // 결과발표
	}

}
```

지급되는 기본 골드는 InBaseMoney, 순위 골드는 InGoldAmount 배열을 직렬화하여 코드 외적으로 수정하여 지급할 수 있게 구현

```cpp
//PushGameState에서 Controller를 배열에 넣고, 순위 별로 골드를 주는 함수
void APushGameState::GiveGold(TArray<int32> InGoldAmount, int32 InBaseMoney)
{
	int num = RoundRank.Num();
	for (uint8 i = 0; i < num; i++)
	{
		APushPlayerController* controller = RoundRank.Pop();
		UResourceComponent* resource = Helpers::GetComponent<UResourceComponent>(controller->GetCharacter());

		CLog::Log(InGoldAmount[i]);
		resource->AdjustGold_NMC(InGoldAmount[i] + InBaseMoney);
	}
}

void APushGameState::AddToRank(APushPlayerController* InController)
{
	RoundRank.Push(InController);
}
```
