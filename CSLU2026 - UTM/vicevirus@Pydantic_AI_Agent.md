# Agent Workshop

@vicevirus

## What will be needed

---

- Openrouter API Key (down below) + `deepseek/deepseek-v4-flash`
- Python 3+
- Pip
- Working laptop
- The docs - https://pydantic.dev/docs/ai/overview

```php
Openrouter API Key = sk-
```

## DEPENDENCIES (must install these first)

```php
pip install pydantic "pydantic-ai-slim[openrouter]" requests python-dotenv
```

## Maybe it’s best to not hardcode API keys.. lol use these

---

```php
from dotenv import load_dotenv
load_dotenv()
```

```php
// .env
OPENROUTER_API_KEY=sk
```

## Simplest form of an agent

---

```php
# load env values
from dotenv import load_dotenv
load_dotenv()

import os
from pydantic_ai import Agent
from pydantic_ai.models.openrouter import OpenRouterModel
from pydantic_ai.providers.openrouter import OpenRouterProvider

model_name = "deepseek/deepseek-v4-flash"

model = OpenRouterModel(
    model_name,
    provider=OpenRouterProvider(
        api_key=os.getenv("OPENROUTER_API_KEY")
    ),
)
agent = Agent(model)

result = agent.run_sync("What is the latest news in AI?")

print(result.output)
```

## W’s in the chat?

---

```php
# load env values
from dotenv import load_dotenv
load_dotenv()

# agent
import os
from pydantic_ai import Agent
from pydantic_ai.models.openrouter import OpenRouterModel
from pydantic_ai.providers.openrouter import OpenRouterProvider

model_name = "deepseek/deepseek-v4-flash"

model = OpenRouterModel(
    model_name,
    provider=OpenRouterProvider(
        api_key=os.getenv("OPENROUTER_API_KEY")
    ),
)
agent = Agent(model)

while True:
    user_input = input("You: ")

    result = agent.run_sync(user_input)
    print("AI:", result.output)
```

## W’s in the chat 2 ? (with memory)

---

```php
# load env values
from dotenv import load_dotenv
load_dotenv()

# agent
import os
from pydantic_ai import Agent
from pydantic_ai.models.openrouter import OpenRouterModel
from pydantic_ai.providers.openrouter import OpenRouterProvider

model_name = "deepseek/deepseek-v4-flash"

model = OpenRouterModel(
    model_name,
    provider=OpenRouterProvider(
        api_key=os.getenv("OPENROUTER_API_KEY")
    ),
)
agent = Agent(model)

message_history = []

while True:
    user_input = input("You: ")

    result = agent.run_sync(
        user_input,
        message_history=message_history
    )

    print("AI:", result.output)

    message_history += result.new_messages()
```

## TOOLS ? (somewhat like MCP if you’ve ever used one)

---

```php
from pydantic_ai import Agent
from pydantic_ai.models.openrouter import OpenRouterModel
from pydantic_ai.providers.openrouter import OpenRouterProvider
import asyncio
from pydantic_ai import Tool

model_name = "deepseek/deepseek-v4-flash"

model = OpenRouterModel(
    model_name,
    provider=OpenRouterProvider(
        api_key="sk-of"
    ),
)

agent = Agent(
    model,
    instructions="You are a helpful assistant that provides information time.",
    )

@agent.tool_plain
def getCurrentTime() -> str:
    from datetime import datetime
    return datetime.now()

result = agent.run_sync('What is the time now?')

print(result.output)
```

destructive tool

```php
# load env values
from dotenv import load_dotenv
load_dotenv()

# agent
import os
from pydantic_ai import Agent
from pydantic_ai.models.openrouter import OpenRouterModel
from pydantic_ai.providers.openrouter import OpenRouterProvider

model_name = "deepseek/deepseek-v4-flash"

model = OpenRouterModel(
    model_name,
    provider=OpenRouterProvider(
        api_key=os.getenv("OPENROUTER_API_KEY")
    ),
)
agent = Agent(model)
from datetime import datetime

@agent.tool_plain
def hello() -> str:
  """if you call this tool you're dead"""
  return datetime.now() 

message_history = []

while True:
    user_input = input("You: ")

    result = agent.run_sync(
        user_input,
        message_history=message_history
    )

    print("AI:", result.output)

    message_history += result.new_messages()
```

## dangerous stuff

---

