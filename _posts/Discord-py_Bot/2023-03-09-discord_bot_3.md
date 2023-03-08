---
title: "디스코드 봇 만들기 #3 - 봇 명령어 만들기"

categories:
  - Discord-py2.0_Bot
tags:
  - [python, discord-py, discord bot]

toc: true
toc_sticky: true

date: 2023-03-09
# last_modified_at: 2023-03-09
---

## 1. 간단한 봇 답장기능 만들기

지난 포스트에 썼던 코드에서 아래와 같이 코드를 추가해 주세요.

```py
bot = Bot()

@bot.command(name="안녕", aliases=["반가워"])
async def send_hello(ctx: commands.Context):
    await ctx.send("안녕하세요!")

with open("./token.txt", "r") as fr:
    bot.run(fr.read())
```

이렇게 하면 사용자가 `.안녕` 또는 `.반가워` 라고 보냈을 때 `send_hello` 함수가 호출이 되어, 봇이 `안녕하세요!` 를 보내게 됩니다.

![봇 인사 동작 사진](/assets/img/Discord-py_Bot/2023-03-08-discord-py_bot_3/1.1.png)

<br>

## 1.1. 코드 설명

> 3번째 줄
> 
> ```py
> @bot.command(name="안녕", aliases=["반가워"])
> ```
> 
> <table>
>   <tr>
>     <th> name </th>
>     <td> 명령어 이름 </td>
>   </tr>
>   <tr>
>     <th> aliases </th>
>     <td> 추가로 사용할 명령어 이름 </td>
>   </tr>
> </table>
> 
> >1. 만약 `name` 를 생략할 경우 함수의 이름이 명령어 이름이 됩니다.<br>
> >2. `aliases` 는 생략 가능합니다.
> 
> 사용자가 `.안녕` 또는 `.반가워` 를 치면, 이 코드 밑에 올 함수를 실행합니다.

<br>

> 4 ~ 5번째 줄
> 
> ```py
> async def send_hello(ctx: commands.Context):
>     await ctx.send("안녕하세요!")
> ```
> 
<!-- > <table>
>   <tr>
>     <th> ctx </th>
>     <td> 여러가지 정보를 가지고 있는 변수 </td>
>   </tr>
> </table> -->
> 
> `send_hello` 함수를 정의합니다.<br>
> `await ctx.send` 로 `안녕하세요!` 메시지를 보냅니다.

<br>

## 2. 명령어에 추가 인자 받아오기

명령어를 입력할 때 인자를 받아오는 방법에는 여러가지가 있습니다.

인자값을 정해진 갯수 만큼만 받고 싶을 때는 아래와 같이, 함수에 매개변수를 추가해 주면 됩니다.

```py
@bot.command(name="테스트")
async def args_test(ctx: commands.Context, arg1: str, arg2: str, arg3: str):
    await ctx.send(f"첫번째 값: {arg1}\n두번째 값: {arg2}\n세번째 값: {arg3}")
```

![인자 3개를 받아와 출력하는 사진](/assets/img/Discord-py_Bot/2023-03-08-discord-py_bot_3/2.1.png)

이와 같이 값이 3개가 받아지는걸 확인할 수 있습니다.

<br>

인자값을 여러개를 받고 싶을 때에는 아래와 같이, 함수에 `*변수이름` 으로 추가해 주면 됩니다.<br>
이렇게 하면 값을 튜플로 가져오게 됩니다.

```py
@bot.command(name="테스트")
async def args_test(ctx: commands.Context, *args: str):
    await ctx.send(f"{', '.join(args)}\n총합 {len(args)}개")
```

![인자 여러개를 받아와 출력하는 사진](/assets/img/Discord-py_Bot/2023-03-08-discord-py_bot_3/2.2.png)

이와 같이 여러 개가 받아지는 걸 확인할 수 있습니다.

<br>

만약 인자 값 여러 개를 튜플이 아닌 하나의 문자열로 가져오고 싶을 때에는 아래와 같이 하면 됩니다.

```py
@bot.command(name="테스트")
async def args_test(ctx: commands.Context, *, args: str):
    await ctx.send(f"입력된 내용: {args}")
```

