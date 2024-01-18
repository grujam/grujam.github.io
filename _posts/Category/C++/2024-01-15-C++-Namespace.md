---
layout: single
title: "네임 스페이스 (namespace)"
categories: C++
classes: wide
---

## 네임 스페이스 (namespace)

네임 스페이스는 이름 충돌 해결을 위해 등장한 방법입니다.

외부 라이브러리를 추가하거나 다른 클래스를 참조하였는데 동일한 변수 이름이나 함수 이름이 있는 경우, 컴파일러는 어느 것을 가리키는지 알지 못해 모호함 문제가 발생합니다.

이를 해결하기 위해 아래와 같이 네임 스페이스를 통해 어디에 속해있는지 컴파일러가 파악하게 할 수 있으며, 구현 부분도 네임 스페이스를 사용하여 묶을 수 있습니다.

(구현 부분을 네임 스페이스에 묶는 것은 클래스의 멤버 메서드를 작성할 때 볼 수 있습니다)

```cpp
//namespace Test안에 있는 foo함수
namespace Test{
	void foo();
}

//Test의 foo 구현 부
void Test::foo()
{
	std::cout << "Hello, World!";
}
```

위에 사용된 std도 동일하게 std라는 네임 스페이스를 사용하여 cout을 호출한 것입니다.

이러한 네임 스페이스는 using을 통해 네임 스페이스 선언을 생략할 수 있지만, using을 남발한다면 네임 스페이스의 사용 의의를 잃어버리게 되기 때문에 주의하여야 합니다.

```cpp
#include <iostream>

using namespace std;

int main()
{
	//using namespace std를 통해 std를 생략
	cout << "Hello, World";

	return 0;
}
```

**중첩 네임 스페이스**

중첩 네임 스페이스는 여러 네임 스페이스를 중첩하여 선언하는 것을 말합니다.

기존 네임 스페이스는 하나씩 나열하여야 했지만, c++ 17부터는 이를 더블 콜론을 사용해 선언할 수 있습니다.

```cpp
//c++ 17이전
namespace A
{
	namespace B
	{
		namespace C
		{
			void foo();
		}
	}
}

//c++ 17 이후
namespace A::B::C
{
	void foo();
}
```

**네임 스페이스 앨라이어스 (namespace alias)**

alias는 별명을 뜻하는 말로 네임 스페이스가 너무 긴 경우 이를 다른 이름으로 쓰기 위해 사용하는 문법을 말합니다.

```cpp
//이름이 너무 길다
namespace iamverylong::toolongtowritedown::stopgettinglonger
{

}

//다른 이름으로 사용할 수 있다
namespace longname = iamverylong::toolongtowritedown::stopgettinglonger;
```

*참조: 마크 그레고리, "전문가를 위한 C++"*
