# 什么是Docker
>代码接触的环境：开发环境，测试环境，生产环境<br>
代码本地测试之后打包成.war包给测试人员，然后再给运维人员部署到生产环境<br>

因为JDK版本造成的环境问题 解决方案就是连环境和代码一起打包发送<br>
代码和环境装进容器，把容器发给测试人员，测试再把容器发给运维人员<br>

### docker容器用来解决软件跨环境迁移问题

---

# docker概念
1. docker是一个开源的应用容器引擎
2. 诞生时间2013年初，基于Go语言实现，dotCloud公司出品(Docker Inc)
3. Docker可以让开发者打包应用以及依赖包到一个“轻量级”“可移植“的容器中，然后发布到任何流行的Linux机器上
4. 容器是完全使用沙箱机制，相互隔离
5. 容器性能开销极低
6. Docker从17.03开始分为CE(社区)和EE(企业)版本

### 小结：Docker是一种容器技术，解决软件跨环境迁移的问题

---

# 安装Docker
```bash
$yum update -y
$yum install -y yum-utils device-mapper-persistent-data lvm2
$yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
$yum install -y docker-ce --allowerasing
Complete!
$docker -v
Docker version 20.10.10, build b485636
```
Docker安装成功

---

# Docker架构
![docker架构图]()

>镜像(Image):Docker镜像(Image),就相当于是一个root文件系统，类似Ubuntu<br>

>容器(Container):镜像和容器的关系就像是类和对象的关系。镜像是静态定义，容器是运行的实体。容器可以被创建，启动，停止，删除，暂停等。<br>

>仓库(Repository):仓库可以看成一个代码控制中心，用来保存镜像。<br>

1. 本地安装的docker是以守护进程运行在后台
2. daemon包含image镜像(操作系统)和container容器<br>
3. 镜像来源于远程的仓库<br>
仓库分为：官方提供的仓库，包含很多镜像(慢) 和 私有仓库(自己的镜像共享，快)
4. docker客户端

---

