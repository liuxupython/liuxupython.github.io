# 数据卷

在 Docker 中，数据卷挂载是一种常见的技术，用于将宿主机上的目录或文件挂载到容器内部，从而实现容器与宿主机之间的数据共享和持久化存储。数据卷可以绕过容器文件系统，使数据在容器删除后仍然存在。

在 Docker 中，数据卷可以通过两种方式进行创建和使用：

1. 使用 `docker volume create` 命令创建数据卷。
2. 使用 `docker run -v` 创建数据卷并挂载到容器中。



## docker volume create 方式

**创建一个数据卷**

```
docker volume create <volume-nme>
```

**查看数据卷**

```
docker volume ls
```

**查看数据卷详细信息**

```
docker volume inspect <vlume-name>
```

**删除数据卷**

```
docker volume rm <volume-name>
```

**将数据卷挂载到容器**

~~~bash
docekr run -d -v mydata:/data <image>
~~~

- 这样会把名字为 `mydata` 的数据卷挂载到容器中的 `/data` 目录上。

<br>



## docker run -v 方式

```bash
docker run -v [host_path]:[container_path] [image_name]
```

- `-v [host_path]:[container_path]`: 这里 `[host_path]` 是宿主机上的路径，`[container_path]` 是容器内部的路径。这将会把宿主机上的 `[host_path]` 目录挂载到容器内的 `[container_path]` 目录上。
- `[image_name]`: 要运行的 Docker 镜像名称。

例如，如果要将宿主机的 `/opt/data` 目录挂载到容器内的 `/data` 目录，可以这样运行容器：

~~~bash
docker run -v /opt/data:/data [image_name]
~~~



<br>



~~~alert type=important
**`docker volume create` 方式**：<br>

 - **创建数据卷**：使用 `docker volume create` 命令可以创建一个独立的数据卷。<br>
- **命名自定义**：可以为数据卷指定自定义名称，以便于识别和管理。<br>
- **独立存在**：创建的数据卷是独立于容器的，可以在多个容器之间共享使用。<br>
- **持久化存储**：数据卷中的数据在容器删除后仍然存在，可以实现数据的持久化存储。<br><br>

**`docker run -v` 方式**：<br>

- **即时创建**：可以在容器运行时通过 `-v` 参数创建数据卷并挂载到容器中，而无需提前创建数据卷。<br>
- **临时使用**：这种方式创建的数据卷通常是临时性的，适合一次性使用，不需要为数据卷单独命名和管理。<br>
- **持久化存储**：临时创建的数据卷中的数据在容器删除后仍然存在，除非手动删除。
~~~



