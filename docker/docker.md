## 快速入门

首先参考官方文档安装docker: https://docs.docker.com/engine/install/ubuntu/

然后查看是否安装成功

```shell
docker ps
docker images
docker -v
```

可以在阿里云配置: 容器镜像服务-镜像工具-镜像加速器

尝试安装mysql

```shell
docker run -d \
 --name mysql \
 -p 3307:3307 \
 -e TZ=Asia/Shanghai \
 -e MYSQL_ROOT_PASSWORD=123 \
 mysql
```

用docker安装应用时，Docker会自动搜索并下载**镜像**(image)，包含应用本身，运行所需的环境、配置、系统环境库

docker会在运行镜像时创建一个隔离的环境，称之为**容器**（continaer）

**镜像仓库**: 存储和管理镜像的地方，docker官方维护了一个公共的镜像仓库 Docker Hub