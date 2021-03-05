# 执行政策

池策略服务器以每个IP为单位收集若干统计信息。有两个选项。`iptables+ipset`或简单的应用级禁止。禁令默认为禁用。

## 防火墙禁止

首先你需要配置你的防火墙来使用`ipset`，请阅读[本文](https://wiki.archlinux.org/index.php/Ipset)。

在 "policy "部分指定`ipset`的名称，用于禁止。超时参数(以秒为单位)将被传递给这个`ipset`。Stratum将使用`os/exec`命令，如`sudo ipset add banlist x.x.x.x 1800`来禁止，所以你必须正确配置`sudo`，并确保你的系统永远不会询问密码。

例如 `/etc/sudoers.d/pool` 其中 `pool`是池子运行的用户名。

    pool ALL=NOPASSWD。/sbin/ipset

如果你需要一些简单的东西，只需将`ipset`名称设置为空白字符串，就会使用简单的应用程序级别的禁止。

## 限制

在一些奇怪的情况下，你可以执行限制，以防止连接泛滥到地层，有初始设置。`limit`和`limitJump`。策略服务器将在每个有效的共享提交上增加每个IP地址的允许连接数。在分层启动后，分层在指定的 "宽限期 "内不会执行该策略。