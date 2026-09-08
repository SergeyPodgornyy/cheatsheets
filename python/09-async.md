# Python: AsyncIO

Asynchronous execution does not make our code faster, but it allows us to proceed with other functions while we are waiting for a response from the previous. Sometimes asynchronous functions are referred to as **coroutines**. And coroutines are functions that can be suspended and resumed in the future.

```python
import asyncio
from asyncio import Task

async def fetch_data(input: str) -> dict:
    await asyncio.sleep(3)
    return {'input': input}

async def main() -> None:
    task_dev: Task[dict] = asyncio.create_task(fetch_data('dev'))
    task_prod: Task[dict] = asyncio.create_task(fetch_data('prod'))

    # our code becomes asynchronous here when running tasks, as await tells the
    # program we need to wait for this line to complete before moving on to the
    # next one, it is blocking line on the code
    data_dev: dict = await task_dev
    data_prod: dict = await task_prod

asyncio.run(main=main())
```

**Task** in AsyncIO is scheduled and independently managed coroutine. And we can query them, we can cancel them, we can retrieve the results later, and we can handle them in many different ways.

```python
task: Task[dict] = asyncio.create_task(fetch_data('dump'))
await asyncio.sleep(3)
task.cancel(msg='Took too long...')
await asyncio.sleep(1) # it takes some time to register cancel

try:
    data: dict = await task
except asyncio.CancelledError as e:
    print(e)
print(task.cancelled()) # True/False

# we can also grab result without await, if it is ready
try:
    data: dict = task.result()
except asyncio.InvalidStateError as e:
    print(e) # "Result is not set.", if task is not finished yet

# so we can proceed with data, only if task is done
if task.done():
    data: dict = task.result()

# we can also use a timeout feature
try:
    data: dict = await asyncio.wait_for(task, timeout=5)
except asyncio.TimeoutError:
    ...
```

We can also create a bunch of tasks and run them together by creating the `Future`. And a Future represents an eventual result of an asynchronous operation, it creates a promise in the future that something will come back. We can pass in as many coroutines as we want to `gather` function.

```python
import asyncio
from asyncio import Future

tasks: Future[dict] = asyncio.gather(
    fetch_data('dev'),
    fetch_data('demo'),
    fetch_data('prod'),
    # instead of failing, it's just going to return an exception instead of result
    return_exceptions=True
)

results: list[dict | Exception] = await tasks
```

We can also turn any synchronous function to an asynchronous using the `to_thread` function:

```python
import asyncio
import requests
from requests import Response

async def check_status(url: str) -> dict[str, str | int]:
    response: Response = await asyncio.to_thread(requests.get, url, None)
    return {'status_code': response.status_code,
            'website': url}
```

<!-- nav -->
---

← [Python: Generators, Decorators, Memoization](08-gen-decorators.md) · [Index](README.md) · [Python: OOP](10-oop.md) →
<!-- nav -->
