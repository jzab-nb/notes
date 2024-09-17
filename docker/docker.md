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
 
// docker run: 创建并运行容器，-d表示在后台运行
// --name 创建的容器的名字
// -p 宿主机端口:容器内端口
// -e KEY=VALUE: 设置环境变量，不同的镜像需要的变量不一样
// mysql 镜像的名字，一般有两部分组成 [repository]:[tag] 镜像名称:镜像版本，不指定版本则默认latest最新版
```

用docker安装应用时，Docker会自动搜索并下载**镜像**(image)，包含应用本身，运行所需的环境、配置、系统环境库

docker会在运行镜像时创建一个隔离的环境，称之为**容器**（continaer）

**镜像仓库**: 存储和管理镜像的地方，docker官方维护了一个公共的镜像仓库 Docker Hub

## docker命令

![image-20240917142942855](docker.assets/image-20240917142942855.png)

> 下载镜像

docker pull

> 查看镜像

docker images

> 删除镜像

docker rmi

> 创建镜像

docker build

> 打包镜像到本地

docker save

> 安装本地镜像

docker load

> 上传镜像

docker push

> 查看镜像，默认只显示开启的，使用-a显示所有的

docker ps [-a]

> 创建并运行容器

docker run

> 停止运行容器

docker stop

> 启动容器

docker start

> 删除容器 使用-f强制删除

docker rm [-f]

> 查看日志，使用-f实时监控日志

docker logs [-f] 容器名

> 在容器内执行指令 使用-it同时指令名部分传入bash采用可交互的命令行

docker exec [-it] 容器名 指令名

