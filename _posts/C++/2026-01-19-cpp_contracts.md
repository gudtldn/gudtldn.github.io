---
title: "C++26 계약 프로그래밍(Contracts)"
image: https://upload.wikimedia.org/wikipedia/commons/thumb/1/18/ISO_C%2B%2B_Logo.svg/330px-ISO_C%2B%2B_Logo.svg.png
categories:
  - C++
tags:
  - [c++, c++26]

toc: true
toc_sticky: true

date: 2026-01-19
last_modified_at: 2026-01-19
---

## 1. 개요

C++26부터 **계약 프로그래밍(Contracts)** 기능이 도입되었습니다. 기존의 `assert` 매크로가 단순히 런타임에 로직을 체크하는 디버깅 도구였다면, Contracts는 함수의 제약 사항을 **언어적 차원에서 명시**하는 강력한 기능을 제공합니다.

덕분에 개발자는 주석 대신 코드로 함수의 제약 사항을 명확히 명시할 수 있고, 컴파일러는 이 조건을 바탕으로 최적화에 활용하거나 정적 분석 시 잠재적인 버그를 더 정교하게 잡아낼 수 있습니다.

## 2. 새로 추가된 문법 및 기능

이번 C++26에서 Contracts를 위해 다음과 같은 새로운 문법이 추가되었습니다.

### 2.1. 사전 조건(pre) 및 사후 조건(post)

아래 코드와 같이 함수 선언부에 직접 제약 조건을 명시하여 검사할 수 있습니다.

```c++
int divide(int a, int b)
    pre (b != 0)       // 사전 조건: b는 0이 아니어야 함
    post (r : r <= a)  // 사후 조건: 결과값 r은 a보다 작거나 같아야 함
{
    // 람다식에서도 사전/사후 조건 사용 가능
    auto my_func = [](int x) pre(x > 0) { return x; };
    return a / b;
}
```

여기서 사전 조건(`pre`)은 함수 실행 직전에, 사후 조건(`post`)은 함수 반환 직전에 평가됩니다. 특히 사후 조건에서 반환값에 이름을 지정하여 조건을 검사할 수 있습니다.

### 2.2. `contract_assert` 키워드

`contract_assert`는 함수 본문 내에서 특정 상태를 보장하기 위해 사용합니다. 기존 매크로 기반 `assert`와 달리 컴파일러가 직접 구문을 분석하므로 **정적 분석과 최적화**에 훨씬 유리합니다.

```c++
void g(int* p)
{
    // p가 nullptr이 아님을 언어 차원에서 보장
    contract_assert(p != nullptr);
}
```

### 2.3. 계약 위반 처리 (Violation Handling)

#### 2.3.1. 평가 시맨틱(Evaluation Semantics)

이번 Contracts의 또 다른 핵심은 작성된 조건을 언제, 어떻게 검사할지 **빌드 시점에 유연하게 결정할 수 있다는 점**입니다.  
이를 위해 다음과 같은 3가지 **평가 시맨틱(Evaluation Semantics)**을 제공합니다.

| 평가 시맨틱   | 체크 수행 (Checking) | 프로그램 종료 (Terminating) | 주요 용도                         |
| ------------- | -------------------- | --------------------------- | --------------------------------- |
| **`enforce`** | **Yes**              | **Yes**                     | 일반적인 런타임 검사 및 강제 종료 |
| **`observe`** | **Yes**              | No                          | 로그 기록 및 모니터링 (실행 유지) |
| **`ignore`**  | No                   | No                          | 성능 최적화 (검사 생략)           |

- **`enforce`**: 조건이 위반되면 핸들러 호출 후 즉시 프로그램을 종료(std::abort)합니다. 특히 상수 평가(Compile-time) 중 위반이 발생하면 컴파일 에러가 발생합니다.
- **`observe`**: 조건이 위반되더라도 프로그램은 계속 실행됩니다. 위반 시 핸들러만 호출하며, 상수 평가 중 위반 시에는 에러 대신 컴파일러 경고(Warning)를 띄웁니다.
- **`ignore`**: 런타임 조건 검사를 완전히 생략합니다. 하지만 기존 assert 매크로와 달리, 컴파일러가 구문을 분석(Parsing)하므로 항상 문법적으로 올바른 코드 상태를 유지해야 한다는 점이 있습니다.

