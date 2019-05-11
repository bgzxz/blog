---
title: "metrics领域的SLF4J(Micrometer)"
date: 2019-05-02T22:31:34+08:00
draft: false
---
# Micrometer是什么？

供应商中立的应用程序指标facade库

Micrometer为用于最流行的监控系统的仪表客户端提供了一个简单的facade，允许您在没有供应商锁定的情况下检测基于JVM的应用程序代码。可以理解为metrics领域的SLF4J。

# Micrometer实现了哪些功能？

* 多维度的Metrics

    timers, gauges, counters, distribution summaries, and long task timers
* 预配置的绑定

    Out-of-the-box instrumentation of caches, the class loader, garbage collection, processor utilization, thread pools, and more tailored to actionable insight
* spring集成

    Starting with Spring Boot 2.0, Micrometer is the instrumentation library powering the delivery of application metrics from Spring. Support is ported back to Boot 1.x through an additional library dependency
* 支持各种流行的监控系统

    内置支持 AppOptics, Azure Monitor, Netflix Atlas, CloudWatch, Datadog, Dynatrace, Elastic, Ganglia, Graphite, Humio, Influx/Telegraf, JMX, KairosDB, New Relic, Prometheus, SignalFx, Google Stackdriver, StatsD, and Wavefront

# [官网文档](https://micrometer.io/docs)

资料还是比较详细的,对metrics和实现相关[概念的介绍](https://micrometer.io/docs/concepts)

# 上代码

基于官网的源码改造的基于Prometheus和本地日志的[样例代码](https://github.com/bgzxz/micrometer-example)。




