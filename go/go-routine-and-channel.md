---
description: go 协程和管道
---

# go routine & channel

### 1.进程 、线程&#x20;

自行百度

### 2.go 协程和go主线程

一个go线程上可以起多个协程，可以理解为协程是轻量级的线程【编译做优化】

go 协程的特点

* 有独立的栈空间
* 共享程序堆空间
* 调度由用户控制
* 协程是轻量级的线程

例子

```go
package main

import (
	"fmt"
	"strconv"
	"time"
)

// gorouting && channel

// example
// 1）在主线程中，开启一个goroutine，改协程每隔一秒输出一个hello,world
// 2) 在主线程中，每隔一秒也输出hello，world，输出10次后，退出程序
// 3) 要求主线程和goroutine同时执行

func test() {
	for i := 0; i < 5; i++ {
		fmt.Printf(" test() -- hello world ---- %v \n", strconv.Itoa(i))
		time.Sleep(time.Second)
	}
}

func main() {
	go test()
	for i := 0; i < 10; i++ {
		fmt.Printf(" main() -- hello world ---- %v \n", strconv.Itoa(i))
		time.Sleep(time.Second)
	}
}

// 1)主线程是一个物理线程，直接作用在cpu上，是重量级的，非常消耗CPU资源
// 2)协程是主线程开启的，是轻量级的线程，是逻辑态，对资源消耗小。
// 3)Golang的协程机制是主要特点，可轻松开启上万个协程
```

![](<../.gitbook/assets/image (12).png>)

MPG 模式介绍



![](<../.gitbook/assets/image (9).png>)

![](<../.gitbook/assets/image (16).png>)

### 3.golang CPU 设置

```go
import (
	"fmt"
	"runtime"
)

func main() {
	// 获取当前CPU数量
	cpuNum := runtime.NumCPU()
	fmt.Printf("CPU mumber is %v", cpuNum)

	//设置golang运行cpu数
	runtime.GOMAXPROCS(cpuNum - 1)
}
```