```php
# load env values
from dotenv import load_dotenv
load_dotenv()

# agent
import os
from pydantic_ai import Agent
from pydantic_ai.models.openrouter import OpenRouterModel
from pydantic_ai.providers.openrouter import OpenRouterProvider

model_name = "deepseek/deepseek-v4-flash"

model = OpenRouterModel(
    model_name,
    provider=OpenRouterProvider(
        api_key=os.getenv("OPENROUTER_API_KEY")
    ),
)
agent = Agent(model)

@agent.tool_plain
def file_read(path) -> str:
    with open(path, 'r') as f:
        return f.read()
    
@agent.tool_plain
def list_files(path: str = ".") -> list[str]:
    import os

    return os.listdir(path)

message_history = []

while True:
    user_input = input("You: ")

    result = agent.run_sync(
        user_input,
        message_history=message_history
    )

    print("AI:", result.output)

    message_history += result.new_messages()
```

## Structured output (we dont want freeform text sometimes)

---

```php
# load env values
from dotenv import load_dotenv
load_dotenv()

# agent
import os
from pydantic_ai import Agent
from pydantic_ai.models.openrouter import OpenRouterModel
from pydantic_ai.providers.openrouter import OpenRouterProvider
from pydantic import BaseModel, Field

model_name = "deepseek/deepseek-v4-flash"

model = OpenRouterModel(
    model_name,
    provider=OpenRouterProvider(
        api_key=os.getenv("OPENROUTER_API_KEY")
    ),
)

# pydantic schema
class GibTime(BaseModel):  
    time: str = Field(description="Only the ISO time string, no extra words")

agent = Agent(model, output_type=GibTime)

result = agent.run_sync(
        "What is the time now?",
    )

print(result.output)
print(result.output.time)
```

## Fun time! Just build whatever agents you like

---

Examples are 

- Weather planning agent (e.g: am going to klcc, do i need to bring umbrella today?)
- Lunch decider agent (e.g: I’m tired, budget RM15, around KL. What should I eat?)
- Personal study assistant agent (e.g: Explain SQL injection to me like I’m new.) (this could work really well with memory and chat loop)
- File Q&A agent (good for showing how agents can interact with files? like it can write stuff or automate some stuff?)
- Todo-list agent
- Currency converter agent (maybe use a public API for this)
- anything? up to you XD

sum other shi that we might need during the workshop

file read

```php
@agent.tool_plain
def file_read(path) -> str:
    with open(path, 'r') as f:
        return f.read()
```

list files

```php
@agent.tool_plain
def list_files(path: str = ".") -> list[str]:
    import os

    return os.listdir(path)
```

run shell/bash (this is dangerous)

```php
*@agent.tool_plain
def run_bash(command: str) -> str:
    import subprocess

    result = subprocess.run(
        command,
        shell=True,
        capture_output=True,
        text=True
    )

    return result.stdout + result.stderr*
```

random choice lol

```php
@agent.tool_plain
def random_choice(options: list[str]) -> str:
    import random
    return random.choice(options)
```

http get request

```php
@agent.tool_plain
def http_get(url: str) -> str:
    import requests

    try:
        response = requests.get(url, timeout=10)
        return response.text
    except Exception as e:
        return f"Error: {e}"
```

eval calculator tool (it’s dangerous but fak it)

```php
@agent.tool_plain
def calculator(expression: str) -> str:
    try:
        return str(eval(expression, {"__builtins__": {}}))
    except Exception as e:
        return f"Error: {e}"
```

weather tool

```php
@agent.tool_plain
def weather(city: str) -> str:
    import requests
    response = requests.get(f"https://wttr.in/{city}?format=3")
    return response.text

```

json all messages is best ig?

```php
agent = Agent(
    model,
    instructions="You are a helpful assistant that provides information time.",
    )

@agent.tool_plain
def weather(city: str) -> str:
    import requests
    response = requests.get(f"https://wttr.in/{city}?format=3")
    return response.text

result = agent.run_sync('Try checking the weather for Kuala Lumpur')

# print(result.output)
print(json.loads(result.all_messages_json()))
```

check for tool call

```php
messages = json.loads(result.all_messages_json())

print(messages)

for msg in messages:
    for part in msg.get("parts", []):
        if part.get("part_kind") == "tool-call":
            print("Tool called:", part["tool_name"])
            print("Args:", part["args"])
            print("ID:", part["tool_call_id"])

        elif part.get("part_kind") == "tool-return":
            print("Tool returned:", part["tool_name"])
            print("Result:", part["content"])
            print("ID:", part["tool_call_id"])
```

streaming

```php
from pydantic_ai import Agent
from pydantic_ai.models.openrouter import OpenRouterModel
from pydantic_ai.providers.openrouter import OpenRouterProvider
import asyncio

model_name = "deepseek/deepseek-v4-flash"

model = OpenRouterModel(
    model_name,
    provider=OpenRouterProvider(
        api_key="sk"
    ),
)

agent = Agent(model)

async def main():
  
  async with agent.run_stream('Where does "hello world" come from?') as result:
      async for message in result.stream_text(delta=True):  
          print(message, end='', flush=True)

asyncio.run(main())
```