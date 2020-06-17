# 1.Docker概念

> Docker的思想来自于集装箱，集装箱解决了什么问题？在一艘大船上，可以把货物规整的摆放起来。并且各种各样的货物被集装箱标准化了，集装箱和集装箱之间不会互相影响。
>
> ##  1.1 Docker思想
* 镜像（集装箱）、仓库（超级码头）、容器（运行程序的地方）
* 标准化（运输方式、存储方式、API接口）

![IMG_1050](http://47.94.165.69:3031/upload/aaa/7277ce791b7c6.PNG) 

##  1.2 Docker解决了什么问题
* 运行环境不一致

* 应用之间影响内存

  

# 2. Docker核心技术

## 2.1 查看命令

```
$ docker info

$ docker version 
```

## 2.2 镜像image命令

```
$ docker images 

$ docker pull imgPath

$ docker push imgPath

#创建镜像
$ docker build -t img-name:version .

#删除镜像(务必先停掉该镜像下的所有容器) 
$ docker rmi [options] imgId
```

## 2.3 容器container操作命令
```
#启动一个容器（create-start）
$ docker run -d -p [port] imgId  #映射到指定端口
$ docker run -d -P imgId         #映射到随机端口

#创建一个容器
$ docker create (用法同run，只不过create只创建并不会启动) 

#启动/停止/重启容器 
$ docker start/stop/restart 

#杀掉一个容器
$ docker kill conId 

#删除容器（删除处于停止状态的容器） 
$ docker rm conId

#在运行的容器中执行命令 
$ docker exec -it conId bash

#列出正在运行容器 
$ docker ps [OPTIONS] 

#列出所有容器 
$ docker ps -a 
```
## 2.4 私有仓库docker-registry 
```
# 安装并启动docker registry
$ docker run -d -p 5000:5000 registry

#使用tag 让私有仓库的指向本地镜像
$ docker tag img-name localhost:5000/new-img-name:latest

#push到你的私有仓库
$ docker push localhost:5000/new-img-name

#将镜像从仓库中取出，并查看
$ docker pull localhost:5000/new-img-name
$ docker images
```
## 2.5 docker网络
* Bridge
* Host
* None



# 3. Docker-compose

##  3.1配置文件

 Compose的配置文件是`docker-compose.yml`

```
version: '2.0'

services:
  
  db:
    image: mongo
    volumes:
      - ./db-data/mongodb:/data/db
    ports:
      - "27018:27017"
    networks:
      - app_net

  web:
    build: sb-server-koa
    volumes:
      - ./web-data/upload:/home/sb-webserver/public/upload
    ports:
      - "3031:3030"
    links:
      - db
    networks:
      - app_net

networks: 
  app_net:
    driver: bridge
```

以上配置文件说明：

1. 定义了两个服务分别叫做`db`和`web`
2. . db服务使用dockerhub官方的`mongo image`定义服务镜像，web服务行sb-server-koa下的`Dockerfile`来创建镜
3. `volumes`指定docker容器中的文件存在本机上的路径
4. web容器使用`links`和db容器链接
5. 使用`ports`指定docker容器在本机上的对应端口
6. `networks`定义两个容器之间的网络连接方式为桥接`bridge`

## 3.2 常用命令

启动服务，-d 为detached后台运行模式

```
docker-compose up -d
```

查看运行的实例

```
docker-compose ps
```

查看运行日志

```
docker-compose logs
```

停止服务

```
docker-compose stop
```



# 4. 本地私有仓库 Harbor

## 4.1 安装配置

```
1. 离线下载解压
2. 更改配置/权限
3. ./install.sh
4. docker ps -a # 查看 harbor 服务组件运行情况
```



## 4.2 推送镜像

```
# 1. docker 登陆
docker login hub.chenleilei.com:18080
# silverbulllet/Harbor12345

# 2. 打tag
docker tag silverbulllet/forsimba hub.chenleilei.com:18080/my-works/forsimba

# 3. 推送
docker push hub.chenleilei.com:18080/my-works/forsimba
```