# 配置Docker镜像加速器
1. 默认情况下，是从Docker hub(https://hub.docker.com/)下载docker镜像
2. 配置国内镜像加速器
   1. USTC：中科大镜像加速器(https://docker.mirrors.ustc.edu.cn)
   2. 阿里云(相对最快)https://2xabk925.mirror.aliyuncs.com
   3. 网易云
   4. 腾讯云

```bash
# 使用阿里云镜像加速器配置docker
$sudo mkdir -p /etc/docker
$sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://2xabk925.mirror.aliyuncs.com"]
}
EOF
$sudo systemctl daemon-reload
$sudo systemctl restart docker

# 可以再检查一下daemon.json文件中有没有内容
$cat /etc/docker/daemon.json
{
  "registry-mirrors": ["https://2xabk925.mirror.aliyuncs.com"]
}
# 成功配置了阿里云的镜像加速器
```

---

# Docker命令

1. Docker服务相关命令(守护进程相关命令)
   1. 启动docker服务    ```systemctl start docker```
   2. 停止docker服务    ```systemctl stop docker```
   3. 重启docker服务    ```systemctl restart docker```
   4. 查看docker服务状态    ```systemctl status docker```
   5. 开机启动docker服务    ```systemctl enable docker```
2. Docker镜像操作命令
   1. 查看镜像  ```docker images```
   2. 搜索镜像  ```docker search [redis]```(hub.docker.com官网查找官方维护的镜像版本)
   3. 拉取镜像  ```docker pull [redis:6.2]```(默认版本latest)
   4. 删除镜像  ```docker rmi [redis:6.2/ImageID]``` 或者删除全部镜像   ```docker rmi `docker images -q` ```
3. Docker容器操作命令
   1. 查看容器  ```docker ps [-a]```(默认查看正在运行的容器，a：历史容器)
   2. 创建容器  ```docker run [-i] [-t]/[-d] [--name XX] [centos:7] [/bin/bash]```<br>*i：容器保持运行<br>t：分配终端创建后立即打开<br>d：后台运行创建容器<br>name：容器名字用=或者空格连接<br>centos:7：镜像+版本号<br>/bin/bash：打开bash窗口*<br>通常情况下-it：创建的是交互式容器，-id：创建的是守护式容器
   3. 进入容器  ```docker exec [-i] [-t] [name] [/bin/bash]```
   4. 启动容器  ```docker stop [name]```
   5. 停止容器  ```docker start [name]```
   6. 删除容器  ```docker rm [name/`docker ps -aq`]```(docker ps -aq：删除所有容器)
   7. 查看容器信息  ```docker inspect [name/ID]```

---

# Docker容器的数据卷

思考：
>1. Docker容器删除后，在容器中产生的数据也会随之销毁
>2. Docker容器和外部机器可以直接交换文件么？
>3. 容器之间想要进行数据交互？

![数据卷图解]()

1. 数据卷概念
   * 数据卷是宿主机中的一个目录或文件
   * 当容器目录和数据卷目录绑定后，对方的修改会立即同步
   * 一个数据卷可以被多个容器同时挂载
   * 一个容器也可以被挂载多个数据卷

2. 数据卷作用
   * 容器数据持久化
   * 外部机器和容器间通信
   * 容器之间数据交换

3. 配置数据卷
   * 创建启动容器时，使用[-v]参数设置数据卷 ```docker run ... -v [宿主机目录/文件]:[容器内目录/文件]```
      >注意事项：
      >1. 目录必须是绝对路径
      >2. 如果目录不存在，会自动创建
      >3. 可以挂在多个数据卷

4. 配置数据卷容器
   1. 多容器进行数据交换的两个实现方式
      * 多个容器挂在同一个数据卷(不方便)
      * 数据卷容器![数据卷容器]()
   2. 创建数据卷容器，使用[-v]参数设置数据卷
   ```bash
   # 创建数据卷c3
   $docker run -it --name=c3 -v /volume centos:7 /bin/bash

   # 创建启动c1和c2容器，使用[--volumes-from]参数 挂载数据卷
   $docker run -it --name=c1 --volumes-from c3 centos:7 /bin/bash
   $docker run -it --name=c1 --volumes-from c3 centos:7 /bin/bash
   ```

   ---
   
# Docker应用部署
> 学习部署MySQL，Tomcat，Nginx，Redis

1. MySQL部署<br>
   * 需求：在Docker容器中部署MySQL，并通过外部MySQL客户端操作MySQL Server。
   * 实现步骤
     1. 搜索MySQL镜像
     2. 拉取M有SQL镜像
     3. 创建容器
     4. 操作容器中的MySQL
   * 问题：
     1. 容器内的网络服务和外部机器不能直接通信
     2. 外部机器和宿主机可以直接通信
     3. 宿主机和容器可以直接通信
     4. 当容器中的网络服务需要被外部机器访问时，可以将容器中提供服务的端口映射到宿主机的端口上。外部机器访问宿主机的该端口，从而间接访问容器的服务。
     5. 这种操作被称为：端口映射
   * 图解
   ![端口映射]()
```bash
# 通过端口映射，将容器内的3306端口映射到宿主机的3307端口上，然后通过外部机器访问宿主机的3307端口
```

```bash
$docker search mysql
$docker pull mysql   # 默认latest
$mkdir ~/mysql
$cd ~/mysql
$docker run -id \    #创建容器后台运行
> -p 3307:3306 \     #端口映射,将容器端口3306映射到宿主机端口3307
> --name c_mysql \   #容器名字
> -v $PWD/conf:/etc/mysql/conf.d \     #挂载conf配置文件目录
> -v $PWD/logs:/logs \                 #挂载logs日志文件目录
> -v $PWD/data:/var/lib/mysql \        #挂载lib数据文件目录
> -e MYSQL_ROOT_PASSWORD=123456 \      #[-e] env环境 初始化root用户的密码
> mysql              #指定镜像，默认最新版本
```
![外部访问并修改db1]()<br>
![数据库容器中查询]()


2. MySQL部署<br>
   * 需求：在Docker容器中部署Tomcat，并通过外部机器访问Tomcat部署的项目。
   * 实现步骤
     1. 搜索Tomcat镜像
     2. 拉取Tomcat镜像
     3. 创建容器
     4. 部署项目
     5. 测试访问

```bash
$docker search tomcat
$docker pull tomcat   # 默认latest
$mkdir ~/tomcat       # 创建tomcat目录
$cd ~/tomcat
$docker run -id --name=c_tomcat \      # 启动一个名字叫c_tomcat的容器
> -p 8080:8080 \                       # 端口映射8080
> -v $PWD:/usr/local/tomcat/webapps \  # 挂载tomcat目录到容器中tomcat的webapps目录中，方便日后部署项目
> tomcat              # 指定镜像，默认最新版本
```
![tomcat_webapp]()<br>


3. Nginx部署<br>
   * 需求：在Docker容器中部署Nginx，并通过外部机器访问Nginx。
   * 实现步骤
     1. 搜索nginx镜像
     2. 拉取nginx镜像
     3. 创建容器
     4. 部署项目
     5. 测试访问

```bash
$docker search nginx
$docker pull nginx   # 默认latest
$mkdir ~/nginx       # 创建nginx目录
$cd ~/nginx
$mkdir conf
$cd conf             # 在~/nginx/conf/下创建nginx.conf文件，粘贴以下内容
$vim nginx.conf
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
$docker run -id --name=c_nginx \       # 启动一个名字叫c_nginx的容器
> -p 80:80 \                           # 端口映射80
> -v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf \  # 将主机目录下的/conf/nginx.conf挂载到容器的/etc/nginx/nginx.conf 挂载配置目录
> -v $PWD/logs:/var/log/nginx \        # 挂载日志目录
> -v $PWD/html:/usr/share/nginx/html \
> nginx             # 指定镜像，默认最新版本
```
![docker_nginx]()<br>

4. Redis部署<br>
   * 需求：在Docker容器中部署Redis，并通过外部机器访问Redis。
   * 实现步骤
     1. 搜索Redis镜像
     2. 拉取Redis镜像
     3. 创建容器
     4. 测试访问

```bash
$docker search redis
$docker pull redis   # 默认latest
$mkdir ~/redis       # 创建redis目录
$cd ~/redis
$docker run -id --name=c_redis -p 6379:6379 redis
```
![docker_redis]()<br>

---

# Dockerfile
>* Docker镜像本质是什么？<br>
>答：分层的文件系统

>* Docker中一个CentOS镜像为什么只有200MB，但是一个CentOS操作系统的iso文件要几个G？<br>
>答：CentOS的iso镜像文件包含bootfs和rootfs。而docker的CentOS镜像复用操作系统的bootfs，只有rootfs和其他镜像层。

>* Docker中一个tomcat镜像为什么有500M，但是tomcat安装包只有70多M？<br>
>答：由于docker中镜像是分层的，tomcat虽然只有70多M，但是需要依赖于父镜像和基础镜像，所以整个对外暴露的tomcat镜像大小500多M

Linux文件系统由bootfs和rootfs两部分组成
* bootfs：包含BootLoader(引导加载程序)和Kernel(内核)
* rootfs：包含的就是典型Linux系统中的/dev,/proc,/bin,/etc等标准目录和文件
* 不同的Linux发行版，bootfs基本一样，而rootfs不同(例如Ubuntu,Cnetos)

1. Docker镜像原理
* Docker镜像是由特殊的文件系统叠加而成
* 最底端是bootfs，并使用宿主机的bootfs(docker的内核用的是宿主机的内核，内核复用)
* 第二层是root文件系统rootfs，成为bash image
* 然后再往上可以叠加其他的镜像
* 统一文件系统(Union File System)技术能够将不同的层整合成一个文件系统，为这些层提供一个统一的视角，这样就隐藏了多层的存在，在用户的角度来看，只存在一个文件系统
* 一个镜像可以放在另一个镜像的上面。位于下面的镜像称为父镜像，最底部的镜像称为基础镜像。
* 当从一个镜像启动容器时，Docker会在最顶层加载一个读写文件系统作为容器

![docker镜像图解]()<br>

---

# Dockerfile
* Docker镜像如何制作？
1. 容器转化为镜像
```bash
$docker commit 容器id 镜像名称:版本号
$docker save -o 压缩文件名称 镜像名称:版本号
$docker load -i 压缩文件名称
```
*挂载的目录不会打包到新镜像中*
![容器转为镜像]()



2. dockerfile
* Dockerfile是一个文本文件
* 包含了一条条指令
* 每一条指令构建一层，基于基础镜像，最终构建出一个新的镜像
* 对于开发人员：可以为开发团队提供一个完全一致的开发环境
* 对于测试人员：可以直接拿开发时所构建的镜像或者通过Dockerfile文件构建一个新的镜像开始工作
* 对于运维人员：在部署时，可以实现应用的无缝移植

dockerfile关键字：
FROM MAINTAINER LABEL RUN CMD ENTRYPOINT COPY ADD ENV ARG
......

3. Dockerfile案例<br>
   * 需求：定义Dokcerfile，发布Springboot。
   * 实现步骤
     1. 编写dockerfile文件
     2. 通过dockerfile构建镜像：```docker build -f dockerfile [文件路径] -t [镜像名称:版本号]```
     3. 通过镜像创建容器
     4. 从web访问IP:端口号，成功访问到容器上的项目
```dockerfile
FROM Java:8    # 定义父镜像
MAINTAINER charleshe<charleshe.top>   # 定义作者信息
ADD springboot.jar app.jar    # 将jar包添加到容器
CMD java -jar app.jar         #定义容器启动执行的命令
```

4. Dockerfile案例<br>
   * 需求：自定义CentOS7镜像，默认登录路径为/usr，可以使用vim。
   * 实现步骤
     1. 编写dockerfile文件
```dockerfile
FROM centos:7  # 定义父镜像
MAINTAINER charleshe<charles_hjc@outlook.com>   # 定义作者信息
RUN yum install -y vim     #执行安装vim命令
WORKDIR /usr               # 定义默认的工作目录
CMD /bin/bash              # 定义容器气动执行的命令
```

---

# Docker服务编排

#### 微服务架构的应用系统中一般包含若干个微服务，每一个微服务一般都会部署多个实例，如果每个微服务都要手动启停，维护的工作量会很大。

* 要从Dockerfile build image或者去dockerhub拉取image
* 要创建多个container
* 要管理这些container(启动停止删除)

#### 服务编排：按照一定的业务规则批量管理容器
#### Docker Compose：是一个编排多容器分布式部署的工具，提供命令集管理容器化应用的完整开发周期，包括服务构建，启动和停止。
1. 利用Dockerfile定义运行环境镜像
2. 使用docker-compose.yml定义组成应用的各服务
3. 运行docker-compose up启动应用

![docker-compose图解]()

# Docker Compose安装和启用
1. 安装docker-compose
```bash
# Compose目前已经完全支持Linux、Mac OS和Windows，在我们安装Compose之前，需要先安装Docker。下面我 们以编译好的二进制包方式安装在Linux系统中。 
$curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
# 设置文件可执行权限 
$chmod +x /usr/local/bin/docker-compose
# 查看版本信息 
$docker-compose -version
```

2. 卸载docker-compose
```bash
# 二进制包方式安装的，删除二进制文件即可
rm /usr/local/bin/docker-compose
```

3. 使用docker compose编排nginx+springboot项目
* 创建docker-compose目录
```bash
mkdir ~/docker-compose
cd ~/docker-compose
```

* 编写 docker-compose.yml 文件
```bash
version: '3'
services:
  nginx:
   image: nginx
   ports:
    - 80:80
   links:
    - app
   volumes:
    - ./nginx/conf.d:/etc/nginx/conf.d
  app:
    image: app
    expose:
      - "8080"
```

* 创建./nginx/conf.d目录
```shell
mkdir -p ./nginx/conf.d
```

* 在./nginx/conf.d目录下 编写itheima.conf文件
```shell
server {
    listen 80;
    access_log off;

    location / {
        proxy_pass http://app:8080;
    }
   
}
```

* 在~/docker-compose 目录下 使用docker-compose 启动容器
```shell
docker-compose up
```

* 测试访问
```shell
http://192.168.149.135/hello
```

---

# Docker私有仓库
官方的公共仓库：Docker hub(https://hub.docker.com)

### 一、私有仓库搭建

```bash
# 1、拉取私有仓库镜像 
$docker pull registry
# 2、启动私有仓库容器 
$docker run -id --name=registry -p 5000:5000 registry
# 3、打开浏览器 输入地址http://私有仓库服务器ip:5000/v2/_catalog，看到{"repositories":[]} 表示私有仓库 搭建成功
# 4、修改daemon.json   
$vim /etc/docker/daemon.json    
# 在上述文件中添加一个key，保存退出。此步用于让 docker 信任私有仓库地址；注意将私有仓库服务器ip修改为自己私有仓库服务器真实ip 
{"insecure-registries":["私有仓库服务器ip:5000"]} 
# 5、重启docker 服务 
$systemctl restart docker
$docker start registry
```

### 二、将镜像上传至私有仓库

```bash
# 1、标记镜像为私有仓库的镜像     
$docker tag centos:7 私有仓库服务器IP:5000/centos:7
 
# 2、上传标记的镜像     
$docker push 私有仓库服务器IP:5000/centos:7
```


### 三、 从私有仓库拉取镜像 

```bash
#拉取镜像 
$docker pull 私有仓库服务器ip:5000/centos:7
```

---

# Docker相关概念
 ### 一、 docker容器虚拟化 与 传统虚拟机比较
 * 容器就是将软件打包成标准化单元，以用于开发、交付和部署
 * 容器镜像是轻量的、可执行的独立软件包，包含软件运行所需的所有内容：代码、运行环境、系统工具、系统库和设置
 * 容器化软件在任何环境中都能够始终如一的运行
 * 容器服务了软件独立性，使其免受外在环境差异的影响，从而有助于减少团队间在相同基础设施上运行不同软件时的冲突。
![docker容器虚拟化]()

![docker架构图示]()
![VM架构图示]()

#### 相同点：
* 容器和虚拟机具有相似的资源隔离和分配优势

#### 不同点：
* docker虚拟化的是操作系统，VM虚拟化的是硬件
* 传统虚拟机可以运行不同的操作系统，容器只能运行同一类型操作系统(因为要共享内核)
![docker和VM区别]()

### 虚拟机已死，容器才是未来