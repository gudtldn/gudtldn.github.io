---
title: "C++26 컴파일 타임 리플렉션(Reflection)"

categories:
  - C++
tags:
  - [c++, c++26, reflection]

toc: true
toc_sticky: true

date: 2025-07-13
last_modified_at: 2025-07-26
---

> 이 글은 [C++26 초안(P2996R13)][C++26 P2996R13 docs]{:target="_blank"}과 [bloomberg/clang-p2996][bloomberg/clang-p2996]{:target="_blank"} 리플렉션 구현을 기반으로 작성되었습니다.  
> C++26 표준이 확정되기 전까지는 내용이 부정확하거나 변경될 수 있습니다.

## 1. 개요

이번 C++26 표준에서는 많은 C++ 개발자들이 오랫동안 기다려온 기능인 컴파일 타임 리플렉션(Reflection)이 도입될 예정입니다.

> 리플렉션(Reflection)이란?
> 리플렉션은 프로그램이 자신의 구조를 검사하고 수정할 수 있는 능력을 말합니다.

지금까지는 C++에서 리플렉션을 구현하기 위해 매크로, 템플릿 메타프로그래밍, 외부 라이브러리 등을 사용해야 했지만, C++26에서부터 리플렉션을 위한 공식적인 문법이 도입되어, 보다 직관적이고 간편하게 리플렉션 기능을 사용할 수 있게 되었습니다.

## 2. 새로 추가된 문법

이번 C++26에서는 리플렉션을 위해 다음과 같은 새로운 문법이 추가되었습니다.

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

## 3. 새로 추가된 `<meta>` 헤더와 Metafunctions

위에서 설명한 리플렉션 연산자(`^^`)와 스플라이서(`[:...:]`)만 사용해서는 리플렉션 기능을 완전히 활용하기 어렵습니다.

그래서 C++26에서는 `<meta>`라는 새로운 헤더 파일이 추가되어, `std::meta::info` 객체를 다루는 데 필요한 다양한 기능을 제공합니다.

여기서는 이후의 활용 예제에서 사용될 핵심적인 함수 몇 가지를 중심으로 소개하겠습니다.

