---
title: "Create your own Overlay Network"
date: 2023-07-30T19:05:01+08:00
draft: false
---

## Intro

月余之前的一篇博客中，我没有回答如何访问在 NAT 之下的 LXC 容器的问题，其实这个问题就是一个非常典型的内网穿透问题，内网穿透有许多经典方案，例如大名鼎鼎的 *[frp](https://github.com/fatedier/frp)*, 或者直接在路由器上配置端口转发，这些以端口为作为穿透粒度的方案并不能令我满意，它们既不灵活，也不优雅。
{{% btw %}}
这里的 VPN 不是指用来翻墙的某些代理软件，而是指虚拟专用网络，即 Virtual Private Network。
{{% /btw %}}
而 Overlay Network 实际上就是我认为解决内网穿透的完美方案，我使用 *[tailscale](https://tailscale.com/)* 作为我的 Overlay Network 的组网工具。虽然传统的 VPN 也可以视作一种 Overlay Network，但是 Tailscale 很大程度上解决了传统 VPN 方案存在的单机瓶颈问题，即 VPN 服务器，这个服务器往往是一个公网服务器，而国内服务器的带宽和流量限制使得的此类软件体验并不算好。

tailscale 则是通过 Wireguard 构建了一个 [Full Mesh](https://en.wikipedia.org/wiki/Mesh_networking) 的 Overlay Network，使得你的所有设备可以在同一个私有网络（tailscale network，以下简称 tsnet）中，同时 tailscale 也提供了非常好的跨平台支持，例如 Windows, Linux, macOS, iOS, Android 等，这些平台上的设备都可以加入到同一个 tsnet 中，即使没有提供对应平台的客户端，也可以通过宣告路由的方式加入到网络。而 tailscale 最杀手锏的功能就是它提供了 *[非常强大的 NAT 穿透能力](https://tailscale.com/blog/how-nat-traversal-works/)*，它会尽最大可能让你的节点之间直接通信，而不是通过服务器中转，最低程度的降低了对中心服务器带宽的依赖。

那么，什么是 Overlay Network？

{{% btw %}}
此为 ChatGPT 给出的解释，我觉得非常好，因此直接引用。
{{% /btw %}}

> Overlay Network 是一种在底层网络之上创建的虚拟网络。它利用底层网络的基础设施和通信能力，在其上构建一层或多层的虚拟网络，以满足特定的需求和目标。

> 在 Overlay Network 中，网络节点通过逻辑连接而不是物理连接进行通信。这些逻辑连接可以是虚拟链路、隧道或其他技术实现的通信通道。Overlay Network 可以在现有的底层网络之上构建，而无需对底层网络进行修改或更改。

>   Overlay Network 的一个关键特点是它可以提供一种虚拟化的网络环境，使得节点之间的通信看起来像是在同一个私有网络中进行的，而不受底层网络的限制。这使得 Overlay 网络可以用于实现各种功能和应用，如虚拟专用网络（VPN）、容器编排、软件定义网络（SDN）等。

通过 tailscale 构建出属于你自己的 Overlay Network 之后，便可以在这个网络中安全且自由的部署各种服务，并且可以在所有加入 tsnet 设备上访问，不需要任何端口转发，也不需要任何公网 IP。

你可以把你的 NAS，树莓派，手机，都加入到这个网络中，这样你就可以在任何地方访问你的 NAS，或者在任何地方访问你的树莓派，而且这些访问都是安全的。心动不如行动，赶紧来试试吧！

## Headscale

tailscale 作为一个非常优秀的软件，其免费版的功能也足够，但是 tailscale 的服务器以及其作为 NAT 穿透失败的中转服务器从中国大陆连接并不稳定，并且 tailscale 的登录认证方式我也不大喜欢，虽然 tailscale 有其[理由](https://tailscale.com/blog/passkeys/),但需要依赖一些大型的 [Identity Provider](https://en.wikipedia.org/wiki/Identity_provider) 的方式，我还是喜欢不起来。

幸运的是，开源的 tailscale 服务器实现 [Headscale](https://github.com/juanfont/headscale) 解决了这些问题，它可以让你自己搭建一个 tailscale 服务器，而且可以使用自己的域名，自己的证书，自己的数据库，自己的存储，自己的日志，自己的监控，自己的一切，这对于我这样一个 *self-hosting* 狂热者来说，简直是完美。

Headscale 的部署比较简单，其配置文件有着比较好的文档，搭配上 tailscale 上官方的文档，足以使得感兴趣的朋友成功搭建一个属于自己的 Overlay Network（或许可以称之为 hsnet，笑）。我的建议是，在配置 headscale 时，DNS 服务器和 [MagicDNS](https://tailscale.com/kb/1081/magicdns/) 功能一定要打开，这样 hsnet 将会接管加入这个网络的设备的 DNS，这样你就可以在 hsnet 中使用主机名访问设备，而不是 IP，这样更加方便。

## DNS
{{% btw %}}
我之后可能会写一篇文章讲一下 DNS，但是现在先简单介绍一下。
{{% /btw %}}
在 Headscale 接管你的 DNS 时，需要指定一个上游的 DNS 服务器，这个 DNS 服务器将会被 headscale 用来解析在 hsnet 主机名之外的域名，例如 google.com, baidu.com 等。这个 DNS 服务器可以是你自己搭建的，也可以是公共的。但是中国大陆的公共 DNS 或多或少都会被污染，因此我建议使用自己搭建一个无污染的 DNS 服务器，作为 hsnet 的上游 DNS 服务器，这样你在 hsnet 中的设备的 DNS 查询既不会被污染，也不会被劫持，同时也可以在 hsnet 中使用主机名访问设备。

## DERP

Tailscale 在全球范围内运行 DERP 中继服务器，作为在 NAT 穿透期间将 Tailscale 节点点对点连接为一个辅助通道，并作为防止 NAT 穿透失败且无法建立直接连接的备用选项。

除了使用 Tailscale 官方 DERP 服务器之外，我们可以[运行自己的 DERP 服务器](https://tailscale.com/kb/1118/custom-derp-servers/)。从而降低延迟（特别是在中国大陆），提高稳定性。

事实上，Headscale 提供了一个内嵌的 DERP Server，配置起来也非常简单，但是当你的 Headscale 机器带宽有限，也可以在其他机器上建立自己的 DERP Server，我也提供了一个我自己正在使用的更为简单的配置 DERP 的方案，简化了 DERP 服务的搭建，有兴趣的可以看看。

## Tricks

* headscale 在管理端可以对设备的 MagicName 进行修改，可以将主机名修改的更为简短，或者使得可能相同的主机名在不同设备之间不会冲突。
* headscale 可以使用 pre-auth key 的方式加入网络，可以更方便的让主机加入网络，而不需要在每个设备上都登录一遍。
* 有兴趣的同学可以翻翻 tailscale 的文档，里面有许多有趣的功能，例如 MagicDNS, ExitNode 等，这些功能都可以让你的网络更加方便，更加安全。

{{% inspire %}}

## Inspired by

+ [Compare · Tailscale](https://tailscale.com/compare/)
+ [How NAT traversal works · Tailscale](https://tailscale.com/blog/how-nat-traversal-works/)
+ [[译] NAT 穿透是如何工作的：技术原理及企业级实践（Tailscale, 2020）](https://arthurchiao.art/blog/how-nat-traversal-works-zh/)
+ [juanfont/headscale: An open source, self-hosted implementation of the Tailscale control server](https://github.com/juanfont/headscale)
+ [DERP Servers](https://tailscale.com/kb/1232/derp-servers/)

{{% /inspire %}}