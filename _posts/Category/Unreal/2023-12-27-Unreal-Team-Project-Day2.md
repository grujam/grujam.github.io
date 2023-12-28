---
layout: single
title: "Unreal Team Project - Day 2"
categories: Unreal
classes: wide
---

# Day 2 : 엔진 빌드, 기획, 클래스 구조


## 2023.12.27 Wednesday Scrum Log

- 구축된 데디케이티드 서버를 외부 IP에서 접근 가능한지 확인 → 성공. 외부 IP를 통해 연결 가능.

- 소스용 엔진을 통한 런쳐 엔진 리빌드로 패키지 가능한지 확인 → 실패. 각 인원 별 소스용 엔진 환경 구비.

- Source Control 플랫폼 결정 → Git을 통해 Source Control 후 100mb가 넘어가는 파일의 경우 드라이브를 통해 공유. 각 Branch에 작업 후 Test Branch에서 테스트, 오류 없을 시 최종 Main Branch에 적용.

## Daily 작업 과정

### 개발 환경 구축

Unreal 4.26 서버용 소스 엔진으로 개발

Epic Games Github에 존재하는 Unreal 4.26 Source Code를 다운 받아 빌드하여 사용.

*참조: <https://ggjjdiary.tistory.com/57>*

**Comment**

- 빌드에 드는 시간이 오래걸리며 CPU를 많이 잡아먹기 때문에 병렬적인 작업이 불가능하다.

- 다른 인원의 개발 환경 Windows 11과 Visual Studio 2019에서 진행, CPU 점유율이 100%를 찍고 블루스크린 CLOCKED_WATCHDOG_TIMEOUT 발생.   
  → 작업 관리자를 통해 Visual Studio가 사용하는 CPU 갯수를 제한하고 Visual Studio의 도구→설정→빌드 및 실행에서 최대 병렬 프로젝트 수 조절, VC++ 프로젝트 설정에서 최대 동시 C++ 컴파일 수 조절

![](/assets/images/Unreal/빌드및실행.PNG)
*최대 병렬 프로젝트 수 조절을 통해 CPU 과다 사용 제어*

![](/assets/images/Unreal/VC++프로젝트설정.PNG)
*최대 동시 C++ 컴파일 수 = 사용할 CPU 코어 수 → 0이라면 모든 프로세스를 사용하므로 적잘한 값으로 조절*


### 기획

점점 좁아지는 맵에서 다양한 스킬들을 통해 살아남는 게임을 여러 라운드 진행 후 승점 또는 결승전을 통해 최후의 승자 결정.

각 라운드 별 킬과 등수에 비례해 골드 지급, 골드를 통해 스킬 또는 아이템 구매 및 업그레이드.


### 클래스 구조

가능한 기능은 전부 Component로 뺀다.

**현재 정해진 Components**   
- MovementComponent: 객체의 움직임 담당   
- ResourceComponent: 골드 및 체력 등 자원 담당   
- StateComponent: 버프, 디버프, 피격 등 객체의 상태를 담당   

피격 적용은 인터페이스를 통해 데미지를 입을 수 있는 특징을 부여

Skill은 하나의 ASkill 클래스를 Projectile, Area 등의 스킬 타입으로 나누어 구현

추후 아이템 등 다른 구조 결정
