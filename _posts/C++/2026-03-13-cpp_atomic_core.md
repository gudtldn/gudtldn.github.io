---
title: "C++ std::atomic (1): 원자적 연산과 메모리 모델(Memory Model)"
image: https://upload.wikimedia.org/wikipedia/commons/thumb/1/18/ISO_C%2B%2B_Logo.svg/330px-ISO_C%2B%2B_Logo.svg.png
categories:
  - C++
tags:
  - [c++, concurrency, atomic]

toc: true
toc_sticky: true

date: 2026-03-13
last_modified_at: 2026-03-13
---

## 1. 개요

현대 멀티스레딩 환경에서 스레드 간 안전한 데이터 공유는 필수적이지만, `std::mutex`같은 OS기반 Lock은 무겁고, OS의 스케줄링으로 인한 컨텍스트 스위칭 비용이 발생하게 됩니다.

C++의 `std::atomic`은 멀티스레딩 환경에서 OS차원의 Lock없이 안전하게 공유 자원에 접근할 수 있도록 도와주는 라이브러리입니다.

이번 글에서는 다음의 주제들을 다룰 예정입니다:

1. **`std::atomic`의 원리**: 왜 일반 변수의 연산은 멀티스레드에서 안전하지 않은가?
2. **`std::memory_order`와 메모리 모델**: CPU와 컴파일러의 명령어 재배치(Reordering)를 어떻게 제어할 것인가?
3. **CAS(Compare-And-Swap) 연산**: Lock-free 알고리즘의 핵심인 `.compare_exchange_*()` 연산의 사용방법
4. **C++20 대기 메커니즘**: `.wait()`과 `.notify_*()`를 이용한 효율적인 스레드 동기화.

## 2. `std::atomic`의 원리

`std::atomic`은 하드웨어 수준에서 제공하는 **원자적 연산**을 활용하여 멀티스레드 환경에서 안전한 데이터 공유를 가능하게 합니다.

### 2.1. 원자적 연산이란?

여기서 원자적 연산이란, 현실 세계의 "원자"처럼 **더 이상 쪼갤 수 없는 연산을 의미**합니다. 예를 들어, 아래와 같이 일반적인 `int` 변수에 대한 증가 연산(`++counter`)을 예시로 들면

```c++
static int counter = 10;

void test()
{
    ++counter;
}
```

```nasm
; x86-64 Assembly (GCC 15.2)
mov eax, DWORD PTR counter[rip]
add eax, 1
mov DWORD PTR counter[rip], eax
```

이렇게 1줄로 보이는 `++counter`가 실제로는 3개의 명령어로 분해되어 실행됩니다:

1. `mov eax, DWORD PTR counter[rip]`: `counter`의 값을 레지스터 `eax`로 읽어옵니다.
2. `add eax, 1`: `eax`의 값을 1 증가시킵니다.
3. `mov DWORD PTR counter[rip], eax`: 증가된 값을 다시 `counter`에 저장합니다.

이 과정에서 여러개의 스레드가 동시에 `counter`에 접근한다면, 서로의 명령어가 뒤엉키게 되어 **데이터 레이스가 발생**할 수 있습니다.

예를 들어, 두 개의 스레드 A와 B가 동시에 `++counter`를 수행한다고 가정해봅시다:

| 단계 | Thread A (CPU Core 0)                                                | Thread B (CPU Core 1)                                                | `counter`의 값 |
| ---- | -------------------------------------------------------------------- | -------------------------------------------------------------------- | -------------- |
| 1    | `mov eax, counter` (eax = 10)<br>(`counter`를 읽어서 `eax`에 저장)   | -                                                                    | 10             |
| 2    | -                                                                    | `mov eax, counter` (eax = 10)<br>(`counter`를 읽어서 `eax`에 저장)   | 10             |
| 3    | -                                                                    | `add eax, 1` (eax = 11)<br>(`eax`에 1을 더해서 `counter`에 저장)     | 11             |
| 4    | -                                                                    | `mov counter, eax` (counter = 11)<br>(`eax`의 값을 `counter`에 저장) | 11             |
| 5    | `add eax, 1` (eax = 11)<br>(`eax`에 1을 더해서 `counter`에 저장)     | -                                                                    | 11             |
| 6    | `mov counter, eax` (counter = 11)<br>(`eax`의 값을 `counter`에 저장) | -                                                                    | 11             |

결과적으로, 2번의 `++counter` 연산이 수행되었음에도 불구하고 `counter`의 최종 값은 11이 되어버립니다. **Lost Update** 문제라고 불리는 이 현상은 멀티스레드 환경에서 매우 흔하게 발생하는 버그입니다.

### 2.2. `std::atomic`은 어떻게 해결하는가?

여기서 `std::atomic`을 사용하면, 위와 같이 쪼개진 3개의 명령어(읽기, 수정, 쓰기)가 **하드웨어 수준에서 원자적으로 실행**되도록 보장됩니다.

