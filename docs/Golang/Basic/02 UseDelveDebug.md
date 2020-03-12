# 使用delve调试Golang

## 安装
1. vscode中, 通过`ctrl+shift+p`运行`Go: Install/Update Tools`,选择`dlv`安装
2. `go get -u github.com/go-delve/delve/cmd/dlv`

## 使用
### vscode
1. 在vscode中按F5, 弹出`launch.json`文件, 配置
```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Launch",
            "type": "go",
            "request": "launch",
            "mode": "debug", // auto
            "program": "${fileDirname}",
            "env": {
                "GOPATH": "d:/workspace/go space/go"
            },
            "args": []
        }
    ]
}
```
### 命令行debug
```go
package main


import (
	"net/http"
	"github.com/gin-gonic/gin"
)

func HelloHandler(c *gin.Context) {
	firstname := c.DefaultQuery("firstname", "Guest")
	lastname := c.Query("lastname")
	c.String(http.StatusOK, "Hello %s %s", firstname, lastname)
}

func main() {
	router := gin.Default()

	router.GET("/welcome", HelloHandler)
	router.Run(":8000")
}
```
1. 进入要debug的文件目录`cd ../main.go`
```shell
dlv debug main.go // debug
b HelloHandler // break point
c // continue
n // next
s // step in
p value // print value
stepout // step out function
```


```flow
st=>start: Start:>http://www.google.com[blank]
e=>end:>http://www.google.com
op1=>operation: My Operation
sub1=>subroutine: My Subroutine
cond=>condition: Yes
or No?:>http://www.google.com
io=>inputoutput: catch something...
st->op1->cond
cond(yes)->io->e
cond(no)->sub1(right)->op1
```

```sequence
Title: Here is a title
A->B: Normal line
B-->C: Dashed line
C->>D: Open arrow
D-->>A: Dashed open arrow
```