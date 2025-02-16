# Docker compose

Docker Compose 是一个定义和运行多容器应用程序的工具。Compose 简化了整个应用程序的控制，让你能够轻松地在一个简单易懂的 YAML 配置文件中管理服务、网络和卷。只需使用一个命令即可从配置文件中创建和启动所有服务。

Compose 适用于所有环境；生产、准备、开发、测试以及 CI 工作流。它还具有用于管理应用程序整个生命周期的命令：

- 启动、停止和重建服务
- 查看正在运行的服务的状态
- 流式输出正在运行的服务的日志
- 在服务上运行一次性命令

**Compose 的主要优点**，简化容器化应用程序的开发、部署和管理：

- 简化控制
- 高效协作
- 快速应用程序开发
- 跨环境的可移植性
- 广泛的社区和支持

**Docker Compose 的常见使用场景**

- 开发环境：在开发软件时，在隔离环境中运行应用程序并与之交互的能力至关重要。Compose 命令行工具可用于创建环境并与之交互。
- 自动化测试环境：任何持续部署或持续集成流程的一个重要部分是自动化测试套件。
- 单主机部署：Compose 传统上一直专注于开发和测试工作流程，但随着每个版本的发布，我们都在更多面向生产的功能方面取得进展。

<br>



# docker compose 前戏

在详细介绍 docker compose 之前，我们来聊聊为什么需要它，在什么场景下使用它，使用它帮我们解决了什么痛点。

需求：使用 docker 部署一个 flask web 服务，flask 服务需要一个配套的 redis 服务。

**app.py** 简单的web项目，访问一次就会在 redis 中增加一次浏览次数。

~~~python
import time

import redis
from flask import Flask


app = Flask(__name__)
# 如果这里使用的是 redis 的ip则更麻烦
cache = redis.Redis(host='redis', port=6379)


def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)


@app.route('/')
def hello():
    count = get_hit_count()
    return f'Hello World! I have been seen {count} times.\n'
~~~

对于这个项目，使用 docker 来部署需要拆分为两个容器，一个是web容器，一个是 redis 容器。其中，web 容器依赖 redis 容器，且在 web 容器中需要通过字符串 `redis` 这个名字找到 redis 服务的，而不是通过写死的 IP 地址。

基于这个情况，我们需要分步骤依次实现。

**第一步**：创建一个自定义网络，网络类型为 `bridge`

~~~bash
docker network create flask-web-network
~~~



**第二步**：创建一个 redis 容器，容器名为 `redis`，并挂在 `flask-web-network`网络上

~~~bash
docker run -d -P --name redis --network flask-web-network redis
~~~

<br>

**第三步**：准备 web 容器的 Dockerfile，用来构建 web 容器需要的镜像

~~~dockerfile
FROM python:3.10-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt -i https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple
EXPOSE 5000
COPY . .
CMD ["flask", "run", "--debug"]
~~~

其中项目的依赖文件 **requirements.txt**

~~~tex
flask
redis
~~~

<br>

**第四步**：构建 web 镜像

~~~bash
docker build -t flask-app:v1 -f Dockerfile --no-cache .
~~~

<br>

**第五步**：运行 web 容器，并挂在 `flask-web-network` 网络上

~~~bash
docker run -d --name web -p 5000:5000 --network flask-web-network flask-app:v1
~~~

<br>

**第六步**：访问项目

~~~bash
curl http://127.0.0.1:5000/
~~~

<br>



总结：这个需求很简单，思路也很简单，但是实现起来很繁琐。需要手动运行两个容器，并且容器之间有依赖关系和关联关系。

<br>



# docker compose 快速上手

>本节的主要目的是让大家快速上手 docker compose，具体细节不用在意，后面会详细介绍。我们先从整体上认识 docker compose 的基本使用流程。

基于上节 flask-app 项目，我们使用 compose 的方式。第一步先准备一个 compose 文件。

**compose.yaml**

