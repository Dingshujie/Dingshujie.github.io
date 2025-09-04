---
title: gateway API inference extension
categories:
  - Kubernetes
  - Scheduler
  - LLM-d
  - ai-gateway
tags:
  - kubernetes
  - ai-infrastructure
---
大模型的飞速发展，越来越多的人在Kubernetes上部署大模型推理工作负载。在大模型推理工作负载呈现出来的特征与传统L7的应用完全不同。当前传统的负载均衡算法例如轮训，最小连接，哈希环，优先级等算法都不大适用于AI模型后端。大模型推理的负载均衡策略与模型结构（pd分离，moe等），推理实例的wait queue里请求的个数，kv cache利用率，是否有加载LoRA适配器，prefix prompt match等等都强相关；同时chat请求和agent请求对可能对sla要求不同，这些都能影响到gateway负载均衡的策略。所以社区拓展了社区Gateway api, 使用Endpoint
Picker，拓展L7路由能力，希望能结合LLM的指标来做出更合理的路由决策，提升整体资源利用率以及减少成本。

### Endpoint Picker


