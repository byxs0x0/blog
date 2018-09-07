---
title: Docker从入门到实践笔记
date: 2018-08-02 10:52:37
categories: Notes
tags:
description:
---

可以"粗糙的"理解轻量级的虚拟机，但不是虚拟机。
>Docker	和传统虚拟化方式的不同之处。传统虚拟机技术是虚拟出一套硬件 后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程；而容器内的应用进程 直接运行于宿主的内核，容器内没有自己的内核，而且也没有进行硬件虚拟。因此容器要比 传统虚拟机更为轻便。
>Dokcer引擎容器仅包含应用程序及其依赖项。它在宿主操作系统上的用户空间中作为一个独立的进程运行，与其他容器共享内核。因此，它具有VM的资源隔离和优点，但更便于携带和高效。
<!-- more -->

# Docker基本概念
那么Docker和在宿主机上面直接运行一个程序有什么区别呢？
> docker 容器使用 cgroup 名字空间实现了 CPU, 内存, 网络, 文件系统等资源隔离。了解 chroot 的同学可以认为 docker 就是一个更优雅的 chroot。

[Docker 容器概念](http://jm.taobao.org/2016/05/12/introduction-to-docker/)

## Docker Image(镜像)

Dokcer将应用程序极其依赖，打包在images文件中。
对于 Linux 而言，内核启动后，会挂载 root 文件系统为其提供用户空间支持。而 Docker 镜像（Image），就相当于是一个 root 文件系统。比如官方镜像 ubuntu:16.04 就包含了完整的一套 Ubuntu 16.04 最小系统的 root 文件系统。 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。

## Docker container(容器)	

容器和镜像的关系就像类与实例一样。
容器的实质是进程，但与直接在宿主机的进程不同。容器进程属于自己的独立的运行空间。有自己的ROOT文件系统，自己的网络配置，自己的进程空间等等。
每一个容器运行时，会将镜像当作基础层，在其上建立一个容器存储层来作为运行时的读写。
容器存储层的生命周期和容器一样，容器消亡时，容器存储也随之消亡。因此任何在容器存储层的信息都会随着容器删除而消失。
所以，所有的文件写入操作，都应该使用 数据卷 或者 绑定宿主目录。这样会在这些位置直接对宿主(或网络存储)发生读写，其性能和稳定性更高。

数据卷的生存周期独立于容器，容器消亡，数据卷不会消亡。

## Docker Registry(镜像仓库)

一个Docker Registry可以包含多个仓库，一个仓库可以包含多个标签，一个标签对应一个镜像。
<仓库名>:<标签>来执行具体是哪个镜像，如果不给出标签，将以latest作为默认标签。
仓库名常以两段式路径形式出现，比如jwilder/nginx-proxy，前者是多用户环境下的用户名，后者是对应的软件名。
Docker Hub 官方仓库
由于访问官网比较慢，所以有加速器(阿里云加速器,DaoCloud加速器)
		

## Docker Deamon
守护程序--客户端发送Docker命令给守护进程，守护进程来操作容器和镜像
		



# 基本信息
```
docker version 
docker info
docker system df # 查看镜像、容器、数据卷所占的空间
```


# 镜像操作
```
# 镜像信息
docker images     # 查看镜像列表(docker image ls)
docker history    # 镜像内的历史记录

# 获取镜像
docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]  # 获取一个新的镜像
	docker pull medicean/vulapps:s_spring_2   # 如果没给地址，默认从Docker Hub中拉取

# 删除镜像
docker images rm [imageName]    # 删除镜像,参数可以是镜像短ID、镜像长ID、镜像名或者镜像摘要
docker rmi [image_id] # 删除images

# 制作镜像
docker commit -m 'fun' [continar_id] [image_name] # 保存容器的内容，保存后会产生新的IMAGE

# 镜像创建
docker commit \
	--author "" \
	--message "" \
	webserver \
	nginx:v2

慎用commit！！！因为当你只想修改一个文件内容时，由于命令的执行，还有很多文件被改动添加。如果是安装软件包、编译构建，那会有大量的无关内容被添加进来，如果不小心清理，将会导致镜像极为臃肿。
同时，docker commit意味着黑箱操作，除了生成镜像的人知道做了哪些操作，其他人不知道。
因为分层存储，在多次commit时，每一次修改都会让镜像更加臃肿，因为上一层的东西并不会丢失。
```

**注意：**
删除镜像的行为有两种，一种是Untagged和Deleted
因为删除镜像会先删除标签，当该镜像无标签时，才会在进行Deleted操作
所以有些时候删除镜像的时候仅仅只删除了某个标签
同时如果某个镜像依赖于当前镜像，同样也不会进行删除操作。




## 虚悬镜像 
名字与标签为none。同名称的镜像发生了更新，那么pull会将名字转移到新下载的镜像上。
`docker image prune` # 删除虚悬镜像

## 中间层镜像  
在使用了一段时间之后，可能会看见中间层镜像的出现。他同样也是无标签的镜像，他是为了加速镜像构建、重复利用资源。如果删除，可能会导致依赖中间层的镜像无效。
`docker images -a` # 显示中间层在内的所有镜像

## Docker 镜像分层
分层原因是，如果两个不同的IMAGE 前几层都是相同的，那么可以共同存储


## Dockerfile制作镜像
Dockerfile就是一个脚本，其中包含一条条的指令(Instruction)，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建，简单的理解就是类型批处理文件。

### 制作步骤
```
1. 创建文件夹，在文件夹创建文件名为Dockerfile第一个大写
mkdir ubuntuDocker
touch Dockerfile
vim Dockerfile

2. 编写Dockerfile文件
FROM alpine:latest
MAINTAINER xbf 
CMD echo "Hello Docker"

3. 构建镜像
docker build -t nginx:v3 .
docker build [选项] <上下文路径/URL/->
```
**注意：**
`docker build`是在服务器执行的，而非客户端。但是服务端如何获取本地文件呢？这就引入了上下文概念。 **当构建的时候，用户会指定构建镜像的上下文的路径**，`docker build`命令得知这个路径后，会将路径下的所有内容打包，然后上传给Docker引擎。这样Docker引擎收到这个上下文包后，展开就会获得构建镜像所需要的一切文件。
例如`COPY ./file.txt /webapp/`，这并不是要复制执行`docker build`命令所在目录下的`file.txt`，也不是复制`Dockerfile`目录下的`file.txt`，而是复制上下文(context)目录下的`file.txt`。
所以很多初学者`COPY /opt/xxx /app`不会成功。

一般来说，应该会将`Dockerfile`置于一个空目录下，或者项目根目录下。如果该目录下没有所需文件，那么应该把所需文件复制一份过来。同时可以用`.gitignore`语法写一个`.dockerignore`文件来告诉某些文件不作为上下文传递给Docker引擎。

`.`不是指定`Dockerfile`的目录，而是默认不指定`Dockerfile`会在上下文目录中找到名为`Dockerfile`的文件。
也可以用`-f ../../Dockerfile`

#### docker build 其他用法
##### URL
docker build支持`URL`
以下命令指定了`Git repo`，并且指定默认的`master`分支，构建目录为`/8.14/`，然后Docker就会自己去`git clone`这个项目、并进入到指定目录后开始构建。
```
$ docker build https://github.com/twang2218/gitlab-ce-zh.git#:8.14 
docker build https://github.com/twang2218/gitlab-ce-zh.git\#:8.14 
Sending build context to Docker daemon 2.048 kB
Step 1 : FROM gitlab/gitlab-ce:8.14.0-ce.0
8.14.0-ce.0: Pulling from gitlab/gitlab-ce 
aed15891ba52: Already exists 
773ae8583d14: Already exists
```

##### tar压缩包
`docker build http://server/context.tar.gz`，Docker引擎会下载这个包，并自动解压缩，以其作为上下文，开始构建

##### 从标准输入读取Dockerfile进行构建
将输入的文本文件视为`Dockerfile`
`docker build - < Dockerfile`
`cat Dockerfile | docker build -`



### 命令详解

#### FROM
指定基础镜像
 - 基本的操作系统(`ubuntu centos debian alpine`)
 - 服务类镜像(`nginx tomcat apache mysql`)
 - 语言应用镜像(`python node ruby`)
 - 空白镜像(`scratch`)

#### RUN
执行命令行命令

##### 基本格式
```
# shell格式
RUN	echo	'<h1>Hello,	Docker!</h1>'	>	/usr/share/nginx/html/index.html

# exec格式
RUN	["可执行文件",	"参数1",	"参数2"]
```
如果使用RUN分开执行多条命令，那么一次RUN就会执行创建一层镜像。会增加部署构建的时间，也会创建太多层镜像等等。所以我们使用推荐写法
以下写法将这一个“行为”简化为同一层。

```
FROM debian:jessie
RUN buildDeps='gcc libc6-dev make' \
    && apt-get update \
    && apt-get install -y $buildDeps \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-3.2.5.tar.gz" \
    && mkdir -p /usr/src/redis \
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && make -C /usr/src/redis \
    && make -C /usr/src/redis install \
    && rm -rf /var/lib/apt/lists/* \
    && rm redis.tar.gz \
    && rm -r /usr/src/redis \
    && apt-get purge -y --auto-remove $buildDeps

\ 换行
# 注释
&& 分割命令
最后添加了清理工作的命令，删除了为了编译构建所需要的软件，清理了所有下载、展开的文件，并且还清理了apt缓存文件。这是很重要的一步，我们之前说过，镜像是多层存储，每一层的东西并不会在下一层被删除，会一直跟随着镜像。
因此镜像构建时，一定要确保每一层只添加真正需要添加的东西，任何无关的东西都应该清理掉。
很多人初学Docker制作出了很臃肿的镜像的原因之一，就是忘记了每一层构建的最后一定要 清理掉无关文件。
```

#### COPY
复制文件
```
COPY <源路径>... <目标路径> 
COPY ["<源路径1>",... "<目标路径>"]

<源路径>可以是多个，设置可以是通配符，其通配符规则要满足GO
COPY hom* /mydir/
COPY hom?.txt /mydir/

<目标路径>是容器内的绝对路径，也可以是相对于工作目录的相对路径(工作距离需要WORKDIR指令指定)
```

#### ADD
ADD指令和COPY的格式和性质基本一致。但是在COPY基础上增加了一些功能。
它的`<源路径>`可以是一个URL，这种情况下，Docker引擎回去下载，下载后的文件权限自动设置为`600`，如果是压缩包，还需要用`RUN`来解压缩，不如直接用RUN，所以该命令不太实用。

但是如果`<源路径>`是一个`tar`压缩文件的话，压缩格式为`gzip bzip2 xy`的情况下，ADD指令会自动解压缩到`<目标目录>`中。
另外`ADD`指令会令镜像构建缓存失效，从而可能令镜像构建变得比较缓慢。

**所以文件复制用`COPY`，文件需要解压缩用`ADD`。**


#### CMD
CMD指令的格式和RUN格式相似，但是它是用于启动容器的时候，需要制定所运行的程序及参数。例如ubuntu镜像默认的CMD是`/bin/bash`，如果我们直接`docker run -it ubuntu`默认就是进入`bash`。如果我们改变命令为`docker run -it ubuntu uname -r`，我们就用`uname -r`替换了默认的`/bin/bash`，并且输出了内核信息。
```
# shell 格式
CMD <命令>  # 实际为 CMD ["sh","-c",命令]

# exec 格式
CMD	["可执行文件", "参数1", "参数2"...]


在指令上推荐使用exec格式，这类格式在解析时会被解析为JSON数组，因此一定要使用双引号
```
**注意：**
**容器中没有前台、后台的概念，它不是虚拟机**，对于容器而言，其启动程序就是容器应用程序。容器就是为了主进程而存在的，主进程退出了，容器就失去了存在的意义，从而退出，其他辅助进程不是它所关心的。
如果使用`CMD service nginx start`，是希望`upstart`以后台的形式启动`nginx`服务。而这条命令实际上是`CMD ["sh","-c","service nginx start"]`。那么`sh`才是主程序，当`service nginx start`命令执行结束后，`sh`这个主程序也就结束了，自然容器也会退出

正确的做法是直接执行`nginx`可执行文件，并且要求以 **前台**形式运行
`CMD ['nginx','-g','deamon off;']`



#### ENTRYPOINT入口点
ENTRYPOINT的目的和CMD一样，都是在指定容器启动程序参数。ENTRYPOINT在运行时也可以替代，需要通过`--entrypoint`来指定。

当指定ENTRYPOINT后，CMD含义就发生了改变。不再是直接运行其命令，而是将CMD的内容作为参数传给ENTRYPOINT指令，换句话说实际执行时，将变为：`<ENTRYPOINT> "<CMD>"`

##### 场景1
```
# 获取当前容器所用的公网IP
FROM ubuntu:16.04 
RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/* 
CMD [ "curl", "-s", "http://ip.cn" ]

docker run ubuntu -i # 这样运行会报错，因为-i替换了原来的CMD命令

# 改为ENTRYPOINT
FROM ubuntu:16.04 
RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/* 
ENTRYPOINT [ "curl", "-s", "http://ip.cn" ]

docker run ubuntu -i # 这样不会报错，因为<ENTRYPOINT> [CMD]。所以最后是 curl -s http://ip.cn -i
```

##### 场景2
应用运行前的准备工作
例如在运行某一个服务前需要做一些准备工作，可以写一个sh脚本，给ENTRYPOINT去执行。
```
FROM alpine:3.4
...
RUN addgroup -S redis && adduser -S -G redis redis 
...
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 6379
CMD [ "redis-server" ]
```

#### ENV设置环境变量
```
# 定义环境变量
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>

## node.js使用环境变量
ENV	NODE_VERSION 7.2.0

RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.ta
r.xz" \
  && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
  && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
  && grep " node-v$NODE_VERSION-linux-x64.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
  && tar -xJf "node-v$NODE_VERSION-linux-x64.tar.xz" -C /usr/local --strip-components= 1 \
  && rm "node-v$NODE_VERSION-linux-x64.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt \
  && ln -s /usr/local/bin/node	/usr/local/bin/nodejs



# 支持环境变量的指令
ADD COPY ENV EXPOSE LABEL USER WORKDIR VOLUME STOPSIGNAL ONBUTLD
```
**注意**
`DEBIAN_FRONTEND`是一个环境变量，它规定了操作系统应该从哪儿获得用户输入。如果设置为`noninteractive`，代表可以直接运行命令，无需向用户请求输入。(简单的说就是 **非交互式**)。
那么这有什么用呢?
在`docker build`中，当在`Dockerfile`中安装`mysql-server`服务，那么在安装的过程中会让用户输入密码，但是由于`build`是非交互式的过程，所以不会接收任何输入。那么就到卡死在个过程。当我们在这个时候设置`DEBIAN_FRONTEND=noninteractive`,那么会选择默认的选项完成构建。
**但是如果我们在实际中直接这样设置，会导致整个容器的任何交互都会进行无交互式，所以我们需要**`RUN DEBIAN_FRONTEND=noninteractive apt-get install -y mysqli-server`，来对这个命令进行一个规定。


#### VOLUME 定义匿名卷
```
VOLUME ["<路径1>", "<路径2>"...] 
VOLUME <路径> 
```

#### EXPOSE 声明端口
```
EXPOSE <端口1> [<端口2>...]
```
**这仅仅是声明，而不会开放端口**

#### WORKDIR 指定工作目录
`WORKDIR <工作目录路径>`以后各层的当前目录就被改变为指定的目录，如该目录不存在，WORKDIR会帮你建立目录。

**Dockerfile不等同于shell脚本**
```
RUN cd /app
RUN echo "hello" > world.txt

写法是错误的，因为每一个RUN都会创造一层镜像，两个RUN执行的环境根本不同。是两个完全不同的容器。
```

#### USER 指定当前用户
`USER <用户名>`USER指令和WORKDIR相似，都是改变环境状态并影响以后的层。
**切换的用户必须在是已存在的**

如果是以ROOT执行的脚本，在执行期间希望用某个用户执行某个服务，不要用su 和sudo，他们都需要比较麻烦的配置。而且在TTY环境缺失的环境下经常出错。建议使用gosu。
```
# 建立 redis 用户，并使用 gosu 换另一个用户执行命令
RUN groupadd -r redis && useradd -r -g redis redis
# 下载 gosu
RUN wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/1.7/ gosu-amd64" \
    && chmod +x /usr/local/bin/gosu \
    && gosu nobody true
# 设置 CMD，并以另外的用户执行
CMD [ "exec", "gosu", "redis", "redis-server" ]
```


#### HEALTHCHECK 健康检查
...待补充

#### ONBUILD 为他人做嫁衣
...待补充

# 容器操作
```
# 基本信息
docker stats     # 查看docker状态
docker ps        # 查看正在运行的容器
    -a 查看所有容器
docker container ls    # 查看容器信息
	-a 停止的容器也能查看
docker container logs [container ID or NAMES]  # 查看容器输出信息

# 容器基本操作
docker container start/stop/restart # 将一个已经终止的程序启动运行/停止/重启
docker start/stop/restart [ID] # WEB应用容器
docker kill     # 终止容器
docker rm [ID]  # 移除WEB应用容器

# 容器运行
docker run -d -p 8000:8080 medicean/vulapps:s_spring_2     # 运行docker   宿主机:8000  映射到  Docker:8080
	-d 后台运行
	-p 设置端口映射
docker run -it <image_name/continar_id> bash --rm # 进入ubuntu容器的shell
	-i 交互式操作,让容器的标准输入保持打开
	-t Docker分配一个伪终端(pseudo-tty)并绑定到容器的标准输入上
	--rm 退出容器的时候直接删除容器
	-v 挂载一个卷

# 进入容器
docker attach [ID] # 进入Docker中，注意不建议使用它，因为如果用exit命令退出，会导致容器停止
docker exec -it <id/container_name> /bin/bash # 进入正在运行的容器，同时运行bash，使用exit命令退出，不会导致容器停止

# 容器内部操作
docker cp [continar_id]://usr/share  # 把文件拷贝到容器中的指定目录下

# 容器导入导出
docker export [container_id] > file.tar # 导出容器快照到本地文件
cat file.tar | docker import - test/ubuntu:v1.0 # 导入容器快照
docker import http://xxx.xxx 也可以指定URL或者某个目录来导入
```
**注意：**
```
docker run 创建容器操作
	检查本地是否存在指定的镜像，不存在就从公有仓库下载
	利用镜像创建并启动一个容器
	分配一个文件系统，并在只读的镜像层外面挂载一层可读写层 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去 从地址池配置一个	ip	地址给容器
	执行用户指定的应用程序
	执行完毕后容器被终止
```



# Docker数据管理
在Docker内部以及容器之间管理数据，提供独立于容器之外的持久化存储，在容器中管理数据主要有两种方式
 - 数据卷(Volumes)
 - 挂载主机目录(Bind mounts)

## 数据卷
 - 数据卷类似于一个mount
 - 数据卷可以在容器之间共享与重用
 - 对数据卷的修改会立马生效
 - 对数据卷的更新，不会影响镜像
 - 数据卷默认会一直存在，即使容器被删除
	
使用以下参数`--mount -v --volume`挂载，建议使用`-mount`参数

```
docker volume create my-vol # 创建一个数据卷
docker volume ls  # 查看所有数据卷
docker volume rm  # 删除数据卷
docker inspect web # 查看web容器的信息
docker volume prune # 删除无主的数据卷
```

### 实例1
创建一个web容器，并加载一个数据卷到容器的/webapp目录
```
docker run -d -p \ 
-name web \ 
--mount source=my-vol,target=/webapp \
training/webapp \
python app.py
```
	
	
## 挂载主机目录
同样建议用mount，挂载一个目录到容器中，也可以挂载单个文件
```
docker	run	-d	-P	\
--name	web	\
#	-v	/src/webapp:/opt/webapp	\
--mount	type=bind,source=/src/webapp,target=/opt/webapp	\
training/webapp	\
python	app.py


target=/opt/webapp,readonly # 只读
```

# Docker网络功能
Docer允许通过外部访问容器或容器互联的方式来提供网络服务


## 外部访问容器
```
-P # 随机映射一个49000-49900 的端口到内部容器开放的网络端口中
-p # 指定要映射的端口，一个指定端口只能绑定一个容器  -p 参数在一条命令可以用多次 
# ip:hostPoet:containerPort | ip::containerPort | hostPort:containerPort
-p 127.0.0.1:5000:5000 
-p 127.0.0.1::5000 # 会随机分配一个主机端口

docker port nostalgic_morse 5000 # 查看当前端口映射

docker inspect # 获取容器所有变量，Docker可以有一个可变的网络配置
```




# Docker-compose
定义与运行多个Docker容器的应用。通过docker-compose.yml模版文件来定义一组相关联的应用容器为一个项目
利用python编写


服务(Service)
	一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。
项目(Project)
	由一组关联的应用容器组成的一个完整业务单元，在docker-compose.yml文件中定义。
	
一个项目可以由多个服务关联而成，Compose面向项目进行管理


	
# Docker Machine
负责在多种平台上快速安装Docker环境，基于GO语言
	
# Docker Swarm
提供Docker容器集群服务，是Docker官方对容器云生态进行支持的核心方案。
用户可以将多个Docker主机封装为单个大型的虚拟Docker主机，快速打造一套容器云平台。
	
由于swarm mode 已经嵌入到docker核心中，成为了子命令Docker sware。大多数用户使用的是swarm mode

swarm是使用SwarmKit构建的Docker引擎内置(原生)的集群管理和编排工具

节点
	运行Docker的主机可以主动初始化一个Swarm集群或者加入一个已存在的Swarm集群，这样这个运行Docker的主机就成为一个Swarm集群的节点(node)。

节点分为管理节点和工作节点
	管理节点用于Swarm集群的管理，docker swarm命令基本只能在管理节点执行(节点退出集群命令可以在工作节点执行)
	一个Swarm集群可以有多个管理节点，但只有一个管理节点可以成为leader,leader通过raft协议实现。
	
	工作节点是任务执行节点，管理节点将服务(service)下发至工作节点执行。管理节点默认也作为工作节点。

服务和任务
	任务(Task)是Swarm中最小的调度单位
	服务(Services)是指一组任务的集合，服务定义了任务的属性，服务有两种模式：
		replicated service 按照一定规则在各个工作节点上运行指定个数的任务。
		global services 每个工作节点上运行一个任务
		
	






