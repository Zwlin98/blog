---
title: "TCP Initial Sequence Number"
date: 2021-03-30T19:46:39+08:00
---

这几天在学习 `TCP/IP Illustrate Volume I` 的 Charpter 13，里面的初始序列号的选择问题比较有意思，故单独记一篇。

## Initial Sequence Number (ISN)

当一个 TCP 连接建立之后，只要序列号有效 (在窗口内) 且校验和正确，而且具有正确的两个 IP 地址和端口号的任何 `TCP Segment` 都被接受为有效。那么就存在一个问题：旧的连接的 `TCP Segment` 可能在新的连接中才被接收到，而且恰巧这个 `TCP Segment` 还是合法的，那么就会扰乱现有的连接。

因此在每一端发送其 SYN 以建立 TCP 连接之前，需要为该连接选择 ISN。ISN 应该随时间变化，以便每个连接都有不同的连接。[RFC0793] 指定应将 ISN 视为每 `4  microseconds` 递增 1 的 32 位计数器。这样做的目的是使得一个连接上的 `Segment` 的序列号不与另一个 (新) 相同连接上的序列号重叠。

虽然有着 RFC 的保证，可以使得这种情况的发生的概率最小化，然而，一个非常需要数据完整性的应用程序应该**在应用层有它自己的校验过程**，以确保数据已经被无误地传输。

之前提到的问题还有另外一个隐患：任何人都可以伪造 TCP Segment，如果选择了适当的序列号、IP 地址和端口号，就可以中断 TCP 连接。避免这种情况的一种方法是使初始序列号或临时端口号相对难以猜测。因此 ISN 的安全性非常重要，生成的 ISN 必须得随机，不然可能被人预测从而进行 TCP 包的伪造。

在 TCP/IP 详解的第二版书中提到：

> 在现代系统中，ISN 通常以半随机方式选择。Linux 要选择自己的 ISN，需要经过一个相当复杂的过程。它使用基于时钟的方案，但在每个连接的随机偏移处开始时钟。随机偏移量被选择为连接标识符 (4 元组) 上的加密散列函数。散列函数的秘密输入每5分钟更改一次。在 `ISN` 中的 32 位中，最高的 8 位是秘密的序列号，其余的位由散列生成。这会产生一种很难猜测的 `ISN`，但也会随着时间的推移而增加。

现在的 Linux 实现有无变化我没有考证，但是由上述内容可以知道 `ISN` 的选择需要考虑两方面：

1. 是不能与之前连接的序列号重叠。
2. 要有足够强的随机性，防止恶意的 TCP Segment 伪造 (可以伪造 RST，中断正常的 TCP 连接)。

## Reference

[TCP/IP Illustrated，Volume 1 The Protocol - Kevin R。Fall](https://www.oreilly.com/library/view/tcpip-illustrated-volume/9780132808200/)

[RFC0793](https://datatracker.ietf.org/doc/html/rfc793)

[理解 TCP 报文头中的初始序列号](https://jaminzhang.github.io/network/understanding-tcp-isn/)

[TCP/IP 协议中，在建立连接的时候 ISN 序号分配问题？](https://www.zhihu.com/question/49794331)
