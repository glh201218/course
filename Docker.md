# Docker

## 介绍

Docker 是一个开源的应用容器引擎，基于Go 语言并遵从Apache2.0协议开源。
Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。

## 安装(Linux Centos)

```shll
$ sudo yum install -y yum-utils \
    device-mapper-persistent-data \
    lvm2
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

$ yum-config-manager --enable docker-ce-nightly

$ yum-config-manager --enable docker-ce-test

$yum-config-manager --disable docker-ce-nightly

$ yum install docker-ce docker-ce-cli containerd.io
如果报错执行下面的
wget https://download.docker.com/linux/centos/7/x86_64/edge/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
yum install -y  containerd.io-1.2.6-3.3.el7.x86_64.rpm
再执行
$ yum install docker-ce docker-ce-cli containerd.io
```

## 启动

```shll
systemctl start docker
```

## 测试

```shll
docker run hello-world
```

## 设置国内镜像

1. 修改配置文件
    1. `vim /etc/docker/daemon.json`
    2. 添加
    {
        "registry-mirrors": ["https://c8sf7ro9.mirror.aliyuncs.com"]
    }
    3. systemctl daemon-reload
    4. systemctl restart docker
2. 查看是否成功
`ps -ef| grep docker`

## 常用命令

1. `docker info` : docker信息  
2. `docker version`:docker版本信息  
3. `docker --help` : 命令帮助  
4. `docker images` : 列出本地主机上的镜像  
    ### 显示说明
    1. REPOSITORY : 表示镜像的仓库源
    2. TAG : 镜像的标签(版本)
    3. IMAGE ID : 镜像ID
    4. CREATED : 镜像创建时间
    5. SIZE : 镜像大小
    ### 参数选项
    1. -a : 列出本地所有的镜像(含中间映射层)
    2. -q : 只显示镜像id
    3. --digests : 显示镜像的摘要信息
    4. --no-trunc : 显示完整的镜像信息
5. `docker search 镜像名` : 从docker hup上查找关于这个镜像名的所有镜像
    ### 显示说明
    1. NAME : 镜像名称
    2. DESCRIPTION : 镜像说明
    3. STARS : 下载数,点赞数
    4. OFFICIAL : 官方版
    5. AUTOMATED : 自动构建
    ### 参数选项
    1. -s 1000: 搜索点赞数大于 1000的
    2. --no-trunc : 显示完整的镜像信息
    3. --automated : 只列出automated build类型的镜像
6. `docker pull 镜像名称[:TAG]` : 下载镜像
7. `docker rmi 镜像名称[:TAG] ...` : 删除本地镜像
    ### 参数选项
    1. -f : 强制删除
    2. -f $(docker images -qa) : 删除全部
8. `docker run [OPTIONS] IMAGE [COMMAND] [ARG...]` : 新建并启动容器
    ### 参数选项(OPTIONS)
    1. --name="容器新名字" : 为容器指定一个名称
    2. -d : 后台运行容器,并返回容器id,也即启动守护式容器
    3. -i : 以交互模式运行容器,通常与 -t 同时使用
    4. -t : 为容器重新分配一个伪输入终端,通常与 -i 同时使用
    5. -P : 随机端口映射(大写P)
    6. -p 指定端口映射,有以下四种格式(小写p)
        1. ip:hostPort:containerPort
        2. ip::containerPort
        3. hostPort:containerPort
        4. containerPort
9. `docker ps` : 显示docker中运行的容器
    ### 显示说明
    1. CONTAINER ID : 容器id
    2. IMAGE : 镜像id
    3. COMMAND : 以什么形式进入
    4. CREATED : 创建时间
    5. STATUS : 容器状态
    6. PORTS : 端口号
    7. NAMES : 容器名称
    ### 参数选项
    1. -a : 列出当前所有正在运行的容器+历史上运行过的
    2. -l : 显示最近创建的容器
    3. -n : 显示最近n个创建的容器
    4. -q : 静默模式,只显示容器编号
    5. --no-trunc : 不截断输出
10. `exit` : 容器停止退出
11. `ctrl+P+Q` : 容器不停止退出
12. `docker start 容器id或容器名称` : 启动容器
13. `docker restart 容器id或容器名称` : 重启容器
14. `docker stop 容器id或容器名称` : 停止容器
15. `docker kill 容器id或容器名称` : 强制停止容器
16. `docker rm 容器id或容器名称` : 删除容器
17. `docker rm -f 容器id或容器名称` : 强制删除容器
18. `docker rm -f $(docker ps -aq)` : 强制删除多个容器
19. `docker ps -aq | xargs docker rm` : 删除多个已经停止的容器
20. `docker logs 容器id` : 查看容器日志
    ### 参数选项
    1. -t : 加入时间戳
    2. -f : 跟随最新的日志打印
    3. --tail 数字 : 显示最后多少条
