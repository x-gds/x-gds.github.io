---
layout: post
title:  "OkHttp & Gson"
date:   2018-01-16 14:15:34 +0800
categories: Android
---

## OkHttp

我们使用OKHttp处理网络请求时，除了拿`OkHttpClient`配置一些东西外，主要用到的就是`Request(RequestBody)`、`Response(ResponseBody)`以及代表中间过程的`Call`。这里不说请求，在响应（`Response`）中，我们真正用到的就是响应体（`ResponseBody`）。

`ResponseBody`提供了几个方法：`bytes()`返回一个`byte`数组，`byteStream()`返回一个`InputStream`，`charStream()`返回一个`Reader`，`string()`返回一个`String`，还有个`source()`返回一个`BufferedSource`。`BufferedSource`是`okio`中定义的，相比`InputStream`，它提供了一些更加便捷的方法，比如`readLong()`、`readString()`等等。

## Gson

Android中，将对象序列化成Json和将一段Json反序列化成Java对象，一般要么使用原生提供的`JsonObject`和`JsonArray`，要么使用第三方的库，比如Google的Gson，阿里的fastjson，square的moshi等等。这里使用Gson，一是根正苗红，二是使用人数最多，社区最活跃。

Gson使用起来非常简单，序列化就用`gson.toJson(Object)`，反序列化就用`gson.fromJson(String, Type)`。默认情况下，Gson在序列化/反序列化时，用的是反射，所以有一定性能损失。要想避免性能损失的话，有以下两种方式：

* `JsonSerializer`/`JsonDeserializer`：可以自定义序列化/反序列化过程，实现起来和原生的`org.json.JsonXXX`差不多。
* `TypeAdapter`：这种方式是基于流的，`TypeAdapter`提供了两个方法：`read(JsonReader)`和`write(JsonWrite, T)`，`JsonWriter`和`JsonReader`都提供了一些非常方便的方法。

这两种方式中，`JsonSerializer`和`JsonDeserializer`这一对儿作用不大，一般就是在序列化或者反序列化的过程中需要做一些自定义的操作，又不想改变现有代码的情况下，用这一对儿做一下代码扩展。而`TypeAdapter`由于是基于流的，所以可以和OKhttp的数据管道完美对接。

## OkHttp & Gson

一般使用Gson配合OKhttp解析服务器响应时，可能用`ResponseBody`的`string()`方法居多，代码大概是这样的`new Gson().fromJson(response.body().string(), T.class)`。我们项目中有些老代码中，甚至是这样的：先把`ResponseBody`中的流转为字符串，然后把字符串转为`JsonObject`，因为网络框架回调回来的是`JsonObject`，然后再把`JsonObejct`转为字符串，再用Gson将字符串转为Java对象。为啥最后一步要用Gson，因为方便。

在了解到`TypeAdapter`后，由于它是基于流的，所以可以无缝对接`ResponseBody`的`charStream()`。剩下的最后一个问题，就是项目中存在大量的已经定义好的POJO类，我一般称之为模型类，要为这么多数据模型类生成`TypeAdapter`，就只能用代码生成的方法了。

## APT

编译期代码生成，用得最多的工具就是APT（Annotation Processing Tool），大名鼎鼎的ButterKnife、EventBus3、Dagger2等也都是用它来实现编译期代码生成的。

Android开发中可以用`android-apt`或者`annotationProcesser`，前者已经终止维护了，后者是官方提供的。`android-apt`只支持`javac`编译，而`annotationProcesser`同时支持`javac`和`jack`。

这里其他的不多说，下一篇撸一个编译期自动生成`TypeAdapter`的小工具。