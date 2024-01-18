---
layout: single
title: "Unreal Team Project - Day 10"
categories: Unreal
classes: wide
---

# Day 10 : Hit Launch Replication

## 2024.01.09 Tuesday Scrum Log

### Daily Review

- MH → Store UI에서 Skill 생성하게 변경, MainHUD에 Store UI추가

- IS → 클라이언트 색 변경 Replication 실패, ABP 작성

- SW → GameMode 버그 해결

- HJ → 버프들 UObject 타입에서 AActor로 변경, 위젯 제작 중

- 작성자 → 피격 시 Launch 및 Replication 적용


### Works to do

- MH → Shop Component Drag & Drop

- IS → KDA UI 및 ABP 버그 수정

- SW → 클라 종료 시 게임 초기화

- HJ → 휴가

- 작성자 → 캐릭터 색깔 부여 및 Replication

## Daily 작업 과정

**캐릭터 색깔 부여**

캐릭터 색깔을 어디서 부여해야 하는가? → 어느 한 클래스에서 게임 시작 시 모든 캐릭터에게 각 색깔을 부여하는 것이 랜덤 색을 주는 것 보다 겹치지 않고 좋다고 생각하였음


**시도1:** 게임 모드에서 색깔 배열을 생성하고 이를 GameMode의 PostLogin을 통해 각 Controller에 색깔을 부여하고 캐릭터는 BeginPlay에서 Controller의 색을 가져온다.

결과: 실패

이유: 각자 클라이언트의 캐릭터에선 Controller를 찾아 접근할 수 있지만, 다른 클라이언트의 캐릭터에선 Controller가 존재하지 않아 controller의 색을 받아올 수 없어 색이 변하지 않는다.


**시도2:** 게임 모드에서 Controller에게 색을 부여한 후, Controller의 BeginPlay 때 Controller의 Pawn에 접근하여 색을 부여하는 과정을 Server와 NetMulticast 호출을 통해 모든 클라이언트에 적용한다

결과: 실패

이유: Controller의 BeginPlay에서 Character에 접근되지 않음


**시도3:** Controller에 색을 부여한 후 Character에서 NetMulticast를 통해 Controller에 접근했을 때 Controller가 nullptr이 아니라면 controller의 색을 가져와 다시 NetMulticast로 모든 캐릭터에게 해당 색을 부여한다.

결과: 실패

이유: 많은 Server 콜과 NetMulticast콜로 인해 순서가 엉키고, Controller는 클라이언트와 서버에 존재하기 때문에 두번 호출되며 색이 제대로 바뀌지 않는다.


### 다음 날 다시 시도 예정!!
