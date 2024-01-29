---
layout: single
title: "Unreal Team Project - Day 21"
categories: Unreal
classes: wide
---

# Day 21 : Round 분할

## 2024.01.24 Wednesday Scrum Log

### Daily Review

- MH → 지점 스킬 쿨타임 버그 수정, 트랜스볼 스킬 제작

- IS → 킬로그 진행 중

- SW → Ragdoll 처리 및 라운드 초기화 스폰 진행 중

- HJ → RPC 버그 수정, 스킬 제작

- 작성자 → 면접


### Works to do

- MH → 트랜스볼 버그 수정

- IS → 킬로그 제작

- SW → 스킬 제작, 라운드 초기화 및 상점과 결과 시간 공격 금지 적용

- HJ → 지뢰 스킬 버그 수정

- 작성자 → 라운드 분할 및 골드, 킬, 뎃 추가


## Daily 작업 과정

**라운드 분할**

현재 게임모드는 라운드가 분할되어 있지 않고 정해진 시간이 지나면 상점, 라운드, 결과가 돌아가며 진행되게 되어있다.

기존 코드의 시간 계산이 난잡하고 라운드 분할에 맞지 않아 시간 계산을 하는 방법을 수정하고 라운드 수와 라운드 별 시간을 코드 외적으로 지정할 수 있게 직렬화 하였다.

```cpp
//PushGameMode.cpp의 Tick을 통해 MatchState를 변경하는 기존 코드
void APushGameMode::Tick(float DeltaSeconds)
{
	Super::Tick(DeltaSeconds);

	//** '대기시간 > 경기시간 > 결과시간'을 반복
	if (MatchState == MatchState::InProgress) // 대기
	{
		// 대기시간 - 현재시간 + 게임레벨맵에 들어간 시간
		CountdownTime = WarmupTime - GetWorld()->GetTimeSeconds() + LevelStartingTime + tempTime;
		if (CountdownTime <= 0.0f) // 대기시간이 끝나면 경기시작
		{
			SetMatchState(MatchState::Round);
		}
	}
	else if (MatchState == MatchState::Round) // 경기
	{
		// 대기시간 - 현재시간 + 게임레벨맵에 들어간 시간 + 설정한 경기시간
		CountdownTime = WarmupTime - GetWorld()->GetTimeSeconds() + LevelStartingTime + MatchTime + tempTime;
		if (CountdownTime <= 0.0f)
		{
			SetMatchState(MatchState::Result); // 결과발표
		}
	}
	else if (MatchState == MatchState::Result) // 결과발표
	{
		// 대기시간 - 현재 시간 + 게임레벨맵에 들어간 시간 + 설정한 경기시간 + 설정한 결과발표시간
		CountdownTime = WarmupTime - GetWorld()->GetTimeSeconds() + LevelStartingTime + MatchTime + ResultTime + tempTime;
		if (CountdownTime <= 0.0f)
		{
			tempTime = GetWorld()->GetTimeSeconds();
			SetMatchState(MatchState::InProgress);
		}
	}

}

void APushGameMode::OnMatchStateSet()
{
	Super::OnMatchStateSet();

	// GameMode의 MatchState이 변경되면 서버에서 해당되는 PlayerController를 찾아 MatchState을 설정한다.
	for (FConstPlayerControllerIterator It = GetWorld()->GetPlayerControllerIterator(); It; ++It)
	{
		TWeakObjectPtr<APushPlayerController> SelectedPlayer = Cast<APushPlayerController>(*It);
		if (SelectedPlayer.IsValid())
		{
			SelectedPlayer->OnMatchStateSet(MatchState);
		}
	}
}
```

```cpp
void APushGameMode::Tick(float DeltaSeconds)
{
	Super::Tick(DeltaSeconds);
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
					SetActorTickEnabled(false);
					CurrentTime = 0.0f;
					return;
				}

				SetMatchState(MatchState::Result); // 결과발표
			}
			else
			{
				if (Ring.Get() != nullptr)
					Ring.Get()->Shrink(RingRadius[Round], 10);

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
			Ring->Reset();
			SetMatchState(MatchState::InProgress);
		}
	}
}
```

기존에 수많은 변수를 통해 시간 계산을 하는 과정에서, temptime을 지속적으로 갱신해 현재 MatchState로 들어온 후 지난 시간 = GetWorld()->GetTimeSeconds() - tempTime으로 계산하고   
이를 정해진 시간 CountdownTime에서 뺀 값 CurrentTime이 0보다 작아지는 순간 시간이 모두 지난 것으로 판별한다.

RoundTime 배열 추가, 직렬화를 통해 지정된 라운드 수와 시간만큼 라운드를 반복하며 라운드 별 지정된 자기장 크기만큼 자기장을 줄이며, 모든 라운드가 종료되면 자기장을 원 상태로 돌린다.


**Kill, Death, Gold**

킬, 데스, 골드는 기존 UI의 위에 UTextBlock을 추가하고, C++ 에서 BindWidget을 통해 코드로 내용을 변경하면 위젯에 반영되도록 설정.

![](/assets/images/Unreal/팀프KDA위젯.PNG)

![](/assets/images/Unreal/ResourceKDG.PNG)
*WDG_Resource의 Text*

```cpp
//킬, 데스, 골드를 적용하는 변수
UPROPERTY(meta = (BindWidget))
	class UTextBlock* GoldText;
UPROPERTY(meta = (BindWidget))
	class UTextBlock* KillText;
UPROPERTY(meta = (BindWidget))
	class UTextBlock* DeathText;
```

킬, 데스, 골드가 조정될 때 마다 적용될 수 있도록 ResourceComponent의 킬 데스 골드 조정 함수에 추가

NetMulticast를 통해 골드를 모든 클라이언트에서 확인할 수 있도록 조정 하며, SetText를 통해 플레이어의 골드 값을 동기화

```cpp
//ResourceComponent의 골드 조정 파트
void UResourceComponent::AdjustGold_NMC_Implementation(int InValue)
{
	// 골드제한 할거면 Clamp 추가
	Gold += InValue;

	// 위젯 업데이트
	if (PushGameMode)
		PushGameMode->UpdatePlayerList();

	TWeakObjectPtr<APushPlayerController> playerController = Cast<APushPlayerController>(Owner->GetController());

	if (false == playerController.IsValid()) return;

	MainHUD = MainHUD == nullptr ? Cast<AMainHUD>(playerController->GetHUD()) : MainHUD;

	if (false == IsValid(MainHUD)) return;

	if(MainHUD->CheckWidget("Resource"))
	{
		FText gold = FText::FromString(FString::FromInt(Gold));
		MainHUD->GetWidget<UResource>("Resource")->GoldText->SetText(gold);
	}

}
```
