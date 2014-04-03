##golang
* `for`循环中使用闭包时，一定要显示向闭包函数传递某个循环变量，否则闭包只会使用第一次的值。如：
```golang
for _, conn := range conns {
    go func(c Conn) {
        select {
        case ch <- c.DoQuery(query):
        default:
        }
    }(conn)
}
```
而不能这样
```golang
for _, conn := range conns {
    go func() {
        select {
        case ch <- conn.DoQuery(query):
        default:
        }
    }()
}
```
第二种错误的用法中，闭包中的conn是不会变的。

* 使用WaitGroup
```golang
package util

import (
	"sync"
)

type WaitGroupWrapper struct {
	sync.WaitGroup
}

func (w *WaitGroupWrapper) Wrap(cb func()) {
	w.Add(1)
	go func() {
		cb()
		w.Done()
	}()
}
```
这种方法将`WaitGroup`包装起来，其他的struct中需要等待一系列的goroutine返回时，先生成一个`WaitGroupWrapper`对象，再用该对象的`Wrap`方法包裹要并行且等待的函数，一般是匿名函数的形式。最后调用者通过该对象的`Wait()`方法等待所有函数返回。

* map类型不是线程安全的。对map数据并发读写时需要加锁。
```golang
var counter = struct {
	sync.RWMutex
	m map[string]int
}{m: make(map[string]int)}

counter.RLock()
n := counter.m["some_key"]
counter.Unlock()
fmt.Println(n)
```

* 用一个函数包装另一个，当内部程序panic时，外部程序通过recover收集错误，使整个进程得以继续。
```golang
package main

import "fmt"

func safeHandler(cb func() int) {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println(r)
		}
	}()
	cb()
}

func worker() int {
	b := []int{1,2}
    b[3] = 0
	return 1
}

func main() {
	safeHandler(worker)
	fmt.Println("ok, we are safe to continue.")
}
```
* golang中的yield方式。第一，可以采用在生成器函数中向无缓冲管道写入，写入完毕后close管道。消费者for循环中读取管道，当管道close时自动退出循环。
```golang
package main

import "fmt"

// 生成器
func generator(num ...int) <-chan int {
	c := make(chan int)
	go func() {
		for _, v := range num {
			c <- v
		}
		close(c)
	}()
	return c
}
//消费者
func main() {
	gen := generator(1, 1, 2, 3, 5, 8)
	for n := range gen {
		fmt.Println(n)
	}
}
```
这种生成方式要求消费者必须消费掉所有生成的数据，否则生成器一直阻塞并占用资源，形成了内存的泄露。为了避免这种情况的发生，可以在传递给生成器一个done的通道，当通道激活时生成器退出。利用channel的一个特性，即close执行时，在这个channel上的读取会立刻返回。我们不用手动向channel发送done信号，只需在defer函数中关闭它既可。因此上面的程序改为：

```golang
package main

import "fmt"

// 生成器
func generator(done chan int, num ...int) <-chan int {
	c := make(chan int)
	go func() {
		defer close(c)
		for _, v := range num {
			select {
			case c <- v:
			case <- done:
				return
			}
		}
	}()
	return c
}
//消费者
func main() {
	done := make(chan int)
	defer close(done)
	gen := generator(done, 1, 1, 2, 3, 5, 8)
	for n := range gen {
		fmt.Println(n)
	}
}
```
这样，即使消费者没有完全耗尽所有要生产的数据，生产者也能在接到信号后自行退出。
