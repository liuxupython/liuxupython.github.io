# 容器的常用指令



**创建容器（仅创建，不运行）**

```bash
docker create <image>
```

- 等价于 `docker container create <iamge>`

<br>

**启动容器**

```bash
docker start <container>
```

- 等价于 `docker container start <container>`

<br>

**创建并运行容器**

```bash
docker run <image>
```

- 等价于  `docker container run <image>`
- 如果镜像不存在，则先下载镜像再运行容器

<br>

**查看容器列表**

```
docker ps
```

- 等价于 `docker container ls`
- 该命令只列出正在运行的容器，如果想要列出所有容器，加上参数 `-a`

<br>

**停止容器**

```bash
docker stop <container>
```

<br>

**重启容器**

```bash
docker restart <container>
```

<br>

**删除容器**

```bash
docker rm <container>
```

- 等价于 `docker container rm`
- 删除容器，可以通过 容器id 或者 容器名 删除
- 如果要删除正在运行的容器，需要强制删除，加上参数 `-f`

<br>

**进入容器**

```bash
docker exec -it <container> bash
```

- 以交互模式进入容器

<br>



**查看容器日志**

```bash
docker logs <container>
```

<br>