즉, `std::atomic<int> counter;`로 선언된 `counter`에 대해 `++counter;`를 수행하면, CPU는 이 연산을 하나의 원자적 단위로 처리하여, 다른 스레드가 중간에 끼어들 수 없도록 합니다.

```c++
#include <atomic>

static std::atomic<int> counter = 10;

void test()
{
    ++counter;
}
```

```nasm
; x86-64 Assembly (GCC 15.2)
lock add DWORD PTR atom_counter[rip], 1
```

위와 같이 `lock` 접두사가 붙은 명령어로 변환되어, 실행하는 동안 다른 스레드가 `counter`에 접근하지 못하도록 보장합니다. 이를 통해서 멀티스레드 환경에서도 안전하게 `counter`의 값을 증가시킬 수 있습니다.

## 3. `std::memory_order`와 메모리 모델

### 3.1. 명령어 재배치(Instruction Reordering)란?

단일 스레드에서는 결과만 같다면, 컴파일러와 CPU가 성능 최적화를 위해 명령어의 실행 순서를 임의로 바꾸기도 하는데요, 이를 **명령어 재배치(Instruction Reordering)**라고 합니다.

```c++
int x = 0, y = 0;

void thread1()
{
    x = 10;
    y = 20;
}

void thread2()
{
    if (y == 20)
    {
        assert(x == 10); // 이 assert가 실패할 가능성도 존재
    }
}
```

위 코드에서 `thread1`이 `x`와 `y`에 값을 할당하는 순서가 컴파일러나 CPU에 의해 바뀔 수 있습니다.
만약 `y = 20;`이 먼저 실행되고, 그 다음에 `x = 10;`이 실행된다면, `thread2`에서 `y == 20`이 참이지만 `x == 10`이 거짓이 되는 상황이 발생할 수 있습니다.

### 3.2. `std::memory_order`의 종류와 가시성 보장

`std::atomic`의 연산 함수들은 2번째 인자로 `std::memory_order`를 받을 수 있는데, 이를 통해 **어디까지 명령어 재배치를 허용할 것인지**를 제어할 수 있습니다.

1. `std::memory_order_relaxed` (가장 느슨한 조건)
    - **의미**: 연산의 **원자성만 보장**하며, 컴파일러 및 CPU의 최적화를 위한 재배치를 제한하지 않습니다.
    - **용도**: 다른 변수와의 실행 순서 동기화가 불필요한 독립적 카운터 등에 사용됩니다.

2. `std::memory_order_acquire` & `release`  
    이 두 설정은 주로 **데이터의 생산자와 소비자 관계**에서 한 쌍으로 사용됩니다.
    - **`std::memory_order_release` (쓰기)**: 이 연산 이전의 모든 쓰기 작업이 완료되었음을 보장합니다.
    - **`std::memory_order_acquire` (읽기)**: 동일한 변수에 대해 `release`로 저장된 값을 읽는 순간, `release` 이전의 모든 메모리 변경 사항이 현재 스레드에 반영됨을 보장합니다.

3. `std::memory_order_acq_rel`
    - **의미**: 하나의 연산에서 `acquire`와 `release` 기능을 동시에 수행합니다.
    - **용도**: `fetch_add`, `exchange`등 **읽기-수정-쓰기(RMW)** 연산에서 이전 작업의 가시성을 확보하는 동시에 현재의 변경 사항을 다른 스레드에 전파할 때 사용합니다.

4. `std::memory_order_seq_cst` (가장 강한 조건)
    - **의미**: 모든 스레드가 모든 원자적 연산에 대해 동일한 순서를 관측하도록 강제합니다.
    - **용도**: `std::atomic`에서 사용하는 연산의 기본값이며, 성능 비용이 가장 높지만 멀티스레드 환경에서 가장 직관적인 프로그래밍 모델을 제공합니다.

### 3.3. 주요 멤버 함수

1. `.load()` / `.store()`
    값을 원자적으로 읽고 쓰는 가장 기본적인 연산입니다.
    - `.load(order)`: 값을 읽어옵니다.
    - `.store(value, order)`: 값을 저장합니다.

2. `fetch_add()`, `fetch_sub()` (산술 연산)
    값을 수정함과 동시에 이전 값을 반환하는 연산입니다.
    - `fetch_add(value, order)`: 현재 값에 `value`를 더하고 이전 값을 반환합니다.
    - `fetch_sub(value, order)`: 현재 값에서 `value`를 빼고 이전 값을 반환합니다.
    - `fetch_and()`, `fetch_or()`, `fetch_xor()`: 비트 연산을 수행하는 함수들도 존재합니다.

3. `.exchange()`
    현재 값을 새로운 값으로 교체하고, 이전 값을 반환하는 연산입니다.
    - `.exchange(new_value, order)`: 현재 값을 `new_value`로 교체하고, 이전 값을 반환합니다.

