---
title: "Linux 开启 TCP BBR 拥塞控制算法"
date: 2018-10-22T12:05:10+08:00
draft: false
tags: ["Linux", "TCP"]
---

## 前提

要想开启 TCP BBR，需要 Linux kernel 4.9+ ，如果内核版本过低的话需要升级内核，本教程以 Ubuntu 系统为例。

### 开启 TCP BBR

执行 `lsmod | grep bbr`，如果结果中没有 `tcp_bbr` 的话就先执行

```bash
modprobe tcp_bbr
echo "tcp_bbr" >> /etc/modules-load.d/modules.conf
```

执行

```bash
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
```

保存生效

```bash
sysctl -p
```

执行

```bash
sysctl net.ipv4.tcp_available_congestion_control
sysctl net.ipv4.tcp_congestion_control
```

如果结果都有 `bbr`， 则证明你的内核已开启 `bbr`
执行 `lsmod | grep bbr`， 看到有 `tcp_bbr` 模块即说明 `bbr` 已启动
