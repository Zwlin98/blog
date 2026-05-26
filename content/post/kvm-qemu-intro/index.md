---
title: "KVM/QEMU 初探"
date: 2026-05-26T17:07:43+08:00
---

看过我其他文章的朋友应该能看出来，我一直是一个 self-hosted 爱好者。无论是折腾 PVE、网络，还是各种跑在自己服务器上的服务，虚拟化都是绕不开的一层。平时用起来倒是很自然，但越用越会好奇：一台 VM 到底是怎么被跑起来的？PVE 和 `virt-manager` 这类工具背后究竟做了什么？

这个学习计划其实搁置过很久。QEMU/KVM 的入口并不算友好，QEMU 参数、libvirt、KVM API、virtio、tap/bridge 每个方向都能展开，之前总觉得需要一整块时间才能系统梳理。现在有了 Codex 这样的 agent，反而终于可以释放一下这部分好奇心了。

{{% btw %}}

当然，Codex 并不能替我理解 KVM/QEMU。它更像是一个可以被反复追问的搭子，真正有价值的还是我能不能问出下一个问题。

{{% /btw %}}

这里真正有帮助的不是让 Codex 重新解释一遍“KVM 是什么”，而是我可以带着自己当前的理解去提问。比如 `/dev/kvm` 为什么是字符设备，`kvm_intel ... 0` 是否表示没人使用 KVM，`-smp 2` 对应的是 vCPU 线程还是 memory slot，`KVM_RUN` 到底是在运行整台 VM 还是某个 vCPU。每个问题本身都不算宏大，但连续问下去，就会逐渐把脑子里混在一起的层次拆开。

KVM/QEMU 这类系统软件尤其如此。很多名词单独看都不难，但一旦放在一起，就会出现各种边界不清的问题：KVM 和 QEMU 谁才是 hypervisor？libvirt 到底做了什么？vCPU 在 host 上是什么？guest memory 到底存在哪里？这些问题如果不问清楚，后面直接读源码或者堆参数，大概率只会越来越乱。

所以这篇先整理其中一部分问答。

## 谁真正启动了一台 VM？

一开始最容易混的就是 QEMU、KVM、libvirt 之间的关系。

如果平时使用的是 PVE、`virt-manager` 或 `virsh`，很容易形成一种感觉：虚拟机是由这些管理工具“启动”的。但继续往下问一层，就会发现它们更多是管理层。libvirt 负责保存配置、管理生命周期、处理存储池和虚拟网络，并把这些配置翻译成 QEMU 启动参数。

真正把一台 VM 组织起来的是 QEMU。vCPU、内存、磁盘、网卡、串口、固件、设备模型，这些都需要 QEMU 参与。

KVM 的位置更低一些。它在 Linux kernel 里，提供 `/dev/kvm` 这个入口和一组 ioctl。QEMU 通过这些 ioctl 创建 VM、创建 vCPU、注册 guest memory，并让 vCPU 进入 guest 执行。

所以 KVM 本身并不能“直接启动一台虚拟机”。它提供的是虚拟化执行能力，不负责替你组装一台完整的机器。

目前我脑子里的路径大致是：

```text
PVE / virsh / virt-manager
  -> libvirt
    -> QEMU
      -> /dev/kvm
        -> KVM
          -> VMX/SVM（CPU 硬件虚拟化扩展）
```

ESXi、VirtualBox、PVE 这些名字也可以放到这张图里理解。ESXi 更像 VMware 自己的一整套虚拟化产品；VirtualBox 是另一套 VMM 和内核模块体系；PVE 则是基于 Linux、QEMU/KVM、LXC 等组件封装出来的平台。它们都在解决“如何管理和运行虚拟机”的问题，但底层路径并不相同。

对我现在的学习路线来说，先跳过 libvirt 是有必要的。不是因为 libvirt 不重要，而是直接手写 QEMU 命令，能更清楚地看到一台 VM 到底是由哪些部分拼出来的。等这条路径稳定之后，再回头看 libvirt 做了哪些封装，理解成本会低很多。

