---
layout: single
title: "Unreal Team Project - Day 12"
categories: Unreal
classes: wide
---

# Day 12 : GameMode and GameState

## 2024.01.11 Thursday Scrum Log

### Daily Review

- MH → Skill Slot에 키 바인딩 완료

- IS → Meteor Skill Replication 성공

- SW → 블루프린트로 라운드 동기화 진행 중

- HJ → 아이템 효과 리플리케이션 진행 중

- 작성자 → 클라이언트 별 캐릭터 Replication 성공


### Works to do

- MH → 아이템 항목 추가 및 구매 기능 추가

- IS → 휴가

- SW → 라운드 시간 동기화

- HJ → 아이템 효과 리플리케이션 그대로 진행

- 작성자 → 개인 면접 준비 및 GameMode GameState 작업 참여


## Daily 작업 과정

Day 12에서는 GameMode와 GameState 작업을 진행하기 위해 언리얼에서 제공하는 GameMode와 GameState에 대한 지식 학습을 중점으로 하였다.


**GameMode**

GameMode는 말 그대로 현재 맵의 모드를 설정하는 클래스이다.

게임 모드는 서버에만 존재하고 클라이언트에게 Replicate 되지 않기 때문에 GameMode는 서버에서만 작동하며 NetMulticast등의 함수로 클라이언트에게 호출해달라는 방법은 할 수 없다.

게임 모드를 통해 가능한 기능들은 다음 등이 존재한다.

- 플레이어 Login시 해줘야할 초기 세팅

- 현재 플레이어의 수 확인, 게임 시작 플레이어 수 만족 시 시작

- MatchState 변경을 통한 게임 상태 조정

- 클라이언트를 데리고 레벨 변경

- 매치 종료

이외에도 많은 기능이 있지만 아직 사용해보지 못한 관계로 적지 않습니다.


**GameState**

GameState는 게임의 현재 상태를 보유하는 클래스이다.

게임 스테이트는 모드와 달리 서버에서 생성되어 모든 클라이언트에게 Replicate되는 클래스이기 때문에, GameMode를 통해 게임 상태를 변경하면 GameState를 통해 이를 클라이언트에게 적용하는 방식으로 작동된다.

GameState는 Actor이기 때문에 RPC호출도 가능하며, 내부에 Replicate되는 변수를 넣어 모든 클라이언트에 동기화할 수 있다.

GameState가 하는 기능들은 다음과 같다.

- 매치 상태 변경에 대한 호출 함수

- 현재 매치 상태(종료, 진행, 남은 시간 등)를 확인
