# channel

## what is channel 
channel是goroutine和goroutine之间无锁交流的【媒介】，换句话说，channel是一种允许一个goroutine发送数据给另外一个goroutine的技术，默认的channel是双向的，意味着goroutines可以通过同一个channel发、收数据。

![1 _1_.jpg](https://i.loli.net/2019/12/18/MnwWGTqEVQeydzp.jpg)

## how
+ syntax
    + `var Channel_name chan Type`
    + `channel_name := make(chan Type)`

在golang中, 我们用关键字`Channel_name chan Type`创建某种类型的channel，其中channel仅仅可以传递同种类型的数据，不同类型的数据不可以通过一个类型的channel传递。

## Send and Receive Data From a Channel
在golang中，channel主要有两个操作，一个是`sending`，一个是`receiving`，这两种操作被称为通讯。`<-`箭头的方向表明是发送数据还是接受数据。在默认情况下，channel中的发送、接受操作块【直到另一端未就绪】
，允许goroutine之间在没有显式锁或者条件变量的时候互相同步

1. 发送操作：在channel的帮助下用来将goroutine的数据发送给另外一个goroutine，通过channel发送`float64`，`int`，`bool`是安全切容易的，因为这些值会被`拷贝`一份，不会有意外并发的风险，同样的，`strings`类型也是安全的，因为它是`immutable（不可更改）`。在发送`pointers`或者`slice`，`map`等引用的时候，channel是不安全的，因为指针或者引用会被发送者或者接收者在同一时间修改，结果是不可预知的，因此需要==确保只有一个goroutine能够访问==

`Mychannel <- element`表明了数据（element）发送给了Mychannel

2. 接受操作：`element := <- Mychannel`
```go
package main

import "fmt"

func main() {
	fmt.Println("starting Main method")
    
	ch := make(chan int)

	go myfunc(ch)
	ch <- 23
	fmt.Println("End Main method")
}

func myfunc(ch chan int) {
	fmt.Println(234 + <-ch)
}
```

## Closing a Channel

1. `close()`
这是一个内置函数，设了一个表示表明不会再有值发送给该channel。
2. `ele, ok = <- Mychannel`通过循环，接受者会检查该channel是打开的还是关闭的，如果ok是真，则意味着该channel是打开的。
```go
package main

import "fmt"

func main() {
	c := make(chan string)
	// 调用goroutine
	go myFunc(c)
	for {
		res, ok := <-c
		if ok == false {
			fmt.Println("Channel Close", ok)
			break
		}
		fmt.Println("Channel Open", res, ok)
	}
}

func myFunc(mychnl chan string) {
	for v := 0; v < 4; v++ {
		mychnl <- "GeeksforGeeks"
	}
	close(mychnl)
}

```

## Blocking Send and Receive
在channel中，当数据发送给一个channel，直到另外的goroutine读取这个channel前，发送的声明是被阻塞的（发送完后必须等待该信息被读取才能够继续发送）。同样的，当一个channel接受数据的时候在接收到数据前都是被阻塞的。

channel中的零值是`nil`。

`for`循环可以迭代channel中所有连续的值，直到该channel。

```GO
package main

import "fmt"

func main() {
	mychnl := make(chan string)
	go func() {
		mychnl <- "GFG"
		mychnl <- "ABC"
		mychnl <- "ASDJH"
		mychnl <- "asdjjak"
		close(mychnl)
	}()

	for res := range mychnl {
		fmt.Println(res)
	}
}

```
## Length of the Channel
`len()`表明在channel缓冲区中==值的数量==
```go
package main

import "fmt"

func main() {
	mychnl := make(chan string, 4)
	mychnl <- "GFG"
	mychnl <- "ABC"
	mychnl <- "ASDJH"
	mychnl <- "asdjjak"

	fmt.Println("Length of the channel is:", len(mychnl))
}
```

## Capacity of the Channel
`cap()`表明channel缓冲区的大小
```go
package main

import "fmt"

func main() {
	mychnl := make(chan string, 8)
	mychnl <- "GFG"
	mychnl <- "ABC"
	mychnl <- "ASDJH"

	fmt.Println("Capacity of the channel is:", cap(mychnl))
}
```