## KVM 在本机上是什么？

当我问“KVM 是什么”时，最容易得到的是一句抽象定义：KVM 是 Linux 内核里的虚拟化模块。这句话当然没错，但如果停在这里，它仍然只是一个名词。

更有用的问题是：在本机上，我从哪里能看到 KVM？

首先是 `/dev/kvm`：

```bash
ls -l /dev/kvm
```

输出类似：

```text
crw-rw---- 1 root kvm 10, 232 ... /dev/kvm
```

开头的 `c` 表示这是一个 character device，也就是字符设备文件。权限细节会随发行版、用户组和 ACL 配置变化，这里只需要先关注它是 `root:kvm` 下的设备节点。它不是普通文件，更像 Linux kernel 暴露给 userspace 的一个操作入口。QEMU 打开这个设备文件后，通过 ioctl 和 KVM 交互。

再看内核模块：

```bash
lsmod | rg '^kvm'
```

常见输出里会有：

```text
kvm_intel  ...
kvm        ...
```

`kvm` 是通用 KVM 框架；`kvm_intel` 是 Intel VT-x/VMX 后端，AMD 机器上通常对应 `kvm_amd`。

这里还有个小问题值得单独记一下：`lsmod` 里的 Used-by 不是用户态进程列表，不能用它判断有没有 QEMU/VM 正在使用 KVM。所以 `kvm_intel ... 0` 不代表没有 VM 正在使用 KVM。

这一小段问答对我很有帮助。因为它把 KVM 从一个抽象名词落到了具体对象上：一个内核模块，以及一个 userspace 可以打开的字符设备。后面再看到 QEMU 打开 `/dev/kvm`，也就不再只是“使用 KVM 加速”这几个字了。

## VM 在 host 上长什么样？

另一个重要问题是：guest 里看到的一台机器，在 host 上到底是什么？

启动 VM 后，host 上首先能看到的是一个普通 Linux 进程：

```bash
ps -ef | rg qemu-system
```

如果启动时指定：

```bash
-smp 2
```

guest 里看到的是两个 vCPU，而 host 上更接近的视角是：QEMU 进程里有对应的 vCPU 线程。

```bash
ps -T -p <qemu-pid>
```

模型大致如下：

```text
host
  qemu-system-x86_64 进程
    thread A -> guest vCPU0
    thread B -> guest vCPU1
    thread C -> 设备模拟 / main loop / I/O
```

所以后续谈 CPU pinning 时，实际操作对象往往是 host 上的 QEMU vCPU 线程，而不是某个抽象的“虚拟 CPU 实体”。

沿着这个问题继续问下去，就会自然碰到 guest memory。QEMU 进程的地址空间里会有一大块 guest RAM。guest 以为这是自己的物理内存，但在 host 看来，它主要是 QEMU 进程的一段 userspace memory。

```text
guest physical address
  -> KVM memslot
  -> QEMU process userspace_addr
  -> host physical memory
```

QEMU 会通过 `KVM_SET_USER_MEMORY_REGION` 告诉 KVM，guest physical address 的某一段，对应 QEMU 进程里的哪一段内存。也就是说，guest RAM 不是 KVM 模块里单独存了一份，而是由 QEMU 进程提供，再交给 KVM 建立映射关系。

这个模型对我很重要。它把 QEMU 进程、guest physical memory 和 KVM memslot 串了起来。

## 第一条 QEMU 命令为什么这么写？

我是在 Arch Linux 上做实验。为了先把主路径跑起来，只需要安装基础组件：

```bash
sudo pacman -S qemu-base
```

如果需要图形窗口，再装：

```bash
sudo pacman -S qemu-desktop
```

guest 镜像这里优先选择 Arch cloud image 里的 `basic` qcow2，而不是 `cloudimg`。后者更偏 cloud-init 和云平台场景，通常还需要 seed image 注入用户、密码或 SSH key；`basic` 更适合直接在本地 QEMU 里作为 boot disk 启动。