~~~yaml
services:
  redis:
    image: redis
  web:
    build: .
    ports:
      - "5000:5000"
~~~

**Compose 构建镜像**

~~~bash
docker compose build --no-cache
~~~

**Compose 运行服务**

~~~bash
docker compose up -d
~~~

**查看运行的 compose 项目**

~~~bash
docker compose ls
~~~

查看 compose 项目包含的容器

~~~bash
docker compose ps
~~~

启动、停止、清除项目

~~~bash
docker compose up
docker compose stop [service]
docker compose start [service]
docker compose restart [service]
docker compose down
~~~

<br>



# docker compose 常用命令行指令

| 命令    | 描述                                              |
| ------- | ------------------------------------------------- |
| build   | 构建或重新构建服务                                |
| cp      | 在服务容器和本地文件系统之间复制文件/文件夹       |
| down    | 停止并删除容器、网络                              |
| exec    | 在运行中的容器中执行命令                          |
| images  | 列出由创建的容器使用的镜像                        |
| kill    | 强制停止服务容器                                  |
| logs    | 查看容器的输出                                    |
| ls      | 列出正在运行的 Compose 项目                       |
| ps      | 列出容器                                          |
| restart | 重新启动服务容器                                  |
| rm      | 移除已停止的服务容器                              |
| run     | 在一个服务上运行一次性命令                        |
| scale   | 扩展服务                                          |
| start   | 启动服务                                          |
| stats   | 显示容器资源使用统计数据的实时流                  |
| stop    | 停止服务                                          |
| top     | 显示正在运行的进程                                |
| up      | 创建并启动容器                                    |
| version | 显示 Docker Compose 版本信息                      |
| watch   | 监视服务构建上下文，当文件更新时重新构建/刷新容器 |

<br>



# docker compose 的指令变更

Compose是一个容器编排工具。其实，Fig 项目在开发者面前第一次提出了“容器编排”（Container Orchestration）的概念。 后来，Fig 项目被收购后Docker公司并改名为 Compose。

Docker Compose 的第一版于 2014 年发布。它用 Python 编写，并使用 `docker-compose` 命令调用。

Docker Compose 的第二版于 2020 年发布，用 Go 编写，并使用 `docker compose` 命令调用。

