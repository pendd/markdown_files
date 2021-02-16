

## Docker的常用命令

### 帮助命令

```shell
docker version     # 显示docker的版本信息
docker info        # 显示docker的系统信息，包括镜像和容器的数量
docker 命令 --help  # 帮助命令
```

### 镜像命令

**docker images 查看所有本地的主机上的镜像**

```shell
[root@pd ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              bf756fb1ae65        7 months ago        13.3kB

# 解释
REPOSITORY  镜像的仓库源
TAG					镜像的标签
IMAGE ID    镜像的ID
CREATED			镜像的创建时间
SIZE				镜像的大小

# 可选项
-a, --all   # 列出所有镜像
-q, --quiet # 只显示镜像的id
```

**docker search 搜索镜像**

```shell
[root@pd ~]# docker search mysql
NAME                              DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   9891                [OK]                
mariadb                           MariaDB is a community-developed fork of MyS…   3619                [OK]                
mysql/mysql-server                Optimized MySQL Server Docker images. Create…   721                                     [OK]

# 可选项 通过收藏来过滤
--filter=STARS=3000  搜索出来的镜像就是STARS大于3000的

[root@pd ~]# docker search mysql --filter=stars=3000
NAME                DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
mysql               MySQL is a widely used, open-source relation…   9891                [OK]                
mariadb             MariaDB is a community-developed fork of MyS…   3619                [OK]
```

**docker pull 下载镜像**

```shell
# 下载镜像 docker pull 镜像名[:tag]
[root@pd ~]# docker pull mysql
Using default tag: latest  # 如果不写tag 默认就是 latest
latest: Pulling from library/mysql
bf5952930446: Pull complete  # 分层下载，docker image的核心 联合文件系统
8254623a9871: Pull complete 
938e3e06dac4: Pull complete 
ea28ebf28884: Pull complete 
f3cef38785c2: Pull complete 
894f9792565a: Pull complete 
1d8a57523420: Pull complete 
6c676912929f: Pull complete 
ff39fdb566b4: Pull complete 
fff872988aba: Pull complete 
4d34e365ae68: Pull complete 
7886ee20621e: Pull complete 
Digest: sha256:c358e72e100ab493a0304bda35e6f239db2ec8c9bb836d8a427ac34307d074ed # 签名
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest #真实地址

#下面两个是等价的
docker pull mysql
docker.io/library/mysql:latest

#指定版本下载
[root@pd ~]# docker pull mysql:5.7
5.7: Pulling from library/mysql
bf5952930446: Already exists 
8254623a9871: Already exists   # exists 是联合文件系统特性，之前最新版本里面已经有了，现在5.7共用这些文件
938e3e06dac4: Already exists 
ea28ebf28884: Already exists 
f3cef38785c2: Already exists 
894f9792565a: Already exists 
1d8a57523420: Already exists 
5f09bf1d31c1: Pull complete 
1b6ff254abe7: Pull complete 
74310a0bf42d: Pull complete 
d398726627fd: Pull complete 
Digest: sha256:da58f943b94721d46e87d5de208dc07302a8b13e638cd1d24285d222376d6d84
Status: Downloaded newer image for mysql:5.7
docker.io/library/mysql:5.7
```

**docker rmi 删除镜像**

```shell
# 删除指定镜像
[root@pd ~]# docker rmi -f 镜像id  
# 删除多个镜像
[root@pd ~]# docker rmi -f 镜像id 镜像id 镜像id
# 删除全部镜像
[root@pd ~]# docker rmi -f $(docker images -aq)
```

### 容器命令

**说明：我们有了镜像才可以创建容器，下载一个centos镜像来测试学习**

```shell
docker pull centos
```

**新建容器并启动**

```shell
docker run [可选参数] image

# 参数说明
--name="Name"    容器名字 tomcat01 tomcat02 用来区分容器
-d							 后台方式运行
-it 						 使用交互方式运行，进入容器查看内容
-p							 指定容器端口 -p 8080:8080
			-p ip:主机端口:容器端口
			-p 主机端口:容器端口（常用）
			-p 容器端口
-p							 随机指定端口

# 测试 启动并进入容器
[root@pd ~]# docker run -it centos /bin/bash
# 查看容器内的centos 基础版本 很多命令都是不完善的
[root@26a5aa5fa294 /]# ls
bin  etc   lib	  lost+found  mnt  proc  run   srv  tmp  var
dev  home  lib64  media       opt  root  sbin  sys  usr
```

**列出所有运行的容器**

```shell
docker ps [可选参数]
	  	# 列出当前正在运行的容器
-a    # 列出当前正在运行的容器+历史运行过的容器
-n=?  # 显示最近创建的n个容器
-q    # 只显示容器的编号

[root@1d044b7bab25 /]# [root@pd ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
1d044b7bab25        centos              "/bin/bash"         6 minutes ago       Up 6 minutes        
--------------------------
[root@pd ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                        PORTS               NAMES
1d044b7bab25        centos              "/bin/bash"         6 minutes ago       Up 6 minutes                                      vigorous_mendel
26a5aa5fa294        centos              "/bin/bash"         16 minutes ago      Exited (127) 14 minutes ago                       upbeat_kapitsa
55f2c60f1b12        hello-world         "/hello"            2 hours ago         Exited (0) 2 hours ago                            eager_villani
--------------------------
[root@pd ~]# docker ps -q
1d044b7bab25
--------------------------
[root@pd ~]# docker ps -n=1
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
1d044b7bab25        centos              "/bin/bash"         7 minutes ago       Up 7 minutes                            vigorous_mendel
```



**退出容器**

```shell
# 容器直接停止并退出
[root@1d044b7bab25 /]# exit  
# 容器退出但不停止
Ctrl + P + Q   
# 重新回去之前未关闭的容器 docker attach 容器ID
[root@pd ~]# docker attach $(docker ps -q) 
```

**删除容器**

```shell
# 删除指定容器 不能删除正在运行的容器 ，需要加参数 -f
docker rm 容器id   
# 删除所有容器
docker rm -f $(docker ps -aq)
# 删除所有容器
docker ps -a -q|xargs docker rm
```

**启动和停止容器的操作**

```shell
docker start    容器id  # 启动容器
docker restart  容器id  # 重启容器
docker stop     容器id  # 停止当前正在运行的容器
docker kill     容器id  # 强制停止当前容器
```

### 常用其他命令

**后台启动容器**

```shell
docker run -d 镜像名
[root@pd ~]# docker run -d centos

# 问题： docker ps 后会发现centos停止了
# 原因： docker容器使用后台运行，就必须要有一个前台进程，docker发现没有应用，就会自动停止
```

