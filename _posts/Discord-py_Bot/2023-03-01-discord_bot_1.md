---
title: "디스코드 봇 만들기 #1 - 봇 생성하기"

categories:
  - Discord-py_Bot
tags:
  - [python, discord-py, discord bot]

toc: true
toc_sticky: true
 
date: 2023-03-02
last_modified_at: 2023-03-02
---

> 시작하기 전
>- discord 계정이 있어야 합니다.
>- 이 포스트는 python언어를 이용해 포스팅 할 예정입니다.

## 1. Discord Developer Portal에서 애플리케이션 생성하기

먼저 [Discord Developer Portal](https://discord.com/developers/applications "Discord Developer Portal"){:target="_blank"} 에 들어가서, 로그인이 되어있지 않다면 로그인을 해 줍니다.

로그인을 하면 밑과 같은 화면이 뜰 텐데요.

![Applications 화면](/assets/img/Discord-py_Bot/2023-03-01-discord-py_bot_1/1.1.png){:style=""}

여기서 우측 상단에 있는 New Application 버튼을 누르면 새 팝업 화면이 뜨는데, **NAME**칸에 원하는 이름으로 지어주세요.

> 참고
>- 이쪽의 **NAME**칸은 봇의 이름이 아닌 애플리케이션의 이름입니다.<br>따라서 다른 봇들과 구분정도만 할 수 있도록 적어주세요. (봇 이름과 같아도 됩니다.)

![New Application 버튼](/assets/img/Discord-py_Bot/2023-03-01-discord-py_bot_1/1.2.png)

Create 버튼을 눌러 애플리케이션을 만듭니다.

<br>

## 2. Application화면에서 봇 추가하기

애플리케이션까지 다 만들었다면 밑과 같은 화면이 반겨줄 겁니다.

![Applications 정보 화면](/assets/img/Discord-py_Bot/2023-03-01-discord-py_bot_1/2.1.png)

여기서 좌측 **SETTING** 메뉴의 밑에 있는 **Bot** 카테고리를 클릭하고

![Applications 봇 추가 화면](/assets/img/Discord-py_Bot/2023-03-01-discord-py_bot_1/2.2.png)

우측의 **Add bot** 버튼을 클릭합니다.
클릭 후 `ADD A BOT TO THIS APP?` 이라는 팝업이 뜨는데, `Yes, do it!` 버튼을 누르면 봇이 만들어집니다. (만약 2차 인증이 있다면 2차 인증 후)

여기까지 따라오셨다면 밑과 같은 화면이 반겨줍니다.

![Applications 봇 정보 화면](/assets/img/Discord-py_Bot/2023-03-01-discord-py_bot_1/2.3.png)

<br>

## 2.1 봇 상세설정

![봇 설정 이미지](/assets/img/Discord-py_Bot/2023-03-01-discord-py_bot_1/2.1.1.png)

#### 봇 정보 설정

<table>
  <tr>
    <th> ICON </th>
    <td> 봇의 프로필 사진 </td>
  </tr>
  <tr>
    <th> USERNAME </th>
    <td> 봇의 닉네임 </td>
  </tr>
</table>

이 둘은 원하는 대로 설정하면 되고, **TOKEN**은 복사해뒀다가 잘 보관해 둡시다.

#### 봇 의도 설정

<table>
  <tr>
    <th> PUBLIC BOT </th>
    <td> 봇의 소유주가 아닌 다른 사람이, 봇 초대링크로 봇을 추가할 수 있는지 </td>
  </tr>
  <tr>
    <th> REQUIRES OAUTH2 CODE GRANT </th>
    <td> 봇을 추가할 때 디스코드 로그인 강제화 </td>
  </tr>
  <tr>
    <th> PRESENCE INTENT </th>
    <td> 서버 멤버의 상태를 수신 (온라인, 오프라인, 자리비움, 방해금지) </td>
  </tr>
  <tr>
    <th> SERVER MEMBERS INTENT </th>
    <td> 서버를 들어가거나 나갈 때 이벤트를 수신 </td>
  </tr>
  <tr>
    <th> MESSAGE CONTENT INTENT </th>
    <td> 서버의 메시지 내용을 수신 </td>
  </tr>
</table>

여기서 `PUBLIC BOT` 과 `REQUIRES OAUTH2 CODE GRANT` 는 꺼두고, `PRESENCE INTENT`, `SERVER MEMBERS INTENT`, `MESSAGE CONTENT INTENT` 는 켜 주세요.

> 나중에 봇을 어느정도 개발하고 사용하지 않는 기능은 꺼두시는걸 추천합니다.

<br>

## 3. 서버에 봇 추가하기

> 이 목차를 따라하기 전
>- 봇을 테스트 할 서버를 준비해 주세요.

<br>

![변경 후 Applications 봇 정보 화면](/assets/img/Discord-py_Bot/2023-03-01-discord-py_bot_1/3.1.png)

여기서 다시 좌측 **SETTING** 메뉴의 밑에 있는 **OAuth2** 카테고리를 클릭하고, **URL Generator** 를 클릭합니다.

그러면 밑과 같은 화면이 뜨는데요.

![OAuth2 화면](/assets/img/Discord-py_Bot/2023-03-01-discord-py_bot_1/3.2.png)

여기서 봇의 기능을 위한 `bot` 과 추후 설명 할 슬래시(빗금) 명령어를 사용하기 위해 `applications.commands` 를 선택합니다.

그리고 개발 후 테스트 도중 권한에 대한 문제가 없도록, `Adminstrator` 권한을 선택해 관리자 권한으로 초대해 줍니다.

> 나중에 봇을 공개로 돌려 초대할 때, 권한을 조정해 링크를 공유해 주세요.

이렇게 하고 페이지 맨 밑으로 가 보면 밑과 같은 링크가 있습니다.

![OAuth2 링크](/assets/img/Discord-py_Bot/2023-03-01-discord-py_bot_1/3.3.png)

이 링크를 복사해 새 탭의 주소창에 입력 해 주세요.

![봇 초대 화면1](/assets/img/Discord-py_Bot/2023-03-01-discord-py_bot_1/3.4.png)

여기서 봇을 초대하고자 하는 서버를 선택한 후 **계속하기** 버튼을 클릭, **승인** 버튼도 클릭한 뒤 캡차인증을 하면, 드디어 봇 추가가 완료됩니다.

<br>

## 4. 마무리..

제 부족한 글을 읽으며, 여기까지 오느라 수고 많으셨습니다.
다음 편은 `pip`으로 `discord-py` 모듈 설치와 간단하게 봇을 온라인으로 만들어 보겠습니다.

---

질문과 틀린곳, 보완했으면 하는것이 있으면 댓글로 알려주세요.