为了不破坏原始镜像，可以先基于 backing file 创建一个 qcow2 overlay：

```bash
qemu-img create -f qcow2 \
  -F qcow2 \
  -b Arch-Linux-x86_64-basic.qcow2 \
  arch-test.qcow2
```

`qcow2` 全称是 `QEMU Copy On Write version 2`。它是 QEMU 常用的磁盘镜像格式，支持稀疏分配、backing file 和快照等能力。可以用下面的命令查看镜像信息：

```bash
qemu-img info arch-test.qcow2
```

最终实际跑起来的命令如下：

```bash
qemu-system-x86_64 \
  -machine accel=kvm \
  -cpu host \
  -m 4G \
  -smp 2 \
  -drive file=arch-test.qcow2,format=qcow2,if=virtio
```

这条命令本身也可以看成一串问题的答案。相比直接复制一条“最小启动命令”，我更关心的是每个参数到底在回答什么问题：

```text
用不用 KVM？
  -machine accel=kvm

guest 看到什么 CPU？
  -cpu host

给多少初始内存？
  -m 4G

启动几个 vCPU？
  -smp 2

磁盘从哪里来，以什么设备形式接进去？
  -drive file=arch-test.qcow2,format=qcow2,if=virtio
```

{{% btw %}}

最开始我确实尝试过纯串口启动，然后卡在 `Welcome to GRUB!`。后来发现不是 VM 没跑起来，而是输出路径没接对。于是先回到图形窗口，别一开始就给自己增加难度。

{{% /btw %}}

这里没有显式加 `-nographic` 或 `-serial`，因此启动后走的是 QEMU 默认的图形显示路径。这条命令假设本机已经有可用的图形显示后端；如果只想 headless 启动，串口 console 可以之后再单独折腾。

## `strace` 证明了什么？

真正让我对这条路径有感觉的是 `strace`。相比“QEMU 会调用 KVM”这句话，系统调用记录要具体得多：

```bash
strace -f -e openat,ioctl qemu-system-x86_64 -machine accel=kvm ...
```

输出里可以看到 QEMU 打开 `/dev/kvm`：

```text
openat(AT_FDCWD, "/dev/kvm", O_RDWR|O_CLOEXEC) = 20
```

随后是几个关键 ioctl。这里截取的是这次实验里比较关键的调用，实际数量和顺序会受到 QEMU 版本、machine type、固件和设备配置影响。

```text
ioctl(20, KVM_GET_API_VERSION, 0) = 12
ioctl(20, KVM_CREATE_VM, 0) = 21
ioctl(21, KVM_CREATE_VCPU, 0) = 22
ioctl(21, KVM_CREATE_VCPU, 1) = 24
ioctl(21, KVM_SET_USER_MEMORY_REGION, {...}) = 0
```

看到这些输出之后，前面那些概念就不只是文字了。QEMU 确实打开了 `/dev/kvm`，确实通过 ioctl 创建 VM fd 和 vCPU fd，也确实把 guest memory 注册给 KVM。之前的分层图，在这里第一次和本机上的进程行为对应了起来。

这一段可以翻译成：

```text
QEMU 打开 /dev/kvm
  -> 检查 KVM API version
  -> 创建 VM fd
  -> 创建 vCPU fd
  -> 注册 guest memory
  -> 后续通过 KVM_RUN 让 vCPU 进入 guest 执行
```

`KVM_CREATE_VM` 创建的是 VM fd；`KVM_CREATE_VCPU` 基于 VM fd 创建 vCPU fd；`KVM_SET_USER_MEMORY_REGION` 把 guest physical address 的某一段映射到 QEMU userspace memory；`KVM_RUN` 则是运行某个 vCPU。

这里需要注意的是，`KVM_RUN` 运行的是某个 vCPU，不是“一次运行整台 VM”。多 vCPU VM 通常会有多个 QEMU vCPU 线程，各自围绕自己的 vCPU fd 调用 `KVM_RUN`。

