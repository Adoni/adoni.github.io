---
layout:     post
title:      "Python Profiling"
subtitle:   "性能瓶颈查验"
date:       2022-08-08 10:43:00
author:     "Xiaofei"
header-img: "img/in-post/2022-08-08/python-profiling.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Python
    - Tools
    - 性能优化
---


## line-profiler

line-profiler是我们比较早使用的工具，通过装饰器实现，需要开发人员在想要profile的函数上添加`@profile`装饰器。其输出较为友好，虽然是terminal上的。

然而line-profiler终究还是有一系列问题的，比如：

* 侵入式修改：必须在代码上添加相应的`@profile`标签
* 递进式的profile过程：需要逐次递进进行profile，而非一键获取所有递归结果，例如，我们通过`@profile`标签获取了一个函数的结果，然而发现其瓶颈在于调用另一个函数，这时候我们需要给另一个函数也添加`@profile`
  标签，并且重新运行 ，还是比较烦的
* 无法对已经运行的代码（比如一个服务）进行profile：这是由其实现机制决定的
* 干扰运行速度：代码运行时间会变成之前的1.6倍

因此，我们非常希望替换掉line-profiler

## py-spy

### 简介

官网见：https://github.com/benfred/py-spy

安装方法很简单

```
pip install py-spy
```

py-spy是一种不侵入式的方法，类似cProfile，和line-profiler是两个路线

有另一个类似的叫pyflame，之前是uber维护的，现在停止维护了，不知道是不是被py-spy干掉了。

顺道说一下，如果一个开源项目不维护了（比如被archived了，或者），尽量不要用了，不维护了可能是人手不足，可能是有其他更好的替代品……原因很多，但是结果就是出了问题没人修复，所以尽量不要依赖不维护的项目

py-spy的原理是插针法，也就是通过对当前程序的采样获取每行代码的执行时所占的比例。采样的实现依赖于system_call获取当前内存状态，通过PyInterpreterState和PyFrameObject获取调用栈信息（从而能获取其上一级函数和更上层的函数，是火焰图的基础）

使用方法见项目主页，这边我们只做一个样例

### 第一种典型用法

我们以以下代码为例：

```python
# encoding: utf-8
import random

import numpy
from tqdm import tqdm

long_list = list(range(1000000))


def find_x(x):
    y = numpy.random.rand(2000, 100)
    len_y = y.sum()
    return x in long_list


def run():
    """
    这个函数慢的点在于在长list中进行in操作
    """

    for _ in tqdm(range(100)):
        idx = random.randint(0, len(long_list) - 1)
        find_x(idx)


if __name__ == '__main__':
    run()

```

```bash
py-spy record -o profile.svg -- python cpu_demo.py
```

也就是对于`python function_call_demo.py`这个操作进行profile，输出到profile.svg这个文件中。这个操作可以中间停止，也可以指定只运行一段时间，然后自动停止。

我们可以把profile.svg下载到本地然后打开（默认打开应该是浏览器），能看到这样的火焰图：

![image-20220807212317995](/img/in-post/2022-08-08/image-20220807212317995.png)

这里我们可以看到有两个瓶颈，一个是11行，一个是13行，我们看一下代码，发现一个是生成random矩阵，另一个是list中查找元素，的确会比较慢，符合预期

注意，这里展示的是带行号的模式，也就是到行的profiling，也可以到函数，加一个参数`-F`，参数含义参考help文档

### 第二种典型用法

第二种用法是针对已经在运行的程序，首先我们找到程序对应的pid，那么就可以直接针对这个进程进行profiling：

```bash
py-spy record -o profile.svg --pid 30468
```

因为我们要直接去看另一个进程的状态，所以存在权限问题，按照提示我运行了下面的（最后两个叹号在linux里指的是上一条命令）

```
sudo env "PATH=$PATH" !!
```

显示

> py-spy> Sampling process 100 times a second. Press Control-C to exit.

这意味着我们可以随时暂停，就像之前提到的一样。

我们将输出文件`profile.svg`下载到本地打开，效果和上面的第一种用法类似

### 针对卡住的程序——一种迂回方法

有的时候针对于卡住的程序，我们可以直接针对这个卡住的pid进行profiling，看一下时间消耗在哪一行，就是卡在了哪一行。

### 其他方法

参考项目主页

### 不足

* 直接针对pid进行profiling涉及sudo权限
* 需要每次重新download profile.svg，很恶心
* 目前无法直接和源代码关联起来，看起来略费劲

### 小trick

* 可以在shell配置中加入`alias spy="py-spy"`，将py-spy重命名为spy，这样少打两个字母以及一个横线会很舒服

  

## scalene

scalene是另一个python包，用的方案和py-spy不太一样，是通过signal来进行时间判断的，是一种没有那么精准的方法（但是据作者说已经足够了，我也验证过，日常使用没有问题）。

我们还是需要先安装

```bash
pip install -U scalene
```

### 不足

我们这次先说不足，因为里面有一条会影响我们后面添加的一个参数

* 不支持针对pid的profiling

* 针对memory的profiling不支持python3.6和python3.7，如果直接使用，则会提示：`AttributeError: module 'threading' has no attribute '_shutdown_locks_lock'`，所以，我们一般要加上`--cpu-only`，这虽然说的是`cpu`，但是经过测试，gpu的时间也会被考虑，只是不考虑内存而已。当然，支持mem本就是scalene的特色功能（py-spy是否支持没细看）。

  

### 用法

直接运行下面的程序

```bash
scalene cpu_demo.py
```

（就像之前说的，要是是python3.6使用，要加上`--cpu-only `

完事儿会默认打开浏览器，通过网页形式展现，如图

![image-20220807211521407](/img/in-post/2022-08-08/image-20220807211521407.png)

各项信息都很清晰，可以说是相当现代的一种profiling。

可以通过添加`--cli`实现直接在terminal里展示，更适合在服务器上使用（当然网页也可以，开个端口就好，再说，再说……）

![image-20220807205520052](/img/in-post/2022-08-08/image-20220807205520052.png)



### 时间列意义

这里要注意的是，web 界面的TIME部分，是分成三个组成部分的，分别是python、native和system，对应terminal的三列，这三列的意思分别是：

* python：python语言代码耗费时间
* native：非python代码（典型的比如c语言代码）耗费时间
* system：系统调用时间

这三列加起来，才对应的是运行时间，也就是py-spy里的占比，不能但看一项（这一点不好说是好是差，所以没有放到不足里说，但是终究需要



### 一点疑问

为什么`x in long_list`这句话的GPU活动那么多