> 아래 함수들은 [bloomberg/clang-p2996](https://github.com/bloomberg/clang-p2996/blob/p2996/libcxx/include/meta "meta 헤더로 이동하기"){:target="_blank"}에서 자세히 확인할 수 있습니다.

### 3.1. 리플렉션 정보 확인 함수 (Predicate Functions)

아래의 함수들은 리플렉션 객체가 특정한 속성을 가지고 있는지 확인하는 데 사용됩니다.
리플렉션 정보가 특정 조건을 만족하는지 확인하는 데 유용합니다.

| 함수                   | 설명                                                                                                           |
| :--------------------- | :------------------------------------------------------------------------------------------------------------- |
| `has_identifier(info)` | 주어진 `info` 객체가 **식별자를 가지고 있는지** 확인합니다.                                                    |
| `is_type(info)`        | 주어진 `info` 객체가 **타입 정보를 담고 있는지** 확인합니다.                                                   |
| `is_enum_type(info)`   | 주어진 `info` 객체가 **열거형 타입인지** 확인합니다.                                                           |
| `is_function(info)`    | 주어진 `info` 객체가 **함수 정보를 담고 있는지** 확인합니다.                                                   |
| `is_class(info)`       | 주어진 `info` 객체가 **클래스 정보를 담고 있는지** 확인합니다.                                                 |
| `is_public(info)`      | 주어진 `info` 객체가 **public 멤버인지** 확인합니다.<br>`is_protected` 및 `is_private` 함수도 같이 존재합니다! |
| `is_virtual(info)`     | 주어진 `info` 객체가 **가상(virtual) 멤버인지** 확인합니다.                                                    |
| `is_override(info)`    | 주어진 `info` 객체가 **오버라이드된 멤버인지** 확인합니다.                                                     |

### 3.2. 멤버 및 관계 조회 함수 (Lookup Functions)

아래의 함수들은 리플렉션 객체의 내부구조나, 상속, 객체간의 관계를 조회하는 데 사용됩니다.

| 함수                                   | 설명                                                             |
| :------------------------------------- | :--------------------------------------------------------------- |
| `identifier_of(info)`                  | 주어진 `info` 객체의 식별자(이름)를 반환합니다.                  |
| `bases_of(info, ctx)`                  | 주어진 `info` 객체의 **모든 상위 클래스** 목록을 반환합니다.     |
| `members_of(info, ctx)`                | 주어진 `info` 객체의 **모든 멤버** 목록을 반환합니다.            |
| `nonstatic_data_members_of(info, ctx)` | 주어진 `info` 객체의 비정적 데이터 멤버 목록을 반환합니다.       |
| `static_data_members_of(info, ctx)`    | 주어진 `info` 객체의 정적(static) 데이터 멤버 목록을 반환합니다. |
| `enumerators_of(info, ctx)`            | 주어진 `info` 객체의 **열거형 멤버** 목록을 반환합니다.          |
| `is_same_type(info1, info2)`           | 두 `info` 객체가 **같은 타입인지** 확인합니다.                   |

### 3.3. 메타데이터 변환 및 타입 생성 함수

아래의 함수들은 리플렉션 정보를 원래의 타입으로 변환하거나, 템플릿 인자를 대체하여 새로운 타입을 생성하는 데 사용됩니다.

| 함수                         | 설명                                                                                                                                     |
| :--------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------- |
| `extract<T>(info)`           | 주어진 `info` 객체가 가지고 있는 값을 **원래 타입 `T`의 값으로 변환**합니다.                                                             |
| `substitute(info, args...)`  | 주어진 템플릿 리플렉션 정보와, 템플릿 인자의 리플렉션 정보를 조합하여,<br> **특정 템플릿을 구체화한 타입에 대한 info객체를 생성**합니다. |
| `reflect_constant(const T&)` | 주어진 **상수 값을 리플렉션 정보로 변환**합니다.                                                                                         |
| `reflect_object(T&)`         | 주어진 **객체를 리플렉션 정보로 변환**합니다.                                                                                            |
| `reflect_function(T&)`       | 주어진 **함수를 리플렉션 정보로 변환**합니다.                                                                                            |
| `type_of(info)`              | 주어진 **`info` 객체가 가리키는 타입의 `std::meta::info`를 반환**합니다.                                                                 |
| `object_of(info)`            | 주어진 **`info` 객체가 가리키는 객체의 `std::meta::info`를 반환**합니다.                                                                 |
| `constant_of(info)`          | 주어진 **`info` 객체가 가리키는 상수의 `std::meta::info`를 반환**합니다.                                                                 |

## 4. 리플렉션의 활용

그러면 이 리플렉션 기능을 어떻게 활용할 수 있을까요? 몇 가지 예시를 들어보겠습니다.

### 4.1. 멤버 선택 (Selecting Members)

아래 예시코드는 구조체/클래스의 멤버를 이름 및 인덱스로 접근하는 방법입니다.

> [Compiler Explorer에서 실행하기](https://godbolt.org/z/vano5PTjM){:target="_blank"}

```c++
#include <meta>
#include <print>
#include <string_view>

struct MyStruct {
    int a;
    double b;
    char c;
};

// 인덱스로 멤버 선택
template <typename T>
    requires std::is_class_v<T>
consteval auto select_by_index(size_t index) -> std::meta::info {
    // 현재 코드위치에서 MyStruct의 접근할 수 있는 범위 컨텍스트를 가져옵니다.
    constexpr auto ctx = std::meta::access_context::current();

    // T 타입의 멤버 목록에서 인덱스에 해당하는 멤버를 반환합니다.
    return std::meta::nonstatic_data_members_of(^^T, ctx)[index];
}

// 이름으로 멤버 선택
template <typename T>
    requires std::is_class_v<T>
consteval auto select_by_name(std::string_view name) -> std::meta::info {
    using namespace std::meta;

    // 현재 코드위치에서 MyStruct의 접근할 수 있는 범위 컨텍스트를 가져옵니다.
    constexpr auto ctx = access_context::current();

    for (auto field : nonstatic_data_members_of(^^T, ctx)) {
        // 멤버의 이름과 name이 일치하는지 확인후, 멤버의 정보를 담은 info객체를 반환합니다.
        if (has_identifier(field) && identifier_of(field) == name) {
            return field;
        }
    }

    // 만약 일치하는 멤버가 없다면, 빈 info 객체를 반환합니다.
    return std::meta::info{};
}

int main() {
    MyStruct var{1, 2.0, 'c'};

    var.[:select_by_index<MyStruct>(0):] = 10;   // a 멤버에 접근
    var.[:select_by_index<MyStruct>(1):] = 3.14; // b 멤버에 접근
    var.[:select_by_name<MyStruct>("c"):] = 'z'; // c 멤버에 접근

    // 존재하지 않는 멤버에 접근 (컴파일 에러)
    // var.[:select_by_index<MyStruct>(3):] = 42;
    // var.[:select_by_name<MyStruct>("d"):] = 42; // 컴파일 에러

    std::println("a: {}", var.a); // 10
    std::println("b: {}", var.b); // 3.14
    std::println("c: {}", var.c); // z

    return 0;
}
```

### 4.2. 열거형 멤버 목록 조회

C++26에서는 열거형 타입에 대한 리플렉션 기능도 제공됩니다. 이를 통해 열거형의 멤버 목록을 쉽게 조회할 수 있습니다.

> [Compiler Explorer에서 실행하기](https://godbolt.org/z/T51xcqfjj){:target="_blank"}

```c++
#include <meta>
#include <array>
#include <string_view>
#include <type_traits>
#include <iostream>

template <typename T>
    requires std::is_enum_v<T>
consteval auto get_enum_values()
{
    // 임의의 타입 T에 대한 리플렉션 정보를 가져옵니다.
    constexpr auto refl = ^^T;

    // requires 대신 static_assert를 사용할 경우
    static_assert(std::meta::is_enum_type(refl), "T must be an enum type");

    // 열거형의 멤버 목록을 가져옵니다.
    auto enum_values = std::meta::enumerators_of(refl);
    constexpr size_t len = std::meta::enumerators_of(refl).size();

    std::array<std::string_view, len> ret{};
    for (size_t i = 0; i < len; ++i)
    {
        // 각 열거형 멤버의 식별자를 문자열로 변환하여 배열에 저장합니다.
        ret[i] = std::meta::identifier_of(enum_values[i]);
    }

    return ret;
}

enum class MyEnum {
    Value1,
    Value2,
    Value3
};

int main() {
    constexpr auto enum_values = get_enum_values<MyEnum>();

    for (const auto& value : enum_values) {
        std::cout << value << std::endl; // Value1, Value2, Value3
    }

    return 0;
}
```

<!-- ### 4.n. 클래스 정보 추출

기존에는 매크로나 템플릿 메타프로그래밍을 통해 복잡하고, 유지보수가 어려운 코드를 작성해야 했지만, C++26에서는 리플렉션을 통해 간단하게 클래스의 멤버 목록을 조회할 수 있습니다.

```c++
#include <meta>

class MyClass {
public:
    void test_function(int a, double b) {}

protected:
    int my_int;

private:
    double my_double;
};
```

```c++
template <typename T>
consteval auto get_custom_attributes_of_class()
{
    constexpr auto ctx = std::meta::access_context::current();
    auto members = std::meta::nonstatic_data_members_of(^^T, ctx);
    std::array<std::string_view, 2> names{};
    
    for (size_t i = 0; i < members.size(); ++i)
    {
        names[i] = std::meta::identifier_of(members[i]);
    }
    
    return names;
}
```

```c++
#include <array>
#include <string>
#include <string_view>
#include <experimental/meta>
#include <iostream>

enum class Type : uint8_t {
    Unknown,
    Int,
    Float,
    String,
    Bool
};

struct MemberInfo {
    Type type;
    std::string_view member_name;
};

template <typename T>
consteval auto get_members() {
    // 임의의 타입 T에 대한 리플렉션 정보를 가져옵니다.
    constexpr auto refl = ^^T;

    // T의 멤버 리플렉션 정보를 가져옵니다.
    constexpr auto ctx = std::meta::access_context::unchecked();
    auto members = std::meta::nonstatic_data_members_of(refl, ctx);

    // 멤버들을 순회하면서 각 멤버의 이름과 타입을 추출합니다.
    constexpr size_t len = std::meta::nonstatic_data_members_of(refl, ctx).size();
    std::array<MemberInfo, len> member_info{};

    for (size_t i = 0; i < len; ++i) {
        const auto member = members[i];
        member_info[i].member_name = std::meta::identifier_of(member);
        
        // 멤버의 타입을 확인하고 Type 열거형에 매핑합니다.
        auto member_type = std::meta::type_of(member);
        if (std::meta::is_integral_type(member_type)) {
            member_info[i].type = Type::Int;
        } else if (std::meta::is_floating_point_type(member_type)) {
            member_info[i].type = Type::Float;
        } else if (^^std::string == member_type) {
            member_info[i].type = Type::String;
        } else if (^^bool == member_type) {
            member_info[i].type = Type::Bool;
        } else {
            member_info[i].type = Type::Unknown;
        }
    }

    return member_info;
}

consteval auto get_enum_identifier(const std::meta::info member) {
    // 멤버의 식별자를 가져옵니다.
    return std::meta::identifier_of(member);
}

struct MyData {
    int x = 10;
    double y = 20.5;
    std::string label = "Center";
};

int main() {
    constexpr auto members = get_members<MyData>();

    for (const auto& member : members) {
        std::cout << "Member Name: " << member.member_name << ", Type: ";
        switch (member.type) {
            case Type::Int: std::cout << "Int"; break;
            case Type::Float: std::cout << "Float"; break;
            case Type::String: std::cout << "String"; break;
            case Type::Bool: std::cout << "Bool"; break;
            default: std::cout << "Unknown"; break;
        }
        std::cout << '\n';
    }
}
``` -->

---

> 표준이 변경됨에 따라 계속 수정될 예정입니다!

## 참고 문서

- [Reflection for C++26 (P2996R13)][C++26 P2996R13 docs]{:target="_blank"}
- [bloomberg/clang-p2996 GitHub Repository][bloomberg/clang-p2996]{:target="_blank"}

[C++26 P2996R13 docs]: https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2996r13.html
[bloomberg/clang-p2996]: https://github.com/bloomberg/clang-p2996
