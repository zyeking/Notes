# 03：进阶模板用法

## 在模板中定义变量

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"text/template"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		tmpl, err := template.New("test").Parse(`
		{{$name := "Alice"}}
		{{$age := 18}}
		{{$round2 := true}}
		Name: {{$name}}
		Age: {{$age}}
		Round2: {{$round2}}
		`)
		if err != nil {
			fmt.Fprintf(w, "Parse: %v", err)
			return
		}

		err = tmpl.Execute(w, nil)
		if err != nil {
			fmt.Fprintf(w, "Execute: %v", err)
			return
		}
	})
	log.Println("starting HTTP serve...")
	log.Fatal(http.ListenAndServe(":8000", nil))
}
```
![](https://i.loli.net/2019/11/14/KR3SbYcathd2wAl.jpg)

1. 用美元符号`$`作为前缀表示变`$name`,`$age`
2. 变量的定义/赋值必须使用`:=`语法
3. 直接通过`{{$VarName}}`调用
4. 所有变量的操作都属于模板语法的一部分,需要用`{{}}`括起来

### 修改变量的值
```go
    ...
	tmpl, err := template.New("test").Parse(`
		{{$name := "Alice"}}
		{{$age := 18}}
		{{$round2 := true}}
		Name: {{$name}}
		Age: {{$age}}
		Round2: {{$round2}}
		{{$name = "K"}}
		Name: {{$name}}
		`)
	...
```
![](https://i.loli.net/2019/11/14/APR1blvDUkTWrYf.jpg)

1. 类似赋值, 用`=`号直接修改变量值


## 在模板中使用条件判断(if)
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
		tmpl, err := template.New("test").Parse(`
		{{if .yIsZero}}
			除数不能为0
		{{else}}
			{{.result}}
		{{end}}
		`)
		if err != nil {
			fmt.Fprintf(w, "Parse: %v", err)
			return
		}

		x, _ := strconv.ParseInt(r.URL.Query().Get("x"), 10, 64)
		y, _ := strconv.ParseInt(r.URL.Query().Get("y"), 10, 64)

		yIsZero := y == 0
		result := 0.0
		if !yIsZero {
			result = float64(x) / float64(y)
		}

		err = tmpl.Execute(w, map[string]interface{}{
			"yIsZero": yIsZero,
			"result":  result,
		})
		if err != nil {
			fmt.Fprintf(w, "Execute: %v", err)
		}
	})
	log.Println("starting HTTP serve...")
	log.Fatal(http.ListenAndServe(":8000", nil))
}
```
![](https://i.loli.net/2019/11/14/p2sIdU3nZ7ewN9A.jpg)

1. 用`{{}}`将`if`逻辑语句括起来, `if`后面必须返回一个`bool`值
2. `if`语句包括`{{if}}`, `{{else}}`,`{{end}}`


## 等式和不等式
> 1. `eq`：当等式 arg1 `==` arg2 成立时，返回 true，否则返回 false
> 2. `ne`：当不等式 arg1 `!=` arg2 成立时，返回 true，否则返回 false
> 3. `lt`：当不等式 arg1 `<` arg2 成立时，返回 true，否则返回 false
> 4. `le`：当不等式 arg1 `<=` arg2 成立时，返回 true，否则返回 false
> 5. `gt`：当不等式 arg1 `>` arg2 成立时，返回 true，否则返回 false
> 6. `ge`：当不等式 arg1 `>=` arg2 成立时，返回 true，否则返回 false

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"text/template"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		tmpl, err := template.New("test").Parse(`
			{{$name1 := "alice"}}
			{{$name2 := "bob"}}
			{{$age1 := 18}}
			{{$age2 := 23}}
			{{if eq $age1 $age2}}
				年龄相同
			{{else}}
				年龄不同
			{{end}}
			{{if ne $name1 $name2}}
    			名字不相同
			{{end}}
			`)
		if err != nil {
			fmt.Fprintf(w, "Parse: %v", err)
			return
		}
		err = tmpl.Execute(w, nil)
		if err != nil {
			fmt.Fprintf(w, "Execute: %v", err)
			return
		}
	})
	log.Println("Starting HTTP server...")
	log.Fatal(http.ListenAndServe("localhost:8000", nil))
}

```
![](https://i.loli.net/2019/11/14/kcseSZ9nXhjIf4N.jpg)

## 迭代操作(range)
> Go 语言中一般来说有三种类型可以进行迭代操作，数组（Array）、切片（Slice）和 map 类型

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"text/template"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		tmpl, err := template.New("test").Parse(`
			{{range $name := .Names}}
				{{$name}}
			{{end}}
		`)
		if err != nil {
			fmt.Fprintf(w, "Parse: %v", err)
			return
		}

		err = tmpl.Execute(w, map[string]interface{}{
			"Names": []string{
				"Alice",
				"Bob",
				"Carol",
				"David",
			},
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
![](https://i.loli.net/2019/11/14/vkhEKBrH1LbON8y.jpg)

### 获得迭代元素的索引
```go
...
func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        // 创建模板对象并解析模板内容
        tmpl, err := template.New("test").Parse(`
{{range $i, $name := .Names}}
    {{$i}}. {{$name}}
{{end}}
`)
...
```
### map类型
```go
...
func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        // 创建模板对象并解析模板内容
        tmpl, err := template.New("test").Parse(`
{{range $name, $val := .}}
    {{$name}}: {{$val}}
{{end}}
`)
        ...
        // 调用模板对象的渲染方法
        err = tmpl.Execute(w, map[string]interface{}{
            "Names": []string{
                "Alice",
                "Bob",
                "Carol",
                "David",
            },
            "Numbers": []int{1, 3, 5, 7},
        })
        ...
}
```

## with
```go
tmpl, err := template.New("test").Parse(`Inventory
	{{with .Inventory}}
		SKU: {{.SKU}}
		Name: {{.Name}}
		UnitPrice: {{.UnitPrice}}
		Quantity: {{.Quantity}}
	{{end}}
	`)
//****************************//
err = tmpl.Execute(w, map[string]interface{}{
        "Inventory": Inventory{ // 类型
            SKU:       "11000",
            Name:      "Phone",
            UnitPrice: 699.99,
            Quantity:  666,
        },
    })
```
1. 用`{{with}}`来替代相关实例"**Inventory**".


## 空白符号处理
> `{{-` 表示剔除模板内容`左侧`的所有空白符号, `-}}` 表示剔除模板内容`右侧`的所有空白符号.

```go
...
func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        // 创建模板对象并解析模板内容
        tmpl, err := template.New("test").Parse(`Inventory
{{- with .Inventory}}
    SKU: {{.SKU}}
    Name: {{.Name}}
    UnitPrice: {{.UnitPrice}}
    Quantity: {{.Quantity}}
{{- end}}
`)
...
```
