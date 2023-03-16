---
title: ArchLinux 安装过程记录
date: 2020-04-29 20:27:19
---


# 制作镜像

从官网下载镜像，验证签名，并制作 U 盘镜像，Windows 平台可通过 Rufus

# 从 U 盘启动

推荐 UEFI 模式

# 联网

1. 插网线情况下，通过 dhcpcd
```shell
dhcpcd
```
2. 通过无线网络
```bash
wifi-menu
```
3. 利用 ping 测试联网成功与否

# 更新系统时间
```shell
timedatectl set-ntp true
```

# 分区与格式化

```shell
fdisk /dev/sda
```

假设分区成 `/dev/sda1` 512M 作为引导分区，`/dev/sda2` 剩下全部空间作为根分区

## 挂载分区
```shell
mount /dev/sda2 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot 
```

# 更换镜像源

```shell
vim /etc/pacman.d/mirrorlist
```

把在文件第一行添加

```
Server = http://mirrors.aliyun.com/archlinux/$repo/os/$arch
```

# 安装基本的包

```shell
pacstrap /mnt base base-devel linux linux-firmware dhcpcd netctl
```

# 配置 fstab

```shell
genfstab -L /mnt >> /mnt/etc/fstab
```

# chroot

```shell
arch-chroot /mnt
```

# 设置时区并提前安装一些必要包

```shell
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
pacman -S vim dialog wpa_supplicant networkmanager netctl
```

# 设置语言和地区

```shell
vim /etc/locale.gen
```

在文件中找到 `zh_CN.UTF-8 UTF-8` `zh_HK.UTF-8 UTF-8` `zh_TW.UTF-8 UTF-8` `en_US.UTF-8 UTF-8` 这四行，去掉行首的#号，保存并退出。

```shell
locale-gen
vim /etc/locale.conf
```

在文件的第一行加入以下内容：

```
LANG=en_US.UTF-8
```

# 设置主机名

```shell
vim /etc/hostname #在文件的第一行输入你自己设定的一个myhostname
vim /etc/hosts
```
在文件末添加如下内容 (将 `myhostname` 替换成你自己设定的主机名)
```config
127.0.0.1	localhost
::1		localhost
127.0.1.1	myhostname.localdomain	myhostname
```

# 设置 root 密码与安装 Intel-ucode

```shell
passwd
pacman -S intel-ucode
```

# 安装 grub2

```shell
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub
grub-mkconfig -o /boot/grub/grub.cfg
```

# 取消挂载并重启

```shell
umount /mnt/boot
umount /mnt
reboot
```