## `KVM_RUN` 像不像 coroutine？

这是我觉得比较有意思的一个问题。

`KVM_RUN` 可以先类比成 Lua coroutine 的 `resume`。不过这里要注意，硬件层面的 VM-exit 会先回到 KVM kernel，并不一定直接回到 QEMU；只有 KVM 需要 userspace 处理时，`KVM_RUN` 才会返回到 QEMU，这时候才更像一次 `yield`。

```text
QEMU vCPU thread:
  类似 coroutine scheduler 的执行线程

KVM_RUN:
  类似 resume(vcpu)

KVM_RUN 返回 QEMU userspace:
  类似 yield back

exit_reason:
  类似 yield 返回原因
```

典型循环大概长这样：

```c
for (;;) {
    ioctl(vcpu_fd, KVM_RUN);

    switch (run->exit_reason) {
    case KVM_EXIT_IO:
        handle_io();
        break;
    case KVM_EXIT_MMIO:
        handle_mmio();
        break;
    case KVM_EXIT_HLT:
        handle_hlt();
        break;
    }
}
```

vCPU 进入 guest 执行后，并不是 QEMU 每条指令都在旁边解释。大部分时候，guest 借助硬件虚拟化直接运行。遇到某些事件时会先 VM-exit 到 KVM；如果 KVM 可以在内核里处理，guest 可能会继续运行，只有需要 QEMU 处理时，`KVM_RUN` 才会返回到 userspace。

另一个容易误解的点是，guest 里的普通 syscall 通常不会 yield 到 QEMU。guest app 的 syscall 大多在 guest userspace 到 guest kernel 内部完成。更像 yield 的，是需要 userspace 设备模型参与的 PIO/MMIO 访问、`HLT`、shutdown/reset 等会让 `KVM_RUN` 返回的情况。

这层类比目前只服务于一个问题：`KVM_RUN` 让 vCPU 进入 guest 执行，而某些 exit 又会把控制权交还给 QEMU。它是按 vCPU 执行的，而不是按整台 VM 执行。类比真实多核机器的话，CPU0 上发生 syscall 不会让 CPU1 停下来；在 KVM/QEMU 里，vCPU0 因为某个事件退出到 QEMU 时，vCPU1 也可能仍然在 guest 里继续执行。

所以多 vCPU VM 更像一组可以并行推进的执行上下文：

```text
KVM_RUN(vcpu0):
  resume(vCPU0)

KVM_RUN(vcpu1):
  resume(vCPU1)
```

顺着这个类比继续想，不同系统里其实都有类似的 state / event / transition，只是保存状态的位置和触发事件不同：

```text
Lua coroutine:
  state       = 栈、PC、局部变量
  event       = resume / yield
  transition  = runnable <-> suspended

进程 / OS:
  state       = task_struct、寄存器、地址空间、fd table
  event       = syscall / interrupt / page fault / timer
  transition  = running / runnable / sleeping / stopped

VM / VMM:
  state       = vCPU 寄存器、guest memory view、KVM run state
  event       = KVM_RUN / KVM_RUN 返回 / interrupt injection
  transition  = running in guest / returned to QEMU / waiting
```

这里还有一个相关的直觉：交出控制权之后，从执行体自己的内部视角看，世界像是暂停了。

```text
coroutine yield 后:
  它自己的栈和 PC 停住，下次 resume 像是从 yield 返回。

进程被调度出去:
  它的寄存器和执行位置被保存，再调度回来时继续执行。

vCPU 退出到 QEMU:
  guest vCPU 状态被保存，QEMU/KVM 处理事件后再 KVM_RUN。
```

这个视角不等于真的理解了调度器或 KVM 内部实现，但它能帮助我在读系统代码时先问清楚：状态保存在哪里，是什么事件让它停下来，又是谁把它恢复执行。

## 还有哪些问题先放一放？

