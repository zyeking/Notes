# go-gin-example 05：JWT

## 为什么要JWT

> 当前的API接口虽然编写好了，但是这些API可以被随意调用，这样子是不安全的。因此需要通过`jwt-go`的方式来解决



## 如何使用JWT

+ 下载依赖包：`go get -u github.com/dgrijalva/jwt-go`

+ 编写`./pkg/util`下的新建`jwt.go`文件

  + 其中该工具包主要包含`生成Token`、`解析Token`功能
  + NewWithClaims 中加密方法有`SigningMethodHS256`，`SigningMethodHS384`、`SigningMethodHS512`三种`cropto.Hash`方案


```go
package util

import (
	setting "go-gin-example/pkg/settings"
	"time"

	"github.com/dgrijalva/jwt-go"
)

var jwtSecret = []byte(setting.JwtSecret)

// Claims 声明Claims结构体
type Claims struct {
	Username string `json: "username"`
	Password string `json: "password"`
	jwt.StandardClaims
}

// GenerateToken 生成密钥
func GenerateToken(username, password string) (string, error) {
	nowTime := time.Now()
	expireTime := nowTime.Add(3 * time.Hour)

	claims := Claims{
		username,
		password,
		jwt.StandardClaims{
			ExpiresAt: expireTime.Unix(),
			Issuer:    "gin-blog",
		},
	}

	// 对claims 进行加密
	tokenClaims := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
	// 获得加密后的token 
	// 猜测为生成一个加密后的字符串给jwtSecret
	token, err := tokenClaims.SignedString(jwtSecret)

	return token, err
}

// ParseToken 解析密钥
func ParseToken(token string) (*Claims, error) {
	tokenClaims, err := jwt.ParseWithClaims(token, &Claims{}, func(token *jwt.Token) (interface{}, error) {
		return jwtSecret, nil
	})

	if tokenClaims != nil {
		if claims, ok := tokenClaims.Claims.(*Claims); ok && tokenClaims.Valid {
			return claims, nil
		}
	}
	return nil, err
}

```

+ 编写Gin中间件，在`middleware`下新建`jwt`目录，在其中新建`jwt.go`文件写入

```go
package jwt

import (
	"go-gin-example/pkg/e"
	"go-gin-example/pkg/util"
	"net/http"
	"time"

	"github.com/gin-gonic/gin"
)

// JWT json web token
func JWT() gin.HandlerFunc {
	return func(c *gin.Context) {
		var code int
		var data interface{}

		code = e.SUCCESS
		token := c.Query("token")

		if token == "" {
			code = e.INVALID_PARAMS
		} else {
			claims, err := util.ParseToken(token)
			if err != nil {
				code = e.ERROR_AUTH_CHECK_TOKEN_FAIL
			} else if time.Now().Unix() > claims.ExpiresAt {
				code = e.ERROR_AUTH_CHECK_TOKEN_TIMEOUT
			}
		}
		if code != e.SUCCESS {
			c.JSON(http.StatusUnauthorized, gin.H{
				"code": code,
				"msg":  e.GetMsg(code),
				"data": data,
			})
			// 通过Abort确保当前的handler未被调用, 即当验证不正确的时候丢弃掉该处理
			c.Abort()
			return
		}
		c.Next()
	}
}

```

+ 获得token，在`./models`下新建`auth.go`文件写入

```go
package models

// Auth Token认证
type Auth struct {
	ID       int     `gorm:"primary_key" json:"id"`
	Username  string  `json:"username"`
	Password string  `json:"password"`
}

// CheckAuth 验证token
func CheckAuth(username, password string) bool {
	var auth Auth
	db.Select("id").Where(Auth{Username: username, Password: password}).First(&auth)
	if auth.ID > 0 {
		return true
	}
	return false
}

```

在`./api`下新建`auth.go`文件写入

```go
package api

import (
	"go-gin-example/models"
	"go-gin-example/pkg/e"
	"go-gin-example/pkg/util"
	"log"
	"net/http"

	"github.com/astaxie/beego/validation"
	"github.com/gin-gonic/gin"
)

type auth struct {
	Username string `valid: "Required; MaxSize(50)" `
	Password string `valid: "Required; MaxSize(50)"`
}

// GetAuth 获取认证
func GetAuth(c *gin.Context) {
	username := c.Query("username")
	password := c.Query("password")

	valid := validation.Validation{}
	a := auth{Username: username, Password: password}
	ok, _ := valid.Valid(&a)

	data := make(map[string]interface{})
	code := e.INVALID_PARAMS
	if ok {
		isExist := models.CheckAuth(username, password)
		// 如果存在该账号
		if isExist {
			token, err := util.GenerateToken(username, password)
			if err != nil {
				code = e.ERROR_AUTH_TOKEN
			} else {
				data["token"] = token
				code = e.SUCCESS
			}
		} else {
			// 不存在账号
			code = e.ERROR_AUTH
		}
	} else {
		for _, err := range valid.Errors {
			log.Println(err.Key, err.Message)
		}
	}
	c.JSON(http.StatusOK, gin.H{
		"code": code,
		"msg":  e.GetMsg(code),
		"data": data,
	})
}

```

+ 修改`router.go`文件

```GO
package routers

import (
	"go-gin-example/middleware/jwt"
	setting "go-gin-example/pkg/settings"
	"go-gin-example/routers/api"
	v1 "go-gin-example/routers/api/v1"

	"github.com/gin-gonic/gin"
)

// InitRouter 初始化路由
func InitRouter() *gin.Engine {
	r := gin.New()

	r.Use(gin.Logger())
	r.Use(gin.Recovery())

	gin.SetMode(setting.RunMode)
	
            // 调用认证接口 
	r.GET("/auth", api.GetAuth)

	apiv1 := r.Group("/api/v1")
           // Group内的所有请求都需要经过JWT
	apiv1.Use(jwt.JWT())
	{
		apiv1.GET("/tags", v1.GetTags)
		apiv1.POST("/tags", v1.AddTag)
		apiv1.PUT("/tags/:id", v1.EditTag)
		apiv1.DELETE("/tags/:id", v1.DeleteTag)

		apiv1.GET("/articles", v1.GetArticles)
		apiv1.GET("/articles/:id", v1.GetArticle)
		apiv1.POST("/articles", v1.AddArticle)
		apiv1.PUT("/articles/:id", v1.EditTag)
		apiv1.DELETE("/atricles/:id", v1.DeleteTag)
	}
	return r
}

```

+ 调用

  访问` http://127.0.0.1:8000/auth?username=test&password=test123456 `获得token

  ![](https://i.loli.net/2019/10/23/aAQPfD4XqC7yFuS.jpg)

  带着token访问api` http://127.0.0.1:8000/api/v1/articles?token=eyJhbGci... `

  ![](https://i.loli.net/2019/10/23/af5SjuGsxnOTrpe.jpg)





