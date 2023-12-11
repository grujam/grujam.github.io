---
layout: single
title: "DirectX 11 - Project Setting"
categories: DirectX11
classes: wide
---

### DirectX 다운로드

**DirectX : 게임과 같은 멀티미디어 제작을 위한 API**

DirectX는 Microsoft의 사이트에서 다운 받을 수 있으며, 이 프로젝트는 11 버전으로 제작되었습니다.

11버전의 DirectX를 다운받은 후 프로젝트에서 용이하게 접근할 수 있게 C드라이브의 다른 폴더에 저장하였습니다.

![DirectXFolder](/assets/images/DirectX/DirectXFolder.PNG)


### 프로젝트 생성

Visual Studio 2019버전으로 프로젝트를 생성하였습니다.

프로젝트를 진행하며 다양한 솔루션을 추가하기 위해 프로젝트와 솔루션은 다른 디렉토리를 사용하며

먼저 DirectX 11 프로젝트의 틀을 잡기 위해 Framework 솔루션을 생성합니다.


### 프로젝트 속성

DirectX 11을 사용하기 위해 Header파일과 Library 파일들을 프로젝트 속성에 추가하여야합니다.

이를 매크로로 쉽게 추가하고, 다른 프로젝트에서도 동일한 설정을 사용하기 위해 속성관리자에 개인 PropertySheet을 하나 생성합니다. *Visual Studio 2019 이전 버전에는 사전에 생성이 되어있습니다.*

PropertySheet의 사용자 매크로에 DXH를 DirectX Header경로, DXL을 DirectX Library 경로로 설정합니다.

![속성관리자](/assets/images/DirectX/속성관리자.PNG)
*속성 관리자*

![사용자매크로](/assets/images/DirectX/사용자매크로.PNG "사용자 매크로")
*사용자 매크로*

생성된 프로젝트의 속성에서 DXH와 DXL을 통해 DirectX와 Header와 Library를 포함합니다.

![Framework속성페이지](/assets/images/DirectX/Framework속성페이지.PNG "속성 페이지")
*속성 페이지*

DirectX 11 Texture와 IMGUI등을 사용할 수 있게 해주는 Library를 프로젝트의 폴더 옆에 _Libraries폴더로 생성하여 저장하고 프로젝트 라이브러리에 포함합니다.

그 외에 프로젝트의 빠른 빌드와 편의성을 위해 다중 프로세서 컴파일, 미리 컴파일된 헤더를 설정하고, 준수 모드를 끕니다.

![Framework속성C++](/assets/images/DirectX/Framework속성C++.PNG "C/C++ 속성 설정")
*C/C++ 속성 설정*

![Framework속성준수모드](/assets/images/DirectX/Framework속성준수모드.PNG "준수모드")
*준수모드 설정*

![Framework속성미리컴파일된헤더](/assets/images/DirectX/Framework속성미리컴파일된헤더.PNG "미리 컴파일된 헤더")
*미리 컴파일 된 헤더 설정*

![Framework속성라이브러리관리자](/assets/images/DirectX/Framework속성라이브러리관리자.PNG "라이브러리 관리자")
*라이브러리 관리자 추가*


### Framework 및 UnitTest 설정

DirectX 11의 렌더링, Window 등의 기능들을 구현해둔 파일을 Framework 프로젝트에 추가합니다. 해당 파일들의 구현 내용은 프로젝트를 진행하며 추가 포스트로 설명예정입니다.

이 후 실제 프로젝트의 코드가 진행될 프로젝트 UnitTest를 생성하고 Framework의 속성과 동일하게 Setting 및 빌드 종속성을 통해 Framework가 우선적으로 빌드되게 합니다.

![빌드 종속성](/asset/images/DirectX/빌드종속성.PNG)
*빌드 종속성*


**Debug 파일**

Debug 파일은 프로젝트를 진행하며 많은 용량을 잡아먹기 때문에 지속적인 삭제를 요합니다.

이를 편하게 삭제하기 위해 프로젝트 폴더 최상단에 모든 Debug 파일을 모아놓으려면 속성의 일반에 다음과 같은 명령줄을 추가합니다.

   > $(SolutionDir)Debug_$(ProjectName)\

이 명령어를 통해 프로젝트 폴더의 최상단에는 Debug_Framework 폴더와 Debug_UnitTest 폴더가 생성되며 모든 Debug 내용들은 기존의 Debug폴더와 더불어 최상단의 3개의 폴더에서 관리할 수 있습니다.


**Framework 파일**

UnitTest는 빌드를 위해 Framework.lib와 Framework.pdb 파일을 해당 프로젝트 폴더 내에 필요로 합니다.

Framework.lib와 Framework.pdb는 Framework 빌드 후에 Debug 폴더에 나오는 결과물로, 빌드 후 이벤트를 통해 해당 lib와 pdb파일을 UnitTest폴더로 복사시켜주어야 합니다.

![빌드후이벤트](/assets/images/DirectX/빌드후이벤트.PNG)
*빌드 후 두 파일을 UnitTest폴더로 복사 명령어*

![UnitTest폴더](/assets/images/DirectX/UnitTest폴더.PNG)
*복사 후 UnitTest 폴더*