这一轮还顺手厘清了不少边界，但它们更像后续问题的入口，不适合在这篇里完全展开。

比如 virtio。它不是“某个网络协议”，而是一套标准化虚拟设备接口规范。核心抽象是 guest driver 和 host backend 之间共享的 `virtqueue`。之前做 TUN/TAP 的 GSO/GRO 时遇到的 `virtio_net_hdr`，也可以放在这里理解：它不是普通以太网包的一部分，而是 virtio-net 用来随 packet 传递 checksum/GSO 等 offload 信息的 metadata。

比如 vhost。它关注的是“谁来高效处理 virtqueue”。纯 QEMU virtio-net 由 QEMU 处理 virtqueue；`vhost-net` 把主要处理路径放到 host kernel；`vhost-user` 则把设备 backend 放到 QEMU 外部的用户态进程里。

比如网络。`-netdev user` 是 QEMU 传统用户态网络方案，通常基于 libslirp；`passt` 是较新的外部用户态网络后端；`tap + Linux bridge` 则更像把 VM 接入一个二层交换机：

```text
guest virtio-net
  -> QEMU
  -> tap0
  -> Linux bridge br0
  -> host physical NIC / other tap / veth
```

还有 `-smp`、CPU topology、memory hotplug、QEMU hub、QMP 这些问题。它们都值得展开，但当前先把它们放到正确位置即可。真正重要的是先知道：哪些是 QEMU 负责的，哪些是 KVM 负责的，哪些又需要 guest kernel 配合。

## 写在最后

写到这里，最主要的收获不是记住了多少 QEMU 参数，而是通过一连串问题，把几个原本混在一起的概念拆开了：

```text
QEMU 是 host 上的 userspace 进程；
vCPU 在 host 上通常表现为 QEMU 线程；
guest RAM 主要来自 QEMU 进程的 userspace memory；
QEMU 通过 /dev/kvm 和 ioctl 使用 KVM；
KVM_RUN 运行的是某个 vCPU，而不是抽象地运行整台 VM。
```

这可能也是现在用 Codex 学技术时最有价值的地方。知识本身不再稀缺，甚至解释也不稀缺；更重要的是，我能不能根据当前理解提出一个足够具体的问题，并且在答案里继续发现下一个问题。问得越具体，Codex 给出的反馈越容易变成可验证的实验或可修正的模型。

下一步会进入 KVM API 和最小 VMM。到时候会自己写一个小 VMM，真正走一遍：

```text
open("/dev/kvm")
  -> KVM_GET_API_VERSION
  -> KVM_CREATE_VM
  -> KVM_SET_USER_MEMORY_REGION
  -> KVM_CREATE_VCPU
  -> KVM_GET_VCPU_MMAP_SIZE
  -> mmap struct kvm_run
  -> KVM_RUN
  -> KVM_EXIT_HLT / KVM_EXIT_IO
```

第一版只需要足够小：分配一段 guest memory，写入几条 x86 机器码，创建一个 vCPU，设置寄存器，调用 `KVM_RUN`，最后能看到 `KVM_EXIT_HLT` 或 `KVM_EXIT_IO` 即可。

到那个时候，`/dev/kvm`、VM fd、vCPU fd、memslot 和 `KVM_RUN` 就不再只是 QEMU `strace` 里的几行输出，而会变成自己程序里的对象和控制流。

## Inspired By

{{% inspire %}}

+ [Codex](https://openai.com/codex/)
+ [QEMU Documentation](https://www.qemu.org/docs/master/)
+ [QEMU System Emulation](https://www.qemu.org/docs/master/system/index.html)
+ [QEMU Invocation](https://www.qemu.org/docs/master/system/invocation.html)
+ [QEMU Network Emulation](https://www.qemu.org/docs/master/system/devices/net.html)
+ [QEMU Disk Images](https://www.qemu.org/docs/master/system/images.html)
+ [KVM API](https://www.kernel.org/doc/html/latest/virt/kvm/api.html)

{{% /inspire %}}
