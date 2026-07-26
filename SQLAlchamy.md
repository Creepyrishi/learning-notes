```python
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select, insert, update, delete, text

async def addDocument(db: AsyncSession):
    # READ
    result = await db.execute(select(User).where(User.id == 1))
    user = result.scalar_one_or_none()

    # CREATE
    db.add(User(name="Rishi"))
    await db.commit()

    # UPDATE
    user.name = "Updated"
    await db.commit()

    # DELETE
    await db.delete(user)
    await db.commit()

    # RAW SQL
    result = await db.execute(text("SELECT * FROM users WHERE id = :id"), {"id": 1})
    rows = result.fetchall()
```

**Problem**: SQLAlchemy Enum(StatusEnum, native\_enum=True) uses enum member names (CHUNKED, SUMMARIZED, EMBEDDED) as database values instead of string values (chunked, summarized, embedded).
**Error**: invalid input value for enum status\_enum: "CHUNKED" — DB expects lowercase, got uppercase.
**Fix**: Add values\_callable to map enum → .value:

```python
status: Mapped[StatusEnum] = mapped_column(
    Enum(
        StatusEnum,
        native_enum=True,
        name="status_enum",
        values_callable=lambda x: [e.value for e in x],  # ← ADD THIS
    ),
    default=StatusEnum.CHUNKED,
)
```

**Diffrence between AsyncSession and AsyncSessionLocal**

AsyncSession is object of sqlalchamy which is used to perform non blocking i/o in db. And AsycnSessionLocal is convention that was made popular in fastapi docs which is of type AsycnSession and is returned my async\_sessionmaker() fucntion. It is used with context to execute query on the db.

```python
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker, AsyncSession

# 1. Create an async engine (e.g., PostgreSQL with asyncpg)
DATABASE_URL = ""
engine = create_async_engine(DATABASE_URL, echo=True)

# 2. Create the async session maker
AsyncSessionLocal = async_sessionmaker(
    bind=engine,
    class_=AsyncSession,
    expire_on_commit=False
)

# 3. Usage inside a function
async def get_user_data(user_id: int):
    async with AsyncSessionLocal() as session:
        # All DB interactions are awaited
        result = await session.execute(
            select(User).where(User.id == user_id)
        )
        user = result.scalar_one_or_none()
        return user

```

**making AsyncSession generator**
Async session generator generates a AsyncSesssion and give it to a what evey called that generator. this generator is context managed which mean the db session will be closed and cleaned up automatically

this is a generator function

```python
async def get_db_session() -> AsyncGenerator[AsyncSession, None]:
    async with AsyncSessionLocal() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
```

we need to use async generator when we need to use the session and keep it open even after getting out of the context

```python
    async def getClient(self) -> AsyncGenerator:
        try:
            client = Client(api_key=str(settings.GEMINI_API_KEY))
            async with client.aio.live.connect(
                model=settings.GEMINI_LIVE_MODEL, config=self.Config
            ) as session:
                # we cannot use return here because when we return the session 
                # get automatically closed beacuse we are going out of the 
                # async with context so we need to use asyncgenerator
                
                
                # return session
                yield session

        except Exception:
            raise

```

As soon as `return session` happens, Python exits the `async with` block. That calls cleanup on the live connection. So the caller receives a session whose context has already been closed or is about to be closed.

Use an async generator/context manager if the caller needs to use the session while keeping the connection open

this is mostly true for context managers if want to use any thing from `with` in some place we might need to make generator in both `async` and `sync`

\*\*What is func and func.count in the sqlalchamy

```python
from sqlalchemy import func
```

func is a sql function generator. sql databases have some predefined function that we can call via sql for eg **COUNT(\*)**  This will sum the rows.

now to use it via sqlalchemy we need to use func like

```python
result = session.execute(select(func.count()).select_from(User).where(User.age>30))
result = result.scalar()
```

```sql
SELECT COUNT(*)
FROM user
WHERE age > 30;
```

Some database lets us create custom function too.

database pool
