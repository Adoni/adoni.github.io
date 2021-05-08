---
layout:     post
title:      "Matplotlib字体设置"
subtitle:   "matplotlib字体机制、字体缓存、修改字体、中文字体设置"
date:       2021-01-11 14:20:00
author:     "Xiaofei"
header-img: "img/in-post/2021-matplotlib-head.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Python
    - Matplotlib
---



简单记录一下今天在matplotlib中尝试切换字体的过程

## 1. matplotlib 的字体机制

我们举一个简单地例子

假设我们写了这样一段代码：

```python
from matplotlib import pyplot as plt

plt.plot(range(10))
plt.title("哈哈")
plt.show()
```

如果在一台普通的电脑上运行，会有如下结果：

<img src="/img/in-post/2021-matplotlib-0.png" alt="image-20210110213831794" style="zoom: 33%;" />

我们显然可以注意到这里title并没有显示出来。原因是matplotlib的默认字体是英文字体，不包含中文字符集。我们尝试查看一下它的默认字体，运行如下代码

```python
from matplotlib import pyplot as plt
print(plt.rcParams["font.family"])
print(plt.rcParams['font.sans-serif'])
```

我这边的输出结果是：

````
['sans-serif']
['DejaVu Sans', 'Bitstream Vera Sans', 'Computer Modern Sans Serif', 'Lucida Grande', 'Verdana', 'Geneva', 'Lucid', 'Arial', 'Helvetica', 'Avant Garde', 'sans-serif']
````



这里需要解释一下matplotlib里的rcParams，以及这里面的两个重要参数：`font.family`和`font.sans-serif`

根据官方文档，rc是`runtime configuration`的简称，因此rcParams即运行时的相关参数配置，自然包括对于字体的配置。

而根据我们上面的代码，我们知道当前的font.family的值是sans-serif。Sans serif意思是无衬线字体，是一大类字体，因此叫做font family。而plt.rcParams['font.sans-serif']则代表了目前绘图时有哪些字体可以用作sans-serif，同时也定义了他们的搜索顺序。对应的文档在这里：

https://matplotlib.org/3.1.0/gallery/text_labels_and_annotations/font_family_rc_sgskip.html

里面针对rcParams['font.sans-serif']写道：

>  for the font.family you set a list of font styles to try to find in order:
>
>  ```
>  rcParams['font.sans-serif'] = ['Tahoma', 'DejaVu Sans',
>                              'Lucida Grande', 'Verdana']
>  ```

那这里我们明白了什么呢？显然，我们有两种方法去修改字体：

1. 直接修改font.family，这是很多资料里介绍的，然而这样其实是取巧了，或者说是matplotlib帮我们做了一些事情——这时候font.family没有对应的font列表，matplotlib会直接尝试使用font.family作为字体名。其实，matplotlib中，推荐的family只有四种——`[fantasy', 'monospace', 'sans-serif', 'serif']`——最好不要去修改：
2. 修改rcParams['font.sans-serif']：这种写法会更好一些



## 2. 字体缓存天坑

但是，在我们真正修改字体之前，我们需要先知道有哪些字体可以作为['font.sans-serif']，这里有一个大坑——matplotlib的font是有缓存的，因此，即使你安装了新字体，可能也不会显示，反而会提示：

> findfont: Font family ['sans-serif'] not found. Falling back to DejaVu Sans.

为了解决这个问题，我们在安装新字体后，可以运行如下代码以更新font缓存（虽然只需要运行一次，但是我还是建议直接写到画图程序里）

```python
import matplotlib.font_manager as font_manager
font_manager._rebuild()
```



## 3. 查找字体名

在设置之前，我们还是需要知道自己的目标字体在系统中的名字，比如我非常喜欢的一款字体——造字工坊格黑体，我从官网下载并更新后，该怎么知道对应的字体名呢？难道直接写`"造字工坊格黑体"`吗？显然不是的。

这里有几种方法：

1. mac上通过字体册应用找到你电脑上字体的安装位置，单击，就能看到名字：

   ![image-20210110224129338](/img/in-post/2021-matplotlib-1.png)

2. 运行以下代码：

   ```python
   from matplotlib import font_manager
   for font in font_manager.fontManager.ttflist:
       print(font)
   ```

   这会输出所有字体，然后直接搜索即可

   <img src="/img/in-post/2021-matplotlib-2.png" alt="iShot2021-01-10 22.47.40" style="zoom:50%;" />

   

## 4. 设置字体

搞定上述两步——字体更新和查找字体名——之后，我们就可以正常设置字体了：

```python
from matplotlib import pyplot as plt
import matplotlib.font_manager as font_manager  # 这一行和下一行运行后就可以不再重复写了，这里写了只是为了保险
font_manager._rebuild()  # 可以省略, 见上一行

plt.rcParams['font.sans-serif'] = ['MFGeHeiNoncommercial']
plt.plot(range(10))
plt.title("哈哈")
plt.show()
```





<img src="/img/in-post/2021-matplotlib-3.png" alt="image-20210110222513262" style="zoom: 33%;" />

可以看到这里title已经正常设置了。