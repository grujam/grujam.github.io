---
layout: single
title: "Unreal Team Project - Day 13"
categories: Unreal
classes: wide
---

# Day 13 : Timer 동기화

## 2024.01.12 Friday Scrum Log

### Daily Review

- MH → 아이템 사용 및 구매 진행 중

- IS → 추가 적인 스킬 구현

- SW → 블루프린트로 라운드 동기화 성공, C++ 진행 중

- HJ → 아이템 효과 리플리케이션 성공

- 작성자 → GameMode and Gamestate 학습 중


### Works to do

- MH → 아이템 및 스킬 구매, 사용 진행

- IS → 오류나는 Branch 해결 및 KDA UI 작업

- SW → 라운드 시간 동기화

- HJ → 게임이 실제로 플레이 될 맵 탐색

- 작성자 → 특정 인원 수의 플레이어가 접속 후 레벨 옮길 때 타이머 및 인원 수 모든 클라이언트에 동기화


## Daily 작업 과정

로비맵에서 인원 수와 타이머를 동기화? → GameMode와 GameState와 HUD를 통해 Controller와 소통하여 해결

- 로비맵에서 접속한 인원 수를 파악하는 기능은 GameMode에 존재

- 각 플레이어의 HUD는 Controller가 보유

- 모든 Player의 Controller에 접근하기 위해선 GameState의 PlayerArray를 통해 가능

이를 통해 GameMode의 PostLogin으로 플레이어가 접속한 경우, 접속한 인원 수를 늘리고 GameState에게 PlayerArray에 있는 모든 플레이어의 Controller의 HUD에 접근하여 Widget의 내용을 바꾸어주면 동기화 할 수 있다.

**GameMode**

LobbyGameMode 클래스는 PostLogin을 통해 플레이어 접속 시 마다 인원 수를 늘리고, 인원 수를 GameState에게 전달한다.

인원 수가 최대치라면 타이머를 시작하고 1초마다 카운트다운을 GameState에게 전달한다.

```cpp
//LobbyGameMode.cpp

//플레이어 접속 시
void ALobbyGameMode::PostLogin(APlayerController* NewPlayer)
{
	Super::PostLogin(NewPlayer);

	NumOfPlayers++;

	if (false == IsValid(NewPlayer)) return;

	if (HasAuthority())
	{
		TWeakObjectPtr<UGameInstance> GameInstance = GetGameInstance();
		if (GameInstance.IsValid())
		{
			if (NumOfPlayers >= MaxNumofPlayers)
			{
				CLog::Log(NumOfPlayers);
				countdownTimer = StartCountdown;
				GetWorld()->GetTimerManager().SetTimer(LobbyTimeHandle, this, &ALobbyGameMode::CountDown, 1.0f, true, 0);
			}
		}
	}

	ALobbyGameState* lobbyGameState = GetGameState<ALobbyGameState>();

	if (lobbyGameState == nullptr)
	{
		CLog::Log("GameState Nullptr");
		return;
	}

	lobbyGameState->PlayerConnection(NumOfPlayers);
}

//플레이어 수가 최대인 경우 카운트다운 시작
void ALobbyGameMode::CountDown()
{
	ALobbyGameState* lobbyGameState = GetGameState<ALobbyGameState>();

	if (lobbyGameState != nullptr)
	{
		lobbyGameState->CountDown(countdownTimer);
	}

	countdownTimer--;

	if (countdownTimer < 0)
		EnterMap();
}
```

**GameState**

게임 스테이트는 GameMode에게서 전달 받은 플레이어 접속 또는 카운트다운을 PlayerArray를 통해 모든 플레이어의 Controller에게 플레이어 수와 카운트 다운을 동기화하는 역할을 한다.

```cpp
//LobbyGameState.cpp

//플레이어 접속 시 게임 모드에게서 전달받은 후 모든 Controller의 Client에게 인원 수 업데이트
void ALobbyGameState::PlayerConnection(int InPlayerNum)
{
	NumofPlayers = InPlayerNum;
	for (APlayerState* player : PlayerArray)
	{
		ALobbyPlayerController* controller = Cast<ALobbyPlayerController>(player->GetOwner());

		if (controller == nullptr)
		{
			CLog::Log("No Controller");
			continue;
		}

		controller->UpdatePlayerNum_Client(InPlayerNum);
	}
}

//GameMode에게 전달받는 Countdown을 위와 동일하게 Controller들에게 업데이트
void ALobbyGameState::CountDown(int InCountdown)
{
	Count = InCountdown;
	for(APlayerState* player : PlayerArray)
	{
		ALobbyPlayerController* controller = Cast<ALobbyPlayerController>(player->GetOwner());

		if(controller == nullptr)
		{
			CLog::Log("No Controller");
			continue;
		}

		controller->UpdateTimer_Client(InCountdown);
	}
}
```


**Controller**

입력받은 카운트다운 또는 인원 수를 HUD에 보내주는 역할을 한다.

```cpp
void ALobbyPlayerController::UpdateTimer_Client_Implementation(int InTime)
{
	ALobbyHUD* LobbyHUD = Cast<ALobbyHUD>(GetHUD());


	if (LobbyHUD->CheckWidget("LobbyCountDown"))
	{
		LobbyHUD->GetWidget<ULobbyCountDown>("LobbyCountDown")->SetCountdown(InTime);
	}
}

void ALobbyPlayerController::UpdatePlayerNum_Client_Implementation(int InNum)
{
	ALobbyHUD* LobbyHUD = Cast<ALobbyHUD>(GetHUD());

	if(LobbyHUD == nullptr)
	{
		CLog::Log("HUD invalid");
		return;
	}

	if (LobbyHUD->CheckWidget("LobbyCountDown"))
	{
		LobbyHUD->GetWidget<ULobbyCountDown>("LobbyCountDown")->SetPlayerNum(InNum);
	}
	else
	{
		CLog::Log("No Widget");
	}
}

```
