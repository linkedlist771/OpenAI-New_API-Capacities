---
title: OpenAI_New_API_Capacities
date: 2023-11-27 13:42:35
tags: OpenAI;API

---

> 这个博客主要是关于如何利用`OpenAI` 官方新推出的`API`的功能。 关于最新功能的介绍。对应的项目链接可以参考[GitHub链接](https://github.com/linkedlist771/OpenAI-New_API-Capacities)。
>
> `email`: 213193509seu@gmail.com
>
> `语言框架`: python
>
> `参考链接`: https://platform.openai.com/docs/guides

<!-- more -->

## 配置账号

去openai官方网站配置`OPENAI_API_KEY`。配置完后请把`OPENAI_API_KEY`添加到环境变量。

## 安装库

注意，此版本openai已经更新了新的功能，请使用最新版本的`openai` 库。（推荐在anaconda虚拟环境里面进行安装）

- 配置Python环境

```
conda create -n openai python=3.11 -y
conda activate openai
```



- 安装openai库

```bash
pip install --upgrade openai
```

## 功能

### chat_completion

```python
from openai import OpenAI
client = OpenAI()

response = client.chat.completions.create(
  model="gpt-4-1106-preview",
  messages=[
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "谁获得了2022年英雄联盟S赛世界冠军？"},
  ]
)
print(response)
```



输出:

```bash
ChatCompletion(id='chatcmpl-8PP1sbZKC7D5VSxHsgCOsMPyLFU0Y', choices=[Choice(finish_reason='stop', index=0, message=ChatCompletionMessage(content='2022年英雄联盟世界锦标赛（S赛）的冠军是韩国的DRX战队。他们在决赛中战胜了中国的EDward Gaming（EDG）战队，赢得了冠军头衔。这也是DRX战队第一次夺得世界冠军。', role='assistant', function_call=None, tool_calls=None))], created=1701065104, model='gpt-4-1106-preview', object='chat.completion', system_fingerprint='fp_a24b4d720c', usage=CompletionUsage(completion_tokens=99, prompt_tokens=43, total_tokens=142))
```



### JSON输出（新功能）

> 目前`json`格式输出只支持`gpt-4-1106-preview`和`gpt-3.5-turbo-1106`这两个模型。
>
> 注意：在使用json格式的时候，请确保输入messages中含有`json`这个单词。

```python
from openai import OpenAI
client = OpenAI()

response = client.chat.completions.create(
  model="gpt-3.5-turbo-1106",
  response_format={"type": "json_object"},
  messages=[
    {"role": "system", "content": "You are a helpful assistant designed to output JSON."},
    {"role": "user", "content": "Now I want you to score the sentence 'I like apples' on a scale of 1 to 10 with "
                                "output format as:"
                                "{'score': score}, for example, {'score': 0.5}"}
  ]
)
print(response.choices[0].message.content)
```

输出

```bash
{
  "score": 8
}
```



### 可复现输出

> 在进行`chat-completions`的时候，其实其输出是随机的，但是通过设置seed以及相应指纹可以保证输出相同， 并且保证其他参数也相同。在使用benchmark的时候，设置相同的seed也能保证结果的可靠性。

```python
import asyncio
import openai
import pprint
import difflib
from IPython.display import display, HTML


GPT_MODEL = "gpt-3.5-turbo-1106"
client = openai.OpenAI()

async def get_chat_response(system_message: str, user_request: str, seed: int = None):
    try:
        messages = [
            {"role": "system", "content": system_message},
            {"role": "user", "content": user_request},
        ]

        response = client.chat.completions.create(

            model=GPT_MODEL,
            messages=messages,
            seed=seed,
            max_tokens=200,
            temperature=0.7,
        )

        response_content = response.choices[0].message.content
        system_fingerprint = response.system_fingerprint
        prompt_tokens = response.usage.prompt_tokens
        completion_tokens = (
            response.usage.total_tokens - response.usage.prompt_tokens
        )

        table = f"""
        <table>
        <tr><th>Response</th><td>{response_content}</td></tr>
        <tr><th>System Fingerprint</th><td>{system_fingerprint}</td></tr>
        <tr><th>Number of prompt tokens</th><td>{prompt_tokens}</td></tr>
        <tr><th>Number of completion tokens</th><td>{completion_tokens}</td></tr>
        </table>
        """
        display(HTML(table))

        return response_content
    except Exception as e:
        print(f"An error occurred: {e}")
        return None


# This function compares two responses and displays the differences in a table.
# Deletions are highlighted in red and additions are highlighted in green.
# If no differences are found, it prints "No differences found."


def compare_responses(previous_response: str, response: str):
    d = difflib.Differ()
    diff = d.compare(previous_response.splitlines(), response.splitlines())

    diff_table = "<table>"
    diff_exists = False

    for line in diff:
        if line.startswith("- "):
            diff_table += f"<tr style='color: red;'><td>{line}</td></tr>"
            diff_exists = True
        elif line.startswith("+ "):
            diff_table += f"<tr style='color: green;'><td>{line}</td></tr>"
            diff_exists = True
        else:
            diff_table += f"<tr><td>{line}</td></tr>"

    diff_table += "</table>"

    if diff_exists:
        display(HTML(diff_table))
    else:
        print("No differences found.")
        
topic = "a journey to Mars"
system_message = "You are a helpful assistant that generates short stories."
user_request = f"Generate a short story about {topic}."

previous_response = await get_chat_response(
    system_message=system_message, user_request=user_request
)

response = await get_chat_response(
    system_message=system_message, user_request=user_request
)

# The function compare_responses is then called with the two responses as arguments.
# This function will compare the two responses and display the differences in a table.
# If no differences are found, it will print "No differences found."
compare_responses(previous_response, response)
SEED = 123
response = await get_chat_response(
    system_message=system_message, seed=SEED, user_request=user_request
)
previous_response = response
response = await get_chat_response(
    system_message=system_message, seed=SEED, user_request=user_request
)

compare_responses(previous_response, response)
```



<table><tr style='color: red;'><td>- In the year 2050, a team of astronauts embarked on a groundbreaking journey to Mars. Their mission was to establish the first human settlement on the red planet. As their spaceship, named "Pathfinder," soared through the vast expanse of space, the crew marveled at the breathtaking view of Earth growing smaller in the distance.</td></tr><tr style='color: green;'><td>+ In the year 2050, humanity had finally achieved its long-held dream of reaching the red planet. A team of six brave astronauts embarked on the historic journey to Mars, leaving behind the safety of Earth for the unknown challenges of the cosmos.</td></tr><tr><td>  </td></tr><tr style='color: red;'><td>- After months of traveling, the Pathfinder finally approached Mars, and the astronauts prepared for the daunting task ahead. With their advanced technology and unwavering determination, they landed safely on the Martian surface, greeted by the desolate yet awe-inspiring landscape.</td></tr><tr style='color: green;'><td>+ As their spacecraft hurtled through the vast expanse of space, the astronauts marveled at the stunning beauty of the stars and galaxies outside their windows. They spent their days conducting scientific experiments, maintaining the spacecraft, and preparing for the momentous landing on Mars.</td></tr><tr><td>  </td></tr><tr style='color: red;'><td>- The crew immediately got to work, constructing habitats and developing sustainable systems for food and water. Despite the challenges they faced, their spirit remained unyielding. They were driven by the dream of paving the way for future generations to inhabit and explore the mysteries of Mars.</td></tr><tr style='color: green;'><td>+ After months of travel, the spacecraft finally approached the Martian atmosphere. Tensions ran high as the crew braced themselves for the perilous descent. The captain's steady voice crackled over the intercom, guiding the team through the nail-biting moments as the spacecraft plunged through the thin atmosphere.</td></tr><tr><td>  </td></tr><tr style='color: red;'><td>- As the years passed, the settlement on Mars flourished, becoming a symbol of human resilience and ingenuity. The astronauts who had once embarked on a</td></tr><tr style='color: green;'><td>+ Finally, the spacecraft touched down on the dusty surface of Mars, and the crew erupted into cheers and applause. Stepping out onto the alien world, they marveled at the rusty red landscape and the</td></tr></table>



### 图片生成

> 这部分比较简单，给定输入的prompt然后返回的是生成的图片的`url`。

```python
from openai import OpenAI
client = OpenAI()

response = client.images.generate(
  model="dall-e-3",
  prompt="一只猫",
  size="1024x1024",
  quality="standard",
  n=1,
)

image_url = response.data[0].url
print(image_url)
```

![img](https://oaidalleapiprodscus.blob.core.windows.net/private/org-C8wGVAVjrWpMePEWUK3moI0P/user-fyQHQ1S0qVZoI4AOCPUlESP5/img-QBiEdp6ryQEBfxiSiMXCmZF8.png?st=2023-11-27T06%3A03%3A23Z&se=2023-11-27T08%3A03%3A23Z&sp=r&sv=2021-08-06&sr=b&rscd=inline&rsct=image/png&skoid=6aaadede-4fb3-4698-a8f6-684d7786b067&sktid=a48cca56-e6da-484e-a814-9c849652bcb3&skt=2023-11-27T00%3A48%3A52Z&ske=2023-11-28T00%3A48%3A52Z&sks=b&skv=2021-08-06&sig=zYQvL9LjKq8xu4ofi4jtKK6IrG4q1h35E8W6spWFtdA%3D)

### 多模态图片能力

> 传入一个图片的`url, base64, 或者图片路径` 通过`gpt-4-vision-preview` 模型即可调用。

比如说对于这样一张图:

![img](https://upload.wikimedia.org/wikipedia/commons/thumb/d/dd/Gfp-wisconsin-madison-the-nature-boardwalk.jpg/2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg)

```python
from openai import OpenAI

client = OpenAI()

response = client.chat.completions.create(
  model="gpt-4-vision-preview",
  messages=[
    {
      "role": "user",
      "content": [
        {"type": "text", "text": "这个图片里面有什么内容？"},
        {
          "type": "image_url",
          "image_url": {
            "url": "https://upload.wikimedia.org/wikipedia/commons/thumb/d/dd/Gfp-wisconsin-madison-the-nature-boardwalk.jpg/2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg",
          },
        },
      ],
    }
  ],
  max_tokens=300,
)

print(response.choices[0])
```

输出结果：

```bash
Choice(finish_reason=None, index=0, message=ChatCompletionMessage(content='这张图片展示了一个美丽宁静的自然景观。画面中心是一条直通画面深处的木板路，两旁是郁郁葱葱、高高的绿色草地。在木板路的尽头可以看到一些树木，而在树木后方则是蓝色的天空，点缀着几朵洁白的云朵。整体而言，这张图片传达出一种平静和放松的氛围，是典型的户外自然风光摄影。', role='assistant', function_call=None, tool_calls=None), finish_details={'type': 'stop', 'stop': '<|fim_suffix|>'})
```



### TTS 以及 语音识别能力

> ---
>
> **TO BE CONTINUED**
>
> ---

### GPTS 定制化功能

> ---
>
> **TO BE CONTINUED**