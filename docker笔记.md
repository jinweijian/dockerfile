一切说明在官网命令参数详解的对比下，显得黯然失色：

https://docs.docker.com/reference/

# docker 安装

来源于官网 https://docs.docker.com/engine/install/centos/

1. yum 安装gcc相关环境

   ```shell
   yum -y install gcc
   yum -y install gcc-c++
   ```

2. 卸载旧版本

   ```shell
   yum remove docker \
                     docker-client \
                     docker-client-latest \
                     docker-common \
                     docker-latest \
                     docker-latest-logrotate \
                     docker-logrotate \
                     docker-engine
   ```

   在首次在新的主机上安装 Docker 引擎之前，您需要设置 Docker 存储库。之后，您可以从存储库安装和更新 Docker。` --官网

3. 安装所需的软件包

   ```shell
   yum install -y yum-utils
   ```

4. 设置镜像仓库

   ```shell
   yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
   ```

5. 更新yum软件包索引

   ```shell
   yum makecache fast
   ```

6. 安装 Docker CE

   ```shell
   yum -y install docker-ce docker-ce-cli containerd.io
   ```

7. 启动Docker

   ```shell
   systemctl start docker
   ```

8. 通过运行图像验证 Docker 引擎是否正确安装

   ```shell
   docker search mysql
   # 可能查找不到
   docker run hello-world
   # 启动会很慢
   ```

9. 配置镜像加速地址

   ```shell
   vi /etc/docker/daemon.json
   # 填入以下参数
   {
     "registry-mirrors": ["http://hub-mirror.c.163.com", "https://docker.mirrors.ustc.edu.cn"]
   }
   ```

10. 重启docker 服务

    ```shell
    # 重新加载某个服务的配置文件
    sudo systemctl daemon-reload
    # 重新启动 docker
    sudo systemctl restart docker
    ```

# docker volume 数据卷

在我们正常使用 **docker**部署集群环境时，由于集群环境各不相同，所以如果在 **swarm** 模式下使用基础的挂载：***-v localDIR:ContainerDIR*** 的方式，我们会遇到一些很难管理的麻烦与错误；所以 **docker vloume** 命令就很有帮助了

命令：

```shell
docker volume --help
```

![image-20210317181900697](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\image-20210317181900697.png)

创建命令 **docker volume create [OPTIONS] [VOLUME] ** ：

![image-20210317182215934](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\image-20210317182215934.png)



# docker File

```dockerfile
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP app.py
ENV FLASK_RUN_HOST 0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
COPY . .
CMD ["flask","run"]

```



# docker安装php拓展

该节文章摘要于：https://www.cnblogs.com/yinguohai/p/11329273.html

手动安装php拓展时，我们需要去php.ini配置中启用此扩展。

为了部署的快捷性，php提供了docker 安装扩展的快捷命令：

- ==docker-php-source==
- ==docker-php-ext-install==
- ==docker-php-ext-enable==
- ==docker-php-ext-configure==

## docker-php-source

此命令，实际上就是在PHP容器中创建一个/usr/src/php的目录，里面放了一些自带的文件而已。我们就把它当作一个从互联网中下载下来的PHP扩展源码的存放目录即可。事实上，所有PHP扩展源码扩展存放的路径： /usr/src/php/ext 里面。

```dockerfile
docker-php-source extract | delete
# extract 初始化创建一个php扩展源码目录
# delete 删除php扩展源码目录
```

例：

![image-20210317104019748](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\image-20210317104019748.png)

不存在`/user/src/php`目录

使用命令初始化此目录：

```
docker-php-source extract
```

![image-20210317104657861](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\image-20210317104657861.png)



## docker-php-ext-enable

这个命令，就是用来**启动 PHP扩展** 的。我们使用pecl安装PHP扩展的时候，默认是没有启动这个扩展的，如果想要使用这个扩展必须要在php.ini这个配置文件中去配置一下才能使用这个PHP扩展。而 **docker-php-ext-enable** 这个命令则是自动给我们来启动PHP扩展的，不需要你去php.ini这个配置文件中去配置。

案例：

下载并安装**swoole**扩展 - 使用**pecl**命令

```shell
pecl install -o -f  swoole
```

![image-20210317114955125](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\image-20210317114955125.png)



查看当前已有的扩展 - **swoole** 但是没有被启动

![image-20210317115037237](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\image-20210317115037237.png)



我们可以使用这个命令一键启动该扩展

```
docker-php-ext-enable swoole
```

![image-20210317114734279](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\image-20210317114734279.png)

## docker-php-ext-install

该命令可以对PHP的扩展包一键安装并启动！

注意：源拓展包需要手动解压并放到 ==/usr/src/php/ext/== 下面（初次安装，需要初始化该文件夹）

格式：

在已存在`/usr/src/php/ext/php-redis` 这个拓展包存在的情况下，执行命令：

```shell
docker-php-ext-install php-redis 
# 后面的参数就是 /usr/src/php/ext/ 下扩展包名称
```

例：

安装php-redis拓展

```shell
# 下载扩展包
curl -L -o /tmp/redis-5.3.3.tgz  https://pecl.php.net/get/redis-5.3.3.tgz
# 解压拓展包
tar -xvf /tmp/redis-5.3.3.tgz
# 移动拓展包
mv /tmp/redis-5.3.3 /usr/src/php/ext/phpredis
# 安装并启用拓展
docker-php-ext-install phpredis

