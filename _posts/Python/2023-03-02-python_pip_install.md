---
title: "명령 프롬프트에서 pip가 안될 때"

categories:
  - Python
  - pip
tags:
  - [python, pip]

toc: true
toc_sticky: true

date: 2023-03-02
# last_modified_at: 2023-03-02
---

## 1. 에러

명령 프롬프트에 `pip` 를 입력하면 아래 사진과 같이 `'pip'은(는) 내부 또는 외부 명령, 실행할 수 있는 프로그램, 또는
배치 파일이 아닙니다.` 라고 뜹니다.

![cmd pip 없음 사진](/assets/img/Python/2023-03-02-python_pip_install/1.1.png)

<br>

## 2. 원인

여러 가지 원인이 있겠지만 주로 아래 항목과 같은 경우가 많습니다.

1. 시스템 환경 변수에 `pip` 의 경로를 추가하지 않음.
2. `pip install --upgrade` 친 후 `pip` 이 제거됨.

<br>

## 2.1. 해결방법 #1

1번의 해결방법은

먼저 python의 설치 경로를 알아야 합니다.

윈도우 검색창에 **python** 을 적고 **python 3.x** 밑에 `파일 위치 열기` 버튼을 클릭합니다.

![python 검색사진](/assets/img/Python/2023-03-02-python_pip_install/2.1.1.png)

그러면 다음과 같은 창이 뜨는데,

![python 바로가기 폴더 사진](/assets/img/Python/2023-03-02-python_pip_install/2.1.2.png)

여기서 한번 더 `파일 위치 열기` 를 클릭합니다.

![python이 있는 폴더 사진](/assets/img/Python/2023-03-02-python_pip_install/2.1.3.png)

위와 같은 창이 나오면, 폴더의 경로를 복사합니다.

다시 윈도우 검색창으로 가서 **path** 를 적고 **시스템 환경 번수 편집** 을 클릭합니다.

![path 검색 사진](/assets/img/Python/2023-03-02-python_pip_install/2.1.4.png)

그러면 아래와 같은 창이 뜨는데,

![시스템 속성 창 사진](/assets/img/Python/2023-03-02-python_pip_install/2.1.5.png)

여기서 우측 하단에 있는 `환경 변수` 를 누르고,

![환경 변수창 사진](/assets/img/Python/2023-03-02-python_pip_install/2.1.6.png)

하단에 **시스템 변수** 카테고리에서 변수명이 **Path** 인 값을 찾아 `편집` 을 클릭합니다.

![환경 변수 편집창 사진](/assets/img/Python/2023-03-02-python_pip_install/2.1.7.png)

여기서 `새로 만들기` 버튼을 클릭하고, 아까 복사했던 **"python이 설치된 경로"/** 와 **"python이 설치된 경로"/Scripts** 를 붙여넣고 확인을 눌러 설정을 마칩니다.

<br>

## 2.2. 해결방법 #2

2번 같은경우에는 **pip** 파일이 없어졌을 가능성이 큽니다.<br>
따라서 아래 명령어를 명령 프롬프트에다가 치면 복구할 수 있습니다.

```bash
> curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
> python get-pip.py
```

위 명령어는 `https://bootstrap.pypa.io/get-pip.py` 에서 python코드를 다운하고, 컴퓨터에 설치된 python으로 실행해서 복구하는 명령어 입니다.

<br>

---

> 질문과 오타, 보완했으면 하는 곳은 댓글로 알려주시면 감사하겠습니다.