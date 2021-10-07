---
layout:     post
title:      "新机器配置记录"
subtitle:   "开发用机器"
date:       2021-05-08 10:43:00
author:     "Xiaofei"
header-img: "img/in-post/2021-09-18/config.jpg"
header-mask: 0.3
catalog:    true
tags:
    - 基本设置
    - CentOS
---



> 最近遇到了需要重新配置机器的机会，因此特地记录一下

> 请注意：在本篇中，我们统一使用`yourname`作为用户名



## 添加用户

假设你获得的是root用户，那么第一件事情就是创建用户，并伴随可能的添加用户权限等

1. 创建用户：`sudo useradd yourname`
2. 如果是CentOS，你还需要修改密码：`sudo passwd yourname`



## 为用户添加sudo权限

ubuntu用户

`sudo usermod -aG sudo yourname` 

centos用户

`sudo usermod -aG wheel yourname` 



## 完成SSH Key登录

将你电脑上的publish key（一般在`id_rsa.pub`中）粘贴到`.ssh/authorized_keys`中。尝试使用ssh key登录，如果不行，可以先检查文件夹和文件的权限，`.ssh`文件夹需要700权限，`authorized_keys`需要600权限，修改权限的方法如下：

```
chmod 700 .ssh

chmod 600 .ssh/authorized_keys
```

生成key的方法，见其他



## 修改shell为zsh并安装oh-my-zsh

安装zsh很简单：

```
sudo yum install zsh
```



## 安装oh-my-zsh

首先，在国内可能连不上github，所以需要修改github对应的ip，方法是将下面这句放入`/etc/hosts`里，这个需要sudo权限

 ```
 199.232.68.133 raw.githubusercontent.com
 ```

然后用下面这行命令安装oh-my-zsh

`sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"`

多试几次一般会成功





## 安装docker

注意这里安装的是docker engine，详情见

https://docs.docker.com/engine/install/

选择合适你的发行版进行安装，我的是centos，这里记录如下：

```
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
sudo yum install -y yum-utils
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce docker-ce-cli containerd.io
```

然后启动docker

```
sudo systemctl start docker
```

验证方式很简单：

```
sudo docker run hello-world
```

这个可能涉及到下载镜像，等一下就好

大家可能看到，上一步验证需要用到sudo，实际上运行docker可以通过添加当前用户到docker组的形式避免sudo，这也是我们推荐的方式，步骤如下：

```
sudo groupadd docker  # 如果提示组存在，那么就继续下一步就好
sudo usermod -aG docker yourname
```

然后退出后重新登录，一定要【重新登录！！！】

此时再验证一下：

```
docker run hello-world
```



番外：如果你这里发现docker所处的挂载点不够大或者运行的时候提示空间不足，可以切换docker的`Docker Root Dir`，方式见其他



## 安装nvidia-docker2

目的是在镜像中使用gpu

```
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
   && curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.repo | sudo tee /etc/yum.repos.d/nvidia-docker.repo

sudo yum clean expire-cache
sudo yum install -y nvidia-docker2
```

最后一条命令可能由于网络问题失败，多试几次就好



重启docker

```
sudo systemctl restart docker
```

验证一下

```
docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi
```



## 安装docker-compose

到https://github.com/docker/compose/releases中下载你需要的版本，我这里用的是1.29.2，也就是v1里的最后一个版本，v2打算观望一下再使用。下载你需要的版本，我这里下载命令是：

```
wget https://github.com/docker/compose/releases/download/1.29.2/docker-compose-Linux-x86_64
```

下载后，直接重命名为 /usr/local/bin/docker-compose即可：

```
chmod +x docker-compose-Linux-x86_64
sudo mv docker-compose-Linux-x86_64 /usr/local/bin/docker-compose
```

验证一下

找个目录，将以下内容写入`docker-compose.yml`

```
services:
  test:
    image: nvidia/cuda:11.0-base
    command: nvidia-smi
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              capabilities: [gpu, utility]
```

然后在同一目录下执行docker-compose up，应该可以看到打印的gpu信息





## 其他

### 无sudo安装zsh

```
wget -O zsh.tar.xz https://sourceforge.net/projects/zsh/files/latest/download
mkdir zsh && unxz zsh.tar.xz && tar -xvf zsh.tar -C zsh --strip-components 1
cd zsh
./configure --prefix=$HOME
make
make install
```



### 生成ssh key



### 修改Docker Root Dir

如果你这里发现docker所处的挂载点不够大或者运行的时候提示空间不足，可以切换docker的`Docker Root Dir`，方式如下：

1. 使用`docker info`查看当前的Docker Root Dir，与挂载点所剩空间作对比，确认自己的【空间不足】真的是Root Dir所在位置引起的

2. 修改daemon配置，具体而言是

   * `sudo vim /etc/docker/daemon.json`，注意这是一个json文件，需要满足基本的json语法，比如以大括号作为首尾

   * 加入这一行：`"data-root": "/data/docker"`，again，注意这是个json文件，如果你之前没有这个文件，那么你的文件应该长这样子：

     ```json
     {
       "data-root": "/data/docker"
     }
     ```

   * 重启docker：`sudo systemctl restart docker`

3. 再次使用`docker info`查看当前的Docker Root Dir，这时候应该已经改变了

NOTE：任何时候提示`no space left on device`，就去看一下docker info的Docker Root Dir

