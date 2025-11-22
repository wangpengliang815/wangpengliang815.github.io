并发编程在当前软件领域是一个非常重要的概念，随着CPU等硬件的发展，都希望让程序运行的更快。Go语言在语言层面天生支持并发（基于 `goroutine` 的高并发），充分利用现代CPU的多核优势，这也是Go语言能够大范围流行的一个很重要的原因。

# 概念了解

## 串行/并发/并行

**串行**：按照一定顺序先后执行

**并发**：同一时间段内执行多个任务，逻辑上的同时发生。一个处理器（在不同时刻或者说在同一时间间隔内）"同时"处理多个任务。宏观上是并发的，微观上是按排队等待、唤醒、执行的步骤序列执行

**并行**：物理上的同时发生。多核处理器或多个处理器（在同一时刻）同时处理多个任务。并行性允许多个程序同一时刻可在不同 CPU 上同时执行

## 进程/线程/协程

进程（process）：程序在操作系统中的一次执行过程，系统进行资源分配和调度的一个独立单位

线程（thread）：操作系统基于进程开启的轻量级进程，是操作系统调度执行的最小单位

协程（coroutine）：非操作系统提供而是由用户自行创建和控制的 `用户态线程`，比线程更轻量级

## 阻塞/非阻塞

**阻塞**：阻塞是进程(也可以是线程、协程)的状态之一（新建、就绪、运行、阻塞、终止). 指的是当数据未准备就绪，这个进程(线程、协程)一直等待，这就是阻塞

**非阻塞**： 当数据为准备就绪，该进程(线程、协程)不等待可以继续执行，这就是非阻塞

## 同步/异步

**同步**： 发起一个调用时，在没有得到结果之前，这个调用就不返回。调用过程一直在等待。这是同步

**异步**：发起调用后就立刻返回，这次调用过程就结束了，等到有结果了被调用方主动通知调用者结果。这是异步

# goroutine

`goroutine` 是 Go 语言支持并发的核心，一个 `goroutine` 会以一个很小的栈开始其生命周期，一般只需要 `2KB` 。与操作系统线程区别在于：操作系统线程由系统内核进行调度， `goroutine` 是由Go运行时（runtime）负责调度。Go运行时会智能地将 m个 `goroutine` 合理地分配给 `n` 个操作系统线程，实现类似 `m:n` 的调度机制，不再需要Go开发者自行在代码层面维护一个线程池。

`goroutine` 是 Go 程序中最基本的并发执行单元。每一个 Go 程序都至少包含一个 goroutine（`main goroutine`），Go程序启动时它会自动创建。Go语言编程中不需要去自己写进程、线程、协程，当需要让某个任务并发执行的时候，只需要把这个任务包装成一个函数，开启一个 `goroutine` 去执行这个函数就可以了。

# go关键字

Go语言中只需要在函数或方法调用前加上`go`关键字就可以创建一个 `goroutine` ，让该函数或方法在新创建的 `goroutine` 中执行。

```go
go test()  // 创建一个新的 goroutine 运行函数test
```

匿名函数也支持使用 `go` 关键字创建 `goroutine` 执行。

```go
go func(){
  // ...
}()
```

一个 `goroutine` 必定对应一个函数/方法，可以创建多个 `goroutine` 去执行相同的函数/方法。

## 启动单个goroutine

先看一个在 main 函数中执行普通函数调用的示例：

```go
func main() {
	func() {
		fmt.Println("hello")
	}()
	fmt.Println("end...")
}
```

输出：

```bash
hello
end...
```

代码会顺序打印 `hello ` 和 `end...` 是串行的。接下来在匿名函数前加上关键字 `go`，启动一个 `goroutine` 去执行这个函数：

```go
func main() {
	go func() {
		fmt.Println("hello")
	}()
	fmt.Println("end...") // 这里只会输出end...
}
```

输出：

```bash
end...
```

