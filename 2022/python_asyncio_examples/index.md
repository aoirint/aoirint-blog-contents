---
title: Python, asyncioを使った実装例
date: '2022-07-07T11:20:00+09:00'
draft: false
channel: 技術ノート
category: Python
tags:
  - Python
  - 非同期処理
---
# Python, asyncioを使った実装例

- Python 3.9.13 (pyenv)
- Ubuntu 20.04 (WSL2)

## 基本実装

### 同期関数から非同期関数を同期的に呼び出す
- asyncio.run

<details>

```python
import asyncio
import time

def main():
  async def func():
    await asyncio.sleep(3)
    print('func exited') # 1

  asyncio.run(func())

  time.sleep(1)
  print('main exited') # 2

main()

print('exited') # 3
```

</details>

### 同期関数から非同期関数を非同期的に呼び出す
- threading.Thread + asyncio.run

<details>

```python
import asyncio
from concurrent.futures import ThreadPoolExecutor
import threading
import time

def main():
  async def func():
    await asyncio.sleep(3)
    print('func exited') # 3

  thread = threading.Thread(target=lambda: asyncio.run(func()))
  thread.start()

  time.sleep(1)
  print('main exited') # 1

main()

print('exited') # 2
```

</details>

### 非同期関数から同期関数を同期的に呼び出す
- ふつうに呼び出す

<details>

```python
import asyncio
import time

async def main():
  def func():
    time.sleep(3)
    print('func exited') # 1

  func()

  await asyncio.sleep(1)
  print('main exited') # 2

asyncio.run(main())

print('exited') # 3
```

</details>

### 非同期関数から同期関数を非同期的に呼び出す
- asyncio.new_event_loop + ThreadPoolExecutor + EventLoop.run_in_executor

<details>

```python
import asyncio
from concurrent.futures import ThreadPoolExecutor
import time

async def main():
  def func():
    time.sleep(3)
    print('func exited') # 3

  loop = asyncio.new_event_loop()
  executor = ThreadPoolExecutor()
  loop.run_in_executor(executor, func)

  await asyncio.sleep(1)
  print('main exited') # 1

asyncio.run(main())

print('exited') # 2
```

</details>

### 非同期関数から非同期関数を同期的に呼び出す
- awaitキーワード

<details>

```python
import asyncio

async def main():
  async def func():
    await asyncio.sleep(3)
    print('func exited') # 1

  await func()

  await asyncio.sleep(1)
  print('main exited') # 2

asyncio.run(main())

print('exited') # 3
```

</details>

### 非同期関数から非同期関数を非同期的に呼び出す
- threading.Thread + asyncio.run

<details>

```python
import asyncio
import threading

async def main():
  async def func():
    await asyncio.sleep(3)
    print('func exited') # 3

  thread = threading.Thread(target=lambda: asyncio.run(func()))
  thread.start()

  await asyncio.sleep(1)
  print('main exited') # 1

asyncio.run(main())

print('exited') # 2
```

</details>


## よく見るエラー

### RuntimeError: Event loop is closed
TBW

### RuntimeError: This event loop is already running
TBW


## アプリケーション

### FastAPI + schedule
- asyncio.new_event_loop + ThreadPoolExecutor + EventLoop.run_in_executor
- threading.Event
- Dependencies
  - fastapi==0.78.0
  - schedule==1.1.0
  - uvicorn==0.18.2

<details>

```python
import time
import threading
import asyncio
from concurrent.futures import ThreadPoolExecutor
import schedule
from fastapi import FastAPI

app = FastAPI()
schedule_event = threading.Event()

@app.on_event('startup')
async def startup_schedule():
  loop = asyncio.new_event_loop()
  executor = ThreadPoolExecutor()

  def loop_schedule(event):
    while True:
      if event.is_set():
        break
      schedule.run_pending()
      time.sleep(1)

    print('run all existing scheduled jobs')
    schedule.run_all()

    print('exit schedule')

  loop.run_in_executor(executor, loop_schedule, schedule_event)

  schedule.every(1).second.do(lambda: print('tick'))

@app.on_event('shutdown')
async def shutdown_schedule():
  schedule_event.set()
```

</details>


### Capture FFmpeg subprocess report
- TBW
