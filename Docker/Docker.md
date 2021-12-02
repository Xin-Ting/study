# Docker
## 1、什么是Docker？
Docker是一个软件，可以把其它软件安装到Docker中，可以通过Docker相关的命令去操作这些软件，比如：安装/下载/删除/导入/导出等。
Docker使用的软件是特定的软件，和我们平时下载的软件不一样，我们称**Docker的软件安装包为镜像**。一个镜像可以安装多次。镜像要从DockerHub(公有仓库，Docker官方维护)下载。
- Docker安装软件的步骤：
	1. 从镜像仓库下载镜像(软件安装包)；
	2. 把镜像安装到Docker中，**安装好的软件称为容器**，如：安装一个MySQL软件，安装好后叫做MySQL容器；
	3. 配置使用容器。
## 2、Docker解决了什么问题？
可以在一台电脑上安装软件，将其配置好并且导出，就可以在其它电脑上直接使用。解决了我们在开发过程中开发环境、测试环境、线上环境等软件和配置不同步的问题。
## 3、Docker的使用——命令
### 3.1 Docker服务相关命令
```shell
systemctl -help         #查看帮助

systemctl 操作 服务   #centos 7 相关服务命令

systemctl start docker      #启动Docker服务
systemctl stop docker      #停止Docker服务
systemctl restart docker   #重启Docker服务
systemctl status  docker   #查看Docker服务状态
systemctl enable  docker  #开机启动Docker服务

```

### 3.2 ==Docker镜像相关命令==
```shell
docker -help   			#查看帮助

docker images  							 #查看镜像
docker search 镜像名  		  		 	#搜索镜像
docker pull 镜像名:版本号  		  	#下载镜像，不加版本号默认最新
docker rmi 镜像名:版本号/镜像ID  #删除镜像，rm是删除容器

```

### 3.3 ==Docker容器相关命令==
```shell
docker -help   			#查看帮助
#查看
docker ps                #查看正在运行的容器
docker ps -a            #查看所有容器(包括已经停止运行的容器)

# 【重点】创建：以下三种类型仅仅只在创建时有区别，一旦容器创建完再通过docker start启动后所有的容器都是一样的，退出不会关闭
docker run -it --name=容器的别名 镜像名:版本号 /bin/bash    
# 交互式容器：创建完这个容器后，容器会立即启动并且自动进入到容器内部，退出容器后，容器会自动关闭。  -t：代表容器是交互式的；/bin/bash：固定写法，代表进入终端

docker run -id --name=容器的别名 镜像名:版本号
# 守护式容器：创建完这个容器后，容器会立即启动但不会自动进入到容器内部，退出容器后，容器不会关闭。 -d：代表容器是守护式的；

docker create --name=容器的别名 镜像名:版本号
# 创建式容器：创建完这个容器不会立即启动，也不会进入到容器内部

# 其它(任何时候容器名都可以和容器ID互换)
docker start 容器名/容器ID        				  #启动容器 (可以写多个)
docker exec -it 容器名/容器ID /bin/bash    #进入容器
docker stop 容器名/容器ID                          #停止容器
docker rm 容器名/容器ID                          #删除容器(不能删除正在运行的容器，需要先停止)
exit                   											#退出容器
docker inspect 容器名/容器ID                   #查看容器信息
```

### 3.4 ==数据卷==
#### 3.4.1 数据卷解决什么问题？
- 容器删除了，容器中的数据还在吗？不在
- 容器和宿主机(Docker安装的系统，比如：Linux系统)之间可以直接通信，但是window系统和容器之间不能直接通信
- 容器和容器之间是相互独立的，不能直接通信
#### 3.4.2 什么是数据卷？
![](img/Pasted%20image%2020211201113755.png)
**宿主机上的一个文件或者文件夹挂载到了容器上这个文件或者文件夹就称之为数据卷。**
**数据卷容器：一个容器只要挂载了宿主机上的目录，这个容器就称之为数据卷容器。**
#### 3.4.3 数据卷的作用
1. 容器数据持久化
2. 外部机器与容器间接通信
3. 容器之间数据交互

#### 3.4.4 数据卷命令
```shell
docker run -id --name=容器别名 -v 宿主机数据卷目录:容器中映射目录 [-v ... -v ...] 镜像名:版本号  
#三种创建方式都可以加  []中为可选 

# 进入到MySQL容器中，找到数据库文件存储在哪？
-v /home/mysql/data:从容器中找到对应的数据库文件存储目录(mysql容器指定创建，不能更改)
```
### 3.5 常用服务器部署
Tomcat
```shell
docker run -id --name=mycat \
-p 8080(宿主机端口):8080(容器端口) \      #端口映射
-v /home/mycat(宿主机目录，可以更改):/usr/local/tomcat/webapps(tomcat容器目录，不能更改) \
tomcat

```
MySQL
Nginx
Redis
【总结】安装命令，复制粘贴
### 3.6 容器导入/导出
```shell
docker commit 容器名 镜像名   #容器导出为镜像
docker save -o tar 文件名 镜像名   #镜像导出为压缩文件
docker load -i tar 文件名   #导入压缩文件为镜像
```

### 3.7  Dockerfile
 Dockerfile：是一个普通文件，可以编译成一个Docker镜像，我们可以自己去编写一个Dockerfile文件就可以制作自己的镜像。
 ![](img/Pasted%20image%2020211201170328.png)