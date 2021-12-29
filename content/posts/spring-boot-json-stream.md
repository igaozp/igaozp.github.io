---
title: "Spring Boot 中实现服务端主动数据推送"
date: 2021-12-29T10:45:10+08:00
draft: false
tags: ["Java", "Spring Boot", "ndjson", "SSE"]
---

在日常开发中通常使用 `WebSocket` 实现服务端的主动地数据推送，这里简单介绍两种其他的数据推送方案，`ndjson` 和 `SSE`。

<!--more-->

## ndjson

**ndjson** 全称 **Newline Delimited JSON**，即使用换行符分割的 JSON，同时能够保证每一行的内容都是一个完整的 JSON。这里使用 ndjson 并不是因为它可以实现服务端数据推送，而是凭借 **ndjson** 的数据格式的特性，作为服务端的 JSON 数据流的数据输出，来更好地帮助我们实现数据推送的效果。

这里使用 **Spring Boot** 作为基础的服务端框架，简单演示一下相关代码及功能。首先要确保项目依赖中包含 `spring-boot-starter-webflux` 依赖包。

代码示例

```java
@GetMapping(value = "ndjson", produces = MediaType.APPLICATION_NDJSON_VALUE)
Flux<Data> ndjson() {
    return dataStream();
}

private Flux<Data> dataStream() {
    return Flux.interval(Duration.ofSeconds(1))
            .take(5)
            .map(i -> new Data(i, Instant.now()));
}
```

代码中定义了一个私有方法用来生成数据流，效果是每隔 1 秒生成一条数据。在 `@GetMapping`中指定了 `MediaType.APPLICATION_NDJSON_VALUE` 作为数据的响应格式。

## SSE

**SSE** 全称 **Server-sent events**，即服务器发送事件，服务端可使用 **SSE** 生成相应的数据流。

代码示例，同样需要 **WebFlux** 支持。

```java
@GetMapping(value = "sse", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
Flux<Data> sse() {
    return dataStream();
}

private Flux<Data> dataStream() {
   return Flux.interval(Duration.ofSeconds(1))
            .take(5)
            .map(i -> new Data(i, Instant.now()));
}
```

与 **ndjson** 方案唯一的不同是，数据响应的格式替换为了`MediaType.TEXT_EVENT_STREAM_VALUE`。

## 总结

虽然 **ndjson** 和 **SSE** 方案都可以实现服务端的数据推送，但这两种的数据通信都是单向通信，只能从服务端到客户端，无法实现双向通信，需要根据自身的场景需要酌情考虑使用。

项目的完整代码可参考 [StreamJsonController](https://github.com/igaozp/surabaya/blob/f8cc3573759ac22f63801e54edde563f6706a1ec/spring/src/main/java/xyz/andornot/controller/StreamJsonController.java)。

## 参考内容

[https://nurkiewicz.com/2021/08/json-streaming-in-webflux.html](https://nurkiewicz.com/2021/08/json-streaming-in-webflux.html)

[https://www.ruanyifeng.com/blog/2017/05/server-sent_events.html](https://www.ruanyifeng.com/blog/2017/05/server-sent_events.html)