#### 2.3.2. 위반 핸들러 (Contract-violation handler)

계약 위반이 발생했을 때 최종적으로 호출되는 함수로, 이 함수는 특이하게도 표준 라이브러리(`std`)가 아닌 **글로벌 네임스페이스(`::`)**에 정의합니다.

```c++
// C++26부터 제공되는 핸들러 시그니처
void handle_contract_violation(std::contracts::contract_violation cv);
```

##### 1. 링크 타임 교체 (Replaceable)

기존 `std::set_terminate`같은 런타임에 등록하는 방식이 아니라, `operator new`처럼 **링크 타임에 결정**됩니다.

- **우선순위**: 사용자가 직접 정의한 함수가 있으면 우선적으로 링크되며, 정의되지 않은 경우에만 기본 핸들러가 호출됩니다.
- **주의사항**: 프로그램 전체에서 **단 하나의 `.cpp` 파일**에서만 정의해야 하며, 반드시 **외부 연결(External Linkage)**이 가능해야 합니다. (익명 네임스페이스 사용 시 무시됨)

##### 2. 예외 처리 및 원인 파악

단순히 조건이 거짓(`false`)일 때뿐만 아니라, **평가 중 예외가 발생**해도 핸들러가 호출됩니다.

이때 핸들러는 **암시적 핸들러** 내부에서 실행되므로 **`std::current_exception()`**을 통해 구체적인 예외 원인을 파악할 수 있습니다.

## 3. 주의 사항

Contracts를 사용할 때 가장 주의해야 할 점은 **계약 조건 내에서의 부작용(Side Effects) 발생**입니다.

### 3.1. 변수 수정 시 컴파일 에러

계약 조건(Predicate)은 관찰이 목적이므로, 내부에서 사용되는 **모든 지역 변수**는 자동으로 **`const`** 취급합니다.

```c++
void update_score(int score) 
    pre(score = 100)  // error: 조건식 내부에서 지역 변수 수정 불가
{ 
    /* ... */ 
}
```

### 3.2. 평가 횟수 및 실행 여부의 불확실성

계약 조건(Predicate) 내부에서 상태를 변경하는 코드를 작성하면 예상치 못한 버그가 발생할 수 있습니다.

```c++
static int count = 0;
void process() 
    pre(count++ < 10) // 위험: count가 실제로 증가할지 알 수 없음 (Unspecified)
{ 
    /* ... */ 
}
```

위 코드에서 `count`가 실제로 증가할지는 **컴파일러나 빌드 설정에 따라 달라집니다.** 최적화 과정에서 평가 자체가 아예 생략될 수도 있고, 반대로 여러 번 수행될 수도 있기 때문에 **실행 결과가 보장되지 않습니다.**

따라서 계약 조건은 실행 여부와 상관없이 프로그램의 흐름에 영향을 주지 않도록, 항상 **상태를 변경하지 않는 코드(Side-effect free)**로 작성하는 것이 원칙입니다.

---

## 참고 문서

- [C++26 Contracts – cppreference.com][cppreference/contracts]{:target="_blank"}
- [Contracts in C++26 – Modernes C++][modernescpp/contracts]{:target="_blank"}
- [P2900R6: Contracts for C++ (ISO C++ Proposal)][isocpp/p2900r6]{:target="_blank"}

[cppreference/contracts]: https://en.cppreference.com/w/cpp/language/contracts.html
[modernescpp/contracts]: https://www.modernescpp.com/index.php/contracts-in-c26/
[isocpp/p2900r6]: https://isocpp.org/files/papers/P2900R6.pdf
