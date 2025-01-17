---
layout: single
title: "Unreal Reflection"
categories: Unreal
classes: wide
---

### Unreal Reflection System
언리얼 리플렉션 시스템, 언리얼의 기능을 구현하는 강력한 기술로 언리얼의 제작사 에픽 게임즈에서는 언리얼을 다음과 같이 설명하고 있다.

> Reflection is the ability of a program to examine itself at runtime.

리플렉션은 런타임에 프로그램이 자가 진단하게 하는 기능이라는 뜻이다.

언리얼은 리플렉션을 통해 직렬화(Serialization), 가비지 콜렉션, 네트워크 리플리케이션, 그리고 **블루프린트와 C++간의 호환성**을 구현한다.

기본적으로 C++는 가비지 콜렉션 기능이 없고, 언리얼에서는 C++에서 작성된 클래스, 구조체, 열거체 등을 파악할 방법이 없으므로 리플렉션을 통해 이를 보완할 수 있다.


#### 언리얼과 C++

언리얼 C++는 **opt-in**방식이다. 어떤 것을 언리얼에서 인식할 지 프로그래머가 골라준다는 의미이다.
프로그래머가 어떠한 클래스, 구조체 등을 언리얼에 직렬화하도록 고른다면, Unreal Header Tool이 컴파일 시 해당 정보를 가져오는 구조를 가지고 있다.

Unreal Header Tool이 어떤 정보를 가져오는지 인식하기 위한 내용은 다음과 같다.

**#include "FileName.generated.h"**

위에 인클루드 되는 헤더는 언리얼 C++을 한번이라도 다뤄본 적이 있는 개발자라면 익숙하다. C++ 클래스 등을 추가할 때 자동적으로 위에 추가되는 헤더이다.

해당 헤더가 인클루드 된 파일은 UENUM(), UCLASS(), UPROPERTY(), USTRUCT() 등의 언리얼 매크로를 사용할 수 있고, UHT가 이를 인식하여 해당 정보들을 파싱하게 된다.

언리얼에서 제공되는 Unreal Reflection을 통한 C++ 클래스 예시:

![ClassAStrategyChar](/assets/images/Unreal/ClassAStrategyChar.PNG)

AStrategyChar는 ACharacter를 상속받은 커스텀 클래스이다.

코드 내용에 UCLASS, UPROPERTY등의 매크로와 매크로 내부에 적히는 메타 정보를 통해 해당 클래스와 멤버들이 언리얼에 어떤 방식으로 보여지고 사용될 지 정해진다.

매크로 내부에 EditDefaultsOnly, BlueprintCallable을 통해 해당 변수가 어느 상황에서 수정될 수 있고, 해당 함수가 블루프린트에서 호출될 수 있는지 여부를 정하는 방식으로 사용된다.

그 외에 Category를 지정하는 등 C++의 내용을 블루프린트에게 알려주는 강력한 기능을 갖고있다.

**한계점**

Unreal Header Tool은 실제로는 C++ 파서가 아니다. C++에서의 모든 요소를 언리얼에서 파싱할 수 없기 때문에 이런 경우 UHT에서 에러가 발생했다는 경고를 보낸다.

예를 들어 ifdef는 C++에서는 문제 없이 동작하지만 언리얼에서는 컴파일 오류가 발생할 가능성이 존재한다.

Unreal C++을 사용할 때에는 안전이 보장된 C++ 요소들을 사용하도록 항상 주의하여야 한다.
