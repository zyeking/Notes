# 使用Docker搭建Go开发环境

## 基本使用

1. 拉取Golang镜像`docker pull golang`

2. 运行容器`docker run --rm -it -name go-demo golang bash`进入到bash中

   + 设置goproxy`go env -w GOPROXY=https://goproxy.io,direct`
   + 设置gomodule`export GO111MODULE=on`

   ![image-20200508161732265](https://i.loli.net/2020/05/08/okJGuSX3mNDOARP.png)

3. 在容器中运行go项目

```bash
docker run --rm -it --name go-demo \
 -v $PWD: /go/src/example/go-demo \ 
 -p 8000:8000
 golang
```

通过`-v`将本地目录/数据挂载到容器中，`-p`指定主机和容器的端口映射。

切换到bash会话中运行`go run go/src/example/go-demo`



## 使用docker-compose

在项目代码跟目录创建`docker-compose.yml`

```yaml
version: '3'
services:
  app:
    image: golang:latest
    volumes:
      # 将项目代码根目录映射到容器中的相关目录
      - $PWD: /go/src/example/demo
    ports:
      - "8000:8000"
    # 执行go run
    command: go run /go/src/example/demo/main.go
```

+ 启动docker-compose并且：`docker-compose up -d`
+ 修改代码，重新编译项目：`docker-compose restart`
+ 进入容器中执行命令：`docker-compose exec <container name> bash`，例如`docker-compose exec app go test`



