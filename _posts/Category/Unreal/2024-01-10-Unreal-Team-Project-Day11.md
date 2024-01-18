---
layout: single
title: "Unreal Team Project - Day 11"
categories: Unreal
classes: wide
---

# Day 11 : Character Color Replication

## 2024.01.09 Wednesday Scrum Log

### Daily Review

- MH → Skill Widget Drag & Drop 성공, WDG_Main에 모든 위젯 통일

- IS → Meteor Skill Replication 테스트 중

- SW → 특정 인원 수 입장 시 새 레벨로 이동 성공, 이동 전 라운드 시간 동기화 실패

- HJ → 버프 쿨 타임 컴포넌트 통일 관리에서 각 버프 객체로 변경, Resource Component 내 RPC 함수 추가

- 작성자 → 클라이언트 별 색깔 변경 및 Replication 실패


### Works to do

- MH → Skill Slot과 SkillData 바인딩

- IS → Meteor Skill Replication, Player 번호 받아와 KDA UI에 적용

- SW → 라운드 시간 동기화

- HJ → Resource Widget 버그 수정, 다른 클라이언트에게 디버프 부여

- 작성자 → 클라이언트 별 색깔 변경 및 Replication


## Daily 작업 과정

**캐릭터 색깔 부여2**

소 뒷걸음치다 쥐 잡은격으로 문제 해결

GameMode의 PostLogin하나로 해결이 가능한 문제였다.

GameMode에서 PostLogin을 통해 controller의 접속이 완료될 때, 해당 controller는 이미 Character를 지정받았으며, 이 캐릭터의 색깔을 바꿔주면 모든 클라이언트에서 Replicate되어 동일한 색상이 되는 것을 확인할 수 있었다.

이유? → 아마 Postlogin에서 서버가 플레이어의 Controller와 Character를 생성해어 할당해주면, 다른 클라이언트에서 이 캐릭터의 복사본을 생성하는 것이라 이런 결과가 나오는 것 같다.

추후 게임 시작 전 플레이어 별 Replicate되어야 하는 세팅이 존재한다면 PostLogin을 적극적으로 활용하도록 하겠다.

```cpp
//PushGameMode.cpp의 PostLogin
void APushGameMode::PostLogin(APlayerController* NewPlayer)
{
	Super::PostLogin(NewPlayer);

	APushPlayerController* controller = Cast<APushPlayerController>(NewPlayer);

	if (controller == nullptr)
		return;

	APushCharacter* character = Cast<APushCharacter>(controller->GetPawn());

	if (character == nullptr)
    return;

  //그저 서버에서 색깔을 지정해주기만 하면 된다.
	if(index >= Colors.Num() - 1)
		character->BodyColor = FLinearColor::Black;
	else
		character->BodyColor = Colors[index++];
}
```
