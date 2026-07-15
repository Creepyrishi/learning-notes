#### Terminologies

1. Event loop

Event loop is like a loop and it have a thing called control and that control can be give to any other function to let the execute.

Say we have 2 functions to run first one needs 5 sec to fetch data from internet. We have 2 options either we can wait for the data in first function to be fetched, in that case we are running the code sequentially.

Second option is to pause the function 1 let the function 1 run separately when it is fetching data from the internet as when the data has been fetched there is nothing that python need to do, it's the os and network stuffs not the python. So in that time we can run function 2 and when the fetching is completed we can go back to the function 1 to complete it. This way we will make our program 5 seconds faster.

Now a question arises, why not run the both function in parallel. We don't because python doesn't historically support true parallelism because of global interpreter lock (GIL). Without going in technical detail it means single process at one time.

Good news: GIL removal is coming, python 3.13 has added an option to build python without GIL but its still experimental, by default normal python still has GIL.

Another good news: there is a technique with which we can do true parallelism, shown below in ProcessPoolExecutor part.

When in event loop, when a function is being executed it means event loop gave that function the control and when waiting for some I/O bound task or it finishes executing, the control is given back to the event loop and the event loop gives it to the next one.

2. Awaitables

Awaitables are those things which have `__await__()` method implemented with it and we can use `await` keyword to await it. We cannot await synchronous functions because they don't know how to yield control and resume later.

Awaitables can be awaited in async function.

Basic example:

```python
import asyncio

async def A():
    do_a()
    do_b()
    await do_c()
    do_d()

async def B():
    do_x()
    do_y()
```

Say we have run this code and the line `do_c` is some I/O bound operation, maybe fetching an image from internet.

When we start to run, `do_a` will run, after it finishes `do_b` will run, and then `do_c` will be awaited. `do_c` will be paused when it starts to fetch the image from the internet and the control will be given back to the event loop until `do_c` gets the image from the internet. Now the event loop gives the control to function `B` to execute until the image gets fetched.

That's it. It saves time. Instead of waiting for image to download, it lets it be there and moves on to execute another function.

There are 3 types of awaitables:

1. coroutine
2. task
3. future

When we call an async function it returns us a coroutine, it does not get executed. To execute that coroutine we need to await it.

```python
async def A():
    pass

x = A()  # this will give us a coroutine object, and to execute the function we need to await that coroutine.

result = await x
```

Coroutine functions are created using `async` keyword.

**Tasks**

Tasks are wrappers around the coroutine which will run when it will have a chance.

We can create tasks by using:

```python
async def FuncA():
    pass

# Schedules the task, but does not immediately run it, it waits for event loop to give it control
task = asyncio.create_task(FuncA())
```

Concurrency is achieved by doing background tasks.

**Future**

It is a low level concept that I will ignore.

Running a synchronous function as async can be done in two ways:

1. By using `to_thread`

This will execute in a different thread, not the same thread. that is the whole point, the blocking function runs on a separate thread so it doesn't block our event loop thread.

```python
import asyncio

def A(x, y):
    pass

async def B():
    task1 = asyncio.create_task(asyncio.to_thread(A, x, y))
    result = await task1
```

2. Using `ProcessPoolExecutor`

This is true multi core execution. In this method, a new instance of python will run on a different core with its own interpreter and GIL, and it will execute the code that we provide it, independent of our main code.

```python
import asyncio
from concurrent.futures import ProcessPoolExecutor

def A(x, y):
    pass

async def main():
    loop = asyncio.get_running_loop()
    with ProcessPoolExecutor() as exect:
        task1 = loop.run_in_executor(exect, A, x, y)
        result = await task1
```

As this is true parallelism we can do it for CPU bound tasks too, and this will also increase the memory cost as a new python instance will be run, this trade off should be considered.

**Running multiple tasks or coroutines**

There are times when we want to run many coroutines and want to get all results at once, or even terminate everything when there is an error in any one.

There are 2 ways to achieve it:

1. `gather`

```python
result = await asyncio.gather(coro1, coro2, ...)
result = await asyncio.gather(task1, task2, ...)
```

When we add coroutines in the gather, they get converted to tasks internally.

This will await all the tasks and give the result in a list in the same order in which the coroutines are given.

However there might be some exception in any one and we might want to discard all of them. by default `return_exceptions=False` and in that case if any task raise an error, gather will immediately raise that exception to us, other tasks keep running in background but we don't get a clean result list. if we want the error to just sit in the result list instead of getting raised, we need to set `return_exceptions=True`.

```python
result = await asyncio.gather(task1, task2, ..., return_exceptions=True)
```

The above can also be replicated via `TaskGroup`:

```python
async def main():
    async with asyncio.TaskGroup() as tg:
        t1 = tg.create_task(coro1)
        t2 = tg.create_task(coro2)
```

Both tasks will be created and awaited automatically.


**asyncio.wait**
this is used to run multiple task at the same time and block the execution of the function when a specific condition is met.

```python
tasks = [task1, task2, task2]
done, pending = await asyncio.wait(tasks, timeout=None, return_when=ALL_COMPLETED)
```

**Thread vs Process vs celery**

thread share memory with in the same process but different process or celery workers have separate memory and spawning the process cost more than thread. in the case of FAISS load and build i was doing it using thread was better. and as FAISS is written in c++ it automatically releases the GIL when doing heavy computation

Process are only good when doing pure-python cpu bound work

**asyncio.semaphor**
Semaphore is used to limit the total number of tasks that we want to run at a single time. 

```python
import asyncio

# 1. Create a semaphore that allows a max of 3 concurrent tasks
sem = asyncio.Semaphore(3)

async def access_resource(task_id):
    # 2. Wait in line until a slot is free
    async with sem:
        print(f"Task {task_id} is inside the room.")
        await asyncio.sleep(2)  # Simulating some I/O work
        print(f"Task {task_id} is leaving.")

async def main():
    # Trigger 10 tasks all at once
    tasks = [access_resource(i) for i in range(10)]
    await asyncio.gather(*tasks)

asyncio.run(main())
```

```
Task 0 is inside the room.
Task 1 is inside the room.
Task 2 is inside the room.  <-- Stops here! The first 3 take all the keys.
(2 second pause...)
Task 0 is leaving.
Task 3 is inside the room.  <-- Task 3 grabs the key Task 0 just dropped.
Task 1 is leaving.
Task 4 is inside the room.
```

semaphore uses `.accuire()` and `.release()` under the hood we might use it when we use the semaphore without a context manager. but using with context manager is the best
