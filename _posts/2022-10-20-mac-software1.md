---
layout:     post
title:      "神兵利器-1"
subtitle:   "我的Mac装机清单"
date:       2022-10-20 10:43:00
author:     "Xiaofei"
header-img: "img/in-post/2022-10-20-mac-software/background.jpg"
header-mask: 0.7
catalog:    true
tags:
    - Tools
---



> 非常非常个人向的软件清单，我会表达自己对于一些软件的看法，如果和您的看法不一致，大家求同存异。
>
> 为什么强调个人向呢，主要是因为自己对于一些东西可能存在偏执，我对于大而全的软件一向兴趣泛泛，比如vim和emacs，大学的时候我就和朋友做过讨论，发现谁也说服不了谁，我还是喜欢用vim，他则是鉴定的emacs党。另外我比较喜欢GUI的东西，比如git，我喜欢看界面而非用命令行。



## typora

感觉是mac上最好用的markdown编辑器，喜欢它的原因基本是如下三点：

* 一来是所见即所得的模式，从感官上让人很舒服，尤其是表格、代码框和图片，能不见到那一大堆纯文本真的舒坦。我之前用过macdown是左右分栏的，不是很舒服

* 二来是比较简单的界面，既得益于markdown（不需要各种按钮）的语法，也来源于其功能的单一性（与之形成对比的是基于类似vscode这样的编辑器+插件实现的markdown写作，或者一些IDE自带的markdown渲染功能）

* 最后是机器快速的启动速度，应该是原生app，而非electron或者react native等手段（比如marktext）



> 最近貌似强制收费了，思来想去，还是付费了



### 序号

我喜欢在markdown里从二级标题开始记录序号，可以通过配置css自动做到这一点，参考https://support.typora.io/Auto-Numbering/

简单来说，通过【偏好设置>外观>打开主题文件夹】来到主题文件，编辑[theme].css（比如github.css），将下面这段贴到最后（我自己喜欢从二级标题开始，所以对参考链接里的内容做了一点点修改）

```css
/** initialize css counter */
#write {
    counter-reset: h2
}

h2 {
    counter-reset: h3
}

h3 {
    counter-reset: h4
}

h4 {
    counter-reset: h5
}

h5 {
    counter-reset: h6
}

/** put counter result into headings */

#write h2:before {
    counter-increment: h2;
    content: counter(h2) ". "
}

#write h3:before,
h3.md-focus.md-heading:before /** override the default style for focused headings */ {
    counter-increment: h3;
    content: counter(h2) "." counter(h3) ". "
}

#write h4:before,
h4.md-focus.md-heading:before {
    counter-increment: h4;
    content: counter(h2) "." counter(h3) "." counter(h4) ". "
}

#write h5:before,
h5.md-focus.md-heading:before {
    counter-increment: h5;
    content: counter(h2) "." counter(h3) "." counter(h4) "." counter(h5) ". "
}

#write h6:before,
h6.md-focus.md-heading:before {
    counter-increment: h6;
    content: counter(h2) "." counter(h3) "." counter(h4) "." counter(h5) "." counter(h6) ". "
}

/** override the default style for focused headings */
#write>h3.md-focus:before,
#write>h4.md-focus:before,
#write>h5.md-focus:before,
#write>h6.md-focus:before,
h3.md-focus:before,
h4.md-focus:before,
h5.md-focus:before,
h6.md-focus:before {
    color: inherit;
    border: inherit;
    border-radius: inherit;
    position: inherit;
    left:initial;
    float: none;
    top:initial;
    font-size: inherit;
    padding-left: inherit;
    padding-right: inherit;
    vertical-align: inherit;
    font-weight: inherit;
    line-height: inherit;
}
```



### 格式化功能

Typora有一个衍生的功能，就是可以对markdown进行格式化，最典型的是对表格的格式化。我们可以直接将表格贴进去，然后直接再复制，粘贴到其他地方，就会发现已经格式化完毕了。

例如，我们非格式化的表格长这样

```markdown
| Tables   | A1  |  A2 |
|----------|-------------|------|
| B1 |  aaaaaaaaaaaaaaaaaaaa | 1600                        |
| B 2 |    bb   |   $12 |
| ddddddddddddB 3 | cccccccccccccccccc |    1 |
```

粘贴进去再复制出来，结果是：

```markdown
| Tables          | A1                   | A2   |
| --------------- | -------------------- | ---- |
| B1              | aaaaaaaaaaaaaaaaaaaa | 1600 |
| B 2             | bb                   | $12  |
| ddddddddddddB 3 | cccccccccccccccccc   | 1    |
```



### 代码块和中文输入法

中文输入法下，想要输入代码块一般而言要：

