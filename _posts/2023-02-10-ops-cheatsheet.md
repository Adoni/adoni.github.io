---
layout:     post
title:      "onnxruntime-gpu安装"
subtitle:   ""
date:       2023-02-10 10:43:00
author:     "Xiaofei"
header-img: "img/in-post/2023-02-10-ops-cheatsheet/background.jpg"
header-mask: 0.3
catalog:    true
tags:
    - machine learning inference
---



## 问题描述

在安装了onnxruntime-gpu这个包之后，依旧无法使用CUDAExecutionProvider



## 尝试1

重新安装onnxruntime-gpu

```bash
pip uninstall onnxruntime-gpu
pip install onnxruntime-gpu
```



结果是出现了CUDAExecutionProvider，但是当实际运行的时候报错：

```
Failed to load library libonnxruntime_providers_cuda.so with error: libcublasLt.so.12: cannot open shared object file: No such file or directory
```



## 尝试2

在网上查了一下也许是cuda、cudnn版本与onnxruntime不一致导致的。也就是说，onnxruntime的版本决定于cuda+cudnn的版本，参见：







那么怎么查看cuda和cudnn版本呢：

* cuda：使用nvidia-smi或者nvcc -V，或者使用python
* cudnn：直接使用python

既然二者都可以使用python，我就直接用python（准确说是pytorch，没有pytorch的可以查查其他方法）了：

```python
import torch
print(torch.version.cuda)
print(torch.backends.cudnn.version())
```



我这边结果分别是12.1和90100，其中cudnn的90100就是9.x，而我的pytorch的版本是2.6.0，我安装的是1.20.1的onnxruntime-gpu，感觉能和表格里对得上。那么问题是什么呢？



这时候，我想要不要尝试回退到上一个稳定版本，也就是1.19.2，因此：



```bash
pip install onnxruntime-gpu==1.19.2
```



再次运行我的程序，发现已经可以使用CUDAExecutionProvider了



## 结论

先尝试 `pip install onnxruntime-gpu==1.19.2`

