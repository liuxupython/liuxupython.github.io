# Docker 网络

Docker 默认会创建三个网络，分别是 `bridge`、`host`、`none`。

- bridge：桥接网络，Docker 默认使用的网络模式，使用 `docker run` 命令创建容器时如果不指定网络模式，那么就会使用 bridge 模式。
- host：主机网络，使用宿主机的网络，容器将不会获得一个独立的网络命名空间，配置和宿主机共享，容器将不会隔离宿主机网络，使用宿主机的 IP 和端口。
- none：无网络、禁用网络，容器拥有自己的网络命名空间，但是并不为容器进行任何网络配置，这个网络模式的容器只适合于只进行数据处理，没有任何网络的应用场景。



**列出网络**

```
docker network ls
```



**查看网络信息**

```
docker network inspect <network>
```



**创建网络**

```
docker network create <network>
```

- 创建自定义网络时，可以使用参数 `--driver` 指定创建的网络属于什么类型，默认是 `bridge`



**删除网络**

```
docker network rm <network>
```



**运行容器时指定挂载网络**

~~~bash
docker run --network bridge --name my-nginx nginx
~~~

- 不指定 `--network` 即默认挂载在 `bridge`上，即挂载于 `docker0`上。



<br>

### **默认 bridge 网络与自定义 bridge 网络的区别**

- **默认 bridge 网络**：在默认的 `bridge` 网络中，容器之间只能通过 IP 地址通信，无法通过容器名称通信，因为默认 bridge 网络没有内置 DNS 服务。
- **自定义 bridge 网络**：在自定义 bridge 网络中，Docker 提供了 DNS 服务，因此容器之间可以通过容器名称通信。



新建自定义 bridge，并将两个容器挂在上面

```bash
# 创建自定义 bridge 网络
docker network create my-network

# 运行容器并加入自定义网络
docker run -d --name nginx1 --network my-network nginx
docker run -d --name nginx2 --network my-network nginx
```

在 `nginx1` 中，你可以直接通过容器名称 `nginx2` 来 ping 通它，同理反之亦然。Docker 的 DNS 服务会将 容器名称 解析为它的 IP 地址，从而实现通信。



<br>

## docker run --link 不推荐使用

`docker run --link` 是 Docker 早期版本中用于实现容器之间通信的一种机制。它可以将一个容器的网络信息（如 IP 地址和端口）注入到另一个容器的环境变量和 `/etc/hosts` 文件中，从而实现容器之间的通信。但**容器之间是单向通信**。不过，**`--link` 是一个过时的功能**，在 Docker 的现代版本中，推荐使用**自定义网络**（如自定义 `bridge` 网络）来代替 `--link`。

~~~bash
docker run -d --name nginx1  nginx
docker run -d --name nginx2 --link nginx1:mynginx1 nginx
~~~

`--link nginx1:mynginx1`：将 `nginx2` 链接到 `nginx1`，并为 `nginx1` 设置一个别名 `mynginx1`，这样做的结果是，在`nginx2` 可以使用容器名称 `nginx1` 或者 `mynginx1` ping 通过。但是在 `nginx1` 中无法使用容器名 `nginx1` ping 通

