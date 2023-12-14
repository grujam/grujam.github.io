---
layout: single
title: "Unreal Engine - Local Network Connections"
categories: Unreal
classes: wide
---

### Unreal Engine - Local Network Connections

Unreal에서 동일 네트워크에 존재하는 컴퓨터를 서버로 오픈하고 이에 클라이언트로서 접속하는 방법입니다.

첫 번째로 언리얼 프로젝트를 기본 TPS 프로젝트로 생성합니다.

언리얼 에서는 언리얼 UE4Editor.exe와 생성한 프로젝트의 uproject 파일의 경로를 가져와 cmd에 작성하여 실행할 수 있습니다.

UE4Editor.exe: "C:\Program Files\Epic Games\UE_4.26\Engine\Binaries\Win64\UE4Editor.exe"    
uproject: "C:\Unreal\PuzzlePlatforms\PuzzlePlatforms.uproject"    
이 끝에 -game을 작성하고 실행하면 Standalone 모드로 프로젝트를 실행할 수 있습니다.

![](/assets/images/Unreal/Local-game.PNG)
*cmd 화면*

![](/assets/images/Unreal/Local-GameOpen.PNG)
*Standalone 모드로 실행 된 프로젝트*

-log를 사용하면 프로젝트가 실행되는 log창을 확인할 수 있으며, Map의 경로를 붙여 특정 Map을 오픈할 수도 있습니다.   
![](/assets/images/Unreal/Local-Log.PNG)
*-log로 실행된 Log화면*

로컬 네트워크를 통해 서버를 열고 클라이언트를 연결하는 방법은 다음과 같습니다.

1. 위와 동일하게 실행하며 열고 싶은 맵의 경로와 이름을 적고 -server와 -log를 추가하여 cmd를 실행하면 이전과 같이 Unreal 프로젝트는 켜지지 않지만 로그만 켜지는 것을 알 수 있습니다.

2. ipconfig를 통해 본인 서버 컴퓨터의 ip를 확인합니다. (IPv4주소를 사용하여야 합니다)

3. 1번에서 실행했던 code에 맵의 경로, -server, -log를 모두 지우고 ip 주소와 -game을 실행하면 이번에는 Standalone 형태로 Unreal 프로젝트가 켜지며, 맵에 서버 컴퓨터의 더미 하나가 있는 것을 확인할 수 있습니다.

이 방식으로 여러 개의 클라이언트가 서버에 접속할 수 있습니다.

![](/assets/images/Unreal/Local-Client1.PNG)
*접속한 클라이언트1*

![](/assets/images/Unreal/Local-Client2.PNG)
*접속한 클라이언트2*
