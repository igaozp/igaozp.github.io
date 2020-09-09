---
title: "GitHub Star 仓库管理利器 - stargazed"
date: 2020-09-09T21:22:10+08:00
draft: false
tags: ["GitHub", "stargazed"]
---

个人在 GitHub 收藏了非常多的开源项目仓库，到目前为止收藏的项目大概 3000+，这些项目检索起来非常繁琐，导致一些收藏的项目由于检索效率的问题没有利用起来。

所幸无意间发现了一个名为 [stargazed](https://github.com/abhijithvijayan/stargazed) 工具，可以将 GitHub 用户收藏过的项目，进行分类并输出到 Markdown 文件中。
stargazed 会将项目按语言分类并生成目录，方便跳转查看不同编程语言的项目，并且生成的项目条目包括项目名称、项目地址、项目描述、作者等信息，可以非常方便地检索。

### 如何使用

该工具需要用户提供提供一个 GitHub 仓库，用来存储生成的 Markdown 文件。支持通过 GitHub 的 Personal access tokens 来自动创建仓库，同时支持 GitHub Actions 来实现定时更新文件信息。

使用该工具需要安装 Node.js 环境，安装命令：

```bash
npm install -g stargazed
```

使用命令如下，需要将 GITHUB_USERNAME 和 GITHUB-TOKEN 替换为自己的信息，运行成功后会在自动在 awesome-stars 仓库中生成文件。

```bash
stargazed --username GITHUB_USERNAME --token "GITHUB-TOKEN" --repository "awesome-stars"  --sort --workflow
```

上述命令会自动创建 GitHub Actions 配置文件，会定时更新生成的文件，即第一次执行后后续可以无需再次介入。

关于项目的其他信息可以访问 https://github.com/abhijithvijayan/stargazed 进行查看。