```

![image-20210317112249941](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\image-20210317112249941.png)

## docker-php-ext-configure

**docker-php-ext-configure** 一般都是需要跟 **docker-php-ext-install** 搭配使用的。它的作用就是，当你安装扩展的时候，需要自定义配置时，就可以使用它来帮你做到。

例：

```dockerfile
FROM php:7.1-fpm
RUN apt-get update \
	# 相关依赖必须手动安装
	&& apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng-dev \
    # 安装扩展
    && docker-php-ext-install -j$(nproc) iconv mcrypt \
    # 如果安装的扩展需要自定义配置时
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd
```



# docker-compose

## docker-compose安装

运行以下命令以下载 Docker Compose 的当前稳定版本：

```shell
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

将可执行权限应用于二进制文件：

```shell
 sudo chmod +x /usr/local/bin/docker-compose
```

创建软链：

```shell
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

测试是否安装成功：

```shell
docker-compose --version
```



## docker-compose说明

1、官方文档创建了一个应用 app.py

2、Dockerfile 应用打包为镜像

3、Docker-compose yaml文件 （定义整个服务器，需要的环境。web、redis）完整的上线服务

4、启动 compose 项目 （docker-compose up）

流程：

1. 创建网络 - compose_default
2. 执行 Docker-compose yaml
3. 启动服务。

Docker-compose yaml

Creating Docker-compose 文件中的service 中的镜像

例

```yaml
version: '3'	# 当前 compose 版本
services:		# 服务（容器）的列表 
	web:		# web服务（容器）
		build: .	# 使用当前文件下的Dockerfile文件来创建一个镜像	
		ports:		# 绑定端口号
			- "5000:5000"	# 本地:容器
	redis:		# redis服务（容器）
		image:"redis:alpine"	# 镜像来源 - 直接使用官方镜像

```



**docker-compose 默认规则**

- docker images 自动下载
- 默认的服务名： 文件名_服务名 _num
  - 多个服务器，集群中的服务编号
  - 集群状态中，运行多个服务容器的编号。弹性、高可用
- 

**docker-compost 网络规则**

![1615642915134](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615642915134.png)

10个服务 => 项目 （项目中的容器都在同个网络下 - 域名访问）

![1615643182317](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615643182317.png)

如果在同一个网络下，我们可以直接通过域名访问！ - 高可用！



**docker-compose 停止**

在存在docker-compose .yml 的文件夹下运行

```shell
docker-compose stop
```



**docker-compose  设置环境变量 .env 文件**

```shell
$ cat .env
TAG=v1.5	

$ cat docker-compose.yml
version: '3'
services:
  web:
    image: "webapp:${TAG}"
    
docker-compose up web webapp:v1.5
```

]

**Docker小结**

1. Docker 镜像。 run => 容器
2. Dockerfile 构建镜像（服务打包）
3. docker-compose 启动项目（编排、多个微服务环境）
4. Docker 网络



## yaml规则

docker-compose.yaml 核心

`https://docs.docker.com/compose/compose-file/compose-file-v3/`

```shell
# 共3层

# 第一层 - 版本
version: '' # compose 版本 ，向下兼容 见官网参考 Refarence
# 第二层 - 服务
service: # 服务
	服务1： 
		container_name: '' # 容器名称
		images : '' # 所选的镜像（可直接远端下载使用的）
		build .	# 可以使用当前文件下面的 Dockerfile 自动构建一个镜像（与上面二选一）
		network # 所选网络
		ports: 	# 绑定的端口
		depends_on: 	# 依赖启动 - 默认先启动这里面依赖的容器
		 	- 服务2
		 	- 服务N
		links :	# 网络互通的服务列表
			- 服务2 
			- 服务N
	服务2 ：
		....
# 第三层 - 其他配置：网络、挂载卷等一下其他全局规则

		
```



