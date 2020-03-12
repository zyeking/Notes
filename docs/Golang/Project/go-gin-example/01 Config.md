# go-gin-example 01：配置

## Golang 环境安装配置

### 下载

### 配置环境变量

## 文件配置

```shell
$ mkdir go-gin-example && cd go-gin-example
$ go env -w GO111MODULE=on
$ go env -w GOPROXY=https://goproxy.cn,direct
$ go mod init [MODULE_PATH]
$ ls
> go.mod
```

+ `mkdir go-gin-example && cd go-gin-example`：创建并切换到项目目录。
+ `go env -w GO111MUDOLE=on`：打开Go module 开关。
+ `go env -w GOPROXY=...`：设置GOPROXY代理，第一个为七牛Go代理，`direct`为Go在拉取模块遇到错误会回源到原模块版本的源地址去抓取。
+ `go mod init [Module_Path]`：初始化Go module，产生go.mod文件

```shell
module go-gin-example

go 1.12
```

### GOMODULE基础使用

+ `go get`：拉取新的依赖
  + 拉取最新的版本：`go get golang/org/x/text@lastest`
  + 拉取`master`分支的最新commit：`go get golang.org/x/text@master`
+ `go tidy`：整理依