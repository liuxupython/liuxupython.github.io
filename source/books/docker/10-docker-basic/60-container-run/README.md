# 运行容器

运行容器一般指的是 `docker run` 命令，这个命令非常重要，主要体现在它可以配合很多参数实现不同的功能。

```
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
# OPTIONS:
# -d: 后台运行容器
# -i: 以交互模式运行容器，通常与 -t 同时使用
# -t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用
# --name: 为容器指定一个名字
# --rm: 容器退出后自动删除容器文件
# -p: 端口映射，格式为：主机(宿主)端口:容器端口
# -v: 挂在数据卷
# -e: 为容器设置环境变量
# -w: 指定容器的工作目录
# --network: 指定容器的网络连接类型
# --link: 把一个容器的网络和另一个容器联通（单向）
```

我们可以做如下尝试，体验体验 `docker run` 的强大之处。

1. 创建一个 nginx 容器

~~~bash
docker run nginx
~~~



2. 创建一个 nginx 容器，指定：后台运行

~~~bash
docker run -d nginx
~~~



3. 创建一个 nginx 容器，指定：后台运行、设置环境变量

~~~bash
docker run -d -e XYZ=hello nginx
~~~



4. 创建一个 nginx 容器，指定：后台运行、设置环境变量、设置工作目录

~~~bash
docker run -d -e XYZ=hello -w /root nginx
~~~



5. 创建一个 nginx 容器，指定：后台运行、设置环境变量、设置工作目录，自定义容器名字

~~~bash
docker run -d -e XYZ=hello -w /root --name my-nginx nginx
~~~



6. 创建一个 nginx 容器，指定：后台运行、设置环境变量、设置工作目录，自定义容器名字，设置推出时自动销毁

~~~bash
docker run -d --rm -e XYZ=hello -w /root --name my-nginx nginx
~~~

