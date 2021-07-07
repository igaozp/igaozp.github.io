---
title: "IDEA 编译大项目报错解决"
date: 2021-07-07T20:53:10+08:00
draft: false
tags: ["Java", "IDEA"]
---

最近使用 IDEA 启动 Java 项目时发现在编译阶段出现如下的错误：

- `java.lang.OutOfMemoryError: GC overhead limit exceeded`
- `java.lang.InternalError`
- `java.lang.OutOfMemoryError: Java heap space`

出现上述的错误，通常在项目工程比较大的时候才会出现。此时只需要调整 IDEA 的设置，
将 **File | Settings | Build, Execution, Deployment | Compiler** 中的 
**Shared build process heap size(Mbytes)** 配置的阈值适当提高，问题通常便会解决。
