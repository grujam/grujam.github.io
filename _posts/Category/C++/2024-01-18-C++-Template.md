---
layout: single
title: "템플릿 (Template)"
categories: C++
classes: wide
---

## 템플릿 (Template)

C++에서는 재사용성을 확보하기 위해 제네릭 프로그래밍(Generic Programming) 이라는 것을 지원하며, 이를 위한 대표적인 도구가 바로 템플릿입니다.

객체지향 기법의 일부는 아니지만 객체지향과 함께 강력한 기능을 발휘하기 때문에 사용하면 좋지만, 높은 난이도로 사용되기 어렵습니다.

하지만 라이브러리 류의 코드는 유동성을 위해 템플릿을 필수적으로 사용하며, 이러한 코딩을 위해 반드시 알아야둬야할 내용입니다.


### 클래스 템플릿

클래스 템플릿은 클래스 생성 시 멤버 변수 타입, 메서드 매개변수, 리턴 타입 등을 매개 변수로 받는 클래스입니다.

이러한 클래스 템플릿은 컨테이너에 많이 사용되며, 예시로 C++ STL에서 제공되는 표준 컨테이너들을 보면 대부분 템플릿을 통해 저장할 변수의 타입을 받고 있음을 확인할 수 있습니다.

*템플릿 선언 예시*

사용된 optional은 c++17부터 추가된 기능으로 값 또는 포인터 전달 방식을 모두 사용할 수 있도록 해주는 기능입니다.

```cpp
//다양한 타입을 벡터에 저장하는 컨테이너 클래스
template <typename T>
class Container
{
public:
	Container();

private:
	vector<std::optional<T>> vec;
};
```

### 템플릿 인스턴스화

작성된 템플릿을 아래와 같이 선언하여 사용하는 것을 템플릿 인스턴스화라고 합니다.

템플릿을 인스턴스화 할때는 Container cont나 Container<> cont와 같이 타입을 명시해주지 않는다면 오류가 발생하게 됨으로 주의해야 합니다.

```cpp
int main()
{
	Container<int> cont;

	return 0;
}
```


### 컴파일러와 템플릿

템플릿을 이해하고 원활히 사용하기 위해서는 다른 클래스와는 다른 컴파일러와 템플릿의 관계를 알아야 합니다.

다른 코드와 달리 컴파일러는 컴파일 타임 때 템플릿 코드를 마주친다면 **문법 검사만 진행하고 실제로 코드를 컴파일하지 않습니다.**   
왜냐하면 어떤 타입이 들어가는지 모르기 때문입니다.

그렇기에 템플릿을 사용하는 코드를 마주친 후에야 코드를 컴파일하게 됩니다.

이 때, 컴파일러가 코드를 컴파일 하는 방법은 이미 작성된 코드가 있는지 확인하며, 없다면 템플릿 코드의 **T를 입력받은 타입으로 모두 바꾸어 새로운 코드를 작성하여** 그 내용을 토대로 사용하게 됩니다.

그러므로 실제로 컴파일러가 하는 작업은 **복사 붙여넣기와 단어 바꾸기**가 전부입니다.


### 클래스 템플릿 코드 분할

클래스 템플릿을 작성하다보면 반드시 마주치게 되는 오류가 있습니다.

다른 메서드와 같이 템플릿 메서드의 선언 부분만 헤더에 작성하고 본문을 cpp에 작성한다면, 오류가 발생하게 됩니다.

*왜 이런 오류가 발생하나?* → 템플릿은 해당 메서드를 사용할 때 마다 선언과 본문을 보고 이를 토대로 코드를 작성해야 하기 때문에 본문을 찾지 못해 오류가 발생합니다.

*해결방법?*

이러한 오류의 해결 방법은 3가지가 있습니다.

1. 헤더에 본문을 작성한다: 가장 흔한 방법으로 템플릿 메서드를 정의할 때 본문도 헤더 파일에 작성한다면 오류가 발생하지 않습니다.

2. 헤더에 cpp를 include한다: 헤더에서 cpp 파일을 include 하면 오류는 발생하지 않지만 해당 cpp 파일이 빌드 목록에 추가되지 않도록 주의하여야 합니다.

3. hpp 파일 형식을 사용한다: 헤더와 cpp 파일을 묶어서 사용하는 hpp라는 파일 형식을 사용하면 문제를 해결할 수 있지만, 자주 사용되는 방법은 아닙니다.


### 클래스 템플릿 인스턴스화 제한

만약 특정 타입으로만 인스턴스화 될 수 있도록 제한하고 싶은 경우에는 이를 명시적으로 표기해주어야 합니다.

cpp 파일에서 해당 클래스의 생성자를 템플릿 형식으로 작성한 후, 파일의 마지막에 인스턴스화를 허락할 타입을 작성한다면 해당 타입으로만 인스턴스화 가능하게 됩니다.

아래 예시에서는 Container 클래스 템플릿은 int, float, queue<int>의 타입으로만 인스턴스화 되도록 제한해둔 코드입니다.

```cpp
//Container.h
template <typename T>
class Container
{
	//구현 내용 생략
};
```

```cpp
//Container.cpp
template <typename T>
Container<T>::Container()
{

}

template class Container<int>;
template class Container<float>;
template class Container<std::queue<int>>;
```



*참조: 마크 그레고리, "전문가를 위한 C++"*
