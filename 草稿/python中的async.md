# Python多线程进阶1



在之前的博客中，我们介绍了两种简单的多进程实现思路。



第一种是采用`ThreadPoolExecutor`，将列表数据map到多个executor的同一个function上，代码如下

```python
from concurrent.futures import ThreadPoolExecutor
with ThreadPoolExecutor(max_workers=16) as exe:
  result = exe.map(func, data)
```



第二种也是采用`ThreadPoolExecutor`，但是利用了submit功能，配合`as_complete`函数不断将完成的结果拿出来，代码如下

```python
with ThreadPoolExecutor(max_workers=max_workers) as exe:
    bar = tqdm.tqdm(total=len(batch), leave=False)
    futures = {}
    for datum in THE_LONG_DATA_GENERATOR:
        future = exe.submit(func, datum)
	      futures[future] = None
    for future in as_completed(futures, timeout=60 * 20):
        result = future.result()
        # Do your work here with result
        bar.update(1)
```



然而这两种方法本质上是假设func中的功能是统一的，那如果有很多个不同的func呢，或者说，每次执行要调用不同的后端呢



## 应用1-简单负载均衡



> 负载均衡有更加专业的实现方式，例如nginx、或者rpc大多数有相应的支持。另外负载均衡要实现服务发现等一系列任务，本文都没有覆盖，这也是标题中“伪”字的含义。本文只想使用纯python的方式实现某种简单的负载均衡。
>
> 准确来说，本文的目的是介绍一种多线程工作方式





前几天遇到了一个问题，同事给我提供了多个worker，我可以通过某种方式（如rpc或者restful或者直接python包访问等等）的形式实现某个机器学习算法的执行结果，由于是rpc服务且我不熟悉rpc且时间紧迫，为了快速实现负载均衡，将他们统一在同一个包里，便于其他人调用，因此想到了用python直接实现



本质上，负载均衡的思路和多进程多线程是类似的，因此我最先想到的是