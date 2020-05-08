# [Docker-Compose快速入门](https://www.runoob.com/docker/docker-compose.html)



## 什么是Docker Compose?

> 用于定义和运行多容器Docker应用程序的工具，通过Compose使用YAML文件配置应用程序需要的所有服务。实现使用一个命令就能够从YAML文件配置中创建并启动所有服务。



## 使用步骤

1. 使用Dockerfile定义应用程序环境
2. 在`docker-compose.yml`中搭建应用程序所需的服务
3. 执行`docker-compose up`运行整个应用程序



## 示例

```python
# composetest/app.py
import time

import redis
from flask import Flask

app = Flask(__name__)
# 通过redis缓存
cache = redis.Redis(host='redis', port=6379)


def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)


@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```

创建`composetest/requirements.txt`文件指定库

```plaintext
flask
redis
```

创建`composetest/Dockerfile`文件定义`app.py`的环境

```dockerfile
# 从python3.7 alpine镜像开始构建
FROM python:3.7-alpine

# 设置工作目录
WORKDIR /code 
# 设置flask命令使用的环境变量
ENV FLASK_APP app.py
ENV FLASK_RUN_HOST 0.0.0.0
# 安装gcc 
RUN apk add --no-cache gcc musl-dev linux-headers
# 复制requirements.txt并且安装依赖
COPY requirements.txt requirements.txt
# 执行pip
RUN pip install -r requirements.txt
# 将当前项目复制到镜像中的工作目录
COPY . . 
# 设置容器默认的执行命令
CMD [ "flask", "run" ]
```

创建`composetest/docker-compose.yml`文件指定应用程序所需的服务

```yaml
# yaml 配置
# 指本yml已从的compose版本
version: '3'
services:
  web:
    # 构建镜像的上下文路径: ./Dockerfile
    build: .
    # 对外映射的端口
    ports:
     - "5000:5000"
  redis:
    image: "redis:alpine"

# 定义了两个service,分别是web和redis
# web 服务使用从Dockerfile 当前目录中构建的镜像
```

使用Compose命令运行`docker-compose up -d`

![](https://i.loli.net/2020/05/08/MLIDkNvBcaAzb7T.png)

## yml配置

```yaml
version: "3.7"
services:
  webapp:
    build:
      context: ./dir
      dockerfile: Dockerfile-alternate
      args:
        buildno: 1
      labels:
        - "com.example.description=Accounting webapp"
        - "com.example.department=Finance"
        - "com.example.label-with-empty-value"
      target: prod
      container_name: my-web-container
      command: ["bundle", "exec", "thin", "-p", "3000"]
```

+ context：指定Dockerfile的上下文路径
+ dockerfile：指定构建镜像的Dockerfile文件名
+ labels：设置构建镜像的标签
+ container_name：设置容器名称
+ command：覆盖容器启动的默认指令



依赖关系

```yaml
version: "3.7"
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

+ docker-compose up：根据依赖关系启动服务，先启动db，redis再启动web
+ docker-compose stop：根据依赖关系廷式服务，先关闭web，再关闭db和redis
+ docker-compose up SERVICE：自动包含SERVICE的依赖项，通过`docker-compose up`还将创建并启动db，redis



挂载 

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



环境变量

> 布尔值需要用引号，确保YML解析器不会将其转换为True/False

```yaml
environment:
  RACK_ENV: development
  SHOW: 'true'
```

