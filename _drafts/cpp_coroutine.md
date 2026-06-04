---
title: "C++20 코루틴(Coroutines)"
image: https://upload.wikimedia.org/wikipedia/commons/thumb/1/18/ISO_C%2B%2B_Logo.svg/330px-ISO_C%2B%2B_Logo.svg.png
categories:
  - C++
tags:
  - [c++, coroutines]

toc: true
toc_sticky: true

date: 1234-01-23
last_modified_at: 1234-01-23
---

## 1. 개요

C++20에서 도입된 코루틴(Coroutines)은 비동기 프로그래밍과 제너레이터를 쉽게 구현할 수 있도록 하는 기능입니다. 코루틴은 함수의 실행을 일시 중단하고 나중에 다시 시작할 수 있는 기능을 제공합니다. 이를 통해 비동기 작업을 보다 간결하고 효율적으로 작성할 수 있습니다.

## 2. 코루틴의 기본 개념

코루틴은 일반 함수와 달리 `co_await`, `co_yield`, `co_return`과 같은 키워드를 사용하여 실행을 제어합니다. 코루틴은 다음과 같은 세 가지 주요 요소로 구성됩니다:

### 2.1. Promise Type

Promise Type은 코루틴이 반환하는 값과 상태를 관리하는 객체입니다. 코루틴이 시작될 때 Promise Type 객체가 생성되며, 코루틴이 반환하는 값이나 예외를 처리합니다.

모든 코루틴은 `std::coroutine_traits`에 따라 아래와 같은 함수를 Promise Type에 정의해야 합니다

| 함수                                  | 설명                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| `get_return_object()`                 | 코루틴이 반환하는 객체를 생성하는 함수입니다.                |
| `initial_suspend()`                   | 코루틴이 시작될 때 일시 중단할지 여부를 결정하는 함수입니다. |
| `final_suspend()`                     | 코루틴이 종료될 때 일시 중단할지 여부를 결정하는 함수입니다. |
| `return_void()` 또는 `return_value()` | 코루틴이 값을 반환할 때 호출되는 함수입니다.                 |
| `unhandled_exception()`               | 코루틴에서 예외가 발생했을 때 호출되는 함수입니다.           |

### 2.2. Coroutine Handle

Coroutine Handle은 코루틴의 실행 상태를 제어하는 객체입니다. 이를 통해 코루틴을 일시 중단하거나 재개할 수 있습니다. Coroutine Handle은 `std::coroutine_handle` 클래스로 표현됩니다.

### 2.3. Awaitable Object

Awaitable Object는 `co_await` 키워드로 대기할 수 있는 객체입니다. Awaitable Object는 `operator co_await`를 구현하여 코루틴이 대기할 수 있도록 합니다. 이를 통해 비동기 작업이 완료될 때까지 코루틴을 일시 중단할 수 있습니다.

## 3. 코루틴의 사용 예시

```cpp
#include <coroutine>


```
