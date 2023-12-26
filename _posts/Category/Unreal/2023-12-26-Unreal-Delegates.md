---
layout: single
title: "Unreal Engine - Delegates"
categories: Unreal
classes: wide
---

### Unreal Engine - Delegates

Delegate는 C++ 오브젝트 상의 멤버 함수를 가리키고 실행시키는 데이터 유형입니다.

멤버 함수를 바인딩하여 사용하기 때문에 오브젝트의 유형을 알 필요 없이 멤버 함수만을 실행할 수 있으며

복사해도 안전하기 때문에 값 전달이 가능하지만 메모리를 heap에 할당하기 때문에 메모리 할당과 해제에 시간이 걸리므로 가급적이면 참조 전달하는 것이 좋습니다.

**델리 게이트 종류**

- Single-Cast: 하나의 함수에 연동하여 호출하는 델리게이트입니다   
- Multi-Cast: 여러 함수에 연동하여 호출하는 델리게이트입니다   
- Event: 여러 함수에 연동하여 호출하지만 접근권이 제한되어있는 델리게이트입니다   
- Dynamic: Single 또는 Multi 캐스트가 가능한 직렬화를 지원하는 델리게이트입니다


### Delegate 선언

Delegate 선언은 기본적으로 DECLARE_로 시작하며, Single, Multi, Event, Dynamic, 매개변수, 반환값 등에 따라 선언 방법이 바뀝니다.

*Example*
> DECLARE_DELEGATE: Single-Cast 델리게이트   
> DECLARE_MULTICAST_DELEGATE: Multi-Cast 델리게이트   
> DECLARE_DYNAMIC_DELEGATE: Dynamic 델리게이트   
> DECLARE_DELEGATE_OneParam: 한개의 매개변수를 갖는 델리게이트   
> DECLARE_DELEGATE_RetVal_TwoParams: 반환값과 두 개의 매개변수를 갖는 델리게이트   


### Delegate 호출

Single-Cast의 경우 Bind의 이름을 갖는 여러 함수를 통해 함수를 지정하며, Multi-Cast의 경우 Add의 이름을 갖는 함수를 통해 함수를 묶습니다.

연동된 함수들은 Execute()를 통해 실행되거나, IsBound()를 통해 묶임 여부를 확인하거나, ExecuteIfBound()를 통해 묶였다면 바로 실행하게 할 수 있습니다.


**Payload**

Dynamic 델리게이트를 제외한 다른 델리게이트는 Payload라는 기능을 지원합니다.

Payload는 바인딩 된 함수를 호출할 때 불러지는 임의의 데이터로 델리게이트 자체적으로 데이터를 저장하며, 최대 4개까지 지정할 수 있습니다.


**SPARSE**

SPARSE 문구는 델리게이트를 선언할 때 마지막에 들어가는 문구로 잘 사용되지 않는 델리게이트에게 메모리 최적화 기능을 제공합니다.

> DECLARE_MULTICAST_SPARSE_DELEGATE_ThreeParams

이는 잘 사용되지 않는 Delegate를 굳이 바인딩하지 않고 1 byte 공간을 할당하여 저장한 후, 바인딩 된 경우 Sparse 델리게이트가 저장된 곳을 찾아 가져와 사용하는 방식입니다.

자주 호출되지 않는 델리게이트를 적은 메모리를 통해 저장하기 때문에 메모리를 아낄 수 있지만, 그만큼 바인딩과 호출에 추가적인 과정을 요구하기에 일반적인 델리게이트보다 느립니다.