## 4. CAS(Compare-And-Swap) 연산

이 외에도, **특정 조건일 때만 값을 바꾸고 싶을 때** 사용할 수 있는 CAS 연산도 존재합니다. C++에서는 `.compare_exchange_strong()`과 `.compare_exchange_weak()` 두 가지 버전으로 제공됩니다.

### 4.1. `.compare_exchange_strong()` vs `.compare_exchange_weak()`

이렇게 `strong`과 `weak` 버전이 존재하는 이유는, **가짜 실패(Spurious Failure)** 때문입니다.

{: .prompt-info }
> 가짜 실패란, CAS 연산이 실제로는 성공할 조건임에도 불구하고, 하드웨어나 CPU의 최적화로 인해 실패하는 현상을 말합니다.

| 종류     | 가짜 실패(Spurious Failure) 발생 여부 | 특징                                                                           |
| -------- | ------------------------------------- | ------------------------------------------------------------------------------ |
| `strong` | 발생하지 않음                         | 값이 같으면 무조건 성공 보장. 사용이 직관적                                    |
| `weak`   | 발생 가능                             | 값이 같아도 하드웨어(캐시 탈락 등) 이유로 실패할 수 있음. 반복문에서 사용 권장 |

### 4.2. 사용 예시

기본적인 인자의 의미는 다음과 같습니다.

```c++
bool success = target.compare_exchange_strong(
    expected,      // 비교할 값 (실패 시 현재 값으로 업데이트됨)
    desired,       // 교체할 값
    success_order, // 성공 시, 데이터가 교체될 때 적용할 order (주로 release, acq_rel)
    failure_order  // 실패 시, 읽기만 수행할 때 적용할 order (주로 relaxed, acquire)
);
```

- **성공 시 (`true`)**: `target == expected`이므로 `target`에 `desired`를 저장합니다. 이때는 데이터를 실제로 수정(Write)하므로 `release`나 `acq_rel`을 사용하여 다른 스레드에 변경 사항을 전파합니다.
- **실패 시 (`false`)**: `target != expected`이므로 값을 바꾸지 않습니다. 대신 **`expected` 변수가 `target`의 최신 값으로 업데이트** 되므로 루프에서 바로 재시도를 할 수 있습니다. 이때는 값만 읽어온(Read) 상태이므로 `relaxed`나 `acquire`정도의 낮은 메모리 순서를 지정하는 것이 일반적입니다.

{: .prompt-warning }
> 주의: `failure_order`는 `success_order`보다 강한 메모리 순서를 지정할 수 없으며, failure_order는 값을 수정하지 않으므로 `release`나 `acq_rel`은 사용할 수 없습니다.

`strong`과 `weak`의 활용 예시

```c++
std::atomic<int> target(10);
int expected = 10;
int desired = 20;

// strong: 단일 조건문에서 주로 사용
if (target.compare_exchange_strong(expected, desired))
{
    // 성공 시 로직
}

// weak: 루프를 돌며 성공할 때까지 재시도
// 하드웨어 아키텍처에 따라 strong보다 성능상 이득일 때가 많음
while (!target.compare_exchange_weak(expected, desired))
{
    // 실패 시 expected가 현재 값으로 갱신되므로,
    // 로직에 따라 desired를 다시 계산하거나 그대로 재시도 가능
}
```

## 5. C++20 대기 메커니즘 (`wait()`과 `notify_*()`)

기존에는 atomic 변수의 상태를 기다리기 위해서 **Spin-lock**을 구현해서 사용하거나, 무거운 `condition_variable`을 함께 사용해야 했습니다. C++20부터는 `std::atomic`에 **대기 메커니즘**이 추가되어, 효율적으로 스레드 동기화를 할 수 있게 되었습니다.

### 5.1. 주요 함수와 동작

- `.wait(old_value, order)`: 현재 값이 `old_value`와 같다면, 값이 변경될 때까지 현재 스레드를 대기(Blocking) 상태로 만듭니다. (OS수준에서 스레드를 휴면 상태로 전환하여 CPU 자원을 낭비하지 않음)
- `.notify_one()`: `.wait()`으로 대기 중인 스레드 중 하나를 깨웁니다.
- `.notify_all()`: `.wait()`으로 대기 중인 모든 스레드를 깨웁니다.

### 5.2. 사용 예시

```c++
std::atomic<bool> is_ready = false;

// Thread 1: 작업 완료 대기
void worker()
{
    // is_ready가 false인 동안 대기
    is_ready.wait(false); 
    
    // 알림을 받은 후 작업 시작
    process_data();
}

// Thread 2: 작업 완료 알림
void master()
{
    prepare_data();
    is_ready.store(true);
    is_ready.notify_one(); // 대기 중인 스레드에게 신호
}
```
