## Nginx 环境搭建

### 下载

官网下载地址：http://nginx.org/en/download.html

下载tar.gz包 上传到服务器上

### 安装

安装前先检查一些环境是否存在

1. gcc 编译器是否安装

   ```shell
   # 检查是否安装
   yum list installed | grep gcc
   # 执行安装
   yum install -y gcc
   ```

2. openssl 库是否安装

   ```shell
   # 检查是否安装
   yum list installed | grep openssl
   # 执行安装
   yum install -y openssl-devel
   ```

3. pcre 库是否安装

   ```shell
   # 检查是否安装
   yum list installed | grep pcre
   # 执行安装
   yum install -y pcre-devel
   ```

4. zlib 库是否安装

   ```shell
   # 检查是否安装
   yum list installed | grep zlib
   # 执行安装
   yum install -y zlib-level
   ```

5. 一次性安装

   ```shell
   yum install -y gcc openssl openssl-devel pcre pcre-devel zlib zlib-devel
   ```

**正式安装Nginx**

- 解压上传的nginx文件 
- 进入解压后的nginx主目录
- 执行命令： ./configure --prefix=/usr/local/nginx 
- (其中 --prefix 是指定 nginx 的安装路径) 注意：等号左右不要有空格
- 执行命令进行编译：make
- 执行命令进行安装：make install
- 安装成功后，切换到/usr/local/nginx 目录下，查看内容

```shell
[root@pd nginx]# pwd
/usr/local/nginx
[root@pd nginx]# ll
总用量 16
drwxr-xr-x 2 root root 4096 9月  10 10:19 conf
drwxr-xr-x 2 root root 4096 9月  10 10:19 html
drwxr-xr-x 2 root root 4096 9月  10 10:19 logs
drwxr-xr-x 2 root root 4096 9月  10 10:19 sbin
```

### 启动

#### 普通启动

切换到nginx安装目录的sbin目录下，执行： ./nginx



#### 通过配置文件启动

./nginx -c /usr/local/nginx/conf/nginx.conf

/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf

其中 -c 是指定配置文件，而且配置文件路径必须是绝对路径



#### 检查 Nginx 是否启动

通过查看进程： p s -ef  | grep nginx

```shell
[root@pd sbin]# ps -ef | grep nginx
root      5414     1  0 9月01 ?       00:00:00 nginx: master process nginx
nginx     5415  5414  0 9月01 ?       00:00:00 nginx: worker process
root     22307 19597  0 10:33 pts/0    00:00:00 grep --color=auto nginx
```

nginx 体系结构由 master 进程和 worker 进程组成

master 进程读取配置文件，并维护 worker 进程，而 worker 进程则对请求进行实际处理

### 关闭

#### 优雅关闭 Nginx

查找 nginx 的进程号： ps -ef | grep nginx

执行命令： kill -quit 主 pid

```shell
[root@pd sbin]# ps -ef | grep nginx
root      5414     1  0 9月01 ?       00:00:00 nginx: master process nginx
nginx     5415  5414  0 9月01 ?       00:00:00 nginx: worker process
root     22351 19597  0 10:51 pts/0    00:00:00 grep --color=auto nginx
[root@pd sbin]# kill -QUIT 5414
```

注意：

- 其中 pid 是主进程号的 pid （master process） ，其他为子进程 pid (worker process)
- 这种关闭方式会处理完所有请求后再关闭，所以称之为优雅的关闭

#### 快速关闭 Nginx

查找 nginx 的进程号： ps -ef | grep nginx

执行命令： kill -term 主 pid  或者 kill 主 pid

```shell
[root@pd sbin]# ps -ef | grep nginx
root      5414     1  0 9月01 ?       00:00:00 nginx: master process nginx
nginx     5415  5414  0 9月01 ?       00:00:00 nginx: worker process
root     22351 19597  0 10:51 pts/0    00:00:00 grep --color=auto nginx
[root@pd sbin]# kill -TERM 5414
```

```shell
[root@pd sbin]# ps -ef | grep nginx
root      5414     1  0 9月01 ?       00:00:00 nginx: master process nginx
nginx     5415  5414  0 9月01 ?       00:00:00 nginx: worker process
root     22351 19597  0 10:51 pts/0    00:00:00 grep --color=auto nginx
[root@pd sbin]# kill 5414
```

#### 重启 Nginx

```shell
[root@pd sbin]# pwd
/usr/local/nginx/sbin
[root@pd sbin]# ./nginx -s reload
```

### 配置检查

当修改 nginx 配置文件后，可以使用 Nginx 命令进行配置文件语法检查

```shell
[root@pd usr]# /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
```

### 查看版本号

```shell
# 查看 nginx 的版本
[root@pd usr]# /usr/local/nginx/sbin/nginx -v
nginx version: nginx/1.18.0
# 查看 nginx 的版本、编译器版本和配置参数
[root@pd usr]# /usr/local/nginx/sbin/nginx -V
nginx version: nginx/1.18.0
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC) 
configure arguments: --prefix=/usr/local/nginx
```

```shell
[root@pd sbin]# pwd
/usr/local/nginx/sbin
[root@pd sbin]# ./nginx -v
nginx version: nginx/1.18.0
[root@pd sbin]# ./nginx -V
nginx version: nginx/1.18.0
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC) 
configure arguments: --prefix=/usr/local/nginx
```

