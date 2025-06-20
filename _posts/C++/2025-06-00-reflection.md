---
title: "C++26 컴파일 타임 리플렉션(Reflection)"

categories:
  - C++
tags:
  - [c++, c++26, reflection]

toc: true
toc_sticky: true

date: 2025-06-20
last_modified_at: 2025-06-20
---

## C++26 컴파일 타임 리플렉션(Reflection)

이번 C++26 표준에서는 많은 C++ 개발자들이 오랫동안 기다려온 기능인 컴파일 타임 리플렉션(Reflection)이 도입될 예정입니다.

> 리플렉션(Reflection)이란?
> 리플렉션은 프로그램이 자신의 구조를 검사하고 수정할 수 있는 능력을 말합니다.

지금까지는 C++에서 리플렉션을 구현하기 위해 매크로, 템플릿 메타프로그래밍, 외부 라이브러리 등을 사용해야 했지만, C++26에서부터 리플렉션을 위한 공식적인 문법이 도입되어, 보다 직관적이고 간편하게 리플렉션 기능을 사용할 수 있게 되었습니다.

## 새로 추가된 문법

이번 C++26에서는 리플렉션을 위해 다음과 같이 새로운 문법이 추가되었습니다.

### 1. 리플렉션 연산자: `^^`

리플렉션 연산자 `^^`는 표현식 앞에 붙어 해당 클래스나 함수, 변수 등의 구조 정보를 추출하는 데 사용됩니다. 이렇게 변환된 정보는 `std::meta::info`라는 특별한 타입으로 표현됩니다.

```cpp
#include <meta>

namespace MyNamespace {}
class MyClass {};

int main() {
    // int 타입에 대한 리플렉션 정보 가져오기
    auto int_refl = ^^int;

    // MyNamespace 네임스페이스에 대한 리플렉션 정보 가져오기
    constexpr auto namespace_refl = ^^MyNamespace;

    // MyClass 타입에 대한 리플렉션 정보 가져오기
    std::meta::info my_class_refl = ^^MyClass;
}
```

### 2. 스플라이서: `[: :]`

스플라이서 `[: :]`는 리플렉션 연산자와는 반대로, `std::meta::info` 객체가 담고 있는 정보를 다시 원래의 코드 요소(타입, 변수 등)로 되돌리는 역할을 합니다.

```cpp
namespace MyNamespace {
    class TestClass {};
}

class MyClass {};

int main() {
    constexpr auto int_refl = ^^int;
    constexpr auto ns_refl = ^^MyNamespace;
    constexpr auto my_class_refl = ^^MyClass;

    // 리플렉션 정보를 원본 타입으로 변환
    typename[:int_refl:] int_var = 42;
    [:ns_refl:]::TestClass namespace_var{};
    typename[:my_class_refl:] my_class_var{};
}
```

위 코드에서처럼 리플렉션 정보를 다시 타입으로 사용하기 위해 `typename[:int_refl:]` 형태로 쓸 수 있습니다. 이때 `typename` 키워드는 스플라이서가 변환할 결과물이 타입이라는 것을 컴파일러에게 명시적으로 알려주는 역할을 합니다.
