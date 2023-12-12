### docker安装

1. 更新源
   ```sh
   yum -y update
   ```
2. 移除旧版本的docker
   ```sh
   yum remove docker docker-common docker-selinux docker-engine
   ```
3. 安装docker的yum库
   ```sh
   ## 安装yum工具
   
   yum install -y yum-utils
   
   ## 配置docker的yum源
   
   yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo 
   ```
4. 安装docker
   ```
   yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   ```
5. 验证安装
   ```sh
   ## 查看docke版本
   docker -v
   
   docker images
   ## 连接失败
   ```
6. 启动和校验
   ```sh
   # 启动docker
   systemctl start docker
   # 停止docke
   systemctl stop docker
   # 重启
   sysmtectl restart docker
   # 设置开机自启
   systemctl enable docker
   # 执行docker ps命令，不报错说明安装成功
   docker ps
   ```
7. 配置镜像加速
   ```sh
   # 使用阿里云加速
   ## 登录阿里云
   ## 开通镜像服务
   ## 产品=>容器=>ACR
   ## 镜像工具=》镜像加速器
   
   ## 修改daemon配置文件
   
   mkdir -p /etc/docker
   
   tee /etc/docker/daemon.json <<-'EOF'
   {
     "registry-mirrors": ["https://te8qk3rv.mirror.aliyuncs.com"]
   }
   EOF
   
   systemctl daemon-reload
   
   systemctl restart docker
   ```

### docker部署MySQL

停止服务器的MySQL、服务器已安装好docker、网络开通

```sh
docker run -d \
--name mysql \
-p 3306:3306 \
-e TZ=Asia/Shanghai \
-e MYSQL_ROOT_PASSWORD=root \
mysql

##docker run 创建并运行一个容器，-d 让容器在后台运行
##--name mysql 给容器起名字，唯一
## -p 宿主机端口3306:容器端口3306 设置端口映射
## -e KEY=VALUE 设置环境变量
## mysql 运行的镜像的名字  镜像名:镜像版本  不写版本默认最新版本
## mysql容器会创建默认匿名挂载，操作不方便

#### 修改宿主机端口多次执行即可配置MySQL集群

##报错 You have to remove (or rename) that container to be able to reuse that name.
##docker ps 查看容器
##没有容器 docker ps -l 显示最新创建的容器和状态
##copy containerId 
##删除容器 docker rm containerId 即可

##格式化容器信息
##docker ps --format "table {{.ID}}\t{{.Image}}\t{{.Ports}}\t{{.Status}}\t{{.Names}}"
##docker ps -a 展示所有容器(包括停止的容器)
```

本地远程连接服务器MySQL

Docker 安装应用时会自动搜索并下载应用镜像(image)，镜像除应用本身还包括了应用运行需要的环境、配置、系统函数库；docker在运行镜像时会创建一个隔离环境，容器(container)

![截图](519c7bcc53391b357069eddf6a7f2773.png)

<br/>

### docker常见命令

```
https://docs.docker.com/
```

![截图](f011128f95eb6886380df35da0c406ad.png)

```sh
# 查看日志
docker logs -f 容器名 #-f查看持续产生的日志

# 进入容器
docker exec -it 容器名 bash #-it 添加一个可输入的终端  bash命令行交互方式

# 进入容器&&进入MySQL
docker exec -it mysql2 mysql -uroot -p

## 设置命令别名
vi ~/.bashrc

alias 自定义命令 = '原始命令'

## 使别名生效

source ~/.bashrc
```

```sh
# 创建nginx容器

# 修改容器内html目录下index.html，查看变化
## 容器只包含运行的最小环境，编辑文件是一个问题
### 解决方法：数据卷volume
### 数据卷是一个虚拟目录，是容器目录与宿主机目录之间映射的桥梁
### 挂载容器目录
#### 执行docker run 使用 -v 数据卷:容器内目录 ==> 挂载数据卷
#### 创建容器时，若挂载了数据卷且数据卷不存在，自动创建数据卷
 docker run -d --name nginx -p 80:80 -v html:/usr/share/nginx/html nginx
 
# 将静态资源部署到nginx
```

- 数据卷 映射宿主机、容器

![截图](a0bfd852563c5a41e8af9c97fb4e2b7d.png)

- 数据卷命令

```sh
docker volume create #创建数据卷
docker volume ls #查看所有数据卷
docker volume rm #删除指定数据卷
docker volume inspect #查看某个数据卷的详情
docker volume prune #清楚未使用的数据卷

```

```sh
docker inspect #查看数据卷详情
```

- mysql容器数据挂载

```sh
`#基于宿主机目录实现MySQL数据目录、配置文件、初始化脚本挂载
#本地路径
docker run -v 本地目录:容器内目录
# 本地目录须以'/'或'./'开头，以名称开头会被识别为数据卷而非本地目录

