---
title: Go-sync.Cond 笔记
date: 2020-10-22 09:10:45
---

<!--more-->

## Concept

> Cond implements a condition variable, a rendezvous point for goroutines waiting for or announcing the occurrence of an event.
>
> Each Cond has an associated Locker L (often a *Mutex or *RWMutex), which must be held when changing the condition and when calling the Wait method.
>
> A Cond must not be copied after first use.

条件变量并不是被用来保护临界区和共享资源的，它是用于协调想要访问共享资源的那些线程的。当共享资源的状态发生变化时，它可以被用来通知被互斥锁阻塞的线程。

## Methods

### func NewCond

```go
func NewCond(l Locker) *Cond
```

NewCond returns a new Cond with Locker l.

### func (*Cond) Broadcast

```go
func (c *Cond) Broadcast()
```

Broadcast wakes all goroutines waiting on c.

It is allowed but not required for the caller to hold c.L during the call.

### func (*Cond) Signal

```go
func (c *Cond) Signal()
```

Signal wakes one goroutine waiting on c, if there is any.

It is allowed but not required for the caller to hold c.L during the call.

### func (*Cond) Wait

```go
func (c *Cond) Wait()
```

Wait atomically unlocks c.L and suspends execution of the calling goroutine. After later resuming execution, Wait locks c.L before returning. Unlike in other systems, Wait cannot return unless awoken by Broadcast or Signal.

**Because c.L is not locked when Wait first resumes, the caller typically cannot assume that the condition is true when Wait returns. Instead, the caller should Wait in a loop**:

```go
c.L.Lock()
for !condition() {
    c.Wait()
}
... make use of condition ...
c.L.Unlock()
```

## Example

```go
package main

import (
	"log"
	"sync"
	"time"
)

func main() {
	var mailbox uint8

	var lock sync.RWMutex

	sendCond := sync.NewCond(&lock)
	recvCond := sync.NewCond(lock.RLocker())

	var wg sync.WaitGroup

	wg.Add(2)
	max := 5
	go func(max int) {
		defer wg.Done()

		for i:=1;i<=max;i++{
			time.Sleep(500*time.Millisecond)
			lock.Lock()
			for mailbox ==1 {
				sendCond.Wait()
			}
			log.Printf("sender [%d]: the mailbox is empty.",i)
			mailbox =1
			log.Printf("sender [%d]: the letter has been sent.", i)
			lock.Unlock()
			recvCond.Signal()
		}
	}(max)

	go func(max int) {
		defer wg.Done()

		for j:=1;j<=max;j++{
			time.Sleep(500*time.Millisecond)
			lock.RLock()
			for mailbox == 0{
				recvCond.Wait()
			}
			log.Printf("receiver [%d]: the mailbox is full.", j)
			mailbox = 0
			log.Printf("receiver [%d]: the letter has been received.", j)
			lock.RUnlock()
			sendCond.Signal()
		}
	}(max)

	wg.Wait()
}
```

## Explanation

条件变量的Wait方法主要做了四件事。

+ 把调用它的 goroutine（也就是当前的 goroutine）加入到当前条件变量的通知队列中。
+ 解锁当前的条件变量基于的那个互斥锁。
+ 让当前的 goroutine 处于等待状态，等到通知到来时再决定是否唤醒它。此时，这个 goroutine 就会阻塞在调用这个Wait方法的那行代码上。
+ 如果通知到来并且决定唤醒这个 goroutine，那么就在唤醒它之后重新锁定当前条件变量基于的互斥锁。自此之后，当前的 goroutine 就会继续执行后面的代码了。

## References

+ https://pkg.go.dev/sync#Cond
+ https://time.geekbang.org/column/article/41588
+ https://time.geekbang.org/column/article/41717
