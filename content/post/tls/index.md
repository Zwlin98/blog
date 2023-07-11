---
title: "TLS 指北（一）TLS 基础"
date: 2022-06-10T14:26:04+08:00
draft: true
---

最近在准备秋招，在此记录一下 TLS 以及 https，很久之前写过一篇用 certbot 部署 https 网站的笔记。当时没有理论基础，对 https 一知半解，现在总结一下有关的知识，并记录下我自己折腾 https 的常用方案。
因为计划包含的内容比较多，可能会包括多篇文章，计划包括内容如下：
+ TLS 基础
    + 概念
    + PKI (Public Key Infrastructure)
+ TLS 握手
    + TLSv1.2 握手
    + TLSv1.3 握手
+ TLS 抓包
    + wireshark 抓包
    + wireshark 解密 https
+ TLS/HTTPS 的部署
    + acme.sh 和 certbot
    + let's encryption
    + 自建 CA
+ 其他常见技术/协议
    + ACME
    + DNSSEC
    + HSTS
    + OCSP
    + OCSP Stapling

## TLS 基础
### 概念
传输层安全性协议 (英语：Transport Layer Security，缩写：TLS) 及其前身安全套接层 (英语：Secure Sockets Layer，缩写：SSL) 是一种安全协议，目的是为互联网通信提供安全及数据完整性保障。因此可以称之为 SSL/TLS，但是我个人还是喜欢简称为 TLS。

TLS 历史悠久，目前最广泛使用的协议是 TLS1.3 和 TLS1.2，之前的版本都已经被认为是不安全的，因此，本文的内容主要基于 TLS1.2 和 TLS1.3。

TLS 协议的优势是与高层的应用层协议 (如 HTTP、FTP、Telnet 等) 无耦合。**因此 https 实际上是 HTTP + TLS**，应用层协议能透明地运行在 TLS 协议之上，由 TLS 协议进行创建加密通道需要的协商和认证。应用层协议传送的数据在通过 TLS 协议时都会被加密，从而保证通信的私密性。
### 密码基础
有幸读过《密码朋克》，认识到了解基础的现代密码学知识，对于普通人来说也是非常重要的。因此我十分建议有兴趣的人阅读这本书。实际上，现在区块链技术就是现代密码学，可以说各种涉及到安全，保密，隐私的技术的基础，都是现代密码学。 
TLS 的基础同样是现代密码学，对密码学大致需要了解的内容包括：
+ 哈希函数
+ 非对称加密
+ 对称加密

可以阅读 [Practical Cryptography for Developers](https://cryptobook.nakov.com/) 了解其中的知识。
### 如何做到安全
通常认为，如果通信过程具备了四个特性，就可以认为是“安全”的。
这四个特性是：机密性、完整性，身份认证和不可抵赖。
+ 机密性
    > 机密性 (Secrecy/Confidentiality) 是指对数据的“保密”，
      只能由可信的人访问，对其他人是不可见的“秘密”，
      简单来说就是不能让不相关的人看到不该看的东西。
+ 完整性
    > 完整性 (Integrity，也叫一致性) 是指数据在传输过程中没有被篡改，
      不多也不少，“完完整整”地保持着原状。
+ 身份认证
    > 身份认证 (Authentication) 是指确认对方的真实身份，
      也就是“证明你真的是你”，保证消息只能发送给可信的人。
+ 不可抵赖
    > 不可抵赖 (Non-repudiation/Undeniable)，
      意思是不能否认已经发生过的行为，不能“说话不算数” “耍赖皮”。

而 TLS 正是利用密码学中的相关知识，来达到这四个目标，其具体如何做到会在之后的介绍中提到。

{{% inspire %}}
## Inspired By
+ https://github.com/nakov/Practical-Cryptography-for-Developers-Book
+ https://en.wikipedia.org/wiki/Transport_Layer_Security

{{% /inspire %}}
