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

이번 C++26 표준에 도입된 [컴파일 타임 리플렉션(Reflection)](https://gudtldn.github.io/posts/cpp_reflection "새 창에서 관련 포스팅 열기"){:target="_blank"}은 `^^` 연산자와 `[: :]` 스플라이서를 통해 각종 코드 요소의 구조 정보를 조회할 수 있는 강력한 기능을 제공합니다. 이를 통해서 클래스의 멤버 목록을 얻거나, 특정 함수의 시그니처를 확인하는 등의 작업이 가능해졌습니다.

하지만, 만약 코드의 구조 정보 외에 추가적인 메타데이터를 함께 저장하고 싶다면 어떻게 해야 할까요?

이러한 요구사항을 해결하기 위해 리플렉션의 확장으로 **어노테이션(Annotation)**이 제안되었습니다. 어노테이션은 기존 속성(Attribute) 문법을 확장하여, 선언부에 컴파일 타임 값을 직접 첨부하는 새로운 방법을 제시합니다.

## 어노테이션(Annotation) 문법

어노테이션은 다음과 같은 문법으로 사용됩니다:

```cpp
[[=expr]] int my_variable;
[[=expr1, =expr2]] void my_function();
struct [[=expr]] MyStruct {
    [[=expr]] int member;
};
```

## 참고 문서

- [Annotations for Reflection (P3394)](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3394r3.html){:target="_blank"}
