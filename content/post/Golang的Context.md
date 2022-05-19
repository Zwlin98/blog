---

title: "Go Concurrency Patterns: Context"
date: 2020-10-20 11:04:49

---

## Overview 

> In Go servers, each incoming request is handled in its own goroutine. Request handlers often start additional goroutines to access backends such as databases and RPC services. The set of goroutines working on a request typically needs access to request-specific values such as the identity of the end user, authorization tokens, and the request's deadline. When a request is canceled or times out, all the goroutines working on that request should exit quickly so the system can reclaim any resources they are using.

Package context defines the Context type, which carries deadlines, cancellation signals, and other request-scoped values across API boundaries and between processes.
<!--more-->
Incoming requests to a server should create a Context, and outgoing calls to servers should accept a Context. The chain of function calls between them must propagate the Context, optionally replacing it with a derived Context created using WithCancel, WithDeadline, WithTimeout, or WithValue. **When a Context is canceled, all Contexts derived from it are also canceled.**

The WithCancel, WithDeadline, and WithTimeout functions take a Context (the parent) and return a derived Context (the child) and a **CancelFunc**. Calling the CancelFunc cancels the child and its children, removes the parent's reference to the child, and stops any associated timers. Failing to call the CancelFunc leaks the child and its children until the parent is canceled or the timer fires. The go vet tool checks that CancelFuncs are used on all control-flow paths.

## Rules

+ Do not store Contexts inside a struct type; instead, pass a Context **explicitly** to each function that needs it. The Context should be the first parameter, typically named ctx:

  ```go
  func DoSomething(ctx context.Context, arg Arg) error {
  	// ... use ctx ...
  }
  ```
  
+ **Do not pass a nil Context**, even if a function permits it. Pass **context.TODO** if you are unsure about which Context to use.

+ Use context Values only for request-scoped data that transits processes and APIs, not for passing optional parameters to functions.

+ The same Context may be passed to functions running in different goroutines; **Contexts are safe for simultaneous** use by multiple goroutines.

## Usages

### WithCancel

#### Introduce

WithCancel returns a **copy** of parent with a new Done channel. The returned context's Done channel is closed when the returned cancel function is called or when the parent context's Done channel is closed, whichever happens first.

Canceling this context releases resources associated with it, so code should call cancel as soon as the operations running in this Context complete.

#### Example
```go
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	gen := func(ctx context.Context) <- chan int{
		dst := make(chan int)
		n := 1
		go func() {
			for{
				select {
				 case <-ctx.Done():
					 fmt.Println("gen exit")
					 return
				case dst<-n:
					n++
				}
			}

		}()
		return dst
	}

	ctx,cancel := context.WithCancel(context.Background())

	for n := range gen(ctx){
		fmt.Println(n)
		if n==5 {
			break
		}
	}
	cancel()

	time.Sleep(500*time.Millisecond)
}
```
[Play](https://play.golang.org/p/BCfxv-Ho7IT)

### WithDeadLine

#### Introduce

WithDeadline returns a copy of the parent context with the deadline adjusted to be no later than d. If the parent's deadline is already earlier than d, WithDeadline(parent, d) is semantically equivalent to parent. The returned context's Done channel is closed when the deadline expires, when the returned cancel function is called, or when the parent context's Done channel is closed, whichever happens first.

Canceling this context releases resources associated with it, so code should call cancel as soon as the operations running in this Context complete.

#### Example

```go
package main

import (
	"context"
	"fmt"
	"time"
)

const shortDuration = 1 * time.Millisecond

func main() {
	d := time.Now().Add(shortDuration)
	ctx, cancel := context.WithDeadline(context.Background(), d)

	defer cancel()

	select {
	case <-time.After(1 * time.Second):
		fmt.Println("overslept")
	case <-ctx.Done():
		fmt.Println(ctx.Err())
	}
}
```

[Play](https://play.golang.org/p/XaoQ8uQH2Dq)

### WithTimeout

#### Introduce

WithTimeout returns WithDeadline(parent, time.Now().Add(timeout)).

Canceling this context releases resources associated with it, so code should call cancel as soon as the operations running in this Context complete.

### WithValue

#### Introduce

WithValue returns a copy of parent in which the value associated with key is val.

**Use context Values only for request-scoped data that transits processes and APIs, not for passing optional parameters to functions.**

The provided key must be comparable and should not be of type string or any other built-in type to avoid collisions between packages using context. Users of WithValue should define their own types for keys. To avoid allocating when assigning to an interface{}, context keys often have concrete type struct{}. Alternatively, exported context key variables' static type should be a pointer or interface.

#### Example

```go
package main

import (
	"context"
	"fmt"
	"time"
)

type favContextKey string

func main() {
	k := favContextKey("language")
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	ctx2 := context.WithValue(ctx, k, "-> Route 2")

	for num := range gen(ctx2){
		fmt.Println(num)
		if num==5{
			break
		}
	}

	cancel()
	fmt.Println("Call Cancel!")
	time.Sleep(1 * time.Second)
}

func gen(ctx context.Context) <-chan int {
	dst := make(chan int)
	n := 1
	k := favContextKey("language")
	go func() {
		for {
			select {
			case <-ctx.Done():
				fmt.Println("-> Route 1")
				return
			case dst <- n:
				fmt.Println(ctx.Value(k))
				n++
			}
		}
	}()
	return dst
}

```

[Play](https://play.golang.org/p/skic91d1vUx)

## References

+ [1] https://pkg.go.dev/context
+ [2] https://blog.golang.org/context
+ [3] https://blog.golang.org/pipelines
+ [4] https://time.geekbang.org/column/article/42158
