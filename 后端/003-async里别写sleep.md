# `async def` 里只要有一个 `time.sleep`，你写的就不是并发服务

**症状**

`work` 改成 `async def` 之后，两个请求不是同时处理，而是**一个接一个**；严重的时候，整个服务像死了一样，点谁都没反应，日志干干净净。

**为什么**

三层，各管一段：

- **内核**先接受连接（TCP 握手跟你的代码无关）
- **uvicorn 的事件循环**去读 socket、把请求交给你的函数
- **你的函数**干活

`time.sleep()` 是**同步阻塞调用**。它在 `async def` 里跑，按住的是**事件循环本身**——不是"这个请求慢"，是"整个服务停摆"。

对比：

| 写法 | 谁在跑 | 结果 |
|---|---|---|
| `def work()`（同步） | Starlette 自动丢线程池 | 不卡，别的请求照进 |
| `async def work()` + `time.sleep` | 事件循环自己 | **全站排队** |
| `async def work()` + `await asyncio.sleep` | 事件循环（让出控制权） | 正常并发 |

**解法**

```python
import asyncio
await asyncio.sleep(10)          # async 里用这个

from fastapi.concurrency import run_in_threadpool
await run_in_threadpool(sync_func)   # 非要用同步库
```

**顺带：`BackgroundTasks` 不是队列**

- 它只在**当前请求生命周期**里：响应发出去之后，在**同一个进程**里把函数跑完。
- 没有持久化、没有重试、没有独立 worker；`--workers 4` 时任务只在接住它的那个进程里。
- 真队列：Celery / RQ / arq + Redis。

**记住**

> **`async def` 里出现一个 `time.sleep`，你写的就不是并发服务。**
> 阻塞调用不会丢请求，只会让请求迟到——**迟到超过超时，就等于丢了**。

---

*2026-11-27 · 自己动手把"卡住"逼出来的那一次*