这一次的执行结果只在终端打印了 `end...` 并没有打印 `hello`，原因在于：

> Go程序启动时会为 main 函数创建一个默认的 `goroutine` 。上面代码中在 main 函数中使用 go 关键字创建了另外一个 `goroutine` 去执行匿名函数，而此时 `main goroutine` 还在继续往下执行，程序中此时存在两个并发执行的 `goroutine` 。当 main 函数结束时整个程序也就结束了，同时 main goroutine 也结束了，所有由 `main goroutine`  创建的 `goroutine` 也会一同退出。也就是说 main 函数退出太快，另外一个 `goroutine` 中的函数还未执行完程序就退出了，导致未打印出“hello”。其实跟C#中主线程退出导致所有的子线程都会退出一个道理。

使用 `time.Sleeep` 让 `main goroutine` 等待一下：

```go
func main() {
	go func() {
		fmt.Println("hello")
	}()
	fmt.Println("end...") // 先输出:end... 再输出:hello
	time.Sleep(time.Second)
}
```

重新编译后再次执行，程序会在终端输出如下结果，并且会短暂停顿一下。

```bash
end...
hello
```

这里 `end...` 会首先被输出的原因在于：程序中创建 `goroutine` 执行函数需要一定的开销，而此时 main 函数所在的 `goroutine` 是继续执行的，所以会先输出 `end...`。



上面代码中使用 `time.Sleep` 让 `main goroutine` 等待匿名函数执行结束是不优雅也是不准确的，Go 语言中 `sync` 包提供了一些常用的并发原语。当并不关心并发操作结果或者有其它方式收集并发操作的结果时，`WaitGroup` 是实现等待一组并发操作完成的好方法。简单试一下：

```go
func main() {
	wg.Add(1) // 启动一个goroutine登记+1
	go func() {
		defer wg.Done() // goroutine结束登记-1
		fmt.Println("hello")
	}()
	fmt.Println("end...")
	wg.Wait() // 等待所有线程执行完,先输出:end... 再输出:hello
}
```

将代码编译后执行，得到的输出结果和之前一致，但是这一次程序不再会有多余的停顿， 匿名函数的 `goroutine` 执行完毕后程序直接退出。

## 启动多个goroutine

```go
package main

import (
	"fmt"
	"sync"
)

var wg sync.WaitGroup

func printHello(i int) {
	defer wg.Done()
	fmt.Println("hello", i)
}

func main() {
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go printHello(i)
	}
	wg.Wait()
}
```

多次执行上面的代码会发现每次终端上打印数字的顺序都不一致。因为10个 `goroutine` 是并发执行的，而 `goroutine` 的调度是随机的。

再来看个问题，将 `printHello()` 修改为匿名函数实现：

```go
package main

import (
	"fmt"
	"sync"
)

var wg sync.WaitGroup

func printHello(i int) {
	defer wg.Done() // goroutine结束就登记-1
	fmt.Println("hello:", i)
}

// 使用匿名函数发现多次输出同一个值
func closure2() {
	for i := 0; i < 5; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			// 这里匿名函数的i是由外层循环提供的其实就是一个闭包
			fmt.Println("hello:", i)
		}()
	}
	wg.Wait()
}
func main() {
	closure2()
}
```

多次执行上面的代码会发现每次终端上可能存在多次输出同一个值的问题，问题在于：匿名函数这里形成了一个闭包

```go
go func() {
			defer wg.Done()
			// 这里匿名函数的i是由外层循环提供的其实就是一个闭包
			fmt.Println("hello:", i)
		}()
```

解决办法也很简单：修改一下将参数 `i` 传入匿名函数中使用新变量接收：

```go
func closure3() {
	for i := 0; i < 5; i++ {
		wg.Add(1)
		go func(i int) {
			defer wg.Done()
			fmt.Println("hello:", i)
		}(i)
	}
	wg.Wait()
}
```



