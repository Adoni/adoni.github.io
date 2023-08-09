---
layout:     post
title:      "运维管理小经验"
subtitle:   "不会运维的算法不是好全栈"
date:       2023-02-10 10:43:00
author:     "Xiaofei"
header-img: "img/in-post/2023-02-10-ops-cheatsheet/background.jpg"
header-mask: 0.3
catalog:    true
tags:
    - 运维
---



## 服务器使用

### 查看某个端口对应的进程

```bash
sudo netstat -lnutp | grep port
```



### tmux恢复到屏幕大小

有时候我们在某个屏幕下开了一个tmux的session，结果断了连接后再换一个另一个尺寸的屏幕，发现周围一堆白点，能用的窗口大小被限制在上一个屏幕大小，如图：

<img src="/img/in-post/2023-02-10-ops-cheatsheet/image-20230227202140018.png" alt="image-20230227202140018" style="zoom:50%;" />



这时候可以执行：

```bash
tmux a -d -t name
```



### 设置时间

```bash
sudo rdate -t 60 -s stdtime.gov.hk
sudo hwclock -w
```



### 文件查找

```bash
find . -name "*email*"
```

 



### 磁盘挂载

```bash
fdisk -l
mount /dev/sda /data
```



### 查看DNS

```bash
cat /etc/resolv.conf
```



### install zsh without root

```
wget -O zsh.tar.xz https://sourceforge.net/projects/zsh/files/latest/download
mkdir zsh && unxz zsh.tar.xz && tar -xvf zsh.tar -C zsh --strip-components 1
cd zsh
./configure --prefix=$HOME
make
make install
```



### 其他

1. 命令行里，`Ctrl+a`可以回到一条命令的开始，`Ctrl+e`可以回到结束
2. `ps aux | grep cmd`可以找到cmd对应的进程号，在找到进程号之后，使用`ll /proc/进程号` 可以看到这个进程的详细信息，比如这个cmd的路径
3. 文件（文件夹）按照大小排序：`du -sh * | sort -h `
4. 文件夹下文件数量：`ls | wc -l`



### mac中查看文件编码

```bash
file -I filename
```



### 安装mysqlclient可能遇到问题

尝试：

```undefined
sudo yum install python3-devel mysql-devel
```



### bash历史信息按前缀提示

虽然我很不喜欢用bash，但是总有一些情况需要用到bash（比如帮其他人debug）。而bash让我最讨厌的是按↑键时会直接上一条，而不是匹配我已经输入的前缀找到相同前缀的历史输入，这时候勉强可以：

```shell
bind '"\e[A": history-search-backward'
bind '"\e[B": history-search-forward'
```

这样可以实现类似效果



### 找不到某个.so文件

```bash
find / -name your_lib.so
```



### 找到io瓶颈

https://cloud.tencent.com/developer/article/1718267

```
iostate
```

这个命令可以看读写速度



```
iostat -x 1 1000
```

这个命令可以持续观测io情况



iotop -oP





## Docker

### docker info

`docker info`这个命令可以帮助我们查看一些docker的配置信息，比如`Docker Root Dir`这一行指明了数据的存储路径，根目录所在磁盘比较小的时候就需要修改这一项。



### docker配置信息

配置文件在`/etc/docker/daemon.json`



### docker-proxy

之前`docker-compose down`之后再将服务重新用`docker-compose up`拉起时遇到报错，错误提示是：

> ERROR: for d_fr_en3  Cannot start service d_fr_en3: driver failed programming external connectivity on endpoint d_fr_en3 (f4c63e58fe2221ac2b283f76e75ee5d5cc7790d47f78c528167902a097c20e05): Bind for 0.0.0.0:30390 failed: port is already allocated

意思是相应端口被占用了，但是我明明没有占用，通过看占用的进程，发现是docker，然后用`docker container ls -all`看一下是一个`docker-proxy`在占用，可以通过`ps -aux | grep docker-proxy`这条命令找到。这个是docker自带的proxy，那么我们就需要干掉它，可以直接kill掉，但是治标不治本。其实，我们可以直接停用docker-proxy，这需要修改`/etc/docker/daemon.json`里的配置信息，需要加上`"userland-proxy": false`，例如：

