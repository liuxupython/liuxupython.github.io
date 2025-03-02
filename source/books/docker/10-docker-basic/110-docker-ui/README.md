# Docker可视化工具



**Portainer** 是一款 Docker 可视化管理工具，可让您轻松构建和管理 Docker、Docker Swarm、Kubernetes 和 Azure ACI 中的容器。 Portainer 将管理容器的复杂性隐藏在易于使用的 UI 后面。

官方地址：https://www.portainer.io/



**安装使用**

- 详看文档：https://docs.portainer.io/start/install-ce/server
- linux安装 portainer 

~~~bash
docker run -d -p 9000:9000 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:2.21.0
~~~

