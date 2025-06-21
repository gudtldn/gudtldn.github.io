---
title: "C++26 Annotations for Reflection"

categories:
  - C++
tags:
  - [c++, c++26, reflection, annotations]

toc: true
toc_sticky: true

date: 2025-06-20
last_modified_at: 2025-06-20
---

## C++26 Annotations for Reflection

이번 C++26 표준에 도입된 [컴파일 타임 리플렉션(Reflection)](https://gudtldn.github.io/posts/cpp_reflection "새 창에서 관련 포스팅 열기"){:target="_blank"}은 리플렉션 연산자(`^^`)와 스플라이서(`[: ... :]`)를 통해 각종 코드 요소의 구조 정보를 조회할 수 있는 강력한 기능을 제공합니다. 이를 통해서 클래스의 멤버 목록을 얻거나, 특정 함수의 시그니처를 확인하는 등의 작업이 가능해졌습니다.

하지만, 만약 코드의 구조 정보 외에 추가적인 메타데이터를 함께 저장하고 싶다면 어떻게 해야 할까요?

이러한 요구사항을 해결하기 위해 리플렉션의 확장으로 **어노테이션(Annotation)**이 제안되었습니다. 어노테이션은 기존 속성(Attribute) 문법을 확장하여, 선언부에 컴파일 타임 값을 직접 첨부하는 새로운 방법을 제시합니다.

## 어노테이션(Annotation) 문법

어노테이션은 다음과 같은 문법으로 사용할 수 있습니다.

| 문법                      | 설명                              | 사용 예시                                  |
| ------------------------- | --------------------------------- | ------------------------------------------ |
| `[[=value]]`              | 기본 값 어노테이션                | `int [[=42]] x;`                           |
| `[[=some_object]]`        | 객체 어노테이션                   | `struct [[=Debug{}]] Point { int x, y; };` |
| `[[=function_call()]]`    | 함수 호출 결과 어노테이션         | `void [[=get_priority()]] func();`         |
| `[[=complex_expression]]` | 복잡한 표현식 어노테이션          | `auto [[=sizeof(T) > 8]] large_data;`      |
| `[[=first, =second]]`     | 다중 어노테이션                   | `bool [[=serialize, =validate]] flag;`     |
| `[[=namespace::type{}]]`  | 네임스페이스 포함 타입 어노테이션 | `struct [[=serde::derive]] Person {};`     |

아래는 다양한 코드 요소에 어노테이션을 적용하는 예시입니다.

```c++
struct AttrTag {};
inline constexpr AttrTag attr_tag{};

struct AttrValue {
    int value;
    constexpr AttrValue(int v) : value(v) {}
};

// 변수에 어노테이션 적용
[[=attr_tag, =AttrValue(42)]]
[[=AttrValue(1)]]
int my_variable;

// 함수에 어노테이션 적용
[[=attr_tag, =AttrValue(123)]]
void my_function();

// 구조체/클래스에 어노테이션 적용
struct [[=attr_tag]] MyStruct {
    [[=AttrValue(42)]]
    int member;
};
```

## 어노테이션 사용 예시

이렇게 추가한 어노테이션은 `<meta>` 헤더에 있는 각종 헬퍼 함수들을 통해 조회할 수 있습니다.

## 참고 문서

- [Annotations for Reflection (P3394)](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3394r3.html){:target="_blank"}
