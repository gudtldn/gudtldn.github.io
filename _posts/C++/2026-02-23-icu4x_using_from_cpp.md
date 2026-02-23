---
title: "CMake를 활용한 ICU4X C++ 환경 구축"
image: https://icu4x.unicode.org/_astro/icon_glow.DlSPCPuh_iekJd.webp
categories:
  - C++
tags:
  - [c++, rust, ffi, cmake]

toc: true
toc_sticky: true

date: 2026-02-23
last_modified_at: 2026-02-23
---

## 1. 개요

ICU4X는 Rust로 작성된 국제화 라이브러리로, C++에서도 사용할 수 있도록 FFI(외부 함수 인터페이스)를 제공합니다. 이번 글에서는 CMake를 활용하여 ICU4X를 C++ 프로젝트에 통합하는 방법을 설명하겠습니다.

## 2. 사전 준비 사항

1. [Rust Toolchain](https://www.rust-lang.org/tools/install){:target="_blank"} 설치: ICU4X 핵심 로직 컴파일을 위한 `cargo` 및 `rustc`가 필요합니다.
2. C++ 컴파일러 설치: C++17 이상을 지원하는 컴파일러가 필요합니다. (GCC, Clang, 또는 MSVC)
3. [CMake](https://cmake.org/download/){:target="_blank"} 설치: 프로젝트 빌드 시스템으로 CMake를 사용합니다.
4. [git](https://git-scm.com/downloads){:target="_blank"} 설치: CMake에서 `FetchContent` 모듈을 사용하여 ICU4X 소스를 가져올 때 필요합니다.

## 3. CMakeLists.txt 설정

### 3.1. `FetchContent`를 사용하여 ICU4X 소스 가져오기

> 아래 Tag는 당시 레포지토리의 최신 릴리즈 버전으로 작성되었습니다.

```cmake
# =====================================================================
# FetchContent 설정 (의존성 자동 다운로드)
# =====================================================================
include(FetchContent)

# Corrosion (Rust 빌드 연동 모듈) 가져오기
FetchContent_Declare(
    Corrosion
    GIT_REPOSITORY https://github.com/corrosion-rs/corrosion.git
    GIT_TAG        v0.6.1
    GIT_SHALLOW    TRUE
)

# ICU4X 소스코드 가져오기
FetchContent_Declare(
    icu4x_src
    GIT_REPOSITORY https://github.com/unicode-org/icu4x.git
    GIT_TAG        icu@2.1.0
    GIT_SHALLOW    TRUE
)

FetchContent_MakeAvailable(Corrosion icu4x_src)
```

### 3.2. Corrosion을 사용하여 ICU4X 빌드 및 C++ 바인딩 생성

```cmake
# =====================================================================
# ICU4X Crate(Cargo.toml)를 CMake 타겟으로 가져오기
# Usage: https://corrosion-rs.github.io/corrosion/usage.html
# =====================================================================
corrosion_import_crate(
    MANIFEST_PATH "${icu4x_src_SOURCE_DIR}/ffi/capi/Cargo.toml"
    OVERRIDE_CRATE_TYPE icu_capi=staticlib
)
```

{: .prompt-info }
> ICU4X의 `ffi/capi/Cargo.toml`에는 빌드 결과물 형식을 정의하는 [lib] 섹션이나 crate-type 명시가 빠져 있습니다. Rust는 기본적으로 `rlib` 형식으로 빌드하기 때문에 C++에서 정상적으로 링크를 하려면 `staticlib` 또는 `cdylib` 형식으로 빌드하도록 명시해야 합니다.

### 3.3. C++ 타겟 설정 및 Corrosion 타겟 링크

```cmake
# 빌드된 Rust 라이브러리(icu_capi)를 `my_test_target`에 링킹
target_link_libraries(my_test_target PRIVATE
    icu_capi
)

# C++ 바인딩 헤더 경로 지정
target_include_directories(my_test_target PRIVATE
    "${icu4x_src_SOURCE_DIR}/ffi/capi/bindings/cpp"
)
```

## 4. C++ 코드 작성

> 아래는 ICU4X C++ FFI를 활용하여 숫자 포맷팅을 수행하는 간단한 예제입니다. ([원본](https://icu4x.unicode.org/2_1/cppdoc/#autotoc_md2){:target="_blank"})

```cpp
#include <icu4x/DecimalFormatter.hpp>
#include <icu4x/Logger.hpp>

#include <iostream>

int main()
{
    using namespace icu4x;

    // For basic logging
    Logger::init_simple_logger();

    // Create a locale object representing Bangla
    const std::unique_ptr<Locale> locale = Locale::from_string("bn").ok().value();

    std::cout << "Running test for locale " << locale->to_string() << std::endl;

    // Create a formatter object with the appropriate settings
    const std::unique_ptr<DecimalFormatter> formatter =
        DecimalFormatter::create_with_grouping_strategy(*locale, DecimalGroupingStrategy::Auto).ok().value();

    // Create a decimal representing the number 1,000,007
    const std::unique_ptr<Decimal> decimal = Decimal::from(1000007);

    // Format it to a string
    std::string out = formatter->format(*decimal);

    // Report formatted value
    std::cout << "Formatted value is " << out << std::endl;
    if (out != "১০,০০,০০৭")
    {
        std::cerr << "Output does not match expected output" << std::endl;
        return 1;
    }

    return 0;
}
```

## 5. 빌드 및 실행

1. CMake 프로젝트 디렉토리에서 빌드 디렉토리를 생성하고 이동합니다.

```bash
mkdir build && cd build

cmake ..
cmake --build .
```

## 6. 전체 예제 코드 (Repository)

위 과정이 모두 적용된 전체 소스 코드는 [여기](https://github.com/gudtldn/icu4x_ffi_test){:target="_blank"}에서 확인하실 수 있습니다.

---

## 참고 문서

- [Unicode ICU4X Repository][icu4x-repository]{:target="_blank"}
- [ICU4X C++ FFI Documentation][icu4x-cpp-ffi-docs]{:target="_blank"}
- [Corrosion Documentation][corrosion-docs]{:target="_blank"}

[icu4x-repository]: https://github.com/unicode-org/icu4x
[icu4x-cpp-ffi-docs]: https://icu4x.unicode.org/2_1/cppdoc/index.html#autotoc_md1
[corrosion-docs]: https://corrosion-rs.github.io/corrosion/usage.html
