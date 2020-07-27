---
title: "使用 Multimap 对集合按属性进行分组"
date: 2020-07-25T12:05:10+08:00
draft: false
tags: ["Java", "Guava"]
---

## 问题描述

存在一种场景，给定一个数据集合，根据集合元素中的某个字段属性对整个集合数据进行分组。

例如有如下的数据列表（省略非关键信息）：

```json
[
    {
        "id": 1,
        "createTime": "2020-07-23 21:01:23"
    },
    {
        "id": 2,
        "createTime": "2020-06-23 21:01:23"
    },
    {
        "id": 3,
        "createTime": "2020-05-23 21:01:23"
    }
]
```

想要根据`createTime`创建时间的月份，对数据进行分组，得到类似这样的结构：

```json
{
    "JULY": [
        {
            "id": 1,
            "createTime": "2020-07-23 21:01:23"
        }
    ],
    "JUNE": [
        {
            "id": 2,
            "createTime": "2020-06-23 21:01:23"
        }
    ],
    "MAY": [
        {
            "id": 3,
            "createTime": "2020-05-23 21:01:23"
        }
    ]
}
```

## 解决方案

可以使用 `Guava` 提供的 `Multimap` 方便的进行处理，`Multimap` 可以支持同一个 `key` 放入多个 `value`。 因此针对上述场景，将月份数据作为 `Map` 的 `key`，每个月份对应的数据元素作为 `value`，便可得到对应的分组结构。参考代码如下：

```java
import com.google.common.base.Function;
import com.google.common.collect.Multimap;
import com.google.common.collect.Multimaps;
import org.checkerframework.checker.nullness.qual.Nullable;

import java.time.LocalDateTime;
import java.time.Month;
import java.util.ArrayList;
import java.util.Collection;
import java.util.Collections;

public class MultimapExample {

    static class Data {
        private Integer id;

        private LocalDateTime createTime;

        public Data(Integer id, LocalDateTime createTime) {
            this.id = id;
            this.createTime = createTime;
        }

        public void setId(Integer id) {
            this.id = id;
        }

        public void setCreateTime(LocalDateTime createTime) {
            this.createTime = createTime;
        }

        public Integer getId() {
            return id;
        }

        public LocalDateTime getCreateTime() {
            return createTime;
        }

        @Override
        public String toString() {
            return "id: " + id + ", createTime: " + createTime + "\n" ;
        }
    }

    // test with Java 14
    public static void main(String[] args) {
        // generate random data
        Collection<Data> collection = new ArrayList<>(Collections.emptyList());
        for (int i = 0; i < 10; i++) {
            collection.add(new Data(i, LocalDateTime.now().plusMonths(i)));
        }
        System.out.println(collection);

        // Data convert to Month function
        java.util.function.Function<Data, @Nullable Month> function = new Function<>() {
            @Nullable
            @Override
            public Month apply(@Nullable Data data) {
                if (data == null) {
                    return null;
                }
                return data.getCreateTime().getMonth();
            }
        };
        
        Multimap<Month, Data> result = Multimaps.index(collection, function::apply);
        System.out.println(result.asMap());
    }
}
```

代码中核心的部分，创建了一个 `Function` 用于获取数据元素对应的分组的元素（输入 `Data` 数据获取对应的分组月份），使用 `Multimaps.index` 方法生成分组数据。 

如果想要对生成的分组数据进行展示、输出等操作，需要调用 `asMap` 方法将 `Multimap` 转换成朴素的 `Java Map` 进行展示输出。
