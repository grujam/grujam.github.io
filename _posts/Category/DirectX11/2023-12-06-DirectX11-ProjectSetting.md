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

![속성관리자](/assets/images/DirectX/속성관리자.PNG "속성관리자")
*속성 관리자*

![사용자매크로](/assets/images/DirectX/사용자매크로.PNG "사용자 매크로")

생성된 프로젝트의 속성에서 DXH와 DXL을 통해 DirectX와 Header와 Library를 포함합니다.

![Framework속성페이지](/assets/images/DirectX/Framework속성페이지.PNG "속성 페이지")

DirectX 11 Texture와 IMGUI등을 사용할 수 있게 해주는 Library를 프로젝트의 폴더 옆에 _Libraries폴더로 생성하여 저장하고 프로젝트 라이브러리에 포함합니다.

그 외에 프로젝트의 빠른 빌드와 편의성을 위해 다중 프로세서 컴파일, 미리 컴파일된 헤더를 설정하고, 준수 모드를 끕니다.

![Framework속성C++](/assets/images/DirectX/Framework속성C++.PNG "C/C++ 속성 설정")

![Framework속성준수모드](/assets/images/DirectX/Framework속성준수모드.PNG "준수모드")

![Framework속성미리컴파일된헤더](/assets/images/DirectX/Framework속성미리컴파일된헤더.PNG "미리 컴파일된 헤더")

![Framework속성라이브러리관리자](/assets/images/DirectX/Framework속성라이브러리관리자.PNG "라이브러리 관리자")
