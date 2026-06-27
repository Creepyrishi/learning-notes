**lifespan**
```python
from contextlib import asynccontextmanager
from fastapi import FastAPI
import asyncio
import discord

bot = discord.Client(intents=discord.Intents.default())

@asynccontextmanager
async def lifespan(app: FastAPI):
    asyncio.create_task(bot.start("YOUR_TOKEN"))
    yield

app = FastAPI(lifespan=lifespan)
```

- Code **before** `yield` → runs when FastAPI starts
- Code **after** `yield` → runs when FastAPI shuts down


**FastAPI Dependency Injection** 

**Core idea**
- Declare dependencies using `Depends()`
- FastAPI **automatically calls them and injects results**
Basic usage

```python
async def route(x: Type = Depends(dep_func)):
```

Key rules
- Do **not** call dependency yourself
```python
    Depends(func)      # ✅ correct
    Depends(func())    # ❌ wrong
```

- Dependencies can have parameters → FastAPI fills them from:
    - query, path, body, headers        
    - other dependencies

Passing custom values (patterns)

```python
def dep_fixed():
    return func("value")
Depends(dep_fixed)
##
Depends(lambda: func("value"))
##
class Dep:
    def __init__(self, val):
        self.val = val
    def __call__(self):
        return func(self.val)

Depends(Dep("value"))
##
Depends(partial(func, val="value"))
##
def make_dep(val):
    def dep():
        return func(val)
    return dep

Depends(make_dep("value"))
```

Mental model
- You **declare what you need**
- FastAPI **builds and injects it automatically**
- Use wrappers/classes when you need **custom inputs**


**Webhooks**






1. Uvicorn, asgi, starlet
2. cookies
3. pollings
compelete converstaional ai by tmmrow

https://medium.com/@rwilliams_bv/intro-to-databases-for-people-who-dont-know-a-whole-lot-about-them-a64ae9af712
 cerery, boto for s3, docker, redis
 https://www.youtube.com/watch?v=RIVcqT2OGPA
 git
 TF-IDF
 creating corn jobs and auto run script in linux
 RAG eval
 __future__ and Context managing, asycnio.semaphor
 https://www.youtube.com/watch?v=oAkLSJNr5zY
 KV cache optimization, sliding window attention, memory compression techniques
 [context refinement](https://chatgpt.com/c/6a2b90db-54d4-83ee-978b-82d628d0d218)


https://www.youtube.com/playlist?list=PL-osiE80TeTsak-c-QsVeg0YYG_0TeyXI
this have a lot of thing that i want to learn this is fastapi tutorial  it have almeic, boto aws, postgres, etc etc check it out