---

title: git-常用设置
date: 2019-04-04 13:15:05

---

## git 设置代理

``` git
$ git config --global http.proxy http://proxyUsername:proxyPassword@proxy.server.com:port
```
<!-- more -->
例如设置本地无认证的 socks5 代理，端口 1080
``` git
$ git config --global http.proxy socks5://127.0.0.1:1080
```

## git 设置 ssh

1. 设置用户信息
``` git
$ git config --global user.name "username"
$ git config --global user.email "your@email.address"
```

2. 生成 ssh 密匙对
``` git
$ ssh-keygen -t rsa -C "your@email.address"
```

3. 将公钥复制到 github 的 setting 里

4. 测试
``` shell
$ ssh -T git@github.com
```

## 通过 ssh 协议进行 push 等操作

### `git clone` 时直接使用 git@仓库

### 修改 https 为 ssh
1. 查看当前远程仓库
``` git
$ git remote -v
> origin  https://github.com/USERNAME/REPOSITORY.git (fetch)
> origin  https://github.com/USERNAME/REPOSITORY.git (push)
```
2. 修改远程仓库
``` git
$ git remote set-url origin git@github.com:USERNAME/REPOSITORY.git
```
3. 再次查看
``` git
$ git remote -v
> origin  git@github.com:USERNAME/REPOSITORY.git (fetch)
> origin  git@github.com:USERNAME/REPOSITORY.git (push)
```
*以上默认远程为 github*
