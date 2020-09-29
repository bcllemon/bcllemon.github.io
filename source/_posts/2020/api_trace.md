layout: post
title: "API 调用追踪"
date: "2020-09-29 10:45"
categories: 技术
---
# 目的
开发排查系统问题用得最多的手段就是查看系统日志，但是在分布式环境下使用日志定位问题还是比较麻烦，需要借助"全链路追踪ID"把上下文串联起来.

目前大多数分布式追踪系统的思想模型都来自 Google's Dapper 论文
![](/images/2020/api_trace_1.png)

* ID
64位的整数
* Trace Tree : 一次跟踪
* 树节点是Span，包括：
   * Span ID
   * 开始时间和结束时间
   * RPC时间数据
   * 应用自定义标识数据annotations
* 树节点之间的边是Span 和它的父节点之间的关系
* 没有父节点的Span是根Span (root span)
* 所有树节点都有共同的一个Trace ID 标识这次跟踪

# 方案
## 无入侵增加 traceId
使用 Logback 的 MDC 机制，在日志模板中加入 traceId 标识，取值方式为 %X{traceId}

1. 系统入口 创建 traceId 的值
2. 使用 MDC 保存 traceId
3. 修改 logback 配置文件模板格式添加标识 %X{traceId}
> MDC（Mapped Diagnostic Context，映射调试上下文）是 log4j 和 logback 提供的一种方便在多线程条件下记录日志的功能。

## 跨进程传递
1. 「dubbo服务」 使用 org.apache.dubbo.rpc.Filter 创建一个过滤器进行 traceId 传递
* 服务消费者：负责传递链路追踪 ID
* 服务提供者：负责接收 ID 并保存到 MDC 中
2. 基于 Spring 的Rest 服务
* 服务消费者: 自定义 Http 客户端,统一负责传递链路追踪 ID
* 服务提供者: 使用 Filter 进行 traceId 传递

## Spring Cloud 全家桶解决方案
Spring-Cloud-Sleuth是Spring Cloud的组成部分之一，为SpringCloud应用实现了一种分布式追踪解决方案，其兼容了Zipkin, HTrace和log-based追踪
