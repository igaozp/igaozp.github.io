---
title: "使用 GitHub Actions 自动构建部署 Hugo 静态页面"
date: 2020-08-30T17:05:10+08:00
draft: false
tags: ["GitHub Actions", "Hugo", "CI/CD"]
---

`GitHub Actions` 可以实现应用的持续集成、持续部署，可以与 `GitHub Pages` 整合实现静态网页的自动构建和部署。

本博客使用的主题需要使用 `Node.js` 环境完成前端资源的构建，同时需要 `Hugo` 环境完成静态页面的生成，因此需要多语言环境分步完成应用的构建。

在 `GitHub Actions` 配置使用多个构建步骤，前几个步骤用来实现多个语言框架环境的准备，后面的步骤完成应用的构建，最后完成应用的发布。

下面 `GitHub Actions` 工作流程实现 `Hugo` 页面构建的详细配置文件：

```yml
# This is a basic workflow to help you get started with Actions

name: GitHub Pages

# Controls when the action will run. Triggers the workflow on push or pull request
# events but only for the master branch
on:
  push:
    branches: 
      - master

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v2

      - name: Setup Node.js environment
        uses: actions/setup-node@v1.4.3
      
      - name: Setup Go environment
        uses: actions/setup-go@v2.1.2
      
      - name: Hugo setup
        uses: peaceiris/actions-hugo@v2.4.12
    
      - name: Install Dependencies
        run: npm install --dev @babel/cli @babel/core @babel/preset-env browserslist clipboard cssnano postcss-cli postcss-import postcss-mixins postcss-nested postcss-preset-env postcss-url

      - name: Build
        run:  |
          hugo -v --gc --minify
          echo 'andornot.xyz' > public/CNAME
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_branch: gh-pages
          publish_dir: ./public
          force_orphan: true
```

按照配置，只有当 `master` 分支产生提交时才会触发构建操作，构建完成后将静态网页资源发布到 `gh-pages` 分支。

最初尝试将 `GitHub Actions` 配置文件放在非 `master` 分支中，例如 `blog` 分支，预想当 `blog` 分支提交新内容时触发构建，
并将构建完成的资源发布到 `master` 分支，但尝试了一下似乎配置放在非 `master` 分支无法触发构建操作。

因此最后将构建静态页面需要的文章、主题等资源与 `GitHub Actions` 配置一同放在 `master` 分支，
单独使用并设置 `gh-pages` 分支作为构建完成后资源发布和 `GitHub Pages` 页面访问的分支。

这样便完成了整个 `Hugo` 应用的自动构建和部署，无需在本地环境进行配置环境生成静态页面，只需将新写的内容提交到 `master` 分支即可。
