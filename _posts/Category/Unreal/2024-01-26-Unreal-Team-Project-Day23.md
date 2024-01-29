---
layout: single
title: "Unreal Team Project - Day 23"
categories: Unreal
classes: wide
---

# Day 23 : 게임 종료 시 순위 출력

## 2024.01.26 Friday Scrum Log


### Daily Review

- MH → 예비 크로스 헤어 제작, 카메라 셰이크 적용, 소환 스킬 작업

- IS → 에임 오프셋 작업

- SW → Ragdoll, Respawn 버그 수정

- HJ → 은신 아이템 추가, 아이템 이펙트들 추가

- 작성자 → 라운드 별 순위 판별 후 골드 지급


### Works to do

- 남은 모든 작업 마무리!


## Daily 작업 과정

**게임 종료 시 순위 출력**

게임 순위는 킬 순위로 결정, 동일 킬이라면 동일 순위

모든 라운드가 종료된 후, 킬 순위를 판별하여 순위를 출력한다.

킬은 각 캐릭터의 ResourceComponent에서 관리되며 이를 가져와 확인하면 쉽게 순위를 판별할 수 있다.

현재는 최대 라운드 수가 없이 계속 반복되므로 최대 라운드 수를 조정할 수 있도록 변수를 추가하고, 마지막 게임이 끝난 후 순위를 출력토록 한다.

게임 라운드는 PushGameMode에서 관리되며, PushGameState를 통해 PlayerArray에 접근하여 순위를 출력토록 한다.

*추가 내용*   
다른 파트에서 라운드 종료 시 내용을 추가하고 싶을 수 있으므로 OnRoundEnd라는 Dynamic Multicast Delegate를 생성하고, 결과 발표 시 BroadCast를 진행.


**최대 라운드**

최대 라운드는 변수 TotalNumOfGames를 추가하고 직렬화하여 수정, MatchState가 Result과 되는 경우 OnRoundEnd.BroadCast를 통해 호출되는 RoundEnd함수에서 진행한 라운드 수 증가.

최대 라운드 종료 시 PushGameState의 ShowTotalRank 호출.

```cpp
//PushGameMode
void APushGameMode::BeginPlay()
{
	Super::BeginPlay();

	tempTime = GetWorld()->GetTimeSeconds();
	CountdownTime = WarmupTime;
	CurrentTime = 0.0f;

	PushGameState = Cast<APushGameState>(GameState);

	if(PushGameState == nullptr){ // 예외처리
		CLog::Log("pushGameState == nullptr !!");
		return;
	}

	TWeakObjectPtr<UWorld> world = GetWorld();

	if (world.IsValid())
	{
		if (IsValid(RingClass))
		{
			Ring = world->SpawnActor<ARing>(RingClass);
			Ring->SetActorLocation(FVector(-45800.f, 15700.f, -24000.f));
		}
	}

	OnRoundEnd.AddDynamic(this, &APushGameMode::RoundEnd);
}

void APushGameMode::Tick(float DeltaSeconds)
{
	Super::Tick(DeltaSeconds);
	PushGameState->SetTime(CurrentTime);
	if (MatchState == MatchState::InProgress) // 상점
	{
		CurrentTime = (CountdownTime - (GetWorld()->GetTimeSeconds() - tempTime));
		if (CurrentTime <= 0.0f)
		{
			CountdownTime = RoundTime[Round];
			tempTime = GetWorld()->GetTimeSeconds();
			SetMatchState(MatchState::Round);
		}
	}
	else if (MatchState == MatchState::Round) // 라운드s
	{
		CurrentTime = (CountdownTime - (GetWorld()->GetTimeSeconds() - tempTime));
		if (CurrentTime <= 0.0f)
		{
			if (RoundTime.Num() - 1 <= Round)
			{
				tempTime = GetWorld()->GetTimeSeconds();
				CountdownTime = ResultTime;
				Round = 0;

				if (Games >= TotalNumOfGames)
				{
					SetMatchState(MatchState::TotalResult);
					PushGameState->ShowTotalRank();
					SetActorTickEnabled(false);
					CurrentTime = 0.0f;
					return;
				}

				SetMatchState(MatchState::Result); // 결과발표
			}
			else
			{
				if (Ring.Get() != nullptr)
					Ring.Get()->Shrink(RingRadius[Round], ShrinkRate);

				CountdownTime = RoundTime[++Round];
				tempTime = GetWorld()->GetTimeSeconds();
			}
		}
	}
	else if (MatchState == MatchState::Result) // 결과발표
	{
		CurrentTime = (CountdownTime - (GetWorld()->GetTimeSeconds() - tempTime));
		if (CurrentTime <= 0.0f)
		{
			tempTime = GetWorld()->GetTimeSeconds();
			CountdownTime = WarmupTime;
			if (OnRoundEnd.IsBound() == true)
				OnRoundEnd.Broadcast();
			SetMatchState(MatchState::InProgress);
		}
	}
}


void APushGameMode::RoundEnd()
{
	PushGameState->GiveGold(MoneyPerRank, BaseMoney);
	PushGameState->UpdateGameNum(++Games);
	Ring->Reset();
}
```


**순위 출력**

PushGameState에서 순위 출력을 하는 과정은 다음과 같다.

1. PushGameState의 ShowTotalRank에서는 PlayerArray에 접근해서 플레이어의 Controller를 전부 배열에 넣는다.

2. TArray의 Sort에 람다식으로 기준을 킬 수로 배열을 정렬한다.

3. 순위는 1부터 시작해서 Controller의 Client RPC를 통해 순위를 출력하고, 킬 수가 동일하다면 순위 변동X, 동일하지 않다면 순위가 내려간다.

```cpp
//PushGameState의 ShowTotalRank
void APushGameState::ShowTotalRank()
{
	TArray<APushPlayerController*> Controllers;


	for (APlayerState* player : PlayerArray)
	{
		APushCharacter* character = Cast<APushCharacter>(player->GetPawn());

		if (character == nullptr)
			continue;

		APushPlayerController* controller = Cast<APushPlayerController>(character->GetController());

		if (controller == nullptr)
			continue;

		Controllers.Add(controller);
	}

	Controllers.Sort([](APushPlayerController& A, APushPlayerController& B)
	{
		uint8 AKill = A.resourceComponent->GetKill();
		uint8 BKill = B.resourceComponent->GetKill();

		return AKill > BKill;
	});

	int Rank = 1;
	uint8 CurrentKill = Controllers[0]->resourceComponent->GetKill();

	for (APushPlayerController* controller : Controllers)
	{
		uint8 thisKill = controller->resourceComponent->GetKill();

		if (thisKill < CurrentKill)
		{
			Rank++;
			CurrentKill = thisKill;
		}

		controller->ShowRank_Client(Rank, RankWidget);
	}
}
```
