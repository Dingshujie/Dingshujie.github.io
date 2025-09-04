---
title: Kubernetes dynamic resource allocation
categories:
  - Kubernetes
  - device
tags:
  - kubernetes
  - ai-infrastructure
---
最开始社区重新设计dra框架的初衷主要是由于当前的device plugin机制有很多局限性，很难以满足当前一些设备的发展，主要集中体现下以下几个场景：
- Device initialization:当你的应用中使用FPGA,我希望FPGA设备reprogramed or reconfured,这样应用无须完整的设备权限或者root权限。这样对集群来说更安全。当前的device plugin是没有这样的机制，可以对设备进行初始化的。
- Device cleanup:当应用程序结束，需要有个机制能给设备做clean up,这样不会有一些数据，参数残留，当前device plugin的流程中时没有post stop action
- Partial allocation: workload能仅需要部分的设备的能力，设备能进行切分，例如Nvidia mig 或者SR-IOV，很多云厂商提供了gpu虚拟化相关的能力），在某些场景下共享设备能提高设备的利用率，减少成本。当前的dp机制是不能提供partial allocation的，当前都是依赖设备在节点上动态分配好，现在一些厂商可以利用静态分配，先切分出一部分的设备，来达到Partial allocation的能力
- Optional allocation: 可能需要一些非强制分配的能力，如果有满足要求的设备则分配，如果不满足也可以调度
- support over fabric device: 原来dp更多的使用node-local resource，有些network-attached device对于网络可达的节点来说都可使用，当前dp的机制是没有办法分配的




社区1.30为了解决之前dra框架的调度性能问题以及很难与cluster autoscaler协同的问题，引入了一个新的api以及sturctured parameters，来解耦scheduler和device controller的调用关系
引入的新api可以有以下几个增强：
- network-attached device。原有的device plugin受限于必须是属于节点的资源。
- 在多个容器或者多个pod之间共享设备。之前的device plugin是不支持容器共享设备的，虽然也能在原有的api上使pod的容器支持共享，但是支持pods间的共享需要一个完整的api设计
- 可以自定义deivce的配置。当前都是通过pod annotation传递这些参数，可能还涉及到侵入修改dp或者csi driver
有别于传统的node-local device, dra with structured parameters
- dra driver以resourceslice的形式上报可用的资源，这个资源存储在apiserver，被调度器使用来给pod分配设备。
- 当用户需要使用相关资源的时候，可以先创建resourceclaim，在claim对象中定义要使用多少device，设备必须包含哪些capabillities
- 当有这么一个resource claim，scheduler通过resource slice来评估哪个节点能满足resource claim的诉求。
- 