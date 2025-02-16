# 端口映射

Docker 端口映射是一种常见的 Docker 容器网络配置，用于将容器内部的端口映射到宿主机上的端口，从而可以通过宿主机的端口访问容器内部的服务。

在 Docker 中，端口映射可以通过 `-p` 或 `--publish` 标志来实现。以下是 Docker 端口映射的基本用法：

~~~bash
docker run -d -p [host_port]:[container_port] [image_name]
~~~

- `-d`: 在后台运行容器。
- `-p [host_port]:[container_port]`: 这里 `[host_port]` 是宿主机上的端口，`[container_port]` 是容器内部的端口。通过这种映射，宿主机的 `[host_port]` 端口将会被映射到容器内部的 `[container_port]` 端口上。
- `[image_name]`: 要运行的 Docker 镜像名称。



例如，如果要将容器内的 80 端口映射到宿主机的 8080 端口，可以这样运行容器：

~~~bash
docker run -d -p 8080:80 nginx
~~~



这将会启动一个 nginx 容器，并将容器内的 80 端口映射到宿主机的 8080 端口上。现在，可以通过访问 `http://localhost:8080` 来访问 NGINX 服务。

~~~alert type=important
宿主机端口和容器端口可以不一样，但是宿主机端口必须是未被占用的端口。
~~~



