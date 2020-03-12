# go-gin-example 09：Docker

## Docker是什么
> Docker is a set of platform as a service (PaaS) products that use OS-level virtualization to deliver software in packages called containers.[6] Containers are isolated from one another and bundle their own software, libraries and configuration files; they can communicate with each other through well-defined channels.[7] All containers are run by a single operating-system kernel and are thus more lightweight than virtual machines.[8]

Docker是Paas的产品, 通过被成为`容器`的`系统级别`的虚拟化交付软件.
## 如何使用Docker

### 安装Docker(WIN10 PRO)
1.  `右键WINDOWS` - `应用和功能` - `程序和功能` - `启用或关闭WINDOS功能` - `开启Hyper-V`
> 开启`Hyper-V`可能导致`ShadowSocksR`端口被占用
2. [官网](https://www.docker.com/get-started)
### Docker基本指令

### 编写Dockerfile
```shell
FROM golang:latest

ENV GOPROXY https://goproxy.cn,direct
WORKDIR $GOPATH/src/go-gin-example
COPY . $GOPATH/src/go-gin-example
RUN go build .

EXPOSE 8000
ENTRYPOINT ["./go-gin-example"]
```
1. `FROM`: 指定基础镜像, 该指令必须要有,且得为第一条
2. `WORKDIR`: 指定工作目录路径, 若目录不存在, 则会创建改目录
3. `COPY`: 源路径 ... 目标路径, `COPY`指令将`Dockerfile`文件所存在的上下文目录`复制`到目标路径位置
4. `RUN`: 执行命令
5. `EXPOSE`: 声明`运行时容器`提供服务端口, 仅仅是一个`声明`, 不会因为这个声明而开启这个端口
6. `ENTRYPOINT`: 指定`容器`启动程序的及参数(执行`./go-gin-example)`


### 运行
```
docker build -t gin-blog-docker .
docker images
docker run -p 8000:8000 gin-blog-docker
```
1. 在`.`当前环境运行`docker build`创建/构建镜像, `-t`指定名称
2. `docker images` 查看镜像是否创建成功
3. 在本地8000端口运行容器
4. 发现`dial tcp 127.0.0.1:3306: connect: connection refused`错误
### 配置Mysql Docker
```
docker pull mysql
docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -d mysql
```
1. 拉取mysql镜像
2. 配置mysql端口

#### 修改配置文件`conf.ini`
```ini
[database]
TYPE = mysql
USER = root
PASSWORD = rootroot
HOST = mysql:3306
NAME = blog
TABLE_PREFIX = blog_
```

### 关联Golang容器和Mysql容器
`docker run --link mysql:mysql -p 8000:8000 gin-blog-docker`
## 效果
