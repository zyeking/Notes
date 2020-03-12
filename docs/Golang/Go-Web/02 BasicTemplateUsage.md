# 02：基础模板

## 为什么要用模板
HTTP服务器响应固定的字符串不符合实际环境, 通过`text/template`包向客户端**响应动态内容**.

## 特性
> 1. 将模板应用于给定的数据结构来执行模板，模板的编码与 Go 语言源代码文件相同，需为 **UTF-8 编码**
> 2. 模板中的注解（Annotation）会根据数据结构中的元素来执行并派生具体的显示结构，这些元素一般指结构体中的字段或 map 中的键名
> 3. 模板的执行逻辑会依据点（Dot，"."）操作符来设定当前的执行位置，并按序完成所有逻辑的执行。
> 4. 模板中的行为（Action）包括数据评估（Data Evaluation）和控制逻辑，且需要使用双层大括号（{{ 和 }}）包裹。除行为以外的任何内容都会原样输出不做修改。
> 5. 模板解析完成后，从设计上可以并发地进行渲染，但要注意被渲染对象的并发安全性。例如，一个模板可以同时为多个客户端的响应进行渲染，因为输出对象（Writer）是相互独立的，但是被渲染的对象可能有各自的状态和时效性。

## 如何使用模板
### 实例
```go
package main

import (
	"fmt"
	"text/template"
	"log"
	"net/http"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		// 创建模板对象并解析模板内容
		tmpl, err := template.New("test").Parse("Hello world!")
		if err != nil {
			fmt.Fprintf(w, "Parse: %v", err)
			return
		}

		// 调用模板对象的渲染方法
		err = tmpl.Execute(w, nil)
		if err != nil {
			fmt.Fprintf(w, "Excute: %v", err)
			return
		}
	})

	log.Println("starting HTTP server...")
	log.Fatal(http.ListenAndServe(":8000", nil))
}
```
1. 引入`text/template`包.
2. 调用`template.New`方法根据给定的名称新建模板, 返回一个`*template.Template`对象.
3. `*template.Template`对象的`Parse`方法接受字符串参数(文本模板内容), 解析并返回解析中遇到的错误.
4. 调用`template.Execute`渲染模板, 参数分别为`输出对象`和`指定数据对象`, 实现了`io.Writer`接口的实例都可以作为输出对象.