docker run -d \
--name mysql \
-p 3306:3306 \
-e TZ=Asia/Shanghai \
-e MYSQL_ROOT_PASSWORD=root \
-v /root/mysql/data:/var/lib/mysql \
-v /root/mysql/init:/docker-entrypoint-initdb.d \
-v /root/mysql/conf:/etc/mysql/conf.d \
mysql
```

conf配置文件

```sh
[client]
default_character_set=utf8mb4
[mysql]
default_character_set=utf8mb4
[mysqld]
character_set_server=utf8mb4
collation_server=utf8mb4_unicode_ci
init_connect='SET NAMES utf8mb4'
```

init初始化脚本

```
sql脚本
```

<br/>

### DockerFile

#### 镜像结构

镜像中包含了应用程序所需要的运行环境、函数库、配置，及应用本身等各种文件，分层打包而成。

分层打包繁琐，使用dockerfile描述镜像结构和构建过程，使得docker可以依次构建镜像

<br/>

---

https://docs.docker.com/engine/reference/builder/

```sh
dockerfile 包含需要执行构建镜像需要的操作的指令，docker可以根据dockerfile构建镜像
# 指定基础镜像
FROM # FROM centos:6
#设置环境变量
ENV #ENV KEY VALUE
#拷贝本地文件到镜像指定目录
COPY #COPY ./jrell.tar/gz /temp 
#执行Linux的shell命令，一般为安装过程命令
RUN #RUN tar -zxvf /temp/jrell.tar.gz && EXPORTS path=/temp/jrell:$path
#指定容器运行时监听的端口，类似于注释提示
EXPOSE #EXPOSE 8080
#镜像中应用的启动命令，容器运行时调用
ENTRYPOINT #ENTRYPOIONT java -jar xx.jar
```

<br/>

- 基于Ubuntu基础镜像构建Java环境

![截图](5ac74c3e9f2bd053b73b0862d3a7e531.png)

- 基于Java11构建镜像

![截图](69e80400593634c01224eed0ce7afb5a.png)

#### 镜像制作

```sh
#构建镜像
docker build -t myImage:1.0 .

#加载tar包镜像
docker load -i xxx.tar


```

#### 容器网络通信

所有容器以bridge方式连接到docker的一个虚拟网桥上（docker容器自动分配）

![截图](9203bdcce179510278474a7f40bde520.png)

<br/>

加入自定义网络使得可以通过容器名互相访问

```sh
#创建一个网络
docker network create
#查看所有网络 
docker network ls
#删除指定网络
docker network rm
#清楚未使用的网络
docker network prune
#使指定容器连接加入某网络  容器已存在
docker network connect
#使指定容器连接离开某网络
docker network disconnect
#查看网络详细信息
docker network inspect
```

```sh
#创建容器时分配网桥
docker run -d \
--name dd \
-p 8080:8080 \
--network dnet \
docker-demo
```

#### 前后端分离部署

- nginx部署前端

```sh
docker run -d \
--name nginx \
-p 18080:18080 \
-p 18081:18081 \
-v /root/nginx/html:/usr/share/nginx/html \
-v /root/nginx/nginx.conf:/etc/nginx/nginx.conf \
--network dnet \
nginx
```

<br/>

### 使用DockerCompose关联多个容器

DockerCompose 通过单独的docker-compose.yml文件定义一组相关联的应用容器，帮助开发者实现多个相互关联的docker容器的快速部署

一个完整的项目=>一个DockerCompose文件

一个容器=>一个服务service

```yaml
version: "3.8"

services:
  containerA:
    image: A
    container_name:A
    ports:
      - "11:11"
  containerB:
    image:B
    container_name:B
    ports:
      - "22:22"
  containerC:
    image:C
    container_name:C
    ports:
      - "33:33"
```

<br/>

DockerCompose文件

```yaml
version: "3.8"

services:
  mysql:
    image: mysql
    container_name: mysql
    ports:
      - "3306:3306"
    environment:
      TZ: Asia/Shanghai
      MYSQL_ROOT_PASSWORD: root
    volumes:
      #相对路径，如有需要改成绝对路径
      - "./mysql/conf:/etc/mysql/conf.d"
      - "./mysql/data:/var/lib/mysql"
      - "./mysql/init:/docker-entrypoint-initdb.d"
    networks:
      - hm-net
  hmall:
    build: 
      context: .
      dockerfile: Dockerfile
    container_name: hmall
    ports:
      - "8080:8080"
    networks:
      - hm-net
    depends_on:
      - mysql
  nginx:
    image: nginx
    container_name: nginx
    ports:
      - "18080:18080"
      - "18081:18081"
    volumes:
      - "./nginx/nginx.conf:/etc/nginx/nginx.conf"
      - "./nginx/html:/usr/share/nginx/html"
    depends_on:
      - hmall
    networks:
      - hm-net
networks:
  hm-net:
    name: hmall
```

<br/>

<br/>

DockerCompose指令

![截图](c1623977062e7190b8276a8d942e7aa2.png)

docker compose up -d
