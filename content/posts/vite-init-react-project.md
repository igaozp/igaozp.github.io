---
title: "《关于使用 create-react-app 初始化 React 项目屡遭失败愤而投入 Vite 怀抱的那件事》"
date: 2021-03-14T11:00:10+08:00
draft: false
tags: ["React", "Vite"]
---

最近在学习 React 时，最开始使用官方的 create-react-app 工具进行项目的初始化，完成工具的安装后进行项目初始化时遇到了一些问题。

<!--more-->

使用 crp 创建项目时遇到了关于 node-gyp 的错误，node-gyp 是用来构建跟平台相关的原生模块，在 Windows 下需要额外安装 windows-build-tools 之类
的工具，安装这些工具需要耗费大量的磁盘空间，遂放弃在 Windows 下使用 crp，进而在 WSL 环境中使用。

在 Linux 环境中使用 crp 也需要使用 node-gyp 来进行构建原生模块，只不过是使用的 make 工具，所以安装 make 相关的依赖。本以为能够继续顺利地使用 crp 完成项目地初始化，
结果遇到了 `gyp ERR! stack Error: `make` failed with exit code: 2` 及 `gyp: Call to 'pkg-config pixman-1 --libs' returned exit status 127 while in binding.gyp. while trying to load binding.gyp` 之类的
问题，在查找了几种解决方案并尝试之后均无果而终。

决定放弃使用 crp 初始化项目寻找其他的替代方案，突然想到 Vite 最近已经更新 2.0 了，看了下相关文档发现除支持生成 Vue 项目外还支持 React 项目。

全局安装 Vite 工具
```
npm install -g vite
```

使用相关的模板初始化项目，这里使用 React 项目的模板，basic 可替换为任意的项目名称
```
npm init vite basic --template react
```

使用 Vite 顺利完成 React 项目的初始化，但运行项目时遇到了 esbuild 执行文件丢失的情况导致无法启动项目，可能是使用 npm 镜像下载依赖导致的，
可以使用 node 手动执行 `\node_modules\esbuild` 中的 install.js 来下载 esbuild 的可执行文件，随后项目可正常启动。

完。