1. 切换英文输入法
2. 输入三个反引号\`\`\` 
3. 回车
4. 切换回原本输入法（如果需要的话）

反正我个人极其不喜欢输入法的切换，所以我利用mac的快捷短语的功能实现输入···直接转换成\`\`\` 的功能。

同样道理还可以实现两个反引号的输入。

<img src="/img/in-post/2022-10-20-mac-software/image-20230103165903558.png" alt="image-20230103165903558" style="zoom:50%;" />

## Apifox

（以下表述可能过时，毕竟很久没用过postman了）

再见Postman，我要拥抱apifox了，比较喜欢的一些点：

* 区分测试环境、正式环境
* 支持同一个接口多个test case
* 启动速度比postman快太多



## jetbrains全家桶

说是全家桶，我比较熟悉的也就是PyCharm和Datagrip，尤其是PyCharm。PyCharm是面向python的IDE，从个人角度，我非常不建议使用VSCode自己配置python开发环境，原因包括：

* 繁琐：vscode毕竟是一款编辑器，额外功能需要使用插件完成，先不论插件质量如何，单是配置，就够恶心的了。当然这个主要和之前提到的个人习惯有关，我更喜欢术业有专攻的软件，不喜欢大而全的软件
* 丑：vscode的样子不符合我的审美

总之，我的基本观点就是，普通开发人员，不要折腾vscode



> Pycharm2022.2.X版本貌似对ssh interpreter做了较大的改动，我还在2022.1.4，升级了几次都感觉很恶心

> 应用一位师兄的话：“pycharm用的远程开发策略太落后了”，不过幸亏我喜欢代码在本地，而非远端，所以目前没有啥问题



## draw.io

用来画图，非常爽。本来我以为这玩意儿非常小众，知道有一次一位其他公司的合作者让我给他发架构图的原始文件，问我能不能导出成draw.io的格式，我才发现这玩意儿还是挺流行的。

draw.io之前是一个网站，但是现在又app（应该不是原生app，类似electron封装的）



## SourceTree

之前提到，我比较喜欢GUI的东西，不喜欢通过命令行，对于git操作也是，或者说尤其是，毕竟git在GUI上更加直观一些。这个软件是Atlassian家开发的，就是收购BitBucket的那家，感觉很帅

值得一提的是，SourceTree的windows版使用的默认ssh是putty，而非openssh，要改变的话需要到【工具>选项>一般>SSH客户端配置】中设置

<img src="/img/in-post/2022-10-20-mac-software/image-20221125200647497.png" alt="image-20221125200647497" style="zoom:50%;" />



## iTerm2<0.3.12

基本上算是超级知名的mac软件，然而这货在0.3.12的版本里一直有一个bug，发现bug后我就一直降级使用了，之后修复了再说。这个bug简单来说就是多行内容划线选中后拷贝文字会多一些换行符，很恶心。



## Office系列

基本上就是三件套Word、Powerpoint、Excel，Excel用的略少，其他Outlook之类的不用。

虽然我对于原生软件比较喜欢（比如日历和提醒事项），但是pages和keynote还是再见吧，还是office的通用性更强一些，工作上我总不能用word写完一个文件然后发给一个用windows的同事吧，而且office真的做的不错，我印象中是萨蒂亚·纳德拉当CEO后品质迅速上升的，也可能是错觉。而且微软一向被誉为苹果最佳开发者，当时M1芯片刚出的时候微软也是很快就完成了支持，就感觉品质有保障。



## MacVim

打开一些txt、log、数据文件啥的，需要一个文本编辑器，之前我用的是vscode，然而启动太慢，我又不能容忍它关闭不退出放在程序坞里占一个位置，所以vim几乎就是唯一选择了，不过好在macvim软件很棒，非常稳定，用起来很舒服。

Vim配置我也选简单的，毕竟我也不打算用来写过多代码，配置如下

```
syntax enable
"colorscheme monokai
colorscheme onedark

set guifont=Monaco:h18
set nu

"设置折叠，但是由于会影响秒开，所以关闭
"set foldmethod=syntax
"set foldlevel=5
"set foldenable
"nnoremap <space> za


if has("gui_macvim")
  " Press Ctrl-Tab to switch between open tabs (like browser tabs) to 
  " the right side. Ctrl-Shift-Tab goes the other way.
  noremap <C-Tab> :tabnext<CR>
  noremap <C-S-Tab> :tabprev<CR>

  " Switch to specific tab numbers with Command-number
  noremap <D-1> :tabn 1<CR>
  noremap <D-2> :tabn 2<CR>
  noremap <D-3> :tabn 3<CR>
  noremap <D-4> :tabn 4<CR>
  noremap <D-5> :tabn 5<CR>
  noremap <D-6> :tabn 6<CR>
  noremap <D-7> :tabn 7<CR>
  noremap <D-8> :tabn 8<CR>
  noremap <D-9> :tabn 9<CR>
  " Command-0 goes to the last tab
  noremap <D-0> :tablast<CR>
endif

set expandtab           " enter spaces when tab is pressed
set textwidth=120       " break lines when line length increases
set tabstop=4           " use 4 spaces to represent tab
set softtabstop=4
set shiftwidth=4        " number of spaces to use for auto indent
set autoindent          " copy indent from current line when starting a new line
set smartindent
set smarttab
set expandtab
set number

" make backspaces more powerfull
set backspace=indent,eol,start

set ruler                           " show line and column number
syntax on               " syntax highlighting
set showcmd             " show (partial) command in status line
set showmatch
```



## iShot

截图，工具，比微信、钉钉自带的都好不少（好吧，我也没深度用过），比较喜欢的功能点：

* 滚动截图
* 序号标注
* 截图钉住在屏幕上

最后一个功能可能没说明白，图示如下

![image-20221125202853450](/img/in-post/2022-10-20-mac-software/image-20221125202853450.png)



## 自动切换输入法

是的，app名字就叫“自动切换输入法”，极其直白……和iShot是同一家出的，这家的app有的名字很漂亮，有的就很直白（比如还有一个app叫LiuHai，就是专门美化macbook刘海的……，一度怀疑是不是两拨产品经理酷😂）。作用很简单，就是给每个app规定一个默认输入法，切换到这个app的时候自动切换输入法（产品经理：名字这不就来了吗……）。

举个例子，我在PyCharm用英文输入法写代码呢，结果钉钉有人给我发消息，我切换回钉钉，这时候需要切换输入法，但是装了这个软件后，我就啥都不用干了，默认就回到中文输入法了。回复完毕，回到PyCharm页面，又自动切换为了英文输入法。

