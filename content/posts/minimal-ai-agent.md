+++
date = '2026-07-24T17:20:00+08:00'
draft = false
title = '从零理解：用几十行代码搭一个最小 AI Agent'
tags = ["AI", "Agent", "Python"]
categories = ["学习"]
summary = "用中文拆解 Minimal AI agent 教程：Agent 本质是循环，核心只有调用模型、解析动作、执行命令三步。"
+++

> 本文是对 [Minimal AI agent tutorial](https://minimal-agent.com/)（作者 Kilian Lieret 等）的中文理解与讲解，不是逐字翻译。原教程对应的研究实现是 [mini-swe-agent](https://github.com/SWE-agent/mini-swe-agent)，在 SWE-bench 等场景里成绩很强，说明「简单循环」并不等于「玩具」。

## 先抓本质：Agent 就是一个大循环

很多人以为 Agent 很复杂，其实从顶层看就三件事不断重复：

1. 把当前对话历史发给语言模型（LM）
2. 从模型回复里取出「要执行的动作」
3. 真正执行这个动作，把输出再塞回对话里

伪代码可以写成：

```python
messages = [{"role": "user", "content": "帮我修 main.py 里的 ValueError"}]
while True:
    lm_output = query_lm(messages)
    messages.append({"role": "assistant", "content": lm_output})
    action = parse_action(lm_output)
    if action == "exit":
        break
    output = execute_action(action)
    messages.append({"role": "user", "content": output})
```

`messages` 是整段对话记忆。模型每次只看这份列表，所以「它记得做过什么」靠的是你不断往里追加，而不是模型自己有神秘长期记忆。

`role` 常见三种：

| role | 含义 |
|------|------|
| `system` | 行为规则、输出格式约定 |
| `user` | 人类指令，或「环境执行结果」回传给模型 |
| `assistant` | 模型自己的回复 |

注意：命令执行结果通常也用 `user`（或等价角色）塞回去，因为对模型来说，这是「外界反馈」，不是它自己说的话。

要实现这个循环，你只需要补齐三块：**查模型、解析动作、执行动作**。

## 第一步：调用语言模型

原教程支持 OpenAI、Anthropic、OpenRouter、LiteLLM、智谱 GLM 等。若不想绑死某一家，用 LiteLLM 最省事：

```python
from litellm import completion

def query_lm(messages: list[dict[str, str]]) -> str:
    response = completion(
        model="openai/gpt-4o-mini",  # 按你实际可用的模型改
        messages=messages,
    )
    return response.choices[0].message.content
```

密钥不要写死在代码里，用环境变量，例如 PowerShell：

```powershell
$env:OPENAI_API_KEY="你的密钥"
```

快速自测：

```python
print(query_lm([{"role": "user", "content": "掷一个 d20"}]))
```

能正常返回文字，说明调用链路通了。

## 第二步：从回复里解析「动作」

你可以让模型用原生 tool calling，但教程故意用更通用的做法：让模型把 bash 命令包在固定格式里，再用正则提取。

推荐用 Markdown 风格：

````text
我先看看当前目录有什么文件。

```bash-action
ls -la
```
````

解析函数：

```python
import re

def parse_action(lm_output: str) -> str:
    matches = re.findall(
        r"```bash-action\s*\n(.*?)\n```",
        lm_output,
        re.DOTALL,
    )
    return matches[0].strip() if matches else ""
```

也可以用 XML：`<bash_action>...</bash_action>`。多数强模型两种都行；小模型/开源模型若经常格式翻车，可以两种都试。

正则里几个关键点：

- `.*?`：非贪婪，匹配到第一个结束标记就停
- `re.DOTALL`：让 `.` 也能跨行，这样才能解析多行命令

## 第三步：执行动作

最直接的方式是用 `subprocess` 在终端跑命令，并把 stdout/stderr 一起收回来：

```python
import subprocess
import os

def execute_action(command: str) -> str:
    result = subprocess.run(
        command,
        shell=True,
        text=True,
        env=os.environ,
        encoding="utf-8",
        errors="replace",
        stdout=subprocess.PIPE,
        stderr=subprocess.STDOUT,
        timeout=30,
    )
    return result.stdout
```

这样写有两个「看起来像缺陷」的限制：

1. 子进程里的 `cd` 不会改变父进程当前目录
2. `export` 之类环境变量也很难跨命令持久化

教程的观点很有意思：**这些限制在实践里往往不致命**。逼模型多用绝对路径、少依赖隐藏状态，反而更清晰。Claude Code 一类产品也有类似「子 shell 执行、环境变量不跨步持久」的思路。

> 安全提醒：`shell=True` + 让模型自由拼命令，等于把本机 shell 交给模型。本地实验可以，生产环境务必隔离（Docker、沙箱、权限收紧）。

## 系统提示：告诉模型「怎么行动」

循环再对，若模型不知道输出格式，它就会瞎聊。所以开头要有 `system` 消息，例如：

```python
messages = [{
    "role": "system",
    "content": (
        "你是助手。要执行命令时，必须包在 "
        "```bash-action\\n...\\n``` 中。"
        "任务完成时执行 exit。"
    ),
}, {
    "role": "user",
    "content": "列出当前目录文件",
}]
```

## 拼起来：最小可运行版本

下面是「LiteLLM + bash-action」的最小骨架（约几十行）：

```python
import re
import subprocess
import os
from litellm import completion

def query_lm(messages):
    response = completion(model="openai/gpt-4o-mini", messages=messages)
    return response.choices[0].message.content

def parse_action(lm_output: str) -> str:
    matches = re.findall(r"```bash-action\s*\n(.*?)\n```", lm_output, re.DOTALL)
    return matches[0].strip() if matches else ""

def execute_action(command: str) -> str:
    result = subprocess.run(
        command,
        shell=True,
        text=True,
        env=os.environ,
        encoding="utf-8",
        errors="replace",
        stdout=subprocess.PIPE,
        stderr=subprocess.STDOUT,
        timeout=30,
    )
    return result.stdout

messages = [{
    "role": "system",
    "content": "你是助手。执行命令时用 ```bash-action\\n...\\n```。结束时运行 exit。",
}, {
    "role": "user",
    "content": "列出当前目录文件",
}]

while True:
    lm_output = query_lm(messages)
    print("LM:", lm_output)
    messages.append({"role": "assistant", "content": lm_output})
    action = parse_action(lm_output)
    print("Action:", action)
    if action == "exit":
        break
    output = execute_action(action)
    print("Output:", output)
    messages.append({"role": "user", "content": output})
```

跑通这一版，你就已经有了一个「会看终端、会改思路」的最小 Agent。

## 让它更稳：别让异常直接把循环打死

最小版容易卡死：超时、格式错误、模型一次输出多个动作……教程的核心补丁是：

**已知错误不要立刻崩溃，把错误信息当反馈塞回 `messages`，让模型自己纠错。**

```python
while True:
    try:
        # query -> parse -> execute
        ...
    except Exception as e:
        messages.append({"role": "user", "content": str(e)})
```

再进一步可以分层：

- `FormatError`：没找到恰好 1 个动作，附上「正确格式示例」提醒模型
- `OurTimeoutError`：超时后告诉它「别开 vim 这类交互程序」
- `NonterminatingException`：可恢复，继续循环
- `TerminatingException`：优雅退出（比单纯 `if action == "exit"` 更好扩展）

另外再给命令环境加一些「反卡住」变量，例如：

```python
env_vars = {
    "PAGER": "cat",
    "MANPAGER": "cat",
    "LESS": "-R",
    "PIP_PROGRESS_BAR": "off",
    "TQDM_DISABLE": "1",
}
```

避免 `man`、`pip`、进度条之类进入交互/刷屏模式，把 Agent 挂死。

## 从教程到工程：mini-swe-agent

原教程对应的工程实现 [mini-swe-agent](https://github.com/SWE-agent/mini-swe-agent) 基本就是同一套蓝图，只是拆成模块：

| 组件 | 职责 |
|------|------|
| `Agent` | 大 `while` 循环（`run`） |
| `Model` | 对接不同 LM |
| `Environment` | 执行动作（本地 `subprocess` 或 `docker exec`） |

换 Docker 环境听起来唬人，实质往往只是把 `subprocess.run(...)` 换成 `docker exec ...`。隔离性更好，思路不变。

这也是这篇教程最有说服力的地方：**强 Agent 不一定靠复杂框架，先把「观察-行动-反馈」循环做对，再逐步加容错。**

## 读完你可以带走什么

1. Agent ≠ 魔法框架，核心是 `messages` 上的闭环。
2. 最小实现只要三函数：`query_lm` / `parse_action` / `execute_action`。
3. 系统提示负责约束「动作编码格式」。
4. 鲁棒性主要来自：把异常变成可消费的反馈，而不是堆更多库。
5. 本地能跑后，再用 Docker/沙箱考虑安全与可复现。

若你想继续动手，建议顺序是：先跑通上面最小循环 → 故意制造格式错误/超时看它能否自救 → 再去读 mini-swe-agent 源码对照模块划分。

原文与更多细节：[https://minimal-agent.com/](https://minimal-agent.com/)
