celery uses as system a task queue which is used to distribute the task to multiple celery workers and it need a message broker to transfer the task from celery client of out code to the celery worker.  there are multiple services that can be used as broker popular are RabbitMQ and Reddis. 

by default the number of worker is set to number of core of the pc

celery working animation
https://www.youtube.com/watch?v=TzVkED3y3Ig

while working on a project we need to make a Celery app and then we can import it another files and create task with it.

```
project/
	celery/
		__init__.py
		celery.py
		tasks.py
```

**Creating celery app**
```python
from celery import Celery

app = Celery('proj',
             broker='amqp://',
             backend='rpc://',
             include=['proj.tasks'])

# Optional configuration, see the application user guide.
app.conf.update(
    result_expires=3600,
)
```

**creating tasks**
```python
from .celery import app
# there also is @shared_task but it is used when there is no or many instace of celery app
@app.task
def add(x, y):
    return x + y

@app.task
def mul(x, y):
    return x * y

@app.task
def xsum(numbers):
    return sum(numbers)
```

then we need to start the celery working by using this command
```
celery -A proj worker -l INFO
```
or
```python 
#celery.py
if __name__ == '__main__':
    app.start()
```

more important command are here
https://docs.celeryq.dev/en/stable/reference/cli.html

after creating a task we can call those task to make it go to celery worker and execute it. there are 3 methods to call.
`.delay(arg1, agr2)`, `apply_async()` and `__call__`

delay
```python
# x and y are the args of the mul task 
mul.delay(x, y)
```

apply_async
while delay will send the task immidieatly in apply_async will gives us control over a lot of the thigs. we can may be execure after say 1 min or with exact date and time too.
https://docs.celeryq.dev/en/main/reference/celery.app.task.html#celery.app.task.Task.apply_async
here i can see what are everything i can do with apply_async

```python
mul.apply_async(args=[x, y], countdown=20)
```

\_\_call\_\_ 
this is calling the tasks like a normal function which mean it will executed in the current process it self and wont be sent to broker.

```python
add(4, 4)
```

out of these all three only `__call__` won't return AsyncResult


**AsycnResult**

**Check Task Status**
- `.state`: Returns a string of the exact state (e.g., `PENDING`, `STARTED`, `SUCCESS`, `FAILURE`).
- `.ready()`: Returns `True` if the task has finished executing (successfully or failed).
- `.successful()`: Returns `True` if the task finished with a `SUCCESS` status.
- `.failed()`: Returns `True` if the task raised an unhandled exception.

**Fetch the Output**
after the task finishes we can get its result
- `.get(timeout=None)`: Blocks your current thread until the task finishes and returns the actual result. You can pass a `timeout` in seconds to prevent waiting forever.
- `.result`: Holds the return value or the exception instance (once populated).
- `.traceback`: Returns the full Python stack trace if the task failed.

**Manage and Control the Task**
we can stop and remove the task using their id
- `.revoke(terminate=True)`: Tells the Celery worker to cancel and kill the task.
- `.forget()`: Removes the task result from your backend storage (Redis/Database) to free up memory.
- `.id`: Provides the unique UUID string for the task, allowing you to recreate the `AsyncResult` object later or in a different HTTP request. [21]

```python
from celery.result import AsyncResult
from my_app import app

# 1. Trigger the task
result = my_task.delay(10, 20)
task_id = result.id  # Save this UUID to check status later

# 2. Check status without blocking
if not result.ready():
    print(f"Task is still in state: {result.state}")

# 3. Safely get the result with a timeout
try:
    data = result.get(timeout=5)
    print(f"Task succeeded! Output: {data}")
except Exception as e:
    print(f"Task failed or timed out: {e}")
```


**celery beat**
https://docs.celeryq.dev/en/4.0/userguide/periodic-tasks.html
**config**
https://docs.celeryq.dev/en/stable/userguide/configuration.html
celery canvas
celery result and backend
