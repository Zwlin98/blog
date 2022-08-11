---

title: 连接树莓派
date: 2019-12-26 18:39:24

---
*本文的使用树莓派是Raspberry Pi 4B 2GB版本*
## 方法1 使用屏幕
<!-- more -->
使用屏幕的办法十分简单，将线连接好，按照官方教程即可。
## 方法2 使用SSH
### 局域网连接方法
待更新，暂时未尝试
### 网线直连方法
在需要携带树莓派的情况下，此方法较为推荐，仅需要一根电源线和网线即可。本方法在我的笔记本上测试成功。

#### 前提条件:
- 保证树莓派已经启动了SSH
- 树莓派的dhcp默认开启(通常不用管)
- 需要一定的计算机网络基础知识

#### 具体操作:

1. 设置wifi的那张网卡共享网络给有线网卡
2. 先不要设置RaspberryPi的静态IP，通过`arp -a`获得树莓派的IP
3. SSH到树莓派上设置静态IP
修改dhcpcd.conf,此文件有着很好的注释，可以参考，文件尾部有着设置静态IP的方法
``` shell 
$ sudo vim /etc/dhcpcd.conf
# 在文件尾部按照注释追加
interface eth0
static ip_address=192.168.66.66/24
static routers=192.168.66.1
static domain_name_servers=192.168.66.1
``` 
4. 设置有限网卡静态IP，可以从上面的配置看出，我把有线网卡的IP设为`192.168.66.1`，而树莓派的静态IP为`192.168.66.1`
5. 重启树莓派,ssh连接`192.168.66.66`即可

网上教程很多，这里不多赘述，之所以第一步要共享，是因为要让树莓派能够访问外网。如果不需要树莓派访问外网，那么就不需要共享网络。

## 方法3 VNC连接

按照[官网教程](https://www.raspberrypi.org/documentation/remote-access/vnc/README.md)， 我遇到一个`can not currently show the desktop`的错误，根据这个[视频](https://www.youtube.com/watch?v=uFCZDuOWR3Y)解决了。有同样问题的可以试试。

## 其他方法暂无需求

请参考[官方文档](https://www.raspberrypi.org/documentation/remote-access/)

![Raspberry Pi 4B](https://blog-1300571114.cos.ap-shanghai.myqcloud.com/hero-shot.png)







