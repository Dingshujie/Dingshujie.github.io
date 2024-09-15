---
title: Kubernetes dynamic resource allocation
categories:
  - Kubernetes
  - device
tags:
  - kubernetes
  - ai-infrastructure
---
社区1.30为了解决之前dra框架的调度性能问题以及很难与cluster autoscaler协同的问题，引入了一个新的api以及sturctured parameters，来解耦scheduler和device controller的调用关系
引入的新api可以有以下几个增强：
- network-attached device。
- 在多个容器或者多个pod之间共享设备。之前的device plugin是不支持容器共享设备的，虽然也能在原有的api上使pod的容器支持共享，但是支持pods间的共享需要一个完整的api设计
- 