21. `docker top 容器id` : 查看容器内的进程
22. `docker inspect 容器id` : 查看容器内部细节
23. `docker attach 容器id` : 进入容器并以命令行交互
24. `docker exec -t 容器id 执行的命令` : 不进入容器执行容器内的命令  
    1. `docker exec -it 容器id /bin/bash`: 进入命令提示行
25. `docker cp 容器id:路径 拷贝到宿主机的位置` : 容器内的文件复制到宿主机上
26. `docker commit -m="描述信息" -a="作者" 容器id 要创建的目标镜像名:[标签名]` : 提交容器副本使之成为一个新的镜像

## 容器数据卷
### 作用 : `容器停止和关闭后的数据持久化`
### 容器添加数据卷
* 命令添加  
    `docker run -it -v /宿主机的绝对目录路径:/容器内目录 镜像名` : 容器内文件夹和宿主机文件夹关联  
    `docker run -it -v /宿主机的绝对目录路径:/容器内目录:ro 镜像名` : 容器内文件夹和宿主机文件夹关联,容器内目录只读,主机可写可读
* DockerFile添加  
    1. 在根目录新建目录 : mydocker
    2. 在新建目录中添加文件 : dockerfile
    3. 编辑dockerfile:
    ```txt
        FROM centos
        VOLUME ["dataVolumenContainer1","/dataVolumeContainer2"]
        CMD echo "finished,---------success1"
        CMD /bin/bash
    ```
    3. 构建新的镜像 : docker build -f /mydocker/Dockerfile -t gglh/centos .
    4. 查看 : docker images
    5. 启动 : docker run -it gglh/centos
    6. 进入目录并添加文件 :   
        cd dataVolumenContainer1
        touch containt01.txt
    7. 查看宿主机对应目录 :  
        1. 新建命令提示符
        2. 查看容器配置文件 : docker inspect 容器id
        3. 配置文件中有对应目录  
#### `*注` : Docker挂载主机目录Docker访问出现cannot open directory ...Permission denied.解决方法:在挂载目录后多加一个--privileged=true即可

## 数据卷容器
`容器之间互通数据`  
1. docker run -it --name dc01 gglh/centos
2. docker run -it --name dc02 --volumes-from dc01 gglh/centos
## Dockerfile解析

### 执行流程:
1. docker 从基础镜像运行一个容器
2. 执行一行指令并对容器做出修改
3. 执行类似docker commit的操作提交一个新的镜像层
4. docker 再基于刚提交的镜像运行一个新容器
5. 执行dockerfile中的下一条指令直到所有指令都执行完成
### 体系结构:  
* `FROM` : 基础镜像,当期新镜像是基于那个镜像的
* `MAINTAINER` : 镜像维护者的名字和邮箱地址
* `RUN` : 容器构建时需要运行的命令
* `EXPOSE` : 当前容器对外暴露的端口
* `WORKDIR` : 指定在创建容器后,终端默认登录的进来工作目录,一个落脚点
* `ENV` : 用来在构建镜像过程中设置环境变量
* `ADD` : 将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和解压tar压缩包
* `COPY` : 类似ADD,拷贝文件和目录到镜像中,将从构建上下文目录中<源路径>的文件/目录复制到新的一层的镜像内的<目标路径>位置
* `VOLUME` : 容器数据卷,用于数据保存和持久化
* `CMD` : 指定一个容器启动时要运行的命令,dockerfile中可以有多个cmd指令,但只有最后一个生效,cmd会被docker run之后的参数替换
* `ENTRYPOINT` : 指定一个容器启动时要运行的命令,ENTRYPOINT的目的和cmd一样,都是在指定容器启动程序及参数
* `ONBUILD` : 当构建一个被继承的dockerfile时运行命令,父镜像在被子继承后父镜像的onbuild被触发
## 案例
`Base镜像(scratch):Docker Hub中99%的镜像都是通过在Base镜像中安装和配置需要的软件构建出来的`  
修改centos,让它变成支持vim和ifconfig命令

1. 编写dockfile
```txt
FROM centos
ENV mypath /tmp
WORKDIR $mypath
RUN yum -y install vim 
RUN yum -y install net-tools 
EXPOSE 80
CMD /bin/bash
```
2. 构建  
`docker build -f dockerfile文件路径 -t 新镜像名字:tag  保存路径`
3. 测试  
`docker run -it  新镜像名字:tag`
4. 列出镜像变更历史  
`docker history 镜像名`
## 案例2
```txt
FROM centos
RUN yum -y install curl
ENTRYPOINT ["curl","-s","http://ip.cn"]
ONBUILD RUN echo "father onbuild-----886"
```
2. 构建  
`docker build -f dockerfile文件路径 -t 新镜像名字:tag  保存路径`
3. dockerfile
```txt
FROM 新镜像名字:tag
RUN yum -y install curl
ENTRYPOINT ["curl","-s","http://ip.cn"]
```
2. 构建  
`docker build -f dockerfile文件路径 -t 新镜像名字:tag  保存路径`

## 常用安装
