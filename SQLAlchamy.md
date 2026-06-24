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


**Problem**: SQLAlchemy Enum(StatusEnum, native_enum=True) uses enum member names (CHUNKED, SUMMARIZED, EMBEDDED) as database values instead of string values (chunked, summarized, embedded).
**Error**: invalid input value for enum status_enum: "CHUNKED" — DB expects lowercase, got uppercase.
**Fix**: Add values_callable to map enum → .value:
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

AsyncSession is object of sqlalchamy which is used to perform non blocking i/o in db. And AsycnSessionLocal is convention that was made popular in fastapi docs which is of type AsycnSession and is returned my async_sessionmaker() fucntion. It is used with context to execute query on the db.

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

 
 **What is func and func.count in the sqlalchamy
 
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