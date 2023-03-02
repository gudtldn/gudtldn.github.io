---
title: "디스코드 봇 만들기 #2 - 봇 온라인으로 만들어보기"

categories:
  - Discord-py_Bot
tags:
  - [python, discord-py, discord bot]

toc: true
toc_sticky: true

date: 2023-03-02
# last_modified_at: 2023-03-02
---

> 시작하기 전
>- python3.8 이상 3.10 이하의 버전이 설치되어 있어야 합니다.
>- python의 기본적인 문법은 알고 있어야 합니다.

## 1. discord.py 모듈 설치하기

명령 프롬프트에서 아래 명령어를 입력해 모듈을 설치해 주세요.

```bash
> pip install discord.py
```

![윈도우 검색창에 cmd 그림과 명령어 입력한 그림](/assets/img/Discord-py_Bot/2023-03-02-discord-py_bot_2/1.1.png)

> 만약 pip 명령어가 없다고 뜨면 [이 포스팅](https://gudtldn.github.io/posts/python_pip_install/){:target="_blank"} 을 참조해 주세요.

설치가 완료되면 명령 프롬프트창을 닫으셔도 됩니다.

<br>

## 2. 봇 프로젝트 폴더 만들기

먼저 작업할 컴퓨터에 디스코드 봇 프로젝트를 담을 폴더를 새로 하나 만들어 주세요.

![프로젝트 폴더 아이콘 사진](/assets/img/Discord-py_Bot/2023-03-02-discord-py_bot_2/2.1.png)

그다음으로 폴더 안에 `discord_bot.py` 파일과 봇 토큰을 담을 `token.txt` 파일을 만들어 주세요.

> 참고
>- 파일 이름은 아무렇게나 지어도 됩니다.

![폴더 내부 파일 사진](/assets/img/Discord-py_Bot/2023-03-02-discord-py_bot_2/2.2.png)

<br>

## 3. 봇 온라인으로 만들기

discord.py로 봇을 만드는 방법에는 여러가지가 있지만, 이 포스팅에서는 `command.Bot` 을 상속받아서 봇을 만들도록 하겠습니다.


`token.txt` 를 에디터로 열고 지난 포스팅 에서 복사해둔 TOKEN을 붙여넣거나 [Discord Developer Portal](https://discord.com/developers/applications "Discord Developer Portal"){:target="_blank"} 에서 좌측 **SETTING** 메뉴중 **BOT** 카테고리를 선택하고, 중앙에 있는 **Reset Token** 버튼을 눌러 새로운 토큰을 복사해 붙여넣어 주세요.

![Application BOT 사진](/assets/img/Discord-py_Bot/2023-03-02-discord-py_bot_2/3.1.png)
![token.txt 메모장 사진](/assets/img/Discord-py_Bot/2023-03-02-discord-py_bot_2/3.2.png)

그다음 `discord_bot.py` 를 에디터로 열고 아래와 같은 코드를 적어주세요.

```py
import discord
from discord.ext import commands

class Bot(commands.Bot):
    def __init__(self):
        super().__init__(
            command_prefix=".",
            intents=discord.Intents.all()
        )

    async def on_ready(self):
        print(f"{self.user.name}으로 로그인")
        
        await self.change_presence(
            status=discord.Status.online,
            activity=discord.Game("봇 테스트")
        )

bot = Bot()
with open("./token.txt", "r") as fr:
    bot.run(fr.read())
```

파일을 저장하고 `discord_bot.py` 파일을 실행하면 드디어 봇이 온라인이 됩니다.

![봇 온라인 사진](/assets/img/Discord-py_Bot/2023-03-02-discord-py_bot_2/3.3.png)

<br>

## 3.1. 코드 설명

> 1 ~ 2번째 줄
> 
> ```py
> import discord
> from discord.ext import commands
> ```
> 
> discord.py 사용을 위한 모듈을 불러옵니다.

<br>

> 4번째 줄
> 
> ```py
> class Bot(commands.Bot):
> ```
> 
> `commands.Bot` 를 상속받은 `Bot` 클래스를 새로 만듭니다.

<br>

> 5 ~ 9번째 줄
> 
> ```py
> def __init__(self):
>     super().__init__(
>         command_prefix=".",
>         intents=discord.Intents.all()
>     )
> ```
> 
> `Bot` 클래스의 생성자가 실행이 될 때 부모클래스의 생성자도 실행합니다.
> 
> <table>
>   <tr>
>     <th> command_prefix </th>
>     <td> 봇의 접두사 </td>
>   </tr>
>   <tr>
>     <th> intents </th>
>     <td> 봇의 의도 설정 </td>
>   </tr>
> </table>

<br>

> 11 ~ 17번째 줄
> 
> ```py
> async def on_ready(self):
>     print(f"{self.user.name}으로 로그인")
>     
>     await self.change_presence(
>         status=discord.Status.online,
>         activity=discord.Game("봇 테스트")
>     )
> ```
> 
> [비동기 함수]<br>
> 봇이 켜졌을 때 자동으로 실행됩니다.
> - `{자신의 봇 이름}으로 로그인` 이 출력됩니다.
> - 봇을 `온라인`으로 바꾸고, 사용자 지정상태를 `봇 테스트` 로 설정합니다.

<br>

> 19번째 줄
> 
> ```py
> bot = Bot()
> ```
> 
> 변수 `bot`을 `Bot`으로 초기화 합니다.

<br>

> 20 ~ 21번째 줄
> 
> ```py
> with open("./token.txt", "r") as fr:
>     bot.run(fr.read())
> ```
> 
> `token.txt` 파일에서 토큰을 읽고, 봇을 실행합니다.

<br>

## 4. 마무리..

이번 포스팅에서는 봇을 켜는 것까지 알아봤습니다.<br>
다음 포스팅은 봇 명령어를 만드는 방법에 대해 알아보겠습니다.

이상 제 포스팅을 봐 주셔서 감사합니다.

<br>

---

> 질문과 오타, 보완했으면 하는 곳은 댓글로 알려주시면 감사하겠습니다.