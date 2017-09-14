---
title: go的是否需要用goroutine pool？
date: 2017-08-03 23:10:02
tags: go 
categories: go
---
> 这几天无聊，想到java有自己的线程池，是否对应go也有它的goroutine pool呢，所以搜了下，标准库没有，github有，但都大同小异，所以自己实现了一个。

<!-- more -->
# 一个简单的goroutine pool
````go
package workerpool

import (
	"sync"
	"fmt"
)

type task func()

type worker struct {
	stopC chan bool
}

type WorkerPool struct {
	num int
	sync.Mutex
	taskQ chan task
	workers []*worker
}

func NewWorkerPool(workerNum int,queueCap int) *WorkerPool{
	return &WorkerPool{num:workerNum,taskQ:make(chan task,queueCap),workers:make([]*worker,workerNum)}
}

func (wp *WorkerPool) Execute(t task){
	wp.taskQ<-t
}

func (wp *WorkerPool) Start() *WorkerPool{
	for i:=0;i<wp.num;i++{
		wp.workers[i]=&worker{ make(chan bool)}
		w:=wp.workers[i]
		go func(i int) {
			for {
				    stop:=false
					select {
					    case f:=<-wp.taskQ:
							f()
					    case stop=<-w.stopC:
						     break

					}

				if stop{
					break
				}
			}
			fmt.Println("stop")
		}(i)
	}
	return wp
}

func (wp *WorkerPool) Stop(){
	for _,w:=range wp.workers{
		w.stopC<- true
	}
}
````
代码很简单，就是`NewWorkerPool`一个池子的时候设置goroutine的数量和任务队列的大小。`Start`后就创建那么多goroutine去任务队列取任务执行，取不到任务就自循。`Execute`方法是把任务压进队列，如果队列满了就阻塞。

# 性能
要测试性能，肯定要有对比，以下是没有使用pool:
````go
func nopool() {
	wg := new(sync.WaitGroup)
    //执行1000000次，每次都启动一个goroutine
	for i := 0; i < 1000000; i++ {
		wg.Add(1)
		go func(n int) {
			for j := 0; j < 100000; j++ {

			}
			defer wg.Done()
		}(i)
	}
	wg.Wait()
}
````
以下是简单版的只是单纯限制goroutine数量和任务队列的代码，没有任何封装的:
````go
func gopool() {
	wg := new(sync.WaitGroup)
    //队列100
	data := make(chan int, 100)

    //goroutine 数量10个
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func(n int) {
			defer wg.Done()
			for _ = range data {
				func(){
					for j := 0; j < 100000; j++ {

					}
				}()

			}
		}(i)
	}

    //执行1000000个任务
	for i := 0; i < 1000000; i++ {
		data <- i
	}
	close(data)
	wg.Wait()
}
````
然后是主角:
````go
func workerpool() {
	wg := new(sync.WaitGroup)
    //十个goroutine，队列容量100
	wp:=NewWorkerPool(10,100)
	wp.Start()
    //提交1000000任务
	for i := 0; i < 1000000; i++ {
		wg.Add(1)
		wp.Execute(func() {
			for j := 0; j < 100000; j++ {

			}
			wg.Done()
		})
	}

	wg.Wait()
}
````
上面代码基本都是做同样一件事，但是后两个只开10个goroutine，第一个就开了1000000个，结果：
````bash
BenchmarkNopool-8                      1        7966900091 ns/op
BenchmarkGopool-8                      1        7949844269 ns/op
BenchmarkWorkerPool-8                  1        7997732135 ns/op
````
可以看出来，没有区别，重新run几次基本没有多大变化。

# 总结
由于go本身有对goroutine有调度，所以自己实现的池子来调度其实好像没有什么用。还有可能我自己能力实现不好，没发挥池子的作用😀。
但是用更少的goroutine能完成同样的事情，应该是一种优化，而且这里的goroutine执行都是简单的循环，没有复杂的业务，一旦业务复杂，更少goroutine能够减少内存和goroutine切换时的cpu资源，有可能上面性能的比较会拉开。

