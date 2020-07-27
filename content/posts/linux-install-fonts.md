---
title: "Linux 字体极简安装教程"
date: 2017-07-25T12:05:10+08:00
draft: false
tags: ["Linux", "Font"]
---

创建字体目录

```bash
sudo mkdir /usr/share/fonts/oh-my-fonts`
```

将下载的字体复制到相应的目录

```bash
sudo cp 存放字体的目录/*  /usr/share/fonts/oh-my-fonts
```

修改字体目录权限

```bash
sudo chmod 644 /usr/share/fonts/oh-my-fonts/*
cd /usr/share/fonts/oh-my-fonts
```

建立字体缓存

```bash
sudo mkfontscale
sudo mkfontdir
sudo fc-cache -fv
```
