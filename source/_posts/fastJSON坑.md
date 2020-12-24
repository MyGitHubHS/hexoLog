---
title: fastJSON坑
date: 2020-12-21 15:53:18
tags: fastJSON
---

## 问题描述
&ensp;&ensp;&ensp;&ensp;在开发时遇到一个问题，A服务要通过Feign（Feign接口在公共模块C里）去访问B服务的接口返回一个包含某实体类的List集合（实际上是返回Map，List只是其中一部分），但是该实体类在除C以外的另一个<!-- more -->公共模块D，A和B都引入了D，但是C没有引入，所以直接通过List传输的路有点曲折，我选择将List转换成JSON字符串进行传输，于是现在返回的格式是Map<String,String>。
&ensp;&ensp;&ensp;&ensp;一切准备“妥当”之后就开始本地Debug跑流程了，在一系列操作之后，数据终于返回了，是我想要的数据，高兴了一分钟。接下来做的就是获取Map里的数据，反序列化，对第一个元素反序列化后进行一系列操作后无误，接着进行对第二个数据进行相同操作，但是就这时候抛出了个类型转换异常，排查了半天发现了我反序列化使用的是
``` bash
JSONObject.parseObject(listBaseResult.get("XXX"), List.class);
```
用这个方法返回的List并没有指明是什么类型，可以使用任意类型的List去接收，但是在后续操作中就会报异常。本着先解决问题再深究问题的原因，我开始尝试这其他反序列化方法，最后使用
```bash
JSON.parseArray(listBaseResult.get("XXX"), XX.class);
```
就解决问题了。
&ensp;&ensp;&ensp;&ensp;问题解决了，去分析原因，去看源码发现我们使用JSONObject.parseObject()方法时，会只适用于反序列化一般的引用类型、基本类型的包装类型的集合以及一般的引用类型的数组（这里String也可以看作基本类型的包装类型），但是不适用于List这样需要指明泛型的数据结构，如果返回List这样的集合的时候，内部元素会被反序列化JSONObject类型，JSONObject类型对于Map的一种实现，所以如果使用JSONObject.parseObject()去反序列化成List集合的话，它内部实际上是将我们内部的元素属性名和值以key-value结构存储于JSONObject内。