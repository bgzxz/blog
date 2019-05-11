---
title: "实现应用程序可观察性的3种方式"
date: 2019-05-01T22:31:34+08:00
draft: false
---

项目上线报错了，

线上功能报错了，

用户投诉页面卡顿了，

系统故障导致业务数据异常了
。。。

作为程序员，在日常的工作中总是要面对如上的哪些问题。只有更多的了解系统的内部状态信息，才能更快更精确的定位和解决这些问题。那么如何让系统提供更多的有用信息了。

提升系统的可观测性可以从以下三个方法中获取：

* Logging - 纪录事件

* Metrics － 纪录指标数据事件

* Tracing － 纪录系统之间的因果关系事件

# [参考资料](https://www.dotconferences.com/2017/04/adrian-cole-observability-3-ways-logging-metrics-tracing)