## 总结

项目 compose 三层

- 工程 Porject （项目代码）
- 服务 service
- 容器 运行实例！

# Docker Swarm



**概念**

![1615727566998](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615727566998.png)

**命令**

![1615727789770](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615727789770.png)



**==docker swarm init== 创建一个节点**

**docker swarm init --help** 

![1615727877546](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615727877546.png)

docker swarm 第一次使用时，需要指定一台服务器的地址作为主节点链接

```shell
docker swarm init --advertise-addr IP 
```

初始化：

![1615728147639](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615728147639.png)

执行了初始化命令==docker swarm init --advertise-addr IP==之后我们得到提示的一条命令：系统以及生产了一个token 可以用 ==docker swarm join-token manager== - 加入以IP为主机的docker swarm 集群中来

**docker swarm join 加入一个节点**

子服务器上面运行 init 之后返回的命令

```

```

![1615728719387](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615728719387.png)

提示加入成功！

查看主机上面节点的数量

```shell
docker node ls
```

![1615728816728](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615728816728.png)

以及成功加入

**主机上面获取token**

当我们初始化主机之后返回的命令没有保存，并且想让其他子机作为**worker-工作者**加入到swarm集群中作为子服务器时，我们可以使用命令 ==join-token worker==重新生成！

```shell
docker swarm join-token worker
```

![1615729089418](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615729089418.png)

当我们需要将另一台服务器作为 主服务器管理的时候，那我们可以使用命令 ==join-token manager==

```
docker swarm join-token manager
```

![1615729310151](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615729310151.png)

运行该命令的服务器，将作为管理者 **manager** 加入

![1615729341635](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615729341635.png)

此时：

![1615729457843](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615729457843.png)

服务器：VM-0-4就是管理者，VM-0-17是工作者

## Raft协议

集群可用：3个主节点至少要保证两台管理节点存活。才能保证服务正常使用！

当我们上面配置中的其中一台主节点关闭之后，整个swarm 集群将不可用！

2主1从的场景下，关闭其中一个主机：

管理者1关闭：

![1615730670774](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615730670774.png)

管理者2不可用：

![1615730699532](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615730699532.png)

工作机不可用：

![1615730723483](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615730723483.png)

重启 管理机1 后：

![1615730856671](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615730856671.png)

swarm服务全部恢复

**leader选举策略**

当3主双从状态下 **leader** 主机不可用之后，会在剩下的两个 **管理机 ** 中重新选举一个 **leader** 主机，当原 **leader** 主机恢复后，会重新加入到服务器集群中来，但是他的身份会降级变成 **管理者** **Reachable** 状态！！



## 体会

当前环境：

**leader:**  81.68.244.223 - VM-0-6-centos, 

**manager：**121.4.153.97 - VM-0-4-centos，

**work：** 121.4.151.24 - VM-0-17-centos

弹性、扩缩容！集群！

docker-compose up!跑动一个单机

集群：swarm 

**容器** 变成 **服务**

![1615776541011](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615776541011.png)

实验：

创建一个服务

```shell
docker service create -p 80:80 -d --name SERVICE_NAME IMAGES:TAG
# SERVICE_NAME => 服务名称
# IMAGES:TAG => 所选镜像
```

![1615778718342](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615778718342.png)

此时访问 **leader** 

![1615779120025](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615779120025.png)

访问 **manager**

![1615779155321](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615779155321.png)

访问 **work**

![1615779181472](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615779181472.png)

但是我们在 **work** 与 **manager** 服务器上面均找不到80端口提供的容器

**manager**

![1615779302437](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615779302437.png)

**work**

![1615779326278](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615779326278.png)

**结论**：在==docker swarm==集群中的任意一台管理主机上面创建一个服务的前提下，访问集群中任意一台的该服务端口，都会允许访问！（类似于负载均衡的转发）

我们查看服务的列表

```shell
docker service ls
```

![1615780400381](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615780400381.png)

发现一个很重要的概念：**副本：REPLICAS**

可以理解为：一个副本相当于一个容器

如上的例子中，我们服务默认只启动了一个副本在创建服务的机器 **leader** 中，其存在我们刚刚创建的服务容器！

