---
title: "kcp 与 kcp-go 的设计与实现"
date: 2024-11-22T11:08:07+08:00
---

## 引言

最近深入研究了 KCP 的实现，通读了 KCP 原版及其 Go 语言实现（kcp-go）的源码。在阅读代码的过程中，还参考了许多优秀的博客文章，十分感谢这些博主的分享。有了这些珠玉在前，我也整理了一些自己对 KCP 的理解，作为学习过程中的记录和总结。

## 两个核心数据结构

`ikcpcb` 是 KCP 的控制块（KCP Control Block）数据结构，包含一条 KCP 连接的所有状态信息和参数。作为 KCP 协议的核心数据结构，它负责管理数据传输、重传机制以及流量控制等功能。

`segment` 是表示 KCP 数据段的结构体，用于描述一个数据包或控制包。每个 `segment` 都包含数据段的头部信息和数据部分。

这两个数据结构定义了 KCP 使用的所有关键字段。关于它们的具体含义，[Luyu Huang 的博文](https://luyuhuang.tech/2020/12/09/kcp.html#总结)中有详细说明，这里就不再赘述了。

## 两重队列与窗口控制

在 KCP 中，数据的发送和接收分别依赖于两种队列：

### 发送队列和发送缓冲区

1. **发送队列 (`snd_queue`**): 调用者在调用 `kcp.send()` 时，数据会被加入发送队列，而不会立即被发送。
2. **发送缓冲区 (`snd_buf`)**: 在调用 `update(flush)` 时，KCP 会尝试将 `snd_queue` 中的数据移入 `snd_buf`，然后遍历发送缓冲区，进行数据包的首次发送或重传。通常，发送缓冲区的队首是 `snd_una` 所代表的数据包。
    - 当 `snd_nxt >= snd_una + cwnd`（即拥塞窗口已满）时，发送缓冲区中的所有数据包都处于等待 ACK 的状态，此时发送队列中的数据无法移入发送缓冲区。
   - 如果在这种情况下持续调用 `kcp.send()`，数据会堆积在 `snd_queue` 中，而不会被真正发送，导致所谓的[缓存积累延迟问题](https://github.com/skywind3000/kcp/wiki/Flow-Control-for-Users)。

**拥塞窗口 (`cwnd`)** 大小受以下因素控制：

- **远端窗口大小（`rmt_wnd`）**
- **本地发送窗口大小（`snd_wnd`）**
- **拥塞控制算法**（如 TCP 的慢启动和拥塞避免）

如果 KCP 配置禁用了慢启动和拥塞避免，则 `cwnd` 仅取决于 `snd_wnd` 和 `rmt_wnd` 的较小值，其中 `snd_wnd` 是用户配置的固定值。

### 接收队列和接收缓冲区

1. **接收缓冲区（`rcv_buf`）**: 数据包在接收时会先放入接收缓冲区。由于传输中可能出现丢包或乱序，接收缓冲区确保数据按顺序排列。
2. **接收队列（`rcv_queue`）**: 调用 `kcp.recv()` 或 `kcp.input()` 时，从 `rcv_buf` 中到 `rcv_nxt` 为止的连续数据包会被移入接收队列，供调用者读取。
    - 如果接收队列的长度 `nrcv_que` 超过 `rcv_wnd`（接收窗口大小），数据将不再从 `rcv_buf` 移入 `rcv_queue`，表明接收窗口已满。

接收窗口的设计主要是为了处理乱序和丢包问题，只有当 `rcv_buf` 中的数据按顺序排列时，才会移入 `rcv_queue`。

### 窗口滑动和速率调整

- **接收端**
   接收端需要及时处理 `rcv_queue` 中的数据，以便窗口及时滑动，确保接收窗口有足够的空间。
- **发送端**
   KCP 会将剩余接收窗口大小（`rcv_wnd - nrcv_que`）通知发送方，发送方据此调整 `cwnd`，以避免发送速率超过接收端的处理能力。如果接收到的数据包序号超过 `rcv_nxt + rcv_wnd`，超出窗口太多，这些数据包会被直接丢弃。

这种设计平衡了可靠性和效率，确保了数据的有序性和网络传输的流畅性。

## RTT/RTO 的计算和设置

KCP 作为一种 ARQ 协议（Automatic Repeat reQuest，自动重传请求协议），发送方需要接收方返回的确认（ACK）来保证数据包成功到达。

### 发送缓存与超时重传

- **发送缓存**: 为确保可靠性，KCP 在发送缓存中暂存已发送但未确认的数据包。一旦接收到对应的 ACK，发送方会将该数据包从发送缓存中移除。
- **超时重传 (RTO)**: 每个数据包都被赋予一个超时时间（RTO，Retransmission Time-Out）。如果在 RTO 时间内未收到 ACK，KCP 会重传该数据包。

### 流水线传输

为了提高传输效率，KCP 支持流水线方式连续发送多个数据包，而不是等待每个数据包被确认后再发送下一个。

- **发送限制**
   连续发送的数据包数量受拥塞窗口（`cwnd`）限制。

### RTO 与 RTT 的更新机制

- **RTO 设置**: KCP 会在每次发送或重传数据包时，为该包计算并设置新的超时时间（RTO）。
- **RTO 更新**: 接收到 ACK 后，KCP 会更新整个连接的 RTO：
   - **KCP 原版**: 每收到一个 ACK，就根据其 RTT 更新一次 RTO。
  - **`kcp-go` 实现**: 当接收到一批 ACK 时，仅使用最新的 RTT 更新一次 RTO。

RTO 和 RTT 的计算方法遵循 TCP 标准，[RFC6298](https://datatracker.ietf.org/doc/html/rfc6298) 中有详细说明。相关文献和博客对其原理有较多解析，这里不再赘述。

### 重传逻辑

在重传数据包时，KCP 会重新设置超时时间：

- **默认设置**: 重传超时时间为 `resent_ts = current + rto * 2`。
- **`nodelay` 模式**: 如果启用了 `nodelay`，重传超时时间则调整为 `current + rto * 1.5`，以进一步减少延迟。

KCP 的这些设计既保证了传输的可靠性，又在不同场景下优化了传输效率，尤其是在需要快速响应的情况下。

## kcp-go 中的 check/update 机制

KCP 是一个运行在用户态的纯 ARQ 协议实现。为了保证操作的正确性，**对同一个 KCP 对象的操作需要在单线程环境中进行**，并通过循环调用 `kcp.update()` 来更新其内部状态，实现协议的运行逻辑。

### 单线程与多连接场景的优化

- **单连接**: 正常情况下，每个 KCP 对象需要周期性调用 `kcp.update()`，以处理定时任务、发送 ACK 和检查重传等操作。
- **多连接**: 在管理大规模 KCP 连接时，频繁调用 `kcp.update()` 会带来性能开销。为了减少不必要的调用，可以使用 `kcp.check()` 优化：
  - `kcp.check()` 会返回一个时间点，表示下一次需要调用 `kcp.update()` 的时间（假设中途没有新的 `kcp.send()` 或 `kcp.input()` 操作）。
  - 这种方法被主流 KCP 使用方案采用，可显著降低资源消耗。

### `kcp-go` 的改进

`kcp-go` 对 `kcp.update()` 和 `kcp.check()` 进行了改进，并已经将其标记为 **deprecated**，引入了更直接的实现方式：

1. `kcp.flush()` 的主要功能
   - **发送 ACK**: 处理 `kp.input()` 生成的待发送 ACK。
   - **窗口管理**: 检查是否需要发送窗口探测包（探测远端接收窗口）和窗口响应包。
   - **队列迁移**: 尽量将数据从 `snd_queue` 移入 `snd_buf`。
   - **数据发送**: 发送未发送的数据包，处理重传（包括快速重传和超时重传）。
   - **拥塞控制（可选）**: 根据丢包情况调整 `cwnd`，进行流量控制。
2. 触发时机
   * `kcp-go`在调用 `kcp.input()` 后立即执行 `kcp.flush()` ，因为此时可能需要：
     * 回复新的 ACK。
     * 收到包后释放 `snd_buf` 的空间，触发新的数据包发送。
   * `kcp.flush()` 的返回值等效于原生 KCP 的 `kcp.check()`，表示下一次需要调用 `flush()` 的时间点。`kcp-go` 会根据这个返回值，将任务调度到相应的时间点。


相比原版 KCP，`kcp-go` 的改动让状态更新和数据处理的逻辑更紧密结合，简化了接口的使用：

- 通过 `flush()` 函数，将 `update()` 和 `check()` 的功能统一，减少显式调用的复杂性。
- 在需要立即响应的场景（如收到数据包）中，自动触发必要的处理流程，提高了实时性和效率。

这些改动使 `kcp-go` 在多连接场景下更高效，同时保持了与原版 KCP 思路的一致性。

{{% inspire %}}

## Inspired by

- [skywind3000/kcp: :zap: KCP - A Fast and Reliable ARQ Protocol](https://github.com/skywind3000/kcp)
- [xtaci/kcp-go: A Crypto-Secure Reliable-UDP Library for golang with FEC](https://github.com/xtaci/kcp-go)
- [详解 KCP 协议的原理和实现 - Luyu Huang's Blog](https://luyuhuang.tech/2020/12/09/kcp.html#%E6%80%BB%E7%BB%93)
- [可靠 UDP 的实现 (KCP over UDP) - 孙同学博客](https://sunyunqiang.com/blog/reliable_udp_protocol/)
- [KCP-GO 的重传机制以及带宽利用率的提升 – 微宇宙](https://zhensheng.im/2021/03/10/kcp-go%E7%9A%84%E9%87%8D%E4%BC%A0%E6%9C%BA%E5%88%B6%E4%BB%A5%E5%8F%8A%E5%B8%A6%E5%AE%BD%E5%88%A9%E7%94%A8%E7%8E%87%E7%9A%84%E6%8F%90%E5%8D%87.meow)
- [kcptun 开发小记 - 知乎](https://zhuanlan.zhihu.com/p/53849089)

{{% /inspire %}}
