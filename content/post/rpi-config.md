---

title: 树莓派镜像安装及配置
date: 2020-01-12 13:38:06

---

## 镜像刷入

个人使用官方镜像，可以在[官网](https://www.raspberrypi.org/downloads/)下载到，推荐使用[官方的系统](https://www.raspberrypi.org/downloads/raspbian/)带桌面的但不带推荐软件，这样可以省去安装很多软件，同时如果不需要桌面，也可以考虑 lite 版。
<!--more-->
下载完镜像之后，可以使用官方推荐的 [balenaEtcher](https://www.balena.io/etcher/) 写到 SD 卡里，但是我这里使用的是 [Rufus](https://rufus.ie/) 是一个功能非常强大的，开源免费的快速制作 U 盘系统启动盘和格式化 USB 的实用小工具，实测发现，它也可以用来写树莓派的镜像。Rufus 非常好用，体积很小，只有几 M 而已，而且绿色免安装。我平时重装系统，制作系统镜像等都是使用 Rufus 来完成的。
![Rufus 界面](https://blog-1300571114.cos.ap-shanghai.myqcloud.com/2020-01-12_14-20-28.png)
选择 SD 卡之后，点开始即可。

## 开启 SSH
如果没有屏幕，那么就需要在局域网环境下，通过 ssh 连接树莓派，其他连接方式可以参考我之间的文章，开启 ssh 连接方式需要在写好的镜像的 boot 里创建一个名为 `ssh` 的空文件，登录树莓派的默认用户名和密码为 `pi` 和 `raspberry`，建议登录进去后修改密码。

## 系统初始设置

### 第一步修改软件源
修改成国内镜像源，我这里选择清华大学镜像源，修改方式具体参考[官方文档](https://mirrors.tuna.tsinghua.edu.cn/help/raspbian/)，这里只直接给出这个版本的 `raspbian(buster)` 的修改方法。

编辑 `/etc/apt/sources.list` 文件，删除原文件所有内容，用以下内容取代：
``` config
deb http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main non-free contrib
deb-src http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main non-free contrib
```
编辑 `/etc/apt/sources.list.d/raspi.list` 文件，删除原文件所有内容，用以下内容取代：
```
deb http://mirrors.tuna.tsinghua.edu.cn/raspberrypi/ buster main ui
```
### 安装代理
请根据自己的情况查找资料安装。
### 安装 zsh
依次执行下面的命令，在这里可能需要代理。
```shell
$ sudo apt install git zsh curl
# git设置代理
$ git config --global http.proxy socks5://127.0.0.1:1080
#无代理版
$ sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
#代理版
$ sh -c "$(curl -x socks5h://127.0.0.1:1080 -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```
## 常用软件配置记录

### Docker
针对树莓派版本
网络条件足够好的话，建议使用官方提供的一键脚本
```shell
$ curl -sSL https://get.docker.com | sh
```
先删除旧版本，并安装依赖
```shell
$ sudo apt remove docker docker-engine docker.io
$ sudo apt-get install apt-transport-https ca-certificates curl gnupg2 software-properties-common
```
添加国内软件源
```shell
$ echo "deb [arch=armhf] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/debian \
     $(lsb_release -cs) stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list
```
更新软件源并安装，一定要加上参数 `--no-install-recommends`，否则在树莓派 4 上会有一个依赖问题。
```shell
$ sudo apt-get update
$ sudo apt install --no-install-recommends docker-ce
```

### SAMBA
```shell
$ sudo apt-get update
$ sudo apt install samba
```
配置 samba
编辑 `/etc/samba/smb.conf`，其实该配置文件描述的已经很清楚，有兴趣的可以仔细看看，这里贴一个我的 samba 配置。
```config
[Media]
path = /media/yp
browseable = yes
valid users = pi
read only = no
writable = yes
guest ok = no
```
添加用户并配置密码
```shell
sudo smbpasswd -a pi
```
连接测试，测试连接可以有很多办法，在 win 系统可以在运行中键入如下命令测试，其中 `Media` 是根据自己的 samba 配置文件中写的设置的。
![连接测试](https://blog-1300571114.cos.ap-shanghai.myqcloud.com/2020-01-12_14-57-08.png)

### 挂载移动硬盘
首先找到自己的移动硬盘，可以根据大小，同时记录下名字，比如我这里是 `/dev/sda`。
```shell
sudo fdisk -l
```
如果选择一直插在树莓派上，建议格式化成 ext4 文件系统。
```shell
sudo mkfs.ext4 /dev/sda
```
挂载
```shell
sudo mount /dev/sda /media/yp
```
设置开机自动挂载，编辑 `/etc/fstab`，添加一行。
```shell
/dev/sda /media/yp ext4 defaults 0 1
```
### 配置 Aira2

### 配置 smartDNS

### 配置 Pihole
以上三个建议用 docker 部署，可以参考这些软件的官方说明和 `docker hub`。
