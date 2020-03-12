# Docker 02：Dockerfile中的ENTRYPOINT、RUN与CMD

## CMD与ENRTYPOINT
+ RUN
+ CMD
    + 每个`Dockerfile`中只能有一个`CMD`, 如果有多个那么只会执行最后一个.
    + `CMD`相当于启动`docker`时候后面添加的参数:`docker run -itd --name aaa docker_image /bin/bash -c.` 
        + 镜像名称后面跟了`/bin/bash -c`等价于在dockerfile中的`CMD ["/bin/bash", "-c"]`
        + 若执行的时候添加了参数,默认的`CMD`中的参数则无效
+ ENTRYPOINT
    + 类似`CMD`, 每个`Dockerfile`只能有一个`ENTRYPOINT`, 如果存在多个只执行最后一个.
    + 必定执行, 不会动态改变
    
## 书写格式
+ Shell格式：<instruction> <command>, 例如: apt-get install python3
+ Exec <instruction> ["executable", "param1", "param2", ...], 例如:  ["apt-get", "install", "python3"]

## CMD命令
`CMD echo "hello world"`
1. 运行`docker run -it [image]`, 输出hello world
2. 运行`docker run -it [image]/bin/bash`, `CMD`命令会被忽略, 命令`bash`会被执行

## ENTRYPOINT
`ENTRYPOINT ["/bin/echo", "Hello"]`
1. 运行`docker run -it [image]`, 输出Hello
2. 运行`docker run -it [image] kk`, 输出Hello kk, 原本的Hello仍然会输出

修改`Dockerfile`为
```
ENTRYPOINT ["/bin/echo", "Hello"]
CMD ["world"]
```
1. 运行`docker run -it [name]`, 输出Hello world
2. 运行`docker run -it [name] king`, 输出Hello king, `CMD`参数被动态替换.
## RUN命令
```dockerfile
RUN apt-get update && apt-get install -y \  
 bzr \
 cvs \
 git \
 mercurial \
 subversion
```
1. `apt-get update` 和 `apt-get install`放在同一个`RUN`指令执行可以==保证每次安装的都是最新的包==; 如果将`apt-get install`放在单独的的`RUN`中执行, 则会使用`apt-get update`创建的镜像层, 这层镜像可能是很久以前缓存的.

## 总结
+ `CMD`设置容器启动后==默认执行的命令以及参数==, 设置的指令可以被`docker run`命令后面的命令函参数==动态替换==
+ `ENTRYPOINT`配置容器启动时的执行命令, ==不会被忽略, 一定会被执行==, 即使运行了`docker run`时指定了其他命令
+ 使用 RUN 指令安装应用和软件包，构建镜像。
+ 如果 Docker 镜像的用途是运行应用程序或服务, 比如运行一个 MySQL, 应该优先使用 Exec 格式的 ENTRYPOINT 指令.==CMD 可为 ENTRYPOINT 提供额外的默认参数, 同时可利用 docker run 命令行替换默认参数.==
+ 如果想为容器设置默认的启动命令,可使用 CMD 指令.用户可在 docker run 命令行中替换此默认命令.



# REF:
[Dockerfile RUN，CMD，ENTRYPOINT命令区别](https://www.jianshu.com/p/f0a0f6a43907)

[Dockerfile中ENTRYPOINT 和 CMD的区别](https://www.cnblogs.com/Presley-lpc/articles/9230271.html?from=singlemessage)