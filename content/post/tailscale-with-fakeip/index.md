---
title: "When tailscale subnet router meet fake IP"
date: 2024-07-27T10:06:27+08:00
---

我非常喜欢 [Tailscale](https://tailscale.com/) 这个组网工具，但在移动端设备上，通常只能开启一个 VPN。虽然 Tailscale 可以配置 [Exit-nodes](https://tailscale.com/kb/1103/exit-nodes)，但这样会接管全部流量，这显然不太理想。

Tailscale 能够自动接管设备的 DNS，并配置上游以支持其 Magic DNS 功能，同时，Tailscale 还支持 advertise-routes。利用这两个功能，再结合 FakeDNS (FakeIP) 功能，可以将国外网站的 DNS 指向 FakeDNS 的 CIDR，并将这个 CIDR 宣告到 Tailscale 的某个节点。然后，在这个节点上运行代理程序，就能实现所有 Tailscale 节点的透明代理。

我用 mosDNS + clash 实现了上述思路，目前运行体验非常好，配置并不算复杂，捋一遍两个工具的文档即可。Sliamb 的 HiFi 冲浪系列[文章](https://blog.03k.org/post/paopaogateway.html)也给了我很大的启发，整体思路和文章中基本一致，只不过文中是真正的局域网，而我的方案是构建于 Tailscale 之上的虚拟局域网 (笑。

整个方案核心思路就是利用 DNS + FakeIP 对流量进行分流，基于对 DNS 的更细粒度控制，你可以关闭 FakeDNS 的解析，也可以将不同的域名解析到不同的 FakeIP CIDR, 并为这些 FakeIP CIDR 配置不同的代理节点，进而做到流量控制，负载均衡等目的。

由于 Tailscale 只会路由 FakeIP 的 CIDR，普通的流量并不会受到影响，即使代理节点出问题，也只会影响海外网站的访问，因此该方案的可靠性也还不错。

{{% inspire %}}
## Inspired by
+ [Subnet routers and traffic relay nodes · Tailscale Docs](https://tailscale.com/kb/1019/subnets)
+ [使用 FakeIP 网关更灵活地分流——HiFi 冲浪系列 (二)](https://blog.03k.org/post/paopaogateway.html)
+ [IrineSistiana/mosdns: 一个 DNS 转发器](https://github.com/IrineSistiana/mosdns)
+ [MetaCubeX/mihomo](https://github.com/MetaCubeX/mihomo)
{{% /inspire %}}



