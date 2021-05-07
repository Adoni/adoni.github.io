---
layout:     post
title:      "Python Coding Practice"
subtitle:   "I didn't say it's the best practice, so you know ..."
date:       2019-01-07 10:00:00
author:     "Xiaofei"
header-img: "img/in-post/2019-01-08-python-practice.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Python
---


## Concurrent and multi-process
It’s common to use services to do something in a company. In this situation, we want to use multiple processing to handle traffic delay.

```python
from concurrent.futures import ProcessPoolExecutor
with ProcessPoolExecutor(max_workers=16) as exe:
    result = exe.map(func, data_list)
```

Only three lines!

It’s similar to the previous way I use which rely on thread pool. But this way is much more easier.

By the way, it’s also good to know how efficient it is. My way is to split a big list into multiple chunks, and use ProcessPoolExecutor to solve each chunk. The tool for split large list (or iterable generator) is called [more_itermtools](https://github.com/erikrose/more-itertools). And [tqdm](!) can help you know how fast it is and when it will finish.

Now, a typical way I use is is like:

```python
from concurrent.futures import ProcessPoolExecutor
from tqdm import tqdm
from more_itertools import chunked
batch_size = 100
for batch in tqdm(chunked(THE_LONG_DATA_GENERATOR, batch_size), total=TOTAL_IF_YOU_KNOW):
    with ProcessPoolExecutor(max_workers=16) as exe:
        result = exe.map(func, batch)
```


## Another concurrent style

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

Here we only show the basic use of multi-thread, without any exception processing (e.g. timeout and other errors)

## NamedTuple
`typing.NamedTuple` is a feature of python, which could help us create a class with the `key` you provide. It’s a good way to replace dict, which I used before.

Previously, if I want to put database information together, including `username`, `password`, `host` and `port`, then I will use a dict like:

```python
connection = {
    "username": "foo",
    "password": "bar",
    "host": "localhost",
    "port": "8080"
}
```

This way works but it has one shortcoming. You cannot use IDE to autocomplete or check your syntax, which may leads you to misspell the `username` with `usename`.

However, we can rewrite it as the following:

```python
import typing
ConnectInfo = typing.NamedTuple('ConnectInfo', [('username', str), ('password', str), ('host', str), ('port', str)])
connect = ConnectInfo(
    username="foo",
    password="bar",
    host=localhost",
    port=8080")
```

Notice that like what the name indicates, `NamedTuple` is a tuple, which means the data cannot be changed after initializing.

## Decorator
I believe this [webpage](http://book.pythontips.com/en/latest/decorators.html) already explained this concept clearly. 

## One-liner
For each number in [1, 10], calculate their squire:

`result = [i**0.5 for i in range(1,10)]`

Calculate it again but return as a `set`:

`result = {i**0.5 for i in range(1,10)}`

Return a key-value dict of this manipulate:

`result = {i: i**0.5 for i in range(1,10)}`

## Jupyter
For me, the best thing of Jupyter is the inline plots or said interactive development, which means I can run code on the server and see the plots immediately. Considering that, I used to believe that Jupyter is the best practice when you exploring data science until I read this slides: [I Don’t Like Notebooks - Joel Grus - #JupyterCon 2018 - Google Slides](https://docs.google.com/presentation/d/1n2RlMdmv1p25Xy5thJUhkKGvjtV-dkAIsUXP-AL4ffI/edit#slide=id.g3d168d2fd3_0_6)

But I still believe that Jupyter is a great tool. If you have a dataset and wanna know the data distribute or some statistics measures, use pandas + jupyter; if you want plot on a remote machine, use matplotlib+jupyter; if you want a short tutorial, use markdonw and latex in jupyter. Yes, each of those requirements can be satisfied by other means. However, it means you need to google it and do a lot hacks, which is not friendly to novices. For example, if I want to plot on server and show results on my local screen, I need to google it, and luckily it will tell me to use X11 forwarding, which, however, will mess you up in turn. I will write another post to show how to to it.

For me, to make a balance of both choices, I’m telling myself that don’t write more than 50 lines or 100 lines codes in notebooks.

For more Jupyter details or common tips, please refere [here](!)

## More tips
1. IDE, I mean PyCharm, is much better than Vim (or even VSCode, but someone may disagree)，especially for novices
2. But you should know how to use vim
3. If you’re a huge fan of vim, please google `bounding vim in PyCharm`
4. Please google `function annotation` and `variable annotation` in Python, and you will come back and thanks me.
5. Refer [A “Best of the Best Practices” (BOBP) guide to developing in Python. · GitHub](https://gist.github.com/sloria/7001839) for the other python best practice
6. Try to remove all warnings in IDE, except you know the result.
7. Please try to learn profiling tools for the language you use, which will finally save your time.
8. Refer [GitHub - ianozsvald/ipython_memory_usage: IPython tool to report memory usage deltas for every command you type](https://github.com/ianozsvald/ipython_memory_usage) for memory use
9. Read this please: http://book.pythontips.com/en/latest
10. Use `line_profiler` to find bottlenecks of your codes.
11. Learn how to use decorator will help you when reading other’s code. Also, you can write more flexible codes.
12. Use `tqdm` for big loops
13. Use `abstract` class when you have to deal with different situation with similar codes
14. A good book: [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html) and it's [Chinses version](https://zh-google-styleguide.readthedocs.io/en/latest/)
15. Some time (2020 to 2021), the latest ipython will crash when you press Tab to find tips or auto-complement. You can run `pip install jedi==0.17.2` degrade jedi from 0.18.0 to 0.17.2.

## Concepts Not Included
1. PyCharm interesting features and autopep8 in PyCharm
2. Plot in server

I will try my best to write individual blog for each of them
