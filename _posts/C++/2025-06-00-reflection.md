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

이번 C++26 표준에서는 컴파일 타임 리플렉션(Reflection)이 도입될 예정입니다.

> 리플렉션(Reflection)이란?
> 리플렉션은 프로그램이 자신의 구조를 검사하고 수정할 수 있는 능력을 말합니다.

리플렉션을 사용하면 클래스, 함수, 변수 등의 정보를 컴파일 타임에 얻을 수 있으며, 이를 통해 메타프로그래밍을 보다 쉽게 구현할 수 있습니다.

## 새로 추가된 문법

C++26에서 리플렉션을 위해 다음과 같이 새로운 문법이 추가되었습니다.

### 1. 리플렉션 연산자: `^^`

리플렉션 연산자 `^^`는 표현식 앞에 사용되어 해당 클래스나 함수, 변수 등의 정보를 얻기 위해 사용됩니다. 이렇게 변환된 정보는 `std::meta::info` 타입으로 표현됩니다.

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

### 2. 스플라이서: `[::]`

스플라이서 `[::]`는 리플렉션 연산자와는 반대로, `std::meta::info` 객체를 다시 원본 표현식으로 변환하는데 사용됩니다.

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

위 코드에서 `typename[:int_refl:]`와 같이 사용하여 리플렉션 정보를 원본 타입으로 변환할 수 있습니다. 이때 `typename` 키워드를 사용하여 `int_refl`이 타입임을 명시적으로 지정합니다.
