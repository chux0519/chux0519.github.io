+++
title = "密码管理器"
date = 2021-10-03T11:25:15+08:00
+++

随着使用的软件、服务越来越多，我们需要记忆的密码也越变越多，为了安全性，应该避免使用弱密码或重复的密码，于是我之前选择了 1password 这个服务，前两个月我的订阅刚刚过期，但我又不再想续订了，我认为这个产品我没有花得很值，于是调研了一些替代品，我的需求是，可以同步、可以方便的在多端使用。

最终看到了 https://www.passwordstore.org/ 这个项目，我认为完美的契合我的需求，额外的，它足够轻量，在 PC 端也是命令行优先，功能也足够用，第三方适配的 app 也十分简洁，这里做一次记录和推荐。

<!-- more -->

## 安装

从各个系统的包管理器安装即可，比如 pacman / brew / apt 等等

> brew install pass

## 同步

可以直接通过 git repo 同步，可以自建，也可以用服务，我使用 github pricate repo 同步。

这里涉及到稍微麻烦的一点是 ssh key 的同步和 gpg key 的同步。

多台 PC 间，我直接通过内网拷贝，然后 gpg 导入，设计到几个命令

导出：

1. 查看 ID

> gpg -K --keyid-format=long

2. 导出公钥

> gpg -a --export XXXX

3. 导出私钥

> gpg -a --export-secret-key XXX

然后到另一台 PC 进行导入

> cat pub.k | gpg --import
>
> cat sec.k | gpg --allow-secret-key-import --import

在 mac 上可能会出现问题比如:

```
gpg: error building skey array: Inappropriate ioctl for device
gpg: error reading '[stdin]': Inappropriate ioctl for device
gpg: import from '[stdin]' failed: Inappropriate ioctl for device
gpg: Total number processed: 0
gpg:              unchanged: 1
gpg:       secret keys read: 1
```

运行命令前，设置 `export GPG_TTY=$(tty)` 即可

移动端同样通过内网的 http 传输即可

另外还有一个选择，就是利用 keybase 这款软件，将 gpg 公私钥都上传，然后同步。