```json
{
    "userland-proxy": false
}
```



### nvidia-runtime

忘了是不是需要手动配置，总之配置完之后在`/etc/docker/daemon.json`里的样子是：

```json
{
    "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}
```



### 设置docker container时间

用docker-compose起容器的时候，时间有可能是utc时间，我们可以通过设置环境变量来改成北京时间，在要设置时间的service底下，加上下面这两行（也就是`environment`和`image`在同一层级）：

```yaml
environment:
  TZ: Asia/Shanghai
```



### 其他

1. p.s. 移除docker-compose里的restart可以减少卡死的概率

2. docker run的时候加上`-u 0`可以以root运行
3. 找到一个container对应的路径：`docker inspect 1d97b7523bb9` 
4. 进入一个container的bash：`docker exec -it 35378c61a875 bash`
5. 可以通过删掉不用的镜像来释放空间：`docker system prune`



## GPU

### 批量kill进程

举个例子，有的时候我们并行训练fairseq，ctrl+c后还有残余的进程，这时候我们想批量杀死

```
nvidia-smi | grep your_feature | awk '{print $5}' | xargs kill
```

* `nvidia-smi`：获取各个显卡上的信息

* `grep your_feature`：这里的your_feature其实是视情况而定的，比如我要是发现我要kill的进程都是某个环境里的，而nvidia-smi恰好有这个环境名字，我就可以grep这个环境的名字。例如，下面这张图里都含有triton，那么我就可以`grep triton`（当然这个例子不够恰当，因为他们的pid都一样可以直接干掉pid，我说的是训练的时候有可能pid是不一样的，比如pytorch的并行就不是一样的）

  <img src="/img/in-post/2023-02-10-ops-cheatsheet/image-20230305210610297.png" alt="image-20230305210610297" style="zoom:50%;" />

* `awk '{print $5}'`：获取上述结果的第五列

* `xargs kill`：从上面的pid来进行kill，这里的xargs命令用将上一步的每一个pid传递给kill命令，在管道命令中很常见



1. root用户启动nvidia-persistenced，可以保持不挂

2. 有的时候在A100上跑【并行】训练会卡住，可以尝试增加这个环境变量：`export NCCL_P2P_DISABLE=1`；另外还可以尝试将Dataloader的worker数量设置成1
3. Triton提示`Failed to allocate memory for requested buffer of size`大概率是显存炸了，比如传入的最大batch太大，可以尝试测一下大batch会不会稳定复现



## 数据库 (MySQL)



### 创建用户并给予权限

```sql
CREATE USER your_user_name IDENTIFIED BY 'your_password'  with MAX_USER_CONNECTIONS 5;
grant select,insert,update,create,index on your_database.* to 'your_user_name';
```



### 修改最大连接数

```
UPDATE mysql.user SET max_user_connections = 20 WHERE user='your_user_name';
FLUSH PRIVILEGES;
```



### 检查用户权限

```sql
SHOW GRANTS for your_user_name
```



### 移除用户权限

```sql
REVOKE update ON your_db_name.your_table_name FROM your_user_name;
```





## Redis Client

### 中文

在使用redis-cli连接redis时，加上`--raw`可以显示中文

对比一下，加之前是

```
"\xe4\xb8\x89\xe4\xbd\x93\xe7\x94\xb5\xe8\xa7\x86\xe5\x89\xa7\xe8\xbf\x91\xe6\x9c\x9f\xe4\xb8\x8a\xe6\x98\xa0\xe4\xba\x86\xef\xbc\x8c\xe5\xa5\xbd\xe8\xaf\x84\xe5\xa6\x82\xe6\xbd\xae\xe5\x95\x8a"
```

加之后是

```
三体电视剧近期上映了，好评如潮啊
```



### 常用的命令

1. `select dbnum`
2. `randomkey`
3. `dbsize`
4. `get your_key`
5. `del your_key`
6. `flushdb`：删除当前db里所有key
