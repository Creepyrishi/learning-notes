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


**Dependencies**
with dependencies we can insert anything in the route function it is more like calling any function or classes before doing the route function

there are 2 main ways to add dependencies 
1. is to add a particular path 
```python
@router.get("/hello")
async def hello( db : Annoted[AsyncSession, Depends(get_db_Sessio)]):
	pass
```
2. another way is to just add to whole route level
```python
router = APIRouter(
prefix="/admin",
dependencies = [Depends(is_admin)]
)
```

in dependencies we cannot pass the arguments to the function normally. 
meaning we cannot do this:
```python
@router.get("/hello")
async def hello( db : Annoted[AsyncSession, Depends(get_db_Sessio("example"))]):
	pass

```

be we might need to pass sometime for that there are two ways
1. using callable class
2. using factory function

**Using Callable class**
we use `__call__` to make the class callable it self

in this method we pass the arguments to the `__init__` and then make that class callable via `__call__` which will be be called when the request arrives

```python

class CheckRole:
	async def __init__(self, allowed_role: str):
		self.allowed_role = allowed_role
	
	async def __call__(self, current_user = Depends(get_current_user)):
		if self.allowed_role != current_user.role:
			return HTTPException(details="dont have proper role")
			

@router.get("/delete")
async def delete(check_role = Depends(CheckRole("admin"))):
	pass
		
```

`CheckRole("admin")` with call init when defining the route and when the request comes then the whole class will be called like an function

**Using factory function method**
in this we will have inner function doing actual fastapi things.
```python
# 1. Outer factory function accepts your static arguments
def verify_token_prefix(prefix: str):
    
    # 2. Inner function handles the actual FastAPI request parameters
    def verifier(authorization: Annotated[str, Header()]):
        if not authorization.startswith(prefix):
            raise HTTPException(status_code=401, detail="Invalid token prefix")
        return authorization

    return verifier

# 3. Call the factory function inside Depends() to generate the dependency
@app.get("/secure-data")
def get_data(token: Annotated[str, Depends(verify_token_prefix("Bearer "))]):
    return {"status": "success", "token_used": token}

```

**HTTPBearer()**
this is a fastapi security scheme which returns a `HTTPAuthorizationCredentials` object  with `.credentials` which gives the raw token sting, fetched from the header, after removing prefix like Brear and it also return exception when its missing.

it reads `Authorization: Bearer <token>` from the header

```python
bearer_scheme = 
async def get_current_user(
    credentials: Annotated[HTTPAuthorizationCredentials, Depends(bearer_scheme)],
) -> User:
    token = credentials.credentials
```

**Webhooks**


say we are using process and celery and we doing something that involves memory say we are loading a index via FIASS and we want to keep that in memory but process and celery work on their own memory how can we use it from different instance

what was that something that we can use to not load somehing in memory rather we can point it to our disk idk what was it




session middleware fastapi
_IncludedRouter
Que

transcript_by_turn.setdefault(turn, []).append( sc.input_transcription.text ) what does this do if transcript_by_turn: dict[int, list[str]]

send_to_user already calls ws.send_json/send_bytes, and get_user_text can too (for error messages) — both running as concurrent asyncio tasks on the same socket already, with no lock. Adding background assessment tasks means potentially 3+ coroutines calling ws.send_json concurrently. Starlette/ASGI doesn't guarantee safety for that. Should I add a simple asyncio.Lock around all ws sends to make this safe, or leave it as-is (matching the existing pattern, where this risk already technically exists with 2 tasks)?

git precomit and setting up ruff 
1. Uvicorn, asgi, starlet
2. cookies


RAG eval
ROPE

why is not None