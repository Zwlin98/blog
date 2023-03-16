---
title: "Go 1.18 的 append 变化"
date: 2022-08-02T22:39:17+08:00
---

很多 Go 的文章都提到 append 扩容的具体实现，为了确认文章中提到的 1024 阈值，我翻了下 Go 1.18 的源码，发现实现切片扩容的 growslice 函数实际已经变化了，源代码的位置也有一定的改变，因此在此记录一下。实际上 Go 的源代码变化的非常快，函数的位置也经常改变，我这里记录的源码也只是 Go 1.18 这个特定版本的实现和代码位置。

同时在 [Go 1.18 Release Notes](https://tip.golang.org/doc/go1.18#runtime) 中提到：

> The built-in function `append` now **uses a slightly different formula** when deciding how much to grow a slice when it must allocate a new underlying array. The new formula is less prone to sudden transitions in allocation behavior.

以下只摘出核心代码：

append 的中间代码生成部分挪到了 [src/cmd/compile/internal/ssagen/ssa.go](https://github.com/golang/go/blob/f2a9f3e2e0ce7e582d226ad9a41d3c36b146fc25/src/cmd/compile/internal/ssagen/ssa.go#L3346)

核心的 gorwslice 函数在 [src/runtime/slice.go](https://github.com/golang/go/blob/f2a9f3e2e0ce7e582d226ad9a41d3c36b146fc25/src/runtime/slice.go#L200-L223)

```go
func growslice(et *_type, old slice, cap int) slice {
  //...
	newcap := old.cap
	doublecap := newcap + newcap
	if cap > doublecap {
		newcap = cap
	} else {
		const threshold = 256
		if old.cap < threshold {
			newcap = doublecap
		} else {
			// Check 0 < newcap to detect overflow
			// and prevent an infinite loop.
			for 0 < newcap && newcap < cap {
				// Transition from growing 2x for small slices
				// to growing 1.25x for large slices. This formula
				// gives a smooth-ish transition between the two.
				newcap += (newcap + 3*threshold) / 4
			}
			// Set newcap to the requested cap when
			// the newcap calculation overflowed.
			if newcap <= 0 {
				newcap = cap
			}
		}
	}
  //...
}
```

可以看出在 Go 1.18 中，扩容策略 (不考虑内存对齐的情况下) 变成了：

1. 如果期望容量大于当前容量的两倍就会使用期望容量
2. 如果当前切片的长度小于 256 就会将容量翻倍
3. 如果当前切片的长度大于 256 就会每次增加 25% 的同时再增加 192(256 的 3/4)，直到新容量大于期望容量

按照注释的说法是新的公式 newcap += (newcap + 3*threshold) / 4 会使得容量的变化更加平滑。顺带一提，在之前的版本中公式为 newcap += newcap / 4

源码中的实现才是最准确的，网上的文章都有时效性，切勿奉为圭臬。

