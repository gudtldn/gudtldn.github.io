---
title: "디스코드 봇 만들기 #4 - 빗금(슬래시) 명령어 만들기"

categories:
  - Discord-py2.0_Bot
tags:
  - [python, discord-py, discord bot]

toc: true
toc_sticky: true

date: 2023-03-16
last_modified_at: 2024-04-14
---

## 1. 빗금 명령어 준비하기

코드 최상단에 아래 코드를 입력해 주세요.

```py
from discord import app_commands
```

그다음으로 디스코드 앱 명령어는 글로벌 명령어와 길드 명령어가 존재하는데, 빠른 개발을 위해 개발을 하는 동안에는 길드 명령어로 사용을 해보겠습니다.

먼저 디스코드에서 개발자 모드를 켜야 합니다. 아래와 같이 디스코드 사용자 설정으로 들어가 **고급** 버튼을 누르고 개발자 모드를 켜 주세요.

![디코 사용자 설정 화면](/assets/img/Discord-py_Bot/2023-03-16-discord-py_bot_4/1.1.png)

그리고 빗금 명령어를 사용하고 싶은 서버를 마우스 오른쪽 클릭을 하고 `ID 복사하기` 버튼을 눌러주세요.

![서버 id 복사하는 사진](/assets/img/Discord-py_Bot/2023-03-16-discord-py_bot_4/1.2.png)

다시 코드 에디터로 돌아와서 복사한 서버 아이디를 아래와 같이 코드 상단에다가 입력해 주세요.

```py
GUILDS = [discord.Object(id) for id in (복사한 서버 ID,)]
```

> 만약 서버가 여러개라면 튜플 뒤에 더 입력하시면 됩니다.<br>
> 예: (서버 ID1, 서버 ID2, 서버 ID3, ...)

<br>

그리고 [디스코드 봇 만들기 #2](https://gudtldn.github.io/posts/discord_bot_2/#3-봇-온라인으로-만들기){:target="_blank"} 에서 작성했던 **Bot** 클래스에 아래와 같은 코드를 추가해 주세요.

```py
async def setup_hook(self):
    for guild in GUILDS:
        await self.tree.sync(guild=guild)
```

<table>
  <tr>
    <th> setup_hook() </th>
    <td> 봇이 온라인이 되기 전 호출되는 함수 </td>
  </tr>
  <tr>
    <th> tree.sync() </th>
    <td> 앱 커멘드를 동기화 하는 함수 </td>
  </tr>
</table>

<br>

## 2. 명령어 만들기

적당한 위치에 아래와 같은 코드를 입력해 주세요.

```py
@bot.tree.command(
    name="안녕",
    description="봇이 인사를 합니다.",
    guilds=GUILDS
)
async def app_send_hello(interaction: discord.Interaction):
    await interaction.response.send_message("안녕하세요!")
```

<table>
  <tr>
    <th> name </th>
    <td> 명령어 이름 </td>
  </tr>
  <tr>
    <th> description </th>
    <td> 명령어 설명 </td>
  </tr>
  <tr>
    <th> guilds </th>
    <td> 길드명령어로 사용할 서버 목록 </td>
  </tr>
</table>

그리고 **discord_bot.py** 파일을 실행하고 서버에 가보면 아래 사진과 같은 모습을 볼 수 있습니다.

![명령어 목록 사진](/assets/img/Discord-py_Bot/2023-03-16-discord-py_bot_4/2.1.png)
![명령어 실행 결과 사진](/assets/img/Discord-py_Bot/2023-03-16-discord-py_bot_4/2.2.png)


## 3. 명령어에 매개변수 추가하기

명령어에 매개변수를 추가하려면 아래와 같이 하면 됩니다.

```py
@bot.tree.command(
    name="매개변수",
    description="매개변수 테스트 명령어 입니다.",
    guilds=GUILDS
)
@app_commands.rename(
    arg1="arg1",
    arg2="인수2",
    arg3="인수3"
)
@app_commands.describe(
    arg1="문자열을 입력해 주세요.",
    arg2="정수를 입력해 주세요.",
    arg3="논리값을 입력해 주세요."
)
async def app_args_test(interaction: discord.Interaction, arg1: str, arg2: int, arg3: Optional[bool] = None):
    await interaction.response.send_message(f"arg1: {arg1}\narg2: {arg2}\narg3: {arg3}")
```

이렇게 입력하고 봇을 실행하면 아래와 같은 결과를 볼 수 있습니다.

![명령어 목록 사진](/assets/img/Discord-py_Bot/2023-03-16-discord-py_bot_4/3.1.png)
![매개변수 명령어 입력 사진](/assets/img/Discord-py_Bot/2023-03-16-discord-py_bot_4/3.2.png)
![매개변수 명령어 결과 사진](/assets/img/Discord-py_Bot/2023-03-16-discord-py_bot_4/3.3.png)

## 4. 명령어 예외처리 하기

빗금 명령어도 일반 명령어와 같이 `.error` 데코레이터를 통해 예외처리를 할 수 있습니다.

```py
@bot.tree.command(
    name="예외",
    description="예외를 발생시킵니다.",
    guilds=GUILDS
)
async def app_exception(interaction: discord.Interaction):
    raise Exception("예외발생!")

@app_exception.error
async def app_exception_error(interaction: discord.Interaction, error):
    await interaction.response.send_message(f"에러가 발생하였습니다.\n```{error}```")
```

![예외 발생 결과 사진](/assets/img/Discord-py_Bot/2023-03-16-discord-py_bot_4/4.1.png)

<br>

## 5. 마무리..

이번 포스팅에서는 빗금(슬래시) 명령어에 대해서 알아보았습니다.<br>
다음 포스팅에서는 하이브리드 명령어와, **Context Menu** 에 대해 알아보겠습니다.

이상 제 포스팅을 봐 주셔서 감사합니다.

소스코드: [소스코드로 이동하기](https://github.com/gudtldn/discord-bot_tutorial/tree/discord-bot_tutorial_4 "소스코드로 이동"){:target="_blank"}

<br>

---

> 질문과 오타, 보완했으면 하는 곳은 댓글로 알려주시면 감사하겠습니다.