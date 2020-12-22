---
title: fastJSON坑
date: 2020-12-21 15:53:18
tags: fastJSON
---

##问题描述
&ensp;&ensp;&ensp;&ensp;在开发时遇到一个问题，A服务要通过Feign（Feign接口在公共模块C里）去访问B服务的接口返回一个包含某实体类的List集合（实际上是返回Map，List只是其中一部分），但是该实体类在除C以外的另一个
<!-- more -->
公共模块D，A和B都引入了D，但是C没有引入，所以直接通过List传输的路有点曲折，我选择将List转换成JSON字符串进行传输，于是现在返回的格式是Map<String,String>。
&ensp;&ensp;&ensp;&ensp;一切准备“妥当”之后就开始本地Debug跑流程了，在一系列操作之后，数据终于返回了，是我想要的数据，高兴了一分钟。接下来做的就是获取Map里的数据，反序列化，对第一个元素反序列化后进行一系列操作后无误，接着进行对第二个数据进行相同操作，但是就这时候抛出了个类型转换异常，排查了半天发现了我反序列化使用的是
``` bash
JSONObject.parseObject(listBaseResult.get("XXX"), List.class);
```
用这个方法返回的List是一个Object类型的集合，虽然可以使用任意类型的List去接收，但是在后续操作中就会报异常。
然而因为第一个元素String类型的集合，这样操作也没问题。