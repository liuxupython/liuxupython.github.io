# 安装Docker

Docker可以安装在 Windows、Mac、linux上，本文只介绍如何在 Mac 和 Linux 上安装和学习 Docker

```alert type=note
Docker可以在 Windows 安装和使用，但需要很多繁琐的配置，很容易因为系统的问题安装失败。个人不推荐在 Windows 上安装和使用 Docker。 
```



## Mac 安装 Docker

Mac 安装 Docker 非常方便，可以直接到 [Docker 官网](https://www.docker.com/)下载最新版本的 Docker Desktop，然后点击安装。安装完成后，在启动台中找到 Docker 图标，点击启动 Docker 即可。随后就可以在终端中使用 Docker 指令，也可以在 Docker Desktop 中管理镜像和容器等。

![docker-mac-install](docker-mac-install.png)



## Ubuntu 安装 docker

本教程选择使用腾讯云 Linux 虚拟机，购买一台 Ubuntu 服务器，作为学习使用，基础配置即可（2核2G内存）。如果首次购买，或者学生身份购买，或者试用，价格都很实惠。

大家可能都知道，因为某些原因，需要科学上网才可以无障碍的使用 Docker，下载 Docker 镜像，这也是选择云服务器作为教程机的原因。因为在云服务器上可以通过国内镜像源，下载使用 Docker 镜像。起码目前还能使用，且行且珍惜！！！

```alert type=note
本节安装说明适用于在腾讯云云服务器上安装 docker

如果有条件科学上网，可以参考 docker 官网安装文档
https://docs.docker.com/engine/install/ubuntu/
```



### 第一步：购买云服务器

本教程使用的是腾讯云的 轻量应用服务器，**CPU 2核，内存 2G，默认系统盘 40G 的 Ubuntu 24.04 LTS**。对于初学者，这个配置是足够的，根据自己的情况和经济实力购买(试用)即可。整个购买过程都是很简单的。



<br>

**安装 Docker 的方法**

1. Docker Desktop for Linux 这是最简单、最快捷的入门方式。
2. apt 设置并安装 Docker 。
3. 使用 [便捷脚本](https://docs.docker.com/engine/install/ubuntu/#install-using-the-convenience-script)。仅推荐用于测试和开发环境。

这三种方式各有利弊：

- 方式1 是最简单的，但是在云服务器上没有桌面系统不能使用 Desktop 的方式。
- 方式2 适合流畅访问外网的用户，在国内网络下用这种方式安装会非常缓慢。
- 方式3 适合初学者学习，安装相对简单，并且可以使用国内镜像站。【**推荐的方式**】

<br>

### 第二步：安装 Docker（脚本的方式）

在命令行中执行如下指令

```bash
// 下载脚本
ubuntu@VM-12-14-ubuntu:~$ curl -fsSL https://get.docker.com -o get-docker.sh
ubuntu@VM-12-14-ubuntu:~$ ls
get-docker.sh

// 安装docker
ubuntu@VM-12-14-ubuntu:~$sudo sh get-docker.sh --mirror Aliyun
```



### 第三步：验证 docker 安装成功

~~~bash
ubuntu@VM-12-14-ubuntu:~$ sudo docker version
ubuntu@VM-12-14-ubuntu:~$ sudo docker compose version
ubuntu@VM-12-14-ubuntu:~$ sudo docker pull nginx
~~~





### 第四步：检查 daemon.json

Docker 安装后了之后需要测试是否可以下载镜像，如果因为镜像源的问题不能下载则需要配置腾讯云官方镜像源。

检查是否存在文件：`/etc/docker/daemon.json` ，如果没有则新建这个文件，并把如下配置添加到文件中。

```bash
ubuntu@VM-12-14-ubuntu:~$ cat /etc/docker/daemon.json 
{
    "registry-mirrors": [
        "https://mirror.ccs.tencentyun.com"
    ]
}
```

需要重启 Docker 服务才能生效

~~~bash
sudo systemctl restart docker
~~~



```alert type=note
使用 systemctl 管理 docker 服务的常用指令 <br>
```sudo systemctl start docker``` <br>
```sudo systemctl stop docker```  <br>
```sudo systemctl restart docker```<br>
```sudo systemctl status docker```<br>
```sudo systemctl enable docker```<br>
```





### 第五步： 验证换源成功

~~~bash

ubuntu@VM-12-14-ubuntu:~$ sudo docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
c29f5b76f736: Pull complete 
e19db8451adb: Pull complete 
24ff42a0d907: Pull complete 
c558df217949: Pull complete 
976e8f6b25dd: Pull complete 
6c78b0ba1a32: Pull complete 
84cade77a831: Pull complete 
Digest: sha256:91734281c0ebfc6f1aea979cffeed5079cfe786228a71cc6f1f46a228cde6e34
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
ubuntu@VM-12-14-ubuntu:~$ sudo docker images
REPOSITORY   TAG       IMAGE ID       CREATED      SIZE
nginx        latest    97662d24417b   9 days ago   192MB
~~~

如果看到上述反馈内容，则表明你的 Docker 可以正常使用了，恭喜🎉🎉🎉。