![하나의 텍스트로 받아오는 사진](/assets/img/Discord-py_Bot/2023-03-08-discord-py_bot_3/2.3.png)

이와 같이 하나의 텍스트로 받아지는 걸 확인할 수 있습니다.

<br>

## 3. 매개변수 기본값 설정하기

만약 특정 값을 입력을 안 해도 봇이 작동할 수 있게끔 하려면 아래와 같이 매개변수에 기본값을 지정해 주면 됩니다.

```py
@bot.command(name="기본값")
async def default_value(ctx: commands.Context, arg1: str = None):
    await ctx.send(f"arg의 값: {arg1}")
```

또는

```py
from typing import Optional

@bot.command(name="기본값")
async def default_value(ctx: commands.Context, arg1: Optional[str] = None):
    await ctx.send(f"arg의 값: {arg1}")
```
> 밑 코드에서 `Optional` 이 들어가면 `None` 은 생략 가능합니다.

![기본값 설정후 출력값 사진](/assets/img/Discord-py_Bot/2023-03-08-discord-py_bot_3/3.1.png)

이와 같이 기본값을 설정할 수 있습니다.

<br>

## 4. 타입 어노테이션

타입 어노테이션은 아래 코드와 같이 변수에 타입 힌트를 지정해 주는 것입니다.

```py
string: str
integer: int
boolean: bool
```

이 타입 어노테이션을 사용하면 사용자에게 특정 타입의 값을 받아올 수 있습니다.<br>
아래는 예제입니다.

```py
@bot.command(name="타입")
async def type_annotation(ctx: commands.Context, arg1: str, arg2: int, arg3: float):
    await ctx.send(f"arg1의 타입: {type(arg1)}\narg2의 타입: {type(arg2)}\narg3의 타입: {type(arg3)}")
```

![타입 확인하는 사진](/assets/img/Discord-py_Bot/2023-03-08-discord-py_bot_3/4.1.png)

이와같이 따로 추가 변환없이 특정 타입의 값을 받아올 수 있습니다.

<br>

## 5. 예외처리

봇을 운영하다 보면 원하지 않는 값이 들어올 때가 있는데, 아래 코드와 같이 이에 대한 예외 처리를 할 수 있습니다.

```py
@bot.command(name="예외")
async def exception(ctx: commands.Context, arg: int):
    await ctx.send(f"arg1의 타입: {type(arg)}, 값: {arg}")

@exception.error
async def exception_error(ctx: commands.Context, error):
    await ctx.send(f"예외가 발생하였습니다.\n```{error}```")
```

![예외처리 사진1](/assets/img/Discord-py_Bot/2023-03-08-discord-py_bot_3/5.1.png)

이와같이 명령어를 실행할 함수 이름에 데코레이터를 붙이고, 뒤에 `.error` 를 추가하면 위 명령어에서 발생한 예외를 처리할 수 있습니다.

이 예외를 조금 더 유동적으로 처리하려면 아래와 같이 `isinstance` 함수를 이용하여 예외를 처리할 수 있습니다.

```py
@exception.error
async def exception_error(ctx: commands.Context, error):
    if isinstance(error, commands.errors.MissingRequiredArgument):
        await ctx.send("값을 입력해 주세요.")
    
    elif isinstance(error, commands.errors.BadArgument):
        await ctx.send("숫자만 입력해 주세요.")
    
    else:
        await ctx.send(f"예외가 발생하였습니다.\n```{error}```")
```

![예외처리 사진2](/assets/img/Discord-py_Bot/2023-03-08-discord-py_bot_3/5.2.png)

<br>

## 6. 마무리..

이번 포스팅에서는 명령어를 추가하는 방법에 대해서 알아보았습니다.<br>
다음 포스팅에서는 빗금(슬래시) 명령어에 대해 알아보겠습니다.

이상 제 포스팅을 봐 주셔서 감사합니다.

소스코드: [소스코드로 이동하기](https://github.com/gudtldn/discord-bot_tutorial/tree/discord-bot_tutorial_3 "소스코드로 이동")

<br>

---

> 질문과 오타, 보완했으면 하는 곳은 댓글로 알려주시면 감사하겠습니다.