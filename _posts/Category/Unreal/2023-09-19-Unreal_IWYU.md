---
layout: single
title: "Unreal IWYU"
categories: Unreal
classes: wide
---

### Unreal IWYU

Unreal의 IWYU는 Include What You Use의 줄임말로 엔진의 소스 코드는 컴파일에 필요한 종속성만을 포함하라는 의미를 갖고 있습니다.

기존의 언리얼은 Engine.h 또는 UnrealEd.h의 모놀리식 헤더 파일을 포함하여 사용하며, 컴파일 시간을 빠르게 하기 위해 PCH(PreCompiled Header) 파일에 의존하였습니다.

모놀리식 헤더 파일은 병목 현상을 일으키며, 이는 컴파일 시간을 늘리는 결과를 갖기 때문에 언리얼에서는 IWYU를 추가하여 컴파일 시간을 단축시켰습니다.

**IWYU 4가지 규칙**

1. 모든 헤더 파일은 필수 종속성을 포함합니다.

2. .cpp파일은 일치하는 *.h 파일을 먼저 포함합니다.

3. PCH 파일은 더 이상 명시적으로 포함되지 않습니다.

4. 모놀리식 헤더 파일은 더 이상 포함되지 않습니다.


언리얼은 이러한 규칙들을 CoreMinimal.h의 추가와 UBT를 통해 실행하였으며, build.cs에서 추가적인 조작을 통해 IWYU모드의 실행여부와 PCH파일의 모드를 설정할 수 있습니다.

언리얼의 클래스 템플릿을 확인하면 .h 파일의 첫 포함되는 파일은 CoreMinimal.h, .cpp 파일의 첫 포함되는 파일은 *.h 파일이 적혀있음을 알 수 있습니다.

```cpp
//ActorClass.h의 템플릿
#pragma once

#include "CoreMinimal.h"
%BASE_CLASS_INCLUDE_DIRECTIVE%
#include "%UNPREFIXED_CLASS_NAME%.generated.h"

UCLASS(%UCLASS_SPECIFIER_LIST%)
class %CLASS_MODULE_API_MACRO%%PREFIXED_CLASS_NAME% : public %PREFIXED_BASE_CLASS_NAME%
{
	GENERATED_BODY()

public:
	%PREFIXED_CLASS_NAME%();

protected:
	virtual void BeginPlay() override;

public:
	virtual void Tick(float DeltaTime) override;

	%CLASS_FUNCTION_DECLARATIONS%
	%CLASS_PROPERTIES%
};


//ActorClass.cpp의 템플릿
%MY_HEADER_INCLUDE_DIRECTIVE%

%PREFIXED_CLASS_NAME%::%PREFIXED_CLASS_NAME%()
{
	PrimaryActorTick.bCanEverTick = true;

}


void %PREFIXED_CLASS_NAME%::BeginPlay()
{
	Super::BeginPlay();
	%CURSORFOCUSLOCATION%
}

void %PREFIXED_CLASS_NAME%::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

}

%ADDITIONAL_MEMBER_DEFINITIONS%
```
