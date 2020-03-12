# 流程图
## 请求流程图
1. `URL`请求打到`gin`，由`gin`分发各个请求到相应的接口
2. 接口对应`GET`，`POST`，`PUT`，`DELETE`等多种请求方式
3. 接口接收`URL`中的参数，构造相应的CRUD查询，查询数据库
4. 将数据库返回的数据返回到前端调用者

![](https://i.loli.net/2019/12/20/fBkyUHOq6Q4ALTS.png)

## JWT产生以及认证
> 生成JWT

![](https://i.loli.net/2019/12/20/bgslWo2iP18CcYv.png)

> JWT认证

![](https://i.loli.net/2019/12/20/6V2WLd7hcFUpKXC.png)