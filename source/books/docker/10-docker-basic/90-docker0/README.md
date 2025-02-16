# docker0网卡

docker安装并启动服务后，会在宿主机中添加一个虚拟网卡， `docker0` 网卡。这个 `docker0` 就是 `bridge` 网络。

>使用 `ifconfig` 或 `ip addr` 查看网卡信息



默认情况下通过 `docker run` 运行的容器都挂在 `docker0` 网卡上，即它们之间是在一个 ip 段上，网络互通。换句话说，换句话说在这些容器内可以通过 ip 互联。并且以为 `docker0` 在其中起到了连接容器和宿主机的作用，容器和宿主机之间也可以互联。

启动两个 nginx 容器

~~~bash
docker -d --name nginx1 nginx
docker -d --name nginx2 nginx

docker ps
CONTAINER ID   IMAGE                          COMMAND                   CREATED          STATUS                    PORTS                  NAMES
c2c1607b684d   nginx                          "/docker-entrypoint.…"   27 seconds ago   Up 26 seconds             80/tcp                 nginx2
07b78d04956b   nginx                          "/docker-entrypoint.…"   43 seconds ago   Up 40 seconds             80/tcp                 nginx1           80/tcp                 nginx1
~~~

通过 `ip add` 查看容器 `docker0` 网卡所在的 ip 段是 `172.17.0.1/16`

查看 `docker0`网络上挂载的容器，可以看到两个容器的 ip

~~~bash
docker network inspect bridge
~~~



测试 宿主机访问容器 ip，容器访问宿主机器ip，容器之间互访ip

- 使用 `ip add` 查看IP地址，使用`ping` 测试是否可以访问

~~~bash
[root@VM-12-14-centos ~]# ping 172.17.0.3 -c 2
PING 172.17.0.3 (172.17.0.3) 56(84) bytes of data.
64 bytes from 172.17.0.3: icmp_seq=1 ttl=64 time=0.039 ms
64 bytes from 172.17.0.3: icmp_seq=2 ttl=64 time=0.042 ms

--- 172.17.0.3 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.039/0.040/0.042/0.006 ms
~~~



<br>

# 容器内安装网络工具



Apt-get 换源

>参考：
>
>https://mirrors.tencent.com/help/debian.html
>
>https://mirrors.tuna.tsinghua.edu.cn/help/debian/

- 因为是在腾讯云服务器上，直接使用内网地址（http://mirrors.tencentyun.com）
- 修改 `/etc/apt/sources.list.d/debian.sources` 为如下内容

~~~bash
Types: deb
URIs: http://mirrors.tencentyun.com/debian-security
Suites: bookworm-security
Components: main
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg

Types: deb
URIs: http://mirrors.tencentyun.com/debian
Suites: bookworm bookworm-updates bookworm-backports
Components: main contrib non-free non-free-firmware
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg
~~~



容器内安装工具

- 使用 `ip` 命令，需要安装 `iproute2`
- 使用 `ping` 命令，需要安装 `iputils-ping`
- 使用 `ifconfig` 命令，需要安装`net-tools`



~~~bash
apt-get clean all
apt-get update
apt-get install -y iproute2
apt-get install -y iputils-ping
~~~



<br>

# 虚拟网卡对 veth pair

顾名思义，veth-pair 就是一对的虚拟设备接口，它都是成对出现的。一端连着协议栈，一端彼此相连着

Docker 创建一个容器的时候，会执行如下操作：

 • 创建一对veth pair，分别放到本地主机和容器中。
 • 本地主机一端桥接到默认的 docker0 或指定网桥上，并具有一个唯一的名字，如 vethced5325；
 • 容器一端放到新容器中，并修改名字作为 eth0，这个网卡/接口只在容器的名字空间可见；
 • 从网桥可用地址段中（也就是与该bridge对应的network）获取一个空闲地址分配给容器的 eth0，并配置默认路由到桥接网卡 vethced5325。

完成这些之后，容器就可以使用 eth0 虚拟网卡来连接其他容器和其他网络。如果不指定--network，创建的容器默认都会挂到 docker0 上，使用本地主机上 docker0 接口的 IP 作为所有容器的默认网关。

<img src="https://raw.githubusercontent.com/bigbugboy/pic/main/img/docker0.png?s" alt="docker0" style="zoom:67%;" />