![](https://docs.docker.net.cn/compose/images/v1-versus-v2.png)



对于初学者来说，如果你使用的是最新版本的 docker，可以直接使用 `docker compose` 这个命令来使用 compose 工具；对于老版本的 docker 用户，因为 compose 没有集成到 docker 中，你需要单独安装 docker-compose 工具，然后使用 `docker-compose` 命令来使用 compose 工具。

在 linux 上单独安装 docker-compose 

~~~bash
curl -SL https://github.com/docker/compose/releases/download/v2.29.2/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
~~~

对二进制文件应用可执行权限

~~~bash
chmod +x /usr/local/bin/docker-compose
~~~

测试

~~~bash
docker-compose
~~~

>如果使用的是 docker desktop，则无需安装直接使用 `docker-compose`。另外，`docker compose` 和 `docker-compose` 的命令和用法完全一致（比如：`docker compose up -d` 等价于 `docker-compose up -d`），使用其中一种即可。 

**注意：**

- 如果使用的是 `docker-compose  `命令，一般配套使用 `docker-compose.yaml `文件。
- 如果使用的是 `docker compose` 命令，一般配套使用 `compose.yaml `文件，但是它也支持使用 `docker-compose.yaml` 文件。如果同时存在，则优先使用 `compose.yaml` 文件。

<br>



# compose 文件的结构

Compose 规范是 Compose 文件格式的最新版本，也是推荐版本。它可以帮助你定义一个 Compose 文件，用于配置 Docker 应用程序的服务、网络、卷等。

Compose 文件顶层有4个常用配置项：`name`、`services`、`networks`、`volumes`

**name**

- 顶层 `name` 属性由 Compose 规范定义，作为项目名称。

**services**

- Compose 文件必须声明一个 `services` 顶级元素作为映射，其键是服务名称的字符串表示形式，其值是服务定义。每个服务定义运行时约束和要求以运行其容器。
- 可以有多个服务，一个服务就是一个容器。

**networks**

- 顶级 `networks` 使你能够配置可在多个服务之间相互通信的网络。默认情况下，Compose 为您的应用程序设置单个网络。每个服务的容器都加入默认网络，并且可以与该网络上的其他容器通信，并且可以通过服务的名称发现。
- 要跨多个服务使用网络，必须通过在 `services` 顶级元素中使用 [networks](https://docs.docker.net.cn/compose/compose-file/05-services/) 属性来显式授予每个服务访问权限。

**volumes**

- 顶级 `volumes` 使你能够配置可在多个服务之间重复使用的命名卷。要在多个服务之间使用卷，必须使用 `services` 顶级元素内的 [volumes](https://docs.docker.net.cn/compose/compose-file/05-services/#volumes) 属性显式地授予每个服务访问权限。



**compose.yaml**

~~~yaml
name: myapp

services:
  redis:
    image: redis
    networks:
      - my-network
  web:
    build: .
    ports:
      - "5000:5000"
    networks:
      - my-network
    volumes:
      - my-data:/etc/data

networks:
  my-network:

volumes:
  my-data:
~~~

<br>



# compose 构建镜像

Compose 文件中一个服务是一个容器，容器的创建肯定来自镜像。可以是已有的公共镜像，可以是已有的本地镜像，也可以是根据 `Dockerfile` 构建的镜像。

**Compose文件中使用 `build` 根据 dockerfile 文件构建镜像**。

~~~yaml
services:
  redis:
    image: redis

  web:
    build: 
      context: .
      dockerfile: webapp.Dockerfile
    
    ports:
      - "5000:5000"
~~~

其中，可以这样理解：

- 如果 dockerfile文件的名字是 `Dockerfile`并且和Compose文件同级。则简单配置为
  ~~~yaml
  services:
    web:
      build: .
  ~~~
- 如果 dockerfile名字是 `webapp.Dockerfile`，则使用 `dockerfile`配置，路径使用 `context`配置
  ~~~yaml
  services:
    web:
      build:
        context: .
        dockerfile: webapp.Dockerfile
  ~~~



**Compose 文件中使用 `image` 根据已有镜像构建容器**。可以有如下配置形式。

~~~yaml
services:
   redis1:
     image: redis
   redis2:
     image: redis:5
   redis3: 
     image: redis@sha256:0ed5d592...398bc4b991aac7
   redis4:
     image: library/redis
   redis5: 
     image: docker.io/library/redis
   redis6:
    image: my_private.registry:5000/redis
~~~

<br>



# 设置项目名称和容器名称 

在 Compose 中，默认项目名称就是项目所在文件夹的名字。但是我们也可以自定义项目名。

**方式1：在 Compose 文件中使用配置 `name`**

~~~yaml
name: xxx

services:
  redis:
    image: redis
  web:
    build: . 
    ports:
      - "5000:5000"
~~~

<br>

**方式2：使用环境变量 `COMPOSE_PROJECT_NAME`**

- 在项目路径下新建 `.env` 文件

~~~bash
COMPOSE_PROJECT_NAME=xyz
~~~

> compose 中还有很多内置的环境变量，请查看[官方文档](https://docs.docker.com/compose/environment-variables/envvars/)。

<br>

**方式3：使用参数 `-p`**

~~~bash
docker compose -p aaa up -d
~~~



**定义服务名**

- 使用 `container_name` 定义容器名字。

~~~yaml
services:
  redis:
    image: redis
    container_name: redis
  web:
    container_name: web
    build: . 
    ports:
      - "5000:5000"
~~~



**定义构建的镜像名**

- 使用 `image` 定义被构建的镜像名字

~~~yaml
services:
  redis:
    image: redis
    container_name: redis
  web:
    container_name: web
    image: my-web
    build: . 
    ports:
      - "5000:5000"
~~~

<br>

 

#  映射端口和环境变量

Compose 文件中使用 `ports` 做端口映射。有两种配置方式：简单配置和复杂配置。

**端口映射简单配置**

~~~yaml
ports:
  - "5000"
  - "3000-3005"
  - "8000:8000"
  - "9090-9091:8080-8081"
  - "49100:22"
  - "8000-9000:80"
  - "127.0.0.1:8001:8001"
  - "127.0.0.1:5000-5010:5000-5010"
  - "6060:6060/udp"
~~~

**端口映射复杂配置**

~~~yaml
services:
  redis:
    image: redis
    container_name: my-redis
  web:
    container_name: my-web-app
    image: my-web
    build: .
    ports:
      - name: web
        target: 5000
        host_ip: 127.0.0.1
        published: 8000
        protocol: tcp
~~~



**使用 `environment` 给容器配置环境变量**

方式1：Map 形式

~~~yaml
environment:
  RACK_ENV: development
  SHOW: "true"
  USER_INPUT:
~~~

方式2：Array 形式

~~~yaml
environment:
  - RACK_ENV=development
  - SHOW=true
  - USER_INPUT
~~~



使用 `env_file` 以文件的形式给容器配置环境变量

~~~yaml
env_file: .env
~~~

or

~~~yaml
env_file:
  - ./a.env
  - ./b.env
~~~

>注意：如果在两个文件中同名了环境变量，则使用最后的一个文件中的值。

<br>

也可以在 yaml 文件中使用变量的方式获取环境变量的值。如果 `.env` 文件中不存在变量 `DEBUG`, Compose会给出警告提示。

~~~yaml
web:
  environment:
    - DEBUG=${DEBUG}
~~~

<br>



# 控制服务的启动顺序

>为什么需要控制服务的启动顺序？
>
>一个很好的例子就是需要访问数据库的应用程序。如果这两个服务都使用 docker compose up 启动，则可能会失败，因为应用程序服务可能在数据库服务之前启动，并且找不到能够处理其 SQL 语句的数据库。

在 `services` 下面有两个指令可以控制服务的启动顺序：`depends_on` 和 `links`

**depends_on**：在启动服务时先启动 `web`，并且在关闭服务时先关闭 `web`服务。

~~~yaml
services:
  web:
    build: .
    depends_on:
      - redis
  redis:
    image: redis
~~~

上面的例子是 `depends_on` 的简单配置版本，它还可以和 `condition`、`restart` 等配合使用。

~~~yaml
ervices:
  web:
    build: .
    depends_on:
      mysql:
        condition: service_healthy
      redis:
        condition: service_started
        restart: true
  redis:
    image: redis
  mysql:
    image: mysql
    environment:
      - MYSQL_ROOT_PASSWORD="12345"
    healthcheck:
      test: ["CMD", "mysqladmin" ,"ping", "-h", "localhost"]
      interval: 10s
      retries: 5
      start_period: 30s
      timeout: 10s
~~~

- `condition` 配置被依赖的 `redis` 和 `mysql` 服务 到什么状态时再启动 `web` 服务。它有三个值可以选择
  - `service_started` 等价于 `depends_on` 的简单配置
  - `service_healthy` 达到被认为是[健康](https://docs.docker.com/reference/compose-file/services/#healthcheck)时的状态
  - `service_completed_successfully`

- `restart` 设置为 `true` 时则表示 `redis` 服务重启会自动重启 `web` 服务。



**links**：可以指定服务名称和别名，这样的好处是可以通过服务名或者别名访问被链接服务。

`links` 表达了服务之间的隐式依赖关系，和 `depends_on` 一样可以控制服务启动的顺序。

~~~yaml
web:
  links:
    - db
    - db:database
    - redis
~~~

<br>



# compose 之 networks

默认情况下，Compose 会为根据项目名称自动创建一个网络。服务的每个容器都加入默认网络，并且可以被该网络上的其他容器访问，并且可以通过服务名称发现。

**创建自定义网络**

~~~yaml
services:
  redis:
    image: redis
    container_name: my-redis
    networks:
      - backend
  web:
    restart: always
    container_name: my-web-app
    image: y-web
    build: .
    depends_on:
      redis:
        condition: service_started
        restart: true
    ports:
     - 5000:5000
    networks:
     - backend

networks:
  backend:
    name: xxxxxx
    driver: bridge
    external: true
~~~

其中：

- 参数 `name` 表示自定义网络的名字
- 参数 `driver`  表示网络的类型，默认是 `bridge`
- 参数 `external` 表示是否使用已经存在的网络。

如果通过 Compse 创建网络，那运行项目时会自动按照配置创建网络，删除项目时会自动删除创建的网络。如果使用的是外部网络，compose 运行项目时不会再创建网络，删除项目时也不会删除网络。

<br>



# compose 之 volumes

Compose 文件中配置 `volumes` 可以是要新建的数据卷，也可以是一个存在的数据卷。指定 `volumes` 后，数据会保存在宿主机上，即使容器销毁掉再创建的容器中也会找到以前的数据。

~~~yaml
services:
  web:
    build: .

  mysql:
    image: mysql
    environment:
      - MYSQL_ROOT_PASSWORD=12345
    volumes:
      - db-data:/var/lib/mysql

volumes:
   db-data:
      name: "my-app-data"
      external: true
~~~



<br>

# compose 之 watch

`docker compose watch` 是 Docker Compose 的一个实验性功能，用于监视文件系统的变化，并在检测到更改时自动重启服务或执行其他操作。这个功能特别适合开发环境，因为它可以减少手动重启服务的需求，提升开发效率。

### 用途

- **自动重启服务**：当代码或配置文件发生变化时，自动重启相关服务。
- **提高开发效率**：减少手动操作，专注于代码编写。
- **实时反馈**：立即看到代码更改的效果，加快调试和测试。



~~~yaml
services:
  redis:
    image: redis
  web:
    build: .
    ports:
      - "5000:5000"
    develop:
      watch:
        - action: sync
          path: .
          target: /code
~~~

启动服务 

```bash
docker compose up --watch
```

因为 `action` 配置的是 `sync`， 配置的路径下面的代码变动都会同步到容器中对应的位置，此时不需要手动重启服务，docker compose 会自动帮我们更新在运行的容器。



<br>

# compose 之 include

如果有多个 Compose 文件，单独管理控制，此时可以使用 `include` 管理。

使用 `include` 的好处：

- 您想重用其他 Compose 文件。 
- 您需要将应用程序模型的各个部分分解为单独的 Compose 文件，以便可以单独管理它们或与他人共享。
-  团队需要将 Compose 文件保持在合理的复杂度，以适应其在较大部署中为其自己的子域声明的有限资源量。

**引用基础服务的 Compose 文件**

- 定义基础设施服务的文件 infra.yaml

~~~yaml
services:
  mysql:
    image: mysql
    environment:
      - MYSQL_ROOT_PASSWORD=12345
    ports:
      - 3306:3306    

  redis:
    image: redis
~~~

- 项目的 Compose 文件引入 infra.yaml

~~~yaml
include:
  - infra.yaml

services:
  web:
    build: .
    depends_on:
      - redis
      - mysql
~~~



**覆盖引用的 Compose 文件中的服务**

**方式1**：使用 `compose.override.yaml`

~~~yaml
services:
  redis:
    ports:
      - 6379:6379
    environment:
      - NAME=liuxu
~~~

**方式2**：在 `compose.yaml` 中include 时指定覆盖文件

my_override.yaml

~~~yaml
services:
  redis:
    ports:
      - 6379:6379
    environment:
      - NAME=liuxu
~~~

compose.yaml

~~~yaml
  - path:
      - infra.yaml
      - my_override.yaml

services:
  web:
    build: .
    depends_on:
      - redis
      - mysql
~~~


