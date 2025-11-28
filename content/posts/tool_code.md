---
title: "工具代码记录"
description: "记录工作中常用的工具代码"
keywords: "tool-code"

date: 2023-12-20T00:04:49+08:00
lastmod: 2023-12-20T00:04:49+08:00

categories:
  - Code
tags:
  - Tool-Code
  - 备忘

---

记录一些自己写的、从别处学来的工具代码，方便随时使用。

<!--more-->

## 1. 在新的进程中运行函数(Python)

**Background**: 有时执行一段代码，可能存在对全局环境有不可逆副作用，这时新开一个进程来跑这段代码，就可以随着进程释放将副作用消除。

**应用举例**： Whisper 运行会有内存/显存泄露，其泄露应该在非 Python 层，个人无法解决；而这个泄露又对持续运行的线上服务而言不可接受。这时新起进程来运行就可以解决这个问题。

### 代码片段

```Python
def run_in_independent_process(is_cuda: bool, fn, *args, **kwargs):
    """make function run in another process(used to avoid memory leak or other bad side-effects)"""
    from concurrent.futures import ProcessPoolExecutor
    import multiprocessing

    if is_cuda:
        cxt_name = "forkserver"
    else:
        cxt_name = "fork"
    # logger.info("Create independent process for function [%s] in [%s] mode", fn.__name__, cxt_name)
    cxt = multiprocessing.get_context(cxt_name)
    with ProcessPoolExecutor(max_workers=1, mp_context=cxt) as p:
        future = p.submit(fn, *args, **kwargs)
        return future.result()
```