在**Docker Service**中，**副本replicas**是可以动态扩容的，可以在有管理权限的主机中使用命令：

```shell
docker service update --replicas X SERVICE_NAME
# X => 副本的数量
# SERVICE_NAME => 集群服务的名称
```

![1615781023446](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615781023446.png)

此时，swarm 会平均在链接的节点创建 **服务容器**

此时：

**work**

![1615781158143](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615781158143.png)

**manager**

![1615781185080](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615781185080.png)

都自动创建了服务容器

动态扩容也可以使用命令==scale==：

```shell
docker service scale SERVICE_NAME=X
# SERVICE_NAME 服务名
# X 副本数量
```

单独就更换副本数量的功能上，该命令与update 一致

## 概念总结

**Swarm**

集群的管理和编号。docker可以初始化一个swarm集群，其他节点可以加入。（管理、工作者）

**Node**

就是一个docker节点。多个节点就组成了一个网络集群。（leader、manager、work）

**Service**

服务！可以在节点管理中创建或修改。用户访问！

**Task**

容器内的命令，细节任务！



![1615788774280](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615788774280.png)



**网络模式**

当我们创建了一个swarm 集群之后，我们的docker会新建一个网络：**ingress**

![1615789737311](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615789737311.png)

其使用特殊的 **Overlay** 网络实现子机之间网络互通，并且具有**负载均衡**的功能！

![1615789950201](C:\Users\MSI-NB\AppData\Roaming\Typora\typora-user-images\1615789950201.png)

# Docker Stack

docker-compose 单机部署项目！

Docker Stack  集群部分项目！

```shell
# 单机部署项目
docker-compose up -d 
# 集群部署项目
docker stack deploy 
```



部署命令：

```shell
docker stack deploy -c docker-compose.yml [--with-registry-auth YOUAUTH]
```

上述命令将会在集群上面依据**docker-compose.yml**文件来部署服务



# Docker Sercet



# Docker Config



# Docker 部分错误相关的处理

## docker push or login报错  `certificate is valid for....`

```shell
[root@VM-0-6-centos lnmp]# docker login
Authenticating with existing credentials...
Login did not succeed, error: Error response from daemon: Get https://registry-1.docker.io/v2/: x509: certificate is valid for *.execute-api.us-east-1.amazonaws.com, not registry-1.docker.io	# 报错！登录错误，无法解析！
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username (jinweijian): ^C
[root@VM-0-6-centos lnmp]# yum install bind-utils	# 安装dig
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
base                                                                                                                                                                 | 3.6 kB  00:00:00     
docker-ce-stable                                                                                                                                                     | 3.5 kB  00:00:00     
extras                                                                                                                                                               | 2.9 kB  00:00:00     
updates                                                                                                                                                              | 2.9 kB  00:00:00     
  File "/usr/libexec/urlgrabber-ext-down", line 28
    except OSError, e:
                  ^
SyntaxError: invalid syntax
  File "/usr/libexec/urlgrabber-ext-down", line 28
    except OSError, e:
                  ^
SyntaxError: invalid syntax


Exiting on user cancel
[root@VM-0-6-centos lnmp]# dig @114.114.114.114 registry-1.docker.io # 查看可以使用的解析地址

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-16.P2.el7_8.6 <<>> @114.114.114.114 registry-1.docker.io
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 8210
;; flags: qr rd ra; QUERY: 1, ANSWER: 8, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;registry-1.docker.io.		IN	A

;; ANSWER SECTION:
registry-1.docker.io.	31	IN	A	3.211.199.249	# 这个列表的IP都可以正确解析！
registry-1.docker.io.	31	IN	A	34.231.251.252
registry-1.docker.io.	31	IN	A	52.5.11.128
registry-1.docker.io.	31	IN	A	3.224.96.239
registry-1.docker.io.	31	IN	A	52.20.56.50
registry-1.docker.io.	31	IN	A	54.85.56.253
registry-1.docker.io.	31	IN	A	52.54.232.21
registry-1.docker.io.	31	IN	A	35.169.249.115

;; Query time: 7 msec
;; SERVER: 114.114.114.114#53(114.114.114.114)	
;; WHEN: Thu Mar 18 16:33:11 CST 2021
;; MSG SIZE  rcvd: 177
[root@VM-0-6-centos lnmp]# vim /etc/hosts	# 添加/修改正确的解析到服务器上
# 3.211.199.249    registry-1.docker.io



```



## docker stack deploy 报错：`image reference must be provided`

