+ 实现静态资源服务（Static Resource）
+ 支持HTML模板渲染


+ 后端专注数据生成，RESTful接口返回结构化数值；前端专注界面设计实现，只需要考虑拿到数据后如何渲染的问题，前后端解耦。


todo: 通过`filepath`的相对地址映射到`/usr/web`目录下的真实地址，将真实地址交给`net/http`的`http.FileServer`处理
