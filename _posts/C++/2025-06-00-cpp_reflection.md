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

## 1. C++26 컴파일 타임 리플렉션(Reflection)

이번 C++26 표준에서는 많은 C++ 개발자들이 오랫동안 기다려온 기능인 컴파일 타임 리플렉션(Reflection)이 도입될 예정입니다.

> 리플렉션(Reflection)이란?
> 리플렉션은 프로그램이 자신의 구조를 검사하고 수정할 수 있는 능력을 말합니다.

지금까지는 C++에서 리플렉션을 구현하기 위해 매크로, 템플릿 메타프로그래밍, 외부 라이브러리 등을 사용해야 했지만, C++26에서부터 리플렉션을 위한 공식적인 문법이 도입되어, 보다 직관적이고 간편하게 리플렉션 기능을 사용할 수 있게 되었습니다.

## 2. 새로 추가된 문법

이번 C++26에서는 리플렉션을 위해 다음과 같이 새로운 문법이 추가되었습니다.

### 2.1. 리플렉션 연산자: `^^`

리플렉션 연산자 `^^`는 표현식 앞에 붙어 해당 클래스나 함수, 변수 등의 구조 정보를 추출하는 데 사용됩니다. 이렇게 변환된 정보는 `std::meta::info`라는 특별한 타입으로 표현됩니다.

```c++
#include <meta>

namespace MyNamespace {}
class MyClass {};

int main() {
    // int 타입에 대한 리플렉션 정보 가져오기
    constexpr auto int_refl = ^^int;

    // MyNamespace 네임스페이스에 대한 리플렉션 정보 가져오기
    constexpr auto namespace_refl = ^^MyNamespace;

    // MyClass 타입에 대한 리플렉션 정보 가져오기
    constexpr std::meta::info my_class_refl = ^^MyClass;
}
```

### 2.2. 스플라이서: `[: ... :]`

스플라이서 `[: ... :]`는 리플렉션 연산자와는 반대로, `std::meta::info` 객체가 담고 있는 정보를 다시 원래의 코드 요소(타입, 변수 등)로 되돌리는 역할을 합니다.

```c++
namespace MyNamespace {
    class TestClass {};
}

class MyClass {};

int main() {
    constexpr auto int_refl = ^^int;
    constexpr auto ns_refl = ^^MyNamespace;
    constexpr auto my_class_refl = ^^MyClass;

    // 리플렉션 정보를 원본 타입으로 변환
    typename[:int_refl:] int_var = 42;        // int int_var = 42;
    [:ns_refl:]::TestClass namespace_var{};   // MyNamespace::TestClass namespace_var{};
    typename[:my_class_refl:] my_class_var{}; // MyClass my_class_var{};
}
```

위 코드에서처럼 리플렉션 정보를 다시 타입으로 사용하기 위해 `typename[:int_refl:]` 형태로 쓸 수 있습니다. 이때 `typename` 키워드는 스플라이서가 변환할 결과물이 타입이라는 것을 컴파일러에게 명시적으로 알려주는 역할을 합니다.

주의할 점으로는, 스플라이서에 들어갈 수 있는 값은 반드시 **컴파일 타임에 결정되는 값**이어야 하며, 런타임에 결정되는 값은 사용할 수 없습니다.

## 3. 리플렉션의 활용

그러면 이 리플렉션 기능을 어떻게 활용할 수 있을까요? 몇 가지 예시를 들어보겠습니다.

### 3.1. 멤버 선택 (Selecting Members)

아래 예시코드는 구조체/클래스의 멤버를 이름 및 인덱스로 접근하는 방법입니다.

```c++
#include <meta>
#include <string_view>

struct MyStruct {
    int a;
    double b;
    char c;
};

// 인덱스로 멤버 선택
template <typename T>
consteval auto select_by_index(size_t index) {
    // 현재 코드위치에서 MyStruct의 접근할 수 있는 범위 컨텍스트를 가져옵니다.
    constexpr auto ctx = std::meta::access_context::current();

    // T 타입의 멤버 목록에서 인덱스에 해당하는 멤버를 반환합니다.
    return std::meta::nonstatic_data_members_of(^^T, ctx)[index];
}

// 이름으로 멤버 선택
template <typename T>
consteval auto select_by_name(std::string_view name) {
    using namespace std::meta;

    // 현재 코드위치에서 MyStruct의 접근할 수 있는 범위 컨텍스트를 가져옵니다.
    constexpr auto ctx = access_context::current();

    for (auto field : nonstatic_data_members_of(^^T, ctx)) {
        // 멤버의 이름과 name이 일치하는지 확인후, 멤버의 정보를 담은 info객체를 반환합니다.
        if (has_identifier(field) && identifier_of(field) == name) {
            return field;
        }
    }
}

int main() {
    MyStruct var{1, 2.0, 'c'};

    var.[:select_by_index<MyStruct>(0):] = 10;   // a 멤버에 접근
    var.[:select_by_index<MyStruct>(1):] = 3.14; // b 멤버에 접근
    var.[:select_by_name<MyStruct>("c"):] = 'z'; // c 멤버에 접근

    // 존재하지 않는 멤버에 접근 (컴파일 에러)
    // var.[:select_by_index<MyStruct>(3):] = 42;
    // var.[:select_by_name<MyStruct>("d"):] = 42; // 컴파일 에러

    return 0;
}
```

### 3.n. 클래스 멤버 목록 조회

기존에는 매크로나 템플릿 메타프로그래밍을 통해 복잡하고, 유지보수가 어려운 코드를 작성해야 했지만, C++26에서는 리플렉션을 통해 간단하게 클래스의 멤버 목록을 조회할 수 있습니다.

```c++
#include <meta>

class MyClass {
public:
  void test_function(int a, double b) {}

private:

};
```

## 참고 문서

- [Reflection for C++26 (P3394)](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2996r12.html){:target="_blank"}