**查看日志**

```shell
docker logs -tf --tail 10 容器id

# 自己编写一段shell脚本
docker run -d centos /bin/sh -c "while true;do echo pd;sleep 1;done"

# 显示日志
-tf            # 显示日志
--tail number  # 要显示的日志条数
```

**查看容器中进程信息** 

```shell
docker top 容器id
[root@pd ~]# docker top 497eb6fdd9ba
UID        PID        PPID         C     STIME    TTY                 
root       3902       3874         0     21:33     ?   
root       4606       3902         0     21:41     ?        
```

**查看镜像的元数据**

```shell
docker inspect 容器id
```

**进入当前正在运行的容器**

```shell
# 方式一：docker exec 容器id bashShell

[root@pd ~]# docker exec 497eb6fdd9ba /bin/bash
[root@pd ~]# ls

# 方式二：docker attach 容器id
[root@pd ~]# docker attach 497eb6fdd9ba
dpeng
dpeng
dpeng
dpeng
dpeng
dpeng
dpeng
dpeng

# 区别：
# docker exec    进入容器后开启一个新的终端，可以在里面操作（常用）
# docker attach  进入容器正在执行的终端，不会启动新的进程
```

**从容器内拷贝文件到主机上**

```shell
docker cp 容器id:容器内路径 目的地主机路径

# 进入docker容器内部
[root@pd ~]# docker run -it centos /bin/bash
[root@540e4cd2b8b2 /]# ls 
bin  etc   lib	  lost+found  mnt  proc  run   srv  tmp  var
dev  home  lib64  media       opt  root  sbin  sys  usr
[root@540e4cd2b8b2 /]# cd /home
# 在容器内新建一个文件
[root@540e4cd2b8b2 home]# touch test.java
[root@540e4cd2b8b2 home]# exit
exit
[root@pd ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@pd ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                            PORTS               NAMES
540e4cd2b8b2        centos              "/bin/bash"              52 seconds ago      Exited (0) 9 seconds ago                              magical_lichterman
497eb6fdd9ba        centos              "/bin/sh -c 'while t…"   12 hours ago        Exited (137) About a minute ago                       naughty_chebyshev
1d044b7bab25        centos              "/bin/bash"              16 hours ago        Exited (127) 12 hours ago                             vigorous_mendel
26a5aa5fa294        centos              "/bin/bash"              16 hours ago        Exited (127) 16 hours ago                             upbeat_kapitsa
55f2c60f1b12        hello-world         "/hello"                 18 hours ago        Exited (0) 18 hours ago                               eager_villani
# 将文件从容器拷贝出来到主机上
[root@pd ~]# docker cp 540e4cd2b8b2:/home/test.java /home
[root@pd ~]# ls /home
test.java

```

### Docker安装Nginx

> 步骤

```shell
# 1. 搜索镜像 search  建议去Docker官网搜索，可以看到帮助文档
# 2. 下载镜像 pull
# 3. 运行测试

[root@pd ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              4bb46517cac3        13 days ago         133MB
centos              latest              0d120b6ccaa8        2 weeks ago         215MB

# -d 后台运行
# --name 给容器命名
# -p 宿主机端口:容器内部端口
[root@pd ~]# docker run -d --name nginx01 -p 3344:80 nginx
6b167e2ae4b13cdafc4efa2c2a9ddfc12fcedb80f8d7811570f275919091d19d
[root@pd ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
6b167e2ae4b1        nginx               "/docker-entrypoint.…"   7 seconds ago       Up 6 seconds        0.0.0.0:3344->80/tcp   nginx01

# 测试
[root@pd ~]# curl localhost:3344
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

**端口暴露**

![](https://github.com/pendd/picture/raw/master/Docker/port.png)

### Docker安装Tomcat

```shell
# 官方的使用   加--rm参数，是为了测试使用，退出容器后，该容器就会被删除
docker run --it --rm tomcat:9.0

# 1. 下载tomcat
docker pull tomcat:9.0

# 2. 启动运行
docker run -d -p 3355:8080 --name tomcat01 tomcat

# 3. 进入容器
docker exec -it tomcat01 /bin/bash

