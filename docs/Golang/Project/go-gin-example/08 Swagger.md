# go-gin-example 08： Swagger

## Swagger是什么?
接口生成器：自动生成接口文档

## 如何在golang中使用Swagger
### 安装
1. 安装swag`go get -u github.com/swaggo/swag/cmd/swag`，如果没有将`$GOROOT/bin`添加到`$PATH`中，则需要将swag的可执行文件移到`$GOBIN`中
2. 验证是否安装成功
```bash
$ swag -v 
swag version v1.6.3
```
3. 安装gin-swagger
```bash
go get -u github.com/swaggo/gin-swagger
go get -u github.com/swaggo/gin-swagger/swaggerFiles
```

### 编写swag
```golang

// Response 定义swag文档返回的结构体
type Response struct {
	Code int         `json:"code"`
	Msg  string      `json:"msg"`
	Data interface{} `json:"data"`
}

// GetArticles 获取多个文章
// @Summary 获取多个文章
// @Produce  json
// @Param tag_id body int false "TagID"
// @Param state body int false "State"
// @Param created_by body int false "CreatedBy"
// @Success 200 {object} Response
// @Failure 500 {object} Response
// @Router /api/v1/articles [get]
func GetArticles(c *gin.Context) {
	data := make(map[string]interface{})
	maps := make(map[string]interface{})
	valid := validation.Validation{}

	var state = -1
	if arg := c.Query("state"); arg != "" {
		state = com.StrTo(arg).MustInt()
		maps["state"] = state

		valid.Range(state, 0, 1, "state").Message("状态只允许0或1")
	}

	var tagID = -1
	if arg := c.Query("tag_id"); arg != "" {
		tagID = com.StrTo(arg).MustInt()
		maps["tag_id"] = tagID

		valid.Min(tagID, 1, "tag_id").Message("标签ID必须大于0")
	}

	code := e.INVALID_PARAMS
	if !valid.HasErrors() {
		data["list"] = models.GetArticles(util.GetPage(c), setting.PageSize, maps)
		data["total"] = models.GetTagTotal(maps)

		code = e.SUCCESS
	} else {
		for _, err := range valid.Errors {
			log.Printf("err.key:%s, err.message:%s", err.Key, err.Message)
		}
	}
	c.JSON(http.StatusOK, gin.H{
		"code": code,
		"msg":  e.GetMsg(code),
		"data": data,
	})
}
```
格式
> // @Summary API描述                
// @Produce json[生成..内容]                       
// @Param id path/body/query int[type] false/true "ID"[程序中对应变量名字]              
// @Success 200 string "ok" --成功返回信息           
// @Failure 500 string "bad" --失败返回信息         
// @Routers api/v1/article/{id} [GET]  请求id, 请求方法

## 效果
访问`http://127.0.0.1:8000/swagger/index.html`
![](https://i.loli.net/2019/11/08/u9lDgpAZWCzFrG2.jpg)
![](https://i.loli.net/2019/11/08/fmUxIsPzriJuBL3.jpg)
![](https://i.loli.net/2019/11/08/AVpbfm84DhPIYzX.jpg)