### 渲染变量
```go
package main

import (
    "net/http"
    "log"
)

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Reqeust){
        w.Write([]byte(r.URL.QUERY().GET("val")))
    })
    log.Println("starting HTTP serve...")
    log.Fatal(http.ListenAndServe(":8000", nil))
}
```
![](https://i.loli.net/2019/11/13/uExhkinB1oFXIPG.jpg)

1. HTTP协议通过`GET`请求获取URL参数(URL中?后的值).
2. 调用`*http.Request`对象的`URL.QUERY().GET()`方法.
---
```go
package main

import (
	"fmt"
	"text/template"
	"log"
	"net/http"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		// 创建模板并解析内容
		tmpl, err := template.New("test").Parse("The value is: {{.}}")
		if err != nil {
			fmt.Fprintf(w, "Parse: %v", err)
			return
		}

		// 获取URL参数的值
		val := r.URL.Query().Get("val")

		// 调用模板对象渲染方法
		err = tmpl.Execute(w, val)
		if err != nil {
			fmt.Fprintf(w, "Execute: %v", err)
			return
		}
	})
	log.Println("starting HTTP server...")
	log.Fatal(http.ListenAndServe(":8000", nil))
}
```
![](https://i.loli.net/2019/11/13/weVXNBl7DfuEron.jpg)

1. 模板内容修改成`The value is : {{.}}`, 用了`分隔符`将`.`操作符包裹起来,`.`操作符默认指向`根对象`, 即`template.Execute`中的第二个参数.
2. 在`template.Execute`方法中传入`val`, `.`操作符渲染该变量`val`实现动态输出.

### 渲染复杂对象
```go
package main

import (
	"fmt"
	"text/template"
	"log"
	"net/http"
	"strconv"
)

// Inventory 库存
type Inventory struct {
	SKU       string
	Name      string
	UnitPrice float64
	Quantity  int64,
}

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		tmpl, err := template.New("test").Parse(`Inventory
		SKU: {{.SKU}}
		Name: {{.Name}}
		UnitPrice: {{.UnitPrice}}
		Quantity: {{.Quantity}}
		`)
		if err != nil {
			fmt.Fprintf(w, "Parse:%v", err)
			return
		}

		// 根据URL查询参数的值创建Inventory实例
		inventory := &Inventory{
			SKU:  r.URL.Query().Get("sku"),
			Name: r.URL.Query().Get("name"),
		}

		// 数据处理
		inventory.UnitPrice, _ = strconv.ParseFloat(r.URL.Query().Get("unitPrice"), 64)
		inventory.Quantity, _ = strconv.ParseInt(r.URL.Query().Get("quantity"), 10, 64)

		// 渲染
		err = tmpl.Execute(w, inventory)
		if err != nil {
			fmt.Fprintf(w, "Execute: %v", err)
			return
		}
	})
	log.Println("start HTTP serve...")
	log.Fatal(http.ListenAndServe(":8000", nil))
}
```
![](https://i.loli.net/2019/11/14/3iETWXcjaf5A7R6.jpg)

1. `template.Execute`的第二个参数类型为`interface{}`, 可以传入任何类型的参数.
```go
func (t *Template) Execute(wr io.Writer, data interface{}) error {
	if err := t.escape(); err != nil {
		return err
	}
	return t.text.Execute(wr, data)
}
```
2. `http/template`会根据传入的`根对象`进行底层类型分析, 自动识别变量, 此时的`.`操作符代表`inventory`结构体, 因此可以调用`inventory`的各个属性.
3. 在`Parse`的时候用反引号``将结构体包起来.
### 渲染中调用结构体的方法
```go
func (i *Inventory) Subtotal() float64 {
	return i.UnitPrice * float64(i.Quantity)
}
//******************************//
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		tmpl, err := template.New("test").Parse(`Inventory
		SKU: {{.SKU}}
		Name: {{.Name}}
		UnitPrice: {{.UnitPrice}}
		Quantity: {{.Quantity}}
		Subtotal: {{.Subtotal}}
		`)
```

### map类型作为模板跟对象
```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"strconv"
	"text/template"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		// 创建模板对象并解析模板内容
		tmpl, err := template.New("test").Parse(`Inventory
        SKU: {{.SKU}}
        Name: {{.Name}}
        UnitPrice: {{.UnitPrice}}
        Quantity: {{.Quantity}}
        `)
		if err != nil {
			fmt.Fprintf(w, "Parse: %v", err)
			return
		}

		// 直接将URL 查询参数的值赋值给变量
		sku := r.URL.Query().Get("sku")
		name := r.URL.Query().Get("name")
		unitPrice, _ := strconv.ParseFloat(r.URL.Query().Get("unitPrice"), 64)
		quantity, _ := strconv.ParseInt(r.URL.Query().Get("quantity"), 10, 64)

		err = tmpl.Execute(w, map[string]interface{}{
			"SKU":       sku,
			"Name":      name,
			"UnitPrice": unitPrice,
			"Quantity":  quantity,
		})
		if err != nil {
			fmt.Fprintf(w, "Execute: %v", err)
			return
		}
	})

	log.Println("Starting HTTP server...")
	log.Fatal(http.ListenAndServe("localhost:8000", nil))
}
```
1. 传递给`Execute`一个`map[string]interface{}`作为模板对象,可以传入任意类型的值, 将结构体的所有值都传入.
2. 不再需要单独创建实例, 只需要通过`r.URL.Query().Get(valName)`获取URL查询参数的值.
3. 其中数值需要用到`strconv.ParseInt/ParseFloat`进行转化

### 注释
```go
tmpl, err := template.New("test").Parse(`Inventory{{/* 打印参数的值 */}}
    SKU: {{.SKU}}
    Name: {{.Name}}
    UnitPrice: {{.UnitPrice}}
    Quantity: {{.Quantity}}
    `)
```
1. 通过`{{}}`双层大括号和`/**/`括起来的值是注释

## 模板流程
![](https://i.loli.net/2019/12/20/1XfnymCsSWxUaQh.png)

1. `template.New`新建模板, 返回`*template.Template`对象
2. 调用`*template.Template`对象的`Parse`方法解析模板
3. 传入模板实例
4. 数据处理`strconv`
5. 调用`*template.Template`对象的`Execute`方法渲染模板