# 4. webapps 目录下为空，需要吧webapps.dist目录下文件copy一份到webapps下
cp -r webapps.dist/* webapps

# 5. 测试访问
47.98.221.77:8888  浏览器访问
```

**Docker安装ES+Kibana**

```shell

```

## Docker镜像讲解

### 镜像是什么

镜像是一种轻量级、可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件，它包含运行某个软件所需的所有内容，包括代码、运行时库、环境变量和配置文件。

所有的应用，直接打包成docker镜像，就可以直接跑起来！

**如何得到镜像：**

- 从远处仓库下载
- 朋友拷贝
- 自己制作一个镜像DockerFile

### Docker镜像加载原理

> UnionFS（联合文件系统）

UnionFS（联合文件系统）：Union文件系统（UnionFS）是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下（unite several direcotries into a single virtual filesystem）。Union文件系统是Docker镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。

特性：一次加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录

> Docker镜像加载原理

docker的镜像实际上是由一层一层的文件系统组成，这种层级的文件系统就叫UnionFS。

bootfs（boot file system）主要包含bootloader和kernel，bootloader主要是引导加载kernel，Linux刚启动时会加载bootfs文件系统，在Docker镜像的最底层时bootfs。这一层与我们典型的Linux/Unix系统时一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中来，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs。

rootfs（root file system）在bootfs之上，包含的是典型Linux系统中的/dev，/proc，/bin，/etc等标准目录和文件。rootfs就是各种不同的操作系统发行版，比如Ubuntu，Centos等等。

> 平时我们安装进虚拟机的CentOS都是好几个G，为什么Docker这里才200M？

对于一个精简的OS，rootfs可以很小，只需要包含最基本的命令，工具和程序库就可以了，因为底层直接使用Host的kernel，自己只需要提供rootfs就可以了。由此可见对于不同的Linux发行版，bootfs基本是一致的，rootfs会有差别，因此不同的发行版可以公用bootfs。

### commit镜像

```shell
docker commit 提交容器成为一个新的副本

docker commit -m="提交的描述信息" -a="作者" 容器id 目标镜像名:[TAG]
```

实战测试

```shell
# 1. 启动一个默认的tomcat

# 2. 发现这个默认的tomcat webapps下是空的

# 3. 拷贝webapps.dist目录下的所有文件到webapps

# 4. 将我们操作过的容器通过commit提交为一个镜像，我们以后就可以使用我们修改过的镜像即可

[root@pd ~]# docker commit -m="copy webapps.dis/* to webapps port:8888" -a="dpeng" ed1d926a8245 tomcat_dpeng:1.0
sha256:28b71da6601771f218cb02fcdd3610a56f209d0037f31abc8b15aa5e4d428a5c
[root@pd ~]# docker images
REPOSITORY    TAG   IMAGE ID       CREATED       SIZE
tomcat_dpeng  1.0  28b71da66017  6 seconds ago   657MB
```

## 容器数据卷

### 使用数据卷

> 方式一：直接使用命令挂载 -v

```shell
docker run -it -v 主机目录:容器内目录

# 测试
[root@pd home]# docker run -it -v /home/ceshi:/home centos /bin/bash

# 启动起来之后我们可以通过 docker inspect 容器id
  "Mounts": [   # 挂载 -v 卷
            {
                "Type": "bind",
                "Source": "/home/ceshi",  # 主机内地址
                "Destination": "/home",   # docker容器内的地址
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
        ]
```

挂载之后，宿主机目录和容器内目录下的数据是同步的，

即使停止容器，宿主机修改文件，再次启动容器，容器内的数据依旧是同步的。

**实战：挂载MySQL**

```shell
# 运行容器，需要做数据挂载
# 安装启动mysql，需要配置密码！！！
# 官方测试： 
docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag

# 启动自己的mysql
-v 卷挂载
-e 环境配置
docker run -d -p 3306:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7

# 启动以后用可视化工具远程连接测试
# 在本地测试创建一个数据库，查看映射路径是否ok
```

**具名挂载和匿名挂载**

```shell
# 匿名挂载
-v 容器内路径
-P 随机指定端口
docker run -d -P --name nginx01 -v /etc/nginx/nginx

# 查看所有的volume情况
[root@pd ~]# docker volume list
DRIVER   VOLUME NAME
local               730800c17f2830a3fe93acff2141a198e4b203ed151d3870ba243e3344bbf602

# 具名挂载
-v 卷名:容器内路径
[root@pd ~]# docker run -d -P --name nginx03 -v juming-nginx:/etc/nginx nginx
[root@pd ~]# docker volume ls
DRIVER              VOLUME NAME
local               juming-nginx


[root@pd ~]# docker volume inspect juming-nginx
[
    {
        "CreatedAt": "2020-08-28T15:22:10+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/juming-nginx/_data",
        "Name": "juming-nginx",
        "Options": null,
        "Scope": "local"
    }
]
```

所有docker容器内的卷，没有指定目录的情况下都是在`/var/lib/docker/volumes/xxx/_data`

我们通过剧名挂载可以方便的找到我们的一个卷，大多数情况在使用的是`具名挂载`

> 如果区分具名挂载、匿名挂载和指定路径挂载？

```shell
-v 容器内路径           # 匿名挂载
-v 卷名:容器内路径       # 具名挂载
-v /宿主机路径:容器内路径 # 指定路径挂载
```

> 如何指定容器权限？

```shell
-v 容器内路径:ro   # 只读
-v 容器内路径:rw   # 可读可写

# 不写默认是rw
docker run -d -P --name nginx03 -v juming-nginx:/etc/nginx:ro nginx
docker run -d -P --name nginx03 -v juming-nginx:/etc/nginx:rw nginx
```

### 初识Dockerfile

Dockerfile就是用来构建docker镜像的构建文件

```shell
# 创建一个dockerfile文件，名字可以随意， 建议Dockerfile
# 文件中的内容 指定（大写） 参数
FROM centos
# 匿名挂载
VOLUME ["volume01", "volume02"]
CMD echo "----end----"
CMD /bin/bash

# 这里的每个命令，对应镜像的每一层
```

```shell
[root@pd ~]# cd /home
[root@pd home]# ls 
[root@pd home]# vim dockerfile

# 一定要加后面的.
[root@pd home]# docker build -f dockerfile -t mycentos:1.0 .
Sending build context to Docker daemon  2.048kB
Step 1/4 : FROM centos
 ---> 0d120b6ccaa8
Step 2/4 : VOLUME ["volume01", "volume02"]
 ---> Running in d6680ce34d94
Removing intermediate container d6680ce34d94
 ---> 250d7491c5d9
Step 3/4 : CMD echo "----end----"
 ---> Running in a0336c9975ef
Removing intermediate container a0336c9975ef
 ---> 54ed9d75cf1a
Step 4/4 : CMD /bin/bash
 ---> Running in 52b118adf9cd
Removing intermediate container 52b118adf9cd
 ---> 5ab599cc7cb9
Successfully built 5ab599cc7cb9
Successfully tagged mycentos:1.0

[root@pd home]# docker images
REPOSITORY  TAG   IMAGE ID        CREATED           SIZE
mycentos    1.0  5ab599cc7cb9  About a minute ago   215MB

# 启动自己的刚刚build的容器   可以使用image id 启动也可以名称:tag启动
[root@pd home]# docker run -it mycentos:1.0 /bin/bash
[root@9be6a2f40c95 /]# ls -l
total 56
lrwxrwxrwx  1 root root    7 May 11  2019 bin -> usr/bin
drwxr-xr-x  5 root root  360 Aug 31 03:18 dev
drwxr-xr-x  1 root root 4096 Aug 31 03:18 etc
drwxr-xr-x  2 root root 4096 May 11  2019 home
lrwxrwxrwx  1 root root    7 May 11  2019 lib -> usr/lib
lrwxrwxrwx  1 root root    9 May 11  2019 lib64 -> usr/lib64
drwx------  2 root root 4096 Aug  9 21:40 lost+found
drwxr-xr-x  2 root root 4096 May 11  2019 media
drwxr-xr-x  2 root root 4096 May 11  2019 mnt
drwxr-xr-x  2 root root 4096 May 11  2019 opt
dr-xr-xr-x 92 root root    0 Aug 31 03:18 proc
dr-xr-x---  2 root root 4096 Aug  9 21:40 root
drwxr-xr-x 11 root root 4096 Aug  9 21:40 run
lrwxrwxrwx  1 root root    8 May 11  2019 sbin -> usr/sbin
drwxr-xr-x  2 root root 4096 May 11  2019 srv
dr-xr-xr-x 13 root root    0 Aug 26 06:59 sys
drwxrwxrwt  7 root root 4096 Aug  9 21:40 tmp
drwxr-xr-x 12 root root 4096 Aug  9 21:40 usr
drwxr-xr-x 20 root root 4096 Aug  9 21:40 var
# 这两个目录就是生成镜像时自动挂载的数据卷目录
drwxr-xr-x  2 root root 4096 Aug 31 03:18 volume01
drwxr-xr-x  2 root root 4096 Aug 31 03:18 volume02

[root@pd ~]# docker inspect 9be6a2f40c95
"Mounts": [
            {
                "Type": "volume",
                "Name": "c3e3d3239351c560785023ac35f3f8847880f0dbe51e7ff1a8f9894d19016538",
                # 数据卷挂载目录
                "Source": "/var/lib/docker/volumes/c3e3d3239351c560785023ac35f3f8847880f0dbe51e7ff1a8f9894d19016538/_data",
                "Destination": "volume01",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            },
            {
                "Type": "volume",
                "Name": "772ba3eefbd6d92c632954ca359af30c9f57292fec842716c0c432a9d5533ab9",
                "Source": "/var/lib/docker/volumes/772ba3eefbd6d92c632954ca359af30c9f57292fec842716c0c432a9d5533ab9/_data",
                "Destination": "volume02",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],

```

### 数据卷容器

`--volumes-from`

```shell
[root@pd home]# docker run -it --name docker01 pd/centos:1.0
[root@418af88d2d66 /]# ls -l
total 56
lrwxrwxrwx  1 root root    7 May 11  2019 bin -> usr/bin
drwxr-xr-x  5 root root  360 Aug 31 05:06 dev
drwxr-xr-x  1 root root 4096 Aug 31 05:06 etc
drwxr-xr-x  2 root root 4096 May 11  2019 home
lrwxrwxrwx  1 root root    7 May 11  2019 lib -> usr/lib
lrwxrwxrwx  1 root root    9 May 11  2019 lib64 -> usr/lib64
drwx------  2 root root 4096 Aug  9 21:40 lost+found
drwxr-xr-x  2 root root 4096 May 11  2019 media
drwxr-xr-x  2 root root 4096 May 11  2019 mnt
drwxr-xr-x  2 root root 4096 May 11  2019 opt
dr-xr-xr-x 91 root root    0 Aug 31 05:06 proc
dr-xr-x---  2 root root 4096 Aug  9 21:40 root
drwxr-xr-x 11 root root 4096 Aug  9 21:40 run
lrwxrwxrwx  1 root root    8 May 11  2019 sbin -> usr/sbin
drwxr-xr-x  2 root root 4096 May 11  2019 srv
dr-xr-xr-x 13 root root    0 Aug 26 06:59 sys
drwxrwxrwt  7 root root 4096 Aug  9 21:40 tmp
drwxr-xr-x 12 root root 4096 Aug  9 21:40 usr
drwxr-xr-x 20 root root 4096 Aug  9 21:40 var
drwxr-xr-x  2 root root 4096 Aug 31 05:06 volume01
drwxr-xr-x  2 root root 4096 Aug 31 05:06 volume02
```

`docker01 创建的内容同步到了docker02中`

```shell
[root@pd ~]# docker run -it --name docker02 --volumes-from docker01 pd/centos:1.0
[root@f934f2ec7f57 /]# ls -l
total 56
lrwxrwxrwx  1 root root    7 May 11  2019 bin -> usr/bin
drwxr-xr-x  5 root root  360 Aug 31 05:08 dev
drwxr-xr-x  1 root root 4096 Aug 31 05:08 etc
drwxr-xr-x  2 root root 4096 May 11  2019 home
lrwxrwxrwx  1 root root    7 May 11  2019 lib -> usr/lib
lrwxrwxrwx  1 root root    9 May 11  2019 lib64 -> usr/lib64
drwx------  2 root root 4096 Aug  9 21:40 lost+found
drwxr-xr-x  2 root root 4096 May 11  2019 media
drwxr-xr-x  2 root root 4096 May 11  2019 mnt
drwxr-xr-x  2 root root 4096 May 11  2019 opt
dr-xr-xr-x 99 root root    0 Aug 31 05:08 proc
dr-xr-x---  2 root root 4096 Aug  9 21:40 root
drwxr-xr-x 11 root root 4096 Aug  9 21:40 run
lrwxrwxrwx  1 root root    8 May 11  2019 sbin -> usr/sbin
drwxr-xr-x  2 root root 4096 May 11  2019 srv
dr-xr-xr-x 13 root root    0 Aug 26 06:59 sys
drwxrwxrwt  7 root root 4096 Aug  9 21:40 tmp
drwxr-xr-x 12 root root 4096 Aug  9 21:40 usr
drwxr-xr-x 20 root root 4096 Aug  9 21:40 var
# 下面两个目录可以体现
drwxr-xr-x  2 root root 4096 Aug 31 05:06 volume01
drwxr-xr-x  2 root root 4096 Aug 31 05:06 volume02
```



## DockFile

### 构建步骤

1. 编写一个Dockerfile文件
2. docker build 构建成为一个镜像
3. docker run 运行镜像
4. docker push 发布镜像（DockerHub、阿里云镜像仓库）

### DockerFile的指令

```shell
FROM          # 基础镜像，一切从这里开始构建
MAINTAINER   # 镜像是谁写的。姓名+邮箱
RUN           # 镜像构建的时候需要运行的命令
ADD           # 添加内容
WORKDIR       # 镜像的工作目录
VOLUME        # 挂载的目录
EXPOSE        # 暴露端口配置
CMD           # 指定这个容器启动的时候要运行的命令，只有最后一个生效，可被替代
ENTRYPOINT    # 指定这个容器启动的时候要运行的命令，可以追加命令
ONBUILD       # 当构建一个被继承DockerFile 这个时候就会运行ONBUILD的指定，触发指定
COPY          # 类似ADD，将文件拷贝到镜像中
ENV           # 构建的时候设置环境变量
```

### 实战测试

```shell
# 1. 编写Dockerfile文件
[root@pd dockerfile]# cat mydockerfile-centos-second-test 
FROM centos
MAINTAINER pd<123456@qq.com>

ENV MYPATH /usr/local
# 进入centos中的根目录就是这个
WORKDIR $MYPATH

RUN yum -y install vim
# 网络相关命令
RUN yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH
CMD echo "----end----"
CMD /bin/bash

# 2. 构建镜像
[root@pd dockerfile]# docker build -f mydockerfile-centos-second-test -t mycentos:0.1 .
Sending build context to Docker daemon  3.072kB
Step 1/10 : FROM centos
 ---> 0d120b6ccaa8
Step 2/10 : MAINTAINER pd<123456@qq.com>
 ---> Running in 8ab5a1a03997
Removing intermediate container 8ab5a1a03997
 ---> 39b62d8401e7
Step 3/10 : ENV MYPATH /usr/local
 ---> Running in 93fee2ec49ec
Removing intermediate container 93fee2ec49ec
 ---> e855b87fe764
Step 4/10 : WORKDIR $MYPATH
 ---> Running in dae341962153
Removing intermediate container dae341962153
 ---> e7698cc4957a
Step 5/10 : RUN yum -y install vim
 ---> Running in 6c4117ae1e7e
# 下载步骤省略
Step 6/10 : RUN yum -y install net-tools
 ---> Running in 0704354faae6
# 下载步骤省略
Step 7/10 : EXPOSE 80
 ---> Running in 735a3461e291
Removing intermediate container 735a3461e291
 ---> d54ea501d5b6
Step 8/10 : CMD echo $MYPATH
 ---> Running in 1a2a29489a46
Removing intermediate container 1a2a29489a46
 ---> 3bdd7dd8ddc1
Step 9/10 : CMD echo "----end----"
 ---> Running in 7901e6eaf11e
Removing intermediate container 7901e6eaf11e
 ---> 7ef395b2cd22
Step 10/10 : CMD /bin/bash
 ---> Running in d09cd8c7fd32
Removing intermediate container d09cd8c7fd32
 ---> 6bfc0da2bbe0
Successfully built 6bfc0da2bbe0
Successfully tagged mycentos:0.1
[root@pd dockerfile]# 

# 3. 测试发现工作目录变了、vim ifconfig命令已支持
[root@pd dockerfile]# docker run -it --name mycentos0.1 mycentos:0.1
[root@8efda705692b local]# pwd
/usr/local
[root@8efda705692b local]# vim test.java
[root@8efda705692b local]# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.4  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:04  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

`列出镜像本地进行的变更历史 docker history 镜像id` 

```shell
[root@pd dockerfile]# docker history mycentos:0.1 
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
6bfc0da2bbe0        9 minutes ago       /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "/bin…   0B                  
7ef395b2cd22        9 minutes ago       /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "echo…   0B                  
3bdd7dd8ddc1        9 minutes ago       /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "echo…   0B                  
d54ea501d5b6        9 minutes ago       /bin/sh -c #(nop)  EXPOSE 80                    0B                  
ebad7b4118b8        9 minutes ago       /bin/sh -c yum -y install net-tools             22.8MB              
25e12652e8b2        9 minutes ago       /bin/sh -c yum -y install vim                   57.2MB              
e7698cc4957a        9 minutes ago       /bin/sh -c #(nop) WORKDIR /usr/local            0B                  
e855b87fe764        9 minutes ago       /bin/sh -c #(nop)  ENV MYPATH=/usr/local        0B                  
39b62d8401e7        9 minutes ago       /bin/sh -c #(nop)  MAINTAINER pd<123456@qq.c…   0B                  
0d120b6ccaa8        2 weeks ago         /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
<missing>           2 weeks ago         /bin/sh -c #(nop)  LABEL org.label-schema.sc…   0B                  
<missing>           2 weeks ago         /bin/sh -c #(nop) ADD file:538afc0c5c964ce0d…   215MB               
```

> CMD 和 ENTRYPOINT 区别

`使用CMD`

```shell
[root@pd dockerfile]# vim mydockerfile-cmd
[root@pd dockerfile]# cat mydockerfile-cmd 
FROM centos
CMD ["ls","-a"]
[root@pd dockerfile]# docker build -f mydockerfile-cmd -t cmd-test .
Sending build context to Docker daemon  4.096kB
Step 1/2 : FROM centos
 ---> 0d120b6ccaa8
Step 2/2 : CMD ["ls","-a"]
 ---> Running in 7666f8e95710
Removing intermediate container 7666f8e95710
 ---> f6c20d1bc450
Successfully built f6c20d1bc450
Successfully tagged cmd-test:latest
[root@pd dockerfile]# docker images
REPOSITORY    TAG      IMAGE ID      CREATED         SIZE
cmd-test     latest   f6c20d1bc450  24 seconds ago   215MB
[root@pd dockerfile]# docker run f6c20d1bc450
.
..
.dockerenv
bin
dev
etc
home
lib
lib64
lost+found
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var

# 想追加一个命令-l  组成ls -al
# CMD的情况下 替换了CMD ["ls","-a"] 命令为 CMD ["-l"] 这个不是命令所以报错
[root@pd dockerfile]# docker run f6c20d1bc450 -l
docker: Error response from daemon: OCI runtime create failed: container_linux.go:349: starting container process caused "exec: \"-l\": executable file not found in $PATH": unknown.

# 这种方式就可以了替换了CMD ["ls","-a"] 命令为 CMD ["ls",-al"]
[root@pd dockerfile]# docker run f6c20d1bc450 ls -al
total 56
drwxr-xr-x  1 root root 4096 Aug 31 06:12 .
drwxr-xr-x  1 root root 4096 Aug 31 06:12 ..
-rwxr-xr-x  1 root root    0 Aug 31 06:12 .dockerenv
lrwxrwxrwx  1 root root    7 May 11  2019 bin -> usr/bin
drwxr-xr-x  5 root root  340 Aug 31 06:12 dev
drwxr-xr-x  1 root root 4096 Aug 31 06:12 etc
drwxr-xr-x  2 root root 4096 May 11  2019 home
lrwxrwxrwx  1 root root    7 May 11  2019 lib -> usr/lib
lrwxrwxrwx  1 root root    9 May 11  2019 lib64 -> usr/lib64
drwx------  2 root root 4096 Aug  9 21:40 lost+found
drwxr-xr-x  2 root root 4096 May 11  2019 media
drwxr-xr-x  2 root root 4096 May 11  2019 mnt
drwxr-xr-x  2 root root 4096 May 11  2019 opt
dr-xr-xr-x 92 root root    0 Aug 31 06:12 proc
dr-xr-x---  2 root root 4096 Aug  9 21:40 root
drwxr-xr-x 11 root root 4096 Aug  9 21:40 run
lrwxrwxrwx  1 root root    8 May 11  2019 sbin -> usr/sbin
drwxr-xr-x  2 root root 4096 May 11  2019 srv
dr-xr-xr-x 13 root root    0 Aug 26 06:59 sys
drwxrwxrwt  7 root root 4096 Aug  9 21:40 tmp
drwxr-xr-x 12 root root 4096 Aug  9 21:40 usr
drwxr-xr-x 20 root root 4096 Aug  9 21:40 var
```

`使用ENTRYPOINT`

```shell
[root@pd dockerfile]# vim mydockerfile-entrypoint
[root@pd dockerfile]# cat mydockerfile-entrypoint 
FROM centos
ENTRYPOINT ["ls","-a"]
[root@pd dockerfile]# docker build -f mydockerfile-entrypoint -t entrypoint-test .
Sending build context to Docker daemon   5.12kB
Step 1/2 : FROM centos
 ---> 0d120b6ccaa8
Step 2/2 : ENTRYPOINT ["ls","-a"]
 ---> Running in bbc82d9569da
Removing intermediate container bbc82d9569da
 ---> 35e68f7d4170
Successfully built 35e68f7d4170
Successfully tagged entrypoint-test:latest
[root@pd dockerfile]# docker run 35e68f7d4170
.
..
.dockerenv
bin
dev
etc
home
lib
lib64
lost+found
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var

# 在命令后面追加
# 追加ENTRYPOINT ["ls","-a"] 变为 ENTRYPOINT ["ls","-al"]
[root@pd dockerfile]# docker run 35e68f7d4170 -l
total 56
drwxr-xr-x  1 root root 4096 Aug 31 06:15 .
drwxr-xr-x  1 root root 4096 Aug 31 06:15 ..
-rwxr-xr-x  1 root root    0 Aug 31 06:15 .dockerenv
lrwxrwxrwx  1 root root    7 May 11  2019 bin -> usr/bin
drwxr-xr-x  5 root root  340 Aug 31 06:15 dev
drwxr-xr-x  1 root root 4096 Aug 31 06:15 etc
drwxr-xr-x  2 root root 4096 May 11  2019 home
lrwxrwxrwx  1 root root    7 May 11  2019 lib -> usr/lib
lrwxrwxrwx  1 root root    9 May 11  2019 lib64 -> usr/lib64
drwx------  2 root root 4096 Aug  9 21:40 lost+found
drwxr-xr-x  2 root root 4096 May 11  2019 media
drwxr-xr-x  2 root root 4096 May 11  2019 mnt
drwxr-xr-x  2 root root 4096 May 11  2019 opt
dr-xr-xr-x 93 root root    0 Aug 31 06:15 proc
dr-xr-x---  2 root root 4096 Aug  9 21:40 root
drwxr-xr-x 11 root root 4096 Aug  9 21:40 run
lrwxrwxrwx  1 root root    8 May 11  2019 sbin -> usr/sbin
drwxr-xr-x  2 root root 4096 May 11  2019 srv
dr-xr-xr-x 13 root root    0 Aug 26 06:59 sys
drwxrwxrwt  7 root root 4096 Aug  9 21:40 tmp
drwxr-xr-x 12 root root 4096 Aug  9 21:40 usr
drwxr-xr-x 20 root root 4096 Aug  9 21:40 var
```

### 实战：Tomcat镜像

> 1. 准备镜像文件 tomcat和jdk压缩包

```shell
[root@pd tomcat]# pwd
/home/tomcat
[root@pd tomcat]# ll
总用量 187948
-rw-r--r-- 1 root root  11211292 8月  31 15:09 apache-tomcat-9.0.37.tar.gz
-rw-r--r-- 1 root root 181238643 8月  31 15:09 jdk-8u60-linux-x64.tar.gz
-rw-r--r-- 1 root root         0 8月  31 15:17 readme.txt
```

> 2. 编写Dockerfile文件，docker build命令默认去找`Dockerfile`这个文件，不需要-f指定

```shell
[root@pd tomcat]# cat Dockerfile 
FROM centos
MAINTAINER pd<8619@qq.com>

COPY readme.txt /usr/local/read.txt

# 会自动解压放到 /usr/local 目录下
ADD jdk-8u60-linux-x64.tar.gz /usr/local
ADD apache-tomcat-9.0.37.tar.gz /usr/local

RUN yum -y install vim

ENV MYPATH /usr/local
WORKDIR $MYPATH

ENV JAVA_HOME /usr/local/jdk1.8.0_60 
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.37
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/bin

EXPOSE 8080

CMD /usr/local/apache-tomcat-9.0.37/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.37/logs/catalina.out
```

> 3. 构建镜像

```shell
[root@pd tomcat]# docker build -t diytomcat .
Sending build context to Docker daemon  192.5MB
Step 1/13 : FROM centos
 ---> 0d120b6ccaa8
Step 2/13 : MAINTAINER pd<8619@qq.com>
 ---> Running in af228ad4a45a
Removing intermediate container af228ad4a45a
 ---> f94eca7e79e9
Step 3/13 : COPY readme.txt /usr/local/read.txt
 ---> b92d2bbde816
Step 4/13 : ADD jdk-8u60-linux-x64.tar.gz /usr/local
 ---> 170c62941e04
Step 5/13 : ADD apache-tomcat-9.0.37.tar.gz /usr/local
 ---> 89a3e262c554
Step 6/13 : RUN yum -y install vim
 ---> Running in c80f40f33907
# 省略下载步骤
 ---> 501208724b53
Step 7/13 : ENV MYPATH /usr/local
 ---> Running in 39b4b22a2bb4
Removing intermediate container 39b4b22a2bb4
 ---> 27edb32e1585
Step 8/13 : WORKDIR $MYPATH
 ---> Running in 9b16c66b4709
Removing intermediate container 9b16c66b4709
 ---> 475eab807ee6
Step 9/13 : ENV JAVA_HOME /usr/local/jdk1.8.0_60
 ---> Running in a737e7c2784d
Removing intermediate container a737e7c2784d
 ---> 7ce114813db3
Step 10/13 : ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.37
 ---> Running in d3a6bee4aaa4
Removing intermediate container d3a6bee4aaa4
 ---> 8d8e4ff5082f
Step 11/13 : ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/bin
 ---> Running in 15ad794a198a
Removing intermediate container 15ad794a198a
 ---> 1c023c4f5dac
Step 12/13 : EXPOSE 8080
 ---> Running in 8cb694e33165
Removing intermediate container 8cb694e33165
 ---> 5b0702afaeff
Step 13/13 : CMD /usr/local/apache-tomcat-9.0.37/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.37/logs/catalina.out
 ---> Running in 7b3f0de98e85
Removing intermediate container 7b3f0de98e85
 ---> d0fedfae6ccb
Successfully built d0fedfae6ccb
Successfully tagged diytomcat:latest
```

>4. 访问镜像

```shell
[root@pd tomcat]# docker run -d -p 9090:8080 --name pdtomcat -v /home/tomcat/test:/usr/local/apache-tomcat-9.0.37/webapps/test -v /home/tomcat/tomcatlogs/:/usr/local/apache-tomcat-9.0.37/logs diytomcat
624222e0610c8ba828a852c176598c6746f4473994d72a7e52c6b88d57e0022d

# 在外部挂载 test 和tomcatlogs目录
[root@pd tomcat]# ls -l
总用量 187956
-rw-r--r-- 1 root root  11211292 8月  31 15:09 apache-tomcat-9.0.37.tar.gz
-rw-r--r-- 1 root root       533 8月  31 15:52 Dockerfile
-rw-r--r-- 1 root root 181238643 8月  31 15:09 jdk-8u60-linux-x64.tar.gz
-rw-r--r-- 1 root root         0 8月  31 15:17 readme.txt
drwxr-xr-x 2 root root      4096 8月  31 16:04 test
drwxr-xr-x 2 root root      4096 8月  31 16:04 tomcatlogs

[root@pd home]# docker exec -it 624222 /bin/bash
[root@624222e0610c local]# pwd         
/usr/local
# 容器内tomcat和jdk已解压到/usr/local下
[root@624222e0610c local]# ls -l
total 60
drwxr-xr-x 3 root root 4096 Aug 31 08:04 aegis
drwxr-xr-x 1 root root 4096 Aug 31 07:52 apache-tomcat-9.0.37
drwxr-xr-x 2 root root 4096 May 11  2019 bin
drwxr-xr-x 2 root root 4096 May 11  2019 etc
drwxr-xr-x 2 root root 4096 May 11  2019 games
drwxr-xr-x 2 root root 4096 May 11  2019 include
drwxr-xr-x 8   10  143 4096 Aug  4  2015 jdk1.8.0_60
drwxr-xr-x 2 root root 4096 May 11  2019 lib
drwxr-xr-x 2 root root 4096 May 11  2019 lib64
drwxr-xr-x 2 root root 4096 May 11  2019 libexec
-rw-r--r-- 1 root root    0 Aug 31 07:17 read.txt
drwxr-xr-x 2 root root 4096 May 11  2019 sbin
drwxr-xr-x 5 root root 4096 Aug  9 21:40 share
drwxr-xr-x 2 root root 4096 May 11  2019 src

```

> 5. 访问测试

```shell
# 打开浏览器 输入 47.98.221.77:9090 进行测试
```

> 6. 发布项目（由于做了卷挂载，我们直接在本地编写项目就可以发布了！）

```shell
[root@pd test]# pwd
/home/tomcat/test
[root@pd test]# mkdir WEB_INF
[root@pd test]# cat index.jsp
<%@ page language="java" contentType="text/html; charset=utf-8"
    pageEncoding="utf-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>首页</title>
</head>
<body>
您好，
 <% System.out.println("----test tomcat log----"); %>
</body>
</html>

[root@pd test]# cd WEB-INF/
[root@pd WEB-INF]# cat web.xml 
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="WebApp_ID" version="3.0">
  <welcome-file-list>
    <welcome-file>index.html</welcome-file>
    <welcome-file>index.htm</welcome-file>
    <welcome-file>index.jsp</welcome-file>
  </welcome-file-list>
</web-app>
```

> 7. 访问新发布的项目

```shell
# 47.98.221.77:9090/test
# 查看日志
[root@pd tomcatlogs]# cat catalina.out 
31-Aug-2020 08:41:33.788 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server version name:   Apache Tomcat/9.0.37
31-Aug-2020 08:41:33.790 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server built:          Jun 30 2020 20:09:49 UTC
31-Aug-2020 08:41:33.790 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server version number: 9.0.37.0
31-Aug-2020 08:41:33.790 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log OS Name:               Linux
31-Aug-2020 08:41:33.790 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log OS Version:            3.10.0-862.14.4.el7.x86_64
31-Aug-2020 08:41:33.790 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Architecture:          amd64
31-Aug-2020 08:41:33.790 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Java Home:             /usr/local/jdk1.8.0_60/jre
31-Aug-2020 08:41:33.791 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log JVM Version:           1.8.0_60-b27
31-Aug-2020 08:41:33.791 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log JVM Vendor:            Oracle Corporation
31-Aug-2020 08:41:33.791 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log CATALINA_BASE:         /usr/local/apache-tomcat-9.0.37
31-Aug-2020 08:41:33.791 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log CATALINA_HOME:         /usr/local/apache-tomcat-9.0.37
31-Aug-2020 08:41:33.799 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.util.logging.config.file=/usr/local/apache-tomcat-9.0.37/conf/logging.properties
31-Aug-2020 08:41:33.799 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager
31-Aug-2020 08:41:33.799 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djdk.tls.ephemeralDHKeySize=2048
31-Aug-2020 08:41:33.799 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.protocol.handler.pkgs=org.apache.catalina.webresources
31-Aug-2020 08:41:33.799 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dorg.apache.catalina.security.SecurityListener.UMASK=0027
31-Aug-2020 08:41:33.799 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dignore.endorsed.dirs=
31-Aug-2020 08:41:33.799 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dcatalina.base=/usr/local/apache-tomcat-9.0.37
31-Aug-2020 08:41:33.799 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dcatalina.home=/usr/local/apache-tomcat-9.0.37
31-Aug-2020 08:41:33.799 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.io.tmpdir=/usr/local/apache-tomcat-9.0.37/temp
31-Aug-2020 08:41:33.800 INFO [main] org.apache.catalina.core.AprLifecycleListener.lifecycleEvent The Apache Tomcat Native library which allows using OpenSSL was not found on the java.library.path: [/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib]
31-Aug-2020 08:41:34.691 INFO [main] org.apache.coyote.AbstractProtocol.init Initializing ProtocolHandler ["http-nio-8080"]
31-Aug-2020 08:41:34.741 INFO [main] org.apache.catalina.startup.Catalina.load Server initialization in [1292] milliseconds
31-Aug-2020 08:41:34.822 INFO [main] org.apache.catalina.core.StandardService.startInternal Starting service [Catalina]
31-Aug-2020 08:41:34.823 INFO [main] org.apache.catalina.core.StandardEngine.startInternal Starting Servlet engine: [Apache Tomcat/9.0.37]
31-Aug-2020 08:41:34.845 INFO [main] org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory [/usr/local/apache-tomcat-9.0.37/webapps/ROOT]
31-Aug-2020 08:41:35.466 INFO [main] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory [/usr/local/apache-tomcat-9.0.37/webapps/ROOT] has finished in [621] ms
31-Aug-2020 08:41:35.466 INFO [main] org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory [/usr/local/apache-tomcat-9.0.37/webapps/manager]
31-Aug-2020 08:41:35.556 INFO [main] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory [/usr/local/apache-tomcat-9.0.37/webapps/manager] has finished in [90] ms
31-Aug-2020 08:41:35.556 INFO [main] org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory [/usr/local/apache-tomcat-9.0.37/webapps/examples]
31-Aug-2020 08:41:36.210 INFO [main] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory [/usr/local/apache-tomcat-9.0.37/webapps/examples] has finished in [654] ms
31-Aug-2020 08:41:36.210 INFO [main] org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory [/usr/local/apache-tomcat-9.0.37/webapps/docs]
31-Aug-2020 08:41:36.254 INFO [main] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory [/usr/local/apache-tomcat-9.0.37/webapps/docs] has finished in [44] ms
31-Aug-2020 08:41:36.255 INFO [main] org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory [/usr/local/apache-tomcat-9.0.37/webapps/host-manager]
31-Aug-2020 08:41:36.293 INFO [main] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory [/usr/local/apache-tomcat-9.0.37/webapps/host-manager] has finished in [38] ms
31-Aug-2020 08:41:36.293 INFO [main] org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory [/usr/local/apache-tomcat-9.0.37/webapps/test]
31-Aug-2020 08:41:36.348 INFO [main] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory [/usr/local/apache-tomcat-9.0.37/webapps/test] has finished in [55] ms
31-Aug-2020 08:41:36.357 INFO [main] org.apache.coyote.AbstractProtocol.start Starting ProtocolHandler ["http-nio-8080"]
31-Aug-2020 08:41:36.427 INFO [main] org.apache.catalina.startup.Catalina.start Server startup in [1685] milliseconds
31-Aug-2020 08:48:16.453 INFO [Catalina-utility-2] org.apache.catalina.startup.HostConfig.reload Reloading context [/test]
31-Aug-2020 08:48:16.453 INFO [Catalina-utility-2] org.apache.catalina.core.StandardContext.reload Reloading Context with name [/test] has started
31-Aug-2020 08:48:16.497 INFO [Catalina-utility-2] org.apache.catalina.core.StandardContext.reload Reloading Context with name [/test] is completed
----test tomcat log----
----test tomcat log----
```

### 发布自己的镜像

> DockerHub

```shell
[root@pd tomcatlogs]# docker login --help

Usage:	docker login [OPTIONS] [SERVER]

Log in to a Docker registry.
If no server is specified, the default is defined by the daemon.

Options:
  -p, --password string   Password
      --password-stdin    Take the password from stdin
  -u, --username string   Username
```

> docker push

```shell
# 打个tag
[root@pd tomcatlogs]# docker tag diytomcat sots/diytomcat:1.0
# 推送到DockerHub
[root@pd ~]# docker push sots/diytomcat:1.0
```

### 小结

```shell
# 打包镜像
docker save -o 路径名称 镜像id
# 加载镜像
docker load -i 加载镜像路径
```

## SpringBoot打包Docker镜像

```shell
FROM java:8

COPY *.jar /app.jar

CMD ["--server.port=8080"]

EXPOSE 8080

ENTRYPOINT ["java","-jar","/app.jar"]
```

## Docker Compose

### 简介

Docker Compose 来轻松高效的管理容器。定义运行多个容器。

> 官方介绍

Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration. To learn more about all the features of Compose, see [the list of features](https://docs.docker.com/compose/#features).

Compose works in all environments: production, staging, development, testing, as well as CI workflows. You can learn more about each case in [Common Use Cases](https://docs.docker.com/compose/#common-use-cases).

Using Compose is basically a three-step process:

1. Define your app’s environment with a `Dockerfile` so it can be reproduced anywhere.
2. Define the services that make up your app in `docker-compose.yml` so they can be run together in an isolated environment.
3. Run `docker-compose up` and Compose starts and runs your entire app.

**Compose 是 Docker 官方的开源项目，需要安装**

> docker-compose.yml example

```yaml
version: '2.0'
services:
  web:
    build: .
    ports:
    - "5000:5000"
    volumes:
    - .:/code
    - logvolume01:/var/log
    links:
    - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```

### 安装Compose

> 1. 下载Compose

```shell
# 官方提供 速度很慢
sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# 外部来源 
curl -L https://get.daocloud.io/docker/compose/releases/download/1.26.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```

> 2. 授权

```shell
sudo chmod +x /usr/local/bin/docker-compose

[root@pd bin]# chmod +x /usr/local/bin/docker-compose
[root@pd bin]# docker-compose version
docker-compose version 1.26.2, build eefe0d31
docker-py version: 4.2.2
CPython version: 3.7.7
OpenSSL version: OpenSSL 1.1.0l  10 Sep 2019
```

### 体验

以前都是单个docker run启动容器

现在通过docker-compose编写yaml配置文件，可以通过compose 一键启动所有服务。

**Docker小结：**

1. Docker镜像 run => 容器
2. DockerFile构建镜像 （服务打包）
3. docker-compose启动项目（编排、多个微服务/环境）
4. Docker 网络



**yaml规则**

docker-compose.yaml 核心

```yaml
# 3层
# 第一层：版本
version: ''  
# 第二层：服务
services:
# 服务1
	web:
		images:
		build:
		network:
	# 服务2
	redis:
# 第三层：其他配置 网络/卷、全局规则 
volumes:
networks:
configs:
```

官方文档链接：https://docs.docker.com/compose/compose-file/

> depends_on:

```yaml
# 启动的时候 先启动db服务再启动redis最后启动web服务
version: "3.8"
services:
  web:
    build: .
    # web 服务依赖于db服务呵redis服务
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

### 实战

1. 编写微服务项目

2. Dockerfile构建镜像

   ```dockerfile
   FROM java:8
   
   COPY *.jar /app.jar
   
   CMD ["--server.port=8080"]
   
   EXPOSE 8080
   
   ENTRYPOINT ["java","-jar","/app.jar"]
   ```

3. docker-compose.yml 编排项目

   ```yaml
   version: '3.8'
   services:
     pdapp:
       # 表示当前路径
       build: .
       image: pdapp
       depends_on:
         - redis
       ports:
         - "8080:8080"
     redis:
       image: "redis:alpine"
   ```

4. 上传服务器 jar包、Dockerfile、docker-compose.yml 文件

5. docker-compose up

6. 如果项目要重新部署打包 docker-compose up --build