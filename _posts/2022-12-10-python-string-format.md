---
layout:     post
title:      "Python String Format"
subtitle:   "fstring! fstring! fstring!"
date:       2022-12-10 10:43:00
author:     "Xiaofei"
header-img: "img/in-post/2022-12-10/background.jpg"
header-mask: 0.7
catalog:    true
tags:
    - Python
---



## python字符串格式化的几种方案

string是我们写代码绕不开的基础类型，而构造string也是我们经常遇到的场景。

在我刚刚接触python的时候，学到的构造string的方法是C或C++的风格，即通过`%d, %s, %f`等特殊字符构造，然后通过一个`%`将参数与占位符一一对应，比如这样

```python
a = 1
b = "hello"
print("a=%d, b=%s" % (a, b))
```

紧接着，逐渐学习到format的用法，最大的好处是不需要区分类型，例如：

```python
a = 1
b = "hello"
print("a={0}, b={1}".format(a, b))
```

这种写法在计数的那部分略微冗余，可以简写成这样：

```python
a = 1
b = "hello"
print("a={}, b={}".format(a, b))
```

但是数量不匹配的时候还是要写明数字，比如这种情况：

```python
a = 1
b = "hello"
print("a={0}, b={1}, a={0}".format(a, b))
```

format还支持用键值的形式，比如：

```python
a = 1
b = "hello"
print("a={a}, b={b}, a={a}".format(a=a, b=b))
```

这其实已经和fstring很像了，但是依旧有一个巨大问题没有解决——不够直观，即变量和string整体是分离的，需要程序员人眼去对应。



这时候，fstring出现了。

fstring是formatted string的缩写，也就是格式化的字符串，可以将变量直接嵌入到string中构造新的string。比如上面的代码可以写成：

```python
a = 1
b = "hello"
print(f"a={a}, b={b}, a={a}")
```

简洁了很多！



而且，fstring还有一个好处——速度！我一开始知道的时候也很吃惊，fstring居然是众多python字符串格式化方案中最快的方式，做到了兼具速度和可读性。该论断来自于：https://www.scivision.dev/python-f-string-speed/ 和https://www.reddit.com/r/Python/comments/pivojb/performance_comparison_of_string_formatting/



## fstring与string.format混用

fstring的一大限制是在构造字符串的时候，所有变量的值就已经有了，但是如果我想准备一个模板，然后里面的key是随着程序运行才逐渐赋值的（比如通过for循环或者用户输入拿到不同的值），该怎么办？这时候，比较常见的用法是通过之前提到的键值形式的format方法。

例如：

```python
template = "a={a}, b={b}, a={a}"
for a in range(10):
    print(template.format(a=a, b="hello"))    
```

但是如果我的原始template中的某个字符串也是变量呢：例如我要针对b="hello"和b="world"构造两个template，要手写吗？如果有200个怎么办，其实，fstring也可以保留`{}` 这种占位符的，例如下面的代码：

```python
b = "hello"
template = f"a={{a}}, b={b}, a={{a}}"
print(template)
```

会发现template的值是`"a={a}, b=hello, a={a}"`，这样后续的format就顺理成章了。





## fstring的格式控制

“格式控制”这里主要包括：

* 小数位数
* 左侧填充占位符（对齐）和右侧填充占位符（对齐）
* 日期

格式控制主要是靠变量后面的冒号和格式来实现的。



### 小数位数

```python
a = 11.123456789
print(a)
print(f"{a:0.2f}")
```

输出为

```
11.123456789
11.12
```



### 左侧填充占位符（对齐）和右侧填充占位符（对齐）

填充占位符的用法可以总结为：

```python
f"{变量:占位符>整体长度}"
```

这里需要注意，占位符应当是单个字符（因为python3是unicode，所以可以是中文），其中>代表左填充，变成<的话，就变成了右填充。

例子：

```python
a = "hello"
print(f"{a:*>10}")
print(f"{a:*<10}")
print(f"{a:0>10}")
a = 1
print(f"{a:0>4}")
```

输出为：

```
*****hello
hello*****
00000hello
0001
```

最后一条的效果等价于`str(a).zfill(4)`。

另外，在变量为数字的情况下，且占位符为空格或者0的时候，可以不写`>`或者`<`，例如：

```
a = "hello"
print(f"{a: 10}")
print(f"{a:010}")
```



但是我个人感觉这样不够清晰，而且适用情况太单一，遇到复杂情况还是需要加上`>`或者`<`，莫不如直接只记一种用法。



另外注意上面的写法中的占位符和长度分别是普通字符和数字。填充的占位符或长度也可以是变量，但是需要将上面的用法更改为：

```
f"{变量:{占位符变量}>{整体长度变量}"
```

比如下面这个例子里，我们一次将占位符设置为空格、星号、井号，输出也相应地以三种字符进行填充

```python
num_list = [10, 100, 1000, 10000]
symbol_list = [" ", "*", "#", "dd"]
length = 10
for symbol in symbol_list:
    for num in num_list:     
		print(f"{num:{symbol}>{length}}")
```

输出为

```
        10
       100
      1000
     10000
********10
*******100
******1000
*****10000
########10
#######100
######1000
#####10000
```



### 日期

```python
from datetime import datetime
date = datetime(2022, 1, 2, 3, 4 ,5)
print(date)
print(f"year and month: {date:%Y/%m}")
```

输出为

```
2022-01-02 03:04:05
year and month: 2022/01
```
