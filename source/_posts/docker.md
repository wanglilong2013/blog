---
title: docker
date: 2019-05-06 21:30:10
tags: [docker]
---
# Docker
-------

## 1. Docker 简介
`Docker` 属于 `Linux` 容器的一种封装，提供简单易用的容器使用接口。`Docker` 
将应用程序与该程序的依赖，打包在一个文件里面。运行这个文件，就会生成一个虚拟容器。程序在这个虚拟容器里运行，就像在真实的物理机上运行一样。

## 2. Docker 用途
------------------
1. 提供一次性的环境。比如，本地测试他人的软件、持续集成的时候提供单元测试和构建的环境。

2. 提供弹性的云服务。因为 Docker 容器可以随开随关，很适合动态扩容和缩容。

3. 组建微服务架构。通过多个容器，一台机器可以跑多个服务，因此在本机就可以模拟出微服务架构。


## 3. Dcoker 常用术语解释
----------------

`Docker镜像文件`  Docker 把应用程序及其依赖，打包在 image 文件里面。只有通过这个文件，才能生成 Docker 容器。image 文件可以看作是容器的模板。Docker 根据 image 文件生成容器的实例。同一个 image 文件，可以生成多个同时运行的容器实例。

`虚悬镜像`:镜像列表中，有一个特殊的镜像，这个镜像没有仓库名，没有标签，均为 <none> ：

`中间层镜像`  docker images -a  #显示包括中间层镜像和顶级镜像。这样会看到很多无标签的镜像，与虚悬镜像不同，这些镜像都是其它镜像所依赖的镜像。这些无标签镜像不应该删除，否则会导致上层镜像因为依赖丢失而出错。实际上，这些镜像也没必要删除，因为相同的层只会存一遍，而这些镜像是别的镜像的依赖，因此并不会因为它们被列出来而多存了一份。只要删除那些依赖它们的镜像后，这些依赖的中间层镜像也会被连带删除。

`Docker容器`  Docker容器类似于一个轻量级的沙箱，Docker利用容器来运行和隔离应用。容器是从镜像创建的应用运行实例。可以将其启动、开始、停止、删除，而这些容器都是彼此相互隔离的、互不可见的，可以把容器看做是一个简易版的Linux系统环境（包括root用户权限、进程空间、用户空间和网络空间等）以及运行在其中的应用程序打包而成的盒子。

`数据卷`  数据卷是一个可供容器使用的特殊目录，它将主机操作系统目录直接映射进容器，类似于Linux中的mount操作。

## 4. Docker常用命令
----------------
```
docker images -a //列出当前所有image文件 

docker rmi imageName //删除image文件

docker ps //查看正在运行的容器

docker ps -a //查看所有容器

docker inspect containerName | grep IPA //查看指定容器IP

docker exec -it containerName /bin/bash //以交互方式进入指定容器

docker container start containerName1 containerName2 //启动容器

docker container stop containerName1 containerName2  //停止容器

docker container rm containerName1 containerName2  //删除容器

docker container rm $(docker ps -aq) //删除所有容器
```

## 5. Dockerfile
-------------------
Dockerfile 是用来说明如何自动构建 docker image 的指令集文件，在 Dockerfile 中编写好指令集之后，我们就可以通过 docker build 命令构建镜像，Dockerfile 文件中命令的顺序就是构建过程中执行的顺序。

`FROM`：依赖镜像
`MAINTAINER`：镜像作者信息
`RUN`：在shell或者exec的环境下执行的命令
`ADD`：将主机文件复制到容器中
`CMD`：指定容器启动默认执行的命令
`EXPOSE`：指定容器在运行时监听的端口
`WORKDIR`：指定RUN、CMD与 ENTRYPOINT 命令的工作目录
`VOLUME`：授权访问从容器内到主机上的目录

## 6. Docker 安装(debian 9)
---------------
```
//1. apt源更换为中科大或163,并更新apt包索引（建议中科大）
apt update

//2. 安装依赖包
apt-get install \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg2 \
     software-properties-common

//3. 添加GPG 密钥，并添加 docker-ce 软件源
curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/debian/gpg | apt-key add -

add-apt-repository "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/debian \
$(lsb_release -cs) stable"    

apt update

//4.安装docker-ce
apt install docker-ce

//5. 本地用户需要把当前用户加入到docker用户组
gpasswd -a ${USER} docker

//6. docker 镜像加速(由于某些原因我们在拉取镜像的时候可能会失败，所有需要更换镜像)
//  /etc/docker/daemon.json
{
    "registry-mirrors":[
        "http://hub-mirror.c.163.com",
        "https://registry.docker-cn.com",
        "https://docker.mirrors.ustc.edu.cn"
    ]
}

//7.重启docker服务
systemctl restart docker

```

## 7. compose

Compose 是一个定义和运行多容器 Docker应用程序的工具, 通过YAML文件来来配置应用程序的服务, 通过 docker-compose up 来创建并启动所有服务。

### 7.1 compose 安装
```
//下载最新版的docker-compose文件 
curl -L "https://github.com/docker/compose/releases/download/1.23.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

//添加可执行权限
chmod +x /usr/local/bin/docker-compose

//测试
docker-compose --version
docker-compose version 1.8.0, build unknown

```

### 7.2 compose 运行步骤
--------------------------
1. 使用Dockerfile 定义应用程序所依赖的环境(也可以使用已有镜像文件)
2. 用docker-compose.yml定义组成程序的服务，使他们可以在隔离环境中一起运行
3. 运行`docker-compose up`命令启动整个应用程序

### 7.3 compose 常用命令
----------------

```
docker-compose up  //根据docker-composer.yml中编排的服务创建并启动容器

docker-compose stop server1Name server2Name  //停止服务

docker-compose start server1Name server2Name  //启动服务

docker-compose restart server1Name server2Name  //重启服务
```

## 8. 快速开始
----------------------

1) 切换至项目(`docker`)根目录

2) 首次运行时需要准备docker-compose.yml 配置文件

```
//准备配置文件,可以根据自己实际情况修改配置项(SOURCE_DIR 、 PHP_LOG_PATH)
cp .env.example .env
```

3) 启动服务(nginx、php、mysql、redis)

```
//首次运行
docker-compose up

//非首次运行
docker-compose start server1Name server2Name
```

4) 浏览器中输入localhost:8000(virtualbox端口转发)回车

## 9.如何使用compose运行一个新的PHP项目
-----------------------------
1）准备nginx配置文件(example.conf)，放在 docker/conf/conf.d目录下,在这之前你需要确认你的项目目录已经挂载到nginx容器和php容器内
```
server {
    listen 80;
    server_name xx.xx.com;
    access_log /var/log/nginx/xx.access.log; 
    error_log /var/log/nginx/xx.error.log; 
    root /var/www/html/xx/public;

    location / {
        index index.php index.html;
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        fastcgi_pass   php56:9000;
        fastcgi_index  index.php;
        include        fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}

```
2) 重启nginx服务(重启服务之前可以先进入nginx容器内检测是否有语法错误 nginx -t)
```
docker-compose restart nginx
```
3) 如果virtualbox端口转发配置的是80：80 需要在宿主机内添加host映射 127.0.0.1  xx.xx.com

4) 在浏览器中输入xx.xx.com:8000(这里的8000端口是我virtualbox 对应的80端口转